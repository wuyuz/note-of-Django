### Django之URL路由系统

#### 一、URL配置

​	**URL配置(URLconf)就像Django 所支撑网站的目录**。它的本质是URL与要为该URL调用的视图函数之间的映射表。你就是以这种方式告诉Django，对于这个URL调用这段代码，对于那个URL调用那段代码。

- ##### 基本结构

  ```python
  from django.conf.urls import url
  #循环urlpatterns，找到对应的函数执行,匹配上一个路径就找到对应的函数执行，就不再往下循环了，并给函数传一个参数request，和wsgiref的environ类似，就是请求信息的所有内容
  urlpatterns = [
       url(正则表达式, views视图函数，参数，别名),
  ]
  
  参数说明：
      1、正则表达式：一个正则表达式字符串
      2、views视图函数：一个可调用对象，通常为一个视图函数或一个指定视图函数路径的字符串
      3、参数：可选的要传递给视图函数的默认参数（字典形式），传了就得用，不用报错
      4、别名：一个可选的name参数，类似于默认参数
      
  注意：在总路由的url中可以设置命名空间：namespace=        
  ```



  **注意**: Django 2.0版本中的路由系统已经替换成下面的写法，但是dango2.0是向下兼容1.x版本的语法的

  ```python
from django.urls import path
  
urlpatterns = [
      path('articles/2003/', views.special_case_2003),
      path('articles/<int:year>/', views.year_archive),
      path('articles/<int:year>/<int:month>/', views.month_archive),
      path('articles/<int:year>/<int:month>/<slug:slug>/', views.article_detail),
]
  ```

  

#### 二、正则表达式详解

For more complex matching requirements, you can define your own path converters.

#####无名参数（通过分组匹配的相当于位置参数参数视图函数）

- urls.py文件：下面配置的是总路由，其实后面涉及到分发路由后，下列设置适用于分发的路由中

  ```python
  from django.conf.urls import url
  from . import views
  
  urlpatterns = [
      url(r'^articles/2003/$', views.special_case_2003), #思考：如果用户想看2004、2005、2006....等，你要写一堆的url吗，是不是在articles后面写一个正则表达式/d{4}/就行啦，网址里面输入127.0.0.1:8000/articles/1999/试一下看看
      
      url(r'^articles/([0-9]{4})/$', views.year_archive), 
      url(r'^articles/([0-9]{4})/([0-9]{2})/$', views.month_archive), #思考，如果你想拿到用户输入的什么年份，并通过这个年份去数据库里面匹配对应年份的文章，你怎么办？怎么获取用户输入的年份啊，分组/(\d{4})/，一个小括号搞定
      url(r'^articles/([0-9]{4})/([0-9]{2})/([0-9]+)/$', views.article_detail),  #有多少个分组，函数就有多少个参数    
      
  ]  #类似于分组
  
  #注意：空路径往往适用于首页，但是要注意到写法，因为很容易会截断所有路径，所以必须以空开头以空结尾：url(r'^$',views.index)
  #静态路由：例如第一个路由，只能匹配单个路由的路由。
  ```

- views.py 中视图函数的写法：

  ```python
  #第一个参数必须是request，后面跟的三个参数是对应着上面分组正则匹配的每个参数的，要这主意位置，类似于位置参数，一对一传参
  def article_detail(request,year,month,day):
      return HttpResponse(year+month+day)
  ```

- **注意事项**

  ```python
  1、urlpatterns中的元素按照书写顺序从上往下逐一匹配正则表达式，一旦匹配成功则不再继续。
  2、若要从URL中捕获一个值，只需要在它周围放置一对圆括号（分组匹配）。
  3、不需要添加一个前导的反斜杠（也就是写在正则最前面的那个/），因为每个URL 都有。例如，应该是^articles 	  而不是 ^/articles。
  4、每个正则表达式前面的'r' 是可选的但是建议加上。
  5、^articles&  以什么结尾，以什么开头，严格限制路径
  6、可以多层的include来分发路由
  ```
  
