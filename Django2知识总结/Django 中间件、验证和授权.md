## Django 中间件、验证和授权



中间件是Django框架级别的钩子，是对request和response处理过程中的一个插件，比如在request到达视图函数之前，我们可以使用中间件处理一些事情，在返回response之前也可以做一些事情。我们之前使用中间件哪是通过重写类的五种方法



#### 自定义中间件

这里我们通过另一种方式来自定义中间件

- 使用函数的中间件

  在app下创建一个middleware.py文件,相当于装饰器，那么最先调用的方法是内置的middleware函数，也就是说装饰器的所装饰的对象是一个get_reponse函数，这里传入

  ```python
  def app_middleware(get_response):
      def middleware(request):
      
          print('reqeust 到达view之前执行的代码')
          # 在这个代码执行之前，都是request到view之前的代码
          response = get_response(request)
          print('response 到达浏览器之前执行的代码')
          # response 对象到达浏览器之前执行的代码
  
          return response
      return middleware
  ```

  视图函数

  ```python
  def index(request):
      print('views执行中...')
      return HttpResponse('OK')
  ```

  注册到中间件

  ```PYTHON
  MIDDLEWARE = [
  	...
      'app.middlewares.app_middleware',
  ]
  ```

  执行结果

  ```
  reqeust 到达view之前执行的代码
  views执行中...
  response 到达浏览器之前执行的代码
  ```

- 使用类来自定义中间件

  同样在app/middleware.py文件中, 用法一样

  ```python
  class AppMiddleware:
  
  	def __init__ (self,get_response):
  		print('之前初始化之前diamond')
  		self.get_response = get_response
  	
  	def __call__(self,request):
          
  		print('reqeust 到达view之前执行的代码')
          # 在这个代码执行之前，都是request到view之前的代码
          response = self.get_response(request)
          print('response 到达浏览器之前执行的代码')
          # response 对象到达浏览器之前执行的代码
  
          return response
  ```

- 老式方法,会被遗弃

  ```python
  from django.utils.deprecation import MiddlewareMixin
  
  class AppMiddlewareMixin(MiddlewareMixin):
  	
  	def __init__(self,get_response):
  		print('执行之前初始化...')
  		super().__init__(get_response)
  	
      # 视图之前执行
  	def process_request(self,reqeust):
  		pass
  		
  	# response到达浏览器之前执行的方法
  	def process_response(self,request,response):
  		pass
  		return response	
  ```

  

#### Django内置中间件讲解

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware', # 给request中封装session对象
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',  #csrf保护中间件
    'django.contrib.auth.middleware.AuthenticationMiddleware', # 给request添加user对象
    'django.contrib.messages.middleware.MessageMiddleware', 
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

- django.middleware.common.CommonMiddleware：

  - 限制settings.DISALLOWED_USER_AGENTS中指定的请求头来访问本网站，DISALLOWED_USER_AGENTS是一个正则表达式的列表，代码如下

    ```
    import re
    DISALLOWED_USER_AGENTS = [
    	re.compile(r'^\s$|^$'),
    	re.compile(r'.*phantomjs.*')
    ]
    ```

  - 如果开发者子在定义url的时候，最后有一个斜杠，但是用于没有写，会自动添加

- django.middleware.gzip.GZipMiddleware ： 将相应数据进行压缩，小于200长度，不会压缩

- django.middleware.security.SecurityMiddleware： 做一些安全处理的中间件，比如xss防御的请求头



#### Django的验证和授权

Djnago有一个内置的授权系统，他用来处理用户、分组、权限以及基于cookie的会话系统。Django中包括验证和授权，验证时验证这个用户是否时他声称的人，授权时给与他相应的权限。

Django内置的权限系统：

```
1、用户
2、权限
3、分组
4、一个可以配置的密码哈希系统
5、一个可插拔的后台管理系统
```



#### 使用授权系统

默认中创建一个django项目后，其实就已经集成了授权系统，哪那些部分跟授权系统有关？

- INSTALL_APPS:

  ```
  1、django.contrib.auth： 包含了一个核心授权框架，以及大部分模型定义
  2、django.contrib.contenttypes:Content Type系统，可以用来关联模型和权限
  ```

- 中间件

  ```
  1、sessionMiddleware :用来管理session
  2、AuthenticationMiddleware : 用来处理和当前session相关的用户
  ```

  

#### User模型

user模型是这个框架的核心（通过INSTALL_APPS中的auth实现），他的完整路径是在 django.contrib.auth.models.User 。以下对这个User对象做简单了解：

