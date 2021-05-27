---
title: 实现一个自己的简单Mybatis
date: 2020-10-29 15:16:27
tags:
- Mybatis
- 框架
- JAVA
categories:
- Mybatis
---

# 实现一个自己的简单Mybatis

没有什么比自己直接实现一遍框架能够更让人看清楚这个框架是个啥了，手撕开始。

最终的整体目录结构如下：

![目录结构](D:%5CWorkspace%5Cgitbook%5Crecord_source_code%5CMybatis%5C%E5%AE%9E%E7%8E%B0%E4%B8%80%E4%B8%AA%E8%87%AA%E5%B7%B1%E7%9A%84%E7%AE%80%E5%8D%95Mybatis%5C%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.png)

首先解析一下Mybatis入门的时候所使用到的这些类都是个啥：

```java
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

1. Resources类：

   这个类是获取相关的配置资源的类，它下面的getResourceAsStream()方法是获取到我们项目下的xml文件，使这个xml文件变成输入流。

2. SqlSessionFactoryBuilder类：

   这是个SqlSessionFactory的构造类，类下的build()方法通过给他传递配置文件的字节流，他会读取到其中的配置文件的关键信息（驱动、sql的url、用户名以及密码）的配置信息，然后创建出SqlSessionFactory的实例对象。

3. SqlSessionFactory类：

   会根据SqlSessionFactoryBuilder类的build()方法传递的字节流信息来创建相关的SqlSessionFactory类实例。该类下有个openSession()方法，能够创造出对应的代理对象。

4. SqlSession类：

   代理方法的实现类，用于实现DAO层的接口方法，根据工厂的openSession()方法，从配置文件中获取到DAO层接口的映射信息，通过Proxy.newProxyInstance()方法动态的声明我们的代理方法，实现代理方法就需要一个方法集成InvocationHandler接口并且实现它的invoke()方法。

整体流程如上所述，我并没有讲全，在第2步的时候，我们是读到了xml文件，但是我们怎么解析它呢？

我们使用 dom4j 和 jaxen 两个 jar 包来实现xml文件的解析，下面是maven的依赖配置：

```xml
    <dependencies>        
		<dependency>
            <groupId>dom4j</groupId>
            <artifactId>dom4j</artifactId>
            <version>1.6.1</version>
        </dependency>

        <dependency>
            <groupId>jaxen</groupId>
            <artifactId>jaxen</artifactId>
            <version>1.1.6</version>
        </dependency>
    </dependencies>
```

有个小坑，dom4j 2.0.0版本往上解析xml的方式变了，为了能和博文的XML解析的工具类对应上所以推荐用以上的依赖配置。

我们注重的是框架的执行过程，所以xml的解析过程我就不会去说了，直接提供工具类。

我把上面提到的4点中的细节补全。

在第二点中，我们通过XML解析到了配置文件的关键数据，那么我们就需要将这些数据存储到一个资源类之中，先看看我们的配置文件需要存储一点什么：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <environments default="mysql">
        <environment id="mysql">
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/test"/>
                <property name="username" value="root"/>
                <property name="password" value="899421"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper class="top.lehanbal.dao.UserDao"/>
        <!--
        <mapper class="top.lehanbal.dao.UserDao"/>
        <mapper resource="top/lehanbal/dao/UserDao.xml"/>
        -->
    </mappers>
</configuration>
```

数据库的连接信息以及驱动，还有mappers的映射路径。

那么我们的资源类Configuration类就将这些资源封装。

```java
package top.lehanbal.mybatis.cfg;

import java.util.HashMap;
import java.util.Map;

public class Configuration {
    private String driver;
    private String url;
    private String username;
    private String password;
    private Map<String, Mapper> mappers = new HashMap<>();

    public String getDriver() {
        return driver;
    }

    public void setDriver(String driver) {
        this.driver = driver;
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public void setMappers(Map<String, Mapper> mappers) {
        this.mappers.putAll(mappers);
    }

    public Map<String, Mapper> getMappers() {
        return mappers;
    }
    
    @Override
    public String toString() {
        return "Configuration{" +
                "drive='" + driver + '\'' +
                ", url='" + url + '\'' +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}

```

