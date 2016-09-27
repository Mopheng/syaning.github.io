---
layout: post
title:  Node核心模块之crypto
date:   2016-09-27 18:00:00 +0800
---

### Hash

哈希函数（散列函数）主要用于生成消息摘要(Message Digest)，即将任意大小的数据映射到一个固定大小的数据。最常见的如MD5，SHA1等。

```
---------  hash function  --------------
| input |---------------->| hash value |
---------                 --------------
```

在Node中，通过`crypto.getHashes()`可以查看所支持的哈希算法：

```js
crypto.getHashes()
// => [ 'DSA', 'DSA-SHA', 'DSA-SHA1', ... ]
```

下面是一个MD5的例子：

```js
var hash = crypto.createHash('md5')
  .update('input')
  .digest('hex')

console.log(hash)
// => a43c1b0aa53a0c908810c06ab1ff3967
```

### HMAC

在了解HMAC之前，需要先了解MAC(Message Authentication Code)，即消息认证码。

发送方使用MAC算法将消息和密钥进行计算，得到MAC值，然后将消息和MAC值发送出去。接收方拿到数据后，通过相同的MAC算法把消息和密钥进行计算，和收到的MAC值进行比较。如果一致，则说明消息在发送过程中没有被篡改。

```
-----------
|   key   |----
-----------   |   MAC algorithm   -------
              |------------------>| Mac |
-----------   |                   -------
| message |----                      |
-----------                          |
     |_______________________________|
(Sender)             |
---------------------|-------------------------------
(Receiver)           |
     ________________|_______________________________
     ↓                                              ↓
-----------                                      -------
| message |----                                  | Mac |
-----------   |   MAC algorithm   -------        -------
              |------------------>| Mac |___________|
-----------   |                   -------   equal?
|   key   |----
-----------
```

HMAC即Hash-based Message Authentication Code，即使用一个Hash函数作为MAC算法。例如：

```js
var hmac = crypto.createHmac('sha256','secret')
  .update('message')
  .digest('hex')

console.log(hmac)
// => 8b5f48702995c1598c573db1e21866a9b825d4a794d169d7060a03605796360b
```

[cookie-signature](https://github.com/tj/node-cookie-signature)中对cookie签名用的就是HMAC实现。

### Cipher & Decipher

Cipher用于加密，Decipher用于解密。加密分为对称加密和非对称加密。

对称加密指的是加密和解密使用相同的密钥。例如A向B发消息，A使用密钥进行加密发送给B，B收到密文后使用相同的密钥进行解密从而得到消息明文。由于加密和解密使用的是相同的密钥，因此加入密钥泄露，那么别人截获了消息同样可以进行解密。

非对称加密指的是加密和解密使用不同的密钥。通常会有一对公钥和私钥，经过公钥加密的消息只有使用私钥才能解密，经过私钥加密的消息只有经过公钥才能解密。例如A向B发消息，A使用B的公钥加密消息发送出去，B收到密文后使用自己的私钥进行解密。由于B的私钥只有B拥有，因此如果别人截获了密文，也是无法解析的。

下面是一个使用aes256算法（对称加密）的例子：

```js
var cipher = crypto.createCipher('aes256', 'password')
var encrypted = cipher.update('input', 'utf8', 'hex')
encrypted += cipher.final('hex')

console.log(encrypted)
// => fcfb38e53f2400797beb819795f6a459

var decipher = crypto.createDecipher('aes256', 'password')
var decrypted = decipher.update(encrypted, 'hex', 'utf8')
decrypted += decipher.final('utf8')

console.log(decrypted)
// => input
```

### Sign & Verify

todo