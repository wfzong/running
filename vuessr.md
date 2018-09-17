# 开发项目的vue ssr版本

## 什么是服务器端渲染(SSR)？

Vue.js 是构建客户端应用程序的框架。默认情况下，可以在浏览器中输出 Vue 组件，进行生成 DOM 和操作 DOM。然而，也可以将同一个组件渲染为服务器端的 HTML 字符串，将它们直接发送到浏览器，最后将这些静态标记"激活"为客户端上完全可交互的应用程序。

服务器渲染的 Vue.js 应用程序也可以被认为是"同构"或"通用"，因为应用程序的大部分代码都可以在**服务器**和**客户端**上运行。


## 为什么使用服务器端渲染(SSR)？

与传统 SPA（Single-Page Application - 单页应用程序）相比，服务器端渲染(SSR)的优势主要在于：

- 更好的 SEO，由于搜索引擎爬虫抓取工具可以直接查看完全渲染的页面。
- 更快的内容到达时间(time-to-content)，特别是对于缓慢的网络情况或运行缓慢的设备。


### 环境构建
> .babelrc 
```
// es6 compile to es5 相关配置
{
  "presets": [
    [
      "env",
      {
        "modules": false
      }
    ]
  ],
  "plugins": ["syntax-dynamic-import"]
}

npm i -D babel-loader@7 babel-core babel-plugin-syntax-dynamic-import babel-preset-env
```

### 基本用法
> server.js

```
npm install vue vue-server-renderer express --save

const server = require('express')();
const Vue = require('vue');

const renderer = require('vue-server-renderer').createRenderer();

server.get('*', (req, res) => {
  const app = new Vue({
    data: {
      name: 'vue~',
      url: req.url
    },
    template: `<div>Hello from {{name}}, and from: {{url}}</div>`
  });


  renderer.renderToString(app, (err, html) => {
    if (err) {
      res.status(500).end('err');
      return;
    }
    res.send(html);
  });
});

server.listen(3003);
console.log('running at: http://localhost:3003');

```
