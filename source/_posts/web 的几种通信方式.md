---
title: web 的几种通信方式
date: 2017-08-18 01:11:07
tags:
     - web
    
---

+ 上一篇文章中主要讲述一个完整的HTTP请求过程，了解一个连接是如何建立的。那么，这里再来聊聊web 中常用的几种通信方式，
主要讲述其概念和应用场景或实现方式。概况一下，主要有四种方式，它们分别是短轮询、长轮询(comet)、长连接(SSE)、WebSocket。它们大体可以分为两类，一种是在HTTP基础上实现的，包括短轮询、comet和SSE；另一种不是在HTTP基础上实现是，即WebSocket。

 <!-- more -->

### Ajax 轮询

#### 1、原理
（1）、客户端向服务器端发起请求，和上面最基本的请求原理是一样
(2)、不同的是该请求方式是通过Ajax实现，设定一定的间隔时间对服务器发送请求(比如1 秒)，通过setInterval()方法，实现每隔一段时间向服务器发送请求的功能。
(3)、服务器端返回结果到客户端

#### 2、实现
（1）、前端实现
```
<html>
<head>
<meta charset="utf-8">
<title>短轮询ajax实现</title>
<script type="text/javascript" src="../jquery.min.js"></script>
</head>
<body>
    
        <div id="msgs"></div>
 
</body>
<script type="text/javascript">
    function showUnreadNews(){
            $(document).ready(function() {
//使用jquery的ajax方法
                $.ajax({
                  type: "GET",
                  url: “shortAjax.php",   
                  dataType: "json",
        
                  success: function(msg) {
                      $.each(msg, function(id, title) {
                          $("#msgs").append("<a>" + title + "</a><br>");
                      });
                  }
               });
            });
        }
 
        setInterval('showUnreadNews()',2000);//2秒请求一次
</script>
</html>
```
（2）后端实现
```
<?php
$arr = array('title'=>'标题','text'=>'内容');   //服务器端更新的数据
echo json_encode($arr);  //返回数据
?>
```
#### 3、应用
这种方式由于需要不断的建立http连接，严重浪费了服务器端和客户端的资源。最初是用来实现实时更新数据的， 但服务器的压力和用户数成正比，最终会超过服务器的承受范围，造成严重后果。
因此短轮询不适用于那些同时在线用户数量比较大，并且很注重性能的Web应用。

### comet
#### 1、原理
comet与传统的ajax区别在于，客户端会与服务器端保持一个长连接，此时该连接先挂起，不做响应。只有当客户端需要的数据更新时，服务器才会主动将数据推送给客户端。comet的实现有两种方式，一种方式是使用基于ajax的长轮询，另一种方式是基于Iframe及htmlfile的流方式。基于ajax的长轮询方式中，服务器端在接受到客户端ajax发送的请求后，不立即返回响应，而是阻塞请求直到超时或有数据更新。当服务器端在上述情况下返回响应后，客户端通过js再次发送请求建立连接，重复上述步骤。

#### 2、实现
前端实现
```
<!doctype html>
<html>
<head>
<meta charset="utf-8">
<title>长轮询ajax实现</title>
<script type="text/javascript" src="../jquery.min.js"></script>
</head>
<body>
<input type="button" id="btn" value="click">
<div id="msg">数据情况</div>
</body>
<script type="text/javascript">
$(function(){
        $("#btn").bind('click',{btn:$('#btn')},function(e){
            $.ajax({
                type: 'POST',
                dataType: 'json',               
                url: 'ajax.php',
                timeout: '20000',//设置请求超时时间
                data: {time: '2000000'},// 每次请求等待时间，将其传到后台用来挂起请求
                success: function(data,status){
//对返回的数据进行判断和读取
                    if(data.success == '1'){
                    $("#msg").append('<br>[有数据]'+data.text);
                    e.data.btn.click();  //再此发送请求
                }
                    // 未从服务器中获的数据
                      if(data.success == '0'){
                        $("#msg").append('<br>[无数据]');
                            e.data.btn.click();
                    }
                },
                // ajax超时,继续查询
                error:function(XMLHttpRequest,textStatus,errorThrown){
                     if(textStatus == "timeout"){
                        $("#msg").append('超时');
                        e.data.btn.click();
                     }
                }
          });
        });
    });
</script>
</html>
```
后端 PHP 实现

```
<?php
  set_time_limit(0);// 无限请求超时时间
  usleep($_POST['time’]);//通过前台传过来的数据设置挂起时间
  while(true){   //无限循环
      $rand = rand(1,999);
      if($rand < 500){
          $arr = array('success'=>'1','name'=>'有值','text'=>$rand);
          echo json_encode($arr);
          exit();
      }else{
          $arr = array('success'=>'0','name'=>'无值','text'=>$rand);
          echo json_encode($arr);
          exit();
      }
   }
?>
```

#### 3、应用
长轮询和短轮询比起来，明显减少了很多不必要的http请求次数，相比之下节约了资源。长轮询的缺点在于，连接挂起也会导致资源的浪费。应用场景跟短轮询相差无几。

### SSE
#### 1、概述
SSE是HTML5新增的功能，全称为Server-SentEvents。它可以允许服务推送数据到客户端。SSE在本质上就与之前的长轮询、短轮询不同，虽然都是基于http协议的，但是轮询需要客户端先发送请求。而SSE最大的特点就是不需要客户端发送请求，可以实现只要服务器端数据有更新，就可以马上发送到客户端。

#### 2、原理
SSE不需要依赖客户端向服务器发送请求，而是可以直接在服务器端有数据更新时进行发送到客户端，相比于轮询的“拉数据”，这种“推数据” 有着低延迟、高性能的优势。

