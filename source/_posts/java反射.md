---
title: java反射
date: 2020-10-29 14:37:14
tags:
- JAVA
- 反射
categories:
- JAVA
---

## 定义

JAVA反射机制是在运行状态中，对任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称之为java的反射机制。

 JVM读取相应类的字节码文件叫做反射。

## 用途

开发过程中经常会遇到某个类的某个成员变量、方法是私有的或者只对系统开放，这个时候使用JAVA反射机制就能够获取所需的私有成员或是方法，同样也能够使用反射来进行类的创建，降低耦合程度。另外，使用反射肯定会比直接调用慢，运行很多很多次程序的情况，反射大概比直接调用慢个50来倍，但是我们其实没有这么多需要运行百万级的反射程序。

如果我们需要大量的进行反射调用，那么进行缓存处理，不要反复去使用反射。

## 反射相关的类

|     类名      |                       用途                       |
| :-----------: | :----------------------------------------------: |
|    Class类    | 代表类的实体，在运行的Java应用程序中表示类和接口 |
|    Field类    |                 代表类的成员变量                 |
|   Method类    |                 代表类的成员方法                 |
| Constructor类 |                 代表类的构造方法                 |

### **Class类(重点)**

**获取类相关的方法**

|            方法            |                          用途                          |
| :------------------------: | :----------------------------------------------------: |
| asSubclass(Class<U> class) |         把传递的类的对象转换成代表其子类的对象         |
|            Cast            |           把对象转换成代表类或者是接口的对象           |
|      getClassLoader()      |                      获得类加载器                      |
|        getClasses()        | 返回一个数组，数组中包含该类中所有公共类和接口类的对象 |
|    getDeclaredClasses()    |   返回一个数组，数组中包含该类中所有类和接口类的对象   |
| forName(String className)  |                  根据类名返回类的对象                  |
|         getName()          |                  获得类的完整路径名字                  |
|       newInstance()        |                      创建类的实例                      |
|        getPackage()        |                       获取类的包                       |
|      getSimpleName()       |                      获取类的名字                      |
|      getSuperclass()       |               获取当前类继承的父类的名字               |
|      getInterfaces()       |               获取当前类实现的类或是接口               |

**获取类中属性的方法**

|             方法              |          用途          |
| :---------------------------: | :--------------------: |
|     getField(String name)     | 获得某个共有的属性对象 |
|          getField()           | 获得所有共有的属性对象 |
| getDeclaredField(String name) |    获得某个属性对象    |
|      getDeclaredFields()      |    获得所有属性对象    |

**获取类中注解相关的方法**

|                      方法                       |                   用途                   |
| :---------------------------------------------: | :--------------------------------------: |
|     getAnnotation(Class<A> annotationClass)     | 返回该类中参与参数类型匹配的公有注解对象 |
|                getAnnotations()                 |        返回该类所有的共有注解对象        |
| getDeclaredAnnotation(Class<A> annotationClass) |  返回该类中与参数类型匹配的所有注解对象  |
|            getDeclaredAnnotations()             |         返回该类中所有的注解对象         |

**获取类中构造器相关的方法**

|                        方法                        |                  用途                  |
| :------------------------------------------------: | :------------------------------------: |
|     getConstructor(Class..<?> parameterTypes)      | 获得该类中与参数类型匹配的公有构造方法 |
|                  getConstructor()                  |       获得该类的所有共有构造方法       |
| getDeclaredConstructor(Class...<?> parameterTypes) |   获得该类中与参数类型匹配的构造方法   |
|             getDeclaredConstructors()              |         获得该类中所有构造方法         |

**获取类中方法相关的方法**

|                            方法                            |          用途          |
| :--------------------------------------------------------: | :--------------------: |
|     getMethod(String name, Class...<?> parameterTypes)     | 获得该类某个公有的方法 |
|                        getMethod()                         | 获得该类所有共有的方法 |
| getDeclaredMethod(String name, Class...<?> parameterTypes) |    获得该类某个方法    |
|                    getDeclaredMethods()                    |    获得该类所有方法    |

 **类中其他重要的方法**

