---
title: HTTP与HTTPS的区别
date: 2020-10-29 14:24:36
tags:
- HTTP
- HTTPS
categories:
- 计算机网络
---

1. http协议：超文本传输协议，信息是明文传输，并且不验证通信方的身份，无法验证报文的完整性/
2. https协议：超文本传输安全协议（HTTP Secure），具有安全性的SSL（Secure Socket Layer 安全套接层）或者TLS（Transport Layer Security 安全层传输协议）的组合使用。在HTTP的基础上具备对报文的加密处理，通信方身份认证和内容完整性保护，这就是HTTPS



HTTPS并不是应用层的一种新协议。只是HTTP通信接口部分使用的是SSL（Secure Socket Layer）和TLS（Transport Layer Security）协议代替。

一般来说，HTTP是直接和TCP进行通信。当我们使用了SSL在之后，就会变成先和SSL通信，再由SSL与TCP进行通信。所以说，HTTPS就是HTTP套了一层SSL的皮。

HTTP协议使用的端口是80，HTTPS使用的端口是443。