- **补充说明**：

  ```python
  # 是否开启URL访问地址后面不为/跳转至带有/的路径的配置项，也就是说，如果一个网址请求没有在尾部加/,服务器会打回去（重定向），让它加上再来访问，如果不想打回去，直接断开将下面参数改为False，就正在setting最后加
  APPEND_SLASH=True  #（默认为True，会重定向,即设置为False后，不会因为你结尾没写/而给你主动加/匹配）
  
  -----------------------------------------------------------------------
  #如下面
  from django.conf.urls import url
  from app01 import views
  urlpatterns = [
          url(r'^blog/$', views.blog),
  ]
  
  #访问 http://www.example.com/blog 时，默认将网址自动转换为 http://www.example/com/blog/ 。如果在settings.py中设置了 APPEND_SLASH=False，此时我们再请求: http://www.example.com/blog 时就会提示找不到页面。
  ```
  



#### 三、分组命名匹配（相当于关键字参数传入视图函数）

- ​      上面的示例使用简单的正则表达式分组匹配（通过圆括号）来捕获URL中的值并以位置参数形式传递给视图。在更高级的用法中，可以使用分组命名匹配的正则表达式组来捕获URL中的值并以关键字参数形式传递给视图。在Python的正则表达式中，分组命名正则表达式组的语法是`(?P<name>pattern)`，其中`name`是组的名称，`pattern`是要匹配的模式。匹配出的名字会默认按顺序传给后面的函数。

  ```python
  from django.conf.urls import url
  from . import views
  
  urlpatterns = [
      url(r'^articles/2003/$', views.special_case_2003), #注意正则匹配出来的内容是字符串，即便是你在url里面写的是2003数字，匹配出来之后也是字符串
  　　 url(r'^articles/(\d{4})/$', views.year_archive),#year_archive(request,n),小括号为分组，有分组，那么这个分组得到的用户输入的内容，就会作为对应函数的位置参数传进去,别忘了形参要写两个了，明白了吗？相当于位置参数
      
      url(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),#某年的，(?P<year>[0-9]{4})这是命名参数（正则命名匹配还记得吗？），那么函数year_archive(request,year)，形参名称必须是year这个名字。而且注意如果你这个正则后面没有写$符号，即便是输入了月份路径，也会被它拦截下拉，因为它的正则也能匹配上
      url(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/$', views.month_archive),#某年某月的
      url(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/(?P<day>[0-9]{2})/$', views.article_detail), #某年某月某日的
      
    这个实现与前面的示例完全相同，只有一个细微的差别：捕获的值作为关键字参数而不是位置参数传递给视图函数。例如，针对网址： /articles/2017/12/  相当于按以下方式调用视图函数：
       views.month_archive(request, year="2017", month="12") #year和month的位置可以换，没所谓了，因为是按照名字来取数据的，还记得关键字参数吗？ 并且也可以给这个函数的参数设置默认值。上面的网址会给这个函数传递两个参数  
      
    #在实际应用中，使用分组命名匹配的方式可以让你的URLconf 更加明晰且不容易产生参数顺序问题的错误，但是有些开发人员则认为分组命名组语法太丑陋、
  ```



- **实例演示**：FBV和CBV之间的传参

  ```python
   #url.py文件中
  from django.conf.urls import url
  from app3 import views
  
  urlpatterns = [  
      #命名参数
      url(r'index/(?P<year>\d{4})/',views.Date.as_view(name = '类中必须有name')),
      url(r'index/(?P<word>\w{4})/',views.date),
      
      # 无名参数
      url(r'^index/(\d{4}|\d+)/',views.index),  # 路径是从上至下匹配，一旦合格就跳出路径匹配
      url(r'^index/(\w+|\w{4})/',views.Index.as_view()),
  ] 
  
  ---------------------------views.py视图------------------------
  from django.shortcuts import render,HttpResponse
  from django.views import View
  
  def date(request,year):
      print(year)   #  可在形参中设置默认值，但必须名字保持一致
      return HttpResponse('ok')
  
  class Date(View):
      name = ''  # 外部传入name时必须这里要有一个name
      def dispatch(self,request,year,*args,**kwargs):
          ret = super().dispatch(request,year)
          print(args, kwargs)  # () {}
          print('在源码层')
          print(self.name)  # 类中必须有name
          return ret
          
      def get(self,request,year):
          print(year)
          return HttpResponse('类方法：ok')
      
  def date(request, word):
      print(word)
      return HttpResponse('函数执行成功')
      
         
  def index(request,get_id = 1):
      print(get_id)  # 无名参数接收参数并答应，还可以设置默认参数
      return HttpResponse('ok')
  
  class Index(View):
      def dispatch(self, request, *args, **kwargs): # dispatch 分发的意思
          ret = super().dispatch(request,*args,**kwargs)
          print(args,kwargs)  #('ok',) {}
          return ret
          
      def get(self,request,wrod): # requst必须有，因为在源码hander中要使用,在调用hander(request,*args,**kwargs)时自动传参
          print(wrod)
          return  HttpResponse('nice')
             
  ```

  



