# MyBatis

###### 配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="">
    <resultMap id="" type="" extends="">
        <id column="" jdbcType="" property="" />
    	<result column="" jdbcType="" property="" />
        <collection property="" resultMap="" columnPrefix=""/>
        <collection property="" ofType="">
            <id column="" jdbcType="" property="" />
            <result column="" jdbcType="" property="" />  
        </collection>
        <association property="" resultMap="" columnPrefix=""/>
        <association property="" javaType="">
            <id column="" jdbcType="" property=""/>
            <result  column="" jdbcType="" property=""/>
        </association>
    </resultMap>
    <select id="" resultMap=""></select>
    <update id="">
        <foreach collection="" item="" separator="" open="" close="" index=""></foreach>
    </update>
    <insert id=""></insert>
</mapper>
```

备住：

- mapper
  1. namespace：命名空间，对应接口的包+接口名
- resultMap
  1. id：resultMap唯一标识
  2. type：对应的实体类
  3. extends：继承其他resultMap
  4. --id：主键
     1. column：对应数据库字段
     2. jdbcType：数据库字段类型
     3. property：实体类属性
  5. --result：普通字段
     1. column：对应数据库字段
     2. jdbcType：数据库字段类型
     3. property：实体类属性
  6. --collection：一对多或者多对多
     1. property：实体类属性
     2. resultMap：关联结果集
     3. columnPrefix：前缀
     4. ofType：映射到list集合属性中pojo的类型
  7. --association：一对一
     1. property：实体类属性
     2. resultMap：关联结果集
     3. columnPrefix：前缀
     4. javaType：指定pojo中属性的类型
- select
  1. id：对应接口的方法名
  2. resultMap：关联resultMap
- update
  1. id：对应接口的方法名
  2. --foreach
     1. collection：接收@Param("")的值
     2. item：集合中元素迭代时的别名
     3. separator：元素之间的分隔符，例如在in()的时候，separator=","会自动在元素中间用“,“隔开，避免手动输入逗号导致sql错误，如in(1,2,)这样。该参数可选
     4. open：foreach代码的开始符号，一般是(和close=")"合用。常用在in(),values()时。该参数可选
     5. close：foreach代码的关闭符号，一般是)和open="("合用。常用在in(),values()时。该参数可选
     6. index：在list和数组中,index是元素的序号，在map中，index是元素的key，该参数可选
- insert
  1. id：对应接口的方法名

###### 经验

mybatis中resultMap和resultType的区别

mybatis中在查询进行select映射的时候，返回类型可以用resultType，也可以用resultMap。resultType是直接表示返回类型的,而resultMap则是对外部ResultMap的引用，但是resultType跟resultMap不能同时存在。

1. 使用resultType

   这些情况下,MyBatis 会在幕后自动创建一个 ResultMap,基于属性名来映射列到 JavaBean 的属性上。如果列名没有精确匹配,可以在列名上使用 select 字句的别名来匹配标签。

2. resultType对应的是java对象中的属性，大小写不敏感；resultMap对应的是对已经定义好了id的resultTupe的引用，key是查询语句的列名，value是查询的值，是对数据库列名的书写，所以大小写敏感；
   
3. 使用resultType的时候，要保证结果集的列名与java对象的属性相同，而resultMap则不用。

4. 另外，resultMap 元素，它是 MyBatis 中最重要最强大的元素，它能提供级联查询，缓存等功能。



###### 原理

1. 读取核心配置文件
2. 根据配置文件生成SqlSessionFactory工厂对象
3. 创建SqlSession
4. 调用Executor执行数据库操作&&生成具体SQL指令
5. 对查询结果二次封装
6. 提交与事务

------

[mybatis多层数据包含集合和对象的组装 - 简书 (jianshu.com)](https://www.jianshu.com/p/cb240c350193)
