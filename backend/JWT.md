# authservice-JWT

## why JWT

在过去的web项目中，通常使用的是Cookie-Session的认证方式。大致的流程如下：

1. 用户使用用户名个密码登录，发送信息到服务器。
2. 服务器拿到用户的信息，后会生成一份保存用户信息的session数据和与之对应的cookie id，然后保存session在在服务器，把对应的session id返回给用户并保存在浏览器。
3. 后续来自该用户的每次请求都会携带相应的cookie。
4. 服务端通过带来的cookie找到之前保存的与之对应的用户信息。

那么这种方式有什么弊端呢？

- 不支持跨域

- 通过数据库查询响应的用户信息耗时

- CSRF攻击(跨站伪造请求)

  链接：https://juejin.cn/post/7093035836689612836

## 什么是JWT

JSON Web Token (JWT) 是一种用于在各方之间安全传输信息的紧凑、自包含的方式。它广泛应用于互联网应用中的身份验证和信息交换。JWT 的设计使其适合通过 URL、POST 参数或 HTTP 标头等方式高效传输。

### JWT 的组成

JWT 通常由三个部分组成，每一部分用点号 (`.`) 分隔：
1. **Header（头部）**
2. **Payload（有效载荷）**
3. **Signature（签名）**

例如，一个完整的 JWT 看起来像这样：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

#### 1. Header（头部）
Header 通常由两部分组成：令牌类型（即 "JWT"），以及所使用的签名算法（如 HMAC SHA256 或 RSA）。

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

此 JSON 被 Base64 编码以形成 JWT 的第一部分。

#### 2. Payload（有效载荷）
Payload 部分包含声明（claims）。声明是关于实体（通常是用户）和其他数据的陈述。有三种类型的声明：
- **Registered claims**：预定义的声明集，包括 `iss`（发行者），`exp`（过期时间），`sub`（主题），`aud`（受众）等。
- **Public claims**：可以自由定义，不与标准冲突的声明。需要在 IANA JSON Web Token Registry 中注册，或确保它们是独一无二的 URI。
- **Private claims**：双方约定使用，不在注册列表中。

示例：

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
}
```

这个 JSON 也被 Base64 编码以形成 JWT 的第二部分。

#### 3. Signature（签名）
为了创建签名部分，需要将编码后的头部和有效载荷用`.`连接，并使用签名算法对其进行签名。签名算法通常会使用一个密钥（对于 HMAC SHA256）或一个 RSA 私钥。

对前面两个部分进行签名：

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

生成的签名用于验证消息在传输过程中未被篡改。

### JWT 的工作流程

1. **用户登录**：用户使用凭证（用户名和密码）进行登录。
2. **生成 JWT**：服务器验证凭证通过后，生成一个 JWT，并将其返回给客户端。
3. **存储 JWT**：客户端（通常是浏览器）将 JWT 存储在 Local Storage 或 Cookies 中。
4. **附带 JWT 请求**：每次客户端请求时将 JWT 附加在 HTTP 头部（通常是 `Authorization` 字段）。
5. **验证 JWT**：服务器接收到请求后，验证 JWT 是否有效（检查签名和过期时间等），如果有效则处理请求，否则返回错误响应。

### 总结

JWT 是一种轻量级、安全且跨平台的认证和信息交换机制，通过将验证信息打包在令牌内部，可以实现无状态的服务器认证，并且具有较好的扩展性和易用性。它广泛应用于现代 Web 开发、无状态的分布式系统以及微服务架构中。