##### URLconf匹配的位置：

- URLconf 在请求的URL 上查找，将它当做一个普通的Python 字符串。不包括GET和POST参数以及域名。也就是浏览器发过来的请求地址，

  ```python
  例如：http://www.example.com/myapp/ 请求中，URLconf 将查找到myapp/。
       而在http://www.example.com/myapp/?page=3 请求中， URLconf 仍将查找myapp/。也就是说会检测到特殊字符结束
              
  #也就是说：URL上的匹配规则类似于：贪婪匹配，相同的接收网址接收触发一个视图函数，且一个视图函数可以处理多种请求（大多数函数POST和GET）
  #URLconf 不检查请求的方法。换句话讲，所有的请求方法 —— 同一个URL的POST、GET、HEAD等等 —— 都将路由到相同的函数。（即请求方式并检测不到，所以他们将转到同一个文件（URLconfg，即对应的函数逻辑一致）下进行处理。
  
  #注意在使用GET方法请求网址时URL的一些规律：
  http://www.aspxfans.com:8080/news/index.asp?boardID=5&ID=24618&page=1#name
  1.协议部分：http:,在"HTTP"后面的“//”为分隔符
  2.域名部分：该URL的域名部分为“www.aspxfans.com”,去域名服务器转ip
  3.端口部分：跟在域名后面的是端口，域名和端口之间使用“:”作为分隔符
  4.虚拟目录部分：从域名后的第一个“/”开始到最后一个“/”为止，是虚拟目录部分。虚拟目录也不是一个URL必须的部分。本例中的虚拟目录是“/news/” 
  5.文件名部分：从域名后的最后一个“/”开始到“？”为止，是文件名部分，如果没有“?”,则是从域名后的最后一个“/”开始到“#”为止，是文件部分，如果没有“？”和“#”，那么从域名后的最后一个“/”开始到结束，都是文件名部分。本例中的文件名是“index.asp”。文件名部分也不是一个URL必须的部分，如果省略该部分，则使用默认的文件名
  6.锚部分：从“#”开始到最后，都是锚部分。本例中的锚部分是“name”。锚部分也不是一个URL必须的部分
  7.参数部分：从“？”开始到“#”为止之间的部分为参数部分，又称搜索部分、查询部分。本例中的参数部分为“boardID=5&ID=24618&page=1”。参数可以允许有多个参数，参数与参数之间用“&”作为分隔符。
  
  ------------------------------------------------------------------------------
  需要注意的是：
  1、request.GET.get('') #不是只能正在GET方法中有，也就是说此方法获取的是URL携带的参数，在POST中也可以获取，
  2、需要注意的是：？后面携带的是给路径携带的参数，如果多个参数就使用&分隔；且路径还是路径只是带有参数而已。
  3、当获取前端的多个值的时候，可以使用getlist，返回一个列表: request.POST.getlist('pk')
  ```

  ![1560768324820](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1560768324820.png)

  

- ##### URL的查找规律：贪婪匹配

  ```python
  #假如在app应用下
  urlpatterns = [
      url(r'test/ok/test/$',views.test)   # 此方法可以匹配到：app/ok/test/ok/test/,因为没加^
      url(r'^test/ok/test/$',views.test)  # 此方法不能匹配到: app/test/ 说明URL查找遵循贪婪匹配
  ]
  ```

- **捕获的参数永远都是字符串**：（命名参数）

  ```python
  每个在URLconf中捕获的参数都作为一个普通的Python字符串传递给视图，无论正则表达式使用的是什么匹配方式。例如，下面这行URLconf 中：（即使匹配到数字，也会返回一个转字符串）
  
  url(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),
  #传递到视图函数views.year_archive() 中的year 参数永远是一个字符串类型。
  ```

