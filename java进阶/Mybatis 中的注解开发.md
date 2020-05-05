# Mybatis 中的注解开发

在 Mybatis 的注解开发中，常用的注解如下表所示：
@Intsert	实现新增
@Update	实现更新
@Delete	实现删除
@Select	实现查询
@Results	实现结果集封装
@ResultMap	实现引用 @Results 定义的封装
@One	实现一对一结果集封装
@Many	实现一对多结果集封装
@SelectProvider	实现动态 SQL 映射
@CacheNamespace	实现注解二级缓存的使用



### 1.Mybatis 使用注解实现单表 CURD

#### 1.1 在IUserDao接口中使用注解

```java
public interface IUserDao {

    /**
     * 查询所有用户
     * @return
     */
    @Select("select * from user")
    List<User> findAll();

    /**
     * 保存用户
     * @param user
     */
    @Insert("insert into user(username,address,sex,birthday)values(#{username},#{address},#{sex},#{birthday})")
    void saveUser(User user);

    /**
     * 更新用户
     * @param user
     */
    @Update("update user set username=#{username},sex=#{sex},birthday=#{birthday},address=#{address} where id=#{id}")
    void updateUser(User user);

    /**
     * 删除用户
     * @param userId
     */
    @Delete("delete from user where id=#{id} ")
    void deleteUser(Integer userId);

    /**
     * 根据id查询用户
     * @param userId
     * @return
     */
    @Select("select * from user  where id=#{id} ")
    User findById(Integer userId);

    /**
     * 根据用户名称模糊查询
     * @param username
     * @return
     */
//    @Select("select * from user where username like #{username} ")
    @Select("select * from user where username like '%${value}%' ")
    List<User> findUserByName(String username);

    /**
     * 查询总用户数量
     * @return
     */
    @Select("select count(*) from user ")
    int findTotalUser();
}
```

#### 1.2 测试类

```java
public class AnnotationCRUDTest {
    private InputStream in;
    private SqlSessionFactory factory;
    private SqlSession session;
    private IUserDao userDao;

    @Before
    public  void init()throws Exception{
        in = Resources.getResourceAsStream("SqlMapConfig.xml");
        factory = new SqlSessionFactoryBuilder().build(in);
        session = factory.openSession();
        userDao = session.getMapper(IUserDao.class);
    }

    @After
    public  void destroy()throws  Exception{
        session.commit();
        session.close();
        in.close();
    }


    @Test
    public void testSave(){
        User user = new User();
        user.setUsername("mybatis annotation");
        user.setAddress("北京市昌平区");

        userDao.saveUser(user);
    }

    @Test
    public void testUpdate(){
        User user = new User();
        user.setId(57);
        user.setUsername("mybatis annotation update");
        user.setAddress("北京市海淀区");
        user.setSex("男");
        user.setBirthday(new Date());

        userDao.updateUser(user);
    }


    @Test
    public void testDelete(){
        userDao.deleteUser(51);
    }

    @Test
    public void testFindOne(){
        User user = userDao.findById(57);
        System.out.println(user);
    }


    @Test
    public  void testFindByName(){
//        List<User> users = userDao.findUserByName("%mybatis%");
        List<User> users = userDao.findUserByName("mybatis");
        for(User user : users){
            System.out.println(user);
        }
    }

    @Test
    public  void testFindTotal(){
        int total = userDao.findTotalUser();
        System.out.println(total);
    }
}
```

注意，如果此时实体类的属性与数据库表列名不一致，那么我们应该使用@Results、@Result、@ResultMap 等注解，如下


```java
public interface UserMapper {
    /**
     * 查询所有用户
     *
     * @return
     */
    @Select("SELECT * FROM user")
    @Results(id = "UserMap",value = {//表示id为主键关联从表
            @Result(id = true,property = "userId",column = "id"),
            @Result(property = "userName",column = "username"),
            @Result(property = "userBirthday",column = "birthday"),
            @Result(property = "userSex",column = "sex"),
            @Result(property = "userAddress",column = "address"),
    })
    List<User> listAllUsers();
/**
 * 添加用户
 *
 * @param user
 * @return 成功返回1，失败返回0
 */
@Insert("INSERT INTO user(username,birthday,sex,address) VALUES(#{username},#{birthday},#{sex},#{address})")
@ResultMap("UserMap")
int saveUser(User user);
```
- `@Results` 注解用于定义映射结果集，相当于标签` <resultMap></resultMap>`。
  其中，id 属性为唯一标识。value 属性用于接收 @Result[] 注解类型的数组。
- `@Result` 注解用于定义映射关系，相当于标签` <id /> 和 <result />`
  其中，id 属性指定主键。property 属性指定实体类的属性名，column 属性指定数据库表中对应的列。
- `@ResultMap `注解用于引用 `@Results` 定义的映射结果集，避免了重复定义映射结果集。


### 2. Mybatis 使用注解实现多对一（一对一）@one

