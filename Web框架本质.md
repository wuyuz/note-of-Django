### Web框架本质

​		我们可以这样理解：所有的web应用本质就是一个socket服务端，而用户的浏览器就是一个socket客户端，基于请求做出响应，客户都先请求，服务端做出对应的响应。即客户端按照http协议的请求协议发送请求，服务端按照http协议的响应协议来响应请求，这样的网络通信，我们就可以自己实现Web框架了。

​		准确的说，web框架就是基于socket实现的轮子，也可以比作一个建筑的钢筋混凝土，也就是为我们你提供好的架子。通过对socket的学习，我们知道网络通信，我们完全可以自己写了，因为socket就是做网络通信用的，下面我们就基于socket来自己实现一个web框架，写一个web服务端，让浏览器来请求，并通过自己的服务端把页面返回给浏览器，浏览器渲染出我们想要的效果。在后面的学习中，大家提前准备一些文件：

------

##### 

##### 一、简单的Web框架

- 一般来说，只要我们遵循了http（超文本传输协议，是不是和超文本标记语言HTML很熟啊？）那么我们的消息就一定会在浏览器中显示。

  ```python
  import socket
  from threading import Thread
  
  sk = socket.socket()
  sk.bind(('127.0.0.1',8001)) #默认浏览器端口8001
  sk.listen()
  
  def func(conn):
      from_b_msg = conn.recv(1024)
      str_msg = from_b_msg.decode('utf-8')
      # socket是应用层和传输层之间的抽象层，每次都有协议，协议就是消息格式，那么传输层的消息格式我们不用管，
      # 因为socket帮我们搞定了，但是应用层的协议还是需要咱们自己遵守的，所以再给浏览器发送消息的时候，如果没有按照应用层
      # 的消息格式来写，那么你返回给浏览器的信息，浏览器是没法识别的。而应用层的协议就是我们的HTTP协议，所以我们按照HTTP协议规定
      # 的消息格式来给浏览器返回消息就没有问题了，关于HTTP我们会细说，首先看一下直接写conn.send(b'hello')的效果，然后运行代码，
      # 通过浏览器来访问一下，然后再看这一句conn.send(b'HTTP/1.1 200 ok \r\n\r\nhello')的效果
      # 下面这句就是按照http协议来写的
      # conn.send(b'HTTP/1.1 200 ok \r\n\r\nhello')
      # 上面这句还可以分成下面两句来写
      conn.send(b'HTTP/1.1 200 ok \r\n\r\n')  # 这里的两个回车+换行就是消息头的格式（http知识）
      conn.send(b'hello')
   
  while 1:
      conn,addr = sk.accept()
      t = Thread(target=func,args=(conn,)) #实现并发
      t.start()   
  #凡是有浏览器访问我的电脑的ip和8001端口，就会收到我的hello消息
  ```

- 学会分析浏览器发送的消息（注意每一行都有\r\n，知道消息头结束有\r\n\r\n结束）

  ![1557911721280](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1557911721280.png)

- **注意：**但我们使用最简单的方式以字节流的形式给浏览器发送一段以utf-8编码的字符的时候，如：’哈哈哈‘，会发现会乱码，解决方案：

  ![1560300747375](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1560300747375.png)
  
