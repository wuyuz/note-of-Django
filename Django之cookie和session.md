## Django之cookie和session



#### 会话跟踪

- 我们需要先了解一下什么是会话！可以把会话理解为客户端与服务器之间的一次会晤，在一次会晤中可能会包含多次请求和响应。例如你给10086打个电话，你就是客户端，而10086服务人员就是服务器了。从双方接通电话那一刻起，会话就开始了，到某一方挂断电话表示会话结束。在通话过程中，你会向10086发出多个请求，那么这多个请求都在一个会话中。 客户向某一服务器发出第一个请求开始，会话就开始了，直到客户关闭了浏览器会话结束。

  ```
  在一个会话的多个请求中共享数据，这就是会话跟踪技术。例如在一个会话中的请求如下： 
  
  1、请求银行主页； 
  2、请求登录（请求参数是用户名和密码）；
  3、请求转账（请求参数与转账相关的数据）； 
  4、请求信誉卡还款（请求参数与还款相关的数据）。  
  　　在这上会话中当前用户信息必须在这个会话中共享的，因为登录的是张三，那么在转账和还款时一定是相对张三的转账和还款！这就说明我们必须在一个会话过程中有共享数据的能力。而web中这种能力的实现就要依靠cookie和session。
  ```



#### cookie

- **cookie的由来**：

  ```
  　　　　大家都知道HTTP协议是无状态的。
  　　　　无状态的意思是每次请求都是独立的，它的执行情况和结果与前面的请求和之后的请求都无直接关系，它不会受前面的请求响应情况直接影响，也不会直接影响后面的请求响应情况。
  　　　　一句有意思的话来描述就是人生只如初见，对服务器来说，每次的请求都是全新的。
  　　　　状态可以理解为客户端和服务器在某次会话中产生的数据，那无状态的就以为这些数据不会被保留。会话中产生的数据又是我们需要保存的，也就是说要“保持状态”。因此Cookie就是在这样一个场景下诞生。
  
         并且还有一个问题就是，你登陆我的网站的时候，我没法确定你是不是登陆了，之前我们学的django，虽然写了很多页面，但是用户不用登陆都是可以看所有网页的，只要他知道网址就行，但是我们为了自己的安全机制，我们是不是要做验证啊，访问哪一个网址，都要验证用户的身份，但是还有保证什么呢，用户登陆过之后，还要保证登陆了的用户不需要再重复登陆，就能够访问我网站的其他的网址的页面，对不对，但是http无状态啊，怎么保证这个事情呢？此时就要找cookie了。
  ```

- **什么是cookie**：

  - 首先来讲，cookie是浏览器的技术，Cookie具体指的是一段小信息，**它是服务器发送出来存储在浏览器上的一组组键值对**，可以理解为服务端给客户端的一个小甜点，下次访问服务器时浏览器会自动携带这些键值对，以便服务器提取有用信息。

