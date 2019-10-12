## Django2 基础回顾



#### 常见标签

- if 标签

  ```
  {% if '张三' in persons %}
   	张三
  {%else%}
   	李四
  {%endfor%}
  ```

- for ... in 标签

  ```
  {% for person in persons reversed%}   //反转
  	{{person}}
  {% endfor%}
  ```

- with 标签：在模板中定义变量

  ```
  content ={
  	"person":['李四'，'王五']
  }
  
  {% with person.1 as lis %}
  	{{lis}}  //表示李四
  {% endwith %}
  ```

- url 标签：在模板中，我们经常写url来进行反向解析

  ```
  <a href="{% url 'books:list' %}"  //books命名空间下的list别名url
  ```

- verbatim标签： 我们常使用的{{}}、{%%}都是有特殊意义的，那么可以通过它来取消特殊意义

  ```
  {% verbatim %}
  	{{if 你好}}  // 取消特殊意义
  {% endverbatim %}
  ```

  

#### 常见过滤器

模板中有时候需要对一些数据进行处理才能使用

- date： 将一个日期按照指定的格式输出

  ```
  context={
  	"birthday":datetime.now()
  }
  
  {{birthday|date:"Y/m/d"}}
  {{birthday|date:"Y/m/d H:i:s"}}
  ```

- default :设置默认值,当变量为False的时候才会触发，但是需要变量提前定义

  ```
  {{ value|default:"nothing"}}
  ```

- default_if_none： 该变量为None时触发



#### 自定义模板过滤器

- 显然我们django内置的过滤器是无法满足我们的需求的，我们可以自定义过滤器；在新建一个文件夹templatetags（必须是这个名），放在app文件加下，在里面创建一个my_tags.py文件

  ```python
  from django import template
  
  # 创建模板库对象
  register = template.Library()
  
  #过滤器最多只有两个参数，第一个参数永远是被过滤的参数（也就是过滤器左边的变量）
  def greet(value, word):
  	return value+word
  
  register.filter("greet",greet)  # 当然也可以使用装饰器@register.filter
  
   # 还需要注意，需要将app注册到INSTALL_APPS中；且使用的时候：需要{% load my_filter %};使用时{{ value|greet:"world"}}
  ```

  实际案例： 在微信、微博发送消息的时候，老是出现 "大约1分钟前" 发布消息

  ```
  @register.fiter
  def time_since(value):
  	if not isinstance(value,datetime):
  		return value
  	now = datetime.now()
  	timestamp = (now-value).total_sencond
  	if timestamp < 60:
  		return "刚刚"
  	elif timestamp >= 60 and timestamp < 60*60:
  		minutes = int(timestamp/60)
  		return "%s分钟之前"%minutes
  	else：
  		return "..."
  ```

  

#### 模板引入与继承

- 引入： 当一个html中需要引入一部分html文件的时候，可以使用 include 标签

  ```
  # header.html
  <p> 我是header </p>
  
  # footer.html
  <p> 我是footer </p>
  
  # main.html
  {% include 'header.html' %}
  <p> 我是 main 内容 </p>
  {% include 'footer.html' %}
  
  --------------------------------------------
   #默认include标签包含模板，会自动的使用主模板的上下文（也就是说子模版不写就用模板的，在继承中），也可以使用主模板的变量。如果想传入一些变量，那么可以使用with语句
  
  {% include 'footer.html' with username= '小花'%}
  ```

- 模板继承： 我们可以通过继承模板来节省代码

  ```python
   #母版
  {% block title%} 默认模板 {% endblock %}
  
  ------------------------------------
   # 子板
  {% extends 'base.html' %}
  
  {% block title%} 
  	子板
  {% endblock %}
  
   #如果要使用模板的内容：可以在代码块中{{block.super}}
  ```

  

#### 加载静态文件

- 首先确保注册app到INSTALL_APPS中

- 在settings.py文件中设置STATIC_URL = '/static/'（作用于app下的static文件）；有些静态文件是不和任何app挂钩的，那么可以在settings.py中添加STATICFILES_DIRS参数（作用于项目下的static文件，优先级高于应用static），以后就可以查找下面的静态文件了

  ```
  STATICFILES_DIRS = [
  	os.path.join(BASE_DIR,'static'),
  ]
  ```