- **视图函数中指定默认值**：

  ```python
  # urls.py中
  from django.conf.urls import url
  from . import views
  
  urlpatterns = [
      url(r'^blog/$', views.page),   #^后面必需要一个/，此路由是一个静态路由，下面的是动态的
      url(r'^blog/page(?P<num>[0-9]+)/$', views.page),  #在此有两个路由使用同一个视图函数，这时就需要使用到默认参数了
  ]
  
  # views.py中，可以为num指定默认值，没有传参进来就是一个1
  def page(request, num="1"):pass
  ```
  
  - 在上面的例子中，两个URL模式指向相同的view - views.page - 但是第一个模式并没有从URL中捕获任何东西。（也就是说，在views函数中，使用的是num的默认值）。如果第一个模式匹配上了，page()函数将使用其默认参数num=“1”,如果第二个模式匹配，page()将使用正则表达式捕获到的num值（这是用的是匹配到的num）



- **include其他的URLconfs**（**也叫URL分发**，或则叫跨区域分发路由）：

  ​	  问大家一个问题，views和models文件是不是都放在每一个app应用里面了啊，而urls.py这个文件放在哪了，是不是放在项目文件夹里面了，说明什么，说明是不是所有的app都在使用它，如果你一个项目有10个应用，每个应用有100个url，那意味着你要在urls文件里面要写多少条url对应关系，并且所有的app的url都写在了这一个urls文件里面啊，这样好吗，当然也没有问题，但是耦合程度太高了，所以django在url这里给你提供了一个分发接口，叫做include。
  
  ```python
from django.conf.urls import include, url
  urlpatterns = [
     url(r'^blog/', include('blog.urls')),  # 可以包含其他的URLconfs文件
     url(r'^app01/',include('app01.urls')),  #别忘了要去app01这个应用下创建一个urls.py的文件，现在的意思是凡是以app01开头的路径请求，都让它去找app01下的urls文件中去找对应的视图函数，还要注意一点，此时这个文件里面的那个app01路径不能用$结尾，因为如果写了$，就没办法比配上app01/后面的路径了，
  ]
  
  #在url文件后可以取命名空间，来划分各个空间的别名namespace=''
  from django.conf.urls import url, include
  urlpatterns = [
    url(r'^app01/', include('app01.urls', namespace='app01')),
      url(r'^app02/', include('app02.urls', namespace='app02')),
  ]

  ------------------------------------------------------------
  #在两个应用下都有同一个别名路由：app1和app2
  from django.conf.urls import url
  from app01 import views
  app_name = 'app01'
  urlpatterns = [
      url(r'^(?P<pk>\d+)/$', views.detail, name='detail')
  ]
  
  --------------------------------------------------------------
  #其实总总的总总都是为了好反向路由，也就是说即使别名相同，也没关系，也因为不是一个世界的
  
  反向方法1，模块中：
  {% url 'app01:detail' pk=12 pp=99 %}
  反向方法2，views函数中：
  v = reverse('app01:detail', kwargs={'pk':11})  #url中使用命名参数
  
  --------------------------------------------------------------
  #跨区域分发路由
  from django.conf.urls import include, url
  urlpatterns = [
     url(r'^admin/', admin.site.urls),
     url(r'^blog/', include('blog.urls')),  # 可以包含其他区域的URLconfs文件
  ]
  
  
  #注意点：其实include就是一种路径的拼接：
  urlpatterns = [
     url(r'^blog', include('blog.urls')),  # 总路由中的路径没有/， 那么分发的路径就要有/否则会重叠
  ]
  
  urlpatterns = [
     url(r'^blog/', include('blog.urls')),  # 分发的blog应用中的urls，此时bolg前也没有/
  ]
  #一上能成功拼接的路径： 127.0.0.1：8000/blogblog/  才能访问，其实就是一种拼接
  ```
  
  ​     app01的urls.py的内容：（其实就是将全局的urls.py里面的内容copy一下，放到你在app01文件夹下创建的那个urls.py文件中，把不是这个app01应用的url给删掉就行了）
  
  ```python
  from django.conf.urls import url 
  from app01 import views
  
  urlpatterns = [
      url(r'^articles/2003/', views.special_case_2003,{'foo':'xxxxx'}), #{'foo':'xxxxx'}那么你的视图函数里面必须有个形参叫做foo来接收这种传参，且在命名参数传入的foo，优先级低于后面传入的foo
      url(r'^articles/(\d{4})/(\d{2})/', views.year_archive),
  ]
  
  -----------------------------------------------------------------------------------------
  #优先级
  urlpatterns = [
      url(r'^articles/(?<year>[0-9]{4})/', views.year_archive,{'year':2019}), #路径中的year优先级低于，后面传入的year
  ]
  ```