- **cookie的原理**：

  - cookie的**工作原理**是：浏览器访问服务端，带着一个空的cookie，然后由服务器产生内容，浏览器收到相应后保存在本地；当浏览器再次访问时，浏览器会自动带上Cookie，这样服务器就能通过Cookie的内容来判断这个是“谁”了。'哦，我记得你,问我小甜点的'

  - 查看cookie：使用Chrome浏览器，打开开发者工具

    ![1561533417159](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1561533417159.png)

  - **cookie图解**：

    ![1561533596760](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1561533596760.png)

  - **cookie规范**：

    ```
    1、Cookie大小上限为4KB； 
    2、一个服务器最多在客户端浏览器上保存20个Cookie； 
    3、一个浏览器最多保存300个Cookie，因为一个浏览器可以访问多个服务器。
    　　上面的数据只是HTTP的Cookie规范，但在浏览器大战的今天，一些浏览器为了打败对手，为了展现自己的能力起见，可能对Cookie规范“扩展”了一些，例如每个Cookie的大小为8KB，最多可保存500个Cookie等！但也不会出现把你硬盘占满的可能！ 
    注意，不同浏览器之间是不共享Cookie的。也就是说在你使用IE访问服务器时，服务器会把Cookie发给IE，然后由IE保存起来，当你在使用FireFox访问服务器时，不可能把IE保存的Cookie发送给服务器。
    ```

  - **cookie与HTTP头**：

    ```
    Cookie是通过HTTP请求和响应头在客户端和服务器端传递的： 
    1、Cookie：请求头，客户端发送给服务器端； 
    2、格式：Cookie: a=A; b=B; c=C。即多个Cookie用分号离开； Set-Cookie：响应头，服务器端发送给客户端； 
    3、一个Cookie对象一个Set-Cookie：Set-Cookie: a=A Set-Cookie: b=B Set-Cookie: c=C  
    
     # 在浏览器中的Network的Cookie中有两种分类：
    1、request Cookie: 这是从浏览器发给服务器的cookie信息
    2、response Cookie: 这是服务端最初响应浏览器请求时发给浏览器设置cookie的信息
    ```

  - cookie的覆盖：

    ```
    如果服务器端发送重复的Cookie那么会覆盖原有的Cookie，例如客户端的第一个请求服务器端发送的Cookie是：Set-Cookie: a=A；第二请求服务器端发送的是：Set-Cookie: a=AA，那么客户端只留下一个Cookie，即：a=AA。（字典的键唯一）
    ```

  - cookie的特性及意义

    ```
     特性：
     	1. 由服务器让浏览器进行设置的。
    	2. 浏览器保存在浏览器本地上。
    	3. 下次访问时会自动携带相应的cookie
    	
    为什么有cookie？
    	http协议是无状态的。每次http请求都是对立的，相互之间没有关联。用cookie保存状态。
    ```
  
  
  
- **cookie的登陆验证**：

  - 我们之前写过的一些小网页，有一个问题就是，只要我们输入网址就能访问，这就很困扰，谁都能访问，我们只需要登陆或则会员才能让其访问怎么办，这就要用到我们的cookie了，也就是当用户登陆某网页时，通过其携带的cookie进行判断，其是否登陆，没有登陆则跳转到登陆界面，登陆了修改cookie后再次访问即可。

    ```python
     # login.hmtl文件
    <body>
    <form action="" method="post">
        {% csrf_token %}
        <p>
            用户名: <input type="text" name="username">
        </p>
        <p>
            密码: <input type="password" name="pwd"> <span>{{ error }}</span>
        </p>
        <button>提交</button>
    </form>
    </body>
    
    --------------------------------------------------
     # url.py 文件中
    urlpatterns = [
        url(r'^admin/', admin.site.urls),
        url(r'^login/',views.Login.as_view()),
        url(r'^index/',views.index),
        url(r'^home/',views.home),
        url(r'^loginout/',views.logout),
    ]
    
    --------------------------------------------------
     # views.py文件
    from django.shortcuts import render,redirect,HttpResponse
    from django.views import View
    
    class Login(View):
        
        def get(self,request,*args,**kwargs):
            return render(request,'login.html')
        
        def post(self,request,*args,**kwargs):
            username = request.POST.get('username')
            pwd = request.POST.get('pwd')
            if username == 'alex' and pwd == '123':  # 校验成功后，让浏览器修改cookie值
                url = request.GET.get('return_url')  # 这里取出从哪来的参数，后面跳转会原位置，如home
                if url:
                    ret = redirect(url)  # ret为HttpResponse是HttpResponse对象，我们需要给他塞点cookie
                else:  # 因为可能时/login/来访问
                    ret = redirect('/index/')
                ret.set_cookie('is_login','1')  # 设置给浏览器的最初cookie值，给它分一个牌子，下次浩认得
                return ret  # 对应的浏览器的response cookie中有有一个cookie，然后浏览器就会保存，下次再来访问就会带着这个牌子，浏览器认得；这里先让浏览器设置了cookie值，然后带着cookie重定向到index
            return render(request,'login.html',{'error':'用户名密码错误'})
    
    def login_required(func):
        def inner(request, *args, **kwargs):
            is_login = request.COOKIES.get('is_login') # 之前服务器给浏览器的cookie已经保存在COOKIES中，以后每次请求都携带会携带这个字典
            url = request.path_info
            if is_login != '1':    # 判断这个值来决定 是否跳转 到首页
                return redirect(f'/login/?return_url={url}')  # 这里给每个跳来登录的请求记录为参数在路径中，登陆成功后各回各家
            ret = func(request, *args, **kwargs)      
            return ret
        return inner
            
    @login_required
    def index(request):
        return HttpResponse('首页')
    
    @login_required
    def home(request):  # 这是用户的家目录，换句话说也是要登陆才能进入，问题1：都需要登陆，我们就需要用装饰器
        return HttpResponse('home')   # 问题2：当我最初访问home页面，需要登陆，但是登陆成功后会却跳转到了首页，应该跳转到home才是？，这就需要在访问home时，传递一个参数（装饰器中的重定向） 
    
    def logout(request):  # 删除cookie
        ret = redirect('/login/') # 注销后跳转到登陆页面
        ret.delete_cookie('is_login')
        return ret
    
     # 总结：
        1、在HttpResponse对象中设置cookie值给浏览器标识： redirect(url).set_cookie('is_login','1') # 键值对
        2、服务器获取浏览器请求头中携带来的cookie信息：request.COOKIE.get('key')
        3、Cookie是服务器给浏览器，并让它保存的一对键值对。
        4、Cookie不是Django中的，是Http头中有的。
    ```

  

