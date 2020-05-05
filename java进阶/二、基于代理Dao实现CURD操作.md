# 二、基于代理Dao实现CURD操作

**使用要求：**

1、持久层接口和持久层接口的映射配置必须在相同的包下

2、持久层映射配置中 mapper 标签的 namespace 属性取值必须是持久层接口的全限定类名

3、SQL 语句的配置标签`<select>,<insert>,<delete>,<update>`的 id 属性必须和持久层接口的

方法名相同。



### 1.首先在持久层(dao)接口添加新增方法

```java
import com.itheima.domain.QueryVo;
import com.itheima.domain.User;

import java.util.List;

/**
 *
 * 用户的持久层接口
 */
public interface IUserDao {

    /**
     * 查询所有用户
     * @return
     */
    List<User> findAll();

    /**
     * 保存用户
     * @param user
     */
    void saveUser(User user);

    /**
     * 更新用户
     * @param user
     */
    void updateUser(User user);

    /**
     * 根据Id删除用户
     * @param userId
     */
    void deleteUser(Integer userId);

    /**
     * 根据id查询用户信息
     * @param userId
     * @return
     */
    User findById(Integer userId);

    /**
     * 根据名称模糊查询用户信息
     * @param username
     * @return
     */
    List<User> findByName(String username);

    /**
     * 查询总用户数
     * @return
     */
    int findTotal();

    /**
     * 根据queryVo中的条件查询用户
     * @param vo
     * @return
     */
    List<User> findUserByVo(QueryVo vo);
}

```

### 2.在用户的映射配置文件mapper.xml中配置

#### MyBatis中resultType resultMap 区别

> 基本映射 ：（resultType）使用resultType进行输出映射，只有查询出来的列名和pojo中的属性名一致，该列才可以映射成功。（数据库，实体，查询字段,这些全部都得一一对应）高级映射 ：（resultMap） 如果查询出来的列名和pojo的属性名不一致，通过定义一个resultMap对列名和pojo属性名之间作一个映射关系。（高级映射，字段名称可以不一致，通过映射来实现

> resultType和resultMap功能类似 ，都是返回对象信息 ，但是resultMap要更强大一些 ，可自定义。因为resultMap要配置一下，表和类的一一对应关系，所以说就算你的字段名和你的实体类的属性名不一样也没关系，都会给你映射出来，

ResultType相对与ResultMap而言更简单一点。只有满足ORM（Object Relational Mapping，对象关系映射）时，即数据库表中的字段名和实体类中的属性完全一致时，才能使用，否则会出现数据不显示的情况。

`resultType：`

```xml
<select id="selectUser" parameterType="int" resultType="User">
	select * from user where id = #{id}
</select>
```

`resultMap：`

```xml
<!-- column：数据库中列名称，property：类中属性名称 -->
<resultMap type="User" id="userMap">
	<id column="id" property="id"/>
	<result column="name" property="name"/>
</resultMap>

<select id="selectUserList" resultMap="userMap" > 
	select * from user
</select>
```

#### 配置细节：

**parameterType 属性**：代表传入参数的类型，因为我们要传入的是一个类的对象，所以类型就写类的全名称。

**resultType** **属性**：用于指定结果集的类型。

**sql 语句中使用#{}字符**：它代表占位符，相当于原来 jdbc 部分所学的?，都是用于执行语句时替换实际的数据。

具体的数据是由#{}里面的内容决定的。

**\#{}中内容的写法**：

由于我们保存方法的参数是 一个 User 对象，此处要写 User 对象中的属性名称。它用的是 ognl 表达式。

**ognl 表达式**：

它是 apache 提供的一种表达式语言，全称是：Object Graphic Navigation Language 对象图导航语言

它是按照一定的语法格式来获取数据的。语法格式就是使用 #{对象.对象}的方式



在用户的映射配置文件mapper.xml中配置，**namespace和id共同组成一个固定位置的方法，方便使用查询**

