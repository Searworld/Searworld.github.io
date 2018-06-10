---
layout: post
title:  "Html5 postMessage 跨域交换数据"
author: "Sear"
tags:  ["跨域"]
---

平时做web开发的时候关于消息传递，除了客户端与服务器传值还有几个经常会遇到的问题

1.页面和其打开的新窗口的数据传递

2.多窗口之间消息传递

3.页面与嵌套的iframe消息传递

4.以上跨域数据传递

HTML5新增API，`postMessage`是用于两个窗口（iframe）之间的数据交换，通信。
语法：`window.postMessage(data,targetOrigin)`

postMessage()方法允许来自不同源的脚本采用异步方式进行有限的通信，可以实现跨文本档、多窗口、跨域消息传递。

>NO .1    页面与嵌套的iframe消息传递

`父页面`文件路经 `domain1/father.html`，启动端口 `http://127.0.0.1:3001`
```
<iframe id="iframe" src="http://127.0.0.1:3002/a.html"></iframe>
<script>
    var iframe = document.getElementById('iframe')
//    窗口加载完
    iframe.onload = function () {
        // 向目标源发送数据
        var data = {}
        iframe.contentWindow.postMessage(JSON.stringify(data), 'http://127.0.0.1:3002')
    }

    // 监听有没有数据发送过来
    window.addEventListener('message', function (e) {
        console.log('data from domain2 --->', e.data)  //打印结果 ：{"name":"Sear","number":16}
    }, false)

</script>
```
`子页面`文件路经 `damain2/son.html`,启动端口 `http://127.0.0.1:3002`
```
 // 监听有没有数据发送过来
    window.addEventListener('message', function(e) {
        var data = {}
        data.name = 'Sear';
        data.number = 16;
        //  // 回发数据
        e.source.postMessage(JSON.stringify(data), 'http://127.0.0.1:3001/father.html');
    }, false);
```
我分别启动两个不同的服务，子页面通过iframe 嵌入父页面，通过postMessage()通信，实现数据交换。父传子，子传父。抛数据 --> 监听数据 的过程


>NO.2  窗口与窗口间消息传递

`窗口1`文件路经 `domain1/tab.html`，启动端口 `http://127.0.0.1:3001`
```
<script type="text/javascript">
    var popup = window.open('http://127.0.0.1:3002/tab.html')
    setTimeout(function () {
        popup.postMessage({name:'Sear'},'http://127.0.0.1:3002')
    },1000)
</script>
```


`窗口2`文件路经 `domain2/tab.html`，启动端口 `http://127.0.0.1:3002`
```
<script type="text/javascript">
    // 设置监听，如果有数据传过来，则打印
    window.addEventListener('message', function(e) {
        console.log(e.data);
    });
</script>
```
这里要设置一个定时器的原因是模拟向`目标窗口发送数据必须等目标窗口完全加载完`，也就是说要在目标窗口中先设置好“监听器”，发送窗口发的数据才能被监听到，所以给了个定时器delay.

>安全性

postMessage采用的是“双向安全机制”。发送方发送数据的时候，会确认接受方的源（所以最好不要用*），而接受方监听到message事件后，也可以用event.origin判断是否来自于正确可靠的发送方。
