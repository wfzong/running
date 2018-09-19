# 开发项目的vue ssr版本

## 什么是服务器端渲染(SSR)？

Vue.js 是构建客户端应用程序的框架。默认情况下，可以在浏览器中输出 Vue 组件，进行生成 DOM 和操作 DOM。然而，也可以将同一个组件渲染为服务器端的 HTML 字符串，将它们直接发送到浏览器，最后将这些静态标记"激活"为客户端上完全可交互的应用程序。

服务器渲染的 Vue.js 应用程序也可以被认为是"同构"或"通用"，因为应用程序的大部分代码都可以在**服务器**和**客户端**上运行。


## 为什么使用服务器端渲染(SSR)？

与传统 SPA（Single-Page Application - 单页应用程序）相比，服务器端渲染(SSR)的优势主要在于：

- 更好的 SEO，由于搜索引擎爬虫抓取工具可以直接查看完全渲染的页面。
- 更快的内容到达时间(time-to-content)，特别是对于缓慢的网络情况或运行缓慢的设备。


## 基本用法
安装需要用到的模板
> npm install vue vue-server-renderer express --save

新建 **/server.js** 、**/src/index.template.html**

```javascript
const server = require('express')()
const Vue = require('vue')
const fs = require('fs')

const Renderer = require('vue-server-renderer').createRenderer({
  template:fs.readFileSync('./src/index.template.html', 'utf-8')
})

server.get('*', (req, res) => {

  const app = new Vue({
    data: {
      name: 'vue app~',
      url: req.url
    },
    template:'<div>hello from {{name}}, and url is: {{url}}</div>'
  })
  const context = {
    title: 'SSR test#'
  }
  Renderer.renderToString(app, context, (err, html) => {
    if(err) {
      console.log(err)
      res.status(500).end('server error')
    }
    res.end(html)
  })
})

server.listen(4001)
console.log('running at: http://localhost:4001');
```
通过以上程序，可以看到通过 **vue-server-renderer** 将VUE实例进行编译，最终通过 express 输出到浏览器。

但同时也能看到，输出的是一个静态的纯html页面，由于没有加载任何 javascript 文件，前端的用户交互也无所实现，所以上面的 demo 只是一个极简的实例，要想实现一个完整的 VUE ssr 程序，还需要借助 **VueSSRClientPlugin**(vue-server-renderer/client-plugin) 将文件编译成前端浏览器可运行的 vue-ssr-client-manifest.json 文件和 js、css 等文件，**VueSSRServerPlugin**(vue-server-renderer/server-plugin) 将文件编译成可供node调用的 vue-ssr-server-bundle.json 

真正开始之前，需要了解一些概念

## 编写通用代码
"通用"代码时的约束条件 - 即运行在服务器和客户端的代码，由于用例和平台 API 的差异，当运行在不同环境中时，我们的代码将不会完全相同。
### 服务器上的数据响应
每个请求应该都是全新的、独立的应用程序实例，以便不会有交叉请求造成的状态污染(cross-request state pollution)

### 组件生命周期钩子函数
由于没有动态更新，所有的生命周期钩子函数中，只有 beforeCreate 和 created 会在服务器端渲染(SSR)过程中被调用

### 访问特定平台(Platform-Specific) API
通用代码不可接受特定平台的 API，因此如果你的代码中，直接使用了像 window 或 document，这种仅浏览器可用的全局变量，则会在 Node.js 中执行时抛出错误，反之也是如此。

## 构建配置
如何将相同的 Vue 应用程序提供给服务端和客户端。为了做到这一点，我们需要使用 webpack 来打包 Vue 应用程序。

- 通常 Vue 应用程序是由 webpack 和 vue-loader 构建，并且许多 webpack 特定功能不能直接在 Node.js 中运行（例如通过 file-loader 导入文件，通过 css-loader 导入 CSS）。

- 尽管 Node.js 最新版本能够完全支持 ES2015 特性，我们还是需要转译客户端代码以适应老版浏览器。这也会涉及到构建步骤。

所以基本看法是，对于客户端应用程序和服务器应用程序，我们都要使用 webpack 打包 - 服务器需要「服务器 bundle」然后用于服务器端渲染(SSR)，而「客户端 bundle」会发送给浏览器，用于混合静态标记。