这种方法的服务器端非常简介，只要维护一个服务器和客户端之间的协议即可。前端使用EventSource对象。


服务器端需要提供的协议基本代码如下：

data:firstevent

data:secondevent

id:100

event:myevent

data:thirdevent

id:101

:thisisacomment

data:fourthevent

data:fourtheventcontinue


下面解释一下基本用法。要定义各个事件，每一个事件之间使用一个换行符隔开。每个事件内部可以有多行，每一行都是type:value的形式。type有以下集中选择：

(1)类型为空白，表示该行是注释，会在处理时被忽略。

(2)类型为data，表示该行包含的是数据。以data开头的行可以出现多次。所有这些行都是该事件的数据。

(3)类型为event，表示该行用来声明事件的类型。浏览器在收到数据时，会产生对应类型的事件。

(4)类型为id，表示该行用来声明事件的标识符。

(5)类型为retry，表示该行用来声明浏览器在连接断开之后进行再次连接之前的等待时间。

比如上面的第一个事件，只传输了一个数据，数据内容为firstevent。服务器端通过这个清单发送到客户端，就可以通过前端进行响应的处理，诸如读取新数据、更新界面等。

客户端需要在JavaScript中使用EventSource对象。

首先需要初始化一个EventSource对象，实例化的时候需要传入与其交互的服务器端的文件地址，如：

vares=newEventSource(“sse.php”);

接下来，可以对进行事件的监听。EventSource给出了三种标准事件，它们的名称和触发时机如下表：

open 当成功与服务器建立连接时执行

message 当收到服务器发送的事件时执行

error 当出现错误时执行

和普通的事件一样，可以通过以下两种方法使用这些事件：

es.onmessage=function(e){};

es.addEventListener(“message”,function(e){});
 
#### 3、实现
前端实现
```
<html>
    <head>
        <meta charset="UTF-8">
        <title>basic SSE test</title>
    </head>
    <body>
        <div id=”content”></div>
    </body>
    <script>
        var es = new EventSource("sse.php");
        es.addEventListener("message",function(e){
            document.getElementById("content").innerHTML += "\n"+e.data;
        });
    </script>
</html>
```

后端 PHP 实现

```
<?php
header('Content-Type: text/event-stream'); //这是专门为sse设置的数据格式
$time = date('Y-m-d H:i:s');
//下面这些echo出来的东西就是上面说的服务器端和客户端之间的协议
echo 'retry: 3000'.PHP_EOL; //retry类型的数据，规定了浏览器在连接断开之后进行再次连接之前的等待时间
echo 'data: The server time is: '.$time.PHP_EOL.PHP_EOL;
?>


```

#### #### 4、应用 
SSE的优势很明显，它不需要建立或保持大量的客户端发往服务器端的请求，节约了很多资源，提升应用性能。并且SSE的实现非常简单，并且不需要依赖其他插件。


### WEBSOCKET
1、原理
WebSocket一种在单个 TCP 连接上进行全双工通讯的协议。WebSocket通信协议于2011年被IETF定为标准RFC 6455，并被RFC7936所补充规范。WebSocket API也被W3C定为标准。
WebSocket 使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在 WebSocket API 中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。

它的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于服务器推送技术的一种。

Websocket是应用层第七层上的一个应用层协议，它必须依赖 HTTP 协议进行一次握手 ，握手成功后，数据就直接从 TCP 通道传输，与 HTTP 无关了。

Websocket的数据传输是frame形式传输的，比如会将一条消息分为几个frame，按照先后顺序传输出去。这样做会有几个好处：

1 大数据的传输可以分片传输，不用考虑到数据大小导致的长度标志位不足够的情况。

2 和http的chunk一样，可以边生成数据边传递消息，即提高传输效率。

2、实现
客户端实现
```

var ws = new WebSocket("wss://echo.websocket.org");

ws.onopen = function(evt) { 
  console.log("Connection open ..."); 
  ws.send("Hello WebSockets!");
};

ws.onmessage = function(evt) {
  console.log( "Received Message: " + evt.data);
  ws.close();
};

ws.onclose = function(evt) {
  console.log("Connection closed.");
};      
```
说明：
WebSocket提供了四个事件操作，如下：

onmessage 收到服务器响应时执行

onerroe 出现异常时执行

onopen 建立起连接时执行

onclose 断开连接时执行

服务端实现
WebSocket 服务器的实现，可以查看维基百科的[列表](https://en.wikipedia.org/wiki/Comparison_of_WebSocket_implementations)。具体的用法也不一一列举。

3、应用

社交聊天、弹幕、多玩家游戏、协同编辑、股票基金实时报价、体育实况更新、视频会议/聊天、基于位置的应用、在线教育、智能家居等需要高实时的场景。大家感受最深刻的就是微信聊天和 QQ 聊天了。

### 最后：WebSocket和Socket的区别与联系

首先，Socket 其实并不是一个协议。它工作在 OSI 模型会话层（第5层），是为了方便大家直接使用更底层协议（一般是 TCP 或 UDP ）而存在的一个抽象层。Socket是对TCP/IP协议的封装，Socket本身并不是协议，而是一个调用接口(API)。

Socket通常也称作”套接字”，用于描述IP地址和端口，是一个通信链的句柄。网络上的两个程序通过一个双向的通讯连接实现数据的交换，这个双向链路的一端称为一个Socket，一个Socket由一个IP地址和一个端口号唯一确定。应用程序通常通过”套接字”向网络发出请求或者应答网络请求。

Socket在通讯过程中，服务端监听某个端口是否有连接请求，客户端向服务端发送连接请求，服务端收到连接请求向客户端发出接收消息，这样一个连接就建立起来了。客户端和服务端也都可以相互发送消息与对方进行通讯，直到双方连接断开




