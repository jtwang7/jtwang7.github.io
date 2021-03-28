---
title: Cookie,document.cookie
date: 2021-03-28 17:03
tags: JavaScript
categories: JavaScript
excerpt:  Cookie由服务端在响应阶段设置，浏览器请求阶段添加，属于HTTP协议的一部分。通常存储在浏览器中，数据量较小，常用于用户登录身份验证。
---

# Cookie,document.cookie
1. Cookie : 直接**存储在浏览器**中的一小串数据。是 **HTTP 协议的一部分**，由 RFC 6265 规范定义。
2. Cookie 通常是**由 Web 服务器使用响应 `Set-Cookie HTTP-header` 设置**的。然后**浏览器**使用 `Cookie HTTP-header` 将它们自动添加到（几乎）每个对相同域的请求中。(服务器设置，并由浏览器添加至请求)

## Cookie 作用：身份验证
1. 用户登录后，服务器**在响应中**使用 Set-Cookie HTTP-header 来设置具有唯一“会话标识符（session identifier）”的 cookie。
2. 下次如果请求是由**相同域**发起的，浏览器会使用 Cookie HTTP-header 通过网络发送 cookie。
3. 服务器接收时识别cookie就知道是谁发起了请求。
**就好比公司打卡，第一天上班HR根本不认识你，此时会给你发工作证来标明你唯一的身份，之后打卡，HR只要识别你工作证就能知道是你打卡而不是别人打卡。**

## document.cookie
cookie 是服务器在响应阶段设置的，在浏览器端可以通过 `document.cookie` 访问
### 本质
`document.cookie` 不是对象属性，而是一个访问器(getter/setter)，因此可以读取和写入。

### 读取
`console.log(document.cookie)` 隐式调用了 get 方法，并返回所有的cookie。
`document.cookie` 的值由 `name=value` 对组成，以`;`分隔。每个都是独立的 cookie。**key 和 value 都不需要 '' **

### 写入
`document.cookie = 'user=John'` 调用 set 方法写入一个cookie
此处只是插入了一个cookie，而不是覆盖了所有cookie，这就证明了document.cookie是访问器而不是对象属性。
cookie 的 name 和 value 可以是任何字符，为了保证格式有效，通常需要用内建的函数(**`encodeURIComponent`**)进行转义。
```
// 特殊字符（空格），需要编码
let name = "my name";
let value = "John Smith"

// 将 cookie 编码为 my%20name=John%20Smith
document.cookie = encodeURIComponent(name) + '=' + encodeURIComponent(value);

alert(document.cookie); // ...; my%20name=John%20Smith
```

## Cookie 的设置选项
在写入 Cookie 时，可以设置相应的选项：选项写在 key=value 之后，并以 `;` 分隔。
`document.cookie = "user=John; path=/; expires=Tue, 19 Jan 2038 03:14:07 GMT"`

1. `path=/mypath`：url 路径前缀，该路径下的页面可以访问该 cookie。必须是绝对路径。默认为当前路径。
如果一个 cookie 带有 `path=/admin` 设置，那么该 cookie 在 `/admin` 和 `/admin/something` 下都是可见的，但是在 `/home` 或 `/adminpage` 下不可见。**即设置路径及其子路径可访问cookie**
通常，我们应该将 path 设置为根目录：path=/，以使 cookie 对此网站的所有页面可见。
2. `domain=site.com`：设置可访问 cookie 的根域，其使得该域下所有子域均可访问 cookie。
默认情况下，cookie 只有在当前域下才能被访问到。如果 cookie 设置在 site.com 下，我们在 other.com 下就无法获取它，甚至在 site.com 子域下也无法访问。显示的 domain 设置可以解决上述问题。
3. `expires=Tue, 19 Jan 2038 03:14:07 GMT`：cookie 的到期日期，那时浏览器会自动删除它。
日期必须完全采用 GMT 时区的这种格式。可以使用 `date.toUTCString` 来获取它。
4. `max-age=3600`：expires 的替代选项，具指明 cookie 的过期时间距离当前时间的秒数。
5. `secure`：设置 Cookie 只能通过 HTTPS 传输
cookie 是基于域的，它们不区分协议。默认情况下，如果我们在 http://site.com 上设置了 cookie，那么该 cookie 也会出现在 https://site.com 上，反之亦然。使用此选项，如果一个 cookie 是通过 https://site.com 设置的，那么它不会在相同域的 HTTP 环境下出现
6. `samesite`：防止 XSRF（跨网站请求伪造）攻击。
XSRF? 用户在某页面登陆时，会向该页面发送cookie，页面核实cookie后允许用户进行后续操作。通常发送cookie请求是用户登录时执行的，但是cookie请求可以通过脚本执行！即黑客在其他网站可以用脚本模拟用户登录，从而欺骗页面。
`samesite=strict` 保证如果用户来自同一网站之外，那么设置了 samesite=strict 的 cookie 永远不会被发送。只有在该网站下的操作才会发送 cookie。但是这样同样会屏蔽一些用户默许的合法链接，比如通过用户自己的笔记或收藏夹访问等。**解决办法：**通过使用两个 cookie 来解决这个问题：一个 cookie 用于“一般识别”，另一个带有 samesite=strict 的 cookie 用于进行数据更改的操作。这样，从网站外部来的用户可以访问网站，但是支付操作等必须是从银行网站启动的，这样第二个 cookie 才能被发送。
`samesite=lax` 宽松模式，当从外部来到网站，则禁止浏览器发送 cookie，但是增加了一个例外。如果以下两个条件均成立，则会发送 samesite=lax cookie：a. HTTP 方法是“安全的”（例如 GET 方法，而不是 POST）b. 执行顶级导航（更改浏览器地址栏中的 URL）
7. httpOnly：禁止任何 JavaScript 访问 cookie。我们使用 document.cookie 看不到此类 cookie，也无法对此类 cookie 进行操作。即 Cookie 操作只能在服务端进行。