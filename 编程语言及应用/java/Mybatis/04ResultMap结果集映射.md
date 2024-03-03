* 解决实体类属性名和字段名不一致的问题
* 暴力解决方式 sql语句 as起别名

```xml
<select id="getUserById" parameterType="int" resultType="com.individuals.mybatisStudy.entity.User">
        select * from test.user where id = #{id}
</select>



<resultMap id="UserMap" type="com.individuals.mybatisStudy.entity.User">
    <result column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="pwd" property="password"/>
</resultMap>
<select id="getUserById" parameterType="int" resultMap="resultMap">
        select * from test.user where id = #{id}
</select>
```

* 之前我们也使用过HashMap，但是Hassmap相当于写死刘返回类型，上面实现的是自定义的类型

```xml
<insert id="addUser" parameterType="map">
        insert into test.user (id,name,pwd) values (#{id},#{name},#{pwd})
    <!-- 但这里的id name pwd必须是map中有的键-->
</insert>
```

* ResultMap还能解决返回的一部分字段设置为对象的问题
