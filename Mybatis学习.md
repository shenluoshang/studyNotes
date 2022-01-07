# Mybatis学习

## 1、第一个Mybatis程序

mybatis中文官网 -> [MyBatis中文网](https://mybatis.net.cn/)

### 1.1、 搭建环境

搭建数据库

```
CREATE DATABASE mybatis;

USE mybatis;

CREATE TABLE user(
	id INT(20) NOT NULL,
	name VARCHAR(30) DEFAULT NULL,
	pwd VARCHAR(30) DEFAULT NULL,
	PRIMARY KEY(id)
)ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO `user`(id,`name`,pwd) VALUES(1,'魏洪志','123456');

INSERT INTO `user`(id,`name`,pwd) VALUES(2,'张誉壬','123456');

INSERT INTO `user`(id,`name`,pwd) VALUES(3,'刘盛阳','456123');
```



新建项目

1. 新建一个普通的maven项目
2. 删除src目录
3. 导入maven依赖

基础项目工程搭建完毕

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

<!--    父工程-->
    <groupId>com.whz</groupId>
    <artifactId>Mybatis-Study</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

<!--    导入依赖-->
    <dependencies>
<!--        mysql驱动-->
        <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.22</version>
        </dependency>
<!--        mybatis-->
        <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.6</version>
        </dependency>
        <!--        Junit-->
        <!-- https://mvnrepository.com/artifact/junit/junit -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.1</version>
            <scope>test</scope>
        </dependency>

    </dependencies>
</project>
```

### 1.2、 创建一个模块

- 编写mybatis的核心配置文件

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
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="201314"/>
            </dataSource>
        </environment>

    </environments>

</configuration>
```

- 编写mybatis工具类

```java
/**
 * sqlSessionFactory  --> sqlSession
 * @author whz
 */
public class MybatisUtils {

    private static SqlSessionFactory sqlSessionFactory;

    static {
        try {
            // 使用mybatis第一步，获取SQL Session Factory对象
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // 既然有了SqlSessionFactory，顾名思义，我们就可以从中获得SqlSession 的实例了
    // SqlSession 完全包含了面向数据库执行SQL 命令所需的所有方法


    public static SqlSession getSqlSessionFactory() {
        return sqlSessionFactory.openSession();
    }
}
```



### 1.3、 编写代码

- 实体类

```java
package com.whz.pojo;

public class User {
    private int id;
    private String name;
    private String pwd;

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

- Dao接口

```java
/**
 * @author w1890
 */
public interface UserDao {
    /**
     * 获取用户列表
     * @return 返回用户
     */
    List<User> getUserList();
}
```



- 接口实现类（由原来的UserDaoImpl转换成一个Mapper配置文件）

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace= 绑定一个对应的Dao/Mapper接口-->
<mapper namespace="com.whz.dao.UserDao">
<!--    select查询语句-->
    <select id="getUserList" resultType="com.whz.pojo.User">
        select * from mybatis.user
    </select>
</mapper>
```

### 1.4、测试

注意点：org.apache.ibatis.binding.BindingException: Type interface com.whz.dao.UserDao is not known to the MapperRegistry.

**MapperRegistry** 是什么

- Junit测试

```java
public class UserDaoTest {

    @Test
    public void test(){

        // 第一步 获得SQLSession对象
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        // 方式一：getMapper
        UserDao userDao = sqlSession.getMapper(UserDao.class);
        List<User> userList = userDao.getUserList();

        for (User user : userList) {
            System.out.println(user);
        }

        // 关闭SQLSession
        sqlSession.close();


    }
}
```



> 测试中可能会遇到的问题

1. 配置文件没有注册
2. 绑定接口错误
3. 方法名不对
4. 返回类型不对
5. maven导出资源问题

### 1.5、学习问题总结

#### 2.1、出现“1 字节的 UTF-8 序列的字节 1 无效”情况

> 原因分析

这是因为我们在配置xml等配置文件的时候，设置的字符编码集为“UTF-8”格式，而idea默认的字符编码为“gbk”格式，这就造成了程序内部的编码错误的现象

> 解决办法

1. 打开文件->设置->编辑器->文件编码
2. 将全局编码，项目编码，属性文件的默认编码，新的UTF-8文件的BOM都更改为“UTF-8”

<img src="Mybatis%E5%AD%A6%E4%B9%A0.assets/image-20220107113138955.png" alt="image-20220107113138955" style="zoom:80%;" />

3. 在maven项目中直接清除target目录重新编译生成

<img src="Mybatis%E5%AD%A6%E4%B9%A0.assets/image-20220107113755051.png" alt="image-20220107113755051"  />



#### 2.2 、“Error querying database. Cause: com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communicat”

> 出现现象

mybatis程序报错如下：

org.apache.ibatis.exceptions.PersistenceException:
Error querying database. Cause: com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure

The last packet successfully received from the server was 7,200 milliseconds ago. The last packet sent successfully to the server was 7,195 milliseconds ago.
The error may exist in com/kuang/dao/UserMapper.xml
The error may involve com.kuang.dao.UserDao.getUserList
The error occurred while executing a query
Cause: com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure

> 解决方法

将mybatis配置文件中的URL：useSSL=true 改为：useSSL=false即可



#### 2.3、出现“Could not find resource com.dao.UserMapper.xml”情况

> 原因分析

这是由于在 mybatis-config.xml 配置文件中 mapper 设置的路径用==**点**==表示造成的这会造成程序的识别有问题。

> 解决办法

将下面的代码

```xml
<mappers>
    <mapper resource="com.dao.UserMapper.xml"/>
	</mappers>
```

改为（斜线）

```xml
<mappers>
    <mapper resource="com/dao/UserMapper.xml"/>
	</mappers>
```



#### 2.4、出现“Could not find resource com/dao/UserMapper.xml”情况

> 原因分析

主要是因为我们的配置文件没有找到。在==Mybatis==中，约定大于配置。我们如果手动在==target== 目录加入UserMapper.xml 文件即可。

> 解决办法

把下面的复制到根目录下的pom.xml 文加下

```xml
<!--    在build中配置resources，来防止我们资源导出失败的问题-->
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

## 2、CRUD

### 2.1、namespace

namespace中的包名要和Dao/mapper接口的包名一致

### 2.2、select

选择，查询语句

- id：就是对应的 namespace 中的方法名
- resultType：Sql语句执行的返回值！
- parameter：参数类型！

> 步骤

1. 编写接口

```java
/**
* 通过ID查找用户
* @param id
* @return 返回用户的id
*/
User getUserById(int id);
```

2. 编写对应的mapper中的sql语句

```xml
<select id="getUserById" parameterType="int" resultType="com.whz.pojo.User">
    select *
    from mybatis.user
    where id = #{id}
</select>
```

3. 测试

```java
@Test
public void getUserById() {
SqlSession sqlSession = MybatisUtils.getSqlSession();

UserMapper mapper = sqlSession.getMapper(UserMapper.class);
User user = mapper.getUserById(1);
System.out.println(user);
    
sqlSession.close();
}
```

### 2.3、Insert



### 2.4、update



### 2.5、delete



### 2.6、总结

> - 测试代码的时候，要注意增删改需要提交事务！

