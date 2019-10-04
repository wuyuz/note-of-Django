

### Django之反向解析和视图函数

#### 一、反向解析

- 在使用Django 项目时，一个常见的需求是获得URL 的最终形式，以用于嵌入到生成的内容中（视图中和显示给用户的URL等）或者用于处理服务器端的导航（重定向等）。人们强烈希望不要硬编码这些URL（费力、不可扩展且容易产生错误）或者设计一种与URLconf 毫不相关的专门的URL 生成机制，因为这样容易导致一定程度上产生过期的URL。在需要URL 的地方，对于不同层级，Django 提供不同的工具用于URL 反查：

  ```
  两种方式：（其实就是用某种方法：将别名反向解析成URL路径）
  1、在模板中：使用url 模板标签。
  2、在Python 代码中：使用django.core.urls.reverse 函数。
  ```

  也就是说：反向解析就是针对html模块和views函数两两之间的相互查找，的一种映射。之前我们通过在标签中嵌套地址来调用函数，现在我们也可以使用函数来具体的改变标签。

  

- **反向解析一**： 使用标签{% url '别名'  参数 %}  （也就是在html中，使用{% url  '别名 ' %}  的别名，在url中通过别名总之找路径）

  - url.py中要设置函数的别名，注意：这个设置le别名的URL路径是针对全局（后面我们会使用命名空间限制），也就是说你设置到总路由urls和分发的路由url都是可以的

    ```python
    #urls中的代码（总路由）
    from django.conf.urls import url,include
    from app import views
    
    urlpatterns = [
        url(r'^index/$',views.index),
        url(r'^app/',include('app.url')),    
    ]
    
    #url中的代码
    from django.conf.urls import url
    from app import views
    
    urlpatterns =[
        url(r'^views/',views.index),
        url(r'^([0-9]{4})/([0-9]{2})/([0-9]+)/$',views.special_case，name = 'special'),
        url(r'^test/$',views.test,name='test'),  #设置别名
        url(r'^blog/(?P<year>[0-9]{4})/$', views.blogs,name = 'blog'),
    ]
    ```
    
  - 在template包中的html文件中标签的引用方式

    ```
      <html lang="en">
      <head>
          <meta charset="UTF-8">
          <title>Title</title>
      </head>
      <body>
      <form action="{% url 'test'%}" method="post">   //注意写法
          <input type="submit" value="测试">
      </form>
          <h4>当前时间:{{ ctime }}</h4>
      </bod>
      </html>
      
      -----------------------------------------------------------------
      //多参数传入 (无名分组 --> 位置参数)
       <form action="{% url 'special' 2018 12 1 %}" method="post">   //注意写法
          <input type="submit" value="测试">
      </form>
      
      -----------------------------------------------------------------
      //多参数传入（命名参数 --> 关键字参数）
       <form action="{% url 'blog' year = 2019 %}" method="post">   //注意写法
          <input type="submit" value="测试">
      </form>
    ```
  
   
  
  - 对应views.py中的函数
  
    ```python
    from django.shortcuts import render,HttpResponse
    
    def index(request):
        import datetime
        now = datetime.datetime.now()
        ctime = now.strftime("%Y-%m-%d %X")
        return render(request,'index.html',{"ctime":ctime})  # 类似于jinja2中的模块渲染，使模块动态起来
    
    def test(request):
        return HttpResponse(b"Test Ok!")
    ```
  
  

![1](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1.gif)

- **注意**：可以给这种形式的反向解析传递参数：url 会将这个参数和别名对应的路径进行拼接，（然后触发别名对应得函数）

  ```python
  \<form action="{% url 'test' book.pk %}" method="post">   #假如这个test别名对应得函数（无名参数）需要一个参数，这里演示了从个页面携带在这个页面信息book.pk跳转到另一个 'test' 对应路径的函数中（test可接收这个参数）
  ```

------



- **反向解析二**：在视图函数中通过reverse('别名')，来查找在url中该别名的路径

