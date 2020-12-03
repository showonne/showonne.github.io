---
title: CORS请求头&&响应头概览
date: 2019-09-10 23:31:47
tags: [HTTP]
---

最近项目中遇到了跨域传Cookie的问题，在之前对于CORS的响应头只接触过`Access-Control-Allow-Origin`，其实除此之外还有很多跨域相关的请求头和响应头，在这整理一下。

<!-- more -->

文中的例子一共涉及两个工程，一个 `cors-client` 服务在本地 8080 端口，一个 `cors-server` 在本地 3000 端口，分别用来发送请求和处理请求。HTTP请求库使用`axios`，服务是`express`搭建的简单服务器。

## Access-Control-Allow-Origin

这是最常见的响应头，表示请求的资源能共享给哪些域。同源安全策略默认阻止跨域获取资源。如果直接发起跨域请求：

```js
const { code } = await axios.post('http://localhost:3000/api/demo');
```

浏览器会报跨域错误：

![](https://s2.ax1x.com/2019/09/10/naljFH.png)

因为没有 `Access-Control-Allow-Origin` 响应头，如果在服务端允许本地 8000 端口跨域访问：

```js
app.post('/api/demo', (req, res) => {
  res.header('Access-Control-Allow-Origin', 'http://localhost:8080')
  res.json({code: 200, message: 'Hello~'});
});
```

我们就能顺利访问到资源了 ，同时可以看到 Response Headers 里的  `Access-Control-Allow-Origin`，就是我们 client 的域：

![](https://s2.ax1x.com/2019/09/10/na1im8.png)

除此之外，还可以设置成 `*` 表示资源可以被任何域共享。


## Access-Control-Allow-Headers

这个请求头用于响应**预请求**，告知实际的请求中可以使用哪些 HTTP 头。不过预请求只有在复杂请求时才会发，复杂请求与简单请求相对，简单请求满足以下条件：

1. request header 是简单的请求头：
    - Accept
    - Accept-Language
    - Content-Language
    - Last-Event-ID
    - Content-Type
    等等非自定义的请求头

2. request method 是下面的请求类型：
    - HEAD
    - GET
    - POST

3. Content-Type 只限三个值：
    - application/x-www-form-urlencoded、multipart/form-data、text/plain

否则就是复杂请求, (实际上xhr绑定了`onloadStart`事件，body有值等也发送了 options 请求）。比如我们通常会在请求头里加上 `x-requested-with`，用来区分请求是否是ajax请求，这时请求就变成了一个复杂请求，刷新浏览器，可以在浏览器里发现浏览器没有立刻发出post请求，而是发送了一个options请求：

![](https://s2.ax1x.com/2019/09/11/na3CC9.png)

可以看到 request headers 里有 `Origin`，`Access-Control-Request-Method`, `Access-Control-Request-Headers`，有了这些信息，我们就可以对客户端想要发起的复杂请求的信息校验，进而决定是否允许真正的请求发送到服务端。其中：
- `Origin` 表示请求是从什么域发起的
- `Access-Control-Request-Headers`告知正式请求会用哪些请求头
- `Access-Control-Request-Method` 告知服务器正式请求的方法


`Access-Control-Allow-Headers` 告知正式请求中允许携带的请求头，简单的请求头，如 `Accept`、`Accept-Language`、`Content-Language`、`Content-Type` （只限于解析后的值为 `application/x-www-form-urlencoded`、`multipart/form-data` 或 `text/plain` 三种MIME类型（不包括参数））是始终被支持的，不需要特意列出。如果客户端发送了其他 `Access-Control-Allow-Headers` 中没有列出的头，就会被CORS策略限制，比如我们在请求头中加入 `x-requested-with`：

```js
const resp = await axios.post('http://localhost:3000/api/demo', {}, {
    headers: {
        'X-Requested-With': 'XMLHttpRequest'
    }
});
```

如果服务端没有做任何处理，浏览器就会报错：

![](https://s2.ax1x.com/2019/09/11/na33KP.png)

在服务端增加处理预检请求逻辑，增加 `Access-Control-Allow-Headers` 响应头：

```js
app.options('*', (req, res) => {
    res.header('Access-Control-Allow-Origin', 'http://localhost:8080')
    res.header('Access-Control-Allow-Headers', 'x-requested-with');
    res.end('');
});
```

这样在预检请求通过后，发送正式请求时就可以携带 `X-Requested-With` 请求头。

## Access-Control-Expose-Headers
这个响应头和 `Access-Control-Allow-Headers` 相反，`Access-Control-Allow-Headers` 表示客户端能携带什么请求头给服务端，`Access-Control-Expose-Headers` 表示服务端能暴露什么响应头给客户端。比如在服务端响应接口时通过`res.header`添加了自定义的响应头`validate-token`，如果没有添加 `Access-Control-Expose-Headers` 头，即使在浏览器调试工具里可以看到`validate-token`确实被返回了，但无法通过js获取到值。只有在服务端设置了响应头，才能在客户端代码中拿到对应响应头的值：

```js
// server
app.post('/api/demo', (req, res) => {
  res.header('Access-Control-Allow-Origin', 'http://localhost:8080')
  res.header('Access-Control-Expose-Headers', 'validate-token');
  res.header('validate-token', 'token-value');
  res.json({code: 200, message: 'Hello~'});
});

// client
const resp = await axios.post('http://localhost:3000/api/demo', {}, {
    headers: {
        'X-Requested-With': 'XMLHttpRequest'
    }
});
console.log(resp.headers['validate-token']); // token-value
```


## Access-Control-Max-Age
表示预检请求的返回结果（即 Access-Control-Allow-Methods 和Access-Control-Allow-Headers 提供的信息）可以被缓存多久，单位是秒。在Firefox中，上限是24小时 （即86400秒），而在Chromium 中则是10分钟（即600秒）。Chromium 同时规定了一个默认值 5 秒。如果值为 -1，则表示禁用缓存。如果没有特殊逻辑的话，预检请求基本都是直接通过，所以如果没有特殊需求，这个响应头的值可以设置得大一些，另外自己验证这个响应头的功能时记得吧 Chrome 里的 `Disable Cache` 关掉。


## Origin
表明获取资源的请求是从什么域发起的。

## Access-Control-Allow-Credentials
这也是一个很重要的响应头，指示当请求是凭证模式（Request.credentials = 'include'）时，是否响应该请求。
Credentials 可以是 cookies, authorization headers 或 TLS client certificates。其中 Cookie 是最常见的。

`Access-Control-Allow-Credentials` 头要和 `XMLHttpRequest.withCredentials` 或 `Fetch API中的Request()` 构造器中的 `credentials` 选项结合使用。Credentials 必须在前后端都被配置（即the Access-Control-Allow-Credentials header 和 XHR 或Fetch request中都要配置）才能使带 `credentials` 的CORS请求成功。

举个例子，如果想在跨域请求带上 Cookie，以 `axios` 为例，客户端首先要设置 `withCredentials: true`

```js
const resp = await axios.post('http://localhost:3000/api/demo', {}, {
    headers: {
        'X-Requested-With': 'XMLHttpRequest'
    },
    withCredentials: true
});
```

同时，服务端也要设置 `Access-Control-Allow-Credentials`，这里 options 和 正式请求都要设置：

```js
app.options('*', (req, res) => {
    res.header('Access-Control-Allow-Origin', 'http://localhost:8080');
    res.header('Access-Control-Max-Age', 60);
    res.header('Access-Control-Allow-Headers', 'content-type,x-requested-with');
    res.header('Access-Control-Expose-Headers', 'validate-token');
    res.header('Access-Control-Allow-Credentials', true);
    res.end('');
});
```

```js
app.post('/api/demo', (req, res) => {
    console.log('[COOKIE]', req.headers.cookie)
    res.header('Access-Control-Allow-Origin', 'http://localhost:8080')
    res.header('Access-Control-Expose-Headers', 'validate-token');
    res.header('validate-token', 'token-value');
    res.header('Access-Control-Allow-Credentials', true);
    res.json({code: 200, message: 'Hello~'});
});
```

刷新浏览器发送请求，服务端就能在跨域的情况加接收到 Cookie：

![](https://s2.ax1x.com/2019/09/12/nwI0lF.png)

## 总结
`CORS`是通过传输的HTTP头组成的一个系统，用来决定浏览器是否要阻止跨域请求的响应。一般情况下，只需要设置 `Access-Control-Allow-Origin` 就能解决一大半的场景；如果需要携带自定义的请求头，响应头，就要用到 `Access-Control-Allow-Headers` 和 `Access-Control-Expose-Headers` 了；`Access-Control-Request-Header` 和 `Access-Control-Request-Method`，`Origin` 无需设置，浏览器会自动带上，在进行自定义校验时可能会派上用场；如果想要在跨域请求头携带认证信息，就一定要设置 `Access-Control-Allow-Credentials` 响应头，客户端也要启用认证模式。

跨域的问题还是很好解决的，因为当JS代码因为同源策略被阻塞时，浏览器给出的错误提示已经足够详细了，只要根据错误信息修改头信息就能万事大吉了~