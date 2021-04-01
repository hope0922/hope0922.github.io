---
layout: post
title: React stopPropagation 无效
category: blog
description: React stopPropagation 无效
---

写代码的过程中，发现在子元素用 stopPropagation 阻止绑定在 document 上的事件的时候，是没办法做到的，只可以阻止父级绑定的事件的触发。

关于这个问题的解释，网上五花八门，有些将其归结为是由于事件委托的原因，我自己尝试了之后发现，其实原生 document 绑定的事件是最后才执行的

```javascript
class ExampleApplication extends React.Component {
  componentDidMount() {
    document.addEventListener("click", () => {
      alert("document click");
    });
  }

  outClick(e) {
    console.log(e.currentTarget);
    alert("outClick");
  }

  onClick(e) {
    console.log(e.currentTarget);
    alert("onClick");
    e.stopPropagation();
  }
  render() {
    return (
      <div onClick={this.outClick}>
        <button onClick={this.onClick}> 测试click事件 </button>
      </div>
    );
  }
}
```

其实从上面例子的输出： 由'onClick',再到'document click'可知，其实原生 document 上绑定的事件时最后执行的，所以并不是因为 document 收到事件快慢的原因而导致这个问题。

最后查阅资料后发现 **出现上述 bug 的主要原因是混用浏览器原生事件跟 React 合成事件**

#### React 有自己的一套事件处理机制，它会将所有的事件都绑定在 document 上，然后再用 dispatchEvent 统一分发，这时候分发的是合成事件。而 onClick(e)这时候拿到的 e 其实是合成事件，只能阻止合成事件的冒泡。

```javascript
class ExampleApplication extends React.Component {
  componentDidMount() {
    document.addEventListener("click", () => {
      alert("document click");
    });
    document.getElementById("div1").addEventListener("click", () => {
      alert("原生outClick");
    });
  }

  outClick(e) {
    console.log(e.currentTarget);
    alert("合成outClick");
  }

  onClick(e) {
    console.log(e.currentTarget);
    alert("onClick");
    e.stopPropagation();
  }
  render() {
    return (
      <div id="div1" onClick={this.outClick}>
        <button onClick={this.onClick}> 测试click事件 </button>
      </div>
    );
  }
}
```

做了点小改动，就是外层的div绑定的函数用原生的方式跟jsx的方式都绑定一次，所以最终的输出为
#### '原生outClick', 'onClick','document click'
## 解决方法 e.nativeEvent.stopImmediatePropagation(); 使绑定在document的其他事件不再触发