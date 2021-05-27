---
title: Jedis工具类
date: 2020-11-18 13:38:01
tags:
- Redis
categories:
- Redis
---

## Jedis连接池工具类

每次连接Jedis很烦，而且反复连接和断开很影响性能，Jedis本身就由JedisPool和JedisConfig类供我们搭建自己的JedisPool工具类来让我们在开发的过程中更为方便。

首先创建我们的jedis.properties配置文件，配置文件放在resources文件夹下。

```properties
#地址
host=127.0.0.1
#端口
port=6379
#总连接的最大数量
maxTotal=50
#空闲连接的最大数量，至少会保持10个活连接
maxIdle=10
```

JedisPool工具类：

```java
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

public class JedisPoolUtil {
    private static JedisPool jedisPool = null;

    static {
        InputStream is = JedisPoolUtil.class.getClassLoader().getResourceAsStream("jedis.properties");
        Properties pro = new Properties();
        try {
            pro.load(is);
        } catch (IOException e) {
            e.printStackTrace();
        }

        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxIdle(Integer.parseInt(pro.getProperty("maxIdle")));
        config.setMaxTotal(Integer.parseInt(pro.getProperty("maxTotal")));

        jedisPool = new JedisPool(config, pro.getProperty("host"), Integer.parseInt(pro.getProperty("port")));
    }

    public static Jedis getJedis(){
        return jedisPool.getResource();
    }
}
```

## Jedis操作工具类

每次操作Jedis都需要去释放连接，如果觉得觉得很烦的话，可以试着使用Jedis操作工具类，这里封装了String方法和Hash方法，需要什么功能就可以封装什么功能。

```java
import redis.clients.jedis.Jedis;

import java.util.Set;

public class JedisUtil {
    /**
     * 获取hash表中的全部key
     * @param name 键
     * @return 全部key
     */
    public static Set<String> getHashAllKey(String name){
        Jedis jedis = null;
        try{
            jedis = JedisPoolUtil.getJedis();
            return jedis.hkeys(name);
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            JedisPoolUtil.returnJedis(jedis);
        }
        return null;
    }

    /**
     * 获取hash表中对应key的值
     * @param hashName hash名
     * @param key 键
     * @return 值
     */
    public static String getHashKV(String hashName, String key){
        Jedis jedis = null;
        try{
            jedis = JedisPoolUtil.getJedis();
            return jedis.hget(hashName, key);
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            JedisPoolUtil.returnJedis(jedis);
        }
        return null;
    }

    /**
     * 删除hash表中的键值对
     * @param hashName hash名
     * @param key 键
     * @return 删除数量
     */
    public static Long delHashKV(String hashName, String key){
        Jedis jedis = null;
        try{
            jedis = JedisPoolUtil.getJedis();
            return jedis.hdel(hashName, key);
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            JedisPoolUtil.returnJedis(jedis);
        }
        return null;
    }

    /**
     * 存放hash键值对
     * @param hashName hash名
     * @param key 键
     * @param value 值
     * @return 存放的数量
     */
    public static Long setHashKV(String hashName, String key, String value){
        Jedis jedis = null;
        try{
            jedis = JedisPoolUtil.getJedis();
            return jedis.hset(hashName, key, value);
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            JedisPoolUtil.returnJedis(jedis);
        }
        return null;
    }

    /**
     * 存储键值对，永久
     * @param k 键
     * @param v 值
     * @return
     */
    public static String setKV(String k, String v){
        Jedis jedis = null;
        try{
            jedis = JedisPoolUtil.getJedis();
            return jedis.set(k, v);
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            JedisPoolUtil.returnJedis(jedis);
        }
        return null;
    }

    /**
     * 存放键值对，有时限
     * @param k 键
     * @param s 存活时间
     * @param v 值
     * @return
     */
    public static String setKV(String k, int s, String v){
        Jedis jedis = null;
        try{
            jedis = JedisPoolUtil.getJedis();
            return jedis.setex(k, s, v);
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            JedisPoolUtil.returnJedis(jedis);
        }
        return null;
    }

    /**
     * 获取键值对的值
     * @param key 键
     * @return 值
     */
    public static String getKV(String key){
        Jedis jedis = null;
        try{
            jedis = JedisPoolUtil.getJedis();
            return jedis.get(key);
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            JedisPoolUtil.returnJedis(jedis);
        }
        return null;
    }

    /**
     * 删除对应的键值对
     * @param key 键
     * @return 删除的数量
     */
    public static Long delKV(String key){
        Jedis jedis = null;
        try{
            jedis = JedisPoolUtil.getJedis();
            return jedis.del(key);
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            JedisPoolUtil.returnJedis(jedis);
        }
        return null;
    }
}
```