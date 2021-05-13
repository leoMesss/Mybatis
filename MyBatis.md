# 什么是 MyBatis？

MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

怎么获得Mybatis

maven仓库

```xml
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.3</version>
</dependency>
```

中文文档https://mybatis.org/mybatis-3/zh/index.html

# 持久化

是一个动词

数据持久化

+ 持久化即使将程序的数据在持久状态 和瞬时状态转化的过程
+ 内存：**断电即失**

+ 数据库(JDBC)，io文件持久化

**为什么需要持久化？**

+ 有一些对象，不能丢掉
+ 内存太贵

# 持久层

是一个名词

Dao层，Service层，Controller层

+ 完成持久化工作
+ 层界限十分明显

# 为什么需要Mybatis

+ 帮助程序员将数据存到数据库中

+ 方便易学
+ 传统的JDBC代码太复杂，简化，框架，自动化
+ 不用Mybatis也可以，更容易上手，技术没有**高低之分**
+ 优点：
	+ 简单易学：本身就很小且简单。没有任何第三方依赖，最简单安装只要两个jar文件+配置几个sql映射文件易于学习
	+ 灵活：mybatis不会对应用程序或者数据库的现有设计强加任何影响。 sql写在xml里，便于统一管理和优化。
	+ 解除sql与程序代码的耦合：通过提供DAO层，将业务逻辑和数据访问逻辑分离
		+ 提供xml标签，支持编写动态sql。

# 第一个Mybatis程序

## 搭建环境

搭建数据库

```sql
CREATE TABLE `user` (
  `id` int NOT NULL,
  `name` varchar(30) DEFAULT NULL,
  `pwd` varchar(30) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
INSERT INTO `user` VALUES(1,'leo1','1213142'),(2,'leo1','12342'),(3,'leo3','111342');
```

创建一个模块

+ 编写mybatis的核心配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--configuration核心配置文件-->
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?serverTimezone=UTC&amp;useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="###164"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="com/leo/dao/UserMapper.xml"/>
    </mappers>
</configuration>
```

+ 编写Mybatis工具类

```java
package com.leo.untils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

