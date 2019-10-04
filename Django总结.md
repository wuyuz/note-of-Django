# Django复习篇

## 一.今日内容

### 1.Django是web框架

- 搭建socket后端服务。

### 2.做了什么事？

```
socket收发消息
根据不同路径返回不同内容
动态的页面(模版渲染)  用到jinja2

浏览器输入地址之后发生什么事情：
	pass

```

### 3.http

```python
请求应答标准
请求的格式：
	'GET URL的路径 HTTP1.1\r\n'
	k1:v1\r\n
	k2:v2\r\n
	\r\n
	请求体(请求数据)

#GET请求没有请求体



响应格式(response)
'HTTP/1.1 状态码 状态描述\r\n'
k1:v1\r\n
k2:v2\r\n
\r\n
响应体(响应数据)

```

### 4.状态码

```
1XX		请求已经接受，进一步处理
2XX		200 OK 请求成功
3XX		重定向 301 永久重定向
			  302 临时重定向
4XX		请求的错误  404错误，403权限不够，402认证错误
5XX		服务器错误500，502网关的错误
```

### django 1.x 与2.x区别

```
1.更简单URL路由语法 path转化器
2.admin应用针对移动设备优化改进
3.支持SQL开窗表达式。
```



### 5.头信息

```
content-type: 
	text/html    HTML格式
	text/plain   纯文本格式
	text/xml	XML格式
	image/jpeg   jpg图片格式
	application/json   json数据格式
	
User-Agent:用户代理
cookie
set-cookie
host
...		 
```

### 6.Django有什么

#### 1.路由

- 参数：

  - 正则表达式

    ```
    r'^$'  [a-zA-Z0-9]{4}  \d  \w
    ? + * .
    ```

  - 视图函数

  - 传入参数

  - 别名

    ```
    分组命名分组
    
    r'^edit_customer/(\d+)/'
    分组  将不获的参数按照 位置传参 传递给视图函数
    r'^edit_customer/(?P<cid>\d+)/'
    命名分组：
    ```

  - url 的命名和反向解析

    ```python
    #1静态路由：
    	url(r'^login',auth.login,name='login')
    模版：
    	{% url 'login' %}
    py文件：
    	from django.urls import reverse
    	reverse('login')
    #2动态路由
    模版：
    	位置传参
    	{% url 'customer' 1 %}
        关键字传参
        {% url 'customer' pk=1 %}
    py文件:
        位置传参
        reverse("customer",args=(1,))
        关键字传参
        reverse("customer",kwargs={'pk':1})
    #关键字传参好处，传入多个参数，不需要关心它的顺序。
        
    ```

  - namespace命名空间

    ```python
     
        {% url 'app01:customer' pk=1 %}
     
      reverse("app01:customer",args=(1,))
    ```

#### 2.视图

- MVC

  ```
  M：model   与数据库交互
  V:视图页面HTML
  C：controller 控制器  控制代码流程
  ```

- MTV

  ```
  M：model  与数据库交互
  T:模版   HTML
  V:view  业务逻辑
  ```

- FBV    CBV

  ```
  def demo(request,*args,**kwargs):
  	#业务逻辑
  	return response
  
  from django.views import View
  class Demo(View):
  	def get(self,request,*args,**kwargs):
  		pass
  		return response
  	def post(self,request,*args,**kwargs):
  		pass
  		return response
  urls.py
  url(r'^xxx/',xxx.as_view())
  
  ```

- 装饰器添加

  ```python
  CBV：
  	from django.utils.decorators import method_decorator
  
  1.加在方法上：
  	@method_decorator(timer)
  	def get(self,request,*args,**kwargs):
  		pass
  2.加在dispatch方法上
  	@method_decorator(timer)
  	def dispatch(self,request,*args,**kwargs):
  		pass
  3.加在类上
  	@method_decorator(timer,name="dispatch")
  	class XX(View):
  		pass
  ```

- 其他

  ```
  @csrf_exempt 只能加在dispath上，其他位置没有效果
  ```

  

- resquest对象

  ```python
  #属性：
  1.request.method		请求方式：GET,POST,DELETE,OPTIONS,PUT,PATCH,HEAD,TARCE
  2.request.GET		 #url携带的参数
  3.request.POST		 #post请求发送的数据  urlencode方式才能有
  4.request.body       #请求体中  不是urlencode方式拿去post数据
  5.request.FILES      #上传的文件，enctype='multipart/form-data'
  6.request.COOKIES    #普通cookie   请求头的cookie
  7.request.session	 #session
  8.request.META	 #头信息  xxx->XX 加HTTP_  -  --> _
  9.request.path_info   #路径  不包含ip和端口 也不包含参数
  
  #方法：
  1.request.get_full_path()    #完整路径，不包含ip和端口，包含参数
  2.is_ajax			
  3.request.get_host()		#获取主机IP
  
  ```