这样我们每次需要相关的配置信息之后，就可以通过这个资源类获取到对应的配置信息。

Mapper类，这里用来存储对应我们再之前的配置文件所写下的映射地址信息以及对应的sql语句。queryString用于封装所存储的sql语句，resultType则是存储对应的类的相对地址。

```java
package top.lehanbal.mybatis.cfg;

public class Mapper {
    private String queryString;
    private String resultType;

    public String getQueryString() {
        return queryString;
    }

    public void setQueryString(String queryString) {
        this.queryString = queryString;
    }

    public String getResultType() {
        return resultType;
    }

    public void setResultType(String resultType) {
        this.resultType = resultType;
    }
}
```

我们需要对配置文件xml进行相关的处理才能读取到对应的文件信息，但是我们处理他们并不是我们所需要关心的重点，所以这里直接提供相关的xml解析工具类

```java
package top.lehanbal.mybatis.utils;

import org.dom4j.Attribute;
import org.dom4j.Document;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;
import top.lehanbal.mybatis.annotations.Select;
import top.lehanbal.mybatis.cfg.Configuration;
import top.lehanbal.mybatis.cfg.Mapper;
import top.lehanbal.mybatis.io.Resources;

import java.io.IOException;
import java.io.InputStream;
import java.lang.reflect.Method;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 *  用于解析配置文件
 */
public class XMLConfigBuilder {
    /**
     * 解析主配置文件，把里面的内容填充到DefaultSqlSession所需要的地方
     * 使用的技术：
     *      dom4j+xpath
     */
    public static Configuration loadConfiguration(InputStream config){
        try{
            //定义封装连接信息的配置对象（mybatis的配置对象）
            Configuration cfg = new Configuration();

            //1.获取SAXReader对象
            SAXReader reader = new SAXReader();
            //2.根据字节输入流获取Document对象
            Document document = reader.read(config);
            //3.获取根节点
            Element root = document.getRootElement();
            //4.使用xpath中选择指定节点的方式，获取所有property节点
            List<Element> propertyElements = root.selectNodes("//property");
            //5.遍历节点
            for(Element propertyElement : propertyElements){
                //判断节点是连接数据库的哪部分信息
                //取出name属性的值
                String name = propertyElement.attributeValue("name");
                if("driver".equals(name)){
                    //表示驱动
                    //获取property标签value属性的值
                    String driver = propertyElement.attributeValue("value");
                    cfg.setDriver(driver);
                }
                if("url".equals(name)){
                    //表示连接字符串
                    //获取property标签value属性的值
                    String url = propertyElement.attributeValue("value");
                    cfg.setUrl(url);
                }
                if("username".equals(name)){
                    //表示用户名
                    //获取property标签value属性的值
                    String username = propertyElement.attributeValue("value");
                    cfg.setUsername(username);
                }
                if("password".equals(name)){
                    //表示密码
                    //获取property标签value属性的值
                    String password = propertyElement.attributeValue("value");
                    cfg.setPassword(password);
                }
            }
            //取出mappers中的所有mapper标签，判断他们使用了resource还是class属性
            List<Element> mapperElements = root.selectNodes("//mappers/mapper");
            //遍历集合
            for(Element mapperElement : mapperElements){
                //判断mapperElement使用的是哪个属性
                Attribute attribute = mapperElement.attribute("resource");
                if(attribute != null){
                    System.out.println("使用的是XML");
                    //表示有resource属性，用的是XML
                    //取出属性的值
                    String mapperPath = attribute.getValue();//获取属性的值"com/itheima/dao/IUserDao.xml"
                    //把映射配置文件的内容获取出来，封装成一个map
                    Map<String,Mapper> mappers = loadMapperConfiguration(mapperPath);
                    //给configuration中的mappers赋值
                    cfg.setMappers(mappers);
                }else{
                    System.out.println("使用的是注解");
                    //表示没有resource属性，用的是注解
                    //获取class属性的值
                    String daoClassPath = mapperElement.attributeValue("class");
                    //根据daoClassPath获取封装的必要信息
                    Map<String,Mapper> mappers = loadMapperAnnotation(daoClassPath);
                    //给configuration中的mappers赋值
                    cfg.setMappers(mappers);
                }
            }
            //返回Configuration
            return cfg;
        }catch(Exception e){
            throw new RuntimeException(e);
        }finally{
            try {
                config.close();
            }catch(Exception e){
                e.printStackTrace();
            }
        }

    }

    /**
     * 根据传入的参数，解析XML，并且封装到Map中
     * @param mapperPath    映射配置文件的位置
     * @return  map中包含了获取的唯一标识（key是由dao的全限定类名和方法名组成）
     *          以及执行所需的必要信息（value是一个Mapper对象，里面存放的是执行的SQL语句和要封装的实体类全限定类名）
     */
    private static Map<String,Mapper> loadMapperConfiguration(String mapperPath)throws IOException {
        InputStream in = null;
        try{
            //定义返回值对象
            Map<String,Mapper> mappers = new HashMap<String,Mapper>();
            //1.根据路径获取字节输入流
            in = Resources.getResourceAsStream(mapperPath);
            //2.根据字节输入流获取Document对象
            SAXReader reader = new SAXReader();
            Document document = reader.read(in);
            //3.获取根节点
            Element root = document.getRootElement();
            //4.获取根节点的namespace属性取值
            String namespace = root.attributeValue("namespace");//是组成map中key的部分
            //5.获取所有的select节点
            List<Element> selectElements = root.selectNodes("//select");
            //6.遍历select节点集合
            for(Element selectElement : selectElements){
                //取出id属性的值      组成map中key的部分
                String id = selectElement.attributeValue("id");
                //取出resultType属性的值  组成map中value的部分
                String resultType = selectElement.attributeValue("resultType");
                //取出文本内容            组成map中value的部分
                String queryString = selectElement.getText();
                //创建Key
                String key = namespace+"."+id;
                //创建Value
                Mapper mapper = new Mapper();
                mapper.setQueryString(queryString);
                mapper.setResultType(resultType);
                //把key和value存入mappers中
                mappers.put(key,mapper);
            }
            return mappers;
        }catch(Exception e){
            throw new RuntimeException(e);
        }finally{
            in.close();
        }
    }

    /**
     * 根据传入的参数，得到dao中所有被select注解标注的方法。
     * 根据方法名称和类名，以及方法上注解value属性的值，组成Mapper的必要信息
     * @param daoClassPath
     * @return
     */
    private static Map<String,Mapper> loadMapperAnnotation(String daoClassPath)throws Exception{
        //定义返回值对象
        Map<String,Mapper> mappers = new HashMap<String, Mapper>();

        //1.得到dao接口的字节码对象
        Class daoClass = Class.forName(daoClassPath);
        //2.得到dao接口中的方法数组
        Method[] methods = daoClass.getMethods();
        //3.遍历Method数组
        for(Method method : methods){
            //取出每一个方法，判断是否有select注解
            boolean isAnnotated = method.isAnnotationPresent(Select.class);
            if(isAnnotated){
                //创建Mapper对象
                Mapper mapper = new Mapper();
                //取出注解的value属性值
                Select selectAnno = method.getAnnotation(Select.class);
                String queryString = selectAnno.value();
                mapper.setQueryString(queryString);
                //获取当前方法的返回值，还要求必须带有泛型信息
                Type type = method.getGenericReturnType();//List<User>
                //判断type是不是参数化的类型
                if(type instanceof ParameterizedType){
                    //强转
                    ParameterizedType ptype = (ParameterizedType)type;
                    //得到参数化类型中的实际类型参数
                    Type[] types = ptype.getActualTypeArguments();
                    //取出第一个
                    Class domainClass = (Class)types[0];
                    //获取domainClass的类名
                    String resultType = domainClass.getName();
                    //给Mapper赋值
                    mapper.setResultType(resultType);
                }
                //组装key的信息
                //获取方法的名称
                String methodName = method.getName();
                String className = method.getDeclaringClass().getName();
                String key = className+"."+methodName;
                //给map赋值
                mappers.put(key,mapper);
            }
        }
        return mappers;
    }
}
```

