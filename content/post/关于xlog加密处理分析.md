---
title: "关于xlog加密处理分析"
date: 2021-05-28T22:25:57+08:00
draft: false
categories: [Security]
---

xlog 使用微型加密算法（TEA，Tiny Encryption Algorithm）对日志数据进行加密，使用 ECDH 密钥交换算法进行对称密钥的协商，对称密钥以数组形式存储在栈区，声明为 `LogCrypt` 类的私有字段。

会话密钥长度与椭圆曲线参数相关，xlog 中使用 16 Bytes 密钥。

### 相关接口

- uECC_make_key ( client_pubkey, client_pri )
  - 生成客户端公私钥对
- uECC_shared_secret ( svr_pubkey, client_pri, secret )
  - 根据服务端公钥与客户端私钥，运算出对称密钥 secret
- __TeaEncrypt ( log, secret )
  - 使用对称密钥加密日志

### 协商流程：

1. `client` 调用 `uECC_make_key` 生成 pubKeyA 与 priKeyA；
2. `server` 调用 `uECC_make_key` 生成 pubKeyB 与 priKeyB；
3. `client` 获取`server` 公钥 `pubKeyB` ，调用 `uECC_shared_secret` 生成对称密钥 `secret`；（此处采用 ECDH 密钥交换算法）
4. `server` 获取`client` 公钥 `pubKeyA` ，调用 `uECC_shared_secret` 生成对称密钥 `secret`；

### ECDH 密钥交换算法原理概述

ECDH 基于 ECC 算法，ECC 是建立在基于椭圆曲线的离散对数问题上的密码体制，给定椭圆曲线上的一个点 P，一个整数 k，求解 Q = kP 很容易，但给定一个点 P、一个点Q，求解 k 很难。ECDH 便建立在该数学难题上。

假设密钥协商的双方 Alice 和 Bob 共享公共参数 p、g 以及 ECDH 算法，以下场景可模拟交换流程（mod 表示取余运算）：

1. Alice 生成私钥 a，通过公式 `g^a mod p = A` 生成公钥 A，并将 p、g、A 发送给 Bob；
2. Bob 获得 p、g、A 后，生成自己的私钥 b，通过`g^b mod p = B` 生成公钥B，通过公式 `A^B mod p = K` 生成会话密钥（对称密钥）K，然后将公钥 B 发送给 Alice；
3. Alice 获得 Bob 的公钥 B，通过 `B^a mod p = K` 生成会话密钥 K，至此，Alice 和 Bob 在未传递私钥 a、b 的情况下，完成了会话密钥协商。

ECDH 不传输对称密钥，而是传输 ECC 椭圆曲线算法生成的公钥，然后通过收到的公钥结合自己的私钥，各自计算出共享的对称密钥，椭圆曲线保证了双方计算出的密钥一定是相同的，且第三方在不知道通信双方私钥的情况下，无法通过从信道中截获的信息反推出对称密钥。

### 代入数据：

1. 假定 p = 23，g = 5；
2. Alice 生成私钥 a = 6，计算 A = g^a mod p = 5^6 mod 23 = 8；
3. Bob 生成私钥 b = 15，计算 B = g^b mod p = 5^15 mod 23 = 19，计算 K = A^B mod p = 8^18 mod 23 = 2；
4. Alice 计算 K' = B^a mod p = 19^6 mod 23 = 2，K' = K。

