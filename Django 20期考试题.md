## Django 20期考试题

##### 前端部分：

1. 清除浮动的方法

   ```
   1.给父盒子添加固定高度 
   2.内墙法：给最后一个浮动元素添加一个空的块级标签，设置该标签的属性为clear:both; 
   3.伪元素清除 给父元素添加一个类
   	.clearx::after{ content:'', display:block; clear:both } 
   4.overow:hidden; BFC区域
   ```

2. 阐述CSS定位的几种方法，并分别描述他们的不同

   ```
   静态定位：静态定位意味着“元素默认显示文档流的位置”。没有任何变化。
   
   相对定位
   	1. 不脱离标准文档流，单独设置盒子相对定位之后，如果不用top,left,right,bottom对元素进行偏移，那么与普通的盒子没什么区别。
   	2. 有压盖现象。用top,left,right,bottom对元素进行偏移之后，明显定位的元素的层级高于没有定位的元素
   	
   绝对定位
   	1. 脱离了标准文档流，不在页面中占位置
   	2. 层级提高，做网页压盖效果
   	
   固定定位
   	它跟绝对定位基本相似，只有一个主要区别：绝对定位固定元素是相对于html根元素或其最近的定位祖先元素，而固定定位固定元素则是相对于浏览器视口本身。
   
   ```
   
3. jquery获取标签的三种方法

   ```
   $('.class')
   $('#id')
   $('标签')
   ```

4. 阐述一下js中window.onload() 和 jquery的$(function(){}) 的区别

   ```
   window.onload是在页面所有的元素都加载完成后才触发。
   $(function(){})是在页面的dom结构加载完毕后就触发 dom里的内容不一定都已经加载完成
   	比如说一个页面有好多图片 而加载这些图片需要一定的时间 window.onload必
   须得等到全部的图片都加载完成后才能触发，而$(function(){})只要在dom加载完
   毕之后就会执行 图片不一定已经加载完成
   	1. $(function(){})不会被覆盖，而window.onload会被覆盖
   	2. window.onload必须等到页面内包括图片的所有元素加载完毕后才能执行。 $(document).ready()是DOM结构绘制完毕后就执行，不必等到加载完毕。
   ```





##### 一、基础题

​		 1. 写出你所知道Djang有关的所有命令(下载、安装等)

```python
1.安装django 
pip install django == 1.11.15 
2.创建项目 
diango-admin startproject 项目名 
3.启动项目 
cd 项目目录 
python manage.py runserver    #127.0.0.1:8000
python manage.py runserver 80 
python manage.py runserver 0.0.0.0:80 
4.创建App 
python manage.py  startapp app01 
5.数据库迁移 
python manage.py makemigrations      #检查 models.py是否有变化,记录变化 
app01/migrations python manage.py migrate         #将变更记录同步到数据库中
```



2.Django的setting.py中,你用到的配置项有哪些?他们的作用是什么?

```
1.INSTALLED_APPS     注册App 让Django 程序可以识别新建的App
2.MIDDLEWARE   中间件   自定义中间件注册进来可以执行其中的方法
3.TEMPLATES  模版的相关配置    主要看DIRS 模版的存放路径
4.DATABASES     数据库相关配置 告诉Django连接什么数据库
5.STATIC_URL='/static/'   静态文件的别名, 模版中已别名开头
6.STATICFILES_DIRS = [   #静态文件的具体存放路径   Django会按照 
		os.path.join(BASE_DIR,'static')
	] 
```



3.Django使用Mysql数据库的流程是什么? 

```
1.创建一个mysql数据库
2.settings 中写配置
DATABASES={
	'defalut':{
		'ENGINE':'django.db.backends.mysql',
		"NAME":'library',
		"USER":'root',
		"HOST":'127.0.0.1',
		"PORT":3306,
		}
}

3.告诉django使用pymysql的模块连接mysql数据库
在与settings 同级的目录下的 __init__.py中写代码:
import pymysql
pymysql.install_as_MySQLdb()

4.在app01/models.py中写类(继承models.Model)
class Publisher(models.Model):
	id = models.AutoField(primary_key = True)
	name = models.CharField(max_length = 32,unique = True)
	
5.数据库迁移的命令
python manage.py makemigrations
python manage.py migrate
```

　

​	   4. ORM是什么?为什么要使用ORM?他的优缺点是什么?他的对应关系是什么?	

```
1.ORM(对象关系映射) 是一种为了解决面向对象与关系型数据库不匹配的技术
2.使用ORM不用在过多得关注SQL语句的编写,而是更加专注于逻辑代码的编写
3.优点:
	1.ORM 提供了对数据库的映射,不用直接编写SQL代码, 只需操作对象就能对数据库操作数据
	2.让软件开发人员专注于业务逻辑的处理,提高了开发效率
	
缺点:
	1.ORM 的缺点是会在一定程度上牺牲程序的执行效率
	2.ORM 的操作是有限的, 也就是ORM定义好的操作是可以完成的
	
4.对应关系
	类    ----->  数据表
	对象  -----> 数据行
	属性  -----> 字段
```

　