- **设置Cookie**：

  ```python
  rep = HttpResponse(...)
  rep = render(request, ...) 
  rep = redirect(request,...)  # 以上都是HttpResponse对象中塞入cookie字典
  
  rep.set_cookie(key,value,...) #等价于：rep['Set-Cookie'] ='is_login=1; Path=/'
  rep.set_signed_cookie(key,value,salt='加密盐',...) # 可加密
  
   # set_signed_cookie响应的参数说明：
  key, 键
  value='', 值
  max_age=None, 超时时间，超过时间cookie失效；默认None是关闭浏览器就失效
  expires=None, 超时时间(IE requires expires, so set it if hasn't been already.)  # 这适用于IE
  path='/', Cookie生效的路径，/ 表示根路径，特殊的：根路径的cookie可以被任何url的页面访问，固定固定路径下是可以访问的
  domain=None, Cookie生效的域名，和path类似
  secure=False, https传输，非http不能传输
  httponly=False 只能http协议传输，无法被JavaScript获取（不是绝对，底层抓包可以获取到也可以被覆盖）
   #如: ret.set_signed_cookie('is_login','1','s21',max_age=1000,secure=True)          
  ```

- **获取Cookie**：

  ```python
  request.COOKIES['key']
  request.get_signed_cookie('key', default=RAISE_ERROR, salt='', max_age=None)
  
   # get_signed_cookie方法的参数说明：
  default: 默认值
  salt: 加密盐
  max_age: 后台控制过期时间
  ```

- **删除Cookie**;

  ```PYTHON
  def logout(request):
      rep = redirect("/login/")
      rep.delete_cookie("user")  # 删除用户浏览器上之前设置的user的cookie值,
      return rep
  ```


- Cookie版登陆验证

  ```python
  from functools import wraps
  def check_login(func):
      @wraps(func)
      def inner(request, *args, **kwargs):
          next_url = request.get_full_path()
          if request.session.get("user"):
              return func(request, *args, **kwargs)
          else:
              return redirect("/login/?next={}".format(next_url))
      return inner
  
  def login(request):
      if request.method == "POST":
          user = request.POST.get("user")
          pwd = request.POST.get("pwd")
          if user == "alex" and pwd == "alex1234":
              # 设置session
              request.session["user"] = user
              # 获取跳到登陆页面之前的URL
              next_url = request.GET.get("next")
              # 如果有，就跳转回登陆之前的URL
              if next_url:
                  return redirect(next_url)
              # 否则默认跳转到index页面
              else:
                  return redirect("/index/")
      return render(request, "login.html")
  
  @check_login
  def logout(request):
      # 删除所有当前请求相关的session
      request.session.delete()
      return redirect("/login/")
  
  
  @check_login
  def index(request):
      current_user = request.session.get("user", None)
      return render(request, "index.html", {"user": current_user})
  ```