#### url和path的区别和原理

-  那么re_path实际上做了什么事情呢？django1.x版本使用url，2.1版本后使用path、re_path

  ```python
  import re
  result = re.search('index/','/index/xiao')
  print(result)   #<_sre.SRE_Match object; span=(1, 7), match='index/'>
  #其实re_path或则urls，相当于执行了re.search。那么要完全匹配'index/'，比如要^和$才行。
  ```
  
  ![1560752402315](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1560752402315.png)
  
  以上是app01url.py下面是总路由urls.py的代码
  
  ![1560752449048](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1560752449048.png)
  
  此时我们在访问之前你的articles相关路径的时候，就需要写上app01开头了
  
  ![1560752511387](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1560752511387.png)
  
  ​	其实相当于做了什么呢，将网址http://127.0.0.1:8000/app01/articles/2003/，里面的路径部分app01/articles/2003/，*到项目的urls.py里面匹配，匹配到了app01/，然后拿着路径剩余的部分articles/2003/去app01里面的urls.py文件里面进行匹配*，找到对应的函数执行。
  
  

**下面我们来说说 $ 号的使用:**

- **情景一**：在include分发路由时，不能使用$

  ![1560752638464](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1560752638464.png)

  ![1560752663230](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1560752663230.png)

  总结：分发/直接导入views包时，函数需要这注意的是有include就不能加$。

  

- **情景二**：如果我们想通过输入http://127.0.0.1:8000/app/，看到app01这个应用的首页，怎么办？就像我现在输入一个http://127.0.0.1:8000来查看网站的首页，也就是说我后面不加任何路径，就看你网址的首页。

  ```
    在结合上一个 'app/$'是没有办法通过include分发看到app的主页的（直接报错），而通过上面的改进方法，'^app/' 的inclue能不能行？
  ```

  ![1560752804223](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1560752804223.png)

  将r''写在最前面会怎么样？

  ​	![1560752889571](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1560752889571.png)

  ​		



  ####path的用法和url类似：

```python
  from django.urls import include, path
  urlpatterns = [
      path('polls/', include('polls.urls')),
      path('admin/', admin.site.urls),
  ]

函数 path() 具有四个参数，两个必须参数：route 和 view，两个可选参数：kwargs 和 name。现在，是时候来研究这些参数的含义了。

  语法：path(route, view, kwargs=None, name=None)

  1、route：

route 是一个匹配 URL 的准则（类似正则表达式）。当 Django 响应一个请求时，它会从 urlpatterns 的第一项开始，按顺序依次匹配列表中的项，直到找到匹配的项。这些准则不会匹配 GET 和 POST 参数或域名。例如，URLconf 在处理请求 https://www.example.com/myapp/ 时，它会尝试匹配 myapp/ 。处理请求 https://www.example.com/myapp/?page=3 时，也只会尝试匹配 myapp/。

  2、view：（也就是说参数会在这里传入HttpRequest传入的，视图函数中必须接收一个request）

当 Django 找到了一个匹配的准则，就会调用这个特定的视图函数，并传入一个 HttpRequest 对象作为第一个参数，被“捕获”的参数以关键字参数的形式传入。稍后，我们会给出一个例子。

  3、kwargs:

任意个关键字参数可以作为一个字典传递给目标视图函数。本教程中不会使用这一特性。

  4、name：

为你的 URL 取名能使你在 Django 的任意地方唯一地引用它，尤其是在模板中。这个有用的特性允许你只改一个文件就能全局地修改某个 URL 模式。以便于反向路由

```



**匹配首页正确写法**：（上面所有都是为了一句话）

  ```python
  	url(r'^$', views.index)   #以空开头，还要以空结尾，写在项目的urls.py文件里面就是项目的首页，写在应用文件夹里面的urls.py文件中，那就是app01的首页  （凡是匹配首页就要用^$) 
  ```

![1558702060124](C:\Users\RootUser\Desktop\知识点复习\Django\Django之URL路由系统.assets\1558702060124.png)

![1558702078003](C:\Users\RootUser\Desktop\知识点复习\Django\Django之URL路由系统.assets\1558702078003.png)

- **注意**：在添加新app应用时，需要去setting中添加路径，以备主程序找到相应的配置文件（可以打开App01Config，你会发现里面就是一个app名字，直接写在这里也是可以的）。

  ![1558110368313](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558110368313.png)

  

