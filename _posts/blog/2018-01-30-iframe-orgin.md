---
layout:     post
title:      iframe跨域传输数据
category: blog
description: iframe跨域传输数据（一）；子页面访问主框架DOM元素；  
---

##iframe跨域传输数据（一）；子页面访问主框架DOM元素；  

如果使用同域的方法，浏览器判断A.html 与 B.html 不同域，会有错误提示。 
Uncaught SecurityError: Blocked a frame with origin “http://localhost” from accessing a frame with origin “http://127.0.0.0“. Protocols, domains, and ports must match.

####实现原理： 
因为浏览器为了安全，禁止了不同域访问。因此只要调用与执行的双方是同域则可以相互访问。 
不同方式的解决方法如下：
###解决方法一：
```javascript
同域下：
使用 window.top或者window.parent.parent获取浏览器最顶层window对象的引用。

可以使用：window.parent.parent.document最顶层元素的DOM对象
window.parent.parent.document.getElementsByTagName("html")来获取父页面的HTML元素。
```  
###解决方法二：
```javascript
不同域名下：
iframe在跨域访问的时候会有严格的要求，比ajax跨域请求还要难解决
浏览器判断是否跨域会根据两种情况，一个是网页的协议(protocol)，一个就是host是否相同，即，就是url的首部
如：
http: (protocol协议)
www.abcd.com:8080 (host)

使用：document.domain =''
1. 对于这种状况，ifreme在做跨域的时候，可以通过在父页面和iframe子页面同时设置document.domain = 'abcd.com'实现降域。子页面和父页面同时设置才会有效果，才会跨域通信，否则会出错，而且值要相同。这种方法跨域传输数据能够得到解决。
注意：
1.设置document.doamin，也会影响到其它跟iframe有关的功能。
典型的功能如：富文本编辑器（因为是iframe来做富文本编辑器的）、ajax的前进后退（因为IE67要用到iframe，参见：IE6与location.hash和Ajax历史记录）。
2.设置document.doamin，IE678下，有时获取location.href时有异常

document.domain ="" 方法只能解决，二级域相同，使用domain方法降域可以实现，如果完全不相同的域，此方法无效
```
###解决方法三：
完全跨域。一级域和二级域都不相同时，想要传输数据或获取父页面的DOM元素有两种方法，

是使用jsonp方式，用ajax将值传输到父页面的后台服务中，缺点是需要后台服务支持。每次都要调用服 务器中的接口

a.com下的a.html，需要嵌入b.com下的b.html。这时建一个静态页面c.html将c.html放到a.com服务器中。b.html在嵌入c.html.这样，将参数值传输到c.html中，c.html就可以使用window.parent.parent访问a.html所在的顶层window对象，就可以操作父页面的DOM元素。 
例如： 
#####a.html:(所在域：22.22.22.22:2222)

```html
<html>
    <body>
    <iframe id="_top" width="0" height="0" scroll="no" src="http://11.11.11.11:1111/b.html">
    </iframe>
    </body>
</html> 
```

#####b.html:(所在域：11.11.11.11)  

```html
<html>
    <body>
    <iframe id="_top" width="0" height="0" scroll="no" src="">
    </iframe>
    </body>
    <script>
        $("#_top").empty().attrt({src:"http://22.22.22.22:2222/c.html?top='300'"});
    </script>
</html>
```  

####c.html

```html
<html>
    <body>
    </body>
    <script type="text/javascript">
        window.onload = function(){
            var url_param = GetRequest(),
                top_ = "";

            if( url_param.top ){
                top_ = Number(url_param.top);
            }
            if ( top_ && top_ != "" && top_ != NaN ){
                // 通过window.parent.parent获取最顶层window对象。
                // scroolTo改变浏览器的滚动条高度
                window.parent.parent.scrollTo(0,top_);
            }
            // 获取URL中的参数，放入以个对象中放回
            function GetRequest() {
                var url = location.search;
                var theRequest = new Object();
                if (url.indexOf("?") != -1) {
                    var str = url.substr(1);
                    var strs = str.split("&");
                    for(var i = 0; i < strs.length; i ++) {
                        theRequest[strs[i].split("=")[0]]=unescape(strs[i].split("=")[1]);
                    }
                }
                return theRequest;
            }
        };
    </script>
</html>
```