- response对象

  ```python
  HttpResponse("字符串")		#返回字符串
  render(request,'模版路径',{})  #返回一个完整页面
  redirect('重定向地址')		#重定向  最终location:地址
  JsonResponse({},safe=False)			#默认放字典
  TemplateResponse(request,"模版的路径",{})  #中间件
  ```

#### 3.模版

```python
{{ 变量 }}
{% 标签 %}    tag	

变量：
	要用变量，首先需要render传参，才能使用
	拿取变量：
    	.索引
        .属性
        .方法
        .字典key
过滤器：
	{{ k1|filter_name}}   {{ k1|filter_name:'xxx'}}
内置过滤器：
	safe     取消转义，   后端：mark_safe
    default		默认
    add			加
    date		日期格式化  'Y-m-d H:i:s'
    filesizeformat  人类可读大小格式

```

- 标签

```python
{% for i in list %}
	{{i}}
{% empty %}
	xxx
{% endfor %}

{{ forloop.counter }}


{%if 条件%}
	xx
{%elif 条件%}
	xxx
{% else %}
	xx
#注意，if不能连续判断   不支持算数运算

{% with xxx as sss %}
{% endwith %}

{% csrf_token %}   #渲染input标签   name = csrfmiddlewaretoken   value= 64位字符串

母版和继承：
{% extends "base.html" %}   

组件：
{% include 'nav.html' %}

静态文件：
{% load static %}
{% static '相对路径' %}
{% get_static_prefix %}	#'/static/'

```

- 自定义filter,simple_tag,inclusion_tag

```python

1.在app下创建名templatetags的python包
2.再创建my_tag.py文件
3.在py文件中代码：
	from django import template
    register = template.Library()#register不能错
4.写函数，加装饰器：
	@register.filter
	def xx1(value,arg):#arg可不写
        return xxx
    
    @register.simple_tag
    def xx2(*args,**kwargs):
        return xxxxx
    
    @register.inclusion_tag("模版")
    def xx3(*args,**kwargs):
        return {'num':range(num)}
#使用：
{% load my_tag%}

{{ k1|xx:'xxxx'}}	#可以使用if判断里面

{% xx2 v1 v2 k1=v1 v2=v2 %}
{% xx3 num=10 %}
```

#### 4.ORM操作

```python
对象关系映射

面向对象-->关系型数据库

类   表
对象 记录
属性 字段

优点：编程不会把时间放在编辑sql语句中，开发效率高
缺点：牺牲了执行效率，因为ORM最终要转成sql.复杂sql语句无法编辑

必会13条：
	返回对象列表
	1.all()		#返回所有数据
	2.filter()	#返回符合条件所有数据
	3.exclude()		#返回不满足条件数据
	4.order_by()	#排序
	5.reverse()		#倒序
    6.distinct()		#去重，不能按照字段去重，是拿整个对象
    7.values()		#获取字段的名和值  Queryset字典
    8.values_list()	#获取字段值	元组
  	返回对象
    1.get()		#获取一个满足条件对象。   没有或者多个报错
    2.first()	#获取第一个对象
    3.last()	#获取最后一个对象
    返回布尔值
    1.exists()		#数据是否存在
    2.count()		#计数

单表的双下划线：
__gt	#大于
__gte	#大于等于
__lt	#小于
__lte	#小于等于
__range=[1,6]  #包含1到6的
__in=[1,2,4]	#包含1，2，4的
__contains		#表示包含	区分大小写
__icontains		#表示包含   不区分大小写
__startswith	#开始
__endswith		#结束
__isnull=True	#判断是不是空
3.外键
描述一对多的关系

on_delete=models.CASCADE  2.0版本必填，1.x版本不设置，源码自己填
	models.SET			设置固定值
	models.SET_DEFAULT ,default='12'		设置默认值
    models.DO_NOTHING			什么不做
 
4.对象查询
正向查
	book_obj.pub   关联的对象
	book_obj.pub_id		拿到id
反向查：
	pub_obj.book_set   --->  关系管理对象
    pub_obj.book_set.all()	--->所有数据对象

related_name = 'books'
	pub_obj.books.all()
    
  外键使用关系对象方法：进行删除，要对外键字段设置  null=True
  外键使用关系对象方法：只能操作对象
	
    
    
5.字段查询
	models.Book.objects.filter('pub__name'='xxx')
    models.Publisher.objects.filter('book__name'='xx')
    指定related_name = 'books'
    models.Publisher.objects.filter('books__name'='xx')
    指定related_query_name = 'bookss'
    models.Publisher.objects.filter('bookss__name'='xx')
6.多对多：
	创建第三张表
    books = models.ManyToManyField(to='Book') #生成一张表
    3种方式创建第三张表：
    	方式1：自动生成
    	方式2：手动创建第三张表，查费劲
    	方式3：thronth=?
    关系管理对象的方法：
    	add  添加关系  add()   #对象   id
        remove  删除关系
        clear	删除所有关系
        set		设置多对多关系  [对象，对象]   [id,id]
        create	 新增一个对象并且同当前的对象设置关系

7.聚合：
	from django.db.models import Max,Min,Count,Avg,Sum
    
    Book.objects.aggregate(Max('price'))#要最高价格
8.分组：
	Author.objects.annotate(Count('books'))   #对象列表    
	Book.objects.values('author').annote(Count('id')) #对象列表[{},{}]
9. F   Q
	from django.db.models import F,Q
    #F
    Book.objects.filter(sale__gt=F('stock'))#比较两列数据
    Book.object.update(sale=F('sale')*10-200)#获取字段后计算
    #Q
    	&与      |或     ~非
        Book.objects.filter(~Q(Q(id__gt=10)|Q(id__lt=5))
                            Q(('id__gt',10))
                            
    q = Q()  					
	Q.connector= 'OR'
    q.children.append(Q(('name__contains',10)))
                            
10.事务
       try:
           with transaction.atomic():
              query_set = Book.objects.filter(id__gt=5).select_for_update()
              query_set
       except Exception as e:
              print(e)
    
    
```