一个用户可以有多个账户，而一个账户只能对应一个用户。在 Mybatis 中，多对一是作为一对一来进行处理的。也就是说，虽然多个账户可以属于同一个用户（多对一），但是在实体类中，我们是在账户类中添加一个用户类的对象引用，以此来表明所属用户。(此时就相当于一对一）

#### 2.1 在Account类中添加一个主表实体User的对象引用（从表类）

```java
public class Account implements Serializable {
    private Integer id;
    private Integer uid;
    private Double money;
//多对一（mybatis中称之为一对一）的映射：一个账户只能属于一个用户
    private User user;

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }
```

#### 2.2 在IAccountDao接口中使用注解

```java
public interface IAccountDao {

    /**
     * 查询所有账户，并且获取每个账户所属的用户信息
     * @return
     */
    @Select("select * from account")
    @Results(id="accountMap",value = {
            @Result(id=true,column = "id",property = "id"),
            @Result(column = "uid",property = "uid"),
            @Result(column = "money",property = "money"),
            //这个注解是引入主表        FetchType(加载时机)  EAGER(立即加载)
            @Result(property = "user",column = "uid",one=@One(select="com.itheima.dao.IUserDao.findById",fetchType= FetchType.EAGER))
    })
    List<Account> findAll();

    /**
     * 根据用户id查询账户信息
     * @param userId
     * @return
     */
    @Select("select * from account where uid = #{userId}")
    List<Account> findAccountByUid(Integer userId);
}
```

- `@One` 注解相当于标签` <association></association>`，是多表查询的关键，在注解中用来指定子查询返回单一对象。其中，select 属性指定用于查询的接口方法，fetchType 属性用于指定立即加载或延迟加载，分别对应` FetchType.EAGER` 和` FetchType.LAZY`
- 在包含` @one` 注解的` @Result` 中，column 属性用于指定将要作为参数进行查询的数据库表列。



#### 2.3 测试类

```java
public class AccountTest {
    private InputStream in;
    private SqlSessionFactory factory;
    private SqlSession session;
    private IAccountDao accountDao;

    @Before
    public  void init()throws Exception{
        in = Resources.getResourceAsStream("SqlMapConfig.xml");
        factory = new SqlSessionFactoryBuilder().build(in);
        session = factory.openSession();
        accountDao = session.getMapper(IAccountDao.class);
    }

    @After
    public  void destroy()throws  Exception{
        session.commit();
        session.close();
        in.close();
    }

    @Test
    public  void  testFindAll(){
        List<Account> accounts = accountDao.findAll();
        for(Account account : accounts){
            System.out.println("----每个账户的信息-----");
            System.out.println(account);
            System.out.println(account.getUser());
        }
    }

}
```

### 3.Mybatis 使用注解实现一对多@many

- 一个用户对应多个账户

####  3.1 User实体类加入Account表的List集合引用

```java
/**
* 
* <p>Title: User</p>
* <p>Description: 用户的实体类</p>
* <p>Company: http://www.itheima.com/ </p>
*/
public class User implements Serializable {
    private Integer userId;
    private String userName;
    private Date userBirthday;
    private String userSex;
    private String userAddress;
//一对多关系映射：主表方法应该包含一个从表方的集合引用
private List<Account> accounts;
public List<Account> getAccounts() {
		return accounts;
}
public void setAccounts(List<Account> accounts) {
		this.accounts = accounts;
}
```



#### 3.2 IUserDao接口中使用注解

```java
public interface IUserDao {

    /**
     * 查询所有用户
     * @return
     */
    @Select("select * from user")
    @Results(id="userMap",value={
            @Result(id=true,column = "id",property = "userId"),
            @Result(column = "username",property = "userName"),
            @Result(column = "address",property = "userAddress"),
            @Result(column = "sex",property = "userSex"),
            @Result(column = "birthday",property = "userBirthday"),
            @Result(property = "accounts",column = "id",
                    many = @Many(select = "com.itheima.dao.IAccountDao.findAccountByUid",
                                fetchType = FetchType.LAZY))
    })
    List<User> findAll();

    /**
     * 根据id查询用户
     * @param userId
     * @return
     */
    @Select("select * from user  where id=#{id} ")
    @ResultMap("userMap")
    User findById(Integer userId);

    /**
     * 根据用户名称模糊查询
     * @param username
     * @return
     */
    @Select("select * from user where username like #{username} ")
    @ResultMap("userMap")
    List<User> findUserByName(String username);

}
```

- `@Many` 注解相当于标签` <collection></collection>`，是多表查询的关键，在注解中用来指定子查询返回对象集合。其中，select 属性指定用于查询的接口方法，fetchType 属性用于指定立即加载或延迟加载，分别对应 `FetchType.EAGER `和 `FetchType.LAZY`
- 在包含 `@Many` 注解的` @Result` 中，column 属性用于指定将要作为参数进行查询的数据库表列。
  
  

#### 3.3 测试类

```java
public class UserTest {
/**
* 测试查询所有
*/
@Test
public void testFindAll() {
List<User> users = userDao.findAll();
    //查看延时加载
// for(User user : users) {
// System.out.println("-----每个用户的内容-----");
// System.out.println(user);
// System.out.println(user.getAccounts());
// }
}
```

### 4 Mybatis 使用注解实现二级缓存

如果使用注解时想开启二级缓存，那么首先应该在 Mybatis 配置文件中开启全局配置

```xml
<settings>
    <!-- 开启缓存 -->
    <setting name="cacheEnabled" value="true"/>
</settings>
```


接着在持久层接口中使用注解即可

```xml
@CacheNamespace(blocking = true)
public interface UserMapper {
	// .....
}
```



- 二级缓存是 **Mapper** 映射级别的缓存，多个 SqlSession 去操作同一个 Mapper 映射的 SQL 语句，多个SqlSession 可以共用二级缓存，**二级缓存是跨 SqlSession 的**。

![二级缓存分析](https://img-blog.csdnimg.cn/20200206103408586.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ExMDkyODgyNTgw,size_16,color_FFFFFF,t_70#pic_center)