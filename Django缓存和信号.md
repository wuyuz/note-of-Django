# day121笔记

## 一.今日安排

```
django-debug-toolbar
缓存
信号
orm性能相关
多个数据库
crm+权限
```

## 二.今日内容

### 1.django-debug-toolbar

- 介绍
  
- django-debug-toolbar 是一组可配置的面板，可显示有关当前请求/响应的各种调试信息，并在单击时显示有关面板内容的更多详细信息。返回HttpResponse会失效
  
- 安装：

  ```
  pip3 install django-debug-toolbar
  ```

  

- settings 配置

```python
#将 debug_toolbar 添加到 INSTALL_APPS 中
INSTALLED_APPS = [
    ...
	'debug_toolbar',
]

#如果是本机调试，还在将127.0.0.1加入 INTERNAL_IPS
INTERNAL_IPS = ("127.0.0.1",)
#在中间件中加入DebugToolbarMiddleware
MIDDLEWARE = [
    ...
	'debug_toolbar.middleware.DebugToolbarMiddleware',
]

#配置jQuery的URL
	#django-debug-toolbar 默认使用的是Google的地址，默认配置如下：
    JQUERY_URL = '//ajax.googleapis.com/ajax/libs/jquery/2.2.4/jquery.min.js'
    #国内用不了的话可以在settings.py中配置一下，例如（我这里特意选用了和原作者相同版本的jQuery）：
	DEBUG_TOOLBAR_CONFIG = {
    "JQUERY_URL": '//cdn.bootcss.com/jquery/2.2.4/jquery.min.js',
}
```

- urls.py中

```python
from django.conf import settings

if settings.DEBUG:#根据setting中的debug来判断，是否开启
    import debug_toolbar

    urlpatterns = [
                      url(r'^__debug__/', include(debug_toolbar.urls)),
                  ] + urlpatterns
```



### 2.缓存

- 由于Django是动态网站，所有每次请求均会去数据进行相应的操作，当程序访问量大时，耗时必然会更加明显，最简单解决方式是使用：缓存，缓存将一个某个views的返回值保存至内存或者memcache中，5分钟内再有人来访问时，则不再去执行view中的操作，而是直接从内存或者memcache中之前缓存的内容拿到，并返回。 

- 开发调试--->起到占位作用，本身不具备缓存。等上线之后更改配置即可使用

  ```
  不做任何缓存。咦？不做任何缓存？没听错吧，那干嘛要用它呢？
  
  因为是开发调试模式，在本地进行调试，调试过程中，所有的相关缓存配置都需要加上，但是自己调试时候不需要加配置(效果半小时失效，不能干等半个小时看效果吧)，要实时看结果。先起到占位作用，等到上线，再改配置就可以使用了。
  ```

- 内存

- 文件

- 数据库

- Memcache缓存(python-memcached模式)

- Memcache缓存(pylibmc模块)

- www.cnblogs.com/maple-shaw/articles/7563029.html

#### 2.1 基于内存进行缓存的配置

```python
1.settings.py

# 此缓存将内容保存至内存的变量中
# 配置：
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
        'LOCATION': 'unique-snowflake',
        'TIMEOUT': 300,         # 缓存超时时间（默认300，None表示永不过期，0表示立即过期）
        'OPTIONS':{
            'MAX_ENTRIES': 300,  # 最大缓存个数（默认300）
            'CULL_FREQUENCY': 3, # 缓存到达最大个数之后，剔除缓存个数的比例，即：1/CULL_FREQUENCY（默认3）
        },
    }
}

# 注：其他配置同开发调试版本

	
2.给视图加缓存
from django.views.decorators.cache import cache_page
@cache_page(5)#装饰器cache_timeout=5表示缓存超时时间5秒
def student_list(request):
    students = models.Student.objects.all()
    print("打印代表没缓存")
    return render(request,'stu.html',{"students":students})
#5秒内除了第一次，多次访问是没有打印结果，代表不走缓存
```

#### 2.2 基于文件进行缓存的配置

