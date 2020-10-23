---
layout:     post
title:     	PC+移动端文件预览解决方案（离线无外网）
category: blog
description: PC+移动端文件预览解决方案（离线无外网）
---
### PC+移动端文件预览解决方案

近期项目中碰到了文件预览的功能，包括office、文本及图片文件。文本和图片预览都是好解决，但是office的excel、ppt和word
本身浏览器是不支持预览加载的。在线预览的话，微软官方的在线预览接口是可以实现office的预览，但时项目部署不支持外网访问，
那么这条路也是走不通了。最后只能后台通过openoffice将上传的文件转成pdf格式，再传给前端实现预览。前端pdf预览浏览器是默认
支持的，但是移动端webview不支持pdf的预览。那就只能通过pdfh5插件转成svg或者canvas进行预览，canvas的性能更好一点。后续
又出现一个问题就是大文件pdf在webview用第三方库渲染成canvas预览时，会有内存溢出的情况出现导致app闪退。最后移动端只能改成
用安卓原生的方法预览，性能也大大提升。


### 2020-8-13更新

因为数据中台不支持文件地址的直接返回，而是返回二进制的文件流。文件预览方案相应的需要作出变化。

```javascript
//初始方案将返回的二进制流blob对象赋值给img、video及iframe进行预览

window.URL.createObjectURL(blob);
```

图片和视频是正常的，赋值给iframe一直预览不出来pdf、txt和html文件
查询资料后发现iframe不能根据blob对象的地址判断文件是什么类型从而进行预览

所以根据文件类型给blob对象加上对应的type
```javascript
    blob = new Blob([blob], {
        type: 'application/pdf;chartset=UTF-8'
        //type: 'text/html;chartset=UTF-8'
    });
```

再次预览，一切正常，完美

但是这种方式需要等待二进制流数据的返回才能进行后续的预览，一般只要把接口给浏览器，浏览器会自动识别文件进行预览。

和后端沟通后 在请求二进制流的接口后带上文件名称和后缀，然后再给iframe，这样就可以不需要等待二进制流的返回就可以直接预览，
浏览器也会自动识别出现等待的loading界面，效果更好。