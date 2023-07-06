---
title: Token、Cookie和Session的联系
tags:
---


## token

token 是一种由服务器端生成的字符串，可以用来标识用户的身份和权限。token 通常由用户的 ID，当前时间，过期时间，签名等信息组成，并通过一些加密算法压缩成定长的十六进制字符串。token 由服务器端在用户登录成功后返回给客户端，客户端将 token 存储在本地，如 cookie 或 localStorage 中。客户端每次访问服务器时，需要手动将 token 放在 HTTP 的 Authorization 字段中发送给服务器。服务器收到 token 后，解密并验证其合法性，如果合法，则根据其中的信息进行相应的处理。token 的优点是无状态，支持跨域访问，缺点是需要额外的验证机制。

生成和验证 token 的方法有以下几个步骤：

1. **在服务器端生成 token**：token 是一种随机生成的字符串，它可以用一些加密算法，如 SHA-256，来根据用户的 ID，当前时间，过期时间等信息生成。生成的 token 应该是不可预测的，不可重复的，有时效性的。
2. **在服务器端保存 token**：生成的 token 应该保存在服务器端的 session 中，与用户的 ID 关联起来。这样，每次用户发送请求时，服务器端就可以从 session 中取出 token，并与请求中的 token 进行比对。
3. **在客户端存储 token**：服务器端生成并返回 token 后，客户端应该将 token 存储在本地，如 cookie 或 localStorage 中。这样，每次客户端发送请求时，就可以携带 token 作为请求参数或请求头。
4. **在服务器端验证 token**：服务器端接收到客户端发送的请求后，应该先从 session 中取出用户的 ID 和 token，并与请求中的 ID 和 token 进行比对。如果一致，则说明该请求是合法的，否则就是 CSRF 攻击或者 token 过期。

## session、token和cookie

cookie、session 和 token 是三种常用的实现用户身份认证和状态管理的技术，它们之间有以下几点区别：

- cookie 是一种存储在客户端的键值对数据，可以用来保存用户的一些信息，如登录状态，偏好设置等。cookie 由服务器端通过 HTTP 的 Set-Cookie 字段设置，客户端每次访问同一个域名时，会自动携带该域名的 cookie 发送给服务器。cookie 的优点是简单易用，缺点是容量有限，不安全，不支持跨域访问。
- session 是一种存储在服务器端的数据结构，通常是一个映射（键值对），可以用来保存用户的一些敏感信息，如账号密码，购物车等。session 由服务器端创建，并通过一个唯一的标识符（session ID）与客户端关联。session ID 通常通过 cookie 传递给客户端，客户端每次访问同一个域名时，会将 session ID 作为 cookie 的一部分发送给服务器。服务器根据 session ID 找到对应的 session，并根据其中的信息进行相应的处理。session 的优点是安全，可以存储复杂的数据结构，缺点是占用服务器资源，有时效性。

## cookie 和 session 的配合使用

- 当用户第一次访问服务器时，服务器会创建一个 session 对象，并生成一个唯一的 session ID，然后把这个 session ID 作为 cookie 的值发送给客户端。
- 当用户再次访问服务器时，客户端会自动将这个 cookie（包含 session ID）发送给服务器，服务器根据 session ID 找到对应的 session 对象，并根据其中的信息进行相应的处理。
- 这样，服务器就可以通过 session 来保存用户的状态信息，而客户端只需要保存一个 session ID 的 cookie 即可。
