---
layout: post
title: webpack 插件的基本原理
category: blog
description: webpack 插件的基本原理
---

## webpack 插件的基本原理

### 基本用法

```javascript
const HtmlWebpackPlugin = require("html-webpack-plugin");
```

```javascript
module.exports = {
  plugins: [
    new HtmlWebpackPlugin({
      template: "./public/index.html", // 模版文件
    }),
  ],
};
```

> 如上是一个 webpack 插件的基本用法，那么他的原理是什么呢

### 基本原理

#### 1. 调用 webpack 函数，接收 config 配置信息，实例化 compiler，初始化内置插件（调用所有内置插件的.apply）

#### 2. 调用 compiler.run 进入编译阶段

- 1.每一次新的编译都会实例化一个 compilation 对象，记录本次编译的基本信息
- 2.进入 make 阶段，即触发 compilation.hooks.make 钩子，从 entry 为入口
  - 1.执行对应的 loader 对模块进行预处理转为标准的 js 模块
  - 2.调用三方插件 acorn 对标准 JS 模块进行分析，递归每个依赖项，生成语法树
- 3.最后调用 compilation.seal render 模块，整合各个依赖项，最后输出一个或多个 chunk

### webpack 插件规范

- 一个 JavaScript 命名函数或 JavaScript 类。
- 在插件函数的 prototype 上定义一个 apply 方法。
- 指定一个绑定到 webpack 自身的事件钩子。
- 处理 webpack 内部实例的特定数据。
- 功能完成后调用 webpack 提供的回调。

> 基本插件架构及使用方法 ↓

```javascript
class HelloWorldPlugin {
  apply(compiler) {
    compiler.hooks.done.tap(
      "Hello World Plugin",
      (stats /* 绑定 done 钩子后，stats 会作为参数传入。 */) => {
        console.log("Hello World!");
      }
    );
  }
}

module.exports = HelloWorldPlugin;
```

```javascript
// webpack.config.js
var HelloWorldPlugin = require("hello-world");

module.exports = {
  // ... 这里是其他配置 ...
  plugins: [new HelloWorldPlugin({ options: true })],
};
```

```javascript
/*
 * @Descripttion: 插件测试
 * @Author: xietianfeng
 * @Date: 2022-01-04 14:35:05
 * @LastEditors: xietianfeng
 * @LastEditTime: 2022-01-04 16:08:40
 */

// 插件输出类
class RemoveCommentPlugin {
  constructor(opts) {
    this.opts = opts;
    this.externalModules = {};
  }
  apply(compiler) {
    // 去除注释正则
    const reg =
      /("([^\\\"]*(\\.)?)*")|('([^\\\']*(\\.)?)*')|(\/{2,}.*?(\r|\n))|(\/\*(\n|.)*?\*\/)|(\/\*\*\*\*\*\*\/)/g;

    // 注册自定义插件钩子到生成资源到 output 目录之前，拿到compilation对象（编译好的stream）
    compiler.hooks.emit.tap("RemoveComment", (compilation) => {
      // 遍历构建产物
      Object.keys(compilation.assets).forEach((item) => {
        // .source()是获取构建产物的文本
        // .assets中包含构建产物的文件名
        let content = compilation.assets[item].source();
        content = content.replace(reg, function (word) {
          // 去除注释后的文本
          return /^\/{2,}/.test(word) ||
            /^\/\*!/.test(word) ||
            /^\/\*{3,}\//.test(word)
            ? ""
            : word;
        });
        // console.info(content);
        // 更新构建产物对象
        compilation.assets[item] = {
          source: () => content,
          size: () => content.length,
        };
      });
    });
  }
}

module.exports = RemoveCommentPlugin;
```
