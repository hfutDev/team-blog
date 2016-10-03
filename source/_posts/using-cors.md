---
title: Using CORS (使用跨域资源共享)
tags: CORS
categories: trans
---

---

```js
原文链接： http://www.html5rocks.com/en/tutorials/cors/
作者：Monsur Hossain
```

## 简介
API 是让你能组装一个 web 富应用的脚手架工具，但这些操作浏览器有时候也难以理解，当你跨域去执行一个请求的时候，这些操作技术就仅限于 `JSON-P`(但出于安全方面的考虑，有时候使用也受限)或者建立一个传统的代理(但建设和维护成本高)。

**跨域资源共享 Cross-Origin Resource Sharing** [`CORS`] 是 W3C 制定的一个标准，允许通过浏览器进行跨域通信。 CORS 建立在 XMLHttpRequest 对象之上，允许开发者用操作同域请求的方式处理跨域请求。

跨域请求的实例随手可见，想象一下，网站 bob.com 想请求 alice.com 网站上的一些数据，在 `同源策略`下，传统的的方式是不允许进行跨域请求的。但如果， alice.com 支持 CORS 请求, alice.com 可以在请求头中添加一些特殊的响应来允许 bob.com 有能力拿到数据。

从这个例子中也能看出， 实现 CORS 需要服务器和客户端同时协调支持。幸运的是，如果你是个客户端的开发者，大多数的技术 实现细节你都可以屏蔽掉。本文的剩余部分将介绍客户端如何发起一个 CORS 请求，以及如何配置服务器端以支持和响应 CORS 

## 发起一个 CORS 请求
这部分将介绍如何通过 JavaScript 发起一个 CORS 请求。
### 创建一个 XMLHtttpRequest 对象
支持 CORS 的浏览器
> * Chrome 3+
> * Firefox 3.5+
> * Opera 12+
> * Safari 4+
> * IE 8+

