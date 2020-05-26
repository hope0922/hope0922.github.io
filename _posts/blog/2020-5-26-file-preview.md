---
layout:     post
title:     	PC+移动端文件预览解决方案
category: blog
description: PC+移动端文件预览解决方案
---
### PC+移动端文件预览解决方案

近期项目中碰到了文件预览的功能，包括office、文本及图片文件。文本和图片预览都是好解决，但是office的excel、ppt和word
本身浏览器是不支持预览加载的。在线预览的话，微软官方的在线预览接口是可以实现office的预览，但时项目部署不支持外网访问，
那么这条路也是走不通了。最后只能后台通过openoffice将上传的文件转成pdf格式，再传给前端实现预览。前端pdf预览浏览器是默认
支持的，但是移动端webview不支持pdf的预览。那就只能通过pdfh5插件转成svg或者canvas进行预览，canvas的性能更好一点。
