# HTTP 消息头 及 浏览器缓存 小结
作为开发人员大家都知道，从网络上获取资源成本比较高，客户端需要和服务端要进行多次通讯，如果能有效利用缓存，可以极大提高 web 应用的性能，所以有必要详细了解一个关于缓存的各个细节。

为防止出现理解上的误差，在开始之前我们约定关于缓存生效的条件是：当用户打个某个网址或者应用 `以后` 把它关闭，然后 `再次打开` 的的情况。

关于用户主动点击了 刷新 或者  强制刷新(ctrl+f5) 的情况，我们后面再说。

## 概念篇

浏览器缓存分两个类型：`非验证性缓存` 和 `验证性缓存`

**非验证性缓存**：浏览器根据过期时间来判断，如果在有效期内，直接从浏览器缓存中存文件，**不发生http请求**，涉及到的 header 字段有 `Cache-Control`、`expires`、`pragma`

**验证性缓存**：给服务端发送请求时，在 header 里附带条件，服务端在处理请求时根据指定条件做出判断，如果符合条件则返回一个 304 状态码并返回空的 body ，浏览器根据状态码判断，从本地缓存中取内容；如果条件为假则返回 200 状态码并返回指定资源。涉及到的 header 字段有 `etag` 、 `last-modified`

非验证性缓存 > 验证性缓存，如果本地缓存在有期内，甚至都不会发出 http 请求


etag —— if-none-match

last-modified —— if-modified-since


[前端工程师学习 Nginx 入门篇](https://juejin.im/entry/56f23b77a34131005438d2e5)


> **Note:** Cache-Control 标头是在 HTTP/1.1 规范中定义的，取代了之前用来定义响应缓存策略的标头（例如 Expires）。所有现代浏览器都支持 Cache-Control，因此，使用它就够了。


## 参考文档
- [https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers)
- [https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Etag](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Etag)
- [https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Last-Modified](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Last-Modified)
- [https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cache-Control](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cache-Control)
- [https://blog.csdn.net/eroswang/article/details/8302191](https://blog.csdn.net/eroswang/article/details/8302191)
- [https://www.v2ex.com/t/356353](https://www.v2ex.com/t/356353)
- [https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching)
- [http://imweb.io/topic/5795dcb6fb312541492eda8c](http://imweb.io/topic/5795dcb6fb312541492eda8c)