---
layout: post
title: react-fiber
category: blog
description: react-fiber
---

### React Fiber 【Fiber 架构就是用 异步的方式解决旧版本 同步递归导致的性能问题】

旧版 React 通过递归的方式进行渲染，使用的是 【JS 引擎自身的函数调用栈】，它会一直执行到栈空为止。
Fiber 实现了自己的【组件调用栈】，它以链表的形式遍历组件树，可以灵活的暂停、继续和丢弃执行的任务。
实现方式是使用了浏览器的 requestIdleCallback 这一 API:
将 计算任务 分给成一个个小任务，分批完成，在完成一个小任务后，将控制权还给浏览器，让浏览器利用间隙进行 UI 渲染

### React Fiber 出现的背景

当页面元素很多，且需要频繁刷新的场景下，浏览器页面会出现卡顿现象，原因是因为 计算任务 持续占据着主线程，从而阻塞了 UI 渲染

### React 框架内部的运作可以分为 3 层

- Virtual DOM 层 --> 描述页面长什么样
- Reconciler 层 --> 负责调用组件生命周期方法，进行 Diff 运算等。
- Renderer 层 --> 根据不同的平台，渲染出相应的页面，比较常见的是 ReactDOM 和 ReactNative

```javascript
 *
 * Fiber 其实指的是一种数据结构，它可以用一个纯 JS 对象来表示
 *
 * const fiber = {
        stateNode,    // 节点实例
        child,        // 子节点
        sibling,      // 兄弟节点
        return,       // 父节点
    }
```

### Fiber Reconciler 的 2 个阶段:

- 阶段一，生成 Fiber 树【本质来说是一个链表】，得出需要更新的节点信息。这一步是一个渐进的过程，可以被打断。
  【让优先级更高的任务先执行，从框架层面大大降低了页面掉帧的概率】
  - 阶段一有两颗树，Virtual DOM 树和 Fiber 树，Fiber 树是在 Virtual DOM 树的基础上通过额外信息生成的。
    它每生成一个新节点，就会将控制权还给浏览器，如果浏览器没有更高级别的任务要执行，则继续构建；反之则会丢弃 正在生成的 Fiber 树，等空闲的时候再重新执行一遍
- 阶段二，将需要更新的节点一次过批量更新，这个过程不能被打断

### 【V16 版本之前】栈调和（Stack reconciler） --> 【V16 版本之后】 Fiber reconciler

React diff 将传统 diff 算法的复杂度 O(n^3) 复杂度的问题转换成 O(n) 复杂度的问题
React Diff 三大策略: 【将 Virtual DOM 树转换成 actual DOM 树的最少操作的过程称为 协调（Reconciliaton）】

- 1.tree diff: 【Web UI 中 DOM 节点跨层级的移动操作特别少，可以忽略不计】
  - React 对 Virtual DOM 树进行层级控制，只会对相同层级的 DOM 节点进行比较，即同一个父元素下的所有子节点，
  - 当发现节点已经不存在了，则会删除掉该节点下所有的子节点，不会再进行比较。
  - 这样只需要对 DOM 树进行一次遍历，就可以完成整个树的比较。复杂度变为 O(n);
- 2.component diff: 【拥有相同类的两个组件 生成相似的树形结构,拥有不同类的两个组件 生成不同的树形结构】
- 3.element diff: 【对于同一层级的一组子节点，通过唯一 id 区分】 当节点属于同一层级：插入、移动、删除 【设置唯一 key 的策略】