#### Session

- **概念**：Session是服务器端技术，利用这个技术，服务器在运行时可以 为每一个用户的浏览器创建一个其独享的session对象，由于 session为用户浏览器独享，所以用户在访问服务器的web资源时 ，可以把各自的数据放在各自的session中，当用户再去访问该服务器中的其它web资源时，其它web资源再从用户各自的session中 取出数据为用户服务。

- Cookie虽然在一定程度上解决了“保持状态”的需求，但是由于Cookie本身最大支持4096字节，以及Cookie本身保存在客户端，可能被拦截或窃取，因此就需要有一种新的东西，它能支持更多的字节，并且他保存在服务器，有较高的安全性。这就是Session。问题来了，基于HTTP协议的无状态特征，服务器根本就不知道访问者是“谁”。那么上述的Cookie就起到桥接的作用。

- **总结而言**：Cookie弥补了HTTP无状态的不足，让服务器知道来的人是“谁”；但是Cookie以文本的形式保存在本地，自身安全性较差；所以我们就通过Cookie识别不同的用户，对应的在Session里保存私密的信息以及超过4096字节的文本。**session就是保存在服务器的一组键值对**

  ![1561541968888](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1561541968888.png)

  ```
   # 总结：
   session：保存在服务器上的一组键值对
   
   #为什么要有session？
   1.cookie保存在浏览器本地，不安全
   2.cookie大小个数受到限制
   
   # 只要我们执行python manage.py migrate 就会创建一个会话表，因为在setting中有这个应用，就会生成一个表
  ```

- **session于Cookie的不同点**：

  ```
  1、Cookie最初是有服务器给浏览器的一条键值对，用于标识一个浏览器（因为本身Http是无状态的)，换句话说，Cookie是最初由HttpResponse对象返回给浏览器，而后每次浏览器请求的request里都有Cookie。
  
  2、session是一次浏览器和服务器的会话，能进入会话状态是基于Cookie唯一标识了一个浏览器（cookie相当于一个人的电话号码，session是一次打电话的过程的记录的话题箱），session的设置和取值都是在request.session中，可以随时打印一下，所以当我们设置它值的时候，响应数据库中的django_session表中就会添加记录（类似的用法还有crsf_token).
  
  3、session必须依赖cookie
  4、同一浏览器，同一个cookie标识，所以一个浏览器无论登陆几次或则几个窗口打开，在session表中只有一条记录，因为cookie一样，也就是随机字符串的key只有一个。
  ```

- cookie和session的关系

  ```
  1.cookie
  定义: cookie是服务器让浏览器保存在浏览器本地的键值对
  原因: http 是无状态,每次请求之间没有任何关系,无法保存状态. 使用cookie 来保存一些状态
  
  2.session
  定义: session 是保存在服务器上的键值对,依赖于cookie
  原因:
  cookie 在浏览器端  不太安全
  cookie 长度受限   session长度不受限
  ```

  