```xml
<mapper namespace="com.itheima.dao.IUserDao"> 
<!-- 配置 查询结果的列名和实体类的属性名的对应关系 -->
    <resultMap id="userMap" type="uSeR">
        <!-- 主键字段的对应 -->
        <id property="userId" column="id"></id>
        <!--非主键字段的对应-->
        <result property="userName" column="username"></result>
        <result property="userAddress" column="address"></result>
        <result property="userSex" column="sex"></result>
        <result property="userBirthday" column="birthday"></result>
    </resultMap>


    <!-- 查询所有 -->//注意这里的dao层
    <select id="findAll" resultMap="userMap">
        <!--select id as userId,username as userName,address as userAddress,sex as userSex,birthday as userBirthday from user;-->
        select * from user;
    </select>

    <!-- 保存用户 -->
    <insert id="saveUser" parameterType="user">
        <!-- 配置插入操作后，获取插入数据的id -->
        <selectKey keyProperty="userId" keyColumn="id" resultType="int" order="AFTER">
            select last_insert_id();
        </selectKey>
        insert into user(username,address,sex,birthday)values(#{userName},#{userAddress},#{userSex},#{userBirthday});
    </insert>

    <!-- 更新用户 -->
    <update id="updateUser" parameterType="USER">
        update user set username=#{userName},address=#{userAddress},sex=#{userAex},birthday=#{userBirthday} where id=#{userId}
    </update>

    <!-- 删除用户-->
    <delete id="deleteUser" parameterType="java.lang.Integer">
        delete from user where id = #{uid}
    </delete>
    
    <!-- 根据id查询用户 -->
    <select id="findById" parameterType="INT" resultMap="userMap">
        select * from user where id = #{uid}
    </select>

    <!-- 根据名称模糊查询 -->
    <select id="findByName" parameterType="string" resultMap="userMap">
          select * from user where username like #{name}
        <!-- select * from user where username like '%${value}%'-->
   </select>

    <!-- 获取用户的总记录条数 -->
    <select id="findTotal" resultType="int">
        select count(id) from user;
    </select>

    <!-- 根据queryVo的条件查询用户 -->
    <select id="findUserByVo" parameterType="com.itheima.domain.QueryVo" resultMap="userMap">
        select * from user where username like #{user.username}
    </select>
</mapper>
```

### 3.添加测试类中的测试方法

```java
package com.itheima.test;

import com.itheima.dao.IUserDao;
import com.itheima.domain.QueryVo;
import com.itheima.domain.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.io.InputStream;
import java.util.Date;
import java.util.List;

/**
 * 测试mybatis的crud操作
 */
public class MybatisTest {

    private InputStream in;
    private SqlSession sqlSession;
    private IUserDao userDao;

    @Before//用于在测试方法执行之前执行
    public void init()throws Exception{
        //1.读取配置文件，生成字节输入流
        in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.获取SqlSessionFactory
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
        //3.获取SqlSession对象
        sqlSession = factory.openSession();
        //4.获取dao的代理对象
        userDao = sqlSession.getMapper(IUserDao.class);
    }

    @After//用于在测试方法执行之后执行
    public void destroy()throws Exception{
        //提交事务
        sqlSession.commit();
        //6.释放资源
        sqlSession.close();
        in.close();
    }

    /**
     * 测试查询所有
     */
    @Test
    public void testFindAll(){
        //5.执行查询所有方法
        List<User> users = userDao.findAll();
        for(User user : users){
            System.out.println(user);
        }

    }

    /**
     * 测试保存操作
     */
    @Test
    public void testSave(){
        User user = new User();
        user.setUserName("modify User property");
        user.setUserAddress("北京市顺义区");
        user.setUserSex("男");
        user.setUserBirthday(new Date());
        System.out.println("保存操作之前："+user);
        //5.执行保存方法
        userDao.saveUser(user);

        System.out.println("保存操作之后："+user);
    }

    /**
     * 测试更新操作
     */
    @Test
    public void testUpdate(){
        User user = new User();
        user.setUserId(50);
        user.setUserName("mybastis update user");
        user.setUserAddress("北京市顺义区");
        user.setUserSex("女");
        user.setUserBirthday(new Date());

        //5.执行保存方法
        userDao.updateUser(user);
    }

    /**
     * 测试删除操作
     */
    @Test
    public void testDelete(){
        //5.执行删除方法
        userDao.deleteUser(48);
    }

    /**
     * 测试删除操作
     */
    @Test
    public void testFindOne(){
        //5.执行查询一个方法
        User  user = userDao.findById(50);
        System.out.println(user);
    }

    /**
     * 测试模糊查询操作
     */
    @Test
    public void testFindByName(){
        //5.执行查询一个方法
        List<User> users = userDao.findByName("%王%");
//        List<User> users = userDao.findByName("王");
        for(User user : users){
            System.out.println(user);
        }
    }

    /**
     * 测试查询总记录条数
     */
    @Test
    public void testFindTotal(){
        //5.执行查询一个方法
        int count = userDao.findTotal();
        System.out.println(count);
    }


    /**
     * 测试使用QueryVo作为查询条件
     */
    @Test
    public void testFindByVo(){
        QueryVo vo = new QueryVo();
        User user = new User();
        user.setUserName("%王%");
        vo.setUser(user);
        //5.执行查询一个方法
        List<User> users = userDao.findUserByVo(vo);
        for(User u : users){
            System.out.println(u);
        }
    }
}
```