public class MybatisUtils {
    private static SqlSessionFactory sqlSessionFactory;
    static {
        try {
            String resource="mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    //既然有了 SqlSessionFactory，顾名思义，我们可以从中获得 SqlSession 的实例
    //SqlSession 提供了在数据库执行 SQL 命令所需的所有方法
    public static SqlSession getSqlSession(){
        return sqlSessionFactory.openSession();
    }
}

```

## 编写代码

+ 实体类

```java
package com.leo.pojo;

public class User {
    private int id;
    private String name;
    private String pwd;

    public User() {
    }

    public User(int id, String name, String pwd) {
        this.id = id;
        this.name = name;
        this.pwd = pwd;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", pwd='" + pwd + '\'' +
                '}';
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPwd() {
        return pwd;
    }

    public void setPwd(String pwd) {
        this.pwd = pwd;
    }
}

```

+ Dao接口

```java
package com.leo.dao;

import com.leo.pojo.User;

import java.util.List;

public interface UserDao {
    List<User> getUserList();
}

```

+ 接口实现类

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace=绑定一个对应的=Dao/Mapper接口-->
<mapper namespace="com.leo.dao.UserDao">
<!--Select查询语句-->
    <select id="getUserList" resultType="com.leo.pojo.User">
        select * from mybatis.user
    </select>
</mapper>
```

## 测试

注意点

+ url中指定serverTimezone

+ 记得mapper

+ 在pom中配置

	```
	<build>
	    <resources>
	        <resource>
	            <directory>src/main/resources</directory>
	            <includes>
	                <include>**/*.properties</include>
	                <include>**/*.xml</include>
	            </includes>
	            <filtering>true</filtering>
	        </resource>
	        <resource>
	            <directory>src/main/java</directory>
	            <includes>
	                <include>**/*.properties</include>
	                <include>**/*.xml</include>
	            </includes>
	            <filtering>true</filtering>
	        </resource>
	    </resources>
	</build>
	```

# CRUD

## namespace

namespace中的包名要和Dao/Mapper接口的包名一致

## select

选择查询语句：

+ id：就是对应的namespace中的方法名
+ resultType：Sql语句执行的返回值！一般返回一个类

+ parameterType：参数类型

1.编写接口

```java
User getUserById(int id);
```

2.编写对应得mapper中的sql语句

```xml
<select id="getUserById" parameterType="int" resultType="com.leo.pojo.User">
    select * from mybatis.user where  id=#{id};
</select>
```

3.测试

```java
@Test
public void getUserByid(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper UserMapper = sqlSession.getMapper(UserMapper.class);
    User user = UserMapper.getUserById(1);
    System.out.println(user);
    sqlSession.close();
}
```

注意增删改需要提交事务`sqlSession.commit();`，但是在`sqlSessionFactory.openSession(true)`设置true就可以自动提交,不需要手动提交事务

## insert

```xml
<insert id="addUser" parameterType="com.leo.pojo.User">
    insert into mybatis.user(id, name, pwd) values (#{id},#{name},#{pwd});
</insert>
```

## update

```xml
<update id="updateUser" parameterType="com.leo.pojo.User">
    update mybatis.user set name=#{name},pwd=#{pwd} where id=#{id};
</update>
```

## Delete

```xml
<delete id="deleteUser" parameterType="int">
    delete from mybatis.user where id=#{id};
</delete>
```

## 分析错误

+ 标签错误，标签使用不规范
+ 路径配置错误，在pom中配置mapper路径

+ 空指针错误，SqlSessionFactory在`private static SqlSessionFactory sqlSessionFactory;`声明之后不要再声明了。
+ 未注册mapper错误，记得在mybatis-config中注册mapper

+ maven资源没有配置好

## 利用HashMap传参

HashMap可以制造类似对象的存储键值对的数据结构，

```java
User getUserByIdMap(Map map);
```

```xml
<select id="getUserByIdMap" parameterType="map" resultType="com.leo.pojo.User">
    select * from mybatis.user where id=#{id}
</select>
```

```java
public void getUserByIdMap(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    HashMap<String, Object> map = new HashMap<String, Object>();
    map.put("id",2);
    System.out.println(mapper.getUserByIdMap(map));
    sqlSession.close();
}
```

采用传入键值对的方式，可以创造一个含有id的map对象，在mapper中可以像取对象元素一样取出，这样不规范，但是很多公司都在这么做

Map传递参数，直接在sql中取出key即可 【parameterType="map"】

对象传递参数，直接在sql中取对象的属性即可【parameterType="Object"】

多个参数用Map，或者注解

## 思考，模糊查询

1.java代码执行的时候，传递通配符% %

```sql
<select id="getUserByLike" parameterType="String" resultType="com.leo.pojo.User">
    select *from mybatis.user where name like #{name}
</select>
```

2.在sql拼接中使用通配符

```sql
select *from mybatis.user where name like concat('%',#{name},'%')
```

这种不安全，比如`List<User> userByLike = mapper.getUserByLike("");`会查出全表

# 配置解析

## 核心配置文件

MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息。

![image-20210419142101367](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210419142101367.png)

## 环境配置（environments）

MyBatis可以配置成适应多环境

不过：对于每个sqlsession，只能使用一种环境

创建多个环境后

可以用不同的getSqlSession方法来使用不同的环境连接数据库

```java
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment);
// sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream,"development");
```

来选择使用的环境

Mybatis的事务管理器有JDBC和MANAGED，默认是JDBC

连接池有UNPOOLED|POOLED|JNDI，默认连接池为pooled

## 属性（properties）

可以通过properties来实现引用配置文件

这些属性可以在外部进行配置，并可以进行动态替换。你既可以在典型的 Java 属性文件中配置这些属性，也可以在 properties 元素的子元素中设置。

编写一个配置文件

db.properties

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?serverTime=UTC&useSSL=false&useUnicode=true&characterEncoding=UTF-8
username=root
password=密码写自己的
```

**注意url不需要转义**

在核心配置文件中引入properties

记得按标签顺序来引用

![image-20210419232529998](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210419232529998.png)

```xml
<properties resource="db.properties">
    <property name="username" value="111"/>
    <property name="password" value="adaad"/>
</properties>
```

可以发现虽然设置了错误的`username`和`password`,但是仍然连接成功，说明引入properties时，优先使用`db.properties`这种外部文件

+ 可以直接引入外部文件
+ 可以增加一些属性配置
+ 如果两个文件有同一个字段，优先使用外部配置文件

## 类型别名（typeAliase）

![](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210504152958469.png)

+ 类型别名是为java类型设置一个短的名字
+ 存在的意义仅在于用来减少类完全限定名的冗余

在Mybatis配置文件中配置Aliase

第一种方法：配置`<typeAlias`标签

![image-20210504153701416](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210504153701416.png)

使用时用alias即可

![image-20210504153933195](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210504153933195.png)

第二种方法：指定一个包名，Mybatis会在包名下面搜索需要的Java Bean，比如：扫描实体类的包，它的默认别名就为这个类的类名，首字母小写

![image-20210504154248616](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210504154248616.png)

一般使用类名首字母小写来表示，大写也可以运行，但是小写表明这是一个包扫描的类型

![image-20210504154409343](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210504154409343.png)

## 设置（settings）

![image-20210504160738282](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210504160738282.png)

![image-20210504160751704](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210504160751704.png)



所以的Mybatis的配置

```xml
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

## 其他配置

- [typeHandlers（类型处理器）](https://mybatis.org/mybatis-3/zh/configuration.html#typeHandlers)
- [objectFactory（对象工厂）](https://mybatis.org/mybatis-3/zh/configuration.html#objectFactory)
- [plugins（插件）](https://mybatis.org/mybatis-3/zh/configuration.html#plugins)
	+ [MyBatis Generator Core](https://mvnrepository.com/artifact/org.mybatis.generator/mybatis-generator-core)
	+ [MyBatis Plus](https://mvnrepository.com/artifact/com.baomidou/mybatis-plus-boot-starter)

## 映射器

MapperRegistry：注册绑定我们的Mapper文件;

方式一：【推荐使用】

```xml
<!--每一个Mapper.xml 都需要在Mybatis核心配置文件中注册-->
<mappers>
    <mapper resource="com/leo/Dao/UserMapper.xml"/>
</mappers>
```

方式二：使用class文件绑定注册

```xml
<!--每一个Mapper.xml 都需要在Mybatis核心配置文件中注册-->
<mappers>
    <mapper class="com.leo.dao.UserMapper"/>
</mappers>
```

注意点：

+ 接口和他的Mapper配置文件必须同名！
+ 接口和他的Mapper配置文件必须在同一个包下



方式三：使用扫描包进行注入绑定

```xml
<!--每一个Mapper.xml 都需要在Mybatis核心配置文件中注册-->
<mappers>
    <package name="com.leo.dao"/>
</mappers>
```

注意点：

+ 接口和他的Mapper配置文件必须同名！
+ 接口和他的Mapper配置文件必须在同一个包下

## 生命周期和作用域（Scope）

![image-20210504171526441](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210504171526441.png)

生命周期类别是至关重要的，因为错误的使用会导致非常严重的并发问题。

### **SqlSessionFactoryBuilder**

+ 一旦通过他创建了SqlSessionFactory，就不需要他了
+ 局部变量

### **SqlSessionFactory**

+ 可以认为是数据库连接池

+ SqlSessionFactory一旦被创建就应该在应用的运行期间一直存在，**没有任何理由丢弃它或重新创建另一个实例。**

+ 因此 SqlSessionFactory 的最佳作用域是应用作用域。
+ 最简单的实现方式就是使用**单例模式**或者静态单例模式。

### SqlSession

+ 连接到连接池的一个请求
+ SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。

+ 用完之后需要赶紧关闭，否则资源被占用

![image-20210504172021131](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210504172021131.png)

这里每一个Mapper，就代表一个具体的业务

# 结果集映射

结果集映射是将sql查询结果映射到接收的实体类上

```xml
<select id="selectUsers" resultType="map">
  select id, username, hashedPassword
  from some_table
  where id = #{id}
</select>
```

上面的语句会将查询结果以以键值对的形式存储到HashMap

```xml
<select id="getUserById" parameterType="int" resultType="user">
    select id,name,pwd from mybatis.user where id=#{id}
</select>
```

对于我们的语句，同样会以键值对的形式存到User类中，但是如果发现sql结果的键与`resultType`无法对应，则会将能对应上的填上，不能对应上的设为空

即![image-20210505175210025](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210505175210025.png)

解决这一问题我们可以

1.改sql中的命名

```xml
<select id="getUserById" parameterType="int" resultType="user">
    select id,name,pwd as password from mybatis.user where id=#{id}
</select>
```

使得键值对改变来适应`resultType`

2.使用结果集映射

在mapper.xml中配置结果集映射

```xml
<resultMap id="UserMap" type="user">
    <result column="pwd" property="password"/>
</resultMap>
```

没必要把所有的字段都配上，Mybatis会自动检测能对应的字段，我们只需要配置不能对应上的

同时在

```xml
<select id="getUserById" parameterType="int" resultMap="UserMap">
    select id,name,pwd from mybatis.user where id=#{id}
</select>
```

中，我们只需要配置`resultMap`即可，不需要配置`resultType`

+ `resultMap` 元素是 MyBatis 中最重要最强大的元素

+ ResultMap 的设计思想是，对简单的语句做到零配置，对于复杂一点的语句，只需要描述语句之间的关系就行了。

+  `ResultMap` 的优秀之处——你完全可以不用显式地配置它们。 虽然上面的例子不用显式配置 `ResultMap`


# 日志

## 日志工厂

如果一个数据库操作，出现了异常，我们需要排错，日志就是最好的帮手

曾经：cout，debug

现在日志工厂

![image-20210506094522124](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210506094522124.png)

- SLF4J 
- LOG4J【掌握】
- LOG4J2
- JDK_LOGGING 
- COMMONS_LOGGING 
- STDOUT_LOGGING 【掌握】 
- NO_LOGGING

加上日志

在Mybatis-config.xml中配置使用日志

```xml
<settings>
    <setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```

![image-20210506113453142](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210506113453142.png)

## Log4j

+ Log4j是[Apache](https://baike.baidu.com/item/Apache/8512995)的一个开源项目，通过使用Log4j，我们可以控制日志信息输送的目的地是[控制台](https://baike.baidu.com/item/控制台/2438626)、文件、[GUI](https://baike.baidu.com/item/GUI)组件
+ 我们也可以控制每一条日志的输出格式
+ 通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程。
+ 这些可以通过一个[配置文件](https://baike.baidu.com/item/配置文件/286550)来灵活地进行配置，而不需要修改应用的代码。

1.先导入log4j包

```xml
<!-- https://mvnrepository.com/artifact/log4j/log4j -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>

```

2.log4j.properties

```properties
#将等级为DEBUG的日志信息输出到console和file这两个目的地，console和file的定义在下面的代码
log4j.rootLogger=DEBUG,console,file

#控制台输出的相关设置
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.Target = System.out
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%c]-%m%n

#文件输出的相关设置
log4j.appender.file = org.apache.log4j.RollingFileAppender
log4j.appender.file.File=./log/kuang.log
log4j.appender.file.MaxFileSize=10mb
log4j.appender.file.Threshold=DEBUG
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n

#日志输出级别
log4j.logger.org.mybatis=DEBUG
log4j.logger.java.sql=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.ResultSet=DEBUG
log4j.logger.java.sql.PreparedStatement=DEBUG
```

3.配置log4j为日志的实现

```xml
<settings>
    <setting name="logImpl" value="LOG4J"/>
</settings>
```

4.log4j的使用，直接测试运行刚才的查询

![image-20210506120031124](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210506120031124.png)

**简单使用**

在要使用

1.Log4j的类中，导入包import org.apache.log4j.Logger;

2.日志对象，参数为当前类的class

```java
static Logger logger = Logger.getLogger(UserDaoTest.class);
```

注意：出现蓝色问号打不开.log文件

![image-20210506122457576](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210506122457576.png)

 然后在filetype下添加*.log即可

![image-20210506122830681](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210506122830681.png)

3.使用

```java
logger.info("info:进入了testLog4j");
logger.debug("debug:进入了TestLog4j");
logger.debug("error:进入了TestLog4j");
```

![image-20210506123254372](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210506123254372.png)

默认情况下是以debug的形式

![image-20210506123121786](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210506123121786.png)

其实，logger.info,logger.debug,logger.debug用处是改变每行前面的中括号提示，方便对日志每行进行区分

# 分页

**为什么要分页**

+ 减少数据的处理量

## 使用Limit分页

```sql
语法：SELECT * FROM user LIMIT startIndex,pageSize
```

1.接口

```java
List<User> getUserLimit(HashMap<String,Integer> map);
```

使用map来传参，方便

2.配置mapper.xml

```xml
<select id="getUserLimit" parameterType="map" resultMap="UserMap">
    select id,name,pwd from mybatis.user LIMIT #{startIndex},#{pageSize}
</select>
```

3.测试

```java
public void getUserByLimit(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    HashMap<String, Integer> map = new HashMap<String, Integer>();
    map.put("startIndex",0);
    map.put("pageSize",2);
    List<User> user = mapper.getUserLimit(map);
    for (User user1 : user) {
        System.out.println(user1);

    }
}
```

![image-20210506142906360](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210506142906360.png)

**注意，LIMIT 后面的startIndex是从0开始的**

## 使用RowBounds分页

1.接口

```java
List<User> getUserByRowBound();
```

2.配置mapper.xml

```xml
<select id="getUserByRowBound" resultMap="UserMap">
    select id,name,pwd from mybatis.user
</select>
```

3.测试

```java
@Test
public void getUserByRowBound(){
    RowBounds rowBounds = new RowBounds(0, 2);
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    List<User> user = sqlSession.selectList("com.leo.Dao.UserMapper.getUserByRowBound", null, rowBounds);
    for (User user1 : user) {
        System.out.println(user1);
    }
}
```

![image-20210506150153613](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210506150153613.png)

## 分页插件

![image-20210506150614916](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210506150614916.png)

https://pagehelper.github.io/

# 使用注解开发

1.注解在接口上实现

```java
    @Select("select * from user")
    List<User> getUsers();
```

2.配置mapper

```xml
<mappers>
    <mapper class="com.leo.Dao.UserMapper"/>
</mappers>
```

3.测试

```java
@Test
public void test(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    List<User> users = mapper.getUsers();
    for (User user : users) {
        System.out.println(user);
    }
    sqlSession.close();
}
```

![image-20210508225136384](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210508225136384.png)

## Mybatis执行流程剖析

![image-20210508232440888](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210508232440888.png)

## 注解CRUD

我们可以在工具类创建的时候实现自动提交事务

设置true

```java
public static SqlSession getSqlSession() {
        return sqlSessionFactory.openSession(true);
}
```

编写接口，增加注释

```java
public interface UserMapper {
    @Select("select * from user")
    List<User> getUsers();
    @Select("select * from user where id=#{id} ")
    User getUserById(int id);
    @Insert("Insert into user(id,name,pwd) values(#{id},#{name},#{password})")
    void addUser(User user);
    @Update("update user set name=#{name},pwd=#{password} where id=#{id}")
    void update(User user);
    @Delete("delete from user where id=#{uid}")
    void deleteUser(@Param("uid") int id);
}
```

注册绑定到config中

```xml
<mappers>
    <mapper class="com.leo.Dao.UserMapper"/>
</mappers>
```

测试类

```java
import com.leo.Dao.UserMapper;
import com.leo.pojo.User;
import com.leo.untils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.List;

public class UserMapperTest {

    @Test
    public void getUsers(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        List<User> users = mapper.getUsers();
        for (User user : users) {
            System.out.println(user);
        }
        sqlSession.close();
    }
    @Test
    public void getUserById(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        User user = mapper.getUserById(1);
        System.out.println(user);
        sqlSession.close();
    }
    @Test
    public void addUser(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        User user = new User(7, "Hello", "124231");
        mapper.addUser(user);
        System.out.println(user);
        sqlSession.close();
    }
    @Test
    public void update(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        User user = new User(7, "aaaa", "124231");
        mapper.update(user);
        System.out.println(user);
        sqlSession.close();
    }
    @Test
    public void delete(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        mapper.deleteUser(7);
        sqlSession.close();
    }
}
```

## 关于@Param()注解

+ 基本类型的参数或者String类型，需要加上
+ 引用类型不需要加
+ 如果只有一个基本类型的话，可以忽略，但是建议大家都加上
+ 我们在SQL中引用的就是我们这里的@Param()中设定的属性名

## #{}和${}区别

#{}可以防止SQL注入，更安全

# Lombok

```
Project Lombok is a java library that automatically plugs into your editor and build tools, spicing up your java.
Never write another getter or equals method again, with one annotation your class has a fully featured builder, Automate your logging variables, and much more.
```

+ java库
+ 插件
+ build tools

## 安装方法

idea中安装插件

![image-20210509102135152](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210509102135152.png)

maven仓库配置依赖

```xml
<!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.16</version>
    <scope>provided</scope>
</dependency>
```



![](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210509101151262.png)

@Data:产生无参构造，getter,setter, toString,hashcode,equals

@AllArgsConstructor:全参构造，注意，显示构造会覆盖Data中的构造方法，即使用@AllArgsConstructor后，Data中的无参构造会消失，需要手动显式构造一个无参构造

@NoArgsConstructor：无参构造

# 多对一处理



1.导入lombok

```xml
<!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.16</version>
    <scope>provided</scope>
</dependency>
```

2.新建实体类Teacher，Student

3.建立Mapper接口

4.建立Mapper.xml文件

5.在核心配置文件中绑定注册我们的Mapper接口或者文件

6.测试查询是否能够成功

## 按照查询嵌套处理

**注意起别名，涉及到pojo的对象如果没有路径，那就是取了别名**

创建接口

```java
//查询所有的学生信息，以及对应的老师
public List<Student> getStudent();
```

 Mapper.xml

```xml
<select id="getStudent" resultMap="Student_teacher">
    select * from student
</select>
<select id="getTeacher" resultType="Teacher">
    select * from teacher where id=#{id}
</select>
<resultMap id="Student_teacher" type="Student">
    <association property="teacher" column="tid" javaType="Teacher" select="getTeacher"/>
</resultMap>
```

说明 由于需要查询两张表，我们按照子查询的方法来，即嵌套处理

在`getStudent`中，查询了student表中的三个字段，即`id`  `name` `tid`,这时利用resultMap来实现sql返回值映射到实体类`Teacher`上，即如上述代码块。

association标签的作用是将查询结果中的参数关联到某个实体类上，其标准的写法是

```xml
<association property='teacher' javaType='Teacher'>
    <id column="id" property="id"/>
    <result column="name" property="name"/>
</association>
```

为了理解`tid`怎么传入`getTeacher`,修改resultMap

```xml
<resultMap id="Student_teacher" type="Student">
    <association property="teacher" column="{uid=tid}" javaType="Teacher" select="getTeacher"/>
</resultMap>
```

其中uid指的是`getTeacher`中#{uid}，tid是teacher中的tid，当嵌套的sql中只有一个参数时，可以直接用column=teacher中的成员，而#{}中的变量名可以任意取，当嵌套的sql中有多个参数时，用column="{uid=teacher中一个成员,tid=teacher中另一个成员}"，`getTeacher`中所需的变量和uid和tid一一对应。不能对应的时候就没法查出。

比如我们上述修改后的例子，结果无法查出

![image-20210509193916632](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210509193916632.png)

再修改`getTeacher`中#{id}为#{uid}

![image-20210509194143392](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210509194143392.png)

## 按照结果嵌套处理

配置mapper.xml

```xml
<select id="getStudent2" resultMap="Student_teacher2">
	select s.id sid,s.name sname,t.name tname,t.id tid from student s,teacher t where s.tid=t.id
</select>
<resultMap id="Student_teacher2" type="Student" >
	<result property="id" column="sid"/>
	<result property="name" column="sname"/>
	<association property="teacher" javaType="Teacher">
		<result property="id" column="tid"/>
		<result property="name" column="tname"/>
	</association>
</resultMap>
```

注意将s.id,t.id,s.name,t.name 都取别名，因为在resultMap中，s.id,t.id表示的都是“id”，s.name,t.name表示的都是“name”，也就是说对于resultMap，查询语句变为`select id,name,name,id from student s,teacher t where s.tid=t.id`，匹配会失败。

注意取了别名sid，sname，tname，tid后，默认的resultMap匹配需要column名和实体类成员名一一对应，我们改完别名后一定要手动匹配result property。

```xml
<association property="teacher" javaType="Teacher">
    <result property="id" column="tid"/>
    <result property="name" column="tname"/>
</association>
```

这是将查询结果关联到Teacher中，property为Teacher中的成员，column则是查询的结果

**其实resultMap是用来映射查询结果和所需的实体类，也就是说，在select中查什么不重要，最终会在resultMap中将查出的结果整理映射到实体类中，resultMap很高明地将sql语句select查询和实体类映射这两部分开操作，给予了用户极大的自由**

# 一对多处理

## 按照结果嵌套处理

```xml
<select id="getTeacherById" resultMap="TeacherStudent">
    select s.id sid,s.tid stid,s.name sname, t.name tname,t.id tid from student s,teacher t where t.id=s.tid and t.id=#{id}
</select>
<resultMap id="TeacherStudent" type="Teacher">
    <result property="id" column="tid"/>
    <result property="name" column="tname"/>
    <collection property="students" ofType="Student">
        <result property="id" column="sid"/>
        <result property="name" column="sname"/>
        <result property="tid" column="stid"/>
    </collection>
</resultMap>
```

## 按照查询嵌套处理

```xml
<select id="getTeacherById2" resultMap="TeacherStudent2">
    select * from teacher where id=#{id}
</select>
<resultMap id="TeacherStudent2" type="Teacher">
    <result property="id" column="id"/>
    <collection property="students"  column="id" javaType="ArrayList" ofType="Student" select="getStudent"/>
</resultMap>
<select id="getStudent" resultType="Student">
    select * from student  where tid=#{id}
</select>
```

![image-20210510200507817](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210510200507817.png)

出现这种情况在collection里面我们使用了getTeacherById2中的`column="id"`,也就是把id给映射到了collection里面，破坏了隐式映射，所以需要进行显示映射，![image-20210510203441112](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210510203441112.png)

## 小结

关联-association【多对一】即在实现接口时需要返回对象list，但是在具体的对象的时候，一个对象里只含有一个包含的对象

集合-association【一对多】即实现接口时只需要返回一个对象，但是在对象里的某个成员是包含对象list

3.javaType & ofType

​			1.JavaType用来指定实体类中属性的类型

​			2.ofType用来指定映射到List或者集合中的pojo类型，泛型中的约束类型

注意点：

+ 保证SQL的可读性，尽量保证通俗易懂
+ 注意一对多和多对一中，属性名和字段的问题
+ 如果问题不好排查错误，可以使用日志，推荐使用Log4j

面试高频

+ Mysql引擎
+ InnoDB底层原理
+ 索引
+ 索引优化

# 动态SQL

**什么是动态SQL：动态SQL就是指根据不同的条件生成不同的SQL**

动态 SQL 是 MyBatis 的强大特性之一。如果你使用过 JDBC 或其它类似的框架，你应该能理解根据不同条件拼接 SQL 语句有多痛苦，例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。利用动态 SQL，可以彻底摆脱这种痛苦。

使用动态 SQL 并非一件易事，但借助可用于任何 SQL 映射语句中的强大的动态 SQL 语言，MyBatis 显著地提升了这一特性的易用性。

如果你之前用过 JSTL 或任何基于类 XML 语言的文本处理器，你对动态 SQL 元素可能会感觉似曾相识。在 MyBatis 之前的版本中，需要花时间了解大量的元素。借助功能强大的基于 OGNL 的表达式，MyBatis 3 替换了之前的大部分元素，大大精简了元素种类，现在要学习的元素种类比原来的一半还要少。

- if
- choose (when, otherwise)
- trim (where, set)
- foreach

## 搭建环境

```sql
CREATE TABLE `blog` (
  `id` varchar(50) NOT NULL COMMENT '博客id',
  `title` varchar(100) NOT NULL COMMENT '博客标题',
  `author` varchar(30) NOT NULL COMMENT '博客作者',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  `views` int NOT NULL COMMENT '浏览量'
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

## 创建一个基础工程

1.导包

2.编写配置文件

3.编写实体类

```java
@Data
public class Blog {
    private String id;
    private String title;
    private String author;
    private Date create_Time;
    private int views;
}
```

4.编写实体类对应Mapper

```java
public interface BlogMapper {
    public void addBlog(Blog blog);
}
```

```xml
<insert id="addBlog" parameterType="Blog">
    insert into mybatis.blog (id, title, author, create_time, views) VALUE (#{id}, #{title}, #{author}, #{create_Time}, #{views})
</insert>
```

5.工具类

```java
public class IdUtils {
    public static String getId(){
        return UUID.randomUUID().toString().replaceAll("-","");
    }
    @Test
    public void test(){
        System.out.println(getId());
    }
}
```

6.测试类

```java
@Test
public void addInitBlog(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
    Blog blog = new Blog();
    blog.setId(IdUtils.getId());
    blog.setTitle("Mybatis");
    blog.setAuthor("牛牛");
    blog.setCreate_Time(new Date());
    blog.setViews(9999);

    mapper.addBlog(blog);

    blog.setId(IdUtils.getId());
    blog.setTitle("Java");
    mapper.addBlog(blog);

    blog.setId(IdUtils.getId());
    blog.setTitle("Spring");
    mapper.addBlog(blog);

    blog.setId(IdUtils.getId());
    blog.setTitle("微服务");
    mapper.addBlog(blog);

    sqlSession.close();
}
```

## IF

**接口**

```java
public List<Blog> queryBlogIf(HashMap map);
```

mapper.xml

```xml
<select id="queryBlogIf" parameterType="map" resultType="Blog">
    select * from mybatis.blog where 1=1
    <if test="title != null">
        and title=#{title}
    </if>
    <if test="author !=null">
        and author=#{author}
    </if>
</select>
```

注意，<if test="map中的元素名 != null"，我也不大清楚是怎么直接拿到的值，但好像就是能拿到：）

测试类

```java
public void queryBlogIf(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
    HashMap map = new HashMap();
    map.put("title","Java");
    List<Blog> blogs = mapper.queryBlogIf(map);
    for (Blog blog : blogs) {
        System.out.println(blog);
    }
    sqlSession.close();
}
```

## Choose

**choose类似Switch，执行由上向下，但是自带break，执行到满足的when即返回，when类似case，otherwise类似default,在case没法匹配的时候，执行otherwise**

**接口**

```java
public List<Blog> queryBlogChoose(HashMap map);
```

Mapper.xml

```xml
<select id="queryBlogChoose" parameterType="map" resultType="Blog">
    select * from mybatis.blog
    <where>
        <choose>
            <when test="title !=null">
                title=#{title}
            </when>
            <when test="author !=null">
                author=#{author}
            </when>
            <otherwise>
                and views=#{views}
            </otherwise>
        </choose>
    </where>
</select>
```

测试类

```java
@Test
public void queryBlogChoose(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
    HashMap map = new HashMap();
    //map.put("title","Java");
    map.put("views","9999");
    List<Blog> blogs = mapper.queryBlogChoose(map);
    for (Blog blog : blogs) {
        System.out.println(blog);
    }
    sqlSession.close();
}
```

## Where

![image-20210511125156035](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210511125156035.png)

在之前的操作中，为了使用where我们用了一个恒等式来作为条件，这显然很不规范，因此，利用`where`标签来实现这一功能

***where* 元素只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。而且，若子句的开头为 “AND” 或 “OR”，*where* 元素也会将它们去除。**

```xml
<select id="queryBlogIf" parameterType="map" resultType="Blog">
    select * from mybatis.blog
    <where>
        <if test="title != null">
            and title=#{title}
        </if>
        <if test="author !=null">
            and author=#{author}
        </if>
    </where>
</select>
```

## Set

![image-20210511210150509](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210511210150509.png)

**接口**

```java
public void updateBlog(HashMap map);
```

Mapper.xml

```java
<update id="updateBlog" parameterType="map">
    update mybatis.blog
    <set>
        <if test="title != null">
            title=#{title},
        </if>
        <if test="author != null">
            author=#{author},
        </if>
    </set>
    <where>
        id=#{id}
    </where>
</update>
```

测试类

```java
    public void updateBlog(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
        HashMap map = new HashMap();
        map.put("title","Java好难");
        map.put("author","牛牛2");
        map.put("id","cd925ba6150d4cfcb1312ec7b037219c");
        mapper.updateBlog(map);
        sqlSession.close();
    }
```

## Trim

![image-20210511211709400](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210511211709400.png)

![image-20210511213416170](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210511213416170.png)

**参数有**

![image-20210511212832202](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210511212832202.png)



其中prefix表示关键字在拼接SQL的前面，值为关键字，suffix表示关键字在拼接SQL的后面，值为关键字，prefixOverrides表示去掉多余的后缀，值为需要去掉的后缀值，suffixOverrides表示去掉多余的前缀，值为需要去掉的前缀值

其实where和set都是Trim来实现的，Trim是用来解决Mybatis文档开头提出的![image-20210511211952576](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210511211952576.png)

**可以说，Trim就是用来解决拼接含前缀比如 where 1=1 `and`中的`and`和含后缀的SET title=?`,` author=? 中的后缀`,`会在拼接SQL的时候出现在最前面和最后面的问题**



## SQL片段

我们可以利用sql标签将一些功能的部分抽取出来方便进行复用

1.使用SQL标签抽取公共的部分

```xml
<sql id="ifTitleAuthor">
    <if test="title != null">
        and title=#{title}
    </if>
    <if test="author !=null">
        and author=#{author}
    </if>
</sql>
```

2.在需要使用的地方使用include标签

```xml
<select id="queryBlogIf" parameterType="map" resultType="Blog">
    select * from mybatis.blog
    <where>
        <include refid="ifTitleAuthor"/>
    </where>
</select>
```

注意事项

+ 最好基于单表来定义SQL片段
+ 不要存在where标签

## Foreach

**foreach**

![image-20210511195618172](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210511195618172.png)

**接口**

```java
public  List<Blog> queryBlogForeach(HashMap map);
```

Mapper.xml

```xml
<select id="queryBlogForeach" parameterType="map" resultType="Blog">
    select * from mybatis.blog
    <where>
        <foreach collection="ids" item="id" open="and (" separator="or" close=")">
            id=#{id}
        </foreach>
    </where>
</select>
```

测试类

```java
@Test
public void queryBlogForeach(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
    ArrayList<String> ids = new ArrayList<String>();
    HashMap map = new HashMap();
    ids.add("9b7ba2798523403799fc2342baee54c0");
    ids.add("cd925ba6150d4cfcb1312ec7b037219c");
    map.put("ids",ids);
    List<Blog> blogs = mapper.queryBlogForeach(map);
    for (Blog blog : blogs) {
        System.out.println(blog);
    }
    sqlSession.close();
}
```

# 缓存

```
查询：连接数据库，耗资源！
一次查询的结果，给暂存在一个直接可以取到的地方！---->内存:缓存

我们再次查询想查询数据的时候就直接走缓存，就不用再次查询
```

## Mybatis缓存

+ Mybatis包含一个非常强大的查询缓存特性，可以非常方便地定制和配置缓存。缓存可以极大的提升查询效率。
+ Mybatis系统中默认定义了两级缓存：一级缓存和二级缓存
	+ 默认情况下，只有一级缓存开启。（SqlSession级别的缓存，也称为本地缓存）
	+ 二级缓存需要手动开启和配置，他是基于namespace级别的缓存
	+ 为了提高扩展性，Mybatis定义了缓存接口cache，我们可以通过实现 cache接口来自定义二级缓存

## 一级缓存

+ 一级缓存也叫本地缓存：SqlSession
	+ 与数据库同一次会话期间查询到的数据回房子本地缓存中
	+ 以后如果需要获取相同数据，直接从缓存中那，没必要再去查询数据库

测试步骤

1.开启日志

2.测试

```java
    @Test
    public void selectUser(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        userMapper mapper = sqlSession.getMapper(userMapper.class);
        User user = mapper.selectUserById(1);
        System.out.println(user);
        System.out.println("===========================================");
        //mapper.update(new User(2,"leo!","1234"));
        //sqlSession.clearCache();
        user = mapper.selectUserById(1);
        System.out.println(user);
        sqlSession.close();
    }
```

![image-20210512154203444](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210512154203444.png)

缓存失效的情况：

1.查询不同的东西

2.增删改会改遍数据，增删改之后SqlSession中的缓存会刷新

去掉上面的注释后

![image-20210512154525094](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210512154525094.png)

3.手动清理缓存

![image-20210512154707111](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210512154707111.png)

小结：一级缓存默认是开启的，只在一次SqlSession中有效，也就是拿到连接和关闭连接之中。

**一级缓存就是一个map**

## 二级缓存

+ 二级缓存也叫全局缓存，一级缓存作用域太低了，所以诞生了二级缓存
+ 基于namespace级别的缓存，一个名称空间，对应一个二级缓存
+ 工作机制
	+ 一个会话查询一条数据，这个数据就会被放在当前会话的一级缓存中
	+ 如果当前会话关闭了，这个会话对应的一级缓存就没了；但是我们想要的是，会话关闭了，一级缓存中的数据被保存到二级缓存中
	+ 新的会话查询信息，就可以从二级缓存中获取内容
	+ 不同的mapper查出的数据会放在自己对应的缓存中

步骤：

1.开启全局缓存

```xml
<setting name="cacheEnable" value="true"/>
```

2.在Mapper中开启默认使用缓存

```xml
<!--在当前Mapper.xml中使用二级缓存-->
    <cache/>
```

直接用需要对实体类序列化，详见问题

参数

![image-20210512160727071](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210512160727071.png)

也可以显示地声明某个查询语句是否使用缓存

```xml
<select id="selectUserById" resultType="User" useCache="true">
    select * from mybatis.user where id =#{id}
</select>
```

测试

可以通过开启两个SqlSession来测试

```java
@Test
public void selectUser(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    userMapper mapper = sqlSession.getMapper(userMapper.class);
    User user = mapper.selectUserById(1);
    System.out.println(user);
    sqlSession.close();
    
    System.out.println("===========================================");

    SqlSession sqlSession2 = MybatisUtils.getSqlSession();
    userMapper mapper2 = sqlSession2.getMapper(userMapper.class);
    User user1 = mapper2.selectUserById(1);
    System.out.println(user1);
    sqlSession.close();
}
```

注意，只有当第一个SqlSession关闭的时候，才会把一级缓存的信息提交给二级缓存，即必须有`SqlSession.close()`才会提交

### 问题

我们会需要将实体类序列化，否则就会报错

```
Caused by: java.io.NotSerializableException: com.leo.pojo.User
```

序列化：`public class User implements Serializable {`

## 小结

+ 只有要开启了二级缓存，在同一个Mapper下就有效
+ 所有的数据都会先放在一级缓存中
+ 只有当会话提交，或者关闭的时候，才会提交到二级缓存中。

## 缓存原理

![image-20210512165907169](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210512165907169.png)

## 自定义缓存Mybatis-Ehcache