5.定义视图函数的时候要注意什么?

```
1.函数的第一个参数是request
2.函数必须返回一个HttpResponse对象
```

 　

6.FBV和CBV是什么? 定义一个简单的CBV.

```python
FBV:函数
CBV:类
CBV版:
class AddClass(View):
	def get(self,request):
		return render(request,"add_class.html")
		
	def post(self,request):
		class_name = request.POST.get("class_name")
		models.Classes.objects.create(name = class_name)
		return redirect("/class_list/")
```

　

7.CBV使用装饰器的方法有哪些,分别是什么?写出简单示例.

```
1.导入方法装饰器
from django.utils.decorators import method_decorator

2.方法:
1.给方法上加装饰器
@method_decorator(wraper)
def get(self,request,*args,**kwargs)
	return HttpResponse("ok")
	
2.给dispatch 上加
@method_decorator(wraper)
def dispatch(self,request,*args,**kwargs):

3.给类上加
@method_decorator(wraper,name = 'get')
class Simple(View)
```



8.写出你所知道的request对象的方法和属性

```
1.属性:
request.method       --->请求的方式8种    GET  POST  PUT  DELETE  OPTIONS
request.GET          ----> 字典  url上携带的参数
request.POST         ----->字典  form 表单通过POST请求提交的数据
request.path_info    ----->URL 路径 不带参数
request.body         ----->请求体
request.FILES           上传的文件{}

2.方法:
request.get_host()   ----->主机地址
request.get_full_path()      ----->URL   路径   带参数
```

 

9.给视图传参数的方式有哪几种?分别是什么,写出示例.

```
1  分组:
url(r'book/([0-9]){4}/([0-9]{2})/$',book)
按照位置参数传递给视图函数

2.命名分组
url(r"book/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/$",book,)
按照关键字参数传递给视图函数

3.参数
url(r"book/(?P<year>[0-9]{4})/(?P<month>[0-9])/$",book,{'year':'1998'})
```

 

　10.如何在URLconf中给url命名?在视图和模版中如何使用url反向解析?请写出所有情况

```
urls.py:
url(r'^author_list/$',views.author_list,name = 'author_list'),
url(r'^home/([0-9]{4})/([0-9]{2})/',views.home,name = 'home'),
url(r'^home/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/',views.home,name = 'home'),

视图中反向解析:
from django.urls import reverse
reverse("author_list")     -----> '/author_list/'
无名分组:
reverse('home',args=('1999','06'))         ---->'/home/1999/06/'
有名分组:
reverse('home',kwargs = {'year':'1998','month':'06'})   ---->'/home/1998/06/'

模板中反向解析:
{% url 'author_list' %}     ---->'/author_list/'
无名分组:
{% url 'home' '2000' '09' %}
有名分组:
{% url 'home' '2000' '09' %}
{% url 'home' month = '09' year = '1898' %}
```

　

11.请写出你所知道的模板语法

```
1.变量 
	{{ 变量名 }}
2.  .的用法
	{{ name_list.0 }}             #列表取值
	{{ name_dict.name1 }}         #字典取值
	{{ p1.name }}                        #对象的属性
	{{ p1.sing }}                        # 对象的方法
3. 过滤器
	{{ 变量名|filter:参数 }}
	
4.tags
	{% for user in user_list %}
		{{ user.name }}
	{% endfor %}

	{% if  条件 %}
		操作
	{% endif %}
```



12.请写出母版和继承的使用方法

```
1.创建一个HTML文件当做母版 'base.html' ,母版中将多个页面的重复代码提取出来
2.在母版中定义多个block ,来区分不同页面的不同内容
3.在子页面中继承母版  {% extends 'base.html' %}
4.在block 中写自己页面独特的内容,用来替换模板中block中的内容
```

　

13.请写出自定义filter的步骤

```
1.在app下创建一个名叫 templatetags 的python包  templatetags不能写错
2.在templatetags 里建一个 py文件 myfilters
3.在py文件中编辑:
from django import template
register = template.Library()            #  register 名字不能错

@register.filter
def add_sb(value,arg):
	return '{}sb'.format(value)

@register.filter(name = 'dsd')
def add_sb(value,arg):
	return '{}sb'.format(value)
4.重启
5.使用filter
{% load myfilters %}
{{ name1|dsd:'very' }}
```



14. 为什么要使用cookie和session?

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



15.Django中操作 cookie 和session 的方法有哪些?

