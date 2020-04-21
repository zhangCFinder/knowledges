[TOC]
## 增

```xml
    <insert id="insertFenceDeviceBind"  parameterType="com.headaerospace.domain.FenceDeviceBind" >
        INSERT INTO fence_device_bind(Id,bindType,fenceId)
        VALUES
        <foreach collection="idList" item="id" separator=",">
            (#{id},#{fenceDeviceBind.bindType},#{fenceDeviceBind.fenceId})
         </foreach>
    </insert>
```
## 删
```xml
    <delete id="delFenceDeviceBind"  parameterType = "java.lang.String">
        delete from fence_device_bind where fenceId=#{fenceId,jdbcType=VARCHAR}
    </delete>
```
## 改
```xml
    <update id="updateMobiles" parameterType="com.headaerospace.domain.FenceDeviceBind">
        update mobiles
        <set>
            <if test="mobileName != null">mobileName=#{mobileName},</if>
            <if test="driver != null">driver=#{driver},</if>
            <if test="carNumber != null">carNumber=#{carNumber},</if>
            <if test="phone != null">phone=#{phone},</if>
            <if test="mobileDesc != null">mobileDesc=#{mobileDesc},</if>
            <if test="createTime != null">createTime=#{createTime}</if>
        </set>
        where mobileId=#{mobileId}
    </update>
```
## 查
```xml
<select id="queryMsgRT" resultType="java.util.Map">
    SELECT
          DATE_FORMAT(
            fm_out.`Epoch`,
            '%Y-%m-%d %H:%i:%S'
          ) AS msgTime,
          fm_out.`MobileID`,
          fm_out.`payloadName` AS msgType,
          fm_out.`msgSource`,
          fm_out.`Fields`
        FROM
          `from_mobile` fm_out
          RIGHT JOIN
            (SELECT
              fm_in.`MobileID`,
              MAX(Epoch) AS lastDate
            FROM
              `from_mobile` fm_in
              RIGHT JOIN (SELECT MobileID FROM mobiles) mobileIds
              ON fm_in.`MobileID`=mobileIds.MobileID
            GROUP BY mobileIds.`MobileID`) t
            ON fm_out.`MobileID` = t.MobileID
            AND fm_out.`Epoch` = t.lastDate
        WHERE fm_out.`valid` = 1
         AND fm_out.`payloadName`=#{msgType,jdbcType=VARCHAR}
          AND fm_out.`Epoch` IS NOT NULL
        GROUP BY fm_out.`Epoch`,
          fm_out.`MobileID`
    </select>
```

## if
参见笔记 **MyBatis Mapper.xml各种判断**

## choose, when, otherwise

有些时候，我们不想用到所有的条件语句，而只想从中择其一二。针对这种情况，MyBatis 提供了 choose 元素，它有点像 Java 中的 switch 语句。
```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE 1=1
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>

```

##  where,trim
```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG
  <where>
    <if test="state != null">
         state = #{state}
    </if>
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
  </where>
</select>
```

* where 元素知道只有在一个以上的if条件有值的情况下才去插入“WHERE”子句。而且，若最后的内容是“AND”或“OR”开头的，where 元素也知道如何将他们去除。

* 如果 where 元素没有按正常套路出牌，我们还是可以通过自定义 trim 元素来定制我们想要的功能。比如，和 where 元素等价的自定义 trim 元素为：
```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG
<trim prefix="WHERE" prefixOverrides="AND |OR ">
 <if test="state != null">
         state = #{state}
    </if>
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
</trim>
</select>
```

* prefix=""：前缀，trim标签体中是整个字符串拼后的结果，它给拼后的整个字符串加一个前缀。

* prefixOverrides=""：前缀覆盖，去掉整个字符串前面的多余字符，会忽略通过管道分隔的文本序列（注意此例中的空格也是必要的）。它带来的结果就是所有在 prefixOverrides 属性中指定的内容将被移除，并且插入 prefix 属性中指定的内容。

* suffix=""：后缀，它给拼后的整个字符串加一个后缀。

* suffixOverrides=""：后缀覆盖，去掉整个字符串后面多余的字符。



## set,trim

