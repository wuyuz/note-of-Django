## Django中实现Websocket



​		我们知道，传统的HTTP协议是无状态的，每次请求（request）都要由客户端（如 浏览器）主动发起，服务端进行处理后返回response结果，而服务端很难主动向客户端发送数据；这种客户端是主动方，服务端是被动方的传统Web模式 对于信息变化不频繁的Web应用来说造成的麻烦较小，而对于涉及实时信息的Web应用却带来了很大的不便，如带有即时通信、实时数据、订阅推送等功能的应 用。在WebSocket规范提出之前，开发人员若要实现这些实时性较强的功能，经常会使用折衷的解决方法：轮询（polling）和Comet技术。其实后者本质上也是一种轮询，只不过有所改进。 

  		伴随着HTML5推出的WebSocket，真正实现了Web的实时通信，使B/S模式具备了C/S模式的实时通信能力。WebSocket的工作流程是这 样的：浏览器通过JavaScript向服务端发出建立WebSocket连接的请求，在WebSocket连接建立成功后，客户端和服务端就可以通过 TCP连接传输数据。因为WebSocket连接本质上是TCP连接，不需要每次传输都带上重复的头部数据，所以它的数据传输量比轮询和Comet技术小了很多。



#### 安装dwebsocket模块

```
pip install  dwebsocket
```



#### 使用方法

- 如果你想为一个单独的视图处理一个websocklet连接可以使用accept_websocket装饰器，它会将标准的HTTP请求路由到视图中。使用require_websocke装饰器只允许使用WebSocket连接，会拒绝正常的HTTP请求。

- 在设置中添加设置MIDDLEWARE_CLASSES=dwebsocket.middleware.WebSocketMiddleware这样会拒绝单独的视图实用websocket，必须加上accept_websocket 装饰器。

- 设置WEBSOCKET_ACCEPT_ALL=True可以允许每一个单独的视图实用websockets



#### 一些方法和属性

`1.request.is_websocket()`

​	如果是个websocket请求返回True，如果是个普通的http请求返回False,可以用这个方法区分它们。

`2.request.websocket`

​	在一个websocket请求建立之后，这个请求将会有一个websocket属性，用来给客户端提供一个简单的api通讯，如果`request.is_websocket()是False，这个属性将是None。`

`3.WebSocket.wait()`

​	返回一个客户端发送的信息，在客户端关闭连接之前他不会返回任何值，这种情况下，方法将返回None

`4.WebSocket.read()`

 	如果没有从客户端接收到新的消息，read方法会返回一个新的消息，如果没有，就不返回。这是一个替代wait的非阻塞方法

`5.WebSocket.count_messages()`

 	返回消息队列数量

`6.WebSocket.has_messages()`

 	如果有新消息返回True，否则返回False

`7.WebSocket.send(message)`

​	 向客户端发送消息

`8.WebSocket.__iter__()`

​	 websocket迭代器



#### 简单代码演示

在templates目录下创建一个index.html文件

```html
<!DOCTYPE html>
<html>
<head>
    <title>django-websocket</title>
    <script src="http://code.jquery.com/jquery-1.11.1.min.js"></script>
    <script type="text/javascript">
    $(function () {
        $('#connect_websocket').click(function () {
            if (window.s) {
                window.s.close()
            }
            /*创建socket连接*/
            var socket = new WebSocket("ws://" + window.location.host + "/serialize/echo/");
            socket.onopen = function () {
                console.log('WebSocket open');//成功连接上Websocket
            };
            socket.onmessage = function (e) {
                console.log('message: ' + e.data);//打印出服务端返回过来的数据
                $('#messagecontainer').prepend('<p>' + e.data + '</p>');
            };
            // Call onopen directly if socket is already open
            if (socket.readyState == WebSocket.OPEN) socket.onopen();
            window.s = socket;
        });
        $('#send_message').click(function () {
            //如果未连接到websocket
            if (!window.s) {
                alert("websocket未连接.");
            } else {
                window.s.send($('#message').val());//通过websocket发送数据
            }
        });
        $('#close_websocket').click(function () {
            if (window.s) {
                window.s.close();//关闭websocket
                console.log('websocket已关闭');
            }
        });
    });
</script>
</head>
<body>
<br>
<input type="text" id="message" value="Hello, World!"/>
<button type="button" id="connect_websocket">连接 websocket</button>
<button type="button" id="send_message">发送 message</button>
<button type="button" id="close_websocket">关闭 websocket</button>
<h1>Received Messages</h1>
<div id="messagecontainer">

</div>
</body>
</html>
```

views.py文件

```python
from django.shortcuts import render
from dwebsocket.decorators import accept_websocket,require_websocket
from django.http import HttpResponse

@accept_websocket
def echo(request):
    if not request.is_websocket():#判断是不是websocket连接
        try:#如果是普通的http方法
            message = request.GET['message']
            return HttpResponse(message)
        except:
            return render(request,'index.html')
    else:
        for message in request.websocket:
            request.websocket.send(message)#发送消息到客户端
```

urls.py文件

```python
from demo import views as v

urlpatterns = [
    url(r'^index2/', v.index2),
    url(r'^echo$', v.echo),
]
```

![1571652276957](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1571652276957.png)