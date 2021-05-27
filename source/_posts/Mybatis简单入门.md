---
title: Mybatis简单入门
date: 2020-10-29 15:14:51
tags:
- Mybatis
- 框架
- JAVA
categories:
- Mybatis
---

## Mybatis概述

mybatis 是一个优秀的基于 java 的持久层框架，它内部封装了 jdbc，使开发者只需要**关注 sql 语句本身**，而不需要花费精力去处理加载驱动、创建连接、创建 statement 等繁杂的过程。

mybatis 通过 xml 或注解的方式将要执行的各种 statement 配置起来，并通过 java 对象和 statement 中 sql 的动态参数进行映射生成最终执行的 sql 语句，最后由 mybatis 框架执行 sql 并将结果映射为 java 对象并返回。

采用 ORM 思想解决了实体和数据库映射的问题，对jdbc进行了封装，屏蔽了 jdbc api 底层访问细节，使我 们不用与 jdbc api 打交道，就可以完成对数据库的持久化操作。

也就是说，这是个在我们 java 代码与数据库打交道的一个框架。

## 回顾JDBC过程以及问题

```java
    public static void main(String[] args) {
        Connection conn = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        try{
            //加载驱动
            Class.forName("com.mysql.cj.jdbc.Driver");
            //通过驱动管理类来获取数据库的连接
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306","用户名","密码");
            //定义sql语句
            String sql = "select * from user where username = ?";
            //获取预处理statement
            ps = conn.prepareStatement(sql);
            //设置sql中的参数，下标从1开始
            ps.setString(1, "lehanbal");
            //执行查询语句
            rs = ps.executeQuery();
            //遍历结果集
            while(rs.next()){
                System.out.println(rs.getString("id") + "" + rs.getString("username"));
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            //注意关闭顺序,从小到大
            if(rs != null){
                try {
                    rs.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if(ps != null){
                try {
                    ps.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if(conn != null){
                try {
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            
        }
    }
```

### JDBC问题

1、数据库链接创建、释放频繁造成系统资源浪费从而影响系统性能，如果使用数据库链接池可解决此问题。

2、Sql 语句在代码中硬编码，造成代码不易维护，实际应用 sql 变化的可能较大，sql 变动需要改变 java 代码。

3、使用 preparedStatement 向占有位符号传参数存在硬编码，因为 sql 语句的 where 条件不一定，可能多也可能少，修改 sql 还要修改代码，系统不易维护。

4、对结果集解析存在硬编码（查询列名），sql 变化导致解析代码变化，系统不易维护，如果能将数据库记 录封装成 pojo 对象解析比较方便。

## Mybatis初体验

既然是持久层框架，肯定要有数据库环境，我使用的是mysql。

数据库创建表的sql语句和所插入的数据sql语句。

```sql
create table user
(
    id       int auto_increment
        primary key,
    username varchar(32)  not null comment '用户名称',
    birthday datetime     null comment '生日',
    sex      char         null comment '性别',
    address  varchar(256) null comment '地址'
)

INSERT INTO test.user (id, username, birthday, sex, address) VALUES (1, '懒汉', '2020-02-27 21:23:01', '女', '四川');
INSERT INTO test.user (id, username, birthday, sex, address) VALUES (2, '懒汉1', '2019-07-02 05:09:04', '男', '广西');
INSERT INTO test.user (id, username, birthday, sex, address) VALUES (3, '一号', '2017-05-06 11:25:10', '女', '广东');
INSERT INTO test.user (id, username, birthday, sex, address) VALUES (4, '二号', '2020-01-18 15:00:11', '男', '陕西');
INSERT INTO test.user (id, username, birthday, sex, address) VALUES (5, '屎号', '2015-05-07 13:44:32', '女', '贵州');
INSERT INTO test.user (id, username, birthday, sex, address) VALUES (6, '三号', '2010-11-08 04:53:06', '男', '云南');
```

数据库环境搭建好了之后就可以开始整Mybatis了。

我的目录结构：

