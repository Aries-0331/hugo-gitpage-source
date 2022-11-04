---
title: "关于QUIC Crypto"
date: 2022-11-04T17:13:59+08:00
draft: false
categories: [crypto, quic]
---

# 简要介绍

QUIC（Quick UDP Internet Connections）是一种默认加密的互联网传输协议，旨在提供安全、快速的 HTTP 传输，以替代 TCP 和 TLS。Google QUIC 的默认加密协议是自家的 QUIC Crypto。

QUIC 与 TCP 的根本不同在于默认提供安全传输的设计原则。最初的 QUIC 结合了 TCP 的三次握手以及 TLS 1.3 的握手，并将 TLS的记录层数据帧格式替换为 QUIC 自己的，同时保留了 TLS 的握手消息。这种做法既保证了连接总是经过认证和加密的，同时使得初始连接建立速度更快。TCP 和 TLS1.3 完成握手需要两轮交互，而 QUIC 握手仅需一轮，如果客户端缓存了服务端的配置信息，那么甚至无需握手。

<div align=center><img src="https://blog.cloudflare.com/content/images/2018/07/http-request-over-tcp-tls@2x.png" width="600"><img src="https://blog.cloudflare.com/content/images/2018/07/http-request-over-quic@2x.png" width="600">

# 协商成本对比

## QUIC

QUIC 的服务端有一个使用私钥签名的`server config`参数列表，包含常规参数、 DH 公钥、有效期等信息，由于这些数据都是静态的，所以一次签名后可以长期使用，不需要每次建立连接时都签名。

通道密钥协商使用`Diffie-Hellman`，DH 算法所需要的服务端参数在`server config`中，客户端参数通过第一个握手消息传递至服务端。为了实现 0-RTT 握手而设计的静态 `server config`，同时也带来了加密连接的前向安全问题 —— 只要服务端还保存有 `server config`，那么一旦密钥泄露，之前使用该密钥加密的数据都是可破解的。

因此 QUIC 提供了两级加密：来自客户端的初始数据使用`server config`中的 DH 值加密，该值可持久化存储几天，服务端收到客户端的连接后，立即回复一个临时的 DH 值，并重新加密连接。

虽然乍一看这种做法没有 TLS1.3 所实现的前向安全更完善，但实际上， TLS 在大规模部署时为了减少网络交互，通常会启用 `SessionTickets`，而`SessionTickets`同样会持久化存储，并被用于破解加密连接，`server config`和`SessionTickets`可以说是半斤八两，相较而言 QUIC 提供了临时密钥用于实际通信，反而稍强于 TLS 的做法。

每个连接使用一个临时密钥，与所有连接共用一个临时密钥，在安全性上的差异可以忽略不计，而牺牲这点安全性差异所带来的复杂度打折是很可观的，所以不妨在较短的时间跨度内，将服务器的 DH 密钥复用至所有连接上。

## TLS1.3

TLS1.3 的握手主要做三件事情：密钥交换、获取服务器参数、认证；

- 密钥交换：该阶段共享生成密钥所需的材料，例如所属计算方式或命名组（ECDHE or DHE），选择密码参数，例如对称加密选项；

- 服务器参数：该阶段确认其它握手参数，例如是否需要基于数字证书的客户端认证；

- 认证：该阶段对服务端进行认证，对客户端选择性认证，并对密钥进行校验以及握手信息的完整性校验；

TLS 的握手分两种：非前向安全握手和前向安全握手，其与 QUIC 的握手效率对比大概为：

- 对于非前向安全握手，客户端做一次公钥加密约34us，服务端做一次私钥解密约1100us；

- 对于前向安全握手，客户端做一次公钥加密、一次DH运算、一次伪随机数运算，大约耗时230us，服务端做一次私钥解密、一次DH运算、一次伪随机数运算，大约耗时1300us；
- QUIC 客户端做一次 DH 运算、两次随机数运算、一次公钥加密，约耗时 184us，服务端做两次随机数运算，约耗时100us；

可以看出 QUIC 与 TLS 在协商阶段的成本差异主要是非对称加解密造成的，当然这里没有包含校验证书链等操作，因为两个协议都有，并且默认两种协议所选的密码算法与选项相同或处于统一性能等级，比如摘要都用SHA-256、DH 曲线都用 Curve25519，如果给 QUIC 选 ECDH P-256 而 TLS 选 Curve25519，那就是另一种结果了。

另外，TLS 有会话恢复机制，而 QUIC 则需要由 C/S 两端各自维护 DH 结果的缓存，虽然没有在协议层面支持这一特性，但牺牲这一便利而带来了大约5倍的效率提升，还是不错的。

# QUIC handshake

QUIC 的协商过程可以简单总结出三个要点：

- `server config`是一个包含服务端临时密钥的参数集，大概每几天更新一次；
- 初次建立连接时的 client hello 其实就是个空的消息，目的就是从 server 获取包含最新 `server config` 的REJ(ect)消息，获取到后会重新发起正式的 client hello；
- server hello 是包含临时密钥的加密消息；

<div align=center><img src="https://lh4.googleusercontent.com/DLKsceJfawl4pkbWeaTx69tgW97gG4TQ3BNWBPF3YgtqbkKhHmzgSyL7iajoAibBW9qRx7DcJiCwF2dGZngMFRI3pylhbttoRKZ6LLvQEJuvRepn8gOeya9FrfYd_PvVGRswL6uQCexAimZZdBuUwI5ttPH_mo-i2bVKQFxu9jMosswjo2e4FQvn0YHq" width=500>

