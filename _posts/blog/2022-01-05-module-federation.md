---
layout: post
title: webpack5 模块联邦
category: blog
description: webpack5 模块联邦
---

### webpack5 模块联邦使用方式

> 项目一

```javascript
//在webpack配置中以插件的形式引入
  const ModuleFederationPlugin = require("webpack/lib/container/ModuleFederationPlugin");
  plugins: [
    new ModuleFederationPlugin({
      name: "app_one_remote",//当前项目名称,需全局唯一
      remotes: {
        app_two_remote: //引用远程runtime状态的项目名称和地址映射到当前项目
          "app_two_remote@http://192.168.9.35:3001/remoteEntry.js",
      },
      shared: ["react", "react-dom"],//是非常重要的参数，制定了这个参数，可以让远程加载的模块对应依赖改为使用本地项目的 React 或 ReactDOM。
    }),
  ],
```

> 项目二

```javascript
//项目二webpack 插件选项配置
  plugins: [
    new ModuleFederationPlugin({
      name: "app_two_remote",//当前项目名称,需全局唯一
      filename: "remoteEntry.js",//整个远程打包文件
      exposes: {
        "./Search": "./src/Search",//导出的Search模块，只有在此申明的模块才可以作为远程依赖被使用
      },
      shared: ["react", "react-dom"],
    }),
  ],
```

#### 这里要注意需要用到动态导入入口文件的方法，因为 webpack 在静态编译阶段时，由于引用了远程的异步包，会存在还没加载远程包无法引用到对应包的情况

> 代码演示

```javascript
import("./test"); //首页入口文件用动态import方式方便webpack编译
```

### 基于模块联邦可以实践一下和后端的微服务一样前端微应用的互相引用，这样可以减少很多重复业务逻辑和组件的开发。
