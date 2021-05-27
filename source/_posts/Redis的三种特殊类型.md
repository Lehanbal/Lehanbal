---
title: Redis的三种特殊类型
date: 2020-11-13 09:38:59
tags:
- Redis
categories:
- Redis
---

## GEO

Redis GEO 主要用于存储地理位置信息，并对存储的信息进行操作，该功能在 Redis 3.2 版本新增。

Zset命令同样适用。

Redis GEO 操作方法有六个，分别是：

- geoadd：添加地理位置的坐标。
- geopos：获取地理位置的坐标。
- geodist：计算两个位置之间的距离。
- georadius：根据用户给定的经纬度坐标来获取指定范围内的地理位置集合。
- georadiusbymember：根据储存在位置集合里面的某个地点获取指定范围内的地理位置集合。
- geohash：返回一个或多个位置对象的 geohash 值。

### geoadd

添加地理位置的坐标。

```bash
# geoadd key longitude latitude member...
127.0.0.1:6379> geoadd china:city 116.408 39.904 beijing
(integer) 1
127.0.0.1:6379> geoadd china:city 104.071 30.67 chengdu 121.445 31.213 shanghai
(integer) 2
127.0.0.1:6379> geoadd china:city 113.265 23.108 guangzhou 114.109 22.544 shenzhen
(integer) 2
```

### geopos

获取地理位置的坐标，显示经度纬度。

```bash
# geopos key member...
127.0.0.1:6379> zrange china:city 0 -1
1) "chengdu"
2) "shenzhen"
3) "guangzhou"
4) "shanghai"
5) "beijing"
127.0.0.1:6379> geopos china:city chengdu shenzhen guangzhou
1) 1) "104.07099992036819458"
   2) "30.67000055930392222"
2) 1) "114.10900086164474487"
   2) "22.54399882788700182"
3) 1) "113.26500087976455688"
   2) "23.10799963305151294"
```

### geodist

计算两个位置之间的距离。unit的单位可选取为m、km、mi、ft。

```bash
# geodist key member1 member2 unit
127.0.0.1:6379> geopos china:city chengdu beijing
1) 1) "104.07099992036819458"
   2) "30.67000055930392222"
2) 1) "116.40800267457962036"
   2) "39.90399988166036138"
127.0.0.1:6379> geodist china:city chengdu beijing km
"1516.9099"
```

### georadius

georadius 以给定的经纬度为中心， 返回键包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素。

一般用于实现功能：查找周围的人。

```bash
# georadius key longitude latitude radius unit [withcoord] [withdist] [count(num)]
127.0.0.1:6379> georadius china:city 100 30 500 km
1) "chengdu"
127.0.0.1:6379> georadius china:city 100 30 500 km withcoord withdist count 2
1) 1) "chengdu"
   2) "397.8245"
   3) 1) "104.07099992036819458"
      2) "30.67000055930392222"
```

### georadiusbymember

georadiusbymember 和 GEORADIUS 命令一样， 都可以找出位于指定范围内的元素， 但是 georadiusbymember 的中心点是由给定的位置元素决定的， 而不是使用经度和纬度来决定中心点。

```bash
# georadiusbymember key member radius unit [withcoord] [withdist] [count(num)]
127.0.0.1:6379> georadiusbymember china:city beijing 1500 km
1) "shanghai"
2) "beijing"
```

### geohash

获取地址的hash值。

```bash
# geohash
127.0.0.1:6379> geohash china:city shanghai beijing
1) "wtw3ed1sct0"
2) "wx4g0bm9xh0"
```

## HyperLogLog

 HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

1. PFADD key element [element ...\]：添加指定元素到 HyperLogLog 中。
2. PFCOUNT key [key ...\]：返回给定 HyperLogLog 的基数估算值。
3. PFMERGE destkey sourcekey [sourcekey ...\]：将多个 HyperLogLog 合并为一个 HyperLogLog。

### PFADD 

添加指定元素到 HyperLogLog 中。

```bash
# PFADD key element [element ...\]
127.0.0.1:6379> PFADD name lehanbal hambaga ainstan lehanbal
(integer) 1
127.0.0.1:6379> PFCOUNT name
(integer) 3
```

### PFCOUNT 

返回给定 HyperLogLog 的基数估算值。

同上。

### PFMERGE 

将多个 HyperLogLog 合并为一个 HyperLogLog。

```bash
127.0.0.1:6379> PFADD name lehanbal hambaga ainstan lehanbal
(integer) 1
127.0.0.1:6379> PFCOUNT name
(integer) 3
127.0.0.1:6379> PFADD othername lehanbal mother father
(integer) 1
127.0.0.1:6379> PFCOUNT othername
(integer) 3
127.0.0.1:6379> PFMERGE name othername
OK
127.0.0.1:6379> PFCOUNT name
(integer) 5
127.0.0.1:6379> PFCOUNT othername
(integer) 3
```

## Bitmap

位图数据结构。可用来做打卡统计。

具有3个指令：

1. setbit：设置位图中的某一位。
2. getbit：获取位图中的某一位。
3. bitcount：统计位图中被设置的位数。

### setbit

设置位图中的某一位。

```bash
# setbit key offset value
127.0.0.1:6379> setbit week 0 1
(integer) 0
127.0.0.1:6379> setbit week 1 1
(integer) 0
127.0.0.1:6379> setbit week 2 0
(integer) 0
127.0.0.1:6379> setbit week 3 0
(integer) 0
127.0.0.1:6379> setbit week 4 0
(integer) 0
127.0.0.1:6379> setbit week 5 1
(integer) 0
127.0.0.1:6379> setbit week 6 1
(integer) 0
```

### getbit

获取位图中的某一位。

```bash
# getbit key offset
127.0.0.1:6379> getbit week 1
(integer) 1
127.0.0.1:6379> getbit week 4
(integer) 0
```

### bitcount

统计位图中被设置的位数。

```bash
# bitcount key [start] [end]
127.0.0.1:6379> bitcount week
(integer) 4
```

