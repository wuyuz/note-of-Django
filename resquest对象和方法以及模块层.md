### resquest对象和方法

#### request对象

- 需要说明的是在Django1.x版本，使用的是request，等于后来的HttpRequest对象

- [官方解释](https://docs.djangoproject.com/en/1.11/ref/request-response/)：当一个页面被请求时，Django就会创建一个包含本次请求原信息（请求报文中的请求行、首部信息、内容主体等）的HttpRequest对象。Django会将这个对象自动传递给响应的视图函数，一般视图函数约定俗成地使用 request 参数承接这个对象。

- ##### 常用方法和属性

  - **属性**：

    ```python
    path_info        返回用户访问url，不包括域名
    method           请求中使用的HTTP方法的字符串表示，全大写表示。
    GET              包含所有HTTP  GET参数的类字典对象
    POST             包含所有HTTP POST参数的类字典对象
    body             请求体，byte类型 request.POST的数据就是从body里面提取到的
    FILES            上传文件：注意点，1、form表单中enctype = 'multipart/form-data';2:chunkes(),一段一段的读取
    
    -------------我们在函数中起的别名是request，其实是HttpRequest属性------------
    #下面的的这些属性是HttpRequest携带的属性，但是我们传入的request是在其基础上封装了的，所以你在函数中调用HttpRequest.GET等是没有用的
    
    1.HttpRequest.GET
    　　一个类似于字典的对象，包含 HTTP GET 的所有参数。详情请参考 QueryDict 对象。可以使用get方法获取
    
    2.HttpRequest.POST
    　　一个类似于字典的对象，如果请求中包含表单数据，则将这些数据封装成 QueryDict 对象。
    　　POST 请求可以带有空的 POST 字典 —— 如果通过 HTTP POST 方法发送一个表单，但是表单中没有任何的数据，QueryDict 对象依然会被创建。因此，不应该使用 if request.POST  来检查使用的是否是POST 方法；应该使用 if request.method == "POST"
    　　另外：如果使用 POST 上传文件的话，文件信息将包含在 FILES 属性中。
       注意：键值对的值是多个的时候,比如checkbox类型的input标签，select标签，需要用：
            request.POST.getlist("hobby")
    
    3.HttpRequest.body
    　　一个字符串，代表请求报文的主体。在处理非 HTTP 形式的报文时非常有用，例如：二进制图片、XML,Json等。
    　　但是，如果要处理表单数据的时候，推荐还是使用 HttpRequest.POST 。 get方式是没有得，post方式是一段通过url转码得字符
    
    4.HttpRequest.path
    　　一个字符串，表示请求的路径组件（不含域名）。
    　　例如："/music/bands/the_beatles/"
    
    5.HttpRequest.method
    　　一个字符串，表示请求使用的HTTP 方法。必须使用大写。
    　　例如："GET"、"POST"
    
    6.HttpRequest.encoding
    　　一个字符串，表示提交的数据的编码方式（如果为 None 则表示使用 DEFAULT_CHARSET 的设置，默认为 'utf-8'）。
       这个属性是可写的，你可以修改它来修改访问表单数据使用的编码。
       接下来对属性的任何访问（例如从 GET 或 POST 中读取数据）将使用新的 encoding 值。
       如果你知道表单数据的编码不是 DEFAULT_CHARSET ，则使用它。
    
    7.HttpRequest.META
     　　一个标准的Python 字典，包含所有的HTTP 首部。具体的头部信息取决于客户端和服务器，下面是一些示例：
    
        CONTENT_LENGTH —— 请求的正文的长度（是一个字符串）。
        CONTENT_TYPE —— 请求的正文的MIME 类型。
        HTTP_ACCEPT —— 响应可接收的Content-Type。
        HTTP_ACCEPT_ENCODING —— 响应可接收的编码。
        HTTP_ACCEPT_LANGUAGE —— 响应可接收的语言。
        HTTP_HOST —— 客服端发送的HTTP Host 头部。
        HTTP_REFERER —— Referring 页面。
        HTTP_USER_AGENT —— 客户端的user-agent 字符串。
        QUERY_STRING —— 单个字符串形式的查询字符串（未解析过的形式）。
        REMOTE_ADDR —— 客户端的IP 地址。
        REMOTE_HOST —— 客户端的主机名。
        REMOTE_USER —— 服务器认证后的用户。
        REQUEST_METHOD —— 一个字符串，例如"GET" 或"POST"。
        SERVER_NAME —— 服务器的主机名。
        SERVER_PORT —— 服务器的端口（是一个字符串）。
     　　从上面可以看到，除 CONTENT_LENGTH 和 CONTENT_TYPE 之外，请求中的任何 HTTP 首部转换为 META 的键时，
        都会将所有字母大写并将连接符替换为下划线最后加上 HTTP_  前缀。
        所以，一个叫做 X-Bender 的头部将转换成 META 中的 HTTP_X_BENDER 键。
    
    8.HttpRequest.FILES
    　　一个类似于字典的对象，包含所有的上传文件信息。
       FILES 中的每个键为<input type="file" name="" /> 中的name，值则为对应的数据。
    　　#注意，FILES 只有在请求的方法为POST 且提交的<form> 带有enctype="multipart/form-data" 的情况下才会包含数据。否则，FILES 将为一个空的类似于字典的对象。
    
    
    9.HttpRequest.COOKIES
    　　一个标准的Python 字典，包含所有的cookie。键和值都为字符串。
    
    10.HttpRequest.session
     　　一个既可读又可写的类似于字典的对象，表示当前的会话。只有当Django 启用会话的支持时才可用。
        完整的细节参见会话的文档。
    
    11.HttpRequest.user(用户认证组件下使用)
    　　一个 AUTH_USER_MODEL 类型的对象，表示当前登录的用户。
    
    　　如果用户当前没有登录，user 将设置为 django.contrib.auth.models.AnonymousUser 的一个实例。你可以通过 is_authenticated() 区分它们。
    
        例如：
    
        if request.user.is_authenticated():
            # Do something for logged-in users.
        else:
            # Do something for anonymous users.
         　　user 只有当Django 启用 AuthenticationMiddleware 中间件时才可用。
     
    12.HttpRequest.path_info： 返回用户访问url，不包括域名
     ----------------------------------------------------------------------
    
        匿名用户
        class models.AnonymousUser
        django.contrib.auth.models.AnonymousUser 类实现了django.contrib.auth.models.User 接口，但具有下面几个不同点：
    
        id 永远为None。
        username 永远为空字符串。
        get_username() 永远返回空字符串。
        is_staff 和 is_superuser 永远为False。
        is_active 永远为 False。
        groups 和 user_permissions 永远为空。
        is_anonymous() 返回True 而不是False。
        is_authenticated() 返回False 而不是True。
        set_password()、check_password()、save() 和delete() 引发 NotImplementedError。
        New in Django 1.8:
        新增 AnonymousUser.get_username() 以更好地模拟 django.contrib.auth.models.User。
        
    11.HttpRequest.session
     　　一个既可读又可写的类似于字典的对象，表示当前的会话。只有当Django 启用会话的支持时才可用。
        完整的细节参见会话的文档。    
    ```
    
  - **HttpRequest和函数中的参数 request的区别**：
  
    ```
    首先我们在函数体中单独使用HttpRequest.GET是会报错的，查看其源码可以发现，函数中request使用的方法都是有的，且HttpRequest的实例化，也就是说所有的request都是HttpRequest的实例化，在每个请求来的时候，wsgi.py会将所有的信息封装到request中（HttpRequest类中实例化）
    ```
  
    
  
  - **方法**：
  
  ```
  1.HttpRequest.get_host()
    
    　　根据从HTTP_X_FORWARDED_HOST（如果打开 USE_X_FORWARDED_HOST，默认为False）和 HTTP_HOST 头部信息返回请求的原始主机。
       如果这两个头部没有提供相应的值，则使用SERVER_NAME 和SERVER_PORT，在PEP 3333 中有详细描述。
    　　USE_X_FORWARDED_HOST：一个布尔值，用于指定是否优先使用 X-Forwarded-Host 首部，仅在代理设置了该首部的情况下，才可以被使用。
    　　例如："127.0.0.1:8000"
    　　注意：当主机位于多个代理后面时，get_host() 方法将会失败。除非使用中间件重写代理的首部。
    
    2.HttpRequest.get_full_path()
    　　返回 path，如果可以将加上查询字符串。
    　　例如："/music/bands/the_beatles/?print=true"
    
     
    3.HttpRequest.get_signed_cookie(key, default=RAISE_ERROR, salt='', max_age=None)
    　　返回签名过的Cookie 对应的值，如果签名不再合法则返回django.core.signing.BadSignature。
    　　如果提供 default 参数，将不会引发异常并返回 default 的值。
    　　可选参数salt 可以用来对安全密钥强力攻击提供额外的保护。max_age 参数用于检查Cookie 对应的时间戳以确保Cookie 的时间不会超过max_age 秒
            >>> request.get_signed_cookie('name')
            'Tony'
            >>> request.get_signed_cookie('name', salt='name-salt')
            'Tony' # 假设在设置cookie的时候使用的是相同的salt
            >>> request.get_signed_cookie('non-existing-cookie')
            ...
            KeyError: 'non-existing-cookie'    # 没有相应的键时触发异常
            >>> request.get_signed_cookie('non-existing-cookie', False)
            False
            >>> request.get_signed_cookie('cookie-that-was-tampered-with')
            ...
            BadSignature: ...    
            >>> request.get_signed_cookie('name', max_age=60)
            ...
            SignatureExpired: Signature age 1677.3839159 > 60 seconds
            >>> request.get_signed_cookie('name', False, max_age=60)
            False
                     
    4.HttpRequest.is_secure()
    　　如果请求时是安全的，则返回True；即请求通是过 HTTPS 发起的。
    
    
    5.HttpRequest.is_ajax()
    　　如果请求是通过XMLHttpRequest 发起的，则返回True，方法是检查 HTTP_X_REQUESTED_WITH 相应的首部是否是字符			  串'XMLHttpRequest'。
    　　大部分现代的 JavaScript 库都会发送这个头部。如果你编写自己的 XMLHttpRequest 调用（在浏览器端），你必须手工设置这个值来让  is_ajax() 可以工作。
    　　如果一个响应需要根据请求是否是通过AJAX 发起的，并且你正在使用某种形式的缓存例如Django 的 cache middleware， 
       你应该使用 vary_on_headers('HTTP_X_REQUESTED_WITH') 装饰你的视图以让响应能够正确地缓存。
  ```
  



##### HttpRequest和QueryDict

- 查看HttpRequest源码

  ```python
  class HttpRequest(object):
      def __init__(self):
          self.GET = QueryDict(mutable=True)
          self.POST = QueryDict(mutable=True)
          self.COOKIES = {}
          self.META = {}
          self.FILES = MultiValueDict()
          ...
   # 我们可以知道GET和POST都是QueryDict的对象，且默认传入了mutable=True ，这样再执行初始化函数时设置了self._mutable = True,
  ```

- 无处不在的`_assert_mutable`函数

  ```python
  def _assert_mutable(self):
         if not self._mutable:
             raise AttributeError("This QueryDict instance is immutable")
  
   # 再QueryDict字典中几乎所有给对象的方法都设置了此函数，用于现在随意修改，必须在给定了这个对象的_mutable为True时才能使用
  ```

- 设置request.POST/GET为可编辑对象的方式

  ```python
  1、request.POST._mutable = True
  2、dp = request.POST.copy() # QueryDict自带深拷贝
  
  3、dp1 = QueryDict(mutable=True) #dp1此时可编辑
  ```

  ![1564395593039](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1564395593039.png)





- **实例：读取上传文件**

  ```python
  def upload(request):
      """保存上传文件前，数据需要存放在某个位置。默认当上传文件小于2.5M时，django会将上传文件的全部内容读进内存。从内存读取一次，写磁盘一次。但当上传文件很大时，django会把上传文件写到临时文件中，然后存放到系统临时文件夹中。"""
      if request.method == "POST":
          # 从请求的FILES中获取上传文件的文件名，file为页面上type=files类型input的name属性值
          filename = request.FILES["file"].name
          # 在项目目录下新建一个文件
          with open(filename, "wb") as f:
              # 从上传的文件对象中一点一点读
              for chunk in request.FILES["file"].chunks():
                  # 写入本地文件
                  f.write(chunk)
          return HttpResponse("上传OK")
  ```

- **实例2：各项数据**

  ```python
  from django.shortcuts import render,HttpResponse,redirect
  
  # Create your views here.
  
  def index(request):
      print(request.method) #请求方式
      print(request.path)   #请求路径，不带参数的
      print(request.POST)   #post请求数据  字典格式
      print(request.GET)    #get的请求数据  字典格式
      print(request.META)   #请求头信息,将来用到哪个咱们再说哪个
      print(request.get_full_path())   #获取请求路径带参数的，/index/?a=1
      print(request.is_ajax())   #判断是不是ajax发送的请求，True和False
      '''
          Django一定最后会响应一个HttpResponse的示例对象
          三种形式：
              1、 HttpResponse('字符串') 最简单
              2、 render(页面)   最重要
                   2.1 两个功能
                      -- 读取文件字符串
                      -- 嵌入变量(模板渲染) html里面：{{ name }} ， {'name':'chao'}作为render的第三个参数，想写多个变量						{'name':'chao','hobby':['篮球','羽毛球']....}
              3、 redirect() 重定向  最难理解,某个网站搬家了，网址变了，访问原来的网址就重定向到一个新网址，就叫做重定向，网站自己做的重定向，你访问还是访问的你之前的，你自己啥也不用做，浏览器发送请求，然后服务端响应，然后服务端告诉浏览器，你直接跳转到另外一个网址上，那么浏览器又自动发送了另外一个请求，发送到服务端，服务端返回一个页面，包含两次请求，登陆成功后跳转到网站的首页，网站首页的网址和你login登陆页面的网址是不用的。             
      '''
      return render(request,'index.html',{'name':'chao'})
  #也就是说，每个视图函数都是以： return HttpResponse/render/redirect 结束
  ```

- **注意**：键值对的值是多个的时候,比如checkbox类型的input标签，select标签，需要用：

  ```python
  1、request.POST.getlist("hobby")  #可以同时取checkbox的多个被选中的值
  2、request.POST.urlencode()  #将QueryDict字典里的键值对编码成指定格式
```
  
  

#### response对象

- 与由Django自动创建的HttpRequest对象相比，HttpResponse对象是我们的职责范围了。我们写的每个视图都需要实例化，填充和返回一个HttpResponse。HttpResponse类位于django.http模块中。

- 具体使用

  ```python
  #传递字符串
  from django.http import HttpResponse
  response = HttpResponse("Here's the text of the Web page.")
  response = HttpResponse("Text only, please.", content_type="text/plain")
  
  #设置或删除响应头信息
  response = HttpResponse()
  response['Content-Type'] = 'text/html; charset=UTF-8'
  del response['Content-Type']
  ```

- **属性**

  ```
  HttpResponse.content：响应内容
  HttpResponse.charset：响应内容的编码
  HttpResponse.status_code：响应的状态码
  ```

  

#### JsonResponse对象

- 　JsonResponse是HttpResponse的子类，专门用来生成JSON编码的响应。

  ```python
  from django.http import JsonResponse
  response = JsonResponse({'foo': 'bar'})
  print(response.content)
  b'{"foo": "bar"}'
  
  #参数和概念
  class JsonResponse(data, encoder=DjangoJSONEncoder, safe=True, json_dumps_params=None,**kwargs)
  这个类是HttpRespon的子类，它主要和父类的区别在于：
  　　1.它的默认Content-Type 被设置为： application/json
  　　2.第一个参数，data应该是一个字典类型，当 safe 这个参数被设置为：False ,那data可以填入任何能被转换为JSON格式的对象，比如list,tuple, set。 默认的safe 参数是 True. 如果你传入的data数据类型不是字典类型，那么它就会抛出 TypeError的异常。
  　　3.json_dumps_params参数是一个字典,它将调用json.dumps()方法并将字典中的参数传入给该方法。
  ```

- 在ajax交互时，HttpResponse每次还要进行解析，很麻烦

  ```python
  #如果这样返回，ajax还需要进行json解析
  #views.py
  return HttpResponse(json.dumps({"msg":"ok!"}))
  
  #index.html
  var data=json.parse(data)
  console.log(data.msg);
  ```

  使用HttpResponse对象来响应数据的时候，还可以通过content_type指定格式：

  ```python
  return HttpResponse(json.dumps(data),content_type="application/json")
  ```

  前端调试窗口就可以看到这个类型：

  ![1558358589152](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558358589152.png)

- 更加简便的JsonResponse，默认就是返回content_type="application/json"。

  ```python
  #如果这样返回，两边都不需要进行json的序列化与反序列化，ajax接受的直接是一个对象
  #views.py
  from django.http import JsonResponse
  return JsonResponse({"msg":"ok!"})
  
  #index.html
  console.log(data.msg);
  ```

- **注意**：默认只能传递字典类型，如果要传递非字典类型需要设置一下safe关键字参数。

  ```python
  response = JsonResponse([1, 2, 3], safe=False)
   # 后端使用 JSON.parse() 将字符串解析成原有类型
  ```



#### 响应对象主要有三种形式

```
1、HttpResponse()
2、render()
3、redirect() 
#HttpResponse()括号内直接跟一个具体的字符串作为响应体，比较直接很简单，所以这里主要介绍后面两种形式。
```



##### render() 说明

```python
render(request, template_name,[context]）
#结合一个给定的模板和一个给定的上下文字典，并返回一个渲染后的 HttpResponse 对象。

参数：
    request：   用于生成响应的请求对象。
    template_name：要使用的模板的完整名称，可选的参数
    context：添加到模板上下文的一个字典。默认是一个空字典。如果字典中的某个值是可调用的，视图将在渲染模板之前调用它。
#render方法就是将一个模板页面中的模板语法进行渲染，最终渲染成一个html页面作为响应体。
```

- **实例：使用render进行渲染html文件**

  ```python
  #views.py中
  from django.shortcuts import render
  def index(request):
      name = 'xiao'
      return render(request,"index.html",{"name":name})
  
  -------------------------------------------------------------------
  #html中
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
  <h3>INDEX</h3>
  <p>Hi, {{ name }}</p>
  </body>
  ```



**redirect() 说明**：给浏览器一个30x状态码

```python
参数可以是：
    1. 一个模型：将调用模型的get_absolute_url() 函数

    2.一个视图，可以带有参数：将使用urlresolvers.reverse 来反向解析名称

    3.一个绝对的或相对的URL，将原封不动的作为重定向的位置。如果是post提交的网址index/会在本地址基础上拼接index/,如果使用/index/则只会到xx/index/。
默认返回一个临时的重定向；传递permanent=True 可以返回一个永久的重定向

-------------------------------------------------------------------
#传递要重定向的一个硬编码的URL
def my_view(request):
    ...
    return redirect('/some/url/')

#注意redirect自带又reverse功能，也就是说，你可以直接写一个别名，也能跳转到相应的网址。

--------------------------------------------------------------------
#例如：下列设置就直接会跳转到manage名字对应的函数中，可带参数，下文有
#在views.py文件中
def test(request):
    return redirect('app:manage')  #这里是因为取了命名空间，没有命名空间就不加app:

#在url文件中
urlpatterns = [
    url(r'^books/manage/$', views.manage,name='manage'),  
]

---------------------------------特殊用法-----------------------------
#1、传递一个具体的ORM对象
from django.shortcuts import redirect
def my_view(request):
    ...
    object = MyModel.objects.get(...)
    return redirect(object)

#2、传递一个视图名称，还可以带参数
def my_view(request):
    ...
    return redirect('some-view-name', foo='bar')  # 这里的some-view-name就是别名

#3、传递要重定向的一个具体的网址
def my_view(request):
    ...
    return redirect('/some/url/')
```

- **关于重定向，简单来说：**

  ![1558401189884](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558401189884.png)
  
  通过修改http协议的header部分，对浏览器下达重定向指令的，让浏览器对location中指定的url提出请求，使浏览器显示重定向网页的内容。

![1558359466547](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558359466547.png)

```
301和302的区别：
　　301和302状态码都表示重定向，就是说浏览器在拿到服务器返回的这个状态码后会自动跳转到一个新的URL地址，这个地址可以从响应的Location首部中获取（用户看到的效果就是他输入的地址A瞬间变成了另一个地址B）——这是它们的共同点。
　　他们的不同在于。301表示旧地址A的资源已经被永久地移除了（这个资源不可访问了），搜索引擎在抓取新内容的同时也将旧的网址交换为重定向之后的网址；
　　302表示旧地址A的资源还在（仍然可以访问），这个重定向只是临时地从旧地址A跳转到地址B，搜索引擎会抓取新的内容而保存旧的网址。 SEO302好于301

2）重定向原因：
（1）网站调整（如改变网页目录结构）；
（2）网页被移到一个新地址；
（3）网页扩展名改变(如应用需要把.php改成.Html或.shtml)。
        这种情况下，如果不做重定向，则用户收藏夹或搜索引擎数据库中旧地址只能让访问客户得到一个404页面错误信息，访问流量白白丧失；再者某些注册了多个域名的。网站，也需要通过重定向让访问这些域名的用户自动跳转到主站点等。
```

- ##### **传递一个具体的ORM对象（了解即可）**

  ```python
  #将调用具体ORM对象的get_absolute_url() 方法来获取重定向的URL：
  from django.shortcuts import redirect
   
  def my_view(request):
      ...
      object = MyModel.objects.get(...)
      return redirect(object)
  ```

- **传递一个视图的名称**

  ```python
  def my_view(request):
      ...
      return redirect('some-view-name', foo='bar')  #在这个相当于重定向，会跳转到视图函数别名的路径，并给该函数传递一个参数foo
  ```

- **传递要重定向到的一个具体的网址**

  ```python
  def my_view(request):
      ...
      return redirect('/some/url/')
  ```

- **当然也可以是一个完整的网址**

  ```python
  def my_view(request):
      ...
      return redirect('http://example.com/')
  ```


- **redirect和render的区别**

  ```python
  #index.html文件
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
  <div>这是index页面</div>
  <h1>{{ name }}</h1>
  
  </body>
  </html>
  
  --------------------------------------------------------
  #login.html文件
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
  
  <div>
      <form action="{% url 'xxx' %}" method="post">
          用户名：<input type="text" name="username">
          密码：<input type="password" name="password">
          <input type="submit">
      </form>
  
  </div>
  
  </body>
  </html>
  
  -------------------------------------------------------
  #urls.py文件
  from django.conf.urls import url
  from django.contrib import admin
  from app01 import views
  urlpatterns = [
      url(r'^index/', views.index),
      url(r'^login/', views.login,name='xxx'),
  ]
  
  -------------------------------------------------------
  #views.py视图函数里面的内容使用 redirect重定向
  from django.shortcuts import render,HttpResponse,redirect
  
  def index(request):
      return render(request,'index.html',{'name':'chao'})
  
  def login(request):
      method = request.method
      if method == 'GET':
          return render(request,'login.html')
      else:
          username = request.POST.get('username')
          password = request.POST.get('password')
          if username == 'chao' and password == '123':
              return redirect('/index/') #重定向到/index/路径，这也是发送了一个请求，别忘了在上面引入这个redirect类，和render、Httpresponse在一个地方引入
               # return render(request,'index.html')  #如果直接用render来返回页面，是一次响应就返回了页面，两者是有区别的，并且如果你用render返回index.html页面，那么这个页面里面的模板渲染语言里面需要的数据你怎么搞，如果这些数据就是人家index那个函数里面独有的呢（或则是通过路径传参），你怎么搞，有人可能就想了，我把所有的数据都拿过来不就行了吗，首先如果数据量很大的话，是不是都重复了，并且你想想如果用户登陆完成之后，你们有进行跳转，那么如果网速不太好，卡一下，你想刷新一下你的页面，你是不是相当于又发送了一个login请求，你刷新完之后，是不是还要让你输入用户名和密码，你想想是不是，所有咱们一般在登陆之后都做跳转。
        # 并且大家注意一个问题昂：redirect('/login/')如果你重定向到你当前这个函数对应的路径下，你想想是什么想过，一直重定向自己的这个网址，浏览器会报错，当然这个注册登陆页面不会出现这个报错的情况，因为需要你用户点击提交才发送请求。你可以试试那个index函数，里面返回一个redirect('/index/')
              #redirect本质上也是一个HttpResponse的操作，看看源码就知道了
          else:
              return HttpResponse('失败')
  
  #在浏览器中查看Network，可以发现有两次请求：login和index，也就是说重定向会通过路径去取视图函数。
  ---------------------------------------------------------------------
  
  ```
  





------

#### Django的模块层

- 之前学习了jinja2来进行模块渲染，在Django中自带模块渲染。[官方](https://docs.djangoproject.com/en/1.11/ref/templates/builtins/#std:templatetag-for)

  ```
  语法：
  　关于模板渲染你只需要记两种特殊符号（语法）：
  　　1、{{  }}和 {% %}
  　　2、变量相关的用{{}}，逻辑相关的用{%%}。
  ```

- ##### 变量

  ```
  在Django的模板语言中按此语法使用：{{ 变量名 }}。
  　当模版引擎遇到一个变量，它将计算这个变量，然后用结果替换掉它本身。 变量的命名包括任何字母数字以及下划线 ("_")的组合。 变量名称中不能有空格或标点符号。
  　
  深度查询据点符（.）在模板语言中有特殊的含义。当模版系统遇到点(".")，它将以这样的顺序查询：
      1、字典查询（Dictionary lookup）
      2、属性或方法查询（Attribute or method lookup）
      3、数字索引查询（Numeric index lookup）
  注意事项：
  	1、如果计算结果的值是可调用的，它将被无参数的调用。 调用的结果将成为模版的值。
  	2、如果使用的变量不存在， 模版系统将插入 string_if_invalid 选项的值， 它被默认设置为'' (空字符串) 
  ```

- **实例1：**

  - views.py文件中

    ```python
    def index(request):
        import datetime
        s = "hello"
        l = [111, 222, 333]  # 列表
        dic = {"name": "yuan", "age": 18}  # 字典
        date = datetime.date(1993, 5, 2)  # 日期对象
    
        class Person(object):
            def __init__(self, name):
                self.name = name
            def dream(self):
                return 'dreamer'
        person_yuan = Person("chao")  # 自定义类对象
        person_egon = Person("yantao")
        person_alex = Person("jinxin")
    
        person_list = [person_yuan, person_egon, person_alex]
    
        return render(request, "index.html", {"l": l, "dic": dic, "date": date, "person_list": person_list})
        # return render(request,'index.html',locals())
        # locals()获取函数内容所有的变量，然后通过render方法给了index.html文件进行模板渲染，如果你图省事，你可以用它，但是很多多余的变量也被传进去了，效率低
    ```

  - html文件中：

    ```html
    <h4>{{s}}</h4>
    <h4>列表:{{ l.0 }}</h4>  
    <h4>列表:{{ l.2 }}</h4>
    
    <h4>字典:{{ dic.name }}</h4>
    <h4>字典:{{ dic.keys }}</h4>
    <h4>字典:{{ dic.items }}</h4>
    <h4>字典:{{ dic.values }}</h4>
    
    <h4>日期:{{ date.year }}</h4>
    
    <!--取列表的第1个对象的name属性的值，不能通过[]来取，要使用.,列表和字典都是如此-->
    <h4>类对象列表:{{ person_list.0.name }}</h4>
    <!--取列表的第1个对象的dream方法的返回值，如果没有返回值，拿到的是none-->
    <h4>类对象列表:{{ person_list.0.dream }}</h4>
    注意：
        调用对象里面的方法的时候，不需要写括号来执行，并且只能执行不需要传参数的方法，如果你的这个方法需要传参数，那么模板语言不支持，不能帮你渲染
    ```

  - **注意**：句点符也可以用来引用对象的方法(无参数方法):

    ```python
    <h4>字典:{{ dic.name.upper }}</h4>
    ```

- **深度查询的方法**

  ```python
  #1、字符串
  #url.py
  def index(request):
      name = 'xiao'
      return render(request,"index.html",{"name":name})
  #index.html
  <p>{{ name }}</p>
  
  #2、列表
  def index(request):
      li = [123, 456, 567]
      return render(request,"index.html",{"li":li})
  
  #index.html
  <p>{{ li }}</p>
  取第一个元素
  <p>{{ li.0 }}</p>
  取第二个元素
  <p>{{ li.1 }}</p>
  取第三个元素
  <p>{{ li.2 }}</p>
  
  #3、字典
  def index(request):
      info = {'name':'xiao','age':'23'}
      return render(request,"index.html",{"info":info})
  
  #index.html
  <p>{{ info }}</p>
  取name
  <p>{{ info.name }}</p>
  取age
  <p>{{ info.age }}</p>
  
  #4、类实例对象
  #修改index视图函数
def index(request):
      class Annimal(object):
          def __init__(self, name, age):
              self.name = name
              self.age = age
          def call(self):
              return "汪汪汪"
  
      dog = Annimal("旺财", 3)
      cat = Annimal("小雪", 4)
      duck = Annimal("小黄", 5)
      animal_list = [dog, cat, duck]
      return render(request,"index.html",{"animal_list":animal_list})
  
  #index.html修改boby部分
  <p>{{ animal_list }}</p>
  取一个对象:
  <p>{{ animal_list.0 }}</p>
  取属性:
  <p>{{ animal_list.0.name }}</p>
  取方法:
  <p>{{ animal_list.0.call }}</p>
  
  #注意：取方法的时候，不要加括号。它不能接收参数！
  #call方法，必须用return。否则取方法时，结果输出None
  #总结：深度查询，就是不停的点点点就行了。
  ```
  
  ![1558366930086](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558366930086.png)



------

#### 过滤器（内置过滤器）

- **语法**：

  ```
  {{obj|filter__name:param}}  # obj：对象；filter_name:语句 
  ```

-  **add**：给一个变量做加法运算，还可以做字符串、列表的拼接

  ```python
  {{ value|add:number }}
  
  #举例：修改index视图函数
  def index(request):
      num = 50
      return render(request,"index.html",{"num":num})
  
  #修改index.html,修改body部分
  <p>{{ num|add:100 }}</p>
  ```

  ![1558367814579](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558367814579.png)

- **default**：如果一个变量是false或则为空，使用给定的默认值。否则，使用变量的值。例如：

  ```python
  语法：{{ value|default:"nothing" }}
  
  #修改index视图函数
  def index(request):
      book_list = []
      return render(request,"index.html",{"book_list":book_list})
  
  #修改index.html，修改body部分
  <p>{{ book_list|default:'没有任何书籍' }}</p>
  ```

  ![1558368294206](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558368294206.png)

- **length**：返回值的长度。它对字符串和列表起作用。例如：

  ```python
  {{ value|length }}
  
  #index函数
  def index(request):
      female_star = ['赵丽颖','宋茜','霍思燕','唐嫣']
      return render(request,"index.html",{"female_star":female_star})
      
  #index.html
  <p>{{ female_star|length }}</p>    #4
  ```

- **filesizeformat**: 将值格式化为一个“人类可读的”文件尺寸（例如‘13KB’，‘4.1MB','102bytes'等等)。例如：

  ```python
  {{ value|filesizeformat }}
  
  #如果 value 是 123456789，输出将会是 117.7 MB
  #index函数
  def index(request):
      file_size = 123456789
      return render(request,"index.html",{"file_size":file_size})
     
  #index.html，修改body部分
  <p>{{ file_size|filesizeformat }}</p>
  ```

  ![1558368764595](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558368764595.png)

- **date** : 时间

  ```python
  如果 value=datetime.datetime.now()
  {{ value|date:"Y-m-d" }}
  
  #修改index视图函数
  def index(request):
      import datetime
      now = datetime.datetime.now()
      return render(request,"index.html",{"now":now})
      
  #修改index.html,修改body部分
  <p>{{ now|date:'Y-m-d H:i:s' }}</p>
  ```

  ![1558368896780](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558368896780.png)

- **slice** ： 切片

  ```python
  如果 value="hello world"
  {{ value|slice:"2:-1" }}
  
  #修改index函数
  def index(request):
      val = 'hello world'
      return render(request,"index.html",{"val":val})
     
  #修改index.html,修改body部分
  <p>{{ val|slice:'1:-1' }}</p>
  
  #还可以设置步长
  <p>{{ val|slice:'1:-1：2' }}</p>
  ```

  ![1558369042223](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558369042223.png)

- **truncatechars** ：截断；如果字符串字符多于指定的字符数量，那么会被截断。截断的字符串将以可翻译的省略号序列（“...”）结尾。

  **truncatewords**：表示一个字符串为准

  ```python
  参数：要截断的字符数
  {{ value|truncatechars:9 }}
  
  #比如后台管理页面，展示文章内容部分，一般取前20个字符串，后面直接用...来代替。修改index函数
  def index(request):
      text = 'Django 教程 Python下有许多款不同的 Web 框架。Django是重量级选手中最有代表性的一位。许多成功的网站和APP都基于Django。'
      return render(request,"index.html",{"text":text})
      
  #index.html函数
  <p>{{ text|truncatechars:20 }}</p>
  ```

  ![1558369165428](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558369165428.png)

- **safe** ： Django的模板中会对HTML标签和JS等语法标签进行自动转义，原因显而易见，这样是为了安全。但是有的时候我们可能不希望这些HTML元素被转义，比如我们做一个内容管理系统，后台添加的文章中是经过修饰的，这些修饰可能是通过一个类似于FCKeditor编辑加注了HTML修饰符的文本，如果自动转义的话显示的就是保护HTML标签的源文件。为了在Django中关闭HTML的自动转义有两种方式，如果是一个单独的变量我们可以通过过滤器“|safe”的方式告诉Django这段代码是安全的不必转义（防止XSS攻击）。比如：

  ```python
  value="<a href="">点击</a>"
  #html文件
  {{ value|safe}}
  
  #举例
  #修改index视图函数，注意：a标签要有内容才行！
  def index(request):
      link = '<a href="http://www.py3study.com/">click</a>'
      return render(request,"index.html",{"link":link})
      
  #修改index.html，修改body部分
  <p>{{ link }}</p>  #不加safe，就是一些文本
  ```

  ![1558369307231](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558369307231.png)

  加了safe：

  ```python
  <p>{{ link|safe }}</p>
  ```

  ![1558369336383](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558369336383.png)

- **cut ：**移除value中所有的与给出的变量相同的字符串

  ```python
  {{ value|cut:' ' }}  
  #如果value为'i love you'，那么将输出'iloveyou'.
  ```

-  **join**:使用字符串连接列表，{{ list|join:', ' }}，就像Python的str.join(list)

-  **timesince**:将日期格式设为自该日期起的时间（例如，“4天，6小时”）

  ```python
  {{ blog_date|timesince:comment_date }}
  #例如，如果blog_date是表示2006年6月1日午夜的日期实例，并且comment_date是2006年6月1日08:00的日期实例，则以下将返回“8小时”：
  ```


##### 注意：

- 当我们将视图函数改为如下，会发生什么：

  ```python
  def login(request):
      return redirect('/login/')
  ```

  ![1558435461458](C:\Users\RootUser\Desktop\知识点复习\Django\resquest对象和方法以及模块层.assets\1558435461458.png)



**过滤器总结**：

| 过滤器             | 描述                                                         |                             示例                             |
| ------------------ | ------------------------------------------------------------ | :----------------------------------------------------------: |
| upper              | 以大写方式输出                                               |                   {{ user.name \| upper }}                   |
| add                | 给value加上一个数值                                          |                  {{ user.age \| add:”5” }}                   |
| capfirst           | 第一个字母大写                                               |              {{ ‘good’\| capfirst }} 返回”Good”              |
| addslashes         | 单引号加上转义号                                             |                                                              |
| center             | 输出指定长度的字符串，把变量居中                             |                  {{ “abcd”\| center:”50” }}                  |
| cut                | 删除指定字符串                                               |        {{ “You are not a Englishman” \| cut:”not” }}         |
| date               | 格式化日期     上面的方式只能配置单个变量的格式，我们可以设置settings中添加：DATE_FORMAT = 'Y-m-d'  TIME_FORMST = 'H:i:s'  修改USE_L10N = False |                 {{ now\|date:"Y-m-d H:i:s"}}                 |
| default            | 如果值不存在，则使用默认值代替                               |                {{ value \| default:”(N/A)” }}                |
| default_if_none    | 如果值为None, 则使用默认值代替                               |                                                              |
| dictsort           | 按某字段排序，变量必须是一个dictionary                       |         {% for moment in moments \| dictsort:”id” %}         |
| dictsortreversed   | 按某字段倒序排序，变量必须是dictionary                       |                                                              |
| divisibleby        | 判断是否可以被数字整除                                       |            `{{ 224 | divisibleby:2 }}  返回 True`            |
| escape             | 按HTML转义，比如将”<”转换为”&lt”                             |                                                              |
| filesizeformat     | 增加数字的可读性，转换结果为13KB,89MB,3Bytes等               |           `{{ 1024 | filesizeformat }} 返回 1.0KB`           |
| first              | 返回列表的第1个元素，变量必须是一个列表                      |                                                              |
| floatformat        | 转换为指定精度的小数，默认保留1位小数                        |    {{ 3.1415926 \| floatformat:3 }} 返回 3.142  四舍五入     |
| get_digit          | 从个位数开始截取指定位置的数字                               |                 {{ 123456 \| get_digit:’1’}}                 |
| join               | 用指定分隔符连接列表                                         |          {{ [‘abc’,’45’] \| join:’*’ }} 返回 abc*45          |
| length             | 返回列表中元素的个数或字符串长度                             |                                                              |
| length_is          | 检查列表，字符串长度是否符合指定的值                         |                {{ ‘hello’\| length_is:’3’ }}                 |
| linebreaks         | 用<p>或<br>标签包裹变量                                      |  {{ “Hi\n\nDavid”\|linebreaks }} 返回<p>Hi</p><p>David</p>   |
| linebreaksbr       | 用<br/>标签代替换行符                                        |                                                              |
| linenumbers        | 为变量中的每一行加上行号                                     |                                                              |
| ljust              | 输出指定长度的字符串，变量左对齐                             |                {{‘ab’\|ljust:5}}返回 ‘ab   ’                 |
| lower              | 字符串变小写                                                 |                                                              |
| make_list          | 将字符串转换为列表                                           |                                                              |
| pluralize          | 根据数字确定是否输出英文复数符号                             |                                                              |
| random             | 返回列表的随机一项                                           |                                                              |
| removetags         | 删除字符串中指定的HTML标记                                   |               {{value \| removetags: “h1 h2”}}               |
| rjust              | 输出指定长度的字符串，变量右对齐，注意：在html页面上再多的空格都是一个空格。我们可以修改源码return('*') |                                                              |
| slice              | 切片操作， 返回列表                                          | {{[3,9,1] \| slice:’:2’}} 返回 [3,9] `{{ 'asdikfjhihgie' | slice:':5' }} 返回 ‘asdik’` |
| slugify            | 在字符串中留下减号和下划线，其它符号删除，空格用减号替换     |      `{{ '5-2=3and5 2=3' | slugify }} 返回 5-23and5-23`      |
| stringformat       | 字符串格式化，语法同python                                   |                                                              |
| time               | 返回日期的时间部分                                           |                                                              |
| timesince          | 以“到现在为止过了多长时间”显示时间变量                       |                  结果可能为 45days, 3 hours                  |
| timeuntil          | 以“从现在开始到时间变量”还有多长时间显示时间变量             |                                                              |
| title              | 每个单词首字母大写                                           |                                                              |
| truncatewords      | 将字符串转换为省略表达方式                                   |  `{{ 'This is a pen' | truncatewords:2 }}返回``This is ...`  |
| truncatewords_html | 同上，但保留其中的HTML标签                                   | `{{ '<p>This is a pen</p>' | truncatewords:2 }}返回``<p>This is ...</p>` |
| urlencode          | 将字符串中的特殊字符转换为url兼容表达方式                    |      {{ ‘http://www.aaa.com/foo?a=b&b=c’ \| urlencode}}      |
| urlize             | 将变量字符串中的url由纯文本变为链接                          |                                                              |
| wordcount          | 返回变量字符串中的单词数                                     |                                                              |
| yesno              | 将布尔变量转换为字符串yes, no 或maybe                        | `{{ True | yesno }}{{ False | yesno }}{{ None | yesno }} ``返回 ``yes``no ``maybe` |



#### 昨日回顾：

### 1.django中所有的命令

下载安装

​	pip  install django==1.11.21  -i 源

创建项目

​	django-admin startproject   项目名称

启动项目

cd 项目根目录下

​	python manage.py  runserver   # 127.0.0.1:8000

​	python manage.py  runserver   80 # 127.0.0.1:80

​	python manage.py  runserver   0.0.0.0:80 # 0.0.0.0:80

创建app

​	python manage.py  startapp  app名称

数据库迁移

​	pyhton manage.py makemigrations

​	pyhton manage.py migrate

### 2.配置

静态文件

​	STATIC_URL = '/static/' 

​	STATICFILES_DIRS = [

​	os.path.join(BASE_DIR,‘static’)，

​	]

模板  TEMPLATES  DIRS  [os.path.join(BASE_DIR,‘templates’)，]

数据库  DATABASES

ENGINE 引擎

NAME   数据库的名称

HOST   ip

PORT   端口   3306

USER  用户民

PASSWORD 密码

app   INSTALLED_APPS  =[  'app01'   'app01.apps.App01Config' ]

中间件  MIDDLEWARE     注释csrf相关的中间   提交POST请求

### 3.request

request.method    ——》  请求方式 GET POST 

request.GET           ——》   url上携带的参数     ?name=ddd&id=11    {}

request.POST        ——》   form表单提交的POST请求的数据   {}

### 4.响应

HttpResponse('字符串')       ——》  字符串

render(request,'模板的文件名',{k1:v1})       ——》 返回的是一个完整的页面

redirect('/index/')    ——》  重定向     Location ： /index/

#### 5.ORM

```python
from django.db import models

class Publisher(models.Model):   # app01_publisher
    pid = models.AutoField(primary_key=Ture)
    name = models.CharField(max_length=32)   # varchar(32)
    
class Book(models.Model):
    title = models.CharField(max_length=32)
    pub = models.ForeignKey('Publisher',on_delete=models.CASCADE)  # pub_id
    
class Author(models.Model):
    name = models.CharField(max_length=32)   # varchar(32)
    books = models.ManyToManyField('Book')
    
-------------------------------------------------------------
from app01 import models
# 查询
models.Publisher.objects.all()  # 查询所有的数据  queryset   对象列表
models.Publisher.objects.get(name='xxx',pid=1)   # Publisher对象  查询不到或者多个 就报错
models.Publisher.objects.filter(name='xxx',pid=1)   # 查询满足条件的所有对象    对象列表
models.Publisher.objects.all().order_by('id')  # 排序  默认升序，降序 -id


pub_obj.pid  pub_obj.pk 
pub_obj.name

book_obj.pub   ——》 所关联的对象
book_obj.pub_id   ——》所关联的对象id


author_obj.books    ——》 关系管理对象
author_obj.books.all()    ——》 所关联的所有对象

# 新增
models.Publisher.objects.create(name='xxxx')

models.Book.objects.create(title='xxxx',pub=Publisher的对象)
models.Book.objects.create(title='xxxx',pub_id=Publisher的对象id)

author_obj = models.Author.objects.create(name='xxxx')
author_obj.books.set([书籍对象的id,书籍对象的id,书籍对象的id])   #  每次都是重新设置，自带保存

# 删除
models.Publisher.objects.get(pk=1).delete()
models.Publisher.objects.filter(pk=1).delete()

# 修改
pub_obj.name = 'xxxx'
pub_obj.save()

book_obj.title = 'xxx'
book_obj.pub = publisher的对象
book_obj.pub_id = publisher的对象的id
book_obj.save() 


author_obj.name = 'xxx'
author_obj.save()
author_obj.books.set([id,id])
```

