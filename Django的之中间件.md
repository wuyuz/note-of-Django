## Django的之中间件

#### 中间件：

​		我们在前面的课程中已经学会了给视图函数加装饰器来判断是用户是否登录，把没有登录的用户请求跳转到登录页面。我们通过给几个特定视图函数加装饰器实现了这个需求。但是以后添加的视图函数可能也需要加上装饰器，这样是不是稍微有点繁琐。学完中间件，我们就可以用更适宜的方式来实现类似给所有请求都做相同操作的功能了。

```
简而言之，一个网站每个网页就得对应一个视图函数，那么如果每个网址来访问，我们都要进行验证是否登陆，即给每个视图函数添加装饰器，显得很麻烦，所以我们可以学习以下中间件。
```

- 我们之前学过的大致流程：

  ```
  网页请求 -->  wsgi.py进行请求信息封装 -->  url.py路由分发  --> 视图函数处理（可能涉及到model.py中与数据库交互，template模块渲染等)  --> 返回到wsgi.py整合响应，返回给浏览器
  
   #其实我们中间省略掉了中间件这一环节，还记得csrf_token响应的中间件吗？
  ```

- 下面我们将加入中间件，重新绘制django流程图：

  ![1561791949244](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1561791949244.png)

  不难发现中间件是必经之地，所以一些必要的逻辑可以写在中间件中。



#### 中间件介绍

```
官方的说法：中间件是一个用来处理Django的请求和响应的框架级别的钩子。它是一个轻量、低级别的插件系统，用于在全局范围内改变Django的输入和输出。每个中间件组件都负责做一些特定的功能。但是由于其影响的是全局，所以需要谨慎使用，使用不当会影响性能。
说的直白一点中间件是帮助我们在视图函数执行之前和执行之后都可以做一些额外的操作，它本质上就是一个自定义类，类中定义了几个方法，Django框架会在处理请求的特定的时间去执行这些方法。

总结：中间件顾名思义，是介于request与response处理之间的一道处理过程，相对比较轻量级，并且在全局上改变django的输入与输出。因为改变的是全局，所以需要谨慎实用，用不好会影响到性能。
```

![1561792429704](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1561792429704.png)

#### 中间件源码部分

- 请求进来后，到达wsgi.py,执行get_wsgi_application函数 --> 执行WSGIHandler() 类的初始化函数

  ```python
    #当然在执行init函数执行，应先触发WSGIHandler()的__call__方法    
      def __init__(self, *args, **kwargs):
          super(WSGIHandler, self).__init__(*args, **kwargs)
          self.load_middleware()  #加载中间件
  ```

