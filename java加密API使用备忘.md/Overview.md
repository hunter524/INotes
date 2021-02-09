# Overview

加密主要分为三部分:

- 加密算法(DES,AES,RSA)
  
- 加密模式(CBC,ECB,CTR,OCF,CFB)

  [Block cipher mode of operation(分组密码工作模式) Wiki][https://zh.wikipedia.org/wiki/%E5%88%86%E7%BB%84%E5%AF%86%E7%A0%81%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F]

  分组加密算法的块加密模式.wiki 称之为分组密码的工作模式

- 填充模式()
  
  分组块加密算法中,当明文块的最后一个块大小小于 16B (128 bit 时需要指定的块的填充模式) [填充定义 wiki][https://zh.wikipedia.org/wiki/%E5%A1%AB%E5%85%85_(%E5%AF%86%E7%A0%81%E5%AD%A6)]

  [常用的填充模式定义][https://zh.wikipedia.org/wiki/%E5%85%AC%E9%92%A5%E5%AF%86%E7%A0%81%E5%AD%A6%E6%A0%87%E5%87%86]

  TODO://通常如 RSA 加密算法的配置:RSA/ECB/PKCS1Padding AES 的加密算法配置为 AES/CBC/PKCS5Padding

- 初始化向量(IV Initialization Vector)

[初始化向量 Wiki][https://zh.wikipedia.org/wiki/%E5%88%9D%E5%A7%8B%E5%90%91%E9%87%8F]

## Cipher

AES,RSA 等

## Key

### 对称加密算法

javax.crypto.KeyGenerator(AES,DES 等)

#### SecretKey

### 非对称加密算法

java.security.KeyPairGenerator(RSA,DSA,DH,EC 等)

#### KeyPair

- PrivateKey

RSAPrivateKey

- PublicKey

RSAPublicKey

#### 公开密钥证书

证书既可以存储公钥信息,也可以存储私钥信息

[公开密钥证书][https://en.wikipedia.org/wiki/Public_key_certificate] 如 X.509

[X.509 证书格式][https://zh.wikipedia.org/wiki/X.509] 证书文件的扩展名又具有多种形式如: pem,cer,crt,der,p7b,p7c,p12,pfx 等.

公钥证书的验证又可以称为证书链.

- X509EncodedKeySpec
  
- KeyFactory
  