类似的用于动态更新语句的解决方案叫做 set。set 元素可以被用于动态包含需要更新的列，而舍去其他的。比如：
```xml
<update id="updateAuthorIfNecessary">
  update Author
    <set>
      <if test="username != null">username=#{username},</if>
      <if test="password != null">password=#{password},</if>
      <if test="email != null">email=#{email},</if>
      <if test="bio != null">bio=#{bio}</if>
    </set>
  where id=#{id}
</update>

```

这里，set 元素会动态前置 SET 关键字，同时也会消除无关的逗号，因为用了条件语句之后很可能就会在生成的赋值语句的后面留下这些逗号。

自定义 trim 元素
```xml
<trim prefix="SET" suffixOverrides=",">
  ...
</trim>
```
## foreach

foreach一共有三种类型，分别为List,Array,Map三种。

### foreach属性

* **item** 
循环体中的具体对象。支持属性的点路径访问，如item.age,item.info.details。 具体说明：在list和数组中是其中的对象，在map中是value。 该参数为必选。
* **collection**
要做foreach的对象，作为入参时，List<?>对象默认用list代替作为键，数组对象有array代替作为键，Map对象用map代替作为键。 当然在作为入参时可以使用@Param("keyName")来设置键，设置keyName后，list,array,map将会失效。 除了入参这种情况外，还有一种作为参数对象的某个字段的时候。举个例子： 如果User有属性List ids。入参是User对象，那么这个collection = "ids" 如果User有属性Ids ids;其中Ids是个对象，Ids有个属性List id;入参是User对象，那么collection = "ids.id" 上面只是举例，具体collection等于什么，就看你想对那个元素做循环。 该参数为必选。

* **separator**
元素之间的分隔符，例如在in()的时候，separator=","会自动在元素中间用“,“隔开，避免手动输入逗号导致sql错误，如in(1,2,)这样。该参数可选。

* **open**
foreach代码的开始符号，一般是(和close=")"合用。常用在in(),values()时。该参数可选。

* **close**
foreach代码的关闭符号，一般是)和open="("合用。常用在in(),values()时。该参数可选。
* **index**
在list和数组中,index是元素的序号，在map中，index是元素的key，该参数可选。

你可以将任何可迭代对象（如 List、Set 等）、Map 对象或者数组对象传递给 foreach 作为集合参数。当使用可迭代对象或者数组时，index 是当前迭代的次数，item 的值是本次迭代获取的元素。当使用 Map 对象（或者 Map.Entry 对象的集合）时，index 是键，item 是值。


#### List与Array示例
```xml
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```



#### map示例

map和List,array相比，map是用K,V存储的，在foreach中，使用map时，index属性值为map中的Key的值。
因为map中的Key不同于list,array中的索引，所以会有更丰富的用法。
```xml
<select id="sel_key_cols" resultType="int">
        select count(*) from key_cols where
        <foreach item="item" index="key" collection="map"
            open="" separator="AND" close="">${key} = #{item}</foreach>
</select>
```

可以看到这里用key=value来作为查询条件，对于动态的查询，这种处理方式可以借鉴。**一定要注意到\$和#的区别，\$的参数直接输出，#的参数会被替换为?，然后传入参数值执行。**

```java
DEBUG [main] - ==>  Preparing: select count(*) from key_cols where col_a = ? AND col_b = ? 
DEBUG [main] - ==> Parameters: 22(Integer), 222(Integer)
DEBUG [main] - <==      Total: 1
```
如果不考虑元素的顺序和map中Key，map和list,array可以拥有一样的效果，都是存储了多个值，然后循环读取出来。



#### 批量插入示例
将数据封装在一个List对象中，这样在映射文件里，我们就可以这样写：

```xml
<!-- 批量新增 -->
<insert id="insertBatch" parameterType="java.util.List">
    INSERT INTO sys_user(<include refid="columns"></include>) VALUES
    <foreach collection="list" item="u" separator=",">
        (#{u.userName}, #{u.userPassword}, #{u.nickName}, #{u.userTypeId}, #{u.isValid}, #{u.createdTime})
    </foreach>
</insert>
```