有了这些工具类，我们可以开始干点事了。

首先我们通过SqlSessionFactoryBuilder的build方法来使用工具类来解析配置文件，封装对应的Configuration对象，建造对应的设计工厂，也就是return相关的新的工厂类。

```java
package top.lehanbal.mybatis.sqlsession;

import top.lehanbal.mybatis.cfg.Configuration;
import top.lehanbal.mybatis.sqlsession.defalut.DefaultSqlSessionFactory;
import top.lehanbal.mybatis.utils.XMLConfigBuilder;
import java.io.InputStream;

public class SqlSessionFactoryBuilder {
    public SqlSessionFactory build(InputStream config) {
        Configuration cfg = XMLConfigBuilder.loadConfiguration(config);
        return new DefaultSqlSessionFactory(cfg);
    }
}
```

通过工厂类我们就可以通过openSession来创建对应的sqlSession对象。

```java
package top.lehanbal.mybatis.sqlsession.defalut;

import top.lehanbal.mybatis.cfg.Configuration;
import top.lehanbal.mybatis.sqlsession.SqlSession;
import top.lehanbal.mybatis.sqlsession.SqlSessionFactory;

public class DefaultSqlSessionFactory implements SqlSessionFactory {
    private Configuration cfg;

    public DefaultSqlSessionFactory(Configuration cfg){
        this.cfg = cfg;
    }

    @Override
    public SqlSession openSession() {
        return new DefaultSqlSession(cfg);
    }
}
```