- 加载静态文件的方式：如果每次在获取静态文件的时候都要输入一长串的路径，我们可以通过标签获取静态文件

  ```
  {% load static %}
  
  <link href='{% static 'index.js'%}'
  ```

- 如果不想每次模板在加载静态文件都使用load加载static标签，那么我们可以在setting.py中的TEMPLATES/OPTIONS添加如下参数：

  ```python
  TEMPLATES = [
      {
          'BACKEND': 'django.template.backends.django.DjangoTemplates',
          'DIRS': [os.path.join(BASE_DIR, 'templates')]
          ,
          'APP_DIRS': True,
          'OPTIONS': {
              'context_processors': [
                  'django.template.context_processors.debug',
                  'django.template.context_processors.request',
                  'django.contrib.auth.context_processors.auth',
                  'django.contrib.messages.context_processors.messages',
              ],
              # 自动在django内置的标签中加载过滤器，之后就不要{%load static%}
              'builtins':['django.templatetags.static']
          },
      },
  ]
  ```

- 如果没有在settings.INSTALLED_APPS中添加django.contrib.staticfiles。那么我们就需要将请求静态文件的url于静态你文件的路径进行映射，代码如下：

  ```python
  from django.conf import settings
  from django.conf.static import static  # static函数返回的是一个列表，列表+列表
  
  urlpatterns = [
  	...
  ] + static(settigs.STATIC_URL,document_root=settings.STATICFILES_DIR[0])
  ```

  

#### 连接数据库

- 我们使用Django来操作MySQL，在python3中驱动程序有多种选择，比如pymsql、mysqlclient等，这里我们使用mysqlclient。

- Django配置连接数据库，在settings.py文件中修改DATABASES 参数

  ```python
  DATABASES = {
      'default': {
          'ENGINE': 'django.db.backends.mysql',
          'NAME': 'dfz',
          'USER':'root',
          'PASSWORD':'123',
          'HOST':'127.0.0.1',
          'PORT':3306
      }
  }
  ```

- 配置与安装

  ```python
   #django连接MySQL，使用的是pymysql模块，必须得安装2个模块。否则后面会创建表不成功！或者提示no module named MySQLdb 
  pip3 install pymysql
  pip3 install mysqlclient
  
   #在应用目录下的\__init__.py文件中添加以下代码（我们这里是在应用 app文件下，设置的）
  import pymysql
  pymysql.install_as_MySQLdb()
  #启动项目，会报错：no module named MySQLdb 。这是因为django默认你导入的驱动是MySQLdb，可是MySQLdb 对于py3有很大问题，所以我们需要的驱动是PyMySQL。（也就是收到设置驱动式mysql）
  ```

- 在Django中操作数据库

  在Django中操作数据库有两种方式：

   1. 使用原生的sql语句操作，connection/raw

      ```python
       # 在Django中使用sql语句操作其实是使用python db api的接口，如果你的mysql驱动使用额是pymysql，那么你就是使用pymysql来操作，只不过Django将数据库连接这部分封装到db中了，我们不需要关系底层的实现，只需要使用即可：
       
      from django.db import connection
      # 获取游标
      cursor = connection.cursor()
      # 拿到游标执行sql语句
      cursor.execute('select * from student')
      # 获取数据
      rows = cursor.fetchall()
      #遍历查询到的数据
      for row in rows:
      	print(row)
          
       ------------------------------------------
       # 在Django中结合视图函数使用，也就是说完全可以使用这种方式来开发，但是可扩展性太弱，且容易SQL注入   
      def index(request):
          # 操作的是settings配置的数据库中的表
          from django.db import connection
          # 获取游标
          cursor = connection.cursor()
          # 拿到游标执行sql语句
          cursor.execute('select * from student')
          # 获取数据
          rows = cursor.fetchall()
          # 遍历查询到的数据
          for row in rows:
              print(row)
          return HttpResponse('书籍')
      ```

   2. 使用ORM模型来操作

      ORM与Django之前的文件内容差不多

      

      

#### ORM知识点补充