#### 5.Cookie   Session

```python
cookie 

1.为什么用cookie?
	Http协议是无状态的，每次请求相互独立的。
	cookie是保存在浏览器上一组组键值对。
	特点：
		1.由服务器让浏览器进行设置
		2.浏览器有权进行读取或保存
		3.下次访问时自动携带响应cookie
2.django操作cookie?
	1.设置:
        response.set_cookie(key,value,max_age=10,)    #设置请求头set-cookie
        response.set_signed_cookie(key,value,salt="")
    2.获取：
    	request.COOKIES[key]   request.COOKIES.get(key)
    3.删除：
    	response.delete_cookie(key)	#本质也是set-cookie,只不过是设置为空，超时设置0

session

1.为什么要有session?
	cookie保存浏览器本地   不安全
    http协议或者浏览器对cookie的大小有限制
2.session是什么？
	session是保存在服务器上一组组键值对，必须依赖cookie

    浏览器第一次请求：根据浏览器来的请求生成一组随机字符串也就是(session_key), 再生成一个字典，字典存放键值对(session_data),设置一个超时时间,存入数据库。服务器做出相应回应session_key放在cookie里面。下次请求从数据库找数据，根据session_key找到session_data,先解密，再反序列化。
    
    
    回浏览器内容    回session_key
4.django里面设置session
	1.设置
		request.session[key] = value
    2.获取
    	request.session[key]    request.session.get[key]
    3.删除
    	del request.session[key]
        request.session.pop(key)
        request.session.delete()#删除所有数据  #不删除cookie 也就是session_id
        request_session.flush()#删除所有数据， 删除cookie
    4.其他
    	request.session.set_expiry(100)  #设置session过期时间
    	request.session.clear_expired()  #清空已经过期session数据
        
settings.py设置：
   		#每次请求都更新session信息
		SESSION_SAVE_EVERY_REQUEST = True
        #浏览器关闭失效
        SESSION_EXPIRE_AT_BROWSER_CLOSE=True
        存储地方：数据库(默认)存储  缓存  缓存+数据库  文件  加密cookie
        #session_data本质，f396ac1768d215221e63cbaa7d358258eefbb7d0:{"login":1}
        
        
        
```

#### 6.ajax

```javascript
//AJAX是JS技术，发请求响应
//Jquery

$.ajax({
	url:'地址',
	type:'post',
	data:{
		k1:'v1',
		k2:'v2'
	},
	sussess:function(res){
		
	},
})

//上传文件
var formdata = new FormData
formdata.append('key','v1')
formdata.append('f1',document.getByElementId('id').files[0])
$.ajax({
	url:'地址',
	type:'post',
	data:FormData,
    conentType:false,   //不需要处理content-Type请求头。否则会变成Urlencode
    processData:flase,  //不需要处理数据的编码
	sussess:function(res){
		
	},
})
//通过django中csrf的校验
前提：必须有cookie    csrftoken
	使用{%csrf_token%} 或
    @ensure_csrf_cooike
1.data中添加csrfmiddlewaretoken的键值对
2.请求中添加x-csrftoken
3.引入文件


```

#### 7.中间件

