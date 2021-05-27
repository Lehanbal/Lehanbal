---
title: Redis五大数据类型
date: 2020-11-11 22:18:14
tags:
- Redis
categories:
- Redis
---

## String

### SET GET

set是通过设置一个key值来对应它的value值，这是最基本的操作指令了。

get这是通过key来获取它所对应的value值。

命令如下：

```bash
# set ‘set key value’
127.0.0.1:6379> SET name lehanbal
OK
# get ‘get key’
127.0.0.1:6379> GET name
"lehanbal"
```

### MSET MGET

MSET是批量设置key，MGET是批量获取值。

```bash
127.0.0.1:6379> MGET name1 name2 name3 name4
1) "le"
2) "leh"
3) "leha"
4) "lehanbal"
127.0.0.1:6379> MSET name5 1 name6 2 name 7
OK
127.0.0.1:6379> MGET name1 name2 name3 name4 name5 name6 name7
1) "le"
2) "leh"
3) "leha"
4) "lehanbal"
5) "1"
6) "2"
7) (nil)
```

注意这里面是保证原子性的，当其中某一项失败的时候，都是一起失败，具备原子性。

### APPEND

通过key来找到对应的value内容，并将新的value值拼接在后面，会返回拼接好的字符串长度。

```bash
# APPEND key value
127.0.0.1:6379> APPEND name ,Yes!
(integer) 13
127.0.0.1:6379> get name
"lehanbal,Yes!"
```

### STRLEN

通过key获取到value的长度。

```bash
# STRLEN key
127.0.0.1:6379> STRLEN name
(integer) 13
```

### INCR DECR

INCR命令能够对一个只有数值的字符串进行+1操作，若没有事先声明key的值，那么这个key会被赋值0然后进行增加操作。

DECR命令则反过来。

```bash
127.0.0.1:6379> set num 2000
OK
127.0.0.1:6379> get num
"2000"
# INCR/DECR key
127.0.0.1:6379> incr num
(integer) 2001
127.0.0.1:6379> decr num
(integer) 2000
```

### INCRBY DECRBY

INCR和DECR的能够按步长进行增加或者减少的命令

```bash
# INCRBY/DECRBY key increment/decrement
127.0.0.1:6379> INCRBY num 5
(integer) 2005
127.0.0.1:6379> INCRBY num 5
(integer) 2010
127.0.0.1:6379> DECRBY num 4
(integer) 2006
127.0.0.1:6379> DECRBY num 4
(integer) 2002
```

### GETRANGE

获取一个字符串的区间内容

```bash
# GETRANGE key start end
127.0.0.1:6379> set name lehanbal,yes!
OK
127.0.0.1:6379> GETRANGE name 0 5
"lehanb"
```

### SETRANGE

从字符串的偏移位置开始，替换后面的字符串内容

```bash
# SETRANGE key offest value
127.0.0.1:6379> SETRANGE name 7 ,NO!
(integer) 13
127.0.0.1:6379> GET name
"lehanba,NO!s!"
```

### SETEX(set with expire)

设置一个key的同时自带过期时间。

```bash
# SETEX key times value
127.0.0.1:6379> SETEX name 10 lehanbal
OK
127.0.0.1:6379> ttl name
(integer) 6
127.0.0.1:6379> ttl name
(integer) -2
127.0.0.1:6379> get name
(nil)
```

### SETNX(set if not exist)

如果没有当前key就设置当前key，返回1，否则失败，返回0.

```bash
# SETNX key value
127.0.0.1:6379> set name1 le
OK
127.0.0.1:6379> set name2 leh
OK
127.0.0.1:6379> set name3 leha
OK
127.0.0.1:6379> keys *
1) "name3"
2) "name2"
3) "name1"
127.0.0.1:6379> SETNX name1 lehanbal
(integer) 0
127.0.0.1:6379> SETNX name4 lehanbal
(integer) 1
127.0.0.1:6379> get name1
"le"
127.0.0.1:6379> get name4
"lehanbal"
```

### MSETNX

SETNX的批量赋值版本。