```
cookie部分
1.设置 cookie
	rep = redirect("/index/")
	rep.set_cookie(key,value,max_age = 1000)
	rep.set_signed_cookie(key,value,max_age = 1000, salt = '')
2.获取 cookie
	request.COOKIES['key']
	request.COOKIES.get('key')                --->推荐的取法
	request.get_signed_cookie('key',salt ='')
3.删除 cookie
	rep.delete_cookie('key')
	
session部分
1.设置session
	request.session['key'] = value
	request.session.setdefault(key,value)            ---->存在就不设置
2.获取session
	request.session['key']
	request.session.get(key,'')
3.删除 session
	request.session.pop(key)                ----->删除某一个键值对
	del request.session['key']

request.session.delete()     -->删除所有的session 键值对
request.session.flush()       -->删除所有的session 键值对,删除了 cookie
4.设置超时时间
	request.session.set expiry()
5.清除当前过期的 session
	request.session.clear_expired()
```

　

16.Django的中间件是什么?请写出 process_request 和process_response 以及process_view三个方法的执行时间、执行顺序和不同返回值不同的流程

```
中间件：中间件是一个用来处理Django的请求和响应的框架级别的钩子。它是一个轻量、低级别的插件系统，用于在全局范围内改变Django的输入和输出。每个中间件组件都负责做一些特定的功能。但是由于其影响的是全局，所以需要谨慎使用，使用不当会影响性能。

1.process_request(self.request)
执行时间 :请求来 先执行,在视图之前
执行顺序: 按照注册顺序执行
返回值:
	返回值是None  正常往后走
	返回值是 HttpResponse 对象   后面的不在走, 直接执行当前中间件的process_response函数,回走

2.process_response(self,request,response)
执行时间: 视图函数之后
执行顺序:安装注册顺序倒序执行
返回值: HttpResponse对象

3.process_view(self,request,view_func,view_args,view_kwargs)
执行时间: 在视图之前 ,process_request之后
执行顺序: 按照注册顺序 顺序执行
返回值:
	None    正常往后走
	HttpResponse 对象   后面的中间件 process_view方法、视图不执行，走所有走过中间件的 process_response方法
```



17. 请写出使用jquery发送.ajax请求，能通过Django的csrf的校验的两种方法

    ```
    前提条件：1、确保有csrftoken的cookie
    		2、在页面中使用{% csrf_token %}
    		3、加装饰器 ensure_csrf_cookie
    		
    1.给data中添加csrfmiddlewaretoken的值
    data: {
    		'csrfmiddlewaretoken':$('[name="csrfmiddlewaretoken"]').val(),
    		a: $("[name='i1']").val(),
    		b: $("[name='i2']").val(),
    	},
    	
    2.加请求头
    headers:{
    		'x-csrftoken':$('[name="csrfmiddlewaretoken"]').val(),
    	},
    
    3.使用文件
    
    ```

    

##### 二、ORM操作

models.py:

 ![img](https://img2018.cnblogs.com/blog/1411892/201809/1411892-20180920185134534-519434334.png)

 

##### 1.查询

a) 查找三年级的班级对象

```
models.Classes.objects.filter(c_name__startswith='三年')
```

b)查找三年二班的所有同学的名字

```
models.Classes.objects.get(c_name = '三年二班').student_set.all().values('s_name')
```

c)查询每个班级的名称和的学生人数

```
models.Classes.objects.annotate(count = Count('student')).values('c_name','count')
```

d)查询年纪最大的老师姓名和年龄

```
1.models.Teacher.objects.order_by('-age').values('t_name','age').first()
2.models.Teacher.objects.filter(age=models.Teacher.objects.aggregate(max=Max('age'))['max']).values('t_name','age')
```

e)分别查询出男女老师的个数

```
models.Teacher.objects.values('sex').annotate(Count('sex'))
```

f)查询名字中有"灰"的学生姓名和成绩

```
models.Student.objects.filter(s_name__contains = '灰').values('s_name','score')
```

g)查询成绩合格的学生姓名和成绩

```
models.Student.objects.filter(score__gte=60).values('s_name','score')
```



##### 2.增加

a)新增一个名为"三年四班"的班级

```
models.Classes.objects.create(c_name = '三年四班')
```

b)给三年二班新增一个名为"小青"的同学

```
models.Student.objects.create(s_name = '小青',my_class_id = 2)
models.Classes.objects.get(c_name = '三年二班').student_set.create(s_name='小青',score = 99 )
```

c)新增一个名为"苑局"的30岁的男老师,他教三年三班

![img](https://img2018.cnblogs.com/blog/1411892/201810/1411892-20181024212209111-1893722591.png)

 



##### 3.修改

a)小红转班了,转到了三年四班

![img](https://img2018.cnblogs.com/blog/1411892/201810/1411892-20181024212220782-471874201.png)

 

b)给所有的学生的成绩都加5分

```
from django.db.models import F
models.Student.objects.update(score = F('score')+5)
```

 ![img](https://img2018.cnblogs.com/blog/1411892/201809/1411892-20180920185202343-1742936353.png)