sqlSession实例如下，getMapper方法来生成对应的代理对象，也就是动态的实现了DAO的实现方法。

```java
package top.lehanbal.mybatis.sqlsession.defalut;

import top.lehanbal.dao.UserDao;
import top.lehanbal.mybatis.cfg.Configuration;
import top.lehanbal.mybatis.sqlsession.SqlSession;
import top.lehanbal.mybatis.sqlsession.proxy.MapperProxy;
import top.lehanbal.mybatis.utils.DataSourceUtil;

import java.lang.reflect.Proxy;
import java.sql.Connection;
import java.sql.SQLException;

public class DefaultSqlSession implements SqlSession {
    private Configuration cfg;
    private Connection conn;

    public DefaultSqlSession(Configuration cfg){
        this.cfg = cfg;
        this.conn = DataSourceUtil.getConnection(cfg);
    }
    /**
     * 创建代理对象
     * @param
     * @return
     */
    @Override
    public <T> T getMapper(Class<T> daoIntefaceClass) {
        return (T) Proxy.newProxyInstance(daoIntefaceClass.getClassLoader(),
                new Class[]{daoIntefaceClass}, new MapperProxy(cfg.getMappers(), conn));
    }

    /**
     * 关闭连接
     */
    @Override
    public void close() {
        if(conn != null){
            try{
                conn.close();
            }catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

生产代理方法，这一步是需要我们手动去实现的，代理方法类必须实现InvocationHandler接口的invoke来实现方法增强。

```java
package top.lehanbal.mybatis.sqlsession.proxy;

import top.lehanbal.mybatis.cfg.Mapper;
import top.lehanbal.mybatis.utils.Executor;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.sql.Connection;
import java.util.Map;

public class MapperProxy implements InvocationHandler {
    private Map<String, Mapper> map;
    private Connection conn;