```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
        'LOCATION': 'G:\homework\day复习篇\day121Django\缓存\cache',
        'TIMEOUT': 300,         # 缓存超时时间（默认300，None表示永不过期，0表示立即过期）
        'OPTIONS':{
            'MAX_ENTRIES': 300,  # 最大缓存个数（默认300）
            'CULL_FREQUENCY': 3, # 缓存到达最大个数之后，剔除缓存个数的比例，即：1/CULL_FREQUENCY（默认3）
        },
    }
}


#只是更改了'BACKEND' :'django.core.cache.backends.filebased.FileBasedCache'
#'LOCATION' 文件存储位置。生成 .djcache后缀文件
```

#### 2.3基于数据库进行缓存的配置

```python
1.
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
        'LOCATION': 'my_cache_table',
        'TIMEOUT': 300,         # 缓存超时时间（默认300，None表示永不过期，0表示立即过期）
        'OPTIONS':{
            'MAX_ENTRIES': 300,  # 最大缓存个数（默认300）
            'CULL_FREQUENCY': 3, # 缓存到达最大个数之后，剔除缓存个数的比例，即：1/CULL_FREQUENCY（默认3）
        },
    }
}

2.Terminal执行命令：
	python3 manage.py createcachetable
   	生成表：表字段cache_key,value,expires
```

#### 2.4基于Memcache进行缓存的配置

```python
#ip端口访问
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': '127.0.0.1:11211',
    }
}

#建立socket访问
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': 'unix:/tmp/memcached.sock',
    }
}  

#多个缓存ip和端口，类似分布式
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': [
            '172.19.26.240:11211',
            '172.19.26.242:11211',
        ]
    }
}
```

#### 2.5全站使用缓存

- 使用中间件，经过一系列的认证等操作，如果内容在缓存中存在，则使用FetchFromCacheMiddleware获取内容并返回给用户，当返回给用户之前，判断缓存中是否已经存在，如果不存在则UpdateCacheMiddleware会将缓存保存至缓存，从而实现全站缓存

```python
MIDDLEWARE = [
'django.middleware.cache.UpdateCacheMiddleware',#最上面
# 其他中间件...
'django.middleware.cache.FetchFromCacheMiddleware',#最下面
]
#只是添加2个中间件UpdateCacheMiddleware作用是更新缓存，FetchFromCacheMiddleware从缓存中获取数据
```

#### 2.6局部模板使用缓存

- 指的是页面返回数据，因为页面有些数据要实时看，有些不需要实时更新的。给不经常发生变化的加上缓存。
- views.py

```python
import time
from django.shortcuts import render
from app01 import models
def student_list(request):
    students = models.Student.objects.all()
    print("打印代表没缓存")
    now = time.time()
    return render(request,'stu.html',{"students":students,"now":now})#往模版传入时间
```

- 模版

```django
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<ul>
    {% for student in students %}
        <li>{{ student.name }}</li>
    {% endfor %}
</ul>
    {# 实时更新 时间#}
    {{ now }}
<br>
{# 导入缓存#}
{% load cache %}
{# 设置缓存 5秒 更新一次，必须还有设置一个key #}
{% cache 5 'keys'%}
缓存{{ now }}{# 内部代码5秒更新一次 #}
{% endcache %}
</body>
</html>
```

#### 2.7 django-redis配置

- <https://django-redis-chs.readthedocs.io/zh_CN/latest/>

- 本身不支持redis做缓存，通过django-redis

- 下载

  ```
  pip install django-redis
  ```

  

```python

CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        #1为redis 的 1号库
        "LOCATION": "redis://127.0.0.1:6379/1",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    }
}

```

- ### 作为 session backend 使用配置

  - Django 默认可以使用任何 cache backend 作为 session backend, 将 django-redis 作为 session 储存后端不用安装任何额外的 backend

  ```
  SESSION_ENGINE = "django.contrib.sessions.backends.cache"
  SESSION_CACHE_ALIAS = "default"
  ```

  

### 3.信号

- Django中提供了“信号调度”，用于在框架执行操作时解耦。通俗来讲，就是一些动作发生的时候，信号允许特定的发送者去提醒一些接受者。