##### 传递额外的参数给视图函数

- URLconfs 具有一个钩子，让你传递一个Python 字典作为额外的参数传递给视图函数。

  `　　　　django.conf.urls.url()` 函数可以接收一个可选的第三个参数，它是一个字典，表示想要传递给视图函数的额外关键字参数。如果函数没有接受，会报错。

  ```python
  from django.conf.urls import url
  from . import views
  
  urlpatterns = [
      url(r'^blog/(?P<year>[0-9]{4})/$', views.year_archive, {'foo': 'bar'})，
      #注意，这就像一个命名分组一样，你的函数里面的必须有一个形参，形参必须叫做foo才行，如果是命名分组的url，那么foo参数写在函数的哪个位置都行，如果不是命名分组，那么都是将这个形参写在参数的最后。
  ]
  ```

  　在这个例子中，对于/blog/2005/请求，Django 将调用views.year_archive(request, year='2005', foo='bar')。这个技术在Syndication 框架中使用，来传递元数据和选项给视图。



##### 总结

- **路由系统url**

  ```python
      无名分组
      url(r'^articles/(\d{4})/(\d{1,2})/$', views.articles_year_month)
      def articles_year_month(request,n1,n2):
      	pass
      	
      有名分组
  	url(r'^articles/(?P<year>\d{4})/(?P<month>\d{1,2})/$', views.articles_year_month),
      def articles_year_month(request,year,month):
      	pass	
  	
  	默认参数
  	    def articles_year_month(request,month,year='2012'):
      	pass
      	
  	额外参数
  	url(r'^articles/(?P<year>\d{4})/(?P<month>\d{1,2})/$', views.articles_year_month,{'foo':'xxx'})
  	
  	def articles_year_month(request,month,foo,year='2012'):pass
  ```

- **路由分发include**

  ```
  项目的路由urls.py里面
  app01,app02
  1 导入include
  2 app文件夹中创建每个app的urls.py文件
  3 url(r'^app01/', include('app01.urls'))
  
  url(r'^$', views.home), #匹配首页
  ```

------

**官方解释各个路由函数**：

- `path`**(**route**,** *view***,** *kwargs=None***,** *name=None***)** 

  ```python
  #出现在Django2.0中，用于返回一个urlpatterns列表中的实例：
  from django.urls import include, path
  
  urlpatterns = [
      path('index/', views.index, name='main-view'),
      path('bio/<username>/', views.bio, name='bio'),
      path('articles/<slug:title>/', views.article, name='article-detail'),
      path('articles/<slug:title>/<int:section>/', views.section, name='article-section'),
      path('weblog/', include('blog.urls')),
      ...
  ]
  
  #说明：route是传入的一个字符串，该字符串可能包含尖括号（<username>如上所示）以捕获URL的一部分并将其作为关键字参数发送到视图（类似于url中的()分组)。尖括号可以包括转换器规范（如int部分<int:section>），其限制匹配的字符并且还可以改变传递给视图的变量的类型。也就是说，path方法可以返回一个转换数据类型的参数给视图函数
  ```

- `re_path`**(**route**,** *view***,** *kwargs=None***,** *name=None***)**

  ```python
  #出现在Django2.0中，用于返回一个urlpatterns列表中的实例：
  from django.urls import include, re_path
  
  urlpatterns = [
      re_path(r'^index/$', views.index, name='index'),
      re_path(r'^bio/(?P<username>\w+)/$', views.bio, name='bio'),
      re_path(r'^weblog/', include('blog.urls')),
      ...
  ]
  
  #说明：该route参数应该是一个字符串或 gettext_lazy()，其中包含与Python的兼容的正则表达式re模块。字符串通常使用原始字符串语法（r''），以便它们可以包含序列，\d而无需使用另一个反斜杠转义反斜杠。进行匹配时，将正则表达式中捕获的组传递给视图 - 如果组已命名，则作为命名参数，否则作为位置参数。值以字符串形式传递，不进行任何类型转换。也就是说和url类似，添加了re模块的方式匹配路径，且没有命名就以位置参数的影视返回。注意url即将消失
  ```

- `include()`

  ```
  include（module，namespace = None）
  include（pattern_list）
  include（（pattern_list，app_namespace），namespace = None）
  
  #说明：A function that takes a full Python import path to another URLconf module that should be "included" in this place（将一个python导入的路径，引到另一个路由中）
  ```

  