```bash
# MSETNX key1 value1 key2 value2 ...
127.0.0.1:6379> MSETNX n1 1 n2 2 n3 3 n4 4
(integer) 1
127.0.0.1:6379> keys *
 1) "name5"
 2) "name1"
 3) "n2"
 4) "n4"
 5) "n1"
 6) "name6"
 7) "name4"
 8) "name"
 9) "name2"
10) "name3"
11) "n3"
127.0.0.1:6379> MSETNX name 1 asdkjna jnaskjdn
(integer) 0
```

原子操作，其中一个失败，全部都会失败。

更高阶一点的用法

```bash
# 通过对:来区分用户的信息使用mget就能很方便的获取数据而不需要去解析json格式的文件
127.0.0.1:6379> mset user:1:name lehanbal user:1:age 24
OK
127.0.0.1:6379> mget user:1:name user:1:age
1) "lehanbal"
2) "24"
```

### GETSET

先GET后SET，返回值是GET的值。

```bash
127.0.0.1:6379> SET name lehanba
OK
127.0.0.1:6379> GETSET name lehanbal
"lehanba"
127.0.0.1:6379> GET name
"lehanbal"
```

## List

通过双向链表实现的存储。如果key不存在的话，就会创造新的链表，如果key存在，就会新增内容。

### LPUSH/RPUSH和LRANGE

向list集合中加入元素，LPUSH是从左边增加元素，而RPUSH则是向右边增加元素。

LRANGE则是按照相应的下标进行集合的遍历

```bash
# LPUSH/RPUSH key value
127.0.0.1:6379> LPUSH list one
(integer) 1
127.0.0.1:6379> LPUSH list two
(integer) 2
127.0.0.1:6379> LPUSH list three
(integer) 3
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "two"
3) "one"
127.0.0.1:6379> RPUSH list zero
(integer) 4
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "two"
3) "one"
4) "zero"
```

### LPOP/RPOP

POP为弹出元素操作，返回值为弹出的元素，L和R分别表示从左边或从右边弹出。（操作接上）

```bash
# LPOP/RPOP key
127.0.0.1:6379> LPOP list
"three"
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
3) "zero"
127.0.0.1:6379> RPOP list
"zero"
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
```

### LINDEX

获取列表指定下标的元素，下标从0开始，和数组一样。（操作接上）

```bash
# LINDEX key index
127.0.0.1:6379> LINDEX list 0
"two"
127.0.0.1:6379> LINDEX list 1
"one"
127.0.0.1:6379> LINDEX list 2
(nil)
```

### LLEN

获取当前列表的长度。（操作接上）

```bash
# LLEN key
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
127.0.0.1:6379> LLEN list
(integer) 2
```

### LREM

从左边开始删除列表中指定数量的指定值元素。

```bash
# LREM key start end
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
3) "one"
4) "one"
5) "two"
6) "one"
127.0.0.1:6379> LREM list 2 one
(integer) 2
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
3) "two"
4) "one"
```

### LTRIM

分割当前列表，保留下从开始下标到结束下标的元素。

```bash
# LTRIM key start end
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
3) "two"
4) "one"
127.0.0.1:6379> LTRIM list 0 2
OK
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
3) "two"
```

### RPOPLPUSH

组合操作，先弹出元素，将弹出的元素压入对应的列表中。

```bash
# RPOPLPUSH source destination
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
3) "two"
127.0.0.1:6379> LRANGE list1 0 -1
(empty list or set)
127.0.0.1:6379> RPOPLPUSH list list1
"two"
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
127.0.0.1:6379> LRANGE list1 0 -1
1) "two"
```

### LSET

替换掉指定位置的元素。

```bash
# LRANGE key index value
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
127.0.0.1:6379> LSET list 0 three
OK
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "one"
```

### LINSERT

将元素插入到列表中，可以根据before或after将元素插入在指定元素的前面或后面。

```bash
# LINSERT key AFTER/BEFORE start end
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "one"
127.0.0.1:6379> LINSERT list BEFORE three four
(integer) 3
127.0.0.1:6379> LRANGE list 0 -1
1) "four"
2) "three"
3) "one"
127.0.0.1:6379> LINSERT list AFTER three two
(integer) 4
127.0.0.1:6379> LRANGE list 0 -1
1) "four"
2) "three"
3) "two"
4) "one"
```