![构建过程](https://cloud.githubusercontent.com/assets/499550/17607895/786a415a-5fee-11e6-9c11-45a2cfdf085c.png)

下面看具体实现过程

### Babel配置

新建 **/.babelrc**  配置
``` json
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

### webpack 配置
新建一个 build 文件夹，用于存放 `webpack` 相关的配置文件
``` bash
/
├── build
│   ├── setup-dev-server.js  # 设置 webpack-dev-middleware 开发环境
│   ├── webpack.base.config.js # 基础通用配置
│   ├── webpack.client.config.js  # 编译出 vue-ssr-client-manifest.json 文件和 js、css 等文件，供浏览器调用
│   └── webpack.server.config.js  # 编译出 vue-ssr-server-bundle.json 供 nodejs 调用
```

先把相关的包安装

安装 webpack 相关的包
> npm i -D webpack webpack-cli webpack-dev-middleware webpack-hot-middleware webpack-merge webpack-node-externals

安装构建依赖的包
> npm i -D chokidar cross-env friendly-errors-webpack-plugin memory-fs rimraf vue-loader

接下来看每个文件的具体内容：
> webpack.base.config.js
``` javascript
const path = require('path')
const { VueLoaderPlugin } = require('vue-loader')

const isProd = process.env.NODE_ENV === 'production'
module.exports = {
  context: path.resolve(__dirname, '../'),
  devtool: isProd ? 'source-map' : '#cheap-module-source-map',
  output: {
    path: path.resolve(__dirname, '../dist'),
    publicPath: '/dist/',
    filename: '[name].[chunkhash].js'
  },
  resolve: {
    // ...
  },
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: {
          compilerOptions: {
            preserveWhitespace: false
          }
        }
      }
      // ...
    ]
  },
  plugins: [new VueLoaderPlugin()]
}
```
`webpack.base.config.js` 这个是通用配置，和我们之前SPA开发配置基本一样。

> webpack.client.config.js
``` javascript
const webpack = require('webpack')
const merge = require('webpack-merge')
const base = require('./webpack.base.config')
const VueSSRClientPlugin = require('vue-server-renderer/client-plugin')

const config = merge(base, {
  mode: 'development',
  entry: {
    app: './src/entry-client.js'
  },
  resolve: {},
  plugins: [
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(
        process.env.NODE_ENV || 'development'
      ),
      'process.env.VUE_ENV': '"client"'
    }),
    new VueSSRClientPlugin()
  ]
})
module.exports = config
```
`webpack.client.config.js` 主要完成了两个工作
- 定义入口文件 `entry-client.js`
- 通过插件 `VueSSRClientPlugin` 生成 `vue-ssr-client-manifest.json`

这个 manifest.json 文件被 server.js 引用
``` javascript
const { createBundleRenderer } = require('vue-server-renderer')

const template = require('fs').readFileSync('/path/to/template.html', 'utf-8')
const serverBundle = require('/path/to/vue-ssr-server-bundle.json')
const clientManifest = require('/path/to/vue-ssr-client-manifest.json')

const renderer = createBundleRenderer(serverBundle, {
  template,
  clientManifest
})

```
通过以上设置，使用代码分割特性构建后的服务器渲染的 HTML 代码，所有都是自动注入。

> webpack.server.config.js
``` javascript
const webpack = require('webpack')
const merge = require('webpack-merge')
const base = require('./webpack.base.config')
const nodeExternals = require('webpack-node-externals') // Webpack allows you to define externals - modules that should not be bundled.
const VueSSRServerPlugin = require('vue-server-renderer/server-plugin')

module.exports = merge(base, {
  mode: 'production',
  target: 'node',
  devtool: '#source-map',
  entry: './src/entry-server.js',
  output: {
    filename: 'server-bundle.js',
    libraryTarget: 'commonjs2'
  },
  resolve: {},
  externals: nodeExternals({
    whitelist: /\.css$/ // 防止将某些 import 的包(package)打包到 bundle 中，而是在运行时(runtime)再去从外部获取这些扩展依赖
  }),
  plugins: [
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV || 'development'),
      'process.env.VUE_ENV': '"server"'
    }),
    new VueSSRServerPlugin()
  ]
})

```
`webpack.server.config.js` 主要完成的工作是：
- 通过 `target: 'node'` 告诉 webpack 编译的目录代码是 node 应用程序
- 通过 `VueSSRServerPlugin` 插件，将代码编译成 `vue-ssr-server-bundle.json`

在生成 `vue-ssr-server-bundle.json` 之后，只需将文件路径传递给 `createBundleRenderer` ，在 `server.js` 中如下实现：
``` javascript
const { createBundleRenderer } = require('vue-server-renderer')
const renderer = createBundleRenderer('/path/to/vue-ssr-server-bundle.json', {
  // ……renderer 的其他选项
})
```

至此，基本已经完成构建

## SSR 程序开发

安装 VUE 相关的依赖包

> npm i axios vue-template-compiler vue-router vuex vuex-router-sync

新增并完善如下文件：
``` bash
/
├── server.js # 实现长期运行的 node 程序
├── src
│   ├── app.js # 新增
│   ├── router.js # 新增 定义路由
│   ├── App.vue # 新增
│   ├── entry-client.js # 浏览器端入口
│   ├── entry-server.js # node程序端入口
└── views
    └── Home.vue # 首页
