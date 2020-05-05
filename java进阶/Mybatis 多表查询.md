# Mybatis 多表查询

在说多表查询之前，首先要了解表之间的关系共有以下几种：

- 一对一 ： 人和身份证就是一对一的关系。一个人只能有一个身份证号，而一个身份证号只能属于一个人。
- 多对一（一对多）：学生和班级就是多对一（一对多）的关系。一个班级有多个学生，而一个学生只属于一个班级。
- 多对多 ： 学生和任课老师就是多对多的关系。一个学生可以有多个任课老师，而一个任课老师可以有多个学生
  



## 1.一对一的关系映射

### 示例

查询所有操作,同时获取到用户下所有账户的信息（多表查询常用方法）

### sql 语句

sql 语句查询所有账户的时候同时获得当前账户的所有信息

```sql
select u.*,a.id as aid,a.uid,a.money from account a ,user u where u.id=a.uid;
```

sql 语句查询所有账户的时候同时获得当前账户的所地址和姓名

```sql
select a.*,u.username,u.address from account a ,user u where u.id=a.uid;
```

### 1. 从表实体应该包含一个主表实体的对象引用

在Account类中

```java
//从表实体应该包含一个主表实体的对象引用
    private User user;

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }
```

### 2.IAccountDao的接口中

```java
public interface IAccountDao {

    /**
     * 查询所有账户
     * @return
     */
    List<Account> findAll();
}
```

### 3.AccountDao的mapper中

```xml
<mapper namespace="top.zoick.dao.IAccountDao">
    <!--定义封装account和user的resultMap-->
    <resultMap id="accountUserMap" type="account">
        <id property="id" column="aid" />
        <result property="uid" column="uid" />
        <result property="money" column="money" />
        <!--一对一的关系映射，配置封装user的内容 column中指名从表的外键 property="user"指的是单个实体类的引用-->
        <association property="user" column="uid" javaType="User">
            <id property="id" column="id"/>
            <result property="username" column="username"/>
            <result property="address" column="address"/>
            <result property="sex" column="sex"/>
            <result property="birthday" column="birthday"/>
        </association>
    </resultMap>

    <!--IAccountDao的查询所有-->
    <select id="findAll" resultMap="accountUserMap">
          select u.*,a.id as aid,a.uid,a.money from account a ,user u where u.id=a.uid;
    </select>
</mapper>
```

注意事项中的javaType=“User” 一定要指名主体表的实体类名然后column中指名从表的外键！！

```xml
<association property="user" column="uid" javaType="User">
```

### 4.AccountDao的测试类中

```java
@Test
public void findAll(){
    List<Account> accounts = accoutDao.findAll();
    for (Account account:accounts) {
       System.out.println("每一个account的信息");
       System.out.println(account);
       System.out.println(account.getUser());
     }
}
```

### 第二种方法是创建一个AccountUser类

不建议使用

------

## 2.一对多的关系映射

### 示例：用户和账户

一个用户可以有多个账户
　　一个账户只能属于一个用户（多个账户也可以属于同一个用户）

### 步骤：

​        1、建立两张表：用户表 帐户表
　　　　　　让用户表和账户表之前具备一对多的关系：需要使用外键在帐户表上添加
　　2、建立两个实体类：用户实体类和账户实体类
　　　　　　让用户和账户的实体类能体现出来一对多的关系
　　3、配置两个配置文件
　　　　　　用户的配置文件
　　　　　　账户的配置文件
　　4、实现配置：
　　　　　　当我们查询用户的时候，可以同时得到用户下所包含的信息
　　　　　　当我们查询账户时，我们可以同时得到账户所属的用户信息

### 创建表与相关数据

```sql
CREATE TABLE `account` (
  `ID` int(11) NOT NULL COMMENT '编号',
  `UID` int(11) default NULL COMMENT '用户编号',
  `MONEY` double default NULL COMMENT '金额',
  PRIMARY KEY  (`ID`),
  KEY `FK_Reference_8` (`UID`),
  CONSTRAINT `FK_Reference_8` FOREIGN KEY (`UID`) REFERENCES `user` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

#### 查询的sql语句

```sql
select * from user u left outer join account a on u.id = a.uid
```

left join（左连接）是left outer join的简写，返回左表中所有记录和右表中连接字段相等的记录，即返回的记录数和左表的记录数一样

![img](https://artzoick.github.io//post-images/mybatis/7.png)

### 1.主表实体中应该包含从表实体的集合引用

User类中

```java
@Getter
@Setter
@ToString
public class User implements Serializable {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;

    //一对多关系映射，主表实体应该包含从表实体的集合应用
    private List<Account> accounts;

}
```

### 2.UserDao的接口中

```java
//  查询所有用户
 List<User> findAll();
```

### 3.UserDao的mapper中

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="top.zoick.dao.IUserDao">
    <!--定义封装user和account的resultMap type主表实体类-->
    <resultMap id="userAccountMap" type="User">
        <id property="id" column="id"></id>
        <result property="username" column="username"></result>
        <result property="sex" column="sex"></result>
        <result property="address" column="address"></result>
        <result property="birthday" column="birthday"></result>
        <!--一对多的关系映射，配置user封装accounts的内容-->
        <!--其中的property指的是从表的集合引用 ofType从表实体类-->
        <collection property="accounts" ofType="Account">
            <id property="id" column="aid"></id>
            <result property="uid" column="uid"></result>
            <result property="money" column="money"></result>
        </collection>
    </resultMap>

    <!--配置查询所有  其中id不能乱写必须是dao接口中的方法  resultType写的是实体类的全路径-->
    <select id="findAll" resultMap="userAccountMap">
          select * from user u left outer join account a on u.id = a.uid
    </select>
</mapper>
```

