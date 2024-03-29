---
layout: post
title: Chrome DevTools 分享
category: blog
description: Chrome DevTools 例会演示分享
---

### Chrome DevTools 分享

#### 箭头按钮

用于在页面选择一个元素来审查和查看它的相关信息，当我们在 Elements 这个按钮页面下点击某个 Dom 元素时，箭头按钮会变成选择状态

#### 设备图标

点击它可以切换到不同的终端进行开发模式，移动端和 pc 端的一个切换，可以选择不同的移动终端设备，同时可以选择不同的尺寸比例，chrome 浏览器的模拟移动设备和真实的设备相差不大，是非常好的选择
这是一个非常标准化的流程， 除了 1、2 两个步骤需要人为操作，其他都是自动运行，减少了人工操作的风险，省去了重复性工作，增强了项目的可见性

#### Elements

功能标签页：用来查看，修改页面上的元素，包括 DOM 标签，以及 css 样式的查看，修改，还有相关盒模型的图形信息，下图我们可以看到当我鼠标选择 id 为 lg_tar 的 div 元素时，右侧的 css 样式对应的会展示出此 id 的样式信息，此时可以在右侧进行一个修改，修改即可在页面上生效， 灰色的 element.style 样式同样可以进行添加和书写，唯一的区别是，在这里添加的样式是添加到了该元素内部，实现方式即：该 div 元素的 style 属性，这个页面的功能很强大，在我们做了相关的页面后，修改样式是一块很重要的工作，细微的差距都需要调整，但是不可能说做到每修改一点即编译一遍代码，再刷新浏览器查看效果，这样很低效，一次性在浏览器中修改之后，再到代码中进行修改

- ALT+左键点击节点 ->展开所有子节点

#### Console 控制台

用于打印和输出相关的命令信息，其实 console 控制台除了我们熟知的报错，打印 console.log 信息外，还可以做数据的格式化预览

#### Sources

js 资源页面：这个页面内我们可以找到当前浏览器页面中的 js 源文件，方便我们查看和调试，在我还没有走出校园时候，我经常看一些大站的 js 代码，那时候其实基本都看不懂，但是最起码可以看看人家的代码风格，人家的命名方式，所有的代码都是压缩之后的代码，我们可以点击下面的{}大括号按钮将代码转成可读格式

#### Network

网络请求标签页：可以看到所有的资源请求，包括网络请求，图片资源，html,css，js 文件等请求，可以根据需求筛选请求项，一般多用于网络请求的查看和分析，分析后端接口是否正确传输，获取的数据是否准确，请求头，请求参数的查看

- 可以通过 Initiator 查看当前请求是哪个方法发出
- 扩展 可以看到请求回来的数据，可以用 filter 筛选对应的请求得到想要的数据
- 限制网速 切换网络形式
- copy as fetch 更改参数调试线上的接口
- preserve log 可以保留打印的调试信息和请求信息

#### Performance

标签页可以显示 JS 执行时间、页面元素渲染时间，不做过多介绍

#### Memory

标签页可以查看 CPU 执行时间与内存占用，不做过多介绍

#### Application

当前浏览器打开网站应用的浏览器数据，缓存-cokkie-本地浏览器数据库信息

#### Security

标签页 可以告诉你这个网站的安全性，查看有效的证书

#### LightHouse

可以帮你分析页面性能，有助于优化前端页面，分析后得到的报告

### 如何调试

怎么找到对应的文件-》使用 ctrl+p 打开对应的文件
隐藏的命令 command ctrl+shift+p
Capture
source 代码打断点，项目代码打 debugger（不建议，容易忘记删除）
clear site data 手动清理缓存
还可以使用条件断点，判断是否进入断点调试
移动端调试 需要外网，usb 连接手机，手机开启开发者模式和 usb 调试模式，连接时授权密钥。
在 chrome 浏览器地址栏输入 chrome://inspect/，点击对应连接设备调试就可以，并切换成移动端模式
一般建议直接用 edge 浏览器进行移动端（不需要外网）方法类似
local override
线上环境如果有 source-map 代码可读性较高的情况 直接用本地 js 文件副本代替线上版本，修改进行调试（只能在编译好混淆的文件中修改才能生效）

- 为什么要做代码混淆和加密，就是为了安全性考虑