```python
Model signals
    pre_init                    # django的model执行其构造方法前，自动触发
    post_init                   # django的model执行其构造方法后，自动触发
    pre_save                    # django的model对象保存前，自动触发
    post_save                   # django的model对象保存后，自动触发
    pre_delete                  # django的model对象删除前，自动触发
    post_delete                 # django的model对象删除后，自动触发
    m2m_changed                 # django的model中使用m2m字段操作第三张表（add,remove,clear）前后，自动触发
    class_prepared              # 程序启动时，检测已注册的app中modal类，对于每一个类，自动触发
Management signals
    pre_migrate                 # 执行migrate命令前，自动触发
    post_migrate                # 执行migrate命令后，自动触发
Request/response signals
    request_started             # 请求到来前，自动触发
    request_finished            # 请求结束后，自动触发
    got_request_exception       # 请求异常后，自动触发
Test signals
    setting_changed             # 使用test测试修改配置文件时，自动触发
    template_rendered           # 使用test测试渲染模板时，自动触发
Database Wrappers
    connection_created          # 创建数据库连接时，自动触发
```

#### 1.信号用法一：

- 对于Django内置的信号，仅需注册指定信号，当程序执行相应操作时，自动触发注册函数，**注册信号，写入与project同名的文件夹下的`__init__.py`文件中**，也是换数据库引擎的地方。这里拿post_save举例

  - `__init__.py`

  ```python
  # post_save:django的model对象保存后，自动触发
  from django.db.models.signals import post_save
  def callback(sender,**kwargs):
      print("执行post_save信号")
      print(sender,kwargs)
  
  post_save.connect(callback)#信号连接，并调用回调函数
  ```

  - views.py

  ```python
  def student_list(request):
      students = models.Student.objects.all()
      print(students)
      models.Student.objects.create(name='xxoo')#创建一个对象，用于触发信号
      return render(request,'stu.html',{"students":students})
  ```

  ![1570170498978](C:\Users\wanglixing\Desktop\知识点复习\串讲笔记\1570170498978.png)

#### 2.信号用法二：

- 通过装饰器receiver,也可以添加多个信号

```python
from django.db.models.signals import post_save
from django.dispatch import receiver
@receiver(post_save)# django的model对象保存后，自动触发
def callback(sender,**kwargs):
    print("执行post_save信号")
    print(sender,kwargs)

post_save.connect(callback)

@receiver(pre_save)# django的model对象保存前，自动触发
def callback2(sender,**kwargs):
    print("执行pre_save信号")
    print(sender,kwargs)

pre_save.connect(callback2)

```

#### 3.自定义信号：

- 在一个py文件定义信号！sig.py

```python
import django.dispatch
pizza_done = django.dispatch.Signal(providing_args=["toppings", "size"])#toppings,size自己定义字段
```

- 在`__init__.py`中注册信号

```python
from sig import pizza_done
def callback(sender, **kwargs):
    print("callback")
    print(sender, kwargs)


pizza_done.connect(callback)

#触发后打印结果：
callback

seven {'signal': <django.dispatch.dispatcher.Signal object at 0x000001BCD6D82B38>, 'toppings': 123, 'size': 456}

```

- 在视图函数触发信号**由于内置信号的触发者已经集成到Django中，所以其会自动调用，而对于自定义信号则需要开发者在任意位置触发**

  ```python
  from django.shortcuts import render
  from app01 import models
  
  from sig import pizza_done
  
  def student_list(request):
  	students = models.Student.objects.all()
  	pizza_done.send(sender='seven', toppings=123, size=456)#给sig中定义的字段传值，发起信号者赋值
  	return render(request,'stu.html',{"students":students})
  ```

  

```
触发信号：单独写文件，如果在视图函数写函数，当代码取消，不方便。如果单独写函数，虽然添加信号会繁琐，但功能不需要取消时候，就方便许多。
```



### 4.orm性能相关

- 尽量不用对象进行查询，多用values
- select_related('关联外键字段')   连表查询    用于多对一，一对一
- prefetch_related('关联外键字段')     子表查询    用于多对一，多对多



- 只要返回某个字段。只是查某些字段   only('name')
- 排除某些字段，defer("name") 

- QuerySet特性

