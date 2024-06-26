---
title: "HTTPS与HTTP"
date: 2020-10-22T22:28:02+08:00
draft: false
categories: [HTTPS]
---

> 概念上理清 HTTPS 与 HTTP 的区别。

在[互联网协议套件](https://zh.wikipedia.org/wiki/TCP/IP%E5%8D%8F%E8%AE%AE%E6%97%8F)中，将网络协议分为四层，从下至上依次为**链路层-网络层-传输层-应用层**，HTTP 属于应用层协议。中文名为超文本传输协议，其设计初衷是为了传输 HTML 页面。

HTTP 主要规定了客户端和服务器之间的通信格式，并不涉及数据包的传输，传输依赖于 TCP/IP 协议。

简单理解就是，将需要传递的数据按照 HTTP 的要求格式化（_挖坑，数据格式_），然后剩下的交给协议栈中的 TCP/IP 层。

因此基于 HTTP 协议的数据传输，其发送阶段的数据处理流程为，应用层格式化，TCP 封装，IP 封装，过程中不涉及数据的加解密，数据为明文形式传输。

而 HTTPS 就是在这一流程基础上增加了安全性功能。通常可以这么理解 HTTPS， HTTPS = HTTP over TLS。（_由于安全问题，普遍已经抛弃 SSL，因此这里就直接按 TLS 写了_）

TLS 表示传输层安全，从 SSL 演进而来，属于应用层与 TCP 层之间的一个协议，其目的是为互联网通信提供安全及数据完整性保障（挖坑，数据完整性），说白了就是确保数据加密传输，以及确保如果数据被篡改了，接收方能察觉到。

那么 TLS 做了什么？

由于要实现数据加解密，因此 TLS 要实现数据的对称加密，不用非对称加密主要是考虑到性能问题。

通信双方进行数据的对称加密传输前，需要验证对端的身份，即身份认证，这一点 TLS 通过数字证书来实现（_挖坑，实现原理_）。

因此基于 HTTPS 协议的数据传输，其发送阶段的数据处理流程为，应用层格式化，TLS 封装，TCP 封装，IP 封装，相较于 HTTP，流程中增加了 TLS 的处理，包括身份认证、数据加解密等。
