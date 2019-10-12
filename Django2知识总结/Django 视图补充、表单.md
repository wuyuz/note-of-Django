## Django 视图补充、表单



#### 给类视图添加装饰器

在开发中，有时候需要给一些视图添加装饰器，如果用函数视图那么非常简单，只要在函数的上面写上装饰器就可以了，但是要给类加装饰器，那么可以通过以下两种方法：

- 装饰dispatch方法

  ```python
  from django.utils.decorators import method_decorator
  
  def login_required(func):
      def wrapper(request, *args, **kwargs):
          if request.GET.get('username'):
              return func(request, *args, **kwargs)
          else:
              return redirect(reverse('login'))  # 为登陆就跳转至登陆页面
      return wrapper
  
  class IndexView(View):
      def get(self, request, *args, **kwargs):
          return HttpResponse('index')
  
      @method_decorator(login_required)  # 必须登陆
      def dispatch(self,request, *args, **kwargs):
          super(IndexView, self).dispatch(request, *args, **kwargs)
  ```

- 装饰在类名上

  ```python
  from django.utils.decorators import method_decorator
  
  def login_required(func):
      def wrapper(request, *args, **kwargs):
          if request.GET.get('username'):
              return func(request, *args, **kwargs)
          else:
              return redirect(reverse('login'))  # 为登陆就跳转至登陆页面
      return wrapper
  
  @method_decorator(login_required,name='dispatch')
  class IndexView(View):
      def get(self, request, *args, **kwargs):
          return HttpResponse('index')
  ```

  

#### 常见错误码

- 404：服务器没有指定的url
- 403：没有权限访问数据，如csrf_token
- 405: 请求的method错误，请求的方式错误
- 400：请求参数错误，bad request
- 500： 服务器内部错误，代码bug
- 502： 一般时部署错误，nginx启动了？uwsgi有问题？



#### HTML中的表单

Django中的表单丰富了传统HTMLu语言中表单，在Django中主要完成下面两件事：

- 渲染表单模板
- 表单验证数据是否合法



#### Django中表单使用流程

继承django.forms.Form

```python
#forms.py
from django.forms import forms

class MessageForm(forms.Form):
	title= forms.CharField(max_length=3,label='标题',err_messages={'max_length':'字符符过多'})
	content=forms.CharField(widget=forms(Textarea,label='内容'))
	reply = froms.BooleanField(required=False,label='回复')
```

然后再视图中，根据GET或POST来通过引用Form类进行操作

```python
from .forms import MessageForm

class IndexView(View):
	def get(self,request):
		form = MessageForm()
		return render(request,'index.html',{'form':form})
		
	def post(self,request):
		form = MessageForm(request.POST)
		if form.is_valid():
			title=form.cleaned_data.get('title')
			content=form.cleaned_data.get('content')
			return HttpResponse('success')
       	else:
            return HttpResponse('fail')
```



#### 常见验证器

在验证某个字段时候，可以传递一个validators参数来指定验证器，但是有些字段有max_length,默认底层也是调用以下的验证器实现

- MaxValueValidater: 验证最大值、MinValueValudator:验证最小值

- MaxLengthValidater:验证最大长度、MinLengthValidater: 验证最小长度

- EmailValidator: 邮箱

- RegexValidator： 正则验证

  ```python
  from django.core import validators
  
  class MyForm(froms.Form):
  	telephone = forms.CharField(validators=[validators.RegexValidator(r'1[345678]\d{9}'),message='请输入正确手机号格式'])
      
  # models.py中使用
  class Book(models.Model):
     price = models.FloatField(validators=[validators.MaxValidator(limit_value=2000)])
  ```



#### 自定义验证

局部钩子，针对某个字段，进行更全面的验证比如：设计数据库判断等...

```python
from django.core import validators

class MyForm(froms.Form):
	telephone = forms.CharField(validators=[validators.RegexValidator(r'1[345678]\d{9}'),message='请输入正确手机号格式'])
	
	def clean_telephone(self):
		telephone = self.clean_date.get('telephone')
		exists = User.objects.filter(telephone=telephone).exists()
		if exists:
			raise forms.ValidationError('手机不正确')
            
		return telephone # 局部钩子要返回数据
```



#### 全局钩子

针对多个字段进行比较或验证

```python
from django.core import validators

class MyForm(froms.Form):
	telephone = forms.CharField(validators=[validators.RegexValidator(r'1[345678]\d{9}'),message='请输入正确手机号格式'])
	
	def clean(self):
        # 如果来到了clean方法，说明了之前每一个字段都可以验证成功，这里往往做的时直段之间的比较
        cleaned_data = super().clean()
        pwd1 = cleaned_data.get('pwd1')
        pwd2 = cleaned_data.get('pwd2')
        if pwd1 != pwd2:
            raise forms.ValidationError(message='两次密码不一致')
        return cleaned_data
```



#### 提取错误信息

如果在验证失败了，那么有一些错误信息时我们需要传给前端的，这个时候我们就需要通过以下属性：