    public MapperProxy(Map<String, Mapper> map, Connection conn){
        this.map = map;
        this.conn = conn;
    }
    /**
     * 方法增强
     * @param proxy
     * @param method
     * @param args
     * @return
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //获取方法名
        String methodName = method.getName();
        String className = method.getDeclaringClass().getName();
        String key = className + "." + methodName;
        Mapper mapper = map.get(key);
        if(mapper == null) throw new IllegalAccessException();
        return new Executor().selectList(mapper, conn);
    }
}

```

代理方法还需要connection对象，所以我们可以写一个工具类获取连接对象：

```java
package top.lehanbal.mybatis.utils;

import top.lehanbal.mybatis.cfg.Configuration;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class DataSourceUtil {
    public static Connection getConnection(Configuration cfg){
        try{
            Class.forName(cfg.getDriver());
            return DriverManager.getConnection(cfg.getUrl(),cfg.getUsername(),cfg.getPassword());
        }catch (ClassNotFoundException | SQLException e) {
            throw new RuntimeException(e);
        }
    }
}
```

在代理方法中，从mapper中获取对应的sql语句和类的地址信息，负责执行SQL语句，并且封装结果集。

```java
package top.lehanbal.mybatis.utils;

import top.lehanbal.mybatis.cfg.Mapper;

import java.beans.PropertyDescriptor;
import java.lang.reflect.Method;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.ResultSetMetaData;
import java.util.ArrayList;
import java.util.List;

/**
 * 负责执行SQL语句，并且封装结果集
 */
public class Executor {
    public <E> List<E> selectList(Mapper mapper, Connection conn) {
        PreparedStatement pstm = null;
        ResultSet rs = null;
        try {
            //1.取出mapper中的数据
            String queryString = mapper.getQueryString();//select * from user
            String resultType = mapper.getResultType();//com.itheima.domain.User
            Class domainClass = Class.forName(resultType);
            //2.获取PreparedStatement对象
            pstm = conn.prepareStatement(queryString);
            //3.执行SQL语句，获取结果集
            rs = pstm.executeQuery();
            //4.封装结果集
            List<E> list = new ArrayList<E>();//定义返回值
            while(rs.next()) {
                //实例化要封装的实体类对象
                E obj = (E)domainClass.newInstance();
                //取出结果集的元信息：ResultSetMetaData
                ResultSetMetaData rsmd = rs.getMetaData();
                //取出总列数
                int columnCount = rsmd.getColumnCount();
                //遍历总列数
                for (int i = 1; i <= columnCount; i++) {
                    //获取每列的名称，列名的序号是从1开始的
                    String columnName = rsmd.getColumnName(i);
                    //根据得到列名，获取每列的值
                    Object columnValue = rs.getObject(columnName);
                    //给obj赋值：使用Java内省机制（借助PropertyDescriptor实现属性的封装）
                    PropertyDescriptor pd = new PropertyDescriptor(columnName,domainClass);//要求：实体类的属性和数据库表的列名保持一种
                    //获取它的写入方法
                    Method writeMethod = pd.getWriteMethod();
                    //把获取的列的值，给对象赋值
                    writeMethod.invoke(obj,columnValue);
                }
                //把赋好值的对象加入到集合中
                list.add(obj);
            }
            return list;
        } catch (Exception e) {
            throw new RuntimeException(e);
        } finally {
            release(pstm,rs);
        }
    }

    private void release(PreparedStatement pstm,ResultSet rs){
        if(rs != null){
            try {
                rs.close();
            }catch(Exception e){
                e.printStackTrace();
            }
        }

        if(pstm != null){
            try {
                pstm.close();
            }catch(Exception e){
                e.printStackTrace();
            }
        }
    }
}
```

如果需要使用注解的方式，那就写一个注解，XML解析文件当中另外标记了注解的存储方法。

```java
package top.lehanbal.mybatis.annotations;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Select {
    String value();
}
```

到此为止，我们的简单Mybatis就写完了。

附上一个运行图：

![结果1](https://gitee.com/lehanbal/blog-image/raw/master/img/%E7%BB%93%E6%9E%9C1.png)

（其实我自己也知道自己讲得八行，但是还是要记录，以后有本事了自己写个更全面的mybatis，主要是涉及到的设计模式没办法详细的说明）