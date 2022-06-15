---
title: Redis发布订阅
date: 2020-11-18 18:43:53
tags:
- Redis
categories:
- Redis
---

Redis 发布订阅 (pub/sub) 是一种消息通信模式：发送者 (pub) 发送消息，订阅者 (sub) 接收消息。

Redis客户端可以订阅任意数量的频道。

订阅频道：

![image-20201118183513237](http://cdn.lehanbal.top/image-20201118183513237.png)

当有新消息通过 PUBLISH 命令发送给频道 channel1 时：

![image-20201118183529589](http://cdn.lehanbal.top/image-20201118183529589.png)

## 示例：

客户端一订阅 lehanbal 频道：

```bash
127.0.0.1:6379> SUBSCRIBE lehanbal
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "lehanbal"
3) (integer) 1
```

客户端二向 lehanbal 频段推送内容

```bash
127.0.0.1:6379> publish lehanbal hello,world
(integer) 1
```

客户端一收到内容：

```bash
1) "message"
2) "lehanbal"
3) "hello,world"
```

| 序号 | 命令以及描述                                                 |
| ---- | ------------------------------------------------------------ |
| 1    | PSUBSCRIBE pattern 订阅一个或多个符合给定模式的频道。        |
| 2    | PUBSUB subcommand [argument [argument ...\]] 查看订阅与发布系统状态。 |
| 3    | PUBLISH channel message将信息发送到指定的频道。              |
| 4    | PUNSUBSCRIBE [pattern [pattern ...\]] 退订所有给定模式的频道。 |
| 5    | SUBSCRIBE channel [channel ...\] 订阅给定的一个或多个频道的信息。 |
| 6    | UNSUBSCRIBE [channel [channel ...\]] 指退订给定的频道。      |