```python
django的中间件是一个类，处理django的请求和响应的框架级别的钩子。
from django.util.deprecation import MiddlewareMixmin
class Jasdk(MiddlewareMixmin):
	pass
5个方法  4个特点：
1.
def process_request(self,request):
执行时间：路由匹配之前
执行顺序：按照注册顺序执行
返回值：
	None   正常流程
	HttpResponse   当前中间件之后的中间件process_request,路由匹配，视图函数都不执行，直接执行当前中间件process_response方法，倒序执行之前中间件process_response方法
2.
def process_response(self,request,response):
执行时间：大致视图函数之后
执行顺序：按照注册顺序倒序
返回值：
	HttpResponse 必须返回一个HttpResponse对象
3.
def process_view(self,request,view_func,view_args,view_kwargs)
执行时间：路由匹配之后，视图函数之前
执行顺序：按照注册顺序执行
返回值：
	None   正常的流程
    HttpResponse 当前中间件之后的中间件process_view,视图函数都不执行，直接执行最后中间件的process_response方法，倒序执行之前的中间件的process_response方法。
    
4.def process_exception(self,request,exception):
执行时间：视图函数出错
执行顺序：按照注册顺序  倒序执行
返回值：
	None    当前中间件没有处理异常，交由下一个中间件处理，如果所有都不处理，django处理错误
    HttpResponse  当前中间件之前中间件process_exception不执行，直接执行最后一个中间件process_response方法，倒序执行之前的中间件中process_response 方法
5.
def prcess_template_response(self,request,response):
执行时间：视图函数返回的是一个template_resposne或者对象有一个render方法
执行顺序：按照注册顺序  倒序执行
返回值：
	HttpResponse 必须返回对象  可以处理模版，数据
    response.template_name = "模版的文件名"
    response.context_data = {}

```

#### 8.form modelform modelformset

- **form**

```python
#定义form
from django import froms

def sd(value):
    #通过校验规则 ，什么都不干
    #不通过校验规则，抛出异常  ValidationError('xxx')
    


class Form(forms.Form):
	name = forms.CharFiled(
        label="姓名"，
        widget=forms.TextInput(attrs={"class":"form-control"})
        error_messages = {
            "required":"此字段必填"
        } ,
        validators = []#校验器
    )
	age = forms.IntegerFiled()
    gender = forms.ChoiceField(choices=(('1','男'),("2",'女')))
    hobby = forms.MultipleChoiceField()
    def clean_age(self):
        #通过校验规则  返回当前字段值
        #不通过规则  抛出异常 ValidationError('xxxx')
        self.cleaned_data
        
    def clean(self):
        #通过校验规则 返回所有字段值
        #不通过校验规则  抛出异常  ValidationError('xxx')   __all__
        self.add_error('xx','xxxxx')

 
```

- 使用

```
1实例化form
form_obj = RegForm()

2.is_valid() 校验
form_obj.is_valid()

3.持久化存储

```

- 应用小总结

```python
样式：
	label  显示标签
	initial 设置默认值
	error_messages={
		"内置校验":"显示校验不通过提示"
	}
	widget  文本输入样式，PasswordInput 密文显示
					   RadioSelect  单选
	ModelMultpleChoiceField   多选
1.内置校验：
	min_length最小长度
	required 不能空
    max_length 最大长度
2.校验器校验   validators=[]
	RegexValidator   正则匹配    系统自带
	自定义校验器 validators=[testuserinfo,]
	def testuserinfo(value):
		if "a" == value:
			raise ValidationError("错误信息")
3.局部钩子：
	通过校验贵的必须返回当前字段值，不通过抛出异常
	def clean_字段名(self):
		v = self.cleaned_data.get("username")
		if 'xxx' in v:
			raise ValidationError("不正确")
		else:
			return v
	3.1定义需要clean_字段名。
	3.2获取该字段通过self.cleaned_data.get("username")
	3.3校验成功返回该值
	3.4校验不成功触发错误ValidationError
4.全局钩子
	4.1所有校验完毕，最后执行全局钩子。
	4.2从self.cleaned_data获取字段值
	4.3如果校验成功返回所有字段值
	4.4没有通过校验，将错误添加字段里
		self.add_error('re_pwd','量词输入不一致')
		触发错误：ValidationError('两次输入不宜设置')
```





- **modelForm**

```
class Mf(forms.ModelForm):
	class Meta:
		model = models.Customer
		fields = "__all__"
		exclude = []
		
		labels = {}
		error_messages = {}
		widgets = {}
	def __init__(self):
		super().__init__(*args,**kwargs)
		for field in self.fields.values():
			fields.widget.attrs['class']='form-control'
```

- modelformset

  - 看原来补给

