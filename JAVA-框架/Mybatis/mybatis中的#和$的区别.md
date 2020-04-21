> `#`相当于对数据 加上 双引号，`$`相当于直接显示数据


1. `#`将传入的数据都当成一个字符串，会对自动传入的数据加一个双引号。如：`order by #user_id#`，如果传入的值是111,那么解析成sql时的值为`order by "111"`, 如果传入的值是id，则解析成的sql为`order by "id"`.　　

2. `$`将传入的数据直接显示生成在sql中。如：`order by \$user_id\$`，如果传入的值是111,那么解析成sql时的值为`order by user_id,`  如果传入的值是id，则解析成的sql为`order by id`.　　

3.` #`方式能够很大程度防止sql注入。　　

4.` $`方式无法防止Sql注入。

5. `$`方式一般用于传入数据库对象，例如传入表名.　　

6. 一般能用`#`的就别用`$`.


MyBatis排序时使用order by 动态参数时需要注意，用`$`而不是`#`

默认情况下，使用#{}格式的语法会导致MyBatis创建预处理语句属性并以它为背景设置安全的值（比如?）。这样做很安全，很迅速也是首选做法，有时你只是想直接在SQL语句中插入一个不改变的字符串。比如，像`ORDER BY`，你可以这样来使用：`ORDER BY ${columnName}`
这里MyBatis不会修改或转义字符串。
要实现动态调用表名和字段名，就不能使用预编译了，需添加`statementType="STATEMENT"` 。
例：
```xml
<select id="getUser" resultType="java.util.Map" parameterType="java.lang.String" statementType="STATEMENT">
    select 
         ${columns}
    from ${tableName}
        where 
　　　　　　　　COMPANY_REMARK = ${company}
  </select>
```

取值说明：
1. `STATEMENT`:直接操作sql，不进行预编译，获取数据：${}  ——有 SQL 注入的风险

`#{xxx}` 就不能用了,需要换成`${xxx}`

sql 直接进行的字符串拼接，如果为字符串需要加上引号

 

2. `PREPARED`:预处理，参数，进行预编译，获取数据：#{ }——默认 

是使用的参数替换，也就是索引占位符，我们的`#`会转换为`?`再设置对应的参数的值。


3. `CALLABLE`:执行存储过程————`CallableStatement` 


重要：接受从用户输出的内容并提供给语句中不变的字符串，这样做是不安全的。这会导致潜在的SQL注入攻击，因此你不应该允许用户输入这些字段，或者通常自行转义并检查。