在[密码学](https://en.wikipedia.org/wiki/Cryptography "密码学") 中，**伽罗瓦/计数器模式**（**GCM**是对称[密钥](https://en.wikipedia.org/wiki/Symmetric-key_algorithm "Symmetric-key algorithm") 密码[分组密码的一种](https://en.wikipedia.org/wiki/Block_cipher "Block cipher")[操作模式](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation "分组密码操作模式")，因其性能而被广泛采用。使用廉价的硬件资源即可实现最先进的高速通信通道的 GCM 吞吐率。

[GCM 算法提供数据真实性（完整性）和机密性，属于关联数据认证加密 (AEAD)](https://en.wikipedia.org/wiki/Authenticated_encryption "认证加密")方法类别。这意味着它需要密钥 K、一些明文 P 和一些关联数据 AD 作为输入；然后，它使用密钥加密明文以生成密文 C，并根据密文和相关数据（保持未加密状态）计算身份验证标签 T。知道 K 的接收者在收到 AD、C 和 T 后，可以解密密文以恢复明文 P，并可以检查标签 T 以确保密文和相关数据都没有被篡改。

GCM使用块大小为128位的分组密码（通常为[AES-128](https://en.wikipedia.org/wiki/AES-128 "AES-128")）以[计数器模式](https://en.wikipedia.org/wiki/Counter_mode "计数器模式") 操作进行加密，并使用[伽罗瓦域](https://en.wikipedia.org/wiki/Galois_field "伽罗瓦域") GF（2 128）中的算术来计算认证标签；由此得名。
### 表现
GCM 需要对每个加密和验证数据块（128 位）进行一次分组密码操作和[伽罗瓦域中](https://en.wikipedia.org/wiki/Galois_field "伽罗瓦域") 的一次 128 位乘法。分组密码操作很容易流水线化或并行化；乘法运算很容易流水线化，并且可以通过一些适度的努力来并行化（通过并行化实际操作，通过根据原始 NIST 提交的内容调整[Horner 的方法，或两者​​兼而有之）。](https://en.wikipedia.org/wiki/Horner%27s_method "霍纳法")


### 使用
GCM 模式用于IEEE 802.1AE (MACsec) 以太网安全、WPA3-Enterprise Wifi 安全协议、IEEE 802.11ad（也称为WiGig）、ANSI ( INCITS ) 光纤通道安全协议 (FC-SP)、IEEE P1619 .1 磁带存储、IETF IPsec标准、 SSH、 TLS 1.2 和 TLS 1.3。 AES-GCM 包含在NSA Suite B 密码学及其 2018 年商业国家安全算法 (CNSA) 套件中的最新替代品中。 GCM 模式用于SoftEther VPN服务器和客户端， 以及自 2.4 版本以来的 OpenVPN 。

###  伽罗瓦域
GCM 将著名的计数器加密[模式](https://en.wikipedia.org/wiki/Block_cipher_modes_of_operation#Counter_(CTR) "分组密码操作模式")与新的伽罗瓦认证模式相结合。其主要特点是易于并行计算用于身份验证的[伽罗瓦域](https://en.wikipedia.org/wiki/Galois_field "伽罗瓦域")乘法。此功能允许比使用链接模式的加密算法（例如[CBC）](https://en.wikipedia.org/wiki/Block_cipher_modes_of_operation#Cipher_Block_Chaining_(CBC) "分组密码操作模式")更高的吞吐量。使用的GF(2 128 ) 字段由多项式定义

![{\displaystyle x^{128}+x^{7}+x^{2}+x+1}](https://wikimedia.org/api/rest_v1/media/math/render/svg/574bb9e8dd4b5d77ec6924e28f7aaed4cae00074)

### 工作原理
下图直观地解释了**GCM 块模式**（伽罗瓦/计数器模式）的工作原理：

![](https://cryptobook.nakov.com/~gitbook/image?url=https%3A%2F%2F795243796-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-LhlOQMrG9bRiqWpegM0%252Fuploads%252Fgit-blob-9193da273d6167779b4913e3033b7a40aeadeb7c%252Fgcm-galois_counter_mode.png%3Falt%3Dmedia&width=300&dpr=4&quality=100&sign=8e75e8f6953d30594113e01df4a74b796a6d7847a3c3439852b60758ac326c6b)

**GCM**模式使用一个**计数器**，每个块都会增加一个计数器，并在每个处理完的块后计算一个消息**认证标签**（MAC 代码）。最终的认证标签是从最后一个块计算出来的。与所有计数器模式一样，GCM 用作**流密码**，因此在开始时为每个加密流使用**不同的 IV**至关重要。