```
接下来逐个看这些文件：

> server.js
``` javascript
const fs = require('fs');
const path = require('path');
const express = require('express');
const { createBundleRenderer } = require('vue-server-renderer');
const devServer = require('./build/setup-dev-server')
const resolve = file => path.resolve(__dirname, file);

const isProd = process.env.NODE_ENV === 'production';
const app = express();

const serve = (path, cache) =>
  express.static(resolve(path), {
    maxAge: cache && isProd ? 1000 * 60 * 60 * 24 * 30 : 0
  });
app.use('/dist', serve('./dist', true));

function createRenderer(bundle, options) {
  return createBundleRenderer( bundle, Object.assign(options, {
      basedir: resolve('./dist'),
      runInNewContext: false
    })
  );
}

function render(req, res) {
  const startTime = Date.now();
  res.setHeader('Content-Type', 'text/html');

  const context = {
    title: 'SSR 测试', // default title
    url: req.url
  };
  renderer.renderToString(context, (err, html) => {
    res.send(html);
  });
}

let renderer;
let readyPromise;
const templatePath = resolve('./src/index.template.html');

if (isProd) {
  const template = fs.readFileSync(templatePath, 'utf-8');
  const bundle = require('./dist/vue-ssr-server-bundle.json');
  const clientManifest = require('./dist/vue-ssr-client-manifest.json') // 将js文件注入到页面中
  renderer = createRenderer(bundle, {
    template,
    clientManifest
  });
} else {
  readyPromise = devServer( app, templatePath, (bundle, options) => {
      renderer = createRenderer(bundle, options);
    }
  );
}

app.get('*',isProd? render : (req, res) => {
        readyPromise.then(() => render(req, res));
      }
);

const port = process.env.PORT || 8088;
app.listen(port, () => {
  console.log(`server started at localhost:${port}`);
});

```
`server.js` 主要完成了以下工作
- 当执行 `npm run dev` 的时候，调用 `/build/setup-dev-server.js` 启动 'webpack-dev-middleware' 开发中间件
- 通过 `vue-server-renderer` 调用之前编译生成的 `vue-ssr-server-bundle.json` 启动 node 服务
- 将 `vue-ssr-client-manifest.json` 注入到 `createRenderer` 中实现前端资源的t自动注入
- 通过 `express` 处理 `http` 请求

`server.js` 是整个站点的入口程序，通过他调用编译过后的文件，最终输出到页面，是整个项目中很关键的一部分

> app.js
``` javascript
import Vue from 'vue'
import App from './App.vue';
import { createRouter } from './router';

export function createApp(context) {
  const router = createRouter();

  const app = new Vue({
    router,
    render: h => h(App)
  });
  return { app, router };
};
```
`app.js` 暴露一个可以重复执行的工厂函数，为每个请求创建新的应用程序实例，提交给 'entry-client.js' 和 `entry-server.js` 调用

> entry-client.js
``` javascript
import { createApp } from './app';
const { app, router } = createApp();
router.onReady(() => {
  app.$mount('#app');
});
```
`entry-client.js` 常规的实例化 vue 对象并挂载到页面中

> entry-server.js
```javascript
import { createApp } from './app';

export default context => {
  // 因为有可能会是异步路由钩子函数或组件，所以我们将返回一个 Promise，
  // 以便服务器能够等待所有的内容在渲染前，
  // 就已经准备就绪。
  return new Promise((resolve, reject) => {
    const { app, router } = createApp(context);

    // 设置服务器端 router 的位置
    router.push(context.url);

    // 等到 router 将可能的异步组件和钩子函数解析完
    router.onReady(() => {
      const matchedComponents = router.getMatchedComponents();

      // 匹配不到的路由，执行 reject 函数，并返回 404
      if (!matchedComponents.length) {
        return reject({ code: 404 });
      }

      resolve(app);
    });
  });
};
```
`entry-server.js` 作为服务器入口，最终经过 `VueSSRServerPlugin` 插件，编译成 `vue-ssr-server-bundle.json` 供 `vue-server-renderer` 调用

`router.js` 和 `Home.vue` 为常规 `vue` 程序，这里不进一步展开了。

至此，我们实现了一个可以完整编译和运行的 `vue ssr` 程序
