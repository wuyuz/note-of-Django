## Session存储机制、上下文



默认情况下，session数据是存储在数据库中的，当然我们可以讲session存到别的地方，可以通过设置设置SESSION_ENGINE 来更session的存储位置，可以有以下方案：

- django.contrib.sessions.backends.db: 使用数据库
- django.contrib.sessions.backends.file : 使用文件存储session
- django.contrib.sessions.backends.cache:  使用缓存来存储session，快但是危险，因为一挂机数据就没有了，必须使用memcached这种方式
- django.contrib.sessions.backends.cached_db: 比较安全，且快，因为是结合数据库+缓存，机器挂了，数据库上
- django.contrib.sessions.backends.signed_cookies: 讲session加载到浏览器的cookie中（加密），类似于Flask，建议设置SESSION_COOKIE_HTTPONLY=True,防止浏览器操作js修改session数据，并且需要使用在settings.py中设置SECRET_KEY进行加密



#### 如何选择Memcached作为session的缓存

- 前提开启了memcached

  ```python
  # settings.py文件中
  ...
  CACHES = {
      'default': {
          'BACKEND':'django.core.cache.backends.memcached.MemcachedCache',
          'LOCATION':[
          	'127.0.0.1:11211',   # 分布式存储
          ]
      }
  }
  
  SESSION_ENGINE = 'django.contrib.sessions.backends.cached_db'
  ```

- 测试memcached是否有数据

  ```
  stats items   # 检测数据，没有
  END
  
  存两份，一份数据库一份缓存
  ```

  

#### 上下文处理器(context_processers)

上下文处理器是可以返回一些数据，在全局模板中都可以使用，比如登陆后的用户信息，在很多页面中都需要使用，那么我们可以放在上下文处理器中，就没有必要每个视图函数中都返回这个对象

在settings.TEMPLATES.OPTIONS.context_processors中，有很多内置的上下文处理器，这些上下文处理器的左右如下：

- django.template.context_processors.debug : 添加一个debug和sql_queries变量，在模板中可以通过它来查看一些数据库查询

- django.template.context_processors.request: 添加一个request变量，这个reqeust变量也就是在视图函数的第一个参数

- django.contrib.auth.context_processors.auth: Django有内置的用户系统，这个上下文处理器会添加一个user对象。

- django.contrib.messages.context_processors.messages： 添加一个messages变量，类似于Flask的g

  ```python
  后端要是用message，需要导入
  from django.contrib import messages
  
  def index(request):
  	...
  	message.info(request,'xxx错误')  # info表示级别，全局可以使用了模板
  ```

  

还记得我们在之前学过的知识中，使用Django的模板的时候，可以直接使用request吗？后端并没有传入模板，然而却可以直接使用，这就是因为Django的上下文的实现

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')]
        ,
        'APP_DIRS': True,
        'OPTIONS': {   # 已经给我们注册号以下全局使用的变量
            'context_processors': [
                'django.template.context_processors.debug', # 处于debug模式时，返回sql查询代码
                'django.template.context_processors.request', # 可在模板中使用request
                'django.contrib.auth.context_processors.auth', #自动生成user对象
                'django.contrib.messages.context_processors.messages',
                'app.context.app_user',  # 这个是我们自己在app下写的一个上下文内容，用于返回登陆用户的信息
            ],
        },
    },
]
```

- app/context.py

  ```python
  from .models import User
  def app_user(request):
      context = {}
      user_id = request.session.get('user_id')
      if user_id:
          try:
              user = User.objects.get(pk=user_id)
              context['app_user'] = user
          except:
              pass
  
      return context
  ```

  注册好上下文内容后，之后我们就不必写如下代码了

  ```python
  #views.py
  def index(request):
      user_id = request.session.get('user_id')
      context = {}
      try:
          user=User.objects.get(pk=user_id)
          context['app_user'] =user #不再需要传入app_user了,因为模板可以直接调用
      except:
          pass
      return render(request, 'index.html',context=context)  
  ```

  模板中可以直接使用，不需要后端每个视图传入app_user

  ```python
                  {% if app_user %}
                      <li><a href="#">{{ app_user.username }}</a></li>
                  {% else %}
                      <li><a href="{% url 'signup' %}">注册</a></li>
                      <li><a href="{% url 'signin' %}">登陆</a></li>
                  {% endif %}
  ```

  