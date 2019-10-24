## Ajax全套



对于WEB应用程序：用户浏览器发送请求，服务器接收并处理请求，然后返回结果，往往返回就是字符串（HTML），浏览器将字符串（HTML）渲染并显示浏览器上。

1、传统的Web应用

> 一个简单操作需要重新加载全局数据

2、AJAX

> AJAX，Asynchronous JavaScript and XML (异步的JavaScript和XML)，一种创建交互式网页应用的网页开发技术方案。
>
> - 异步的JavaScript：
>   使用 【JavaScript语言】 以及 相关【浏览器提供类库】 的功能向服务端发送请求，当服务端处理完请求之后，【自动执行某个JavaScript的回调函数】。
>   PS：以上请求和响应的整个过程是【偷偷】进行的，页面上无任何感知。
> - XML
>   XML是一种标记语言，是Ajax在和后台交互时传输数据的格式之一
>
> 利用AJAX可以做：
> 1、注册时，输入用户名自动检测用户是否已经存在。
> 2、登陆时，提示用户名密码错误
> 3、删除数据行时，将行ID发送到后台，后台在数据库中删除，数据库删除成功后，在页面DOM中将数据行也删除。（博客园）



#### 伪AJax

 由于HTML标签的iframe标签具有局部加载内容的特性，所以可以使用其来伪造Ajax请求。 

```js
<!DOCTYPE html>
<html>
    <head lang="en">
        <meta charset="UTF-8">
        <title></title>
    </head>
 
    <body>
 
        <div>
            <p>请输入要加载的地址：<span id="currentTime"></span></p>
            <p>
                <input id="url" type="text" />
                <input type="button" value="刷新" onclick="LoadPage();">
            </p>
        </div>
 
        <div>
            <h3>加载页面位置：</h3>
            <iframe id="iframePosition" style="width: 100%;height: 500px;"></iframe>
        </div>
 
 
        <script type="text/javascript">
            window.onload= function(){
                var myDate = new Date();
                document.getElementById('currentTime').innerText = myDate.getTime();
 
            };
            function LoadPage(){
                var targetUrl =  document.getElementById('url').value;
                document.getElementById("iframePosition").src = targetUrl;
            }
        </script>
    </body>
</html>
```



#### 原生AJAX

Ajax主要就是使用 【XmlHttpRequest】对象来完成请求的操作，该对象在主流浏览器中均存在(除早起的IE)，Ajax首次出现IE5.5中存在（ActiveX控件）。



#### 跨域AJAX

由于浏览器存在同源策略机制，同源策略阻止从一个源加载的文档或脚本获取或设置另一个源加载的文档的属性。

**特别的**：由于同源策略是浏览器的限制，所以请求的发送和响应是可以进行，只不过浏览器不接受罢了。

浏览器同源策略并不是对所有的请求均制约：

- 制约： XmlHttpRequest
- 不叼： img、iframe、script等具有src属性的标签

*跨域，跨域名访问，如：http://www.c1.com 域名向 http://www.c2.com域名发送请求。*

**1、JSONP实现跨域请求**

JSONP（JSONP - JSON with Padding是JSON的一种“使用模式”)，利用script标签的src属性（浏览器允许script标签跨域）

```js
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>

    <p>
        <input type="button" onclick="Jsonp1();"  value='提交'/>
    </p>

    <p>
        <input type="button" onclick="Jsonp2();" value='提交'/>
    </p>

    <script type="text/javascript" src="jquery-1.12.4.js"></script>
    <script>
        function Jsonp1(){
            var tag = document.createElement('script');
            tag.src = "http://c2.com:8000/test/";
            document.head.appendChild(tag);
            document.head.removeChild(tag);
        }

        function Jsonp2(){
            $.ajax({
                url: "http://c2.com:8000/test/",
                type: 'GET',
                dataType: 'JSONP',
                success: function(data, statusText, xmlHttpRequest){
                    console.log(data);
                }
            })
        }
    </script>
</body>
</html>
```



**2、CORS**