- 首次连接

为了实现 0-RTT handshake，客户端需要获取`server config`，因此首次连接的成本是 1-RTT。

当客户端第一次建立连接时，在握手成功前，客户端会先发送一个不完整的 `hello message`以获取服务期配置及其真实性证明，而服务端为了避免向任意未证实的端发送大量真实性证明，因此需要验证客户端身份，这一过程可能会有多次交互，不过这些是一次性的。

客户端的`hello message`中包含一些键值对，例如服务端域名、源地址token、可接受的认证类型、通用证书集合、缓存的证书等（部分是可选的），主要用来表明身份，在收到客户端的`hello message`后，服务端将返回拒绝消息或`server hello`。hello 表示握手成功，拒绝消息则会附带供客户端握手使用的一些信息。

例如，若客户端没有携带源地址token，而服务端不想给未认证的IP源发送`server config`，那么服务端会在拒绝消息中携带源地址token，以便客户端的下次握手校验能通过。

`server config`通过键值对描述服务端的一系列参数，包含 server config ID、密钥交换算法、认证加密算法、公钥列表、有效期、版本等。客户端收到`server config`后，通过证书链和签名来进行认证，然后就可以发送不会失败的完整握手消息。完整的握手消息除了上述不完整的消息内容外，还包含客户端使用的 server config ID、认证加密算法、密钥家换算法、客户端随机数、服务端随机数、公钥等。

发送完整的`hello message`后，双方就共同拥有了一个非前向安全密钥（也可称为初始密钥），此时客户端就可以给服务端开始发送应用数据了。

- 非首次连接

客户端在首次连接后，会把`server config`存下来，之后再发起连接时，可直接发送业务数据，以此来实现 0-RTT。

# TLS1.3状态机



<div align=center><img src="https://cryptologie.net/upload/sm1.png" width=800>

从握手的状态图可以很快做出判断，TLS 比 QUIC 复杂好多。TLS 是一个相对可靠、完善的协议，但同时由于版本迭代较多，大量历史遗留问题导致其架构、接口、用例非常复杂，如果不考虑历史包袱等兼容性问题，应该可以设计出更优雅的“TLS”。

# 密钥派生

QUIC 的密钥材料通过 HMAC-SHA256 密钥派生函数（HKDF）生成，HKDF 采用先提取再扩展的设计方式，先将输入的字符转换为固定长度的伪随机数，然后将其扩展成若干个伪随机密钥。在 QUIC 的实现中，密钥协商阶段生成的 DH 密钥便是 HKDF 的输入。

- HKDF-Extract

使用原始的密钥材料，派生出一个符合密码学强度的伪随机密钥。HKDF-Extract 的输入包含客户端、服务端的nonce以及密钥协商输出的密钥（pre-master key），HKDF-Extract 的输出是一个伪随机密钥，作为主密钥，长度 32 bytes（前提是使用 SHA-256 作为摘要算法）。

- HKDF-Expand

使用 HKDF-Expand 提取出来的伪随机密钥，扩展出指定长度的密钥（同时保证随机性）。以 HKDF-Extract 生成的 pre-master key 和下列信息作为输入，以生成 forward-secret key：

- QUIC key expansion
- 连接的 GUID
- client hello message
- server config
- DER 编码证书

总结来看就是，client hello message 获取到 server config，并生成 dh key，以该 key 作为 HKDF-Extract 的输入，生成 pre-master key，再通过 HKDF-Expand 生成 forward-secret key，即真正用来加密业务数据的密钥。

# CETV Tag

与 TLS 明文传输客户端证书不同，QUIC 的 client hello 中包含一个 CETV 标志，用于标记客户端证书、通道ID以及其他 hello 消息中的非公开数据。CETV 通过 AEAD 方式加密保护，所用密钥的生成方式与 forward-secret key 的生成方式类似。由于 client hello 只有一次，所以 AEAD 加密时使用的 nonce 可以是 0。

# 证书压缩

TLS 的证书链是按照源文件大小直接传递的，在握手阶段占大头。 QUIC 为了简化交互，将证书压缩后传输，并不传递CA根证书，而是在交互中包含本地缓存证书的摘要值，以确认双方是否共享同一证书链。

# 参考

[QUIC Crypto](https://docs.google.com/document/d/1g5nIXAIkN_Y-7XJW5K45IblHd_L2f5LTaDUDwvZ5L6g/edit#heading=h.sx7u0bm1ucc0)

[QUIC协议详解之Initial包的处理](https://www.upyun.com/tech/article/571/QUIC%E5%8D%8F%E8%AE%AE%E8%AF%A6%E8%A7%A3%E4%B9%8BInitial%E5%8C%85%E7%9A%84%E5%A4%84%E7%90%86.html)

[The Transport Layer Security (TLS) Protocol Version 1.3 draft-ietf-tls-tls13-28](https://datatracker.ietf.org/doc/html/draft-ietf-tls-tls13-28#appendix-A)

[QUIC: A UDP-Based Multiplexed and Secure Transport draft-ietf-quic-transport-27](https://datatracker.ietf.org/doc/html/draft-ietf-quic-transport-27/#section-7)

[QUIC Crypto and simple state machines](https://cryptologie.net/article/446/quic-crypto-and-simple-state-machines/)

[The Illustrated TLS 1.3 Connection](https://tls13.xargs.org/)

<!--more-->
