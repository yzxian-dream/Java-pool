# MyBatis配置文件中的属性标签

### (1)使用properties配置数据库连接信息

可以在标签内部配置数据库连接信息，也可以通过外部文件来配置数据库连接信息。

**第一种url属性(不常用)**

URL属性：
　　URL:Uniform Resource Locator　统一资源定位符 可以唯一标志一个资源的位置
　　写法必须是
　　　　http://localhost:8080/mybatisserver/demo1Servlet
　　　　协议　　主机　　端口　URI
　　URI:Uniform Resource Identifier 统一资源标识符 是在应用中可以可以唯一标志一个资源的位置
　　URL>URI（精准性）

```xml
<properties url="file:///C:/Users/zoick/OneDrive/Tomorrow/%E6%A1%86%E6%9E%B6%E5%AD%A6%E4%B9%A0/MyBatis/day02/day02_eesy_02mybatisCRUD/src/main/resources/jdbcConfig.properties">

</properties>
```

**第二种resource属性（常用）**
用于指定配置文件的位置，是按照类路径来写的，必须存在于类路径下

```xml
<properties resource="jdbcConfig.properties">

</properties>
```

使用 resources 属性引入外部配置文件
编写配置文件 jdbcConfig.properties。配置文件名没有限制，但是配置文件一定要放在类路径resource下

```xml
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/db_mybatis
jdbc.username=root
jdbc.password=123456
```



- 修改 Mybatis 配置文



```xml
<!-- 引入外部文件  -->
    <properties resource="jdbcConfig.properties"/>
    <!--配置环境-->
<environments default="development">
    <environment id="development">
        <!-- 配置事务类型 -->
        <transactionManager type="JDBC"/>
        <!-- 配置数据源（连接池） -->
        <dataSource type="POOLED">
            <property name="driver" value="${jdbc.driver}"/>
            <property name="url" value="${jdbc.url}"/>
            <property name="username" value="${jdbc.username}"/>
            <property name="password" value="${jdbc.password}"/>
        </dataSource>
    </environment>
</environments>
```
### (2)使用typeAliases配置别名

**之前在编写映射文件的时候， `resultType` 这个属性可以写 `int、INT` 等，就是因为 Mybatis 给这些类型起了别名。Mybatis 内置的别名如表格所示：**如果我们也想给某个**实体类**指定别名的时候，就可以采用 `typeAliases` 标签。用法如下：****

```xml
<!--使用typeAliases配置别名，他只能配置domain中类的别名-->
    <typeAliases>
        <!--typeAlias用于配置别名，type属性指定的是实体类中的全限定类名。alias属性指定别名，当指定了别名后不在区分大小写-->
        <typeAlias type="top.zoick.domain.User" alias="user"></typeAlias>
    </typeAliases>
```

> typeAlias 子标签用于配置别名。其中 type 属性用于指定要配置的类的全限定类名（该类只能是某个domain实体类）, alias 属性指定别名。一旦指定了别名，那么别名就不再区分大小写。也就是说，此时我们可以在映射文件中这样写 resultType="user" ，也可以写 resultType="USER"。
> 

### (3)使用package配置别名

- **当我们有多个实体类需要起别名的时候，那么我们就可以使用 `package` 标签。**

```xml
 <typeAliases>
        <!--用于指定要配置别名的包，当指定后，该包下的实体类都会注册别名，并且类名就是别名，不再区分大小写-->
        <package name="top.zoick.domain"/>
    </typeAliases>
```

> **`package` 标签指定要配置别名的包，当指定之后，该包下的所有实体类都会注册别名，并且别名就是类名，不再区分大小写**

其中，配置映射文件位置的中也有package这个标签

```xml
   <mappers>
<!--        <mapper resource="top/zoick/dao/IUserDao.xml"/>-->
        <!--package标签是用于指定dao接口所在的包，当指定了之后就不需要再写mapepr以及resource或者class了-->
        <package name="top.zoick.dao"/>
    </mappers>
```

> **这样配置后，我们就无需一个一个地配置 Mapper 接口了。不过，这种配置方式的前提是映射配置文件位置必须和dao接口的包结构相同**





添加完属性标签后的配置文件：

*SqlMapConfig.xml*

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- mybatis主配置文件-->
<configuration>
    <!--配置properties
        可以在标签内部配置数据库连接信息 也可以通过外部文件来配置数据库连接信息
        resource 属性：（常用）
                用于指定配置文件的位置，是按照类路径来写的，必须存在于类路径下
        URL属性：
            URL:Uniform Resource Locator    统一资源定位符 可以唯一标志一个资源的位置
            写法必须是
                http://localhost:8080/mybatisserver/demo1Servlet
                协议      主机    端口     URI
            URI:Uniform Resource Identifier 统一资源标识符 是在应用中可以可以唯一标志一个资源的位置
            URL>URI（精准性）
    -->
    <properties url="file:///C:/Users/zoick/OneDrive/Tomorrow/%E6%A1%86%E6%9E%B6%E5%AD%A6%E4%B9%A0/MyBatis/day02/day02_eesy_02mybatisCRUD/src/main/resources/jdbcConfig.properties">
<!--        <property name="driver" value="com.mysql.cj.jdbc.Driver"/>-->
<!--        <property name="url" value="jdbc:mysql://localhost:3306/eesy_mybatis"/>-->
<!--        <property name="username" value="root"/>-->
<!--        <property name="password" value="root"/>-->
    </properties>
    
    <!--使用typeAliases配置别名，他只能配置domain中类的别名-->
    <typeAliases>
        <!--typeAlias用于配置别名，type属性指定的是实体类中的全限定类名。alias属性指定别名，当指定了别名后不在区分大小写-->
<!--    <typeAlias type="top.zoick.domain.User" alias="user"></typeAlias>-->

        <!--用于指定要配置别名的包，当指定后，该包下的实体类都会注册别名，并且类名就是别名，不再区分大小写-->
        <package name="top.zoick.domain"/>
    </typeAliases>
    
    

    <!-- 配置环境-->
    <environments default="mysql">
        <!-- 配置mysql的环境-->
        <environment id="mysql">
            <!-- 配置事务的类型-->
            <transactionManager type="jdbc"></transactionManager>
            <!-- 配置数据源（连接池）-->
            <dataSource type="POOLED">
                <!-- 配置链接数据库的四个基本信息-->
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件-->
    <mappers>
<!--        <mapper resource="top/zoick/dao/IUserDao.xml"/>-->
        <!--package标签是用于指定dao接口所在的包，当指定了之后就不需要再写mapepr以及resource或者class了-->
        <package name="top.zoick.dao"/>
    </mappers>


</configuration>
```