- 下面我们写一个html文件，将给来请求的浏览器返回一个html文件

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <!--//这里引用了jd的图标，是一个网址，可以不必再本地-->
      <title>Title</title><link rel="icon" href="https://www.jd.com/favicon.ico">
      <!--直接写在html页面里面的css样式是直接可以在浏览器上显示的-->
      <style>
          h1{
              background-color: green;
              color: white;
          }
      </style>
  </head>
  <body>
  
  <h1>姑娘，你好，我是Jaden，请问约吗？嘻嘻~~</h1>
  <!--直接写在html页面里面的img标签的src属性值如果是别人网站的地址（网络地址）是直接可以在浏览器上显示的-->
  <!--<img src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1550395461724&di=c2b971db12eef5d85aba410d1e2e8568&imgtype=0&src=http%3A%2F%2Fy0.ifengimg.com%2Fifengimcp%2Fpic%2F20140822%2Fd69e0188b714ee789e97_size87_w800_h1227.jpg" alt="">-->
  <!--如果都是网络地址，那么只要你的电脑有网，就可以看到，不需要自己在后端写对应的读取文件，返回图片文件信息的代码，因为别人的网站就做了这个事情了-->
  
  <!--直接写在html页面里面的js操作是直接可以在浏览器上显示的-->
  <script>
      // 写在html中js、css可以直接有效果，否则需要把相应的应用文件读取传到浏览器的缓存区，以用来渲染
      alert('这是我们第一个网页')  
  </script>
  </body>
  </html>
  ```

  ![1557913124315](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1557913124315.png)

  

##### 二、本地文件如何引用

- 上面的html文件自带了css、js以及.ico文件(图标、图片)的样式，但在真正开发的文件中都是使用link来进行文件的引用，通过验证上例使用link引用，会使css、js失效，那么如何处理？

  ![1557914158786](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1557914158786.png)

  ```html
  <link rel="stylesheet" href="test.css">
      <!--上一句失效、加上下面这句，那么我们看浏览器调试窗口中的那个network里面就没有那个favicon.ico的请求了，其实这就是页面title标签文字左边的那个页面图标,但是这个文件是我们自己本地的，所以我们需要在后端代码里面将这个文件数据读取出来返回给前端-->
  <link rel="icon" href="wechat.ico">
  
  当然我们可以使用网上的资源，这样link就会去网上找对应的资源，前提是能联网
  ```

  ![1557915259956](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1557915259956.png)

- 当我们直接在浏览器上保存某个页面的时候，随便一个页面，我们到页面上点击右键另存为，然后存到本地的一个目录下，你会发现这个页面的html、css、js、图片等文件都跟着保存下来了，我保存了一下博客园首页的页面。也就是说我们哪能去下载一个页面时，浏览器会自动打包（为支持这个html页面的各种相应文件，包括图片、css、js等），否则无法显示出效果。

- 我们将这些为正常显示html网页的加载文件叫做**静态文件，** 原来就是将html页面需要的css、js、图片等文件也发送给浏览器就可以了，并且这些静态文件都是浏览器单独过来请求的，其实和标签的属性有有关系，css文件是link标签的href属性：<link rel="stylesheet" href="test.css">，js文件是script标签的src属性：<script src="test.js"></script>，图片文件是img标签的src属性：\<img src="meinv.png" alt="" width="100" height="100"> ，那个.ico文件是link标签的属性：<link rel="icon" href="wechat.ico">，其实这些属性都会在页面加载的时候，单独到自己对应的属性值里面取请求对应的文件数据，而且我们如果在值里面写的都是自己本地的路径，那么都会来自己的本地路径来找，如果我们写的是相对路径，就会到我们自己的网址+文件名称，这个路径来找它需要的文件，所以我们只需要将这些请求做一些响应，将对应的文件数据相应给浏览器就可以了！并且我们通过前面的查看，能够发现，浏览器url的请求路径我们知道是什么，静态文件不是也这样请求的吗，好，我们针对不同的路径给它返回不同的文件。

- **总结**: 我们可以得知，所有含有link、src属性的标签都会独立的向服务器（相对路径时）或网址去取得数据，**一般优先去本地找，没有时会查看缓存中是否有，最后再去网络找**。根据这一点我们就可以判断浏览器返回的信息来返回相应的文件回去。

  

##### 三、返回静态文件的高级Web框架

- ##### 简单版本

  - 通过切割发送到服务端的消息，得到浏览器最初读取html文件时link需要的文件，然后我们就能通过判断来打开相应的文件发送给浏览器

  ```python
  import socket
  sk = socket.socket()
  sk.bind(('127.0.0.1',8001))
  sk.listen()
  #首先浏览器相当于给我们发送了多个请求，一个是请求我们的html文件，而我们的html文件里面的引入文件的标签又给我们这个网站
  # 发送了请求静态文件的请求，所以我们要将建立连接的过程循环起来，才能接受多个请求，没毛病
  while 1:
      conn,addr = sk.accept()
      from_b_msg = conn.recv(1024)
      str_msg = from_b_msg.decode('utf-8')
      #通过http协议我们知道，浏览器请求的时候，有一个请求内容的路径，通过对请求信息的分析，
      # 这个路径我们在请求的所有请求信息中可以提炼出来，下面的path就是我们提炼出来的路径
      path = str_msg.split('\r\n')[0].split(' ')[1]
      print('path>>>',path)
      conn.send(b'HTTP/1.1 200 ok \r\n\r\n')
      #由于整个页面需要html、css、js、图片等一系列的文件，所以我们都需要给人家浏览器发送过去，浏览器才能有这些文件，才能很好的渲染你的页面
      #根据不同的路径来返回响应的内容
      if path == '/': #返回html文件
          print(from_b_msg)
          with open('myhtml.html','rb') as f:
              data = f.read()
          conn.send(data)
          conn.close()
  
      elif path == '/my.css': #返回css文件
          with open('my.css','rb') as f:
              css_data = f.read()
          conn.send(css_data)
          conn.close()
  
      elif path == '/favicon.ico':#返回页面的ico图标
          with open('favicon.ico','rb') as f:
              ico_data = f.read()
          conn.send(ico_data)
          conn.close()
  
      elif path == '/my.js': #返回js文件
          with open('my.js','rb') as f:
              js_data = f.read()
          conn.send(js_data)
          conn.close()
  
      #注意：上面每一个请求处理完之后，都有一个conn.close()是因为，HTTP协议是短链接的，一次请求对应一次响应，这个请求就结束了，所以我们需要写上close，不然浏览器自己断了，你自己写的服务端没有断，就会出问题。（当然一般来说这个短链接是相对的，为了防止用户点击页面后马上跳转到本地的下一个文件中，从而增加连接次数，所以短暂停留3秒，来避免）
  ```
  

![1557916845429](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1557916845429.png)

- ##### 函数加多线程版本

  ```python
  import socket
  from threading import Thread
  #注意一点，不开多线程完全是可以搞定的，在这里只是教大家要有并发编程的思想，所以我使用了多线程
  sk = socket.socket()
  sk.bind(('127.0.0.1',8001))
  sk.listen()
  
  def func1(conn):
      with open('myhtml.html', 'rb') as f:
          # with open('Python开发.html','rb') as f:
          data = f.read()
      conn.send(data)
      conn.close()
  
  def func3(conn):
      with open('my.css', 'rb') as f:
          css_data = f.read()
      conn.send(css_data)
      conn.close()
  
  def func4(conn):
      with open('favicon.ico', 'rb') as f:
          ico_data = f.read()
      conn.send(ico_data)
      conn.close()
  
  def func5(conn):
      with open('my.js', 'rb') as f:
          js_data = f.read()
      conn.send(js_data)
      conn.close()
  
  while 1:
      conn,addr = sk.accept()
      from_b_msg = conn.recv(1024)
      str_msg = from_b_msg.decode('utf-8')
      path = str_msg.split('\r\n')[0].split(' ')[1]
      print('path>>>',path)
      conn.send(b'HTTP/1.1 200 ok \r\n\r\n')
      print(from_b_msg)
      if path == '/':
          t = Thread(target=func1,args=(conn,))
          t.start()
      elif path == '/my.css':
          t = Thread(target=func3, args=(conn,))
          t.start()
      elif path == '/favicon.ico':
          t = Thread(target=func4, args=(conn,))
          t.start()
      elif path == '/my.js':
          t = Thread(target=func5, args=(conn,))
          t.start()
  ```
  
- ##### 更高级版本

  ```python
  import socket
  from threading import Thread
  
  sk = socket.socket()
  sk.bind(('127.0.0.1',8001))
  sk.listen()
  
  def func1(conn):
      conn.send(b'HTTP/1.1 200 ok\r\ncontent-type:text/html\r\ncharset:utf-8\r\n\r\n')
      with open('myhtml.html', 'rb') as f:
          data = f.read()
      conn.send(data)
      conn.close()
  
  def func3(conn):
      conn.send(b'HTTP/1.1 200 ok\r\n\r\n')
      with open('my.css', 'rb') as f:
          css_data = f.read()
      conn.send(css_data)
      conn.close()
  
  def func4(conn):
      conn.send(b'HTTP/1.1 200 ok\r\n\r\n')
      with open('favicon.ico', 'rb') as f:
          ico_data = f.read()
      conn.send(ico_data)
      conn.close()
  
  def func5(conn):
      conn.send(b'HTTP/1.1 200 ok\r\n\r\n')
      with open('my.js', 'rb') as f:
          js_data = f.read()
      conn.send(js_data)
      conn.close()
  
  #定义一个路径和执行函数的对应关系，不再写一堆的if判断了
  l1 = [
      ('/',func1),
      ('/my.css',func3),
      ('/favicon.ico',func4),
      ('/my.js',func5),
  ]
  
  #遍历路径和函数的对应关系列表，并开多线程高效的去执行路径对应的函数，
  def fun(path,conn):
      for i in l1:
          if i[0] == path:
              t = Thread(target=i[1],args=(conn,))
              t.start()
              
  while 1:
      conn,addr = sk.accept()
      # 看完这里面的代码之后，你就可以思考一个问题了，很多人要同时访问你的网站，
      # 你在请求这里是不是可以开起并发编程的思想了，多进程+多线程+协程，妥妥的支持高并发，再配合服务器集群，
      # 这个网页就支持大量的高并发了，有没有很激动，哈哈，但是咱们写的太low了，而且功能很差，容错能力也很差，当然了，
      # 如果你有能力，你现在完全可以自己写web框架了，写一个nb的，如果现在没有这个能力，那么我们就来好好学学别人写好的框架把，
      # 首先第一个就是咱们的django框架了，其实就是将这些功能封装起来，并且容错能力强，抗压能力强，总之一个字：吊。
      from_b_msg = conn.recv(1024)
      str_msg = from_b_msg.decode('utf-8')
      path = str_msg.split('\r\n')[0].split(' ')[1]
      print('path>>>',path)
      # 注意：因为开启的线程很快，可能导致你的文件还没有发送过去，其他文件的请求已经来了，导致你文件
      # 信息没有被浏览器正确的认识，所以需要将发送请求行和请求头的部分写道前面的每一个函数里面去，并且防止出现(http,只认识html，
      # 也就是说它只从html下口，浏览器可能不能识别你的html文件的情况，需要在发送html文件的那个函数里面的发送请求行和请求头的部分加上两个请求头
      # content-type:text/html\r\ncharset:utf-8\r\n
      # conn.send(b'HTTP/1.1 200 ok\r\n\r\n')  不这样写了
      # conn.send(b'HTTP/1.1 200 ok\r\ncontent-type:text/html\r\ncharset:utf-8\r\n\r\n')  不这样写了
      #执行这个fun函数并将路径和conn管道都作为参数传给他
      fun(path,conn)
  ```
  
- ##### 返回动态页面的web框架

  ​		这网页能够显示出来了，但是都是静态的啊。页面的内容都不会变化的，我想要的是动态网站，动态网站的意思是里面有动态变化的数据（这里通过replace来代替，使页面动态），而不是页面里面有动态效果，这个大家要注意啊。没问题，我也有办法解决。我选择使用字符串替换来实现这个需求。（这里使用时间戳来模拟动态的数据，还是只给大家python代码吧）

  ```python
  """
  根据URL中不同的路径返回不同的内容
  返回HTML页面
  让网页动态起来
  """
  import socket
  import time
  sk = socket.socket()
  sk.bind(("127.0.0.1", 8080))  # 绑定IP和端口
  sk.listen()  # 监听
  
  # 将返回不同的内容部分封装成函数
  def index(url):
      with open("index.html", "r", encoding="utf8") as f:
          s = f.read()
          now = str(time.time())
          s = s.replace("@@oo@@", now)  # 在网页中定义好特殊符号，用动态的数据去替换提前定义好的特殊符号
      return bytes(s, encoding="utf8")
  
  def home(url):
      with open("home.html", "r", encoding="utf8") as f:
          s = f.read()
      return bytes(s, encoding="utf8")
  
  # 定义一个url和实际要执行的函数的对应关系
  list1 = [
      ("/index/", index),
      ("/home/", home),
  ]
  
  while 1:
      # 等待连接
      conn, add = sk.accept()
      data = conn.recv(8096)  # 接收客户端发来的消息
      # 从data中取到路径
      data = str(data, encoding="utf8")  # 把收到的字节类型的数据转换成字符串
      # 按\r\n分割
      data1 = data.split("\r\n")[0]
      url = data1.split()[1]  # url是我们从浏览器发过来的消息中分离出的访问路径
      conn.send(b'HTTP/1.1 200 OK\r\n\r\n')  # 因为要遵循HTTP协议，所以回复的消息也要加状态行
      # 根据不同的路径返回不同内容
      func = None  # 定义一个保存将要执行的函数名的变量
      for i in list1:
          if i[0] == url:
              func = i[1]
              break
      if func:
          response = func(url)
      else:
          response = b"404 not found!"
      # 返回具体的响应消息
      conn.send(response)
      conn.close()
  ```
  
- **总结**：我们可以在服务端根据浏览器的消息（请求路径，给予相应的文件回复）

  

##### 四、wsgiref模块 与 WSGI

​			在以上的程序中不难发现有很多重复的代码格式，每个响应函数都会发送固定的格式（GET / HTTP/1.1……）以及每次都要对浏览器的请求，如地址path的拼接等，要知道目前我们还是使用最底层的基于socket的程序，实际上别人早就为我们封装好了这一系列的功能，我们只需要专注于业务逻辑开发既可以（wsgiref模块）。**wsgiref模块其实就是将整个请求信息给封装了起来，就不需要你自己处理了**，假如它将所有请求信息封装成了一个叫做request的对象，那么你直接request.path就能获取到用户这次请求的路径，request.method就能获取到本次用户请求的请求方式(get还是post)等，那这个模块用起来，我们再写web框架是不是就简单了好多啊。

对于真实开发中的python web程序来说，一般会分为两部分：**服务器程序和应用程序**：

```
服务器程序：负责对socket服务器进行封装，并在请求到来时，对请求的各种数据进行整理。