- **更加详细的解读session**:

  - 这里我们做个比喻：浏览器中去请求服务器时(假如此时cookie为空，还未设置），此时服务器收到请求，假如此时设置session，那么第一时间会生成一组字典，key时随机字符串，value是我们设置session对象的值（内嵌字典），通过HttpResponse回复一个sessionid ：key 给浏览器（夹带在cookie中），浏览器记录下来，以后一段时间都会打折这个sessionid去服务器，服务器会根据这个id去打开响应的数据。

    ![1561548874937](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1561548874937.png)

- 相应的各种方法：（和字典用法一样）

  ```python
   # 获取、设置、删除Session中数据
  request.session['k1']
  request.session.get('k1',None)
  request.session['k1'] = 123
  request.session.setdefault('k1',123) # 存在则不设置
  del request.session['k1']
  
  
   # 所有 键、值、键值对
  request.session.keys()
  request.session.values()
  request.session.items()
  
   # 会话session的key
  request.session.session_key
  
   # 将所有Session失效日期小于当前日期的数据删除，默认是两周，但是数据不会则自动清空，除非执行此方法
  request.session.clear_expired()
  
   # 检查会话session的key在数据库中是否存在
  request.session.exists("session_key")
  
   # 删除当前会话的所有Session数据，cookie还在，但是数据没有了
  request.session.delete()
  　　
   # 删除当前的会话数据并删除会话的Cookie。同时删除
  request.session.flush() 
      这用于确保前面的会话数据不可以再次被用户的浏览器访问
      例如，django.contrib.auth.logout() 函数中就会调用它。
  
   # 设置会话Session和Cookie的超时时间
  request.session.set_expiry(value)
      * 如果value是个整数，session会在些秒数后失效。
      * 如果value是个datatime或timedelta，session就会在这个时间后失效。
      * 如果value是0,用户关闭浏览器session就会失效。
      * 如果value是None,session会依赖全局session失效策略。
  ```

- **session版登陆验证**：更安全可靠

  ```python
  from functools import wraps
  
  def check_login(func):
      @wraps(func)
      def inner(request, *args, **kwargs):
          next_url = request.get_full_path()
          if request.session.get("user"):
              return func(request, *args, **kwargs)
          else:
              return redirect("/login/?next={}".format(next_url))
      return inner
  
  
  def login(request):
      if request.method == "POST":
          user = request.POST.get("user")
          pwd = request.POST.get("pwd")
  
          if user == "alex" and pwd == "alex1234":
              # 设置session
              request.session["user"] = user
              # 获取跳到登陆页面之前的URL
              next_url = request.GET.get("next")
              # 如果有，就跳转回登陆之前的URL
              if next_url:
                  return redirect(next_url)
              # 否则默认跳转到index页面
              else:
                  return redirect("/index/")
      return render(request, "login.html")
  
  
  @check_login
  def logout(request):
      # 删除所有当前请求相关的session
      request.session.delete()
      return redirect("/login/")
  
  
  @check_login
  def index(request):
      current_user = request.session.get("user", None)
      return render(request, "index.html", {"user": current_user})
  ```

  

- **Django中的Session配置**

  ```python
   #Django中默认支持Session，其内部提供了5种类型的Session供开发者使用。
   1. 数据库Session
  SESSION_ENGINE = 'django.contrib.sessions.backends.db'   # 引擎（默认）
  
  2. 缓存Session
  SESSION_ENGINE = 'django.contrib.sessions.backends.cache'  # 引擎
  SESSION_CACHE_ALIAS = 'default'                            # 使用的缓存别名（默认内存缓存，也可以是memcache），此处别名依赖缓存的设置
  
  3. 文件Session
  SESSION_ENGINE = 'django.contrib.sessions.backends.file'    # 引擎
  SESSION_FILE_PATH = None                                    # 缓存文件路径，如果为None，则使用tempfile模块获取一个临时地址tempfile.gettempdir() 
  
  4. 缓存+数据库
  SESSION_ENGINE = 'django.contrib.sessions.backends.cached_db'        # 引擎
  
  5. 加密Cookie Session
  SESSION_ENGINE = 'django.contrib.sessions.backends.signed_cookies'   # 引擎
  
  其他公用设置项：
  SESSION_COOKIE_NAME ＝ "sessionid"                       # Session的cookie保存在浏览器上时的key，即：sessionid＝随机字符串（默认）
  SESSION_COOKIE_PATH ＝ "/"                               # Session的cookie保存的路径（默认）
  SESSION_COOKIE_DOMAIN = None                             # Session的cookie保存的域名（默认）
  SESSION_COOKIE_SECURE = False                            # 是否Https传输cookie（默认）
  SESSION_COOKIE_HTTPONLY = True                           # 是否Session的cookie只支持http传输（默认）
  SESSION_COOKIE_AGE = 1209600                             # Session的cookie失效日期（2周）（默认）
  SESSION_EXPIRE_AT_BROWSER_CLOSE = False                  # 是否关闭浏览器使得Session过期（默认）
  SESSION_SAVE_EVERY_REQUEST = False                       # 是否每次请求都保存Session，默认修改之后才保存（默认）
  ```

- ##### session可以存储在那些地方：

  ```
  数据库、redis、内存
  ```

  