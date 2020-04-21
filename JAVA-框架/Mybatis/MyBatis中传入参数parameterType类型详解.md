[TOC]
### MyBatis的传入参数parameterType类型分两种

* 基本数据类型：int,string,long,Date;

* 复杂数据类型：类和Map

###  如何获取参数中的值:

* 基本数据类型：#{参数} 获取参数中的值  
* 复杂数据类型：#{属性名}  ，map中则是#{key}


### 案例：

1. 基本数据类型案例
* 单一参数
```xml

<sql id="Base_Column_List" > 
 id, car_dept_name, car_maker_name, icon,car_maker_py,hot_type 
 </sql> 
 
 <select id="selectByPrimaryKey" resultMap="BaseResultMap" parameterType="java.lang.Long" > 
 select 
 <include refid="Base_Column_List" /> 
 from common_car_make 
 where id = #{id,jdbcType=BIGINT} 
 </select>
```
* 多个参数，使用`@Param注解`，由于需要传入多个参数，在进行Mybatis配置时没有办法同时配置多个参数
此时不再需要写 ` parameterType`。
```xml

<select id="login"  resultType="com.pojo.User"> 
select * from us where name=#{name} and password=#{password} 
</select>
```



2. 复杂类型--map类型
```xml

<select id="queryCarMakerList" resultMap="BaseResultMap" parameterType="java.util.Map"> 
  select 
  <include refid="Base_Column_List" /> 
  from common_car_make cm 
  where 1=1 
  <if test="id != null"> 
   and cm.id = #{id,jdbcType=DECIMAL} 
  </if> 
 <if test="carMakerName != null"> 
   and cm.car_maker_name = #{carMakerName,jdbcType=VARCHAR} 
  </if> 
  <if test="hotType != null" > 
   and cm.hot_type = #{hotType,jdbcType=BIGINT} 
  </if> 
  ORDER BY cm.id 
 </select> 
```

3. 复杂类型--类类型
```xml
<update id="updateByPrimaryKeySelective" parameterType="com.epeit.api.model.CommonCarMake" > 
 update common_car_make 
 <set > 
  <if test="carDeptName != null" > 
  car_dept_name = #{carDeptName,jdbcType=VARCHAR}, 
  </if> 
  <if test="hotType != null" > 
   hot_type = #{hotType,jdbcType=BIGINT}, 
  </if> 
 </set> 
 where id = #{id,jdbcType=BIGINT} 
 </update> 
```

4. 复杂类型--map中包含数组的情况

```xml

<select id="selectProOrderByOrderId" resultType="com.epeit.api.model.ProOrder" parameterType="java.util.HashMap" > 
  select sum(pro_order_num) proOrderNum,product_id productId,promotion_id promotionId 
  from pro_order 
  where 1=1 
  <if test="orderIds != null"> 
   and 
   <foreach collection="orderIds" item="item" open="order_id IN(" separator="," close=")"> 
    #{item,jdbcType=BIGINT} 
   </foreach> 
  </if> 
  GROUP BY product_id,promotion_id 
 </select> 
```


### 注解@Param：
1. 案例一：
    @Param(value="startdate") String startDate ：注解单一属性；这个类似于将参数重命名了一次
如调用mybatis的mapper.xml中配置sql语句（DAO层）
```java
    List<String> selectIdBySortTime(@Param(value="startdate")String startDate); 
```
则xml中的语句，需要配合@param括号中的内容:参数为startdate
```xml
<select id="selectIdBySortTime" resultType="java.lang.String" parameterType="java.lang.String"> 
 select distinct ajlcid from ebd_fh_ajlc where sorttime >= to_date(#{startdate,jdbcType=VARCHAR},'YYYY-MM-DD') and created_date=updated_date 
 and keyvalue in (select distinct companyname from ebd_fh_company_list where isupdate='0') 
 </select> 
```
2. 案例二：注解javaBean
    @Param(value="dateVo") DateVo dateVo；则需要注意编写的参数
```java
    List<String> selectIds(@Param(value="dateVo")DateVo dateVo); 
```

对应的mapping文件

```xml
<select id="selectIds" resultType="java.lang.String" parameterType="com.api.entity.DateVo"> 
select distinct ajlcid from ebd_fh_ajlc where sorttime >= to_date(#  {dateVo.startDate,jdbcType=VARCHAR},'YYYY-MM-DD') and created_date=updated_date 
 and keyvalue in (select distinct companyname from ebd_fh_company_list where isupdate='0') 
</select> 
```


