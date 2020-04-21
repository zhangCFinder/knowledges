[TOC]
### 1. 使用数据库自带的生成器


Role.java实体类
```java
public class Role implements Serializable {
    private String roleId;
    private String name;
    private Integer status;
	public String getRoleId() {
		return roleId;
	}
	public void setRoleId(String roleId) {
		this.roleId = roleId;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public Integer getStatus() {
		return status;
	}
	public void setStatus(Integer status) {
		this.status = status;
	}
}
```

SysRoleDao.java

```java
public interface SysRoleDao {
	public int addRole(SysRole role);
}
```

mapper-role.xml
```xml
<insert id="addRole" parameterType="SysRole" useGeneratedKeys="true" keyProperty="roleId" keyColumn="role_id">
	insert into t_sys_role(
		name,status
	)
	values(
	        #{name,jdbcType=VARCHAR},
		#{status,jdbcType=VARCHAR},
	)
</insert>
```

注：
1. 添加记录能够返回主键的关键点在于需要在<insert>标签中添加以下三个属性
<insert useGeneratedKeys="true" keyProperty="id" keyColumn="id"></insert>。
useGeneratedKeys：必须设置为true，否则无法获取到主键id。
keyProperty：设置为POJO对象的主键id属性名称。
keyColumn：设置为数据库记录的主键id字段名称

2. 新添加主键id并不是在执行添加操作时直接返回的，而是在执行添加操作之后将新添加记录的主键id字段设置为POJO对象的主键id属性

```java

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations= {"classpath:config/spring-core.xml","classpath:config/spring-web.xml"})
public class TestDao{
	@Autowired
	SysRoleDao roleDao;
	@Test
	public void test() {
		SysRole role=new SysRole();
		role.setName("admin10");
		role.setStatus(1);
		System.out.println("返回结果:"+roleDao.addRole(role));
		System.out.println("主键id:"+role.getRoleId());
	}

```

打印结果:
> 返回结果:1
    主键id:12


### 2. 使用selectKey 



#### 数据库(如MySQL,SQLServer)支持auto-generated key field的情况 -- 不推荐

> **因为假如你使用一条INSERT语句插入多个行， LAST_INSERT_ID() 只会返回插入的第一行数据时产生的值。比如我插入了 3 条数据，它们的 id 分别是 21,22,23.那么最后我还是只会得到 21 这个值。**

```xml
<insert id="insertOne">
        insert into user (user_name) values(#{userName})
        <selectKey order="AFTER"  keyProperty="userId" resultType="integer">
            SELECT LAST_INSERT_ID()        
        </selectKey>
    </insert>
```

* selectKey中resultType属性指定期望主键的返回的数据类型

* keyProperty属性指定实体类对象接收该主键的字段名

* order属性指定执行查询主键值SQL语句是在插入语句执行之前还是之后(可取值:after和before)



#### 数据库(如Oracle)不支持auto-generated key field的情况

```xml

<insert id="add" parameterType="EStudent">  
    <selectKey keyProperty="id" resultType="_long" order="BEFORE">   
        select CAST(RANDOM * 100000 as INTEGER) a FROM SYSTEM.SYSDUMMY1  
    </selectKey> 
    insert into TStudent(id, name, age) values(#{id}, #{name}, #{age})
</insert>
```

