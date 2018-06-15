---
layout: post
title:  "封装jsonp+promise实现跨域「 音乐接口」"
author: "Sear"
tags:  ["js"]
---

>jsonp原理

通过动态创建<script>标签，src指向数据地址，其中callback参数作为函数名来包裹住JSON数据返回客户端。也就是说，只要服务端提供的js脚本是动态生成的就行了，这样调用者可以`传一个参数过去告诉服务端“我想要一段调用XXX函数的js代码，请你返回给我”`，于是服务器就可以按照客户端的需求来生成js脚本并响应了。

网上有很多原生请求或者jqery.ajax 的请求方式，这里就不多说，直接上代码

>步骤一、封装jsonp

[github上封装的jsonp](https://github.com/webmodules/jsonp)
1.安装
2.使用方法：
```
jsonp(url, opts, fn)
----------------------------------------------------------------------------------------------------
url : 请求地址
opts: callback名默认是callback、请求超期时间、callback 参数名，默认  callback = _jp
fn : 请求结果
```

> 步骤二、 进一步封装jsonp + promise


文件目录，`js/jsonp.js`文件

```
import originJsonp from 'jsonp' // 导入 安装的 jsonp 模块，命名为 originJsonp
//导出这个方法
export default function jsonp(url, data, option) {
  url += (url.indexOf('?') < 0 ? '?' : '&') + param(data)  //拼接URL

  return new Promise((resolve, reject) => {  //promise了解一下，两个参数resolve,reject
    originJsonp(url, option, (err, data) => { //步骤一的使用方式，jsonp(url, opts, fn)
      if (!err) {
        resolve(data)
      } else {
        reject(err)
      }
    })
  })
}

//处理上面的data数据
export function param(data) {
  let url = ''
  for (let k in data) {
    let value = data[k] !== undefined ? data[k] : ''
    url += '&' + k + '=' + encodeURIComponent(value)
  }
  return url ? url.substring(1) : ''
}

```
>步骤三、如何使用

```
import jsonp from '../js/jsonp'  //导入上面封装的jsonp+promise  方法

 function getSliderList () {
  const url = 'xxxxx'
  const data = Object.assign({}, {
    ..... 参数
  })
  return jsonp(url, data, options)
}

//调用
getSliderList ().then((res)=>{
  console.log(res)
})
```

>例子：调取QQ音乐接口

[github文件地址](https://github.com/Searworld/vue-App/tree/feature-1.3)
文件不止一个，可以找到带有` 封装jsonp+promise跨域请求数据` 查看请求





小白：哇，这么牛，这么说，我可以用这种方式跨域请求数据咯

大菜：哈哈，当然不是。一方面呢， 这种方式只能是GET 请求，另一方面，[web安全:CSRF攻击原理以及防御](https://www.jianshu.com/p/ffb99fc70646)  了解一下，有些呢，服务端为了安全，做了限制，需要 `验证 HTTP Referer 字段`,也就是你的请求来源并不能通过  Referer 验证，请求不合法，并不能拿到数据

小白：啊？`怎么辨别`是这种情况呢？还有在这种情况下`怎么请求`数据呢？

大菜：别急别急，欲知详情，请看下集分解。