- 在load_middleware(）函数中对中间的6种功能函数进行加载

#### 自定义中间件：

- 我们再setting中任意打开一个自带的中间件，不难发现，所有的中间件的类，除了私有的函数，总是会有以下四个函数的身影：

  ```python
  1、process_request(self,request)  # 处理请求
  2、process_view(self, request, view_func, view_args, view_kwargs)  # 处理视图函数
  3、process_template_response(self,request,response)  #处理模板
  4、process_exception(self, request, exception)  #处理报错
  5、process_response(self, request, response)  #处理响应
  ```

-  当用户发起请求的时候会依次经过所有的的中间件，这个时候的请求时process_request,最后到达views的函数中，views函数处理后，在依次穿过中间件，这个时候是process_response（倒叙）,最后返回给请求者。

  ![1561794453887](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1561794453887.png)

**自定义一个中间件实列**：在项目中或者应用中随意创建一个文件夹（对应的在setting中配置的时候，从项目根目录进行查找到所在中间件的类），这里我在应用app下创建了一个mymiddle的文件，并在文件中创建了middleware.py文件

- 在settings.py的MIDDLEWARE配置中注册上述两个自定义中间件：

  ```python
  MIDDLEWARE = [
      'django.middleware.security.SecurityMiddleware',
      'django.contrib.sessions.middleware.SessionMiddleware',
      'django.middleware.common.CommonMiddleware',
      'django.middleware.csrf.CsrfViewMiddleware',
      'django.contrib.auth.middleware.AuthenticationMiddleware',
      'django.contrib.messages.middleware.MessageMiddleware',
      'django.middleware.clickjacking.XFrameOptionsMiddleware',
      'app.mymiddle.middleware.MD1',,  # 自定义中间件MD1，这个写的是你项目路径下的一个路径，例如，如果你放在项目下，文件夹名成为utils，那么这里应该写utils.middlewares.MD1
      'app.mymiddle.middleware.MD2',  # 自定义中间件MD2
  ]
  ```

  

##### process_request(self,request)

​        process_request有一个参数，就是request，这个request和视图函数中的request是一样的。它的返回值可以是None也可以是HttpResponse对象。(**准确来说，url也算使中间件的一部分**)

```
返回值是None的话，按正常流程继续走，交给下一个中间件处理

如果是HttpResponse对象，Django将不执行视图函数，而将回头，穿过它之前的中间件的process_response函数，最后将相应对象返回给浏览器。
```

 下面我们在我们自定义的middleware.py文件中：

```python
from django.utils.deprecation import MiddlewareMixin

class MD1(MiddlewareMixin):
    #自定义中间件，不是必须要有下面这两个方法，有request方法说明请求来了要处理，有response方法说明响应出去时需要处理，不是非要写这两个方法，如果你没写process_response方法，那么会一层一层的往上找，哪个中间件有process_response方法就将返回对象给哪个中间件
    def process_request(self, request):
        print("MD1里面的 process_request")

class MD2(MiddlewareMixin):
    def process_request(self, request):
        print("MD2里面的 process_request")
        pass
    
--------------------------------------------
 # url.py 我们使用视图函数触发，查看结果
from django.shortcuts import render,HttpResponse
def index(request):
    print('app 中的 index')
    return HttpResponse('ok') 

-------------------------结果-----------------
MD1里面的 process_request
MD2里面的 process_request
app01 中的 index视图
----------------------------------------------
 # 当我们交换在settings中的MD1、MD2注册顺序后
MD2里面的 process_request
MD1里面的 process_request
app01 中的 index视图    
```

- **总结**：process_request()的规律

  ```
  执行时间：视图函数之前
  参数：request   —— 》 和视图函数中是同一个request对象
  执行顺序：按照settings的注册的顺序  顺序执行
  返回值：
  		None ： 正常流程
  		return HttpResponse： 后面的中间的process_request、视图函数都不执行，直接执行当前中间件中的process_response方法，倒叙执行之前的中间中process_response方法。（回头）
  ```



------

##### process_response(self, request, response)

​		它有两个参数，一个是request，一个是response，request就是上述例子中一样的对象，response是视图函数返回的HttpResponse对象（可以打印id(response)）。该方法的返回值也必须是HttpResponse对象。

```python
class MD1(MiddlewareMixin):
    def process_request(self, request):
        print("MD1里面的 process_request")
        #不必须写return值,写了就终止下面的加载，回头
        
    def process_response(self, request, response):#request和response两个参数必须有，名字随便取
        print("MD1里面的 process_response")
        #print(response.__dict__['_container'][0].decode('utf-8')) #查看响应体里面的内容的方法，或者直接使用response.content也可以看到响应体里面的内容，由于response是个变量，直接点击看源码是看不到的，你打印type(response)发现是HttpResponse对象，查看这个对象的源码就知道有什么方法可以用了。
　　　　 return response  #必须有返回值，写return response  ，这个response就像一个接力棒一样
        #return HttpResponse('瞎搞') ,如果你写了这个，那么你视图返回过来的内容就被它给替代了

class MD2(MiddlewareMixin):
    def process_request(self, request):
        print("MD2里面的 process_request")
        pass

    def process_response(self, request, response): #request和response两个参数必须要有，名字随便取
        print("MD2里面的 process_response") 
        return response  #必须返回response，不然你上层的中间件就没有拿到httpresponse对象，就会报错
    
--------------------结果----------------------------
MD2里面的 process_request
MD1里面的 process_request
app01 中的 index视图
MD1里面的 process_response
MD2里面的 process_response
```

- **总结：**process_response(self, request, response)规律

  ```
  执行时间：视图函数之后
  参数：
  	request   —— 》 和视图函数中是同一个request对象
  	response   ——》  返回给浏览器响应对象
  
  执行顺序：按照注册的顺序  倒叙执行
  返回值：
  		HttpResponse：必须返回response对象		
   # 总的来说：浏览器就是来要响应的，只有拿到响应就回头，回头途中也可以能被覆盖		
  ```
  
- **实列1**：查看下面代码，写出终端打印的结果：

  ```python
   # middle.py文件中，先注册MD1
  from django.utils.deprecation import MiddlewareMixin
  from django.http import HttpResponse
  
  class MD1(MiddlewareMixin):
      
      def process_request(self, request):
          print("MD1里面的 process_request")
          # return HttpResponse('ok?') 这个如果没注释，会在此就，执行自己的response，然后回头，返回'ok',因为拿到响应了
      
      def process_response(self, request, response):
          print("MD1里面的 process_response")
          return response
      
  class MD2(MiddlewareMixin):
      
      def process_request(self, request):
          print("MD2里面的 process_request")
          return HttpResponse('ok?') #拿到数据后头
      
      def process_response(self, request, response):
          print("MD2里面的 process_response")
          return response 
  
  ------------------------------------------
   #views.py文件中
  from django.shortcuts import render,HttpResponse
  
  def index(request):
      print('index')
      return HttpResponse('ok') 
  -------------------------------------------
   # 结果
  MD1里面的 process_request
  MD2里面的 process_request
  MD2里面的 process_response
  MD1里面的 process_response 
  ```

- **实列2**：通过中间件验证用户是否登陆？

  　   之前我们做的cookie认证，都是通过在函数上面加装饰器搞的，比较麻烦，看看中间件怎么搞，如果写的是session认证的，你必须放在django自带的session中间件的下面，所以自定义中间之后，你需要注意你的中间件的摆放顺序。

  ```python
  class M1(MiddlewareMixin):
  
      def process_request(self,request):
          #设置路径白名单，只要访问的是login登陆路径，就不做这个cookie认证
          if request.path not in [reverse('login'),]:
              print('我是M1中间件') #客户端IP地址
              # return HttpResponse('sorry,没有通过我的M1中间件')
              is_login = request.COOKIES.get('is_login', False)
  
              if is_login:
                  pass
              else:
                  # return render(request,'login.html')
                  return redirect(reverse('login'))
          else:
              return None #别忘了return None，或者直接写个pass
  
      def process_response(self,request,response):
  
          print('M1响应部分')
          # print(response.__dict__['_container'][0].decode('utf-8'))
          return response
          # return HttpResponse('瞎搞')
  ```

- **实列3**：尝试一下通过中间件来控制用户的访问次数，让用户在一分钟之内不能访问我的网站超过20次。

  ```python
   # 可以再应用新建一个文件夹mymiddle，在下面创建一个middleware的py文件，然后在中间件中注册MIDDELEWARES，添加一个'app.mymiddle.middleware.MD1'，相应的middleware：
   
  from django.utils.deprecation import MiddlewareMixin
  from django.shortcuts import render,HttpResponse,redirect,reverse
  import time
  
  class MD1(MiddlewareMixin):
      def process_request(self, request):
          if request.path not in [reverse('login'), ]:  # 忽略那些不需要登陆得的页面，即视图
              print('我是M1中间件')  # 客户端IP地址
              is_login = request.session.get('user', False)
              count = request.session.get('count', False)
              old_time = request.session.get('time', False)
              if is_login and count<20:
                  now_time = time.time()
                  if now_time - old_time < 60:
                      count += 1
                  request.session['time'] = now_time  # 更新最新时间
                  request.session['count'] = count              
              else:
                  request.session.delete()
                  return redirect(reverse('login'))
          else:
              return None
  
   #响应的views.py文件中，：
  import time
  def login(request):
      if request.method=="POST":
          name = request.POST.get('user')
          pwd = request.POST.get('pwd')
          print(name,pwd)
          if models.Student.objects.filter(name = name, score= pwd).exists():
              request.session['user'] = name
              request.session['count'] = 1
              request.session['time'] = time.time()
              return redirect('/app/student/')
                  
      return render(request,'login.html') 
  
   # url.py文件
  urlpatterns = [
      url(r'student/$', views.student,name='login'),
      url(r'login/$',views.login,name='login'),
  ]    
  
  ```

  

- **流程图1**：

  ![1561822819230](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1561822819230.png)



##### process_view(self, request, view_func, view_args, view_kwargs)

​		Django会在调用视图函数之前调用process_view方法。它应该返回None或一个HttpResponse对象。 如果返回None，Django将继续处理这个请求，执行任何其他中间件的process_view方法，然后在执行相应的视图。 如果它返回一个HttpResponse对象，Django不会调用对应的视图函数。 它将执行其对应中间件的process_response方法并将应用到该HttpResponse并返回结果。

- **注意**：返回None走正常流程，要是返回HttpResponse对象，将立马回头

  ```python
  from django.utils.deprecation import MiddlewareMixin
  
  class MD1(MiddlewareMixin):  #  MD2注册在前
      def process_request(self, request):
          print("MD1里面的 process_request")
  
      def process_response(self, request, response):
          print("MD1里面的 process_response")
          return response
  
      def process_view(self, request, view_func, view_args, view_kwargs):
          print("-" * 80)
          print("MD1 中的process_view")
          print(view_func, view_func.__name__) #就是url映射到的那个视图函数，也就是说每个中间件的这个process_view已经提前拿到了要执行的那个视图函数
          ret = view_func(request) #提前执行视图函数，不用到了上图的试图函数的位置再执行，如果你视图函数有参数的话，可以这么写 view_func(request,view_args,view_kwargs) 
          return ret  #直接就在MD1中间件这里这个类的process_response给返回了，就不会去找到视图函数里面的这个函数去执行了。
  
  class MD2(MiddlewareMixin):
      def process_request(self, request):
          print("MD2里面的 process_request")
  
      def process_response(self, request, response):
          print("MD2里面的 process_response")
          return response
  
      def process_view(self, request, view_func, view_args, view_kwargs):
          print("-" * 80)
          print("MD2 中的process_view")
          print(view_func, view_func.__name__)
          
  ------------------------结果----------------------
  MD2里面的 process_request
  MD1里面的 process_request
  -------------------------------------------------------
  MD2 中的process_view
  <function index at 0x000001DE68317488> index
  -------------------------------------------------------
  MD1 中的process_view
  <function index at 0x000001DE68317488> index
  app01 中的 index视图     # 这里只有MD1通过了，所以只输出一次
  MD1里面的 process_response
  MD2里面的 process_response
  ```
  
- **总结**：process_view(self, request, view_func, view_args, view_kwargs)规律

  ```
  执行时间：视图函数之前，process_request之后
  参数：
  		request   —— 》 和视图函数中是同一个request对象
  		view_func  ——》 视图函数
  		view_args   ——》 视图函数的位置参数
  		view_kwargs  ——》 视图函数的关键字参数
  
  执行顺序：按照注册的顺序  顺序执行
  返回值：
  		None ： 正常流程
  		HttpResponse： 后面的中间的process_view、视图函数都不执行，直接执行最后一个中间件中的process_response方法，倒叙执行之前的中间中process_response方法。
  ```

- 图解：

  ![1561915332465](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1561915332465.png)



##### process_exception(self, request, exception)

​	 	这个方法只有在视图函数中出现异常了才执行，它返回的值可以是一个None也可以是一个HttpResponse对象。如果是HttpResponse对象，Django将调用模板和中间件中的process_response方法，并返回给浏览器，否则将默认处理异常。如果返回一个None，则交给下一个中间件的process_exception方法来处理异常。它的执行顺序也是按照中间件注册顺序的倒序执行。

- **程序实例**：当视图函数出错的时候，整个项目其实并没有结束，而是通过它来记录报错，如果我们不进行处理就返回前端 '大黄页'。

  ```python
   #views.py文件
  def index(request):
      print("app01 中的 index视图")
      raise ValueError("呵呵")
      return HttpResponse("O98K")    
  
  -----------------自定义middleware.py文件--------
  from django.utils.deprecation import MiddlewareMixin
  class MD1(MiddlewareMixin):
  
      def process_request(self, request):
          print("MD1里面的 process_request")
  
      def process_response(self, request, response):
          print("MD1里面的 process_response")
          return response
  
      def process_view(self, request, view_func, view_args, view_kwargs):
          print("-" * 80)
          print("MD1 中的process_view")
          print(view_func, view_func.__name__)
  
      def process_exception(self, request, exception):
          print(exception)
          print("MD1 中的process_exception")
  
  class MD2(MiddlewareMixin):
      def process_request(self, request):
          print("MD2里面的 process_request")
          pass
  
      def process_response(self, request, response):
          print("MD2里面的 process_response")
          return response
  
      def process_view(self, request, view_func, view_args, view_kwargs):
          print("-" * 80)
          print("MD2 中的process_view")
          print(view_func, view_func.__name__)
  
      def process_exception(self, request, exception):
          print(exception)
          print("MD2 中的process_exception")
          
  -------------------------结果-------------------------
   
  MD1里面的 process_request
  MD2里面的 process_request
  -------------------------------------------------------
  MD1 中的process_view
  <function index at 0x000001F6C8120AE8> index
  -------------------------------------------------------
  MD2 中的process_view
  <function index at 0x000001F6C8120AE8> index
  app01 中的 index视图
  呵呵
  MD2 中的process_exception
  呵呵
  MD1 中的process_exception
  	Internal Server Error: /index/
  	Traceback (most recent call last):...
  	ValueError: 呵呵
  MD2里面的 process_response
  MD1里面的 process_response
  ```

  

-   **实例2**：更改以下不返回None；直接返回

  ```python
   # 将上列的MD1修改
  class MD1(MiddlewareMixin):
      def process_request(self, request):
          print("MD1里面的 process_request")
  
      def process_response(self, request, response):
          print("MD1里面的 process_response")
          return response
  
      def process_view(self, request, view_func, view_args, view_kwargs):
          print("-" * 80)
          print("MD1 中的process_view")
          print(view_func, view_func.__name__)
  
      def process_exception(self, request, exception):
          print(exception)
          print("MD1 中的process_exception")
          return HttpResponse(str(exception))  # 返回一个响应对象 
   ------------------------------------------------
   # 结果
  MD1里面的 process_request
  MD1里面的 process_request
  -------------------------------------------------------
  MD1 中的process_view
  <function index at 0x000002934D440AE8> index
  -------------------------------------------------------
  MD1 中的process_view
  <function index at 0x000002934D440AE8> index
  app01 中的 index视图
  呵呵
  MD1 中的process_exception
  MD1里面的 process_response
  MD1里面的 process_response  
  ```

  

- **总结**：process_exception(self, request, exception)

  ```
  执行时间（触发条件）：视图层面有错时才执行
  参数：
  		request   —— 》 和视图函数中是同一个request对象
  		exception   ——》 错误对象
  
  执行顺序：按照注册的顺序  倒叙执行
  返回值：
  		None ： 交给下一个中间件取处理异常，都没有处理交由django处理异常
  		HttpResponse： 后面的中间的process_exception不执行，直接执行最后一个中间件中的process_response方法，倒叙执行之前的中间中process_response方法。
  ```

  

##### process_template_response(self,request,response)

​		process_template_response是在视图函数执行完成后立即执行，但是它有一个前提条件，那就是视图函数返回的对象有一个render()方法（或者表明该对象是一个TemplateResponse对象或等价方法）。

- 代码示例;

  ```python
    # middle.py文件中
  from django.utils.deprecation import MiddlewareMixin
  from django.http import HttpResponse
  
  class MD1(MiddlewareMixin):
      def process_request(self, request):
          print("MD1里面的 process_request")
  
      def process_response(self, request, response):
          print("MD1里面的 process_response")
          return response
  
      def process_view(self, request, view_func, view_args, view_kwargs):
          print("-" * 80)
          print("MD1 中的process_view")
          print(view_func, view_func.__name__)
  
      def process_exception(self, request, exception):
          print(exception)
          print("MD1 中的process_exception")
          return HttpResponse(str(exception))
  
      def process_template_response(self, request, response):
          print("MD1 中的process_template_response")
          return response
  
  
  class MD2(MiddlewareMixin):
      def process_request(self, request):
          print("MD2里面的 process_request")
  
      def process_response(self, request, response):
          print("MD2里面的 process_response")
          return response
  
      def process_view(self, request, view_func, view_args, view_kwargs):
          print("-" * 80)
          print("MD2 中的process_view")
          print(view_func, view_func.__name__)
  
      def process_exception(self, request, exception):
          print(exception)
          print("MD2 中的process_exception")
  
      def process_template_response(self, request, response):
          print("MD2 中的process_template_response")
          return response
          
   ---------------------------------------------------
   # views.py文件中
  from django.shortcuts import render,HttpResponse
  from django.template.response import TemplateResponse
  
  def index(request):
      print('index')  
      return TemplateResponse(request,'index.html',{})  #当作render使用
      
  -----------------------------------------------------
   # 结果
  MD1里面的 process_request
  MD2里面的 process_request
  -------------------------------------------------------
  MD1 中的process_view
  <function index at 0x00000223F4CC0AE8> index
  -------------------------------------------------------
  MD2 中的process_view
  <function index at 0x00000223F4CC0AE8> index
  index
  MD2 中的process_template_response
  MD1 中的process_template_response
  [29/Jun/2019 23:17:54] "GET /index/ HTTP/1.1" 200 138
  MD2里面的 process_response
  MD1里面的 process_response  
  ```

- 当然用法不止于此，我们可以更具传入的模板的内容进行校验修改数据等。

  ```python
   def process_template_response(self, request, response):
          # 可以在views中的TemplateResponse，找它爷爷级可以去修改传入的数据等
          response.context_data['name'] = '佩奇' # 修改{}传入模板的数据的name
          # response.template_name = 'index2.html'  # 修改传入的模板
          print("MD2 中的process_template_response")
          return response
  ```

  总结：

  ```
  执行时间（触发条件）：视图返回的是一个templateResponse对象
  参数：
  	request   —— 》 和视图函数中是同一个request对象
  	response   ——》  templateResponse对象
  执行顺序：按照注册的顺序  倒叙执行
  返回值：
  		HttpResponse：必须返回response对象
  ```

- **综上五种方法的图解**：

  ![1561893331463](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1561893331463.png)