- form.errors:  获取的错误信息时包含了html标签的错误信息
- form_errors_get_json_data() :这个方法获取到的时字典类型的错误信息，将某个字段的名字为key，错误信息为值
- form.as_json(): 这个方法是将form.get_json_data()返回的字典dump成json格式的字符串，方便传输

```python
from django.core import validators

class MyForm(froms.Form):
	telephone = forms.CharField(validators=[validators.RegexValidator(r'1[345678]\d{9}'),message='请输入正确手机号格式'])
	
	def get_errors(self):
		errors = self.errors.get_json_data()
		new_errors = {}
		for key,message_dicts in errors.items():
			messages = []
			for message_dict in message_dicts:
				message = message_dict['message']
				messages.append(message)
			new_errors[key] = messages
		return new_errors			
	
	def clean(self):
        # 如果来到了clean方法，说明了之前每一个字段都可以验证成功，这里往往做的时直段之间的比较
        cleaned_data = super().clean()
        pwd1 = cleaned_data.get('pwd1')
        pwd2 = cleaned_data.get('pwd2')
        if pwd1 != pwd2:
            raise forms.ValidationError(message='两次密码不一致')
        return cleaned_data
```

views.py文件

```python
from .forms import MessageForm

class IndexView(View):
	def get(self,request):
		form = MessageForm()
		return render(request,'index.html',{'form':form})
		
	def post(self,request):
		form = MessageForm(request.POST)
		if form.is_valid():
			title=form.cleaned_data.get('title')
			content=form.cleaned_data.get('content')
			return HttpResponse('success')
       	else:
       		print(form.get_errors())  # 调用我们定义的方法来获取错误
            return HttpResponse('fail')
```



#### ModelForm

就是与Model扯上关系的Form表单

- Form与ModelForm的区别，也就是说Form并没有绑定某一个模型，但是在Form中的字段名要和数据库中的对齐，不然不行 

  ```python
  #views.py文件中,以后可以将form类分为一个py文件
  from django import forms
  # 按照Django form组件的要求自己写一个类
  class RegForm(forms.Form):
       username = forms.CharField(label='用户名',min_length=6,widget=forms.TextInput(attrs={'class':'form-control'})) #form字段的名称写的是什么，那么前端生成input标签的时候，input标签的name属性的值就是什么
       pwd = forms.CharField(label='密码',min_length=6,widget=forms.PasswordInput(attrs={'class':'form-control'},render_value=True))    # label是相应的标签名，min_length是最小长度，widget是引入的组件其中，可以是用attr添加该标签的样式，render_value是否保存密码
  
  --------------------------------------------------------------------
   # views.py
   # 使用form组件实现注册方式
  def register2(request):
      form_obj = RegForm()
      if request.method == "POST":
          # 实例化form对象的时候，把post提交过来的数据直接传进去
          form_obj = RegForm(data=request.POST)  #既然传过来的input标签的name属性值和form类对应的字段名是一样的，所以接过来后，form就取出对应的form字段名相同的数据进行form校验
          # 调用form_obj校验数据的方法
          if form_obj.is_valid():
              return HttpResponse("注册成功") 
      return render(request, "register2.html", {"form_obj": form_obj})
  
  --------------------------------------------------------------
   #register2.html文件中
   <form action="" method="post" novalidate class="form-horizontal">
          {% csrf_token %}
          <div class="form-group">
              <label name="username" class="col-md-2 col-md-offset-2">用户名</label>
              <div class="col-md-6  " name="username">
                  {{ form_obj.username }}
                  <span class="text-danger">{{ form_obj.username.errors.0 }}</span>
              </div>
          </div>
          <div class="form-group">
              <label name="pwd" class="col-md-2 col-md-offset-2">密码</label>
              <div class="col-md-6  " name="pwd" style="padding-left:0px;">
                  {{ form_obj.pwd }}
                  <span class="text-danger">{{ form_obj.pwd.errors.0 }}</span>
              </div>
          </div>
          <button>注册</button>
  </form>
  ```

  上面的写的表单需要我们一条一条的写，麻烦

  ```python
  from djagno import froms
  
  class MyForm(forms.ModelForm):
  	class Meta:
  		model = A
          fields='__all__'
          
   # 可指定字段：fields = ['title','content']
   # 排除字段： exclude=['price']
   # 监听错误信息：
  		error_message = {
              'title':{
                  'required':'请传入名字',
                  'invalid';'请输入有效参数',
                  'max_length':'xxx'
              },
              ...
          }
      
  # 相应的内置校验器可以通过Validator在定义models时，写在类中：
  # models.py中使用
  class Book(models.Model):
     price = models.FloatField(validators=[validators.MaxValidator(limit_value=2000),message='xxx'])
  ```

