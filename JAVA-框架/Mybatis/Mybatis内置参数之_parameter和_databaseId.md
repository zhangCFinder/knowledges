
Mybatis中提供了两个常用的内置参数:
**_parameter和_databaseId**

当mybatis的核心配置文件中配置了**databaseIdProvider**:
```xml

<databaseIdProvider type="DB_VENDOR"> 
    <property name="MySQL" value="mysql"/> <!--//多个数据库提供商配置...--> </databaseIdProvider>
```

此时mybatis中内置的参数 **_databaseId** 中保存了用户所指定的对应的数据库厂商标识。

```xml
<select id="selectUsrs" databaseId="mysql" resultType="com.heiketu.pojo.Users">
    <if test="_databaseId == 'mysql'">
        select * from usrs where id = 2
    </if>
</select>
```

mybatis的另一个内置参数 **_parameter** 保存了对应传入的对象:

```xml
<insert id="insertData" parameterType="com.heiketu.pojo.Users">
    insert into usrs values(
      null,
      <if test="_parameter != null">
      #{_parameter.name},
      </if>
      #{_parameter.age},
      #{_parameter.address},
      #{_parameter.companyId}
    )
</insert>
```

此时, **_parameter参数**保存了**com.heiketu.pojo.Users**这个对象。所以可以通过OGNL表达式从 **_parameter参数**中获取到**Users**的对应属性值(也就是把 **_parameter**看作了**users**的别名)。