- **在views中使用reverse函数（反向解析）+  redirect函数（重定向）**

  - views.py文件中（通过函数别名找网址）

    ```python
    from django.shortcuts import render,HttpResponse,redirect
    from django.urls import reverse
    
    def test(request):
        if request.method == "POST":
            user = request.POST.get("username")
            pwd = request.POST.get('password')
            print(user)
            if user == 'wang' and pwd == '123':
                url = reverse('test2',args=(2019,13,11))  # 注意这里的参数也必须满足test2的规则，注意这里针对的是无名参数得传参
                
                url1 = reverse('test1', kwargs={'year':2019,'month':12})  # 这里针对得是命名分组，对应的是关键字传参
                print(url)     #/app/2019/13/11/
                return redirect(url)
        return HttpResponse(b"Test Ok!")
    #这主意使用reverse时，如果有参数，必须加参数得到的网址，再使用redirect重定向
    
    def special_case(request,year,month,day):
        return HttpResponse(year+month+day)
    ```

  - index.hmtl中，我们添加用户和密码框

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
    </head>
    <body>
    <form action="{% url 'test'%}" method="post">   //一样的使用最开始的使用别名找函数
        用户名： <input type="text" name="username">
        密码：  <input type="text" name="password">
        <input type="submit" value="测试">
    </form>
        <h4>当前时间:{{ ctime }}</h4>
    </body>
    </html>
    ```

  - url.py中的路径函数对应表

    ```python
    from django.conf.urls import url
    from app import views
    
    urlpatterns =[
        url(r'^test/$',views.test,name='test'),
        url(r'^app/(?P<year>[0-9]{4})/(?P<month>[0-9]{4})/$',views.test,name='test1')
    ]
    ```
    

  ![2](C:\Users\RootUser\Desktop\知识点复习\Django\gif\2.gif)

- ##### 总结

  ```python
  情况1：    
      在模板（html文件）： 
           {% url '别名' %}  #如果这里调用的函数有参数，必须加上参数（最好函数都使用命名参数）
           {% url 'app:test4' year=2019%}  #命名空间:别名
  
  情况2：
      视图函数（python文件）中
      from django.urls import reverse
      from django.shortcuts import redirect
      url=reverse(别名)  #有参数加参数  如：url = reverse('test2',args=(2019,13,11)) 
      return redirect(url)  #重定向
      
  #需要注意的是：在redirect('index/') 此时会像原来路径上拼接index/
  情况3：
    	redirect('别名')  #redirct自带reverse，且注意是有命名空间限制，有则app:别名
        
  ```



- **注意**：在模块和视图函数中如果所用的别名函数都要参数必须在后面加参数

  ```python
    #url中
   urlpatterns =[
        url(r'^views/',views.index),
        url(r'^(?P<year>[0-9]{4})/$',views.special_case1,name="test"),
    ]
    
    #html中
    <form action="{% url 'app:test4' year=2019 %}" method="post">
        用户名： <input type="text" name="username">
        密码：  <input type="text" name="password">
      <input type="submit" value="测试">
    </form>
  ```

  

  

#### 二、命名空间

- 命名空间（英语：Namespace）是表示标识符的可见范围。在上面的学习中我们知道，取别名是非常方便的，但是如果名字重了，django就不知道找谁了（会找第一个找到的，后面的就废了）。下面我们有两个项目app，app01，如果都有name别名怎么办？

  ```
  官方：一个标识符可在多个命名空间中定义，它在不同命名空间中的含义是互不相干的。这样，在一个新的命名空间中可定义任何标识符，它们不会与任何已有的标识符发生冲突，因为已有的定义都处于其它命名空间中。由于name没有作用域，Django在反解URL时，会在项目全局顺序搜索，当查找到第一个name指定URL时，立即返回。其他的就没用了。
  ```

- 修改方式：

  ```python
  #在总路由中定义命名空间，也就是说，不能人人都通过name来区分函数，应用一多，就找不到方向了，需要划分哈区域（命名空间）
  from django.conf.urls import url,include
  from django.contrib import admin
  from app import views
  
  urlpatterns = [
      # url(r'^admin/', admin.site.urls),
      url(r'^index/$',views.index),
      url(r'^app/',include('app.url',namespace='app')),
      url(r'^app01/',include('app01.url',namespace='app01'))   #取别名
  ]
  ```

- 具体的实现（两个test的相互引用）

  - 总路由就像上面设置的一样，接着时设置app的views.py

    ```python
    from django.shortcuts import render,HttpResponse,redirect
    from django.urls import reverse
    
    def test(request):
        if request.method == "POST":
            user = request.POST.get("username")
            pwd = request.POST.get('password')
            print(user)
            if user == 'wang' and pwd == '123':
                url = reverse('app01:test')  # 注意这里的参数也必须满足test的规则,没有就不写
                print(url)     #/app/2019/13/11/
                return redirect(url)
        return HttpResponse(b"Test Ok!")
    ```

  - app中的html基本就是上个例子的html，注册成功后跳转至app01的test中

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
    </head>
    <body>
    {#先跳转至app中的test#}
    <h2> This is home of app</h2>
    <form action="{% url 'app:test'%}" method="post">  //注意写法 {% url  '命名空间:别名' %}
        用户名： <input type="text" name="username">
        密码：  <input type="text" name="password">
        <input type="submit" value="测试">
    </form>
        <h4>当前时间:{{ ctime }}</h4>
    </body>
    </html>
    
    //注意当有多个命名空间时："app01:app02:xxx:index"
    ```

  - app01的路由和views.py里面的函数

    ```python
    from django.conf.urls import url
    from django.contrib import admin
    from app01 import views
    
    urlpatterns =[
        url(r'^views/',views.index),
        url(r'^([0-9]{4})/([0-9]{2})/([0-9]+)/$',views.special_case,name="test2"),
        url(r'^test/$',views.test,name='test')
    ]
    
    ----------------------------------------------------------------------------
    #views.py中代码
    from django.shortcuts import render,HttpResponse,redirect
    def test(request):
        return render(request,'index2.html')
    ```

  - app01使用的html内容

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
    </head>
    <body>
    <h2>This is home of app01</h2>
    </body>
    </html>
    ```

    

- ##### 总结

  ![3](C:\Users\RootUser\Desktop\知识点复习\Django\gif\3.gif)
  
  ```python
  我的两个app中 url名称重复了，我反转URL的时候就可以通过命名空间的名称得到我当前的URL。
  语法：
  1、模块中使用：
  	{% url 'app01:detail' pk=12 pp=99 %}
  	
  2、views.py在函数使用：v = reverse('app01:detail', kwargs={'pk':11})
	
  3、当有多级路由时，即多个include，我们可以使用：'app01:datail:xxx:index'
  ```
  
  

------

#### 三、Django的视图函数

- 一个视图函数（类），简称视图，是一个简单的Python函数（类），它接受Web请求并且返回Web响应。响应可以是一张网页的HTML内容，一个重定向，一个404错误，一个XML文档，或者一张图片。无论视图本身包含什么逻辑，都要返回响应。代码写在哪里也无所谓，只要它在你当前项目目录下面。除此之外没有更多的要求了——可以说“没有什么神奇的地方”。为了将代码放在某处，大家约定成俗将视图放置在项目（project）或应用程序（app）目录中的名为`views.py`的文件中。

  - 下面是一个简单的视图: 需要注意的是一个视图函数应该具备get、post等触发事件的判断，因为一般的情况是以GET的方式触发，但要考虑特殊性。

    ```python
    from django.http import HttpResponse   #从django.http模块导入了HttpResponse类，以及Python的datetime库
    import datetime
    
    def current_datetime(request):   # 这个request是在每次连接对话时在swgi.py中就是有的，在传入时必须接受他
        #接着，我们定义了current_datetime函数。它就是视图函数。每个视图函数都使用HttpRequest对象作为第一个参数，并且通常称之为request。
        now = datetime.datetime.now()
        html = "<html><body>It is now %s.</body></html>" % now
        return HttpResponse(html)
    	#这个视图会返回一个HttpResponse对象，其中包含生成的响应。每个视图函数都负责返回一个HttpResponse对象。无论时通过render还是其他方法，底层都是继承了HttpResponse来实现消息的回复
    ```

    ![1558347040061](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558347040061.png)

    **我们不难理解，在视图层我们的每个函数或则类，都是接受一个request（里面包括所有来访者信息），处理加工后返回一个HttpRespone回去（返回信息）**



#### 四、CBV 和 FBV

- **概念**：

  ```
  FBV（function base views） ：就是在视图里使用函数处理请求。之前都是FBV模式写的代码，所以就不写例子了。(用函数来写逻辑)
  
  CBV（class base views）    : 就是在视图里使用类处理请求。（用继承View的类来写逻辑）
  
  Python是一个面向对象的编程语言，如果只用函数来开发，有很多面向对象的优点就错失了（继承、封装、多态）。所以Django在后来加入了Class-Based-View。可以让我们用类写View。这样做的优点主要下面两种：
  	1、提高了代码的复用性，可以使用面向对象的技术，比如Mixin（多继承）
      2、可以用不同的函数针对不同的HTTP方法处理，而不是通过很多if判断，提高代码可读性
  ```

  **具体使用**：

  ```python
  #写一个处理GET方法的view，用函数写的话是下面这样 （FBV）
  from django.http import HttpResponse
  def my_view(request):
       if request.method == 'GET':
              return HttpResponse('OK')
        
      
  #class-based view写的话，就是下面这样 （CBV）
  from django.http import HttpResponse
  from django.views import View
  class MyView(View):
        def get(self, request):   #  注意View中默认是有get方法的，这里我们相当于重写了get方法
              return HttpResponse('OK')
  ```

  ​		Django的**url是将一个请求分配给可调用的函数的方法**，而不是一个class。（源码）针对这个问题，class-based view提供了一个`as_view()`静态方法（也就是类方法），调用这个方法，会创建一个类的实例，然后通过实例调用`dispatch()`方法，`dispatch()`方法会根据request的method的不同调用相应的方法来处理request（如`get() `,` post()`等）。到这里，这些方法和function-based view差不多了，要接收request，得到一个response返回。如果方法没有定义，会抛出HttpResponseNotAllowed异常。
  
  ```python
  #下面是扩展内容，也就是CBV是怎么实现的，也就是说重写的dispatch是get、post等8种请求方式得入口。
  class Login(View):
       
      def dispatch(self, request, *args, **kwargs):
          print('before')
          obj = super(Login,self).dispatch(request, *args, **kwargs)
          print('after')
          return obj
   
      def get(self,request):
          return render(request,'login.html')
   
      def post(self,request):
          print(request.POST.get('user'))
          return HttpResponse('Login.post')
  ```
  
  
  
  **CBV源码分析**：
  
  ```python
  1、项目启动 --> 请求访问加载的url.py文件的路径 --> 触发views.py中的 类.as_view():
      #View类中：
  	def as_view(cls,):
  		def view(request,*args,**kwargs):
              self = cls(**initkwarys)  # 这里就是在实例化类
              self.request = request
              ……
              return self.dispatch(request,*args,**kwargs) # 这里就是所以请求的入口，我们可以自定义
          return view  # 执行as_views方法返回一个view内层函数，其实就和我们之前写一个函数名没调用一样，返回的都是函数名，其实执行这个函数，会触发dispatch函数，dispatch函数会用到反射来获取我们调用的方式。
      
  2、请求到来的时候执行view函数 ：
  	1、实例化类 --> self
      2、执行seld.dispatch(request, *args, **kwargs)：
      	2.1、判断请求方式是否被允许：
          		允许，则通过反射获取到对应请求方式的方法   ——》 获得handler，也就是我们get方法
              	不允许，self.http_method_not_allowed  ——》handler
          2.2、 执行handler（request，*args,**kwargs）
          
    #View类中的dispatch函数：
  		http_method_names = ['get', 'post', 'put', 'patch', 'delete', 'head', 'options', 'trace']
  	    def dispatch(self, request, *args, **kwargs):
              if request.method.lower() in self.http_method_names:
                  handler = getattr(self, request.method.lower(), self.http_method_not_allowed) #self为对象，对象里的方法，所以 get(self,request) 就相当于 self.get(request)
              else:
                  handler = self.http_method_not_allowed
              return handler(request, *args, **kwargs)  #需要注意的时这时候时对象.handler就不需要self了
          
   #注意：这里需要注意的是，View类中为我们写好了八种请求方式，当我们只想允许get来请求时，我们可以：
  class Add_student(View):
      http_method_names = ['get',]  # 这里相当于重写了http_method_names
      
      def get(self,request):pass
      
  ---------------------------------------------------------------------
  梳理整个流程：as_view() --> 返回内层view --> 执行dispatch --> 执行请求方法   （每次都有返回值，重点，一层层往上翻）
  ```
  
  
  
  **注意**：使用CBV时，url.py中也可以做出对应的修改：
  
  ```python
  # urls.py
  from django.conf.urls import url
  from myapp.views import MyView #引入我们在views.py里面创建的类
    
  urlpatterns = [
       url(r'^index/$', MyView.as_view()),
  ]
  ```
  
- CBV传参，和FBV（前一章又讲）类似，有名分组，无名分组：

  ```python
  url(r'^cv/(\d{2})/', views.Myd.as_view(),name='cv'),
  url(r'^cv/(?P<n>\d{2})/', views.Myd.as_view(name='xxx'),name='cv'),	#如果想给类的name属性赋值xxx，前提你的Myd类里面必须有name属性（类属性，定义init方法来接受属性行不通，但是可以自行研究一下，看看如何行通，意义不大），并且之前类里面的name属性的值会被覆盖掉,后面的name是一个别名，用于反向解析
  
  -----------------------------------------
  #views中类的写法：
  class Myd(View):
      name = 'sb'
      def get(self,request,n):
          print('get方法执行了')
          print('>>>',n)
          return render(request,'cvpost.html',{'name':self.name})
  
      def post(self,request,n):
          print('post方法被执行了')
          return HttpResponse('post')
      
  #添加类的属性可以通过两种方法设置，第一种是常见的Python的方法，可以被子类覆盖。
  from django.http import HttpResponse
  from django.views import View
  
  class GreetingView(View):
      name = "yuan"
      def get(self, request):
           return HttpResponse(self.name)
    
  # You can override that in a subclass 
  class MorningGreetingView(GreetingView):
      name= "alex"
  
  -------------------------------------------------------------    
  #第二种方法，你也可以在url中指定类的属性：在url中设置类的属性Python 
  urlpatterns = [
     url(r'^index/$', GreetingView.as_view(name="egon")), #类里面必须有name属性，并且会被传进来的这个属性值给覆盖掉
  ]
  ```



- **案例补充**：将函数式和类式进行对比

  ```python
  from django.shortcuts import render,redirect,HttpResponse
  from app import  models
  from django.views import View
  import time
  
  def student_list(request):
      student_obj = models.Student.objects.all()
      return render(request,'student_list.html',{'list':student_obj})
  
  def timer(func):
      def inner(*args, **kwargs):
          strat_timer = time.time()
          ret = func(*args,**kwargs)  # 将要接收到get方法最终的返回值，因为最后执行的相当于inner函数，它得有HttpResponse返回值
          end_timer = time.time()
          print(f"函数执行时间：{end_timer-strat_timer}")
          return ret  # 这里相当于把最终的结果返回        
      return inner
  
  class Student_list(View):
      @timer
      def get(self,request):
          student_obj = models.Student.objects.all()
          return render(request, 'student_list.html', {'list': student_obj})
  
  def add_student(request):
      error_msg = ''
      class_obj_list = models.Class.objects.all()
      if request.method == 'POST':
          student_name = request.POST.get('student')
          class_id = request.POST.get('class_id')
          if student_name:
              models.Student.objects.create(name = student_name,classes_id=class_id)
              return  redirect('/app/student_list/')
          else:
              error_msg = '信息不能为空'           
      return render(request,'add_student.html',{'error_msg':error_msg,'list':class_obj_list})
  
  class Add_student(View):
      def get(self,request):
          error_msg = ''
          class_obj_list = models.Class.objects.all()
          return render(request, 'add_student.html', {'error_msg': error_msg, 'list': class_obj_list})
      
      def post(self,request):
          student_name = request.POST.get('student')
          class_id = request.POST.get('class_id')
          
          # 保存图片：注意这里有一个上传图片的实例
          f1 = request.FILES.get('f1')
          print(f1)
          with open(f1.name,'wb') as f:
              for i in f1.chunks():  #chunks：块的意思
                  f.write(i)
        
          if student_name:
              models.Student.objects.create(name=student_name, classes_id=class_id)
              return redirect('/app/student_list/')
          else:
              error_msg = '信息不能为空'
              class_obj_list = models.Class.objects.all()
          return render(request, 'add_student.html', {'error_msg': error_msg, 'list': class_obj_list})
      
  -------------------------------------------------------------------------------------------
   #对应得html文件：需要注意得是，上传文件得form表单需要调整编码方式 enctype="multipart/form-data"
  <form class="form-horizontal" action="" method="post" enctype="multipart/form-data">
              <div class="form-group">
                  <label for="1" class="col-sm-2 control-label">姓名</label>
                  <div class="col-sm-10">
                      <input type="text" id='1' class="form-control" name="student" placeholder="{{ student_obj.name }}">
                  </div>
                  <label for="class" class="col-sm-2 control-label">班级</label>
                  <div class="col-sm-10" style="margin-top:10px;">
                      <select name="class_id" id="class">
                          {% for class in list %}
                              <option value="{{ class.id }}">{{ class.name }}</option>
                          {% endfor %}
                      </select>
                  </div>
                  <label for="f1" class="col-sm-2 control-label">头像</label>
                  <div class="col-sm-5" style="margin-top:10px;">
                      <input type="file" name = 'f1' class="danger" style="display: inline-block">
                  </div>
              </div>
  
              <div class="form-group">
                  <div class="col-sm-offset-2 col-sm-10 text-danger">
                      {{ error_msg }}
                      <button type="submit" class="btn btn-primary center-block">提交</button>
                  </div>
              </div>
          </form>
  
  -----------------------------------------------------------------------------------------
  #url文件
  urlpatterns = [
      url(r'^student_list/$',views.Student_list.as_view()),
      url(r'^add_student/$',views.Add_student.as_view()),
  ]
  ```

  

#### 五、给视图加装饰器

- **使用装饰器装饰FBV**：FBV本身就是一个函数，所以和普通函数加装饰器无异

  ```python
  def wrapper(func):
      def inner(*args, **kwargs):
          start_time = time.time()
          ret = func(*args, **kwargs)
          end_time = time.time()
          print("used:", end_time-start_time)
          return ret
      return inner
  
  
  # FBV版添加班级
  @wrapper
  def add_class(request):
      if request.method == "POST":
          class_name = request.POST.get("class_name")
          models.Classes.objects.create(name=class_name)
          return redirect("/class_list/")
      return render(request, "add_class.html")
  ```

- **使用装饰器装饰CBV**：类中的方法与独立函数不完全相同，因此不能直接将函数装饰器应用于类中的方法 ，我们需要先将其转换为方法装饰器。Django中提供了method_decorator装饰器用于将函数装饰器转换为方法装饰器。

  ```python
  from django.views import View
  from django.utils.decorators import method_decorator
  
  class AddClass(View):
  
      @method_decorator(wrapper)
      def get(self, request):
          return render(request, "add_class.html")
  
      def post(self, request):
          class_name = request.POST.get("class_name")
          models.Classes.objects.create(name=class_name)
          return redirect("/class_list/")
      
  ------------------------------------------------------
  #另外给cbv添加装饰器的有以下三种方式：
  
  #1、直接添加在dispatch里面，这样每个函数都会执行，即每个请求都会进入前执行一次，执行完执行一次
  from django.utils.decorators import method_decorator
  
  @method_decorator(login_test)
  def dispatch(self, request, *args, **kwargs):
  	res = super(IndexView, self).dispatch(request, *args, **kwargs)
  	return res  #一定要有这个返回值，因为这是get、post返回值得最后一道关口
  
  #2、添加在每一个函数中，只作用于当前请求方式
  from django.utils.decorators import method_decorator
  
  	@method_decorator(login_test)
  	def get(self, request, *args, **kwargs):
  		return render(request, 'index.html')
  
  #3、直接添加在类上，后面的name表示只给get添加装饰器
  from django.utils.decorators import method_decorator
  
  #@method_decorator(login_test, name='get')  get是给get方法加（以这种方式如果想给多个方法加装饰器，需要写多层装饰器，因为name这个参数的值必须是个字符串，并且不能同时写两个方法），login_test表示要加得装饰器，不写name参数则会给所有的请求方式加上装饰器
  @method_decorator(login_test, name='post') # post是给post方法加，可单独使用name来约束
  class IndexView(View):
     	 def get(self,request)：
      	pass
          return 'xxx'
          
  #4、重叠加在类上：
  @method_decorator(timer,name='post')
  @method_decorator(timer,name='get')
  class AddPublisher(View):
  
  ---------------------------------------------------------------------------------
  # 其实不使用method_decorator方法也可以给我们要作用得请求方式添加装饰器，他们得区别是：
  不适用method_decorator：
      func   ——》 <function AddPublisher.get at 0x000001FC8C358598>
      args  ——》 (<app01.views.AddPublisher object at 0x000001FC8C432C50>, <WSGIRequest: GET '/add_publisher/'>)
  
  使用method_decorator之后：
      func ——》 <function method_decorator.<locals>._dec.<locals>._wrapper.<locals>.bound_func at 0x0000015185F7C0D0>
      args ——》 (<WSGIRequest: GET '/add_publisher/'>,)
  ```

- 关于装饰器的综合案例

  ```python
  #views.py里面的内容：
  from django.shortcuts import render,redirect,HttpResponse
  from django.urls import reverse
  from django.utils.decorators import method_decorator
  from django.views import View
  def wrapper(fn):
      def inner(request,*args,**kwargs):
          print('xxxxx')
          ret = fn(request)
          print('xsssss')
          return ret
      return inner
  
  # @method_decorator(wrapper,name='get')#CBV版装饰器方式一
  class BookList(View):
      @method_decorator(wrapper) #CBV版装饰器方式二
      def dispatch(self, request, *args, **kwargs):
          print('请求内容处理开始')
          res = super().dispatch(request, *args, **kwargs)
          print('处理结束')
          return res
      
      def get(self,request):
          print('get内容')
          # all_books = models.Book.objects.all()
          return render(request,'login.html')
      
      @method_decorator(wrapper) #CBV版装饰器方式三
      def post(self,request):
          print('post内容')
          return redirect(reverse('book_list'))
  # @wrapper
  def book_list(request):
      return HttpResponse('aaa')
      
  -----------------------------------------------------
  #urls.py文件中
  url(r'^book_list/', views.book_list,name='book_list'),
  url(r'^booklist/', views.BookList.as_view(),name='booklist'), 
  
  -----------------------------------------------------
  #login.html文件内容
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
  
  <form action="/booklist/" method="post">
      {% csrf_token %}
      用户名：<input type="text" name="username">
      <input type="submit">
  
  </form>
  </body>
  </html>
  
  #注意事项:
  1、添加装饰器前必须导入from django.utils.decorators import method_decorator
  2、添加装饰器的格式必须为@method_decorator()，括号里面为装饰器的函数名
  给类添加是必须声明name
  3、注意csrf-token装饰器的特殊性，在CBV模式下它只能加在dispatch上面(后面再说)
  
  　　下面这是csrf_token的装饰器：
  　　@csrf_protect，为当前函数强制设置防跨站请求伪造功能，即便settings中没有设置csrfToken全局中间件。
  　　@csrf_exempt，取消当前函数防跨站请求伪造功能，即便settings中设置了全局中间件。
  
  注意导入的语法：
  from django.views.decorators.csrf import csrf_exempt,csrf_protect
  　　
  ```
  


- ##### 案例演示：

  - 在视图函数中我们可以结合路由传参和反射来整合代码，比如在学生、教师以及班级模型中，删除函数单一，我们就可以进行整合，（需要注意的时：在使用CBV、和FBV进行传参时，有些需要注意的地方）如下：

    ```python
     # url.py文件中：
    from django.conf.urls import url
    from app import views
    
    urlpatterns = [
        url(r'^(student|class|teacher)_list/$', views.list_func), 
        url(r'^del_(student|class|teacher)/(\d+)/$',views.del_func),
        url(r'^add_(?P<name>student|class|teacher)/$',views.Add_Func.as_view()), # 接受的无名参数将触发get、post函数的一个参数，必须相应的接收参数。
        # url(r'^add_(?P<targe>student|class|teacher)/$', views.Add_Func.as_view()),  # 接受的无名参数将触发get、post函数的一个参数,名字必须时targe    
    ] 
    
    --------------------对应的视图函数views.py---------------------------
    from django.shortcuts import render,redirect,HttpResponse
    from app import  models
    from django.views import View
    
    # 原本的FBV，要写三个del_xxx
    def del_student(request):
        student_id = request.GET.get('id')
        del_list = models.Student.objects.filter(id=student_id)
        if del_list:
            del_list.delete()
            return redirect('/app/student_list/')
        else:
            return HttpResponse('删除错误')
        
    #整合删除方法 （反射）
    def del_func(request, table, get_id):
        print(table.capitalize(),get_id)
        table_name = getattr(models,table.capitalize())
        del_list = table_name.objects.filter(id = get_id)
        if del_list:
            del_list.delete()
            return redirect('/app/student_list/')
        else: return  HttpResponse('删除错误')
    
    #整合首页方法
    def list_func(request, name):
        table_name = getattr(models, name.capitalize())
        obj = table_name.objects.all()
        return render(request,f"{name}_list.html",{'list':obj})
    
    #整合所有的添加方法
    class Add_Func(View):
        error_msg = ''
        @timer
        def dispatch(self, request, name, *args, **kwargs): # 注意如果这里你不写name，将会被arg没收
            ret = super().dispatch(request,name)  # 注意要还给每个参数中都传入这个name
            return ret  # 每个方法都要有return
        
        def get(self,request,name):
            class_obj_list = models.Class.objects.all()
            return render(request, f'add_{name}.html', {'error_msg': self.error_msg, 'list': class_obj_list})
        
        def post(self, request, name):
            obj_name = request.POST.get(name)
            class_id = request.POST.get('class_id')
            cls = getattr(models, name.capitalize())
            
            # 保存图片
            if name == 'student':
                f1 = request.FILES.get('f1')
                if obj_name and not f1:
                    with open(f1.name, 'wb') as f:
                        for i in f1.chunks():  # chunks：块的意思
                            f.write(i)
                            
                    cls.objects.create(name=obj_name, classes_id=class_id)
                    return redirect(f'/app/{name}_list/')
                else:
                    self.error_msg = '信息和头像不能为空'
                    class_obj_list = cls.objects.all()
                return render(request, f'add_{name}.html', {'error_msg': self.error_msg, 'list': class_obj_list})
            
            elif name == 'class':
                class_name = request.POST.get('class')
                if class_name:
                    if models.Class.objects.filter(name=class_name):
                        self.error_msg = '班级信息已经存在'
                    else:
                        models.Class.objects.create(name=class_name)
                        return redirect('/app/class_list/')
                else:
                    self.error_msg = '信息不能为空'
                return render(request, 'add_class.html', {'error':self.error_msg})
            
            elif name == 'teacher':
                class_obj_list = models.Class.objects.all()
                if request.method == 'POST':
                    teacher_name = request.POST.get('teacher')
                    class_ids = request.POST.getlist('class_id')
                    if teacher_name:
                        new_teacher_obj = models.Teacher.objects.create(name=teacher_name)
                        new_teacher_obj.classes.set(class_ids)  # set加列表，给这个关系管理表重新赋值
                        return redirect('/app/teacher_list/')
                    else:      
                        self.error_msg = '老师姓名不能为空'
                return render(request, 'add_teacher.html', {'list': class_obj_list, 'error': self.error_msg})
                
    ```
    
