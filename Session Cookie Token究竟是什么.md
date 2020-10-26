### ** Cookie Session Token究竟是什么**

由于http是无状态的的，就是说每次请求都毫无关联，互不相识。



### cookie是什么

cookie本身是由服务器产生的，生成之后发送给浏览器，并保存在浏览器。
cookie就是浏览器存储本地目录的一小段文本。
cookie是以key-value形式存储的。
cookie大小有限制，为了保证cookie不占用太多磁盘空间，每个cookie大小一般不超过4KB。
cookie默认在会话结束后直接销毁，此种cookie称之为会话cookie。
cookie可以设置过期时间，此种cookie称之为持久cookie。



### Cookie 的实现机制

[Cookie](https://harttle.land/assets/img/blog/cookie.png)是由客户端保存的小型文本文件，其内容为一系列的键值对。 **Cookie是由HTTP服务器设置的，保存在浏览器中**， 在用户访问其他页面时，会在HTTP请求中附上该服务器之前设置的Cookie。 Cookie的实现标准定义在[RFC2109: HTTP State Management Mechanism](https://www.ietf.org/rfc/rfc2109.txt)中。那么Cookie是怎样工作的呢？下面给出整个Cookie的传递流程：

![请求流程](https://image-static.segmentfault.com/372/034/372034436-5c359860bfdca_articlex)

1. 首先，客户端会发送一个http请求到服务器端。
2. 服务器端接受客户端请求后，建立一个session，并发送一个http响应到客户端，这个响应头，其中就包含Set-Cookie头部。该头部包含了sessionId。Set-Cookie格式如下，具体请看[Cookie详解](http://bubkoo.com/2014/04/21/http-cookies-explained/)
   `Set-Cookie: value[; expires=date][; domain=domain][; path=path][; secure]`
3. 在客户端发起的第二次请求，假如服务器给了set-Cookie，浏览器会自动在请求头中添加cookie
4. 服务器接收请求，分解cookie，验证信息，核对成功后返回response给客户端

总之，服务器通过`Set-Cookie`响应头字段来指示浏览器保存Cookie， 浏览器通过`Cookie`请求头字段来告诉服务器之前的状态。 Cookie中包含若干个键值对，每个键值对可以设置过期时间。

**Cookie使用限制**：Cookie 必须在 HTML 文件的内容输出之前设置；不同的浏览器 (Netscape Navigator、Internet Explorer) 对 Cookie 的处理不一致，使用时一定要考虑；客户端用户如果设置禁止 Cookie，则 Cookie 不能建立。 并且在客户端，一个浏览器能创建的 Cookie 数量最多为 300 个，并且每个不能超过 4KB，每个 Web 站点能设置的 Cookie 总数不能超过 20 个。



### session是什么

session是由服务器产生的，存储在服务端等。

session的存储形式多种多样，可以是文件、数据库、缓存等，这需要靠程序如何设计。

session也是以key-value形式存储的。

session是没有大小限制的，这比cookie灵活很多，不过将过多的东西放在其中也并不是明智的做法。

session也有过期时间的概念，默认为30分钟，可以通过tomcat、web.xml等方式进行配置。

session可以主动通过invalidate()方法进行销毁。

session通过session_id识别，如果请求持有正确的session_id，则服务器认为此请求处于session_id代表的会话中。

### Session 的实现机制

Session 是存储在服务器端的，避免了在客户端Cookie中存储敏感数据。 Session 可以存储在HTTP服务器的内存中，也可以存在内存数据库（如redis）中， 对于重量级的应用甚至可以存储在数据库中。

我们以存储在redis中的Session为例，还是考察如何验证用户登录状态的问题。

1. 用户提交包含用户名和密码的表单，发送HTTP请求。

2. 服务器验证用户发来的用户名密码。

3. 如果正确则把当前用户名（通常是用户对象）存储到redis中，并生成它在redis中的ID。

   这个ID称为Session ID，通过Session ID可以从Redis中取出对应的用户对象， 敏感数据（比如`authed=true`）都存储在这个用户对象中。

4. 设置Cookie为`sessionId=xxxxxx|checksum`并发送HTTP响应， 仍然为每一项Cookie都设置签名。

5. 用户收到HTTP响应后，便看不到任何敏感数据了。在此后的请求中发送该Cookie给服务器。

6. 服务器收到此后的HTTP请求后，发现Cookie中有SessionID，进行放篡改验证。

7. 如果通过了验证，根据该ID从Redis中取出对应的用户对象， 查看该对象的状态并继续执行业务逻辑。



### 注意

- cookie只是实现session的其中一种方案。虽然是最常用的，但并不是唯一的方法。禁用cookie后还有其他方法存储，比如放在url中
- 现在大多都是Session + Cookie，但是只用session不用cookie，或是只用cookie，不用session在理论上都可以保持会话状态。可是实际中因为多种原因，一般不会单独使用
- 用session只需要在客户端保存一个id，实际上大量数据都是保存在服务端。如果全部用cookie，数据量大的时候客户端是没有那么多空间的。
- 如果只用cookie不用session，那么账户信息全部保存在客户端，一旦被劫持，全部信息都会泄露。并且客户端数据量变大，网络传输的数据量也会变大



**简而言之, session 里面包含了用户的认证信息和登录状态等信息. 而 cookie 就是用户通行证**



### 有了cookie为什么需要session？

1. 用session只需要在客户端保存一个id，实际上大量数据都是保存在服务端。如果全部用cookie，数据量大的时候客户端是没有那么多空间的。
2. cookie只是实现session的其中一种方案。虽然是最常用的，但并不是唯一的方法。
3. 全部在客户端保存，服务端无法验证，这样伪造和仿冒会更加容易。（伪造一个随机的id很难，但伪造另一个用户名是很容易的）
4. 全部保存在客户端，那么一旦被劫持，全部信息都会泄露
5. 客户端数据量变大，网络传输的数据量也会变大



### token是什么

token是一种轻量级的用户验证方式。
token是无状态的。
token允许跨域访问。
token是服务端生成的一个字符串，保存在客户端（可以放在cookie中），作为请求服务的验证令牌。
token无需存放在服务端，这样服务端无需存放用户信息。
token对服务端压力极小`，因为服务端只需存储秘钥，并支持生成token的算法，无需存储token。
token最简单的构造：用户唯一的身份标识(辨识用户) + 时间戳(用于过期校验) + 签名(防止第三方恶意冒充)。
token无法主动过期，只能等待它达到过期时间后才会失效。
token的产生：首次请求时，服务器对请求参数（如账号、密码）验证通过，则根据用户标识，加上服务的密钥，通过生成算法，生成token。
token的验证：再次请求时，携带此token，则服务端再次根据用户标识，生成token，根据两个token是否一致且未过期来判定用户是否已授权。

以下几点特性会让你在程序中使用基于Token的身份验证

1. 无状态、可扩展
2. 支持移动设备
3. 跨程序调用
4. 安全