查看完整的浏览器支持情况 [http://caniuse.com/#search=cors](http://caniuse.com/#search=cors)

Chrome, Firefox, Opera 和 Safari 全部使用 XMLHttpRequst2 对象，IE 使用类似的 XDomainRequest 对象，大多数时候和 XMLHttpRequest 对象以同样的方式起作用，但是增加了额外的安全限制。

开始之前，我们需要创建一个正确的请求对象 Nicholas Zakas 写了一个简单的函数，帮助我们识别不同的浏览器

```js
function createCORSRequest(method, url) {
    var xhr = new XMLHttpRequrest();
    if("withCredentials" in xhr) {
        //检查 XMLHttpRequest 对象是否包含 "withCredentials" 属性
        // "withCredentials" 仅仅在 XMLHttpRequest2 对象中存在。
        xhr.open(method, url, true);
    } else if (typeof XDomainRequest != "undefined") {
        //否则检查是否存在XDomainRequest
        //XDomainRequest 仅仅存在于 IE 中，IE 用 XDomainRequest 对象发起CORS 请求。
        xhr = new XDomainRequest();
        xhr.open(method, url);
    } else {
        //否则，浏览器不支持 CORS
        xhr = null;
    }
    return xhr;
}

var xhr = createCORSRequest('GET', url);
if(!xhr) {
    throw new Error('CORS not supported');
}
```
### 事件处理函数
原始的 XMLHttpRequest 对象仅有一个事件处理函数 **onreadystatechange**, 处理所有的响应，尽管 onreadystatechange 仍然有效，XMLHttpRequest2引进了一系列新的事件处理函数，以下是一个完整列表：

| 事件处理函数 | 描述 |
| :--------: | :--------: |
| onloadstart | 请求开始的时候|
| onprogress | 正在载入和发送数据的时候 |
| onabort | 当请求废弃的时候， 比如，抛出 abort() 方法 |
| onerror | 当请求失败的时候 |
| onload | 当请求成功完成的时候 |
| ontimeout | 在设定的溢出时间之前没有完成请求的时候 |
| onloadend | 当请求完成的时候，不管是成功还是失败 |

注：IE 的XDomainRequest 对象不支持 **onloadstart** 和 **onloadend** 方法。
[点击访问详情](http://www.w3.org/TR/XMLHttpRequest2/#events)

大多数情况下，你是很少会去处理 **onload** 和 **onerror** 事件。

```js
xhr.onload = function() {
    var responseText = xhr.responseText;
    console.log(responseText);
};

xhr.onerror = function() {
    console.log('There was an error!')
};
```
当报错之后，浏览器不能具体的告诉你那里出了问题。比如 firefox 对所有的错误和空状态都返回一个 0 。浏览器也会在控制台的日志里面报告一个错误信息，但这个信息通过 js 是拿不到的。所以当我们处理 onerror 时， 你知道发生了错误，其他一无所知。

### withCredentials 
标准的 CROS 默认情况下不会发送和设置任何的 cookies 。如果想在请求中包含 cookies，需要将 XMLHttpRequest 对象的 withCredentials 属性设置为 true 。
```js
xhr.withCredentials = true;
```
同时，服务端需要将响应头中的 Access-Control-Allow-Credentials 设置为 true.
```js
Access-Control-Allow-Credentials: true
```
`withCredentials` 属性将把远端域中的 cookies 包含在每次请求中，同时也设置来自远端的任何一个 cookies 。注意这些 cookies 仍然遵守同源的规则，所以你的代码不能从 document.cookies 或者响应头中拿到他们。他们仅受远端域的控制。

### 发起请求
现在你的 CORS 请求配置已完成。已经 准备好发起请求了。通过调用 send() 方法实现。
```js
xhr.send();
```
到这里，假设服务器已经正确的配置好，并且可以正确的响应 CORS 请求，就像你所熟悉的同源 XHR 请求一样。

### 完整演示
这是一个完整的 CORS 请求样例，在浏览器的控制台的网络请求中查看真实请求的实现过程。
```js
// 创建 XHR 对象
function createCORSRequest(method, url) {
    var xhr = new XMLHttpRequest();
    if ("withCredentials" in xhr) {
        xhr.open(method, url, true);
  } else if (typeof XDomainRequest != "undefined") {
        xhr = new XDomainRequest();
        xhr.open(method, url);
  } else {
        xhr = null;
  }
    return xhr;
}
function getTitle(text) {
  return text.match('<title>(.*)?</title>')[1];
}

//发起请求
function makeCorsRequest() {
    var url = 'http://html5rocks-cors.s3-website-us-east-1.amazonaws.com/index.html';

    var xhr = createCORSRequest('GET', url);
    if (!xhr) {
        alert('CORS not supported');
        return;
  }
  
//处理响应
xhr.onload = function() {
    var text = xhr.responseText;
    var title = getTitle(text);
    alert('Response from CORS request to ' + url + ': ' + title);
  };
  xhr.onerror = function() {
    alert('Woops, there was an error making the request.');
  };
  xhr.send();
}
```
## 服务器端添加 CORS 支持

对 CORS 来说，多数比较重的操作在于要在客户端和服务器之间同时操作。当在一个客户端的 CORS 中，浏览器会添加一些常规的请求头，有时候添加一些常规的请求。 这些常规(默认)的请求对客户端来说是隐藏的(但是可以通过一些包分析工具查看他们，比如 [Wireshark](https://www.wireshark.org/))

**CORS flow**

![cors-flow](http://7xrzdf.com1.z0.glb.clouddn.com/cors_flow.png)

浏览器厂商负责浏览器端的接口。这部分释解一个服务器如何配置它的头信息以支持 CORS。

### CORS 的请求类型
跨源请求有两种类型：
1. 简单请求
2. 非简单请求

满足以下标准的请求是简单请求：
```
HTTP 方法是以下之一:
    1. HEAD
    2. GET
    3. POST
HTTP 首部是以下之一：
    1. Accept
    2. Accept-Language
    3. Content-Language
    4. Last-Event-ID
    5. Content-Type, 但值只能是以下三者之一：
        > application/x-www-form-urlencoded
        > multipart/form-data
        > text/plain
```
以这样的特征定义简单请求，是因为浏览器在没有 CORS 的情况下，已经自带了这些请求的功能。例如使用 JSONP 可以解决跨域的 GET 请求，HTML可以用来发起表单 POST。 

任何不满足以上标准的，是 非简单请求 。在浏览器和服务器之间需要一些额外的通信（称为先验请求），后面我们将介绍。

### 处理一个简单请求

让我们从检测一个来自客户端的简单请求开始。JavaScript 代码发起一个简单的 GET 请求，相应的浏览器发起的真实 HTTP 请求; 其中 CORS 特有的首部以粗体标出。

JavaScript:
```js
var url = 'http://api.alice.com/cors';
var xhr = createCORSRequest('GET', url);
xhr.send();
```
HTTP Request:
> GET /cors HTTP/1.1
> **Origin: http://api.bob.com**
> Host: api.alice.com
> Accept-Language: en-US
> Connection: keep-alive
> User-Agent: Mozilla/5.0...

首先要注意，一个有效的 CORS 请求**经常**包含一个原始首部(**Origin header**),这个原始首部是由浏览器添加的，用户是控制不了的。这个首部的值包括：协议，域名和端口（如果不是默认端口不能省略）。说明请求是从哪里发起的。

当出现不包含原始首部的情况时，说明是一个同源请求，所有的跨域请求都会包含一个原始首部。一些同源请求也会包含原始首部，例如 FireFox 同源请求中不包含原始首部，Chrome， Safair 在同源请求中(限于 POST/PUT/DELETE 方法)中包含原始首部。以下是一个包含原始首部的同源请求。

HTTP Request:
```
POST / cors HTTP/1.1
Origin: http://api.bob.com
Host: api.bob.com
```
好消息是，浏览器不希望 CORS 响应首部出现在同源请求中。同源请求的响应是发送给用户的。不管他是不是 CORS 首部，然而，你的服务端代码返回一个源端不再匹配列表中的错误，确保包含发起请求的域（在匹配列表中）。

以下是一个有效的服务端响应，CORS 特有的首部以粗体标出。

HTTP Request
> **Access-Control-Allow-Origin: http://api.bob.com**
> **Access-Control-Allow-credentials: true**
> **Access-Control-Expose-Headers: FooBar**
> Content-Type: text/html; charset=utf-8

所有和 CORS 相关的首部都有 "Access-Control-" 前缀，以下列出每个首部的详情

**Access-Control-Allow-Origin**( required ) 这个首部必须包含在所有有效的 CORS 响应中。省略这个头部，会导致 CORS 请求失败。首部的值可以是原始请求的返回值(向上面就是 http://api.bob.com ),也可以为 × ，允许来自所有域的请求。如果你对请求域不做限制，任何一个源都可以请求。如果想要控制控制可请求的源，首部中填入真实的值。

**Access-Control-Allow-Credentials** (可选)， 默认情况下，cookies 是不包含在首部中的，使用这个首部设置 cookies 是否要包含在 CORS， 请求中。如果需要设置值为 true ,如果不需要设置为 false 。

Access-Control-Allow-Credentials 首部和 XMLHttpRequest2 对象上的 withCredentials 属性一起使用，两者都要设置为 true, 才能保证正确的 CORS 请求。

**Access-Control-Expose-Headers** (可选)，XHR2 对象有一个 getResponseHeader() 方法，又来返回特殊首部的值。 在一个 CORS 请求中， getResponseHeader() 方法仅仅能拿到简单响应的的首部，简单响应首部定义为以下几种：
> * Cache-Control
> * Content-Language
> * Content-Type
> * Expires
> * Last-Modified
> * Pragma

如果想要拿到其他的首部，就用 **Access-Control-Expose-Headers** 首部，他的值是用逗号隔开的一组你想暴露给客户端的响应头。

### 处理一个非简单请求

如果你关注的不仅仅是简单的 GET 请求，还想做更多的事情。可能你想支持更多的 HTTP 方法，像 PUT 或者 DELETE, 或者想支持 JSON 类型，使用头部： Content-Type: application/json. 此时你就需要处理非简单请求啦。

对客户端来说，简单请求和非简单请求很相似。事实上，非简单由两个请求组成。浏览器会先发送一个预验请求( **preflight request** )。预验请求询问服务器是否允许客户端发起真实的请求。一旦被确认允许，浏览器就发起真实请求。浏览器透明的处理这两个请求。预验响应( **preflight response** )同样可以缓存在客户端，这样就不用为每一个请求发起响应了。

下面是一个 非简单请求 的实例
```js
var url = 'http://api.alice.com/cors',
    xhr = creteCORSRequest('PUT', url);
xhr.setRequestHeader('X-Custom-Header', 'value');
xhr.send();
```
Preflight Request:
>OPTIONS /cors HTTP/1.1
**Origin: http://api.bob.com**
**Access-Control-Request-Method: PUT**
**Access-Control-Request-Headers: X-Custom-Header**
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...

和简单请求一样，浏览器为每个请求都加上原始首部，包括预验请求。预验请求发起的是一次 HTTP OPTIONS 请求，所以确保你的服务器能响应处理这个方法。同时也包含其他一些首部。

**Access-Control-Request-Method**  value 为真实请求的的 HTTP 方法，预验请求中经常包括这个首部，即便是我们之前定义的简单 HTTP 方法（GET，POST, HEAD）中也包括。

**Access-Control-Request-Headers**: 包含在请求中由分号隔开的一列非简单首部。

如果，HTTP方法和首部是有效的，服务器应该像下面这样进行响应。

Preflight Request:
>OPTIONS /cors HTTP/1.1
**Origin: http://api.bob.com**
**Access-Control-Request-Method: PUT**
**Access-Control-Request-Headers: X-Custom-Header**
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...

Preflight Response:
>**Access-Control-Allow-Origin: http://api.bob.com**
**Access-Control-Allow-Methods: GET, POST, PUT**
**Access-Control-Allow-Headers: X-Custom-Header**
Content-Type: text/html; charset=utf-8

Access-Control-Allow-Origin (必须)
Access-Control-Allow-Methods (必须)
Access-Control-Allow-Headers (如果预验请求中也包含这个首部，则必须) 
Access-Control-Allow-Credentials (可选)
Access-Control-Max-Age (可选)

预验请求结束并得到确认后，浏览器发起真实请求。真实请求和简单请求很相似。

如果服务器想拒绝 CORS 请求，只需要返回一个没有 CORS 首部的通用响应。只要预验响应中没有 CORS特有的首部，浏览器判定(预验)请求是无效的，不会发起真实请求。

Preflight Request:

>OPTIONS /cors HTTP/1.1
Origin: http://api.bob.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...

Preflight Response:
> // ERROR - No CORS headers, this is an invalid request!
Content-Type: text/html; charset=utf-8

如果 CORS 请求中有错，浏览器会触发 onerror 事件处理函数，在控制台打印出：
> XMLHttpRequest cannot load http://api.alice.com. Origin http://api.bob.com is not allowed by Access-Control-Allow-Origin.

浏览器不会告诉你更多的细节，只告诉你出错了，但不会告诉你那里出错了。

### 简单了解安全性
CORS 能实现跨域请求，但 CORS 首部并不就是安全操作的替代品，不应该依赖 CORS 首部去保障你站点的安全。使用 CORS 首部为浏览器提供跨域请求的能力。同时使用其他一些安全机制对你的网站内容进行安全限制，像 cookies 或者 OAouth2.

## JQuey  实现 CORS
jquery中的 $.ajax() 方法可以同时发起 XHR 和 CORS  请求。但

> jquery CORS 接口不支持 IE 的 XDomainRequest 对象
> $.support.cors 返回 true 如果浏览器支持 CORS

下面是有 JQuery 发起一个 CORS 请求
```js
$.ajax({

  // The 'type' property sets the HTTP method.
  // A value of 'PUT' or 'DELETE' will trigger a preflight request.
  type: 'GET',

  // The URL to make the request to.
  url: 'http://html5rocks-cors.s3-website-us-east-1.amazonaws.com/index.html',

  // The 'contentType' property sets the 'Content-Type' header.
  // The JQuery default for this property is
  // 'application/x-www-form-urlencoded; charset=UTF-8', which does not trigger
  // a preflight. If you set this value to anything other than
  // application/x-www-form-urlencoded, multipart/form-data, or text/plain,
  // you will trigger a preflight request.
  contentType: 'text/plain',

  xhrFields: {
    // The 'xhrFields' property sets additional fields on the XMLHttpRequest.
    // This can be used to set the 'withCredentials' property.
    // Set the value to 'true' if you'd like to pass cookies to the server.
    // If this is enabled, your server must respond with the header
    // 'Access-Control-Allow-Credentials: true'.
    withCredentials: false
  },

  headers: {
    // Set any custom headers here.
    // If you set any non-simple headers, your server must include these
    // headers in the 'Access-Control-Allow-Headers' response header.
  },

  success: function() {
    // Here's where you handle a successful response.
  },

  error: function() {
    // Here's where you handle an error response.
    // Note that if the error was due to a CORS issue,
    // this function will still fire, but there won't be any additional
    // information about the error.
  }
});
```

（完）
