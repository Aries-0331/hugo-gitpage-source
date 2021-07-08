---
title: "TLS协议"
date: 2021-07-06T22:24:45+08:00
draft: false
tags: [protocol, tls]
categories: [计算机网络, 加密]
---

## TLS 协议的设计目标

**构建一个安全传输层（Transport Layer Security）**，在基于连接的传输层（如 TCP）之上提供密码学安全，包括

1. 机密性，信息加密传输，无法被窃听
2. 完整性，信息一旦被篡改，通信双方可感知
3. 认证，通信双方防止身份被冒充

## TLS 的历史

- 1995: SSL 2.0，由Netscape（美国网景公司）提出，设计有缺陷，很快废弃。
- 1996: SSL 3.0，开始流行。2015年被证实不安全，已禁用。
- 1999: TLS 1.0，互联网标准化组织 ISOC 接替网景公司，发布了SSL的升级版 TLS 1.0 版。
- 2006: TLS 1.1，修复部分漏洞。
- 2008: TLS 1.2，增进安全性，2011 年不再兼容 SSL。
- 2018: TLS 1.3，支持0-rtt，大幅增进安全性，砍掉了aead之外的加密方式。

一般所说的 ssl，指的就是 TLS，目前大多指 1.2 版本。 

## 密码算法相关概念

密码学通信协议也是分层构造，大致可以分为四层：

1. 最底层，基础算法原语的实现，例如：aes、rsa、md5、sha256、ecdh......
2. 第二层，是搭配参数后，符合密码学标准的算法，包括块加密算法、流加密算法、签名算法、非对称加密算法、MAC算法等，例如：aes-128-cbc-pkcs7
3. 第三层，把多种标准算法组合而成的半成品组件，例如：对称加密传输组件（aes-128-cbc + hmac-sha256）、认证密钥协商算法（rsassa-OAEP + ecdh-secp256r1）、数字信封（rsaes-oaep + aes-cbc-128 + hmac-sha256）、密钥扩展算法（ PRF - sha256）......
4. 最上层，各种组件拼装而成的成品密码学协议/软件，例如：tls 协议、ssh协议、iMessage协议、bitcoin协议......

从上面分层的角度来看，TLS 大致由 3 个组件拼成：

- 对称加密传输组件
- 认证密钥协商组件
- 密钥扩展组件

这些组件可拆分为 5 类算法，在 TLS 中，这 5 类算法组合在一起，称为一个 CipherSuite：

- 认证算法
- 加密算法
- 消息认证码算法（MAC）
- 密钥交换算法
- 密钥衍生算法

## TLS 协议分层

TLS 主要用于加密数据传输，其主体是一个对称加密传输组件，而为了给这个组件生成通信双方共享的密钥，需要一个认证密钥协商组件，因此，TLS 协议主体有两个：

- record 协议，负责对称加密
- handshake 协议，负责认证密钥协商

以及三个辅助协议：

- changecipher spec 协议，用于通知对端从 handshake 切换到 record
- alert 协议，通知各种返回码
- application data 协议，把 http 等应用层的数据流传入 record 层做处理并传输

这种 **认证密钥协商 + 对称加密传输** 的结构，是大多数加密通信协议的通用结构。

### record 协议

record 协议负责应用数据的对称加密传输，占据 TLS 连接的大部分流量。

主要过程分 4 步：数据分块 —— 数据压缩 —— 加密和完整性保护 —— 添加消息头

[![6jIavQ.md.png](https://z3.ax1x.com/2021/03/26/6jIavQ.md.png)](https://imgtu.com/i/6jIavQ)

### handshake 协议

handshake 协议的主要作用在于，产生 SecurityParameters 并提供给 record 协议，TLS 1.3 相对于 TLS 1.2 做出的最大修改，体现在握手协议，TLS 1.2 握手的总体流程：

- 客户端和服务端协商 TLS 协议版本号和一个 Cipher Suite
- 认证对端身份（ https 中一般是客户端认证服务端的身份）
- 使用密钥协商算法生成共享的 master secret

握手流程如下：

```c
      Client                                               Server

      ClientHello                  -------->
                                                      ServerHello
                                                     Certificate*
                                               ServerKeyExchange*
                                              CertificateRequest*
                                   <--------      ServerHelloDone
      Certificate*
      ClientKeyExchange
      CertificateVerify*
      [ChangeCipherSpec]
      Finished                     -------->
                                               [ChangeCipherSpec]
                                   <--------             Finished
      Application Data             <------->     Application Data

      //*表示可选的消息，或根据上下文在某些情况下会发送的消息
```

握手过程的主要行为：

- 交换 Hello 消息，协商出算法、交换 random 值等；
- 交换必要的密码学参数，以便 client 和 server 协商出 premaster secret；
- 交换证书和密码学参数，以便 client 和 server 做身份认证；
- 根据 premaster secret 和交换的 random 值，生成共享的 master secret；
- 把 SecurityParameters 提供给 record 层；
- client 和 server 确认对端得出了相同的 SecurityParameters，且握手过程的数据没有被篡改；

**TLS 1.2 需要两次数据往返（ 2-RTT ）才能完成握手。**

## TLS 1.3 的主要变化

### 更快的速度

TLS 1.3 完整握手流程：

```c
          Client                                               Server

          ClientHello
          + key_share               -------->
                                                          ServerHello
                                                          + key_share
                                                {EncryptedExtensions}
                                                {CertificateRequest*}
                                                       {Certificate*}
                                                 {CertificateVerify*}
                                                           {Finished}
                                    <--------     [Application Data*]
          {Certificate*}
          {CertificateVerify*}
          {Finished}                -------->
                                    <--------      [NewSessionTicket]
          [Application Data]        <------->      [Application Data]
```

主要分 3 个阶段

- 密钥交换，建立共享密钥数据并选择密码参数（该阶段后所有数据都会被加密）
- Server 参数，建立其它握手参数（Client 是否被认证、应用层协议支持等）
- 认证：认证 Server（并选择性认证 Client），提供密钥确认和握手完整性校验

可以看出，TLS 1.3 只需一次往返（ 1-RTT ）即可完成握手，相比 TLS 1.2 握手时间减半，以访问移动端网站为例，访问效率可能会提高将近 100ms。

实现思路：砍掉 TLS1.2 的各种算法支持，只保留少量协商算法，并采取缓存方式，将密钥交换的协商并入第一个 RTT，从而将整个交互降到 1-RTT。

### 更强的安全性

TLS 1.2 为了考虑兼容性，保留了对 SSL 2.0/3.0 时期许多加密算法和组件的支持，使得 TLS 1.2 具有高度可配置特性，也意味着许多不熟悉 TLS1.2 的人配置出脆弱的站点。

TL3 1.3 在 1.2 的基础上，删除了不安全的加密算法，如：

- RSA 密钥传输 —— 不支持前向安全性
- CBC 模式密码 —— 被证实存在漏洞，易受攻击
- RC4 流密码 —— 在 https 中使用不安全
- SHA-1 哈希函数 —— 建议使用 SHA-2
- 各种自定义 DH 组件 —— 被证实存在漏洞，易受攻击