应用程序：则负责具体的逻辑处理。为了方便应用程序的开发，就出现了众多的Web框架，例如：Django、Flask、web.py 等。不同的框架有不同的开发方式，但是无论如何，开发出的应用程序都要和服务器程序配合，才能为用户提供服务。

那么为了实现服务器程序和应用程序解耦，即不同的服务器程序（Apance、nignx）都能和我们写的应用程序进行桥接，那么我们久需要定制一个规范：WSGI
```

-    **统一规范**：正如上面说的众多web框架，不同的框架有不同的开发方式，这样服务器程序就需要为不同的框架提供不同的支持。这样混乱的局面无论对于服务器还是框架，都是不好的。对服务器来说，需要支持各种不同框架，对框架来说，只有支持它的服务器才能被开发出的应用使用。

  - 最简单的Web应用就是先把HTML用文件保存好，用一个现成的HTTP服务器软件，接收用户请求，从文件中读取HTML，返回。如果要动态生成HTML，就需要把上述步骤自己来实现。不过，接受HTTP请求、解析HTTP请求、发送HTTP响应都是苦力活，如果我们自己来写这些底层代码，还没开始写动态HTML呢，就得花个把月去读HTTP规范。
  -  正确的做法是底层代码由专门的服务器软件实现，我们用Python专注于生成HTML文档。因为我们不希望接触到TCP连接、HTTP原始请求和响应格式，所以，需要一个统一的接口协议来实现这样的服务器软件，让我们专心用Python编写Web业务。
  -  我们来设立一个标准：**只要服务器程序支持这个标准，框架也支持这个标准，那么他们就可以配合使用。一旦标准确定，双方各自实现。这样，服务器可以支持更多支持标准的框架，框架也可以使用更多支持标准的服务器**。

-    WSGI（Web Server Gateway Interface）就是一种规范，它定义了使用Python编写的web应用程序与web服务器程序之间的接口格式（桥梁），实现web应用程序与web服务器程序间的解耦。

-   常用的WSGI服务器有**uwsgi、Gunicorn**。而Python标准库提供的独立WSGI服务器叫**wsgiref**，Django开发环境用的就是这个模块来做服务器。

-   **总的来说**：WSGI服务器，是一种服务器应用程序，用于规范的收发整理客户端和服务端的消息，使我们能专注于逻辑开发。

- 具体说明wsgiref怎么使用：

  ```python
  from wsgiref.simple_server import make_server
  # wsgiref本身就是个web框架，提供了一些固定的功能（请求和响应信息的封装，不需要我们自己写原生的socket了也不需要咱们自己来完成请求信息的提取了，提取起来很方便）
  #函数名字随便起
  def application(environ, start_response):  
      '''
      :param environ: 是全部加工好的请求信息，加工成了一个字典，通过字典取值的方式就能拿到很多你想要拿到的信息
      :param start_response: 帮你封装响应信息的（响应行和响应头），注意下面的参数
      :return:
      '''
      start_response('200 OK', [('Content-Type', 'text/html'),('k1','v1')]) # 这里使我们可以定制消息头，以元组的形式，wsgiref以便做成字典,不写这句话会乱码
      print(environ)  # 一大堆整理的数据字典
      print(environ['PATH_INFO'])  #输入地址127.0.0.1:8000，这个打印的是'/',输入的是127.0.0.1:8000/index，打印结果是'/index'
      return [b'<h1>Hello, web!</h1>']  # 用于返回数据，之前我们需要进行send，这里return，wsgiref帮我们发送
  
  #和咱们学的socketserver那个模块很像啊
  httpd = make_server('127.0.0.1', 8080, application)
  
  print('Serving HTTP on port 8080...')
  # 开始监听HTTP请求:
  httpd.serve_forever()
  ```
  

![1557992417252](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1557992417252.png)



##### 五、完善web项目

###### 简单的实现一个用户登陆认证的项目：

- 通过上面的介绍我们知道wsgiref的简单用法，下面我们为了完善这个项目，需要用大数据库，让用户输入密码和用户名，合法则调整至登陆界面

  ```sql
  mysql> create database db1;
  mysql> use db1;
  mysql> insert into userinfo(username,password) values('chao','666'),('sb1','222');
  ```

- 1、我们需要以下的几个文件：webmodel.py 用于操作数据库，给数据库进行操作。

  ```python
  def createtable():
      import pymysql
      conn = pymysql.connect(
          host = '127.0.0.1',
          port = 3306,
          user = 'root',
          password = '123',
          database = 'db1',
          charset = 'utf8'
      )    # 连接数据库
      cursor = conn.cursor(pymysql.cursors.DictCursor) # 返回字典的游标
      # sql = '''
      # create table userinfo (id int primary key auto_increment,username char(20) not null unique,password char(20) not null);
      # '''
      
      sql = '''
      insert into userinfo (username,password) values('cha','666'),('sb','333')'''
      cursor.execute(sql)
      conn.commit()
      cursor.close()
      conn.close()
  
  #createtable()，还未开发完整，具体什么时候调用这个函数，以及传入的参数还未完善
  ```

  ![1557995407593](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1557995407593.png)

- 2、一个名为webauth.py 文件，用于验证用户输入的密码是否合法

  ```python
  def auth(username,password):
      import pymysql
      conn = pymysql.connect(
          host = '127.0.0.1',
          port = 3306,
          user = 'root',
          password = '123',
          database = 'db1',
          charset = 'utf8'
      )
      
      print('userinfo',username,password)
      cursor = conn.cursor(pymysql.cursors.DictCursor)
      sql = 'select * from userinfo where username=%s and password=%s;' # 为了防止注入将参数放在execute中
      res = cursor.execute(sql,[username,password]) # res 是返回的查询到的条数
      if res:
          return True  # 查询到者返回True
      else:
          return False
  ```

- 3、用户输入用户名和密码的文件，名为web.html,内容如下：（也就是首页）

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>首页</title>
  </head>
  <body>
  <!--如果form表单里面的action什么值也没给，默认是往当前页面的url上提交你的数据，
  所以我们可以自己指定数据的提交路径-->
      <form action="http://127.0.0.1:8080/auth" method="post">
          用户名：<input type="text" name="username">
          密码： <input type="password" name="'password">
          <input type='submit'>
      </form>
      <form action="http://127.0.0.1:8080/auth" method="post">
          用户名：<input type="text" name="username">
          密码： <input type="password" name="password">
          <input type='submit'>
      </form>    
  </body>
  </html>
  ```