- 返回当前时间在views.py文件中， 如果发现项目时间老是使用的UTC时间，那么我们可以在settings.py文件中设置 TIME_ZONE="Asia/Shanghai"、USE_TZ=True

  ```python
  from django.utils.timezone import now,localtime  # 可判断django使用的什么时间
  
  def index(request):
  	...
  	#article = Article(title='acb',create_time=now())
  	article = Article(title='acb',create_time=localtime())  # 根据TIME_ZONE返回时间
      print(localtime('2018-05-01'))
  	return HttpResponse('success')
  
  # 重点：除了上面的两个添加成立，在使用的时候如下,下面的make_aware会把navie时间转换成aware时间
  
  from django.utils.timezone import make_aware
  from datetime import datetime
  
  start_date = make_aware(datetime(year=2018,month=1,day=1))
  end_date = make_aware(datetime(year=2019,month=1,day=1))
  articles = Article.objects.filter(pub_date_range(start_date,end_date))
  ```

- 在进行ORM外键操作的时候，如果我们要连接另外一个app下面的表

  ```
  class Article(models.Model):
  	title = models.CharField(max_length=10)
  	content = models.TextField()
  	category = models.ForeignKey('app2.User',on_delete=models.CASCADE)
  ```

- 外键的删除的操作

  - CASADE: 级联操作，如果外键对应的那条一的数据被删除，相关的多都会被删除
  - PROTECT: 受保护的，如果一条数据有相关的外键就无法被删除
  - SET_NULL: 设置为空，如果外键的那条被删除，就会被NULL代替
  - SET_DEFAULT：被删除后设置值
  - SET() ：如果引用的数据被删除，就会设置括号内地值
  - DO_NOTHING: 什么都不做
  
