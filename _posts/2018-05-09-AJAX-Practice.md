---
layout: post
title:  "AJAX基本知识"
date:   2018-05-09 08:10:30 +0200
categories: web js
---

ajax(asynchronous javascript and xml)是一种用来改善用户体验的技术


## ajax简介
ajax本质是利用浏览器提供的一个特殊对象(XMLHttpRequest)向服务器发送异步请求，服务器返回部分数据，浏览器利用这些数据对当前页面做局部更新。整个过程页面不刷新，不打断用户的操作，ajax对象发送请求时，浏览器不会销毁当前页面，用户可以对当前页面做其他操作。
## ajax使用
### 获取ajax对象
 <img src= "/assets/files/ajax.jpg" alt="加载错误" title="ajax"/>
 
```js
function getAjax(){
var xhr=null;
if(window.XMLHttpRequest){
//非ie浏览器
xhr=new XMLHttpRequest();
}else{
//ie浏览器
xhr=new ActiveXObject('MicroSoft.XMLHttp');
}
return xhr;
}
```
### ajax对象的属性
1. onreadystatechange：绑定一个事件处理函数
2. readystatechange事件：当ajax对象的readyState属性值发送了任何的改变，就会产生该事件。
3. readyState：有五个值(0，1，2，3，4)，用来获得ajax对象与服务器通信的进展状况。其中，4表示ajax对象已经获得了服务器返回的所有据。												                
4. responseText：获得服务器返回的文本数据
5. responseXML：获得服务器返回的xml数据
6. status：获得服务器返回的状态码

### 编程步骤
1. 获得ajax对象: `var xhr=getAjax();`
2. 利用ajax对象发送请求
* 发送get请求：
```js
    //true表示异步,false表示同步(ajax对象发送请求时,浏览器会锁定当前页面,用户不能操作)
    xhr.open(‘get’，'check.do?username=DMETAL&password='，true);
    xhr.onreadystatechange=f1;
    xhr.send(nul);
```
* 发送post请求:
```js
    xhr.open('post','check.do',true);
    //按照http协议要求，post请求应该在请求数据包里面添加content-type消息头
    //ajax对象默认不会添加该消息头，所以需调用setRequestHeader方法来添加
    xhr.setRequestHeader('content-type','application/x-www-form-urlencoded');
    xhr.send('username=DMETAL');
```
* 编写服务器端的程序,服务器端通常只需要返回部分数据
* 编写事件处理函数
```js
    if(xhr.readyState==4&&xhr.status==200){
    var txt=xhr.responseText;
    //更新页面.....
    }
```

### 缓存问题
1. ie浏览器提供的ajax对象在发送get请求时，会比较请求地址是否访问过，如果访问过，则不再发送新的请求，而是显示第一次返回的结果。
    * 解决方案：①发送post请求；②在请求地址后添加一个随机数

### 编码问题
1. 发送get请求时
* 产生乱码原因：ie浏览器会使用gbk编码，其他浏览器会使用utf-8编码，服务器端默认会使用iso-8859-1解码
* 解决方案：服务器端，统一使用URIEncoding=”uft-8“；只针对get请求有效客户端，使用encodeURI函数对中文参数值进行编码；此函数是js内置函数，会使用utf-8对中文进行编码
2. 发送post请求时
* 产生乱码原因：浏览器会使用utf-8来编码，而服务器端会使用iso-8859-1来解码
* 解决方案：`request.setCharacterEncoding("utf-8");`

### JSON(JavaScript object notation)
JSON是一种轻量级的数据交换格式
* 数据交换：将要交换的数据先转换成一种与平台无关的数据格式(如xml)，然后发送给接收方来处理；
* 轻量级：JSON相对于xml，文档更小，解析速度更快

#### 语法
1. 表示一个对象：{属性名：属性值，属性名：属性值....}  {“name”:"dmetal","age":21}
2. 表示对象组成的数组[{},{},{},{}...]

#### 应用场景
1. java对象转换成json字符串：可以使用jackson api （ObjectMapper）
2. json字符串转换成JavaScript对象：可以使用JavaScript内置对象JSON提供的parse方法

## JQuery对ajax编程的支持
1. `$.ajax({});  $.post(url,function(){});  $.getJSON(url,function(){});`
* {}是一个对象，用来控制ajax对象如何向服务器发送请求，常用的选项参数如下：
    * url 指定请求地址("quoto.do");
    * type 指定请求类型 ("get"/"post");
    * data 指定请求参数,有两种格式:("name=dmetal&age=21")或({“name”:"dmetal","age":21})
    * dataType 指定服务器返回的数据类型,包括json;text;html;xml;js
    * success 指定一个函数，用来处理服务器返回的数据;(服务器处理正常，并且ajax对象已经获得了服务器返回的所有数据)
    * error 指定一个函数，用来处理服务器返回的数据(服务器发生异常)
    * async 同步(false)/异步(true)
2. load方法
* 向服务器发送异步请求，并将服务器返回的数据直接添加到符合要求的节点上
* 用法:`$obj.load(url,[data]);`
    * $obj:要操作的节点，是一个JQuery对象
    * url 请求地址
    * data(可选) 请求参数，也有两种格式