```python
1. 尽量不查对象，会跨表或多语句查询，用values直接取字段值，跨表一条语句，且结果为字典
 
    查询指定字段值时，尽量不用对象，而使用values直接取值
    def index(request):
        ret = models.Student.objects.all()  #获取所有对象
        for i in ret:
            print(i.name,i.classname)   #这里要跨表查询对象的外键关联数据值时，orm会每一个对象都发一次sql查询，效率降低
        return render(request,'index.html',{'students':ret,})
    #核心问题时我们通过对象点出来外键关联属性，我们可以不用all查询对象，而是用values直接获取到我们要的属性和值
    def index(request):
        ret = models.Student.objects.values('name','classname')  #获取所有数据
        for i in ret:
            print(i[name],i[classname])   #这里values得到的是字典，字典取值即可，此时orm会只发一次sql做了一次连表查询，效率高
        return render(request,'index.html',{'students':ret,})

2. 对象查询时， 一对一、多对一获取关联对象时，使用select_related(‘外键字段’) 使多条语句sql合并为一条链表查询sql
   ret = models.Student.objects.all().select_related('classes')
   这条语句会在查询对象时，通过使用外键，仍能连表查询，效率提高

3. 对象查询时，多对一多对多时使用prefetch_related(’外键‘)，把多条sql合并使用子查询，减少查询次数
   ret = models.Student.objects.all().prefetch_related('classes')

4. 当对象查询指定字段值时，在orm后添加.only(‘指定字段’)，得到的仍然是对象，与value不同的是value得到的是字典
   ret = models.Student.objects.all().only('name')

5. 当对象查询排除指定字段时，在orm后添加.defer(‘指定字段’)，得到的仍然是对象，与value不同的是value得到的是字典
   ret = models.Student.objects.all().defer('name')
6. query_set特性
   只有在使用时才会查询数据库，而不是遇到查询语句就执行，比如html对象页面渲染
```

  

### 5.库的配置

#### 1.读写分离

- settings配置

```python
#settings.py 配置库信息，生成2个库

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    },
    'db2': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db2.sqlite3'),
    },
}

python3 manage.py makemigrations   #生成迁移文件
python3 manage.py migrate --database db2 #db2数据库迁移


```

- 手动操作

```python
ret = model.Student.objects.all().using('db2')#查询db2库所有数据

models.Student.object.using("default").create(name='xxoo')#往default库写入信息

models.Student.object.using('db2').update()

#db2查询name='ss'，并更改name,并保存在default库中
obj = models.Stundet.objects.using('db2').get(name='ss')
obj.name = "sha"
obj.save(using="default")  # 默认obj.save() 储存在db2中
```

- 自动操作

```python
#myrouter.py

class Router:
	def db_for_write(self,model,**kwargs):
		return "db2"

	def db_for_read(self,model,**kwargs):
		return "default"
#settings.py配置
DATABASE_ROUTERS=['myrouter.Router',]

```

#### 2.一主多从

```python
#myrouter.py

class Router:
	def db_for_write(self,model,**kwargs):
		return "default"

	def db_for_read(self,model,**kwargs):
		return random.choice(['db2','db3',"db4"])
#settings.py配置
DATABASE_ROUTERS=['myrouter.Router',]

```

#### 3.分库分表

- 分库：  

```python
#settings.py配置
#myrouter.py
class Router:
    """
    app01   model  db1
    app02   model  db2
    """
	def db_for_write(self,model,**kwargs):
         app_name = model._meta.app_label#此操作能获得app的名字
         if app_name == "app01":
            return "db1"
        elif app_name == "app02":
			return "app02"

	def db_for_read(self,model,**kwargs):
		app_name = model._meta.app_label#此操作能获得app的名字
        if app_name == "app01":
            return "db1"
        elif app_name == "app02":
			return "app02"
```



#### 4.Django执行原生SQL

- 单起一个文件

```python
import os
import django
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "temp.settings")
djnago.setup()
#以上配置django环境

#extra方法：
from app01 import models
#从Student，找id大于1的对象值
ret = models.Student.object.all().extra(where=['id>%s'],params=['1']).values()
print(ret)

#raw方法：
ret = models.Student.objects.raw('select * from main.app01_classes')
print(ret)
for i in ret:
    print(i)#拿到学生对象
#connection方法：
from django.db import connections
#cursor = connections.cursor()
cursor = connections['db2'].cursor()#指向db2库
cursor.execute("""select * from main.app_classes where id=%s""",[1,])
row = cursor.fetchall()
print(row)
```





### 6.CRM+RBAC

- 客户关系管理系统

#### 1.功能