注意：一对一用的是association，一对多以及多对多用的是collection。

### 4.UserDao的测试类中

```java
//查询所有（一个用户下的账号信息）
    @Test
    public  void findAll( ) throws Exception {
        List<User> users = userDao.findAll();
        for (User user:users
             ) {
            System.out.println("每个用户的信息");
            System.out.println(user);
            System.out.println(user.getAccounts());
        }
    }
```

### 5.成功运行（实现上述功能）

![img](https://artzoick.github.io//post-images/mybatis/8.png)

## 3.多对多的关系映射

### 示例：用户与角色

一个用户可以有多个角色
一个角色可以赋予多个用户

### 步骤：

1、建立两张表：用户表 用户表
　　　　　　让用户和角色表具有多对多的关系。需要使用中间表，中间表中包含各自的主键，在中间表中是外键。
　　2、建立两个实体类：用户实体类和角色实体类
　　　　　　让用户和角色的实体类能体现出来多对多的关系
　　　　　　各自包含对方一个集合引用
　　3、配置两个配置文件
　　　　　　用户的配置文件
　　　　　　角色的配置文件
　　4、实现配置：
　　　　　　当我们查询用户的时候，可以同时得到用户下所包含角色的信息
　　　　　　当我们查询角色时，我们可以同时得到角色所赋予的用户信息

**修改用户实体类 `User` ，并添加角色实体类 `Role`，为了能体现出多对多的关系，两个实体类都必须包含对方的一个集合引用（多对多关系其实我们看成是双向的一对多关系）**

### 表设计

#### 表设置如下：

```sql
CREATE TABLE `role` (
  `ID` int(11) NOT NULL COMMENT '编号',
  `ROLE_NAME` varchar(30) default NULL COMMENT '角色名称',
  `ROLE_DESC` varchar(60) default NULL COMMENT '角色描述',
  PRIMARY KEY  (`ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `user_role` (
  `UID` int(11) NOT NULL COMMENT '用户编号',
  `RID` int(11) NOT NULL COMMENT '角色编号',
  PRIMARY KEY  (`UID`,`RID`),
  KEY `FK_Reference_10` (`RID`),
  CONSTRAINT `FK_Reference_10` FOREIGN KEY (`RID`) REFERENCES `role` (`ID`),
  CONSTRAINT `FK_Reference_9` FOREIGN KEY (`UID`) REFERENCES `user` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
insert  into `role`(`ID`,`ROLE_NAME`,`ROLE_DESC`) values (1,'院长','管理整个学院'),(2,'总裁','管理整个公司'),(3,'校长','管理整个学校');
insert  into `user_role`(`UID`,`RID`) values (41,1),(45,1),(41,2);
```

#### 表数据如下：

![img](https://artzoick.github.io//post-images/mybatis/9.png)

![img](https://artzoick.github.io//post-images/mybatis/9_1.png)

![img](https://artzoick.github.io//post-images/mybatis/9_2.png)

#### sql语句(多表外链接查询语句)

```sql
select u.*, r.id as rid,r.role_name,r.role_desc from role r   
left outer join user_role ur on r.id = ur.rid   
left outer join user u on u.id=ur.uid  
```

注意其中每行末尾的空格
**目的：**为了查询角色下的用户信息
**步骤：**以role表（别名）为主表，左外连接user_role表（别名ur,此表为中间表），连接条件r.id = ur.rid 。
再以这两个表组合的表左外连接user表，连接条件u.id=ur.uid 。

**注意：**[数据库连接（内链接，外连接（左连接，右连接）](https://jingyan.baidu.com/article/60ccbceb9578f164cab197f4.html)

#### 查询结果

![img](https://artzoick.github.io//post-images/mybatis/10.png)

### 1.一个实体表中包含另一个实体表的集合引用

由于是多对多的关系所有不分从表和主表

```java
@Setter
@Getter
@ToString
public class Role implements Serializable {

    private Integer roleId;
    private String roleName;
    private String roleDesc;

    /**
     * 多对多的关系映射：一个角色可以赋予多个用户
     */
    private List<User> users;
}
```

### 2.IRoleDao接口中定义该方法

```java
public interface IRoleDao {
    /**
     * 查询所有角色
     * @return
     */
    List<Role> findAll();
}
```

### 3.IRoleDao的mapper中

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="top.zoick.dao.IRoleDao">
    <!--定义role表的resultmap-->
    <resultMap id="roleMap" type="role">
        <!--此处id的column由于sql语句中将role表中的id改为rid，故column也要变为rid-->
        <id property="roleId" column="rid"/>
        <result property="roleName" column="role_name"/>
        <result property="roleDesc" column="role_desc"/>
        <collection property="users" ofType="user">
            <id column="id" property="id"/>
            <result column="username" property="username"/>
            <result column="address" property="address"/>
            <result column="sex" property="sex"/>
            <result column="birthday" property="birthday"/>
        </collection>
    </resultMap>

    <!--查询所有-->
    <select id="findAll" resultMap="roleMap">
        select u.*, r.id as rid,r.role_name,r.role_desc from role r
         left outer join user_role ur on r.id=ur.rid
         left outer join user u on u.id = ur.uid
    </select>
</mapper>
```

### 4.RoleDao的测试类中

```java
@Test
    public void findAll(){
        List<Role> roles  = roleDao.findAll();
        for (Role role:roles
             ) {
            System.out.println("=====每个角色的信息=====");
            System.out.println(role);
            System.out.println(role.getUsers());
        }
    }
```

### 5.成功运行（实现上述功能）

### ![img](https://artzoick.github.io//post-images/mybatis/10_1.png)

### 