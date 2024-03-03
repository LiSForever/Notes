### CRUD

```xml
    <select id="getUserList" resultType="com.individuals.mybatisStudy.entity.User">
        select * from test.user
    </select>

    <select id="getUserById" parameterType="int" resultType="com.individuals.mybatisStudy.entity.User">
        select * from test.user where id = #{id}
    </select>

    <insert id="addUser" parameterType="com.individuals.mybatisStudy.entity.User">
        insert into test.user (id,name,pwd) values (#{id},#{name},#{pwd})
    </insert>

    <update id="updateUser" parameterType="com.individuals.mybatisStudy.entity.User">
        update test.user set name=#{name}, pwd=#{pwd} where id=#{id}
    </update>

    <delete id="deleteUser" parameterType="int">
        delete from test.user where id=#{id}
    </delete>
```

* 这里的resultType和parameterType可以根据需求设置为任意类型，而不是和com.individuals.mybatisStudy.entity.User紧紧绑定，也不是只能使用基本类型

```xml
<insert id="addUser" parameterType="map">
        insert into test.user (id,name,pwd) values (#{id},#{name},#{pwd})
    <!-- 但这里的id name pwd必须是map中有的键-->
</insert>
```

```java
public interface UserDAO {
    List<User> getUserList();

    User getUserById(int id);

    int addUser(User user);

    int updateUser(User user);

    int deleteUser(int id);
}
```

```java
public class UserDAOTest {
    public UserDAOTest() {
    }

    @Test
    public void test() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserDAO userDAO = (UserDAO)sqlSession.getMapper(UserDAO.class);
        List<User> userList = userDAO.getUserList();
        Iterator var4 = userList.iterator();

        while(var4.hasNext()) {
            User user = (User)var4.next();
            System.out.println(user);
        }

    }

    @Test
    public void testadd() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserDAO userDAO = (UserDAO)sqlSession.getMapper(UserDAO.class);
        int res = userDAO.addUser(new User(666, "ldaaaaa", "66666666666"));
        sqlSession.commit();
        System.out.println(res);
    }

    @Test
    public void testupdate() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserDAO userDAO = (UserDAO)sqlSession.getMapper(UserDAO.class);
        int res = userDAO.updateUser(new User(666, "lda", "23333"));
        sqlSession.commit();
        System.out.println(res);
    }

    @Test
    public void testde() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserDAO userDAO = (UserDAO)sqlSession.getMapper(UserDAO.class);
        int res = userDAO.deleteUser(666);
        sqlSession.commit();
        System.out.println(res);
    }
}
```