|                             方法                             |          用途          |
| :----------------------------------------------------------: | :--------------------: |
|                        isAnnotation()                        |   判断是否是注解类型   |
| isAnnotationPresent(Class<? extends Annotation> annotationClass) | 判断是否是注定类型注解 |
|                      isAnonymousClass()                      |    判断是否是匿名类    |
|                          isArray()                           |     判断是否是数组     |
|                           isEnum()                           |   判断是否是枚举类型   |
|                    isInstance(Object obj)                    |     判断是否是obj      |
|                        isInterface()                         |     判断是否是接口     |
|                        isLocalClass()                        |    判断是否是局部类    |
|                       isMemberClass()                        |    判断是否是内部类    |

### Field类

Field代表类的成员变量(成员变量也成为类的属性)

|            方法             |                        用途                        |
| :-------------------------: | :------------------------------------------------: |
|     equals(Object obj)      |              属性与obj相等则返回true               |
|       get(Object obj)       |               获得obj中对应的属性值                |
|       set(Object obj)       |               设置obj中对应的属性值                |
| setAccessible(Boolean flag) | 是否关闭java语言访问检查（关闭可提高反射运行速度） |

### Method类

只有一个执行方法，但是很重要

|                方法                 |                   用途                   |
| :---------------------------------: | :--------------------------------------: |
| invoke(Object obj, Object ... args) | 传递object对象及参数调用该对象对应的方法 |

### Constructor类

类的构造方法

|              方法               |            用途            |
| :-----------------------------: | :------------------------: |
| newInstance(Object... initargs) | 根据传递的参数创造类的对象 |

## Demo

要玩反射，那么就需要有个反射的对象。

```java
package Lehanbal.study.Reflect;

public class Person {
    private String name;
    private String age;

    public Person() {
    }

    public Person(String name, String age) {
        this.name = name;
        this.age = age;
    }

    public void eat() {
        System.out.println("起飞");
    }

    private void toilet(int genderIdx) {
        switch (genderIdx) {
            case 0 -> System.out.println("进男厕所");
            case 1 -> System.out.println("进女厕所");
            default -> System.out.println("宁就是女权终结者？");
        }
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAge() {
        return age;
    }

    public void setAge(String age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age='" + age + '\'' +
                '}';
    }
}
```

封装一下反射的方法：

```java
package Lehanbal.study.Reflect;

import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class Demo04 {
    private static final String personPath = "Lehanbal.study.Reflect.Person";

    //创建对象
    public static void reflectNewInstance(){
        try{
            Class<?> classPerson = Class.forName(personPath);
            Person person = (Person) classPerson.getConstructor().newInstance();
            person.setName("懒汉");
            person.setAge("24");
            System.out.println(person);
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    //反射私有构造函数
    public static void reflctPrivateConstructor(){
        try {
            Person person = (Person)Class.forName(personPath).getDeclaredConstructor(String.class, String.class).newInstance("懒汉", "23");
            System.out.println(person);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    //反射私有属性
    public static void reflectPrivateField(){
        try{
            Class<?> personClass = Class.forName(personPath);
            Object personObj = personClass.getConstructor().newInstance();
            Field field = personClass.getDeclaredField("TEST");
            field.setAccessible(true);
            System.out.println((String) field.get(personObj));
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    //反射私有方法
    public static void reflectPrivateMethod(){
        try{
            Class<?> personClass = Class.forName(personPath);
            Method method = personClass.getDeclaredMethod("toilet", int.class);
            method.setAccessible(true);
            Object o = personClass.getConstructor().newInstance();
            method.invoke(o, 0);
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        reflectNewInstance();
        reflctPrivateConstructor();
        reflectPrivateField();
        reflectPrivateMethod();
    }
}
```

运行结果如下：

Person{name='懒汉', age='24'}

Person{name='懒汉', age='23'}

Test

进男厕所