- 不一样的save方法：当我们在注册页面时，提交的数据有两个密码，通过验证时save() 会保存所有提取的字段，但是重复的密码字段显得多余，而报错，这是我们可以调用save(commit=False) 来生成一个数据记录对象，这是我们可以进行进一步操作

  ```python
  from django.views.decorators.http import request_POST
  
  @request_POST
  def register(request):
  	form = RegisterForm(request.POST)
  	if form.is_valid():
  		user= form.savr(commit=False)   # 保存模型对象
  		user.password = form.cleaned_data.get('pwd1')  # 操作之后进行保存
  		user.save()
  		return HttpResponse('success')
  	else:
  		return HttpResponse('fail')
  ```

  

#### 文件上传

- 前端的form标签中，enctype='multipart/form-data',不然就不能上传文件

  在form中的input标签中指定input的type='file'

- 后端代码

  ```python
  def save_file(file):
  	with open('somefile.txt','wb') as fp:	
  		for chunk in file.chunks():
  			fp.write(chunk)
  			
  def index(request):
  	if request.method=='GET':
  		form = MyForm()
  		return render(request,'index.html',{'form':form})	
  	else:
  		myfile = request.FILES.get('myfile')
  		save_file(myfile)
  		return HttpResponse('success')
  ```

  

#### 使用模型来自动上传文件

在定义模型的时候，我们可以给存储文件的字段指定为FileField，这个Field可以传递一个upload_toca参数，用来指定的文件保存到哪里

```python
#model.py
class Article(models.Model):
	title = models.CharField(max_length=100)
	content = models.TextField()
	thumbnail = models.FileField(upload_to='files') # 将上传的文件上传到项目同级文件file中，这里存储文件路径
	
# views.py
def index(request):
	if request.method == 'GET':
		return render(request,'index.html')	
	else:
		title = request.POST.get('title')
		content= request.POST.get('content')
		thumbnail = request.POST.get('thumbnail')
		article = Article(title=title,content=content,thumbnail=thumbnail)
		article.save()  # 调用article.save()方法时，就会把下面文件保存到fiels文件夹下，数据库中存放路径，也可以使用create()方法
		return HttpResponse('success')
```

- 当然我们可以指定MEDIA_ROOT和MEDIA_URL

  以上我们使用upload_to来指定上传的文件的目录，我们页可以指定MEDIA_ROOT,就需要在FileFIeld中指定

upload_to，他会自动将文件上传到MEDIA_ROOT的目录下

```python
#setting.py
MEDIA_ROOT=os.path.join(BASE_DIR,'media') # 先创建mdeia文件
MEDIA_URL='/media/'

--------------------------------------------------------
#在总urls.py配置静态文件的路径
from django.urls import path
from app import views
from django.conf.urls.static import static
from django.conf import settings

urlpatterns = [
	path('',views.path),
] + static(settings.MEDIA_URL,document_root=setting.MEDIA_ROOT)  #static返回映射列表，列表+列表

-----------------------------------------------------------
# models.py文件中
class Article(models.Model):
	title = models.CharField(max_length=100)
	content = models.TextField()
	thumbnail = models.FileField(upload_to='%Y/%m/%d/') # 自动在相应mieda中创建文件
```

- 如果要限制上传的文件的扩展名

  ```
  class Article(models.Model):
  	title = models.CharField(max_length=100)
  	content = models.TextField()
  	thumbnail = models.FileField(upload_to='%Y/%m/%d/',validators=[validators.FileExtensionValidator(['txt','pdf']),message='文件后缀有误'])
  ```

- 上传到指定文件

  ```python
  # url.py
  from django.urls import path
  from . import views
  
  app_name = 'app'
  
  urlpatterns = [
      path('register/',views.IndexView.as_view()),
  ]
  
  ------------------------------------------------------
  # view.py
  from django.shortcuts import render,HttpResponse,redirect,reverse
  from django.views.generic import View
  from .forms import ArticleForm
  
  class IndexView(View):
      def get(self, request):
          return render(request,'index.html')
  
      def post(self,request):
          form = ArticleForm(request.POST,request.FILES)
          if form.is_valid:
              form.save()
              return HttpResponse('success')
          else:
              print(form.errors.get_json_data())
              return HttpResponse('fail')
  
      def http_method_not_allowed(self, request, *args, **kwargs):
          return HttpResponse('您访问的方式没有对应的逻辑处理')
  
  --------------------------------------------------------
  #forms.py
  from django import forms
  from .models import Article1
  
  class ArticleForm(forms.ModelForm):
      class Meta:
          model = Article1
          fields = '__all__'
          
  -------------------------------------------------------
  #models.py
  class Article1(models.Model):
      title = models.CharField(max_length=100)
      content = models.TextField()
      my_file = models.FileField(upload_to='%Y/%m/%d/',
                                 validators=[validators.FileExtensionValidator(['txt','pdf'],message='文件缀有误')],
                                 )
  
      class Meta:
          db_table = 'article1'
  
  ---------------------------------------------------------
  <body>
  <form action="" method="post" enctype="multipart/form-data">
      {% csrf_token %}
      标题：<input type="text" name="title">
      内容：<input type="text" name="content">
      文件：<input type="file" name="my_file">  #name要和数据库字段名对应
      <input type="submit" value="提交">
  </form>
  </body>
  ```

  