## Set

Set也是一个集合，和java的set一样，不能够存储重复元素。

### SADD

向Set集合中添加一个元素。

```bash
# SADD key members...
127.0.0.1:6379> SADD set lehanbal
(integer) 1
127.0.0.1:6379> SADD set hello
(integer) 1
127.0.0.1:6379> SADD set man
(integer) 1
```

### SMEMBERS

查看当前Set集合的所有元素。

```bash
# SMEMBERS key
127.0.0.1:6379> SMEMBERS set
1) "man"
2) "lehanbal"
3) "hello"
```

### SISMEMBER

查看当前元素是否在改集合中。存在则返回1，不存在则返回0。

```bash
# SISMENBER key value
127.0.0.1:6379> SMEMBERS set
1) "man"
2) "lehanbal"
3) "hello"
127.0.0.1:6379> SISMEMBER set lehanbal
(integer) 1
127.0.0.1:6379> SISMEMBER set lehanbal??
(integer) 0
```

### SCARD

获取Set集合中的元素个数。

```bash
# SCARD key
127.0.0.1:6379> SMEMBERS set
1) "man"
2) "lehanbal"
3) "hello"
127.0.0.1:6379> SCARD set
(integer) 3
```

### SREM

移除集合中的指定元素。

```bash
# SREM key value...
127.0.0.1:6379> SMEMBERS set
1) "man"
2) "lehanbal"
3) "hello"
127.0.0.1:6379> SREM set man
(integer) 1
127.0.0.1:6379> SMEMBERS set
1) "lehanbal"
2) "hello"
```

### SRANDMEMBER

随机抽选出指定个数的元素。

```bash
# SRANDMEMBER key [num]
127.0.0.1:6379> SMEMBERS set
1) "hambaga"
2) "lehanbal"
3) "le"
4) "hello"
127.0.0.1:6379> SRANDMEMBER set
"lehanbal"
127.0.0.1:6379> SRANDMEMBER set
"hambaga"
127.0.0.1:6379> SRANDMEMBER set 2
1) "hambaga"
2) "lehanbal"
127.0.0.1:6379> SRANDMEMBER set 2
1) "lehanbal"
2) "hello"
```

### SPOP

随机弹出指定个数元素。

```bash
# SPOP key [num]
127.0.0.1:6379> SMEMBERS set
1) "hambaga"
2) "lehanbal"
3) "le"
4) "hello"
127.0.0.1:6379> SPOP set
"lehanbal"
127.0.0.1:6379> SPOP set
"hambaga"
127.0.0.1:6379> SMEMBERS set
1) "le"
2) "hello"
```

### SMOVE

将一个指定的元素移动到另外一个Set集合中。

```bash
# SMOVE source destination member
127.0.0.1:6379> SMEMBERS set
1) "le"
2) "hello"
127.0.0.1:6379> SMOVE set set1 le
(integer) 1
127.0.0.1:6379> SMEMBERS set
1) "hello"
127.0.0.1:6379> SMEMBERS set1
1) "le"
```

### SDIFF SINTER SUNION

SDIFF：求若干个Set集合的差集，以第一个Set集合为基准。

SINTER：求若干个Set集合的交集。

SUNION：求若干个Set集合的并集。

```bash
# SDIFF key1 key2...
# SINTER key1 key2...
# SUNION key1 key2...
127.0.0.1:6379> SADD set1 a b c d
(integer) 4
127.0.0.1:6379> SADD set2 c d e f g
(integer) 5
127.0.0.1:6379> SDIFF set1 set2
1) "b"
2) "a"
127.0.0.1:6379> SINTER set1 set2
1) "c"
2) "d"
127.0.0.1:6379> SUNION set1 set2
1) "b"
2) "a"
3) "e"
4) "g"
5) "c"
6) "f"
7) "d"
```

## Hash

redis中的map集合，和java中的map集合一样。也就是说相当于key-key-value形式存储。

### HSET HGET HMSET HMGET HGETALL

HSET：设置一个hash值。

HGET：获取一个hash值。

HMSET：批量设置hash值。

HMGET：批量获取hash值。

HGETALL：获取全部的hash值。

