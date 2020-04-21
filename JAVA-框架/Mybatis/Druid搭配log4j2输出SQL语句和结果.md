[TOC]
## 一、引言
    其实Druid的内置了log4jdbc来显示SQL语句，虽然显示效果不如原生的log4jdbc效果好，但是因为内置所以不需要其他更多的配置。
## 二、使用
### 1. 创建基于druid的logger
```xml
<bean id="log-filter" class="com.alibaba.druid.filter.logging.Slf4jLogFilter">
<property name="connectionLogEnabled" value="false"/>
<property name="statementLogEnabled" value="false"/>
<property name="resultSetLogEnabled" value="true"/>
<property name="statementExecutableSqlLogEnable" value="true"/>
</bean>
```
###### a. resultSetLogEnabled表示是否显示结果集。
###### b. statementExecutableSqlLogEnable 表示是否显示SQL语句。
### 2. 在 DruidDataSource中配置
```xml
<!-- 数据连接池 -->
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
.....
<property name="filters" value="stat,wall"/>
......
<property name="proxyFilters">
<list>
<ref bean="log-filter"/>
</list>
</property>
</bean>
```

proxyFilters是代理filter的意思，将我们在第一步创建的log-filter写入进去。

## 三、log4j2中的设置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- Log4j 2.x 配置文件。每30秒自动检查和应用配置文件的更新； -->
<configuration status="warn" monitorInterval="30" strict="true" schema="Log4J-V2.2.xsd">
<Properties>
<Property name="logdir">${sys:catalina.base}/logs</Property>
</Properties>
<appenders>
<!-- 输出到控制台 -->
<console name="Console" target="SYSTEM_OUT">
<!-- 需要记录的级别 -->
<!-- <ThresholdFilter level="debug" onMatch="ACCEPT" onMismatch="DENY" /> -->
<PatternLayout pattern="%date{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %level [%C{36}.%M] - %msg%n"/>
</console>
<!-- 输出到文件，按天或者超过80MB分割 -->
<rollingFile name="RollingFile" fileName="conerstone.log"
filePattern="${logdir}/logs/$${date:yyyy-MM}/xjj-%d{yyyy-MM-dd}-%i.log.gz">
<!-- 需要记录的级别 -->
<!-- <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY" /> -->
<PatternLayout pattern="%date{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %level [%C{36}.%M] - %msg%n"/>
<policies>
<onStartupTriggeringPolicy/>
<timeBasedTriggeringPolicy/>
<sizeBasedTriggeringPolicy size="1 MB"/>
</policies>
</rollingFile>
</appenders>
<loggers>
<!-- 全局配置 -->
<root level="info">
<appenderRef ref="Console"/>
<appenderRef ref="RollingFile"/>
</root>
<logger name="org.springframework.web" level="debug" additivity="false">
<appenderRef ref="Console"/>
</logger>
<logger name="com.mc.core.service" level="debug" additivity="false">
<appender-ref ref="Console"/>
</logger>
<!-- druid配置 -->
<logger name="druid.sql.Statement" level="debug" additivity="false">
<appender-ref ref="Console"/>
</logger>
<logger name="druid.sql.ResultSet" level="debug" additivity="false">
<appender-ref ref="Console"/>
</logger>
</loggers>
</configuration>
```

**其中需要特别注意41行之后的代码，表示是否显示sql语句和结果**，如果不想显示结果，可以在第一步中将resultSetLogEnabled 改为false

如果是 properties文件，则配置如下：

```xml
#system log--------------------------------
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target=System.out
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{ISO8601} %l %c%n%p: %m%n
log4j.rootLogger=info
# Druid
log4j.logger.druid.sql=warn, stdout
log4j.logger.druid.sql.DataSource=warn, stdout
log4j.logger.druid.sql.Connection=warn, stdout
log4j.logger.druid.sql.Statement=warn, stdout
log4j.logger.druid.sql.ResultSet=warn, stdout


```
**其中把log4j.logger.druid.sql.Statement后面改成debug，然后 appender重新指定成文件，就能基本完成log sql的需要。不过他是连带预编译语句的创建，设置参数，执行这三种一起log的**



## 四，发现一个问题就是，当SQL语句出错时Druid是不会进行 ？ 替换的。 所以我们需要进行一些修改。

```java
// 使用如下类代替默认的Slf4jLogFilter 进行Filter注册.
/*
	datasource.setProxyFilters(new ArrayList<Filter>() {
		private static final long serialVersionUID = 1L;

		{
			add(constructFilter());
		}
	});
*/
public class DruidSlf4jLoggerFilterEx extends Slf4jLogFilter {

	private static final Logger LOG = LoggerFactory
			.getLogger(DruidSlf4jLoggerFilterEx.class);

	// 将日志信息导入到我们要求的位置
	@Override
	protected void statementLogError(String message, Throwable error) {
		LOG.error(message, error);
		// super.statementLogError(message, error);
	}

	// 在SQL语句执行出错时, 依然进行 ? 替换
	@Override
	protected void statement_executeErrorAfter(StatementProxy statement,
			String sql, Throwable error) {
		if (!this.isStatementLogErrorEnabled()) {
			return;
		}

		if (!isStatementExecutableSqlLogEnable()) {
			statementLogError("{conn-" + statement.getConnectionProxy().getId()
					+ ", " + stmtId(statement) + "} execute error. " + sql,
					error);
			return;
		}

		int parametersSize = statement.getParametersSize();
		if (parametersSize <= 0) {
			statementLogError("{conn-" + statement.getConnectionProxy().getId()
					+ ", " + stmtId(statement) + "} execute error. " + sql,
					error);
		}

		final List<Object> parameters = new ArrayList<Object>(parametersSize);
		for (int i = 0; i < parametersSize; ++i) {
			JdbcParameter jdbcParam = statement.getParameter(i);
			parameters.add(jdbcParam != null ? jdbcParam.getValue() : null);
		}

		/*	Druid源码	
				String dbType = statement.getConnectionProxy().getDirectDataSource()
						.getDbType();
				String formattedSql = SQLUtils.format(sql, dbType, parameters,
						this.getStatementSqlFormatOption());
				statementLogError(
						"{conn-" + statement.getConnectionProxy().getId() + ", "
								+ stmtId(statement) + "} execute error. "
								+ formattedSql, error);
		*/

		final String formattedSql = transSql(parameters, sql);
		statementLogError(
				"{conn-" + statement.getConnectionProxy().getId() + ", "
						+ stmtId(statement) + "} execute error. "
						+ formattedSql, error);
	}
	
	private String transSql(List<Object> parameters, String sql) {

		if (sql.indexOf("?") < 0) {

			return sql;

		}

		for (int i = 0; i < parameters.size(); i++) {
			sql = sql.replaceFirst("\\?", parameters.get(i) != null ? "\'"
					+ parameters.get(i).toString() + "\'" : "NULL");
		}
		return sql;
	}	

	private String stmtId(StatementProxy statement) {
		StringBuffer buf = new StringBuffer();
		if (statement instanceof CallableStatementProxy) {
			buf.append("cstmt-");
		} else if (statement instanceof PreparedStatementProxy) {
			buf.append("pstmt-");
		} else {
			buf.append("stmt-");
		}
		buf.append(statement.getId());

		return buf.toString();
	}


}

```
