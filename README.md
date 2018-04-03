# JWT LEARN NOTE

#### JWT 概述

token 只是一种思路, 一种解决用户授权问题的思考方式, 基于这种思路, 针对不同的场景可以有很多种的实现. 而在众多的实现中, JWT(JSON Web Token) 的实现最为流行.JWT 这个标准提供了一系列如何创建具体 token 的方法, 这些缘故方法和规范可以让我们创建 token 的过程变得更加合理和效率.比如, 传统的做法中, 服务器会保存生成的token, 当客户端发送来token时, 与服务器的进行比对, 但是 jwt 的不需要在服务器保存任何 token, 而是使用一套加密/解密算法 和 一个密钥 来对用户发来的token进行解密, 解密成功后就可以得到这个用户的信息.这样的做法同时也增加了多服务器时的扩展性, 在传统的 token 验证中, 一旦用户发来 token, 那么必须要先找到存储这个 token 的服务器是哪台服务器, 然后由那一台服务器进行验证用户身份. 而 jwt 的存在, 只要每一台服务器都知道解密密钥, 那么每一台服务器都可以拥有验证用户身份的能力.这样一来, 服务器就不再保存任何用户授权的信息了, 也就解决了 session 曾出现的问题.



#### jwt 原理

JWT是Auth0提出的通过对JSON进行加密签名来实现授权验证的方案，编码之后的JWT看起来是这样的一串字符

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpUVyJ9.eyJpc3MiOiJlcmljaXZhbiIsImlhdCI6MTUyMjI1MDQ3NywiZXhwIjoxNTIyMjUwNDc3LCJhdWQiOiJlcmljbGluZy5jbiIsInN1YiI6InVpZCJ9.9ec0edb9298258dd38c2086a40949289
```

token由三部分组成,由符号 . 来分隔开



##### 1.Header 头部

```php
$header = [
 	'alg' => 'HS256',
 	'typ' => 'JTW',
];
```



头部中包括

- alg：加密算法 HMAC SHA256
- typ:声明类型

php中对头部进行处理 ` base64_encode(json_encode($header))`

加密后为eyJhbGciOiJIUzI1NiIsInR5cCI6IkpUVyJ9，第一部分组装完成



##### 2.Payload 荷载

载荷就是存放有效信息的地方。这些有效信息包含三个部分：

- 标准中注册声明
- 公共的声名
- 私有的声明

`公共的声明 ：` 公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息.但不建议添加敏感信息，因为该部分在客户端可解密。

`私有的声明 ：` 私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为base64是对称解密的，意味着该部分信息可以归类为明文信息。

以下为一个payload生成的例子



```php

$payload = [
    'iss' => 'ericivan',
    'iat' => 1522250477,
    'exp' => 1522250477,
    'aud' => 'ericling.cn',
    'sub' => 'uid',
];
$payload=base64_encode(json_encode($payload));
```

生成token:

> eyJpc3MiOiJlcmljaXZhbiIsImlhdCI6MTUyMjI1MDQ3NywiZXhwIjoxNTIyMjUwNDc3LCJhdWQiOiJlcmljbGluZy5jbiIsInN1YiI6InVpZCJ9



##### 3.签名（signature）

签名是一个重要的部分，在服务器内执行，利用前面两个生成的header还有payload，加上服务器端的secret，通过加密算法算出来最后一个部分，加密模拟如下

`$signature=hash_hmac('sha256',$preHeader,'mykey');`

 最后的签名生成后，用.返回连接起来返回到客户端，客户端请求的时候再header头加上Authorization token ,服务器通过计算就可以认证token是否正确



下图是jwt中client与server的交互过程

[jtw auth](https://github.com/Ericivan/-jwt-note/blob/master/jwt.png)