```bash
# HSET key field value
# HGET key field
# HMSET key field value...
# HMGET key field ...
# HGETALL key
127.0.0.1:6379> HSET key1 name lehanbal
(integer) 1
127.0.0.1:6379> HSET key1 age 24
(integer) 1
127.0.0.1:6379> HSET key1 porfession student
(integer) 1
127.0.0.1:6379> HGET key1 name
"lehanbal"
127.0.0.1:6379> HGET key1 age
"24"
127.0.0.1:6379> HGET key1 porfession
"student"
127.0.0.1:6379> HGETALL key1
1) "name"
2) "lehanbal"
3) "age"
4) "24"
5) "porfession"
6) "student"
```

HMSET、HMGET、HMSETNX就不演示了，与String一致。

### HDEL

删除hash元素里面的一个字段。字段被删除了，对应的value也会被删除。

```bash
# HDEL key field
127.0.0.1:6379> HDEL key1 porfession
(integer) 1
127.0.0.1:6379> HGETALL key1
1) "name"
2) "lehanbal"
3) "age"
4) "24"
```

### HLEN

获取当前hash元素的键值长度。

```bash
# HLEN key
127.0.0.1:6379> HGETALL key1
1) "name"
2) "lehanbal"
3) "age"
4) "24"
127.0.0.1:6379> HLEN key1
(integer) 2
```

### HEXISTS

判断hash中指定的字段是否存在。存在返回1，不存在返回0。

```bash
# HEXISTS key field
127.0.0.1:6379> HGETALL key1
1) "name"
2) "lehanbal"
3) "age"
4) "24"
127.0.0.1:6379> HEXISTS key1 name
(integer) 1
127.0.0.1:6379> HEXISTS key1 nnnn
(integer) 0
```

### HKEYS HVALS

获取hash的全部字段。

```bash
# HKEYS key
# HVALS key
127.0.0.1:6379> HGETALL key1
1) "name"
2) "lehanbal"
3) "age"
4) "24"
127.0.0.1:6379> HKEYS key1
1) "name"
2) "age"
127.0.0.1:6379> HVALS key1
1) "lehanbal"
2) "24"
```

## Zset

一个有序集合，在set的基础上增加了一个值。作为权重值来进行排序。

### ZADD

Zset添加元素操作。

```bash
# ZADD key score member
127.0.0.1:6379> ZADD key 500 lehanbal
(integer) 1
127.0.0.1:6379> ZADD key 1000 LE
(integer) 1
```

### ZRANGE

Zset的遍历操作。

```bash
# ZRANGE key start end
127.0.0.1:6379> ZRANGE key 0 -1
1) "lehanbal"
2) "LE"
```

### ZRANGEBYSCORE

将元素按照权重从小到大排序。可以通过min与max排序显示这个区间的元素。

```bash
# ZRANGEBYSCORE key min max [withscores]
127.0.0.1:6379> ZRANGEBYSCORE key -inf +inf withscores
1) "lehanbal"
2) "500"
3) "LE"
4) "1000"
```

### ZREVRANGEBYSCORE

将元素按权重从大到小排序。同上。

```bash
# ZREVRANGE
127.0.0.1:6379> ZREVRANGE key 0 -1 withscores
1) "LE"
2) "1000"
3) "lehanbal"
4) "500"
# ZREVRANGEBYSCORE key max min [score]
127.0.0.1:6379> ZREVRANGEBYSCORE key +inf -inf withscores
1) "LE"
2) "1000"
3) "lehanbal"
4) "500"
```

### ZREM

移除Zset的元素。

```bash
# ZREM key value
127.0.0.1:6379> ZRANGE key 0 -1
1) "hambaga"
2) "lehanbal"
3) "LE"
127.0.0.1:6379> ZREM key hambaga
(integer) 1
127.0.0.1:6379> ZRANGE key 0 -1
1) "lehanbal"
2) "LE"
```

### ZCOUNT

统计区间内的数量。

```bash
# ZCOUNT 
127.0.0.1:6379> ZRANGE key 0 -1
1) "lehanbal"
2) "LE"
127.0.0.1:6379> ZCOUNT key 0 999
(integer) 1
127.0.0.1:6379> ZCOUNT key 0 1000
(integer) 2
```