随着技术的发展，现在的浏览器可以支持主动设置从而允许跨域请求，即：跨域资源共享（CORS，Cross-Origin Resource Sharing），其本质是设置响应头，使得浏览器允许跨域请求。

-  简单请求 OR 非简单请求 

  ```
  条件：
      1、请求方式：HEAD、GET、POST
      2、请求头信息：
          Accept
          Accept-Language
          Content-Language
          Last-Event-ID
          Content-Type 对应的值是以下三个中的任意一个
                                  application/x-www-form-urlencoded
                                  multipart/form-data
                                  text/plain
   
  注意：同时满足以上两个条件时，则是简单请求，否则为复杂请求
  
  简单请求和非简单请求的区别？
  	简单请求：一次请求
  	非简单请求：两次请求，在发送数据之前会先发一次请求用于做“预检”，只有“预检”通过后才再发送一次请求用于数据传输。
  ```

- 关于预检

  ```
  请求方式：OPTIONS
    “预检”其实做检查，检查如果通过则允许传输数据，检查不通过则不再发送真正想要发送的消息
  
  如何“预检”:
      => 如果复杂请求是PUT等请求，则服务端需要设置允许某请求，否则“预检”不通过
          Access-Control-Request-Method
      => 如果复杂请求设置了请求头，则服务端需要设置允许某请求头，否则“预检”不通过
          Access-Control-Request-Headers
  ```

  基于cors实现AJAX请求:

  a、支持跨域，简单请求

  服务器设置响应头：Access-Control-Allow-Origin = '域名' 或 '*'

  ```js
  <!DOCTYPE html>
  <html>
  <head lang="en">
      <meta charset="UTF-8">
      <title></title>
  </head>
  <body>
  
      <p>
          <input type="submit" onclick="XmlSendRequest();" />
      </p>
  
      <p>
          <input type="submit" onclick="JqSendRequest();" />
      </p>
  
      <script type="text/javascript" src="jquery-1.12.4.js"></script>
      <script>
          function XmlSendRequest(){
              var xhr = new XMLHttpRequest();
              xhr.onreadystatechange = function(){
                  if(xhr.readyState == 4) {
                      var result = xhr.responseText;
                      console.log(result);
                  }
              };
              xhr.open('GET', "http://c2.com:8000/test/", true);
              xhr.send();
          }
  
          function JqSendRequest(){
              $.ajax({
                  url: "http://c2.com:8000/test/",
                  type: 'GET',
                  dataType: 'text',
                  success: function(data, statusText, xmlHttpRequest){
                      console.log(data);
                  }
              })
          }
      </script>
  </body>
  </html>
  ```

  Tornado代码

  ```
  class MainHandler(tornado.web.RequestHandler):
      def get(self):
          self.set_header('Access-Control-Allow-Origin', "http://www.xxx.com")
          self.write('{"status": true, "data": "seven"}')
  ```

  b、支持跨域，复杂请求

  由于复杂请求时，首先会发送“预检”请求，如果“预检”成功，则发送真实数据。

  - “预检”请求时，允许请求方式则需服务器设置响应头：Access-Control-Request-Method
  - “预检”请求时，允许请求头则需服务器设置响应头：Access-Control-Request-Headers
  - “预检”缓存时间，服务器设置响应头：Access-Control-Max-Age

  HTML文件

  ```html
  <!DOCTYPE html>
  <html>
  <head lang="en">
      <meta charset="UTF-8">
      <title></title>
  </head>
  <body>
  
      <p>
          <input type="submit" onclick="XmlSendRequest();" />
      </p>
  
      <p>
          <input type="submit" onclick="JqSendRequest();" />
      </p>
  
      <script type="text/javascript" src="jquery-1.12.4.js"></script>
      <script>
          function XmlSendRequest(){
              var xhr = new XMLHttpRequest();
              xhr.onreadystatechange = function(){
                  if(xhr.readyState == 4) {
                      var result = xhr.responseText;
                      console.log(result);
                  }
              };
              xhr.open('GET', "http://c2.com:8000/test/", true);
              xhr.send();
          }
  
          function JqSendRequest(){
              $.ajax({
                  url: "http://c2.com:8000/test/",
                  type: 'GET',
                  dataType: 'text',
                  success: function(data, statusText, xmlHttpRequest){
                      console.log(data);
                  }
              })
          }
  
  
      </script>
  </body>
  </html>
  复制代码
  
  class MainHandler(tornado.web.RequestHandler):
      def get(self):
          self.set_header('Access-Control-Allow-Origin', "http://www.xxx.com")
          self.write('{"status": true, "data": "seven"}')
  b、支持跨域，复杂请求
  
  由于复杂请求时，首先会发送“预检”请求，如果“预检”成功，则发送真实数据。
  
  “预检”请求时，允许请求方式则需服务器设置响应头：Access-Control-Request-Method
  “预检”请求时，允许请求头则需服务器设置响应头：Access-Control-Request-Headers
  “预检”缓存时间，服务器设置响应头：Access-Control-Max-Age
  
  复制代码
  <!DOCTYPE html>
  <html>
  <head lang="en">
      <meta charset="UTF-8">
      <title></title>
  </head>
  <body>
  
      <p>
          <input type="submit" onclick="XmlSendRequest();" />
      </p>
  
      <p>
          <input type="submit" onclick="JqSendRequest();" />
      </p>
  
      <script type="text/javascript" src="jquery-1.12.4.js"></script>
      <script>
          function XmlSendRequest(){
              var xhr = new XMLHttpRequest();
              xhr.onreadystatechange = function(){
                  if(xhr.readyState == 4) {
                      var result = xhr.responseText;
                      console.log(result);
                  }
              };
              xhr.open('PUT', "http://c2.com:8000/test/", true);
              xhr.setRequestHeader('k1', 'v1');
              xhr.send();
          }
  
          function JqSendRequest(){
              $.ajax({
                  url: "http://c2.com:8000/test/",
                  type: 'PUT',
                  dataType: 'text',
                  headers: {'k1': 'v1'},
                  success: function(data, statusText, xmlHttpRequest){
                      console.log(data);
                  }
              })
          }
      </script>
  </body>
  </html>
  ```

  Tornado

  ```
  class MainHandler(tornado.web.RequestHandler):
      
      def put(self):
          self.set_header('Access-Control-Allow-Origin', "http://www.xxx.com")
          self.write('{"status": true, "data": "seven"}')
  
      def options(self, *args, **kwargs):
          self.set_header('Access-Control-Allow-Origin', "http://www.xxx.com")
          self.set_header('Access-Control-Allow-Headers', "k1,k2")
          self.set_header('Access-Control-Allow-Methods', "PUT,DELETE")
          self.set_header('Access-Control-Max-Age', 10)
  ```

  c、跨域获取响应头

  默认获取到的所有响应头只有基本信息，如果想要获取自定义的响应头，则需要再服务器端设置Access-Control-Expose-Headers。

  ```
  class MainHandler(tornado.web.RequestHandler):
      
      def put(self):
          self.set_header('Access-Control-Allow-Origin', "http://www.xxx.com")
  
          self.set_header('xxoo', "seven")
          self.set_header('bili', "daobidao")
  
          self.set_header('Access-Control-Expose-Headers', "xxoo,bili")
  
  
          self.write('{"status": true, "data": "seven"}')
  
      def options(self, *args, **kwargs):
          self.set_header('Access-Control-Allow-Origin', "http://www.xxx.com")
          self.set_header('Access-Control-Allow-Headers', "k1,k2")
          self.set_header('Access-Control-Allow-Methods', "PUT,DELETE")
          self.set_header('Access-Control-Max-Age', 10)
  ```

  d、跨域传输cookie

  在跨域请求中，默认情况下，HTTP Authentication信息，Cookie头以及用户的SSL证书无论在预检请求中或是在实际请求都是不会被发送。

  如果想要发送：

  - 浏览器端：XMLHttpRequest的withCredentials为true
  - 服务器端：Access-Control-Allow-Credentials为true
  - 注意：服务器端响应的 Access-Control-Allow-Origin 不能是通配符 *