- 查询条件

  - exact : 使用精确的=号进行查找，如果提供的是一个None，那么在SQL层面就是介绍为NULL

    ```python
    article = Article.object.get(id__exact=14)
    print(article.query)  # 打印转换成相关的sql语句
    
    article = Article.object.get(id=14)  # 等价于id__exact=4
    
    article = Article.object.get(id__exact=None)
    
    -----------------------------------------------
    select ... from article where id=14
  select ... from article where id IS NULL
    ```

  - iexact: 使用like进行查找（忽略大小写），示例如下

    ```
    article = Article.object.filter(title__iexact='Hello world')

    select ... from article where title like 'hello world'
    ```
  ```
    
  - contains: 大小写敏感，判断某个字段是否包含了某个数据，icontains，相当于like，其实类似于like
  
  ```
    article = Article.object.filter(title__iexact='world')
    ```
  
  - in ： 提取那些给定的field的值是否在给定的容器中，容器可以为list、tuple等
  
    ```
    article = Article.object.filter(id__in=[1,2])
    ```
  
  - gt：大于、lt：小于、gte:大于等于、lte小于等于
  
    ```
    article = Article.object.filter(id__gt=4)
    ```
  
  - startswith: 判断字段值的开始开头、istartswith：忽略大小写、endswith：以什么结尾
  
    ```
    article = Article.object.filter(title__startswith='world')
    ```
  
  - range ：判断某个字段是否在区间
  
    ```python
    from django.utils.timezone import make_aware
    from datetime import datetime
    
    start_date = make_aware(datetime(year=2018,month=1,day=1))
    end_date = make_aware(datetime(year=2019,month=1,day=1))
    articles = Article.objects.filter(pub_date_range(start_date,end_date))
    ```

  - date：针对date或datetime字段

    ```
    article = Article.object.filter(pub_date__date=date(2018,3,3))
    ```

  - year :根据年份查找、month、day

    ```
    article = Article.object.filter(pub_date__year=2018)
    ```

  - isnull 根据值是否为空查找

    ```
    article = Article.object.filter(pub_date__isnull=False) # 判断不为空的数据
    ```

  - regex和iregex： 正则匹配

    ```
    article = Article.object.filter(title__regex=r'^hello')  # 以hello开头的title
    ```

    

- 聚合函数

  ```python
  from django.db import models
  
  class Author(models.Model):
  	name=models.CharField(max_length=100)
  	age=models.IntegerField()
  	email=models.EmailField()
  	
  	class Meta:
  		da_table='author'
  
  class Publisher(models.Model):
  	name = models.CharField(max_length=300)
  	
  	class Meta:
  		db_table='publisher'
  		
  class Book(models.Model):
  	name = models.CharField(max_length=300)
  	pages=models.IntegerField()
  	price= models.FloatField()
  	rating=models.FloatField()
  	author=models.ForeignKey(Author,on_delete=models.CASCADE)
  	publisher=models.ForeignKey(Publisher,on_delete=models.CASCADE)
  	
  	class Meta:
  		db_table = 'book'
  		
  class BookOrder(models.Model):
  	book = models.ForeignKey('book',on_delete=models.CASCADE)
  	price = models.FloatField()
  	
  	class Meta:
  		db_table='book_order'
  ```

  - Avg : 求平均，获得某一字段的平均值

    ```python
    from djang.db.models import Avg
    from django.db import connection  # 查看原生sql
    
    result = Book.objects.aggregate(Avg('price'))
    print(connection.queries)  # 打印原生sql 这里不能使用result了因为，它是作用于QuerySet
    
    print(result)   #{'price_avg':0.9}
     # 指定名字result = Book.objects.aggregate(my_avg=Avg('price'))
    ```

  - Count: 获取指定的对象的个数，示例代码如下

    ```
    from djang.db.models import Avg,Count
    result=Book.objects.aggregate(book_num=Count('id'))
    ```

  - Sum：获取总值

  - Max、Min：求最大最小值

    ```
    result=Author.objects.aggregate(max=Max('age'))
    ```

  - aggregate： 返回使用聚合函数后的字段和值

    ```python
    from djang.db.models import Avg
    from django.db import connection  # 查看原生sq
    result = Book.objects.aggregate(order_avg=Avg('bookorder__price')) # 跨字段，根据odder的订单数的书id分组，聚合求平均值
    
    print(result)
    print(connection.queries) # 会发现在第一张表中会省略掉其他数据于另一张表连接，从而只得到一条数据
    ```

  - annotate： 在原来模型字段的基础之上添加一个使用了聚合函数的字段，并且在使用聚合函数的时候，会使用当前这个模型的主键进行分组

    ```python
    #上面的案例我们使用annotate来实现
    books = Book.objects.annotate(avg=Avg('bookorder__price'))
    for book in books:
    	print("%s: %s"%(book.name,book.avg))
    print(connection.queries)
    ```

- F 、Q表达式

  F表达式是用来优化ORM操作数据库的，针对表中某字段或字段于字段的操作于运算的工具

  ```
  from django.db.models import F,Q
  Employee.objects.update(salary=F('salary')+1000)
  ```

  Q表达式实现了我们扩展查询功能，对查询条件进行封装, 比如：最明显的ORM中没有或概念的查询

  ```
  books = Book.objects.filter(Q(price__lte=-10)|Q(rating__lte=9))
  ```

  

#### QuerySet API

我们通常在操作ORM的时候，都会使用到 模型名.objects 的方式来进行操作，其实 模型名.objects 是一个django.db.models.manager.Manager对象，而Manager这个类是一个空壳，本身是没有任何属性和方法的都是Python动态添加，从QuerySet类拷贝过来的：

![1570756754563](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1570756754563.png)

- 源码

  ```python
  1、通过type(Book.object)可查看类型，得到Manager的位置：from django.db.models.manager import Manager，点击进入
  
  2、Manager类源码：在父类中调用了from_queryset来动态获取继承类，传入的QuerySet类
  
  	class Manager(BaseManager.from_queryset(QuerySet)):
      	pass
      
  3、查看from_queryset源码：
  	@classmethod 
      def from_queryset(cls, queryset_class, class_name=None): # query_class为传入的QuerySet
          if class_name is None: # 默认为空，得到的结果class_name='BaseManagerFromQueryset'
              class_name = '%sFrom%s' % (cls.__name__, queryset_class.__name__)
              
          class_dict = {
              #QuerySet
              '_queryset_class': queryset_class,
          }
          # 给QuerySet添加各种方法
          class_dict.update(cls._get_queryset_methods(queryset_class))
          # type创造类，第一个参数是名字，第二个是继承的类，第三个是携带的数据和方法
          return type(class_name, (cls,), class_dict)
  
  4、_get_queryset_methods方法就是将QuerySet中的一些方法拷贝出来，我们可以点开QuerySet查看所有的方法
  ```

- 必知必会query方法：

  - filter：过滤函数，返回一个新的QuerySet，链式编程

  - exclude：排除满足条件的数据，非

    ```
    Article.objects.exclude(title__contains='hello')
    ```

  - annotate: 给QuerySet中的每个对象添加一个使用的查询表达式（聚合、F、Q）的新字段

    ```
    Article.objects.annotate(author_name=F('author__name'))  # 以上代码将在每个对象中添加一个author__name字段，用来显示这个文章的作者的年纪
    ```

  - order_by : 指定查询结果根据某个字段进行排序，如果想倒叙在字段前加负号

    ```
    article= Article.objects.order_by('create_time')
    ```

  - values: 用来指定提取数据，默认提取所有，返回一个字典、values_list:返回元组

    ```
    articles = Article.object.values('title','content')
    
    articles = Article.object.values_list('title','content')
    ```

  - all: 返回所有数据

  - select_related: 提取某个模型的数据的同时，也提前将相关数据提取出来，减少数据库查询次数

    ```
    article = Article.object.get(pk=1)
    article.author # 显得麻烦，连接两次
    
    article = Article.objects.select_related('author','publisher') # 返回一个列表
    ，根据文章放回所有的作者和出版社
    article = Article.objects.select_related('author').get(pk=2) # 一次查询
    ```

  - prefetch_related: 和上一个类似，只不过针对多对一或多对多时使用: 如要获取标题中带有hello字符串的文章的所有便签

    ```
    articles = Article.objects.prefetch_related('tag_set').fliter(title__contains='hello')
    
    for article in articles:
    	print(article.title)
    	print(article.tag_set.all())
    ```

  - defer：在一些表中，可能存在很多字段，我们可以同故宫defer来过滤字段，only：提取字段

  - get：获取满足条件的数据，返回有且只有一条，多少都不行

  - create: 创建并保存一条数据

    ```
    article= Article.objects.create(title='ok')
    ```

  - get_or_create:根据条件进行查找，找不到则创建

  - bulk_create: 一次创建多个数据

    ```
    Tag.object.bulk_create([
    	Tag(name='111'),
    	Tag(name='222'),
    ])
    ```

  - count:获取提取数据的个数

    ```
    article= Article.objects.count()
    ```

  - exists: 存在

  - distinct：去重

    ```
    orders = BookOrder.objects.values('book_id').distinct()
    ```

  - update: 更新数据

    ```
    Article.objects.filter(category__isnull=True).update(category_id=3)
    ```

  - delete: 删除所有满足条件的数据



#### 根据已有的表自动生成模型

- 在实际开发中，有些时候可能数据已经存在了。如果我们用Django开发一个网站，读取的是之前已经存在的数据库中的数据。那么该如何讲模型与数据库中的表映射？根据旧的数据库生成的对应的ORM模型，有以下步骤:

  - Django给我们提供了一个inspectdb的命令，可见非常方便的将已经存在的表，自动的生成模式。想要inspectdb自动将表生成模型，首先需要在settings.py中配置好数据相关信息，不然找不到数据

    ```
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': 'tornado',
            'USER':'root',
            'PASSWORD':'123',
            'HOST':'127.0.0.1',
            'PORT':3306
        }
    }
    ```

    库中已经有很多数据了

  - 接着通过终端，进入项目目录下执行，之后会将表转换成模型代码，显示在终端

    ```python
    python3 manage.py inspectdb 
    
    # 想输出为文件：python3 manage.py inspectdb > models.py
    ```

    比如

    ```python
    from django.db import models
    
    class Article(models.Model):
    	title=models.CharField(max_length=100)
    	...
    	
    	class Meta:
    		managed=False  # 这就相当于一把锁，如果要修改表中的字段，需要删除这句话,因为会不迁移到数据库中
    		db_table='article'  # 这个是必须和数据库中的表名一致不然会报错，类名可改，这个不可改
    ```

  - 执行python3 manage.py makemigrations 生成初始化迁移脚本，当执行python3 manage.py migrate  --fake-initial, 如果不写这个的话，就会根据models.py的模型在数据库中重新生成表

    ```
    当涉及到到多个app的时候的迁移文件时，可以指定app进行迁移：
    python3 manage.py migrate app2  --fake-initial
    ```

    