```
1.注册和登录
2.销售：
	- 客户信息展示
		公户/私户
	- 跟进记录的管理
	- 报名表的管理
	- 缴费记录的管理
3.班主任
	- 班级管理
	- 课程记录管理
	- 学生学习记录的管理
4.公户和私户
	避免抢单的情况
	有没有现成CRM系统？学邦
	http://www.xuebangsoft.com/case.html
	自己可开发可定制化程度高，当公司业务发生变化，可以根据自己公司业务需求改CRM

5.项目中用多少张表：
	CRM  10多张表：用户表，校区表，部门表，客户表，班级表，跟进记录表，报名表，缴费记录表，课程记录表，学习记录表。

权限：
1.如何实现权限控制？
	在web开发中，url地址当作权限
2.表结构设计：
	6张表(4个model)：
3.登录成功保存用户权限 + 中间件校验用户权限


4.表结构：
	

技术点：
	1.登录成功后保存权限信息：
		- 查询  跨表查询 去空 去重
		- 用户信息存储session
        	- session  list()  
	2.中间件：
		url  request.path_info
		*白名单：
			- settings 谁知白名单
			- re正则匹配
        *登录状态校验
        *免认证的校验
        *权限校验：
        	从session中获取权限信息
        	for循环 + re 校验
        *检验不通过最后返回HttpResponse('没有权限')

```

- 数据结构

```
#简单权限控制
permission=[{"permission":},{}]
```



- **需求发生变化（动态生成一级菜单）**

```
生成一级菜单：
Permission 
	增加字段 title  标题
			is_menu   是否是菜单
```

- 数据结构变化

```
permission_list = [
	{
		url:xxx
	}
]
menu_list = [{'url':url,'title':title,'icon':icon}...]
```

- 动态生成一级菜单

```
include_tag
```

- **需求发生变化（动态生成二级菜单）**

```
生成一个表
Menu   为一级菜单
 
原来 Permission表  为二级菜单
外键关联Menu  

#

```

- 数据结构：

```
menu_dict = {
	一级菜单的id:{
		title:'xx',
		icon:'xxx'
		children:[
			{二级菜单url,二级菜单title},...
		]
	}
}
```

- **需求发生变化（排序）**

```
Menu表
	增加weight字段

排序：有序字典，sorted
```

- 数据结构

```
menu_dict  增加weight字段

中间件CSS样式控制
```

- **需求发生变化：（非菜单权限归属）**

```
Permission表
	增加parent  字段    自关联
   	有parent_id  当前的权限就是子权限   一级菜单
   	有menu_id     当前是父权限    二级菜单

中间件：
	for循环有没有父ID
	判断有没有pid
		有pid当前是子权限   request。current_menu_id
		没有pid当前权限是父权限  二级菜单
	
```

- 更改数据结构

![1570170538422](C:\Users\wanglixing\Desktop\知识点复习\串讲笔记\1570170538422.png)

#### 7.1用户权限变更后

- 当用户的权限变更后,不重新登录,怎么应用最新的权限?

```python
1.用户表加字段 session_key
2.查询用户的session数据，把最新的权限保存进去

#解决思路：
1.用户登录设置session   --->requst.session['keys'] = "values"
2.登录状态访问其他网页，获取当前登录用户session_key
	keys = request.session.session_key
3.查询Session表session_key等于keys
	from django.contrib.sessions.models import Session
    ret = Session.objects.filter(session_key=keys).first()
    获取session_data------>ret.session_data
4.解密session_data 数据
	ses_data=request.session.decode(ret.session_data)   此为dict类型
5.更改数据ses_data,并且加密
	ses_data = {"xxx":'ooo'}
     new_ses_data = request.session.encode(ses_data)
5.将最新session保存至Session数据库中
	ret.session_data = new_ses_data
	ret.save()
```

#### 7.2权限控制到按钮级别,怎么控制到行级别?

- 销售等级不同看到客户也不同，高级别销售能看到更优质客户

```
增加条件表，设置外键关联权限表
设置字段condition --->标识不同客户的条件 如是否是VIP

```

#### 7.3你在开发中遇到什么印象深刻问题？

```
1.json序列化   key是数字   序列化变成字符串
2.表结构变换
3.功能上的取舍   (用户需求变更后，不重新登录怎么应用最新权限)

```