- 字段：内置的User模型有以下字段

  ```
  1、username: 用户名，150长度，不能为空且唯一
  2、fist_name、last_name: 姓、名；可以为空
  3、email：邮箱，可为空
  4、password：密码，经过hash加密，不能为空，该字段是继承AbstractBaseUser的
  5、group：分组，它跟Group表一对多
  6、user_permissions:权限，一个用户可以用多个权限，和Permission表属于多对多的关系
  7、is_staff: 是否可以进入admin站点
  8、is_active: 是否还可用，该用户
  9、is_superuser:是否是超级管理员
  10、last_loign: 上次登陆，该字段是继承AbstractBaseUser的
  11、date_joined:账号创建时间
  ```

- 基本使用

  创建用户

  ```python
  #views.py
  from django.http import HttpResponse
  from django.contrib.auth.models import User
  
  def index(request):
      # 这里create_superuser() 可创建超级管理员
      user = User.objects.create_user(username='wang',email='xxx@qq.com',password='111')
  
      return HttpResponse('success')
  ```

  修改密码

  ```python
  def index(request):
      user = User.objects.get(pk=1)
      # 注意密码是加密的，所以只能调用set_password设置
      user.set_password('xxxx')
      return HttpResponse('success')
  ```

  登陆验证

  ```python
  from django.contrib.auth import authenticate
  
  def index(request):
      user = authenticate(username='xxx',password='sss')
      if user:
          print('OK')
      return HttpResponse('success')
  ```

- 自定义验证

  ```python
  from django.http import HttpResponse
  from django.contrib.auth.models import User
  from django.contrib.auth import authenticate  #系统的验证
  
  # def index(request):
      # User.objects.create_user(username='xxx',password='123')
      # return HttpResponse('success')
  
  def my_auth(username,password):
      user = User.objects.filter(username=username).first()
      if user:
          is_current = user.check_password(password)  # 调用系统的这个方法可以自定义验证
          print('is_corrent:%s'%is_current)
          if is_current:
              return user
          else:
              return None
  
  def index(request):
      user = my_auth('xxx','123')
      if user:
          return HttpResponse('OK')
      else:
          return HttpResponse('NO')
  ```

  

#### 扩展用户模型

Django内置的user有时候不能满足我们的需求，比如在验证用户登陆的时候，他用的是用户名作为验证，而我们有时候需要通过手机号验证，还有比如user的字段不够用，我们需要添加字段，这时候就需要使用扩展用户模型

，有以下方式：

- 设置Proxy模型：

  如果你对djngo提供的字段不太满意，需要新增，可通过以下方式

  ```python
  from django.contrib.auth.models import User
  class Person(User):  # 前提导入了User
  	
  	class Meta:
  		proxy=True
  		
  	def get_blacklist(self):
  		return self.objects.filter(is_active=False)
  ```

  以上的代码，Person继承了User，通过Meta中设置proxy=True,说明这是User的一个代理模型，并不会影响原来的User模型在数据库中的表，而且当我们如果想获取user表中的所有黑名单时：Person.get_blacklist()就可以了

- 一对一外键：

  如果你对用户验证方法authenticate不满意，因为它只能通过用户名登陆，我想通过手机号登陆怎么办？这时候我们可以在原来模型基础上新增字段，使用一对一外键方式连接：

  ```python
  from django.contrib.auth.models import User
  from django.db import models
  from django.dispatch import receiver  #接受信号的函数receiver
  from django.db.models.signals import post_save #使用信号来触发相互绑定
  
  class UserExtension(models.Model):
  	user = models.OneToOneField(User,on_delete=models.CASCADE,related_name='extension')
  	birthday = models.DateField(null=True,blank=True)
  	school = models.CharField(max_length=20)
  	
  @receiver(post_save,sender=User) # 只要User触发了save方法
  def create_user_extension(sender,instance,created,**kwargs): #instance实例，即存储对象
  	if created: # 创建
  		UserExtension.objects.create(user=instance)
  	else:  #更新
  		instance.extension.save()
  ```

  以上定义的UserExtension模型，并于user表一对一绑定，以后我们新增字段到UserExtension模型上，并且还写了接受保存模型的信号凡是，只要user调用了save方法，那么就会创建一个一对一的绑定

- 继承子AbstractUser

  如果对authencicate不满意，可以直接继承AbstractUser

  ```python
  from django.contrib.auth.models import AbstractUser
  
  class User(AbstractUser):
  	telephone = models.CharField(max_length=10,unique=True)
  	school = models.CharField(max_length=10)
  	
  	USERNAME='telephone'  # 偷梁换柱，之后就是验证手机号了
  	REQUIRED_FIELDS=['username','password']
  	
  	# 重写objects
  	objects = UserManager()
  ```

  

#### 权限部分

https://www.bilibili.com/video/av61533158/?p=165