![目录结构](https://gitee.com/lehanbal/blog-image/raw/master/img/目录结构.png)

本文使用Maven来配置Mybatis环境，创建好项目后在pom.xml当中添加以下依赖：

```xml
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.6</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.18</version>
        </dependency>
```

mysql是8.0.18，mybatis则是3.5.6，各位按照实际情况来操作。

然后我们需要编写一个Bean来封装我们所查询的数据，字段保持和数据库的内容一致，User类如下：

```java
package top.lehanbal.domain;

import java.io.Serializable;
import java.util.Date;

public class User implements Serializable {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", birthday=" + birthday +
                ", sex='" + sex + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}

```

之后我们需要写一个数据访问层的DAO接口，用于查询对应的用户数据。

```java
package top.lehanbal.dao;

import top.lehanbal.domain.User;
import java.util.List;

public interface UserDao {
    /**
     * 查询所有操作
     * @return
     */
    List<User> findAll();
}
```

按照常规步骤我们应该去写一个对应接口的实现类来实现对应的数据库查询，但是我们使用了Mybatis框架之后并不需要这么左，我们只需要编写对应的xml映射文件来映射对应的接口，让我们的代理方法能够找到对应的实现类。

映射文件的创建是有要求的，必须和dao层接口在相同的包内，并且要以dao层的接口命名文件名。

![Mybatis层次结构](https://gitee.com/lehanbal/blog-image/raw/master/img/Mybatis层次结构.png)

映射文件UserDao.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- Mybatis约束 -->
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="top.lehanbal.dao.UserDao">
    <!--配置查询所有-->
    <select id="findAll" resultType="top.lehanbal.domain.User">
        select * from user
    </select>
</mapper>
```

namespace是对应着在main包中对应的接口，指名到对应的接口，全类名。

select id指定是的该接口中的方法。

resultType指定的是查询到的内容所封装到的类。

之后便是sql查询语句。

上面只是一个映射关系的xml，我们还需要指定对应的数据库连接并且还要写清楚对应的映射配置位置的配置文档。为了方便加载这个配置文件，我们一般直接放在resource文件夹根目录下。

SqlMapConfig.xml配置文件编写：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="mysql">
        <environment id="mysql">
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/test"/>
                <property name="username" value="用户名"/>
                <property name="password" value="密码"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="top/lehanbal/dao/UserDao.xml"/>
    </mappers>
</configuration>
```

mapper指定了对应的映射文件xml的位置信息。框架会根据这个信息去寻找对应的xml配置文件，然后动态的声明代理方法来实现UserDao接口，来实现我们所需要的查询功能，并且根据xml配置文件指定的返回结果集封装到对应的Bean类之中。

当然我们也可以使用注解来完成映射关系。将mapper换成以下语句：

```xml
<mapper class="top.lehanbal.dao.UserDao"/>
```

当我们使用注解的方式时，指定了对应的类的位置，我们指向了我们声明的接口，注解上只需要写上对应的sql语句即可。

```java
package top.lehanbal.dao;

import org.apache.ibatis.annotations.Select;
import top.lehanbal.domain.User;

import java.util.List;

public interface UserDao {
    /**
     * 查询所有操作
     * @return
     */
    @Select("select * from user")
    List<User> findAll();
}

```

效果和使用了xml配置文件是一样的。

弄完上面就完事大吉了，我们写个测试方法：

```java
package top.lehanbal.test;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import top.lehanbal.dao.UserDao;
import top.lehanbal.domain.User;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

public class MybatisTest {
    public static void main(String[] args) throws IOException {
        //加载配置文件
        InputStream is = Resources.getResourceAsStream("SqlMapConfig.xml");
        //声明构造工厂构造器
        SqlSessionFactoryBuilder build = new SqlSessionFactoryBuilder();
        //使用配置文件和构造器来构造工厂
        SqlSessionFactory factory = build.build(is);
        //通过工厂来声明对应的Sql实体
        SqlSession session = factory.openSession();
        //创建代理对象
        UserDao userDao = session.getMapper(UserDao.class);
        List<User> users = userDao.findAll();
        for(User user : users){
            System.out.println(user);
        }
        session.close();
        is.close();
    }
}
```

来一张运行结果图

![结果](https://gitee.com/lehanbal/blog-image/raw/master/img/结果.png)