- 4、用户验证后跳转的页面，名为websuccess.html，内容如下

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>登陆成功页面</title>
      <style>
          h1{color:lightcoral;}
      </style>
  </head>
  <body>
  <h1>欢迎光临</h1>
  </body>
  </html>
  ```
  
- 5、最后就是我们的主逻辑程序了，前面都是在搭架子

  ```python
  from urllib.parse import parse_qs  #用于解析数据也就是将浏览器发过来的请求如带有&的进行切割解析
  from wsgiref.simple_server import make_server
  import weauth
  def application(environ,start_response):
      start_response('200 OK',[('Content-Type', 'text/html')]) #这个 方法会帮我们实现消息头的发送
      # print(environ) # 拿到请求消息
      print(environ['PATH_INFO'])
      path = environ['PATH_INFO']
      
      #用户获取login页面的请求路径，进行区分
      if path == '/login':  #首页请求
          with open('web.html','rb') as f:
              data = f.read()
               
      elif path == '/favicon.ico':  # 请求图标
          with open('favicon.ico','rb') as f:
              data = f.read()
                
      elif path == '/auth':  #提交按钮触发，进行用户认证
          # 登陆认证
          #1.获取用户输入的用户名和密码
          #2.去数据库做数据的校验，查看用户提交的是否合法
          if environ.get("REQUEST_METHOD") == "POST":
              # 获取请求体数据的长度,因为提交过来的数据需要用它来提取,注意POST请求和GET请求的获取数据的方式不同
              try:
                  request_body_size = int(environ.get('CONTENT_LENGTH', 0))
              except (ValueError):
                  request_body_size = 0
              # POST请求获取数据的方式
              request_data = environ['wsgi.input'].read(request_body_size)
              print('>>>>>', request_data)  # >>>>> b'username=chao&password=123'，是个bytes类型数据
              print('?????', environ['QUERY_STRING'])  # ????? 空的，因为post请求只能按照上面这种方式取数据
              # parse_qs可以帮我们解析数据
              re_data = parse_qs(request_data.decode('utf-8'))  #post需要解码，传输的过程是接收的字节码
              print('拆解后的数据', re_data)  # 拆解后的数据 {b'password': [b'123'], b'username': [b'chao']}
              username = re_data['username'][0]
              password = re_data['password'][0]
              print(username, password)
              # 进行验证：
              status = weauth.auth(username, password)
              if status:
                  # 3.将相应内容返回
                  with open('websuccess.html', 'rb') as f:
                      data = f.read()
              else:
                  data = b'auth error'
              
          if environ.get("REQUEST_METHOD") == "GET":
              # GET请求获取数据的方式，只能按照这种方式取
              print('?????', environ['QUERY_STRING'])  # ????? username=chao&password=123,是个字符串类型数据,返回用户输入的数据和密码
              request_data = environ['QUERY_STRING']  #交给request_data 进行切割，数据解析
      
              # parse_qs可以帮我们解析数据
              re_data = parse_qs(request_data)    # 字符串
              print('拆解后的数据', re_data)        # 拆解后的数据 {'password': ['123'], 'username': ['chao']}
              username = re_data['username'][0]
              password = re_data['password'][0]
              print(username, password)
              # 进行验证：
              status = weauth.auth(username, password)
              if status:
                  # 3.将相应内容返回
                  with open('websuccess.html', 'rb') as f:
                      data = f.read()
              else:
                  data = b'auth error'
          # 但是不管是post还是get请求都不能直接拿到数据，拿到的数据还需要我们来进行分解提取，所以我们引入urllib模块来帮我们分解
          # 注意昂，我们如果直接返回中文，没有给浏览器指定编码格式，默认是gbk，所以我们需要gbk来编码一下，浏览器才能识别
      return [data]
  
  httpd = make_server('127.0.0.1',8080,application)
  print('Serving HTTP on port 8080……')
  httpd.serve_forever()
  ```

  ![1557999994397](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1557999994397.png)

  

###### 起飞版Web框架

- 上面的主逻辑代码使用的太多的if语句，显得很冗余，违背了Python的初衷，所以我们需要更进一步，我们将主逻辑程序的路径选择写在一个列表中。

  ```python
  from urllib.parse import parse_qs  #用于解析数据也就是将浏览器发过来的请求如带有&的进行切割解析
  from wsgiref.simple_server import make_server
  import weauth
  
  def login(environ):   #访问login会触发的函数
      with open('web.html', 'rb') as f:
          data = f.read()
      return data
  
  def auth(environ):  # 提交时触发的函数
      if environ.get("REQUEST_METHOD") == "POST":    
          try:
              request_body_size = int(environ.get('CONTENT_LENGTH', 0))
          except (ValueError):
              request_body_size = 0
          request_data = environ['wsgi.input'].read(request_body_size)
          print('>>>>>', request_data)
          print('?????', environ['QUERY_STRING'])
          re_data = parse_qs(request_data.decode('utf-8'))
          print('拆解后的数据', re_data)
          username = re_data['username'][0]
          password = re_data['password'][0]
          print(username, password)
          status = weauth.auth(username, password)
          if status:
              with open('websuccess.html', 'rb') as f:
                  data = f.read()
          else:
              data = b'auth error'
      
      if environ.get("REQUEST_METHOD") == "GET":
          print('?????', environ['QUERY_STRING'])
          request_data = environ['QUERY_STRING']
          
          re_data = parse_qs(request_data)  # 字符串
          print('拆解后的数据', re_data)  # 拆解后的数据 {'password': ['123'], 'username': ['chao']}
          username = re_data['username'][0]
          password = re_data['password'][0]
          print(username, password)
          status = weauth.auth(username, password)
          if status:
              with open('websuccess.html', 'rb') as f:
                  data = f.read()
          else:
              data = b'auth error'
      return data
  
  def favicon(environ): #图标触发的函数
      with open('favicon.ico', 'rb') as f:
          data = f.read()
      return data
  
  urlpatterns = [    #者就是用来判断请求地址时，于函数做映射，路由
      ('/login',login),
      ('/auth',auth),
      ('/favicon.ico',favicon)
  ]
  
  def application(environ,start_response):  #主逻辑函数
      start_response('200 OK',[('Content-Type', 'text/html')]) #这个 方法会帮我们实现消息头的发送
      # print(environ) # 拿到请求消息
      print(environ['PATH_INFO'])
      path = environ['PATH_INFO']
      
      #用户获取login页面的请求路径，进行区分
      for urlpattern in urlpatterns: #循环路由
          if path == urlpattern[0]:
              data=urlpattern[1](environ)  #接收返回值
              break
      else:
          data = b'sorry 404!'
      return [data]
  
  httpd = make_server('127.0.0.1',8080,application)
  print('Serving HTTP on port 8080……')
  httpd.serve_forever()
  ```

###### 飞升版web框架

- 对之前主逻辑的文件进行拆分：不难发现其实之前的文件都可以不受影响，我们只需要将主逻辑文件分解成三个文件即可：view.py（负责业务逻辑，即浏览器请求网址对应的函数），urls.py（负责路由，分派网址对应的函数），main.py(主逻辑函数，负责具体的判断，筛选)

- 1、main.py 简化的主逻辑文件

  ```python
  from wsgiref.simple_server import make_server
  from urls import urlpatterns
  
  def application(environ,start_response):  #主逻辑函数
      start_response('200 OK',[('Content-Type', 'text/html')]) #这个 方法会帮我们实现消息头的发送
      # print(environ) # 拿到请求消息
      print(environ['PATH_INFO'])
      path = environ['PATH_INFO']
      
      #用户获取login页面的请求路径，进行区分
      for urlpattern in urlpatterns: #循环路由
          if path == urlpattern[0]:
              data=urlpattern[1](environ)  #接收返回值
              break
      else:
          data = b'sorry 404!'
      return [data]
  
  httpd = make_server('127.0.0.1',8080,application)
  print('Serving HTTP on port 8080……')
  httpd.serve_forever()
  ```

- 2、view.py 具体的业务逻辑

  ```python
  from urllib.parse import parse_qs  #用于解析数据也就是将浏览器发过来的请求如带有&的进行切割解析
  import weauth
  
  def login(environ):  # 访问login会触发的函数
      with open('web.html', 'rb') as f:
          data = f.read()
      return data
  
  def auth(environ):  # 提交时触发的函数
      if environ.get("REQUEST_METHOD") == "POST": 
          try:
              request_body_size = int(environ.get('CONTENT_LENGTH', 0))
          except (ValueError):
              request_body_size = 0       
          request_data = environ['wsgi.input'].read(request_body_size)
          print('>>>>>', request_data)
          print('?????', environ['QUERY_STRING'])
          re_data = parse_qs(request_data.decode('utf-8'))
          print('拆解后的数据', re_data)
          username = re_data['username'][0]
          password = re_data['password'][0]
          print(username, password)
          status = weauth.auth(username, password)
          if status:
              with open('websuccess.html', 'rb') as f:
                  data = f.read()
          else:
              data = b'auth error'   
      if environ.get("REQUEST_METHOD") == "GET":
          print('?????', environ['QUERY_STRING'])
          request_data = environ['QUERY_STRING']
          
          re_data = parse_qs(request_data)  # 字符串
          print('拆解后的数据', re_data)  # 拆解后的数据 {'password': ['123'], 'username': ['chao']}
          username = re_data['username'][0]
          password = re_data['password'][0]
          print(username, password)
          status = weauth.auth(username, password)
          if status:
              with open('websuccess.html', 'rb') as f:
                  data = f.read()
          else:
              data = b'auth error'
      return data
  
  def favicon(environ):  # 图标触发的函数
      with open('favicon.ico', 'rb') as f:
          data = f.read()
      return data
  ```
  
- 3、**urls.py 路由文件，映射逻辑**

  ```python
  from view import login,auth,favicon
  
  urlpatterns = [    #者就是用来判断请求地址时，于函数做映射，路由
      ('/login',login),
      ('/auth',auth),
      ('/favicon.ico',favicon)
  ]
  ```




##### MVC模式

​	Web服务器开发领域里著名的MVC模式，所谓MVC就是把Web应用分为模型(M)，控制器(C)和视图(V)三层，他们之间以一种插件式的、松耦合的方式连接在一起，模型负责业务对象与数据库的映射(ORM)，视图负责与用户的交互(页面)，控制器接受用户的输入调用模型和视图完成用户的请求，其示意图如下所示：

- M: model  模型

- V：view   视图   - HTML 

- C: controller  控制器   ——路由 传递指令  业务逻辑，url和views

![1558058226286](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558058226286.png)



##### MTV模式

​		　Django的MTV模式本质上和MVC是一样的，也是为了各组件间保持松耦合关系，只是定义上有些许不同	Django的MTV分别是值：

- M 代表模型（Model）： 负责业务对象和数据库的关系映射(ORM)。
- T 代表模板 (Template)：负责如何把页面展示给用户(html)。
- V 代表视图（View）：   负责业务逻辑，并在适当时候调用Model和Template。包括url和view

　　除了以上三层之外，还需要一个URL分发器，它的作用是将一个个URL的页面请求分发给不同的View处理，View再调用相应的Model和Template，MTV的响应模式如下所示：

![1558058497526](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558058497526.png)

​		一般是用户通过浏览器向我们的服务器发起一个请求(request)，这个请求回去访问视图函数，（如果不涉及到数据调用，那么这个时候视图函数返回一个模板也就是一个网页给用户），视图函数调用模型，模型去数据库查找数据，然后逐级返回，视图函数把返回的数据填充到模板中空格中，最后返回网页给用户。



##### 补充：HTTP相关知识

- **HTTP协议简介**

  - 超文本传输协议（英文：**H**yper**T**ext **T**ransfer **P**rotocol，缩写：HTTP）是一种用于分布式、协作式和超媒体信息系统的应用层协议。HTTP是万维网的数据通信的基础。
  - 通常，由HTTP客户端发起一个请求，创建一个到服务器指定端口（默认HTTP是80端口，https默认端口是443）的TCP连接。HTTP服务器则在那个端口监听客户端的请求。一旦收到请求，服务器会向客户端返回一个状态，比如"HTTP/1.1 200 OK"，以及返回的内容，如请求的文件、错误消息、或者其它信息。

- **HTTP工作原理**

  - HTTP协议采用了请求/响应模型。客户端向服务器发送一个请求报文，请求报文包含请求的方法、URL、协议版本、请求头部和请求数据。服务器以一个状态行作为响应，响应的内容包括协议的版本、成功或者错误代码、服务器信息、响应头部和响应数据。

    ```
    以下是 HTTP 请求/响应的步骤：
    1. 客户端连接到Web服务器
    	一个HTTP客户端，通常是浏览器，与Web服务器的HTTP端口（默认为80）建立一个TCP套接字连接。例如，http://www.luffycity.com。
    
    2. 发送HTTP请求
    	通过TCP套接字，客户端向Web服务器发送一个文本的请求报文，一个请求报文由请求行、请求头部、空行和请求数据4部分组成。
    
    3. 服务器接受请求并返回HTTP响应
    	Web服务器解析请求，定位请求资源。服务器将资源复本写到TCP套接字，由客户端读取。一个响应由状态行、响应头部、空行和响应数据4部分组成。
    
    4. 释放连接TCP连接
    	若connection 模式为close，则服务器主动关闭TCP连接，客户端被动关闭连接，释放TCP连接;若connection 模式为keepalive，则该连接会保持一段时间，在该时间内可以继续接收请求;
    
    5. 客户端浏览器解析HTML内容
    	客户端浏览器首先解析状态行，查看表明请求是否成功的状态代码。然后解析每一个响应头，响应头告知以下为若干字节的HTML文档和文档的字符集。客户端浏览器读取响应数据HTML，根据HTML的语法对其进行格式化，并在浏览器窗口中显示。
    	
    
    面试题：在浏览器地址栏键入URL，按下回车之后会经历以下流程：
       1、 浏览器向 DNS 服务器请求解析该 URL 中的域名所对应的 IP 地址;
       2、 解析出 IP 地址后，根据该 IP 地址和默认端口 80，和服务器建立TCP连接;
       3、 浏览器发出读取文件(URL 中域名后面部分对应的文件)的HTTP 请求，该请求报文作为 TCP 三次	 握手的第三个报文的数据发送给服务器;
       4、 服务器对浏览器请求作出响应，并把对应的 html 文本发送给浏览器;
       5、 释放 TCP连接;
       6、 浏览器将该 html 文本并显示内容; 　
    ```

  - **HTTP的特点**

    ```
    1、基于请求-响应的模式，是基于TCP/IP协议之上的应用层协议
    2、无状态保存：我发什么来着？？我也不知？？
    3、无连接：无连接的含义是限制每次连接只处理一个请求
    ```

  - **HTTP的请求方法**

    ```
    1、GET：向指定的资源发出“显示”请求。使用GET方法应该只用在读取数据，而不应当被用于产生“副作用”的操作中，例如在Web Application中。其中一个原因是GET可能会被网络蜘蛛等随意访问。
    
    2、HEAD：与GET方法一样，都是向服务器发出指定资源的请求。只不过服务器将不传回资源的本文部分。它的好处在于，使用这个方法可以在不必传输全部内容的情况下，就可以获取其中“关于该资源的信息”（元信息或称元数据）。
    
    3、POST：向指定资源提交数据，请求服务器进行处理（例如提交表单或者上传文件）。数据被包含在请求本文中。这个请求可能会创建新的资源或修改现有资源，或二者皆有。
    
    4、PUT：向指定资源位置上传其最新内容。
    
    5、DELETE：请求服务器删除Request-URI所标识的资源。
    
    6、TRACE：回显服务器收到的请求，主要用于测试或诊断。
    
    7、OPTIONS：这个方法可使服务器传回该资源所支持的所有HTTP请求方法。用'*'来代替资源名称，向Web服务器发送OPTIONS请求，可以测试服务器功能是否正常运作。
    
    8、CONNECT：HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器。通常用于SSL加密服务器的链接（经由非加密的HTTP代理服务器）。
    
    请求方式: get与post请求（通过form表单我们自己写写看）区别：
        1、GET提交的数据会放在URL之后，也就是请求行里面，以?分割URL和传输数据，参数之间以&相连，如EditBook?name=test1&id=123456.（请求头里面那个content-type做的这种参数形式，后面讲） POST方法是把提交的数据放在HTTP包的请求体中.
        2、GET提交的数据大小有限制（因为浏览器对URL的长度有限制），而POST方法提交的数据没有限制.
    ```

- **HTTP状态码**

  - 所有HTTP响应的第一行都是状态行，依次是当前HTTP版本号，3位数字组成的状态代码，以及描述状态的短语，彼此由空格分隔。

  - 状态代码的第一个数字代表当前响应的类型：

    ![1560222902096](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1560222902096.png)

- **URL**

  - 超文本传输协议（HTTP）的统一资源定位符将从因特网获取信息的五个基本元素包括在一个简单的地址中：

    ```
    1、传送协议。
    2、层级URL标记符号(为[//],固定不变)
    3、访问资源需要的凭证信息（可省略）
    4、服务器。（通常为域名，有时为IP地址）
    5、端口号。（以数字方式表示，若为HTTP的默认值“:80”可省略）
    6、路径。（以“/”字符区别路径中的每一个目录名称）
    7、查询。（GET模式的窗体参数，以“?”字符为起点，每个参数以“&”隔开，再以“=”分开参数名称与数	  	据，通常以UTF8的URL编码，避开字符冲突的问题）
    8、片段。以“#”字符为起点
    
    以http://www.luffycity.com:80/news/index.html?id=250&page=1 为例, 其中：
    http:传输协议;www.luffycity.com，是服务器；80，是服务器上的默认网络端口号，默认不显示；/news/index.html，是路径（URI：直接定位到对应的资源）；?id=250&page=1，是查询。
    	大多数网页浏览器不要求用户输入网页中“http://”的部分，因为绝大多数网页内容是超文本传输协议文件。同样，“80”是超文本传输协议文件的常用端口号，因此一般也不必写明。一般来说用户只要键入统一资源定位符的一部分（www.luffycity.com:80/news/index.html?id=250&page=1）就可以了;
    ```

- **HTTP请求格式（请求协议）**
  ​	![1560223539633](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1560223539633.png)

  下面使用POST方法来具体说明具体的表现：

  ![1560223751808](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1560223751808.png)



- **HTTP响应格式（响应协议）**

  ![1560223882325](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1560223882325.png)

  下面是一个服务器收到请求后返回的响应格式

  ![1560223936221](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1560223936221.png)





#### 总结

```
将来要说的MVC框架是什么呢：
　　　　M：model.py  就是和数据库打交道用的，创建表等操作
　　　　V：View  视图（视图函数，html文件）
　　　　C：controller  控制器（其实就是我百度云代码里面那个urls文件里面的内容，url（路径）分发与视图函数的逻辑处理）
　　　　
 　　Django叫做MTV框架：
　　　　M：model.py  就是和数据库打交道用的，创建表等操作（和上面一样）
　　　　T：templates  存放HTML文件的
　　　　V：View 视图函数（逻辑处理）
　　　　其实你会发现MTV比MVC少一个url分发的部分
　　　　所以我们学的django还要学一个叫做url控制器（路径分发）的东西，MTV+url控制器就是我们django要学的内容。 　　　　
　　捋一下框架的整个流程吧~~~
```

