## Django之form组件

#### forms组件的功能：

```
1、生成HTML标签
2、验证用户数据（显示错误信息）
3、HTML Form提交保留上次提交数据
4、初始化页面显示内容
```

- **校验字段功能**：之前写的视图函数，提交的数据，没有做校验，就添加到数据库里面了。这样是不对的！比如：用户名，必须要符合一定的长度，密码复杂度，等等。forms组件最大的作用就是做**数据校验**。普通做法，一个一个写校验规则，没有解耦。校验规则，都在视图函数里面。（普通的数据验证也是可以的但是视图函数中会有很多的逻辑判断，所以要使用forms组件）

- **与Ajax相比**：

  ```
  1、forms组件只适用于form表单中，且只在提交后从后端得到后端的数据处理再返回前端，换句话说forms组件会刷新页面。
  2、Ajax也能去后端获取验证，(注意Ajax只是给后端发请求，并接收后端的消息)且只是局部刷新，但是校验量有局限性，适用于单跳数据验证，当涉及到多条数据需要验证的时候，还是要使用forms组件
  ```

- 普通视图函数实现验证：

  ```python
  def register(request):
      error_msg = ""
      if request.method == "POST":
          username = request.POST.get("name")
          pwd = request.POST.get("pwd")
          # 对注册信息做校验
          if len(username) < 6:
              # 用户长度小于6位
              error_msg = "用户名长度不能小于6位"
          else:
              # 将用户名和密码存到数据库
              return HttpResponse("注册成功")
      return render(request, "register.html", {"error_msg": error_msg})
      
  ---------------------------------------------------------------
   #login.html文件
  <form action="/reg/" method="post">
      {% csrf_token %}
      <p>
          用户名:<input type="text" name="name">
      </p>
      <p>
          密码：<input type="password" name="pwd">
      </p>
      <p>
          <input type="submit" value="注册">
          <p style="color: red">{{ error_msg }}</p>
      </p>
  </form> 
  ```

- **使用form组件实现注册功能：**首先需要导入一个forms模块，创建一个自己的继承form.Form的校验类，然后再视图函数中写一个视图函数。

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

- **form组件的使用总结**：

  ```python
  1、在views.py文件中导入form模块
  from django import forms
  class RegForm(forms.Form):
      username = forms.CharField()
      pwd = forms.CharField()
      
  2、在视图函数中使用： 
  def register2(request):
      form_obj = RegForm()  # get方式时返回一个form
      if request.method == "POST":  #post时，带着数据再进入form实例化
          form_obj = RegForm(data=request.POST) 
          if form_obj.is_valid():  #数据请求后，判断
              return HttpResponse("注册成功") 
      return render(request, "register2.html", {"form_obj": form_obj})
  ```

- 在模块中使用的方法：

  ```python
  {{ form.as_table }}   ——》  以表格的形式将它们渲染在<tr> 标签中
  {{ form.as_p }}   ——》  将它们渲染在<p> 标签中
  {{ form.as_ul }}    ——》 将它们渲染在<li> 标签中
  {{ form_obj.as_p }}    --》   生产一个个P标签  input  label,用于展示
  {{ form_obj.errors }}    ——》   form表单中所有字段的错误
  {{ form_obj.username }}     ——》 一个字段对应的input框
  {{ form_obj.username.label }}    ——》  该字段的中午提示
  {{ form_obj.username.id_for_label }}    ——》 该字段input框的id
  {{ form_obj.username.errors }}   ——》 该字段的所有的错误
  {{ form_obj.username.errors.0 }}   ——》 该字段的第一个错误的错误
  
  
  {{ form_obj.non_filed_errors }}     ——》  __all__的错误
  ```



#### form常见字段属性以及插件(widget)讲解：

- **initial**：初始值，input框里面的初始值。

  ```python
  class LoginForm(forms.Form):
      username = forms.CharField(  
      	max_length=8, # 只能为8位
          min_length=8, 
          label="用户名",
          initial="张三"  # 设置默认值，输入框的初始值
      )
      pwd = forms.CharField(min_length=6, label="密码")
  ```

- **error_messages**: 重写错误信息。

  ```python
  class LoginForm(forms.Form):
      username = forms.CharField(
          min_length=8,
          label="用户名",
          initial="张三",
          required=True,  # 默认都是True，表示必填
          error_messages={
              "required": "不能为空",
              "invalid": "格式错误",
              "min_length": "用户名最短8位"
          }
      )
      pwd = forms.CharField(min_length=6, label="密码")
  ```

- **radioSelect(插件）**：单radio值为字符串

  ```python
  class LoginForm(forms.Form):
      username = forms.CharField(  #其他选择框或者输入框，基本都是在这个CharField的基础上通过插件来搞的
          min_length=8,
          label="用户名",
          initial="张三",
          error_messages={
              "required": "不能为空",
              "invalid": "格式错误",
              "min_length": "用户名最短8位"
          }
      )
      pwd = forms.CharField(min_length=6, label="密码")
      gender = forms.fields.ChoiceField(
          choices=((1, "男"), (2, "女"), (3, "保密")), #选择栏
          label="性别",
          initial=3,  # 初始值
          widget=forms.widgets.RadioSelect()
      )
  ```

- **单选select**：

  ```python
  class LoginForm(forms.Form):
      ...
      hobby = forms.fields.ChoiceField(  #注意，单选框用的是ChoiceField，并且里面的插件是Select，不然验证的时候会报错， Select a valid choice的错误。
          choices=((1, "篮球"), (2, "足球"), (3, "双色球"), ),
          label="爱好",
          initial=3,
          widget=forms.widgets.Select()
      )
  ```

- **多选select:**

  ```python
  class LoginForm(forms.Form):
      ...
      hobby = forms.fields.MultipleChoiceField( #多选框的时候用MultipleChoiceField，并且里面的插件用的是SelectMultiple，不然验证的时候会报错。
          choices=((1, "篮球"), (2, "足球"), (3, "双色球"), ),
          label="爱好",
          initial=[1, 3],
          widget=forms.widgets.SelectMultiple()
      )
  ```

- **单选框checkbox**：

  ```python
  #单选的checkbox
      class TestForm2(forms.Form):
          keep = forms.ChoiceField(
              choices=(
                  ('True',1),
                  ('False',0),
              ),
  
              label="是否7天内自动登录",
              initial="1",
              widget=forms.widgets.CheckboxInput(), 
          )
          
      选中:'True'   #form只是帮我们做校验,校验选择内容的时候,就是看在没在我们的choices里面,里面有这个值,表示合法,没有就不合法
      没选中:'False'
      ---保存到数据库里面  keep:'True'
      if keep == 'True':
          session 设置有效期7天
      else:
          pass
  ```

- **多选框checkbox**：

  ```python
  class LoginForm(forms.Form):
      ...
      hobby = forms.fields.MultipleChoiceField(
          choices=((1, "篮球"), (2, "足球"), (3, "双色球"),),
          label="爱好",
          initial=[1, 3],
          widget=forms.widgets.CheckboxSelectMultiple()
      )
  ```

- **date类型**：

  ```python
  from django import forms
  from django.forms import widgets
  class BookForm(forms.Form):
      date = forms.DateField(widget=widgets.TextInput(attrs={'type':'date'}))  #必须指定type，不然不能渲染成选择时间的input框
  ```

- **choice字段注意事项**： 如果这个配置的choice的选项是从数据库中获得的，印美初始化Form再视图函数之前，所以无法自动实时更新数据库中的数据，如果想使数据库总的数据能自动更新，我们需要重写构造方法使choice实时更新。

  ```python
   # 方式一：
  from django.forms import Form
  from django.forms import widgets
  from django.forms import fields
  
  class MyForm(Form):
      user = fields.ChoiceField(
          # choices=((1, '上海'), (2, '北京'),), #没有初始化的饿时候会自动添加
          initial=2,
          widget=widgets.Select()
      )
   
      def __init__(self, *args, **kwargs):
          super(MyForm,self).__init__(*args, **kwargs) #注意重写init方法的时候，*args和**kwargs一定要给人家写上，不然会出问题，并且验证总是不能通过，还不显示报错信息
          self.fields['user'].choices = models.Classes.objects.all().values_list('id','caption')  # 每次带着数据回啦都要去数据库获得一次数据，需要注意的是choices接受的是一对列表
          
   # 方式二：
  from django import forms
  from django.forms import fields
  from django.forms import models as form_model
   
  class FInfo(forms.Form):
      authors = forms.ModelMultipleChoiceField(queryset=models.NNewType.objects.all())  # 多选
      #或者下面这种方式，通过forms里面的models中提供的方法也是一样的。
      authors = form_model.ModelMultipleChoiceField(queryset=models.NNewType.objects.all())  # 多选
      
      #authors = form_model.ModelChoiceField(queryset=models.NNewType.objects.all())  # 单选
      #或者
     #authors= forms.ModelChoiceField(queryset=models.Publisth.objects.all(),widget=forms.widgets.Select()) 单选
    	
      
     authors = forms.ModelMultipleChoiceField(
      queryset=models.Author.objects.all(),
      widget = forms.widgets.Select(attrs={'class': 'form-control'}
     ))
     #如果用这种方式，别忘了model表中，NNEWType的__str__方法要写上，不然选择框里面是一个个的object对象
  ```



**常见form内置字段参数**：

```python
Field
    required=True,               是否允许为空
    widget=None,                 HTML插件
    label=None,                  用于生成Label标签或显示内容
    initial=None,                初始值
    help_text='',                帮助信息(在标签旁边显示)
    error_messages=None,         错误信息 {'required': '不能为空', 'invalid': '格式错误'}
    validators=[],               自定义验证规则
    localize=False,              是否支持本地化
    disabled=False,              是否可以编辑
    label_suffix=None            Label内容后缀
  
CharField(Field)
    max_length=None,             最大长度
    min_length=None,             最小长度
    strip=True                   是否移除用户输入空白
 
IntegerField(Field)
    max_value=None,              最大值
    min_value=None,              最小值
 
FloatField(IntegerField)
    ...
 
DecimalField(IntegerField)
    max_value=None,              最大值
    min_value=None,              最小值
    max_digits=None,             总长度
    decimal_places=None,         小数位长度
 
BaseTemporalField(Field)
    input_formats=None          时间格式化   
 
DateField(BaseTemporalField)    格式：2015-09-01
TimeField(BaseTemporalField)    格式：11:12
DateTimeField(BaseTemporalField)格式：2015-09-01 11:12
 
DurationField(Field)            时间间隔：%d %H:%M:%S.%f
    ...
 
RegexField(CharField)
    regex,                      自定制正则表达式
    max_length=None,            最大长度
    min_length=None,            最小长度
    error_message=None,         忽略，错误信息使用 error_messages={'invalid': '...'}
```



#### 字段校验

- **内置校验**：

  ```
  required=True
  min_length
  max_length
  ```

- **自定义校验器**：都是针对字段属性validationError=[]

  - 1、自定义函数校验器：

    ```python
    from django.core.exceptions import ValidationError
    from django import forms
    def checkname(value):  # 这是填入validators列表中的自定义规则，value是浏览器传回来的该字段对应的值
        # 通过校验规则 不做任何操作
        # 不通过校验规则   抛出异常
        if 'alex' in value:
            raise ValidationError('不符合社会主义核心价值观')
            
     # 视图函数中使用
     class RegForm(forms.Form):
        username = forms.CharField(label='用户名',min_length=6,
                                   widget=forms.TextInput(attrs={'class':'form-control'}),
                                   initial='李四',
                                   required=True,
                                   validators=[checkname,], #将函数名填入
                                   error_messages={
                                       'required':'用户名必填',
                                       'min_length':'长度大于6',
                                   })
    ```

  - 使用内置的校验器：

    ```python
    from django.core.validators import RegexValidator
    from django import forms
    
    class MyForm(Form):
         phone = forms.CharField(
            validators=[RegexValidator(r'^1[1-9]\d{9}$','手机号格式不正确')],
        )    
    ```

  

#### 钩子函数（Hook）：

​	再之前我们学到的字段校验的两种方式，下面我们将使用钩子函数定制自己的校验规则。有个很大的坑，就是执行**顺序的优先级**。

- **局部钩子**：对一个字段进行校验（局部），为什么叫钩子？因为在源码中使用：if hasattr(self,'clean_%s'%name): 去触发后面的函数，只有我们在类中谢了这个函数才能触发（就像鱼钩一样，只有你咬了，才会触发相应的事件）。局部钩子，正常需要返回该字段的值，异常则要主动抛异常

  ```python
  class ReForm(forms.Form):
      username = forms.CharField(
          # min_length=6,  # 注意要是有内置校验字段，最先执行这，如果满足了直接不会再执行自定义函数、钩子函数、全局函数
          widget=forms.widgets.TextInput(attrs={"class": "form-control"})
      )
  
      def clean_username(self):
          value = self.cleaned_data.get("username")  # 从cleaned_data中取出值，正确则返会
          if "666" in value:
              raise ValidationError("光喊666是不行的")  # 不抛错，报错信息无法加入，但是可以自己添加到self.add_error中
          else:
              return value
  ```

  

- **全局钩子**：一般用户所有字段验证都通过了，最后一步的校验。全局钩子通过的验证要return self.cleaned_data,没通过的要主动抛异常

  ```python
  from django import forms
  from django.core.exceptions import ValidationError
  from django.core.validators import RegexValidator
  class RegForm(forms.Form):
      username = forms.CharField(label='用户名',min_length=1,
                                 widget=forms.TextInput(attrs={'class':'form-control'}),
                                 initial='李四',
                                 required=True,
                                 validators=[checkname,],
                                 error_messages={
                                     'required':'用户名必填',
                                     'min_length':'长度大于6',
                                 })
      pwd = forms.CharField(label='密码',min_length=6,widget=forms.PasswordInput(attrs={'class':'form-control'}))
      gender = forms.ChoiceField(
          choices=((1,'男'),(0,'女')),
          label="性别",
          initial=1,)
      re_pwd = forms.CharField(label='确认密码', min_length=6, widget=forms.PasswordInput(attrs={'class': 'form-control'}))
          
      hobby = forms.MultipleChoiceField(
          # choices=models.Class.objects.values_list('id','name'),
          label="班级",
          initial=[1, 3],
          widget=forms.widgets.CheckboxSelectMultiple()
      )
      phone = forms.CharField(
          validators=[RegexValidator(r'^1[1-9]\d{9}$','手机号格式不正确')])
      
      def __init__(self,*args,**kwargs):
          super().__init__(*args,**kwargs)
          self.fields['hobby'].choices = models.Class.objects.values_list('id','name')
      
      def clean_username(self):
          print('ok')
          v = self.cleaned_data.get('username')
          print(v)
          if 'wang' in v:
              raise ValidationError('太多了')
          else:
              return v
          
      def clean(self):  # 全局钩子校验密码
          pwd = self.cleaned_data.get('pwd','')
          re_pwd = self.cleaned_data.get('re_pwd')  # 可能两个都为空
          if pwd == re_pwd:
              return self.cleaned_data   # 正常通过必须返回cleaned_data
          else:
              self.add_error('re_pwd','两次密码不一致')  # 自己添加进一般错误
              raise ValidationError('两次密码不一致')  # 默认添加到__all__中
                        
  -----------------------------------------------------
   # 视图函数
  def test(request):
      form_obj = RegForm()
      if request.method=='POST':
          form_obj = RegForm(data=request.POST) # 前端回来的数据
          if form_obj.is_valid():
              return HttpResponse('注册成功')
      return render(request, 'adds_student.html',{'form_obj':form_obj})
  ```

  

#### is_valid() 的流程：

- 总的来说，就是对前端返回的数据进行验证等顺序： 1内置校验规则  -> 2自定义校验器 ->  3局部钩子  -> 4全局钩子   ( 内置校验规则和自定义校验能第一时间打印，其他的钩子函数是再其后面的，如果前面通过了才有他们出现，且全局钩子的错误出现再__all\__中，可以去[0], 或则自己添加进一般错误队列： self.add_error('re_pwd','两次密码不一致') )

  ```
  1、执行full_clean()的方法：
  	定义错误字典
  	定义存放清洗数据的字典
  2、执行_clean_fields方法：
  	2.1、循环所有的字段
  	2.2、获取当前的值
  	2.3、对字段进行校验 （ 1内置校验规则   2自定义校验器）
  		2.3.1、通过校验：
          	self.cleaned_data[name] = value 如果有3局部钩子，执行进行校验：
  			通过校验——》 self.cleaned_data[name] = value 不通过校验——》     			 self._errors 添加当前字段的错误 并且 self.cleaned_data中当前字段			 的值删除掉。
  	     2.3.2、没有通过校验：self._errors  添加当前字段的错误
  3、执行全局钩子 clean方法  
     注意：每个字段进行校验，一个字段校验不成功，就将错误信息添加到错误信息表，并不会返回错误值（如果取，则为空，取不到），等全部字段都校验成功后，才走全局钩子。
  ```

  

#### form组件使用技巧：

- **HTML方面：使用for循环渲染html文件**：配合bootstrap，注意在form标签后面增加novalidate，表示关闭bootstrap表单验证（关闭前端校验）

  ```
 <\html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <link rel="stylesheet" href="https://cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap.min.css">
  </head>
  <body>
  <div class="container">
      <div class="row">
          <div class="col-md-6 col-md-offset-2">
          <h3>添加用户</h3><br/>
              <form action="" method="post" novalidate>
                  {% csrf_token %}
                  {% for field in form %}
                      <div class="form-group">
                          <label for="">{{ field.label }}</label>
                          {{ field }}<span class="text-danger" >{{ field.errors.0 }}</span>
                      </div>
                  {% endfor %}
                  <br/>
                  <input type="submit" class="btn btn-success btn-sm">
              </form>
          </div>
      </div>
  </div>
  ```



- **视图函数方面： Widgets**：Widget 是Django 对HTML 输入元素的表示。Widget 负责渲染HTML和提取GET/POST 字典中的数据。如果你想让某个Widget 实例与其它Widget 看上去不一样，你需要在Widget 对象实例化并赋值给一个表单字段时指定额外的属性（以及可能需要在你的CSS 文件中添加一些规则）

  ```python
  from django.shortcuts import render,HttpResponse
  from django import forms  # 必须导入模块
  from django.forms import widgets
  
  class UserForm(forms.Form):  # 必须继承Form
      #定义变量，专门给text类型的输入框添加class
      wid = widgets.TextInput(attrs={"class":"form-control"})
      #限制数据为字符串，最小长度4,最大长度12
      name = forms.CharField(min_length=4,max_length=12,label="姓名",widget=wid)
      age = forms.IntegerField(label="年龄",widget=wid)  # 限制为数字
      # 限制为邮箱格式
      email = forms.EmailField(label="邮箱",widget=widgets.EmailInput(attrs={"class":"form-control"}))
      #限制长度为11位
      tel = forms.CharField(min_length=11,max_length=11,label="手机号码",widget=wid)
  
  def index(request):
      return render(request,"index.html")
  
  def adduser(request):
      if request.method == "POST":    
          form = UserForm(request.POST) # 将post数据传给UserForm,每次都验证
          print(request.POST)
          if form.is_valid():  # 验证数据
              print("###success###")
              print(form.cleaned_data)  # 所有干净的字段以及对应的值
              # ErrorDict : {"校验错误的字段":["错误信息",]}
              print(form.errors)
              print(type(form.errors))  # 打印
          else:
              print("###fail###")
              print(form.cleaned_data)
              print(form.errors)
              # 获取email错误信息,返回一个错误列表,可以切片
              print(form.errors.get("email"))
              # 获取第一个错误信息
              print(form.errors.get("email")[0])
          return render(request, "adduser.html", {"form":form})
      else:
          form = UserForm()  #get是初始化from
          return render(request,"adduser.html",{"form":form})
   # attrs 表示设置css样式，它接收一个字典，可以写多个css样式！    
  ```




### Django 之 ModelForm

​		通常在Django项目中，我们编写的大部分都是与Django 的模型紧密映射的表单。 举个例子，你也许会有个Book 模型，并且你还想创建一个form表单用来添加和编辑书籍信息到这个模型中。 在这种情况下，在form表单中定义字段将是冗余的，因为我们已经在模型中定义了那些字段。基于这个原因，Django 提供一个辅助类来让我们可以从Django 的模型创建Form，这就是ModelForm。（将基于Model来创建form）

- **modelForm定义：**form与model的终极结合，会根据你model中的字段转换成对应的form字段，并且并你生成标签等操作。

  ```python
   # 比如此时你的models.py文件中有如下模型
  class Book(models.Model):
      nid = models.AutoField(primary_key=True)
      title = models.CharField( max_length=32)
      publishDate=models.DateField()
      price=models.DecimalField(max_digits=5,decimal_places=2)
      publish=models.ForeignKey(to="Publish",to_field="nid")
      authors=models.ManyToManyField(to='Author',)
      def __str__(self):
          return self.title 
      
    # 在views.py文件中写如下代码，当然后面我们将这个生成form的代码，单独写进一个模块中
  class BookForm(forms.ModelForm):
      class Meta:
          model = models.Book
          fields = "__all__"  # 这将导入所有字段，当然我们可以使用['nid','title',...] 来单独提取想要的字段
          labels = {
              "title": "书名",
              "price": "价格"
          }
          widgets = {  # 给相应的字段添加属性
              "password": forms.widgets.PasswordInput(attrs={"class": "c1"}),
              "publishDate": forms.widgets.DateInput(attrs={"type": "date"}),
          }
          
          
    # class Meta常用参数：
  model = models.Book  # 对应的Model中的类
  fields = "__all__"  # 字段，如果是__all__,就是表示列出所有的字段
  exclude = None  # 排除的字段，排除的字段
  labels = None  # 提示信息
  help_texts = None  # 帮助提示信息
  widgets = None  # 自定义插件
  error_messages = None  # 自定义错误信息
  error_messages = {
      'title':{'required':'不能为空',...} #每个字段的所有的错误都可以写，...是省略的意思，复制黏贴我代码的时候别忘了删了...
  }
  ```

- **批量添加样式：** 和form的一样

  ```python
  class BookForm(forms.ModelForm):
      r_password = forms.CharField() #想多验证一些字段可以单独拿出来写，按照form的写法，写在Meta的上面或者下面都可以，原本模型中没有，就自己创建，如果原有字段想重写，就直接写就是了，默认会使用当前字段的，相当于重写
      class Meta:
          model = models.Book
          # fields = ['title','price']
          fields = "__all__" #['title,'price'] 指定字段生成form
          # exclude=['title',] #排除字段
          labels = {
              "title": "书名",
              "price": "价格"
          }
          error_messages = {
              'title':{'required':'不能为空',} #每个字段的错误都可以写
          }
      #如果models中的字段和咱们需要验证的字段对不齐的是，比如注册时，咱们需要验证密码和确认密码两个字段数据，但是后端数据库就保存一个数据就行，那么验证是两个，数据保存是一个，就可以再接着写form字段
      r_password = forms.CharField()。
      #同样的，如果想做一些特殊的验证定制，那么和form一昂，也是那两个钩子（全局和局部），写法也是form那个的写法，直接在咱们的类里面写：
      #局部钩子：
      def clean_title(self):
          pass
  　　#全局钩子
      def clean(self):
          pass
      def __init__(self,*args,**kwargs): #批量操作
          super().__init__(*args,**kwargs)
          for field in self.fields:
              #field.error_messages = {'required':'不能为空'} #批量添加错误信息,这是都一样的错误，不一样的还是要单独写。
              self.fields[field].widget.attrs.update({'class':'form-control'})
  ```

- **ModelForm的验证：**与普通的Form表单验证类型类似，ModelForm表单的验证在调用is_valid() 或访问errors 属性时隐式调用。我们可以像使用Form类一样自定义局部钩子方法和全局钩子方法来实现自定义的校验规则。如果我们不重写具体字段并设置validators属性的话，ModelForm是按照模型中字段的validators来校验的。

- **save() 方法**：

  ```python
  	# 每个ModelForm还具有一个save()方法。 这个方法根据表单绑定的数据创建并保存数据库对象。 ModelForm的子类可以接受现有的模型实例作为关键字参数instance；如果提供此功能，则save()将更新该实例。 如果没有提供，save() 将创建模型的一个新实例： （传统的create(**dic)方式也是可以的，但是比较麻烦，当一组数据经过校验了，我们可以通过在is_vaild()函数后，取出清除的数据form_obj.cleaned_data(),所以使用models.user.object.create(**form_obj.cleaned_data)也是可以的）
      
  >>> from myapp.models import Book
  >>> from myapp.forms import BookForm
  
  # 根据POST数据创建一个新的form对象
  >>> form_obj = BookForm(request.POST)
  
  # 创建书籍对象
  >>> new_ book = form_obj.save()
  
  # 基于一个书籍对象创建form对象
  >>> edit_obj = Book.objects.get(id=1)
  # 使用POST提交的数据更新书籍对象
  >>> form_obj = BookForm(request.POST, instance=edit_obj)
  >>> form_obj.save()   
  
  ------------------------------------------------------------------------
   # 之前我们通过form组件来保存书籍数据的时候的写法
  def index(request):
      if request.method == 'GET':
          form_obj = BookForm()
          return render(request,'index.html',{'form_obj':form_obj})
  
      else:
          form_obj = BookForm(request.POST)
          if form_obj.is_valid():
              # authors_obj = form_obj.cleaned_data.pop('authors')
              # new_book_obj = models.Book.objects.create(**form_obj.cleaned_data)
              # new_book_obj.authors.add(*authors_obj)
              form_obj.save()  #因为我们再Meta中指定了是哪张表，所以它会自动识别，不管是外键还是多对多等，都会自行处理保存，它完成的就是上面三句话做的事情，并且还有就是如果你验证的数据比你后端数据表中的字段多，那么他自会自动剔除多余的不需要保存的字段，比如那个重复确认密码就不要保存
              return redirect('show')
  
          else:
              print(form_obj.errors)
              return render(request,'index.html',{'form_obj':form_obj})    
  ```

  

- **实列**:使用modelForm来创建一个页面

  ```python
    # html文件中
  {% load static %}
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <link rel="stylesheet" href="{% static 'bootstrap-3.3.0-dist/dist/css/bootstrap.min.css' %}">
  </head>
  <body>
  
  <h1>编辑页面</h1>
  <div class="container-fluid">
      <div class="row">
          <div class="col-md-6 col-md-offset-3">
              <form action="{% url 'edit_book' n %}" novalidate method="post">
                  {% csrf_token %}
                  {% for field in form %}
                      <div class="form-group">
                          <label for="{{ field.id_for_label }}">{{ field.label }}</label>
                          {{ field }}
                          <span class="text-danger">{{ field.errors.0 }}</span>
                      </div>
                  {% endfor %}
  
                  <div class="form-group">
                      <input type="submit" class="btn btn-primary pull-right">
                  </div>
  
              </form>
          </div>
      </div>
  </div>
  </body>
  <script src="{% static 'bootstrap-3.3.0-dist/dist/jQuery/jquery-3.1.1.js' %}"></script>
  <script src="{% static 'bootstrap-3.3.0-dist/dist/js/bootstrap.min.js' %}"></script>
  </html>
   
      ------------------------------------------------------------------
    # 相应的views.py文件中
  def edit_book(request,n):
  
      book_obj = models.Book.objects.filter(pk=n).first()
      if request.method == 'GET':
          # all_authors = models.Author.objects.all() #
          # all_publish = models.Publish.objects.all()
  
          form = BookForm(instance=book_obj)
  
          return render(request,'edit_book.html',{'form':form,'n':n}) #传递的这个n参数是给form表单提交数据的是的action的url用的，因为它需要一个参数来识别是更新的哪条记录
      else:
          form = BookForm(request.POST,instance=book_obj) #必须指定instance，不然我们调用save方法的是又变成了添加操作
          if form.is_valid():
              form.save()
              return redirect('show')
          else:
              return render(request,'edit_book.html',{'form':form,'n':n})  
    
  ```



- **总结加补充：**这是一个神奇的组件，通过名字我们可以看出来，这个组件的功能就是把model和form组合起来，先来一个简单的例子来看一下这个东西怎么用：比如我们的数据库中有这样一张学生表，字段有姓名，年龄，爱好，邮箱，电话，住址，注册时间等等一大堆信息，现在让你写一个创建学生的页面，你的后台应该怎么写呢？首先我们会在前端一个一个罗列出这些字段，让用户去填写，然后我们从后天一个一个接收用户的输入，创建一个新的学生对象，保存其实，重点不是这些，而是合法性验证，我们需要在前端判断用户输入是否合法，比如姓名必须在多少字符以内，电话号码必须是多少位的数字，邮箱必须是邮箱的格式这些当然可以一点一点手动写限制，各种判断，这毫无问题，除了麻烦我们现在有个更优雅（以后在Python相关的内容里，要多用“优雅”这个词，并且养成习惯）的方法：ModelForm先来简单的，生硬的把它用上，再来加验证条件。

  - **创建modleform**：

    ```python
     #首先导入ModelForm
    
    from django.forms import ModelForm
    #在视图函数中，定义一个类，比如就叫StudentList，这个类要继承ModelForm，在这个类中再写一个原类Meta（规定写法，并注意首字母是大写的）
     #在这个原类中，有以下属性（部分）：
    
    class StudentList(ModelForm):
        class Meta:
            model =Student #对应的Model中的类
            fields = "__all__" #字段，如果是__all__,就是表示列出所有的字段
            exclude = None #排除的字段
            #error_messages用法：
            error_messages = {
            	'name':{'required':"用户名不能为空",},
            	'age':{'required':"年龄不能为空",},
            }
            #widgets用法,比如把输入用户名的input框给为Textarea
            
            #首先得导入模块
            from django.forms import widgets as wid #因为重名，所以起个别名
            widgets = {
            	"name":wid.Textarea(attrs={"class":"c1"}) #还可以自定义属性
            }
            #labels，自定义在前端显示的名字
            labels= {
            	"name":"用户名"
            }
            
      # 　然后在url对应的视图函数中实例化这个类，把这个对象传给前端
    def student(request):
        if request.method == 'GET':
            student_list = StudentList()
            return render(request,'student.html',{'student_list':student_list})
     
    ```

    ​    然后前端只需要 {{ student_list.as_p }} 一下，所有的字段就都出来了，可以用as_p显示全部，也可以通过for循环这student_list，拿到的是一个个input框，现在我们就不用as_p，手动把这些input框搞出来，as_p拿到的页面太丑。首先 for循环这个student_list，拿到student对象，直接在前端打印这个student，是个input框student.label ，拿到数据库中每个字段的verbose_name ,如果没有设置这个属性，拿到的默认就是字段名，还可以通过student.errors.0 拿到错误信息有了这些，我们就可以通过bootstrap，自己拼出来想要的样式了，比如：

    ```html
    <body>
    <div class="container">
        <h1>student</h1>
        <form method="POST" novalidate>
            {% csrf_token %}
            {# {{ student_list.as_p }}#}
            {% for student in student_list %}
                <div class="form-group col-md-6">
                    {# 拿到数据字段的verbose_name,没有就默认显示字段名 #}
                    <label class="col-md-3 control-label">{{ student.label }}</label>
                    <div class="col-md-9" style="position: relative;">{{ student }}</div>
                </div>
            {% endfor %}
            <div class="col-md-2 col-md-offset-10">
                <input type="submit" value="提交" class="btn-primary">
            </div>
        </form>
    </div>
    </body>
    ```

  -  **添加记录**：保存数据的时候，不用挨个取数据了，只需要save一下

    ```python
    def student(request):
    
        if request.method == 'GET':
             student_list = StudentList()
             return render(request,'student.html',{'student_list':student_list})
        else:
             student_list = StudentList(request.POST)
             if student_list.is_valid():
             student_list.save()
             return redirect(request,'student_list.html',{'student_list':student_list})  
    ```

  - **编辑数据：**　如果不用ModelForm，编辑的时候得显示之前的数据吧，还得挨个取一遍值，如果ModelForm，只需要加一个instance=obj（obj是要修改的数据库的一条数据的对象）就可以得到同样的效果，保存的时候要注意，一定要注意有这个对象（instance=obj），否则不知道更新哪一个数据，代码示例：

    ```python
    from django.shortcuts import render,HttpResponse,redirect
    from django.forms import ModelForm
    
    from app01 import models
    def test(request):
        model_form = models.Student.objects.all()
        return render(request,'test.html',{'model_form':model_form})
    
    class StudentList(ModelForm):
        class Meta:
            model = models.Student #对应的Model中的类
            fields = "__all__" #字段，如果是__all__,就是表示列出所有的字段
            exclude = None #排除的字段
            labels = None #提示信息
            help_texts = None #帮助提示信息
            widgets = None #自定义插件
            error_messages = None #自定义错误信息
            #error_messages用法：
            error_messages = {
            	'name':{'required':"用户名不能为空",},
            	'age':{'required':"年龄不能为空",},
            }
            #widgets用法,比如把输入用户名的input框给为Textarea
            #首先得导入模块
            from django.forms import widgets as wid #因为重名，所以起个别名
            widgets = {
           	 	"name":wid.Textarea
            }
            #labels，自定义在前端显示的名字
            labels= {
            	"name":"用户名"
            }
    def student(request):
        if request.method == 'GET':
            student_list = StudentList()
            return render(request,'student.html',{'student_list':student_list})
        else:
            student_list = StudentList(request.POST)
            if student_list.is_valid():
                student_list.save()
                return render(request,'student.html',{'student_list':student_list})
    
    def student_edit(request,pk):
        obj = models.Student.objects.filter(pk=pk).first()
        if not obj:
            return redirect('test')
        if request.method == "GET":
            student_list = StudentList(instance=obj)
            return render(request,'student_edit.html',{'student_list':student_list})
        else:
            student_list = StudentList(request.POST,instance=obj)
            if student_list.is_valid():
                student_list.save()
                return render(request,'student_edit.html',{'student_list':student_list})
    ```

    

**登陆实列**：我将ModelForm封装成一个forms.py文件内容如下

```python
from django import forms
from django.core.exceptions import ValidationError
from Ai_crm import models
import hashlib


class RegForm(forms.ModelForm):
    # form写的太多了，使用modelform
    department = forms.ModelChoiceField(queryset=models.Department.objects.all(), initial=1,
                                        widget=forms.widgets.Select())
    password = forms.CharField(min_length=6, widget=forms.PasswordInput(
        attrs={'placeholder': '密码', 'autocomplete': 'off'}))  # 自己重写password
    re_password = forms.CharField(min_length=6, widget=forms.PasswordInput(
        attrs={'placeholder': '确认密码', 'autocomplete': 'off'}))  # 自己新增字段
    mobile = forms.CharField(
        min_length=11,
        widget=forms.TextInput(attrs={'placeholder': '手机号', 'autocomplete': 'off'}),
        error_messages={
            "min_length": "用户名最短11位",
        }
    )
    
    class Meta:
        model = models.UserProfile  # 引入用户表
        fields = '__all__'
        exclude = ['is_active', ]  # 排除此字段
        widgets = {
            'username': forms.EmailInput(attrs={'placeholder': '用户名', 'autocomplete': 'off'}),
            'mobile': forms.TextInput(attrs={'placeholder': '手机号', 'autocomplete': 'off'}),
            'name': forms.TextInput(attrs={'placeholder': '姓名', 'autocomplete': 'off'}),
            # 'department':forms.Select(attrs={'class':'form-control'}), # 这种方式有----，所以我重写此字段
        }
        error_messages = {  # 设置每个字段的报错，这里的报错只针对widgets，或则说是许哟啊自动生成的字段的一些简单报错处理，要想自定义需重写字段如上的手机号
            'username': {
                'required': '必填',
                'invalid': '邮箱格式不正确',
            },
        }
    
    # 使用全局钩子来校验密码，局部钩子不能确保谁前谁后
    def clean(self):
        self._validate_unique = True  # 防止字段重复，Form中自带，ModelForm中必须自己添加
        password = self.cleaned_data.get('password')
        re_password = self.cleaned_data.get('re_password')
        if password == re_password:
            md5 = hashlib.md5()
            md5.update(password.encode('utf-8'))
            self.cleaned_data['password'] = md5.hexdigest()
            return self.cleaned_data
        else:
            self.add_error('re_password', '两次密码不一致')
            raise ValidationError('两次密码不一致！')  # 不抛错，就无法触发异常

 ----------------------------------------------------------
 # views.py问题
from django.shortcuts import render,redirect,reverse
from  Ai_crm import models
import hashlib
from Ai_crm.forms import RegForm
from django.db.models import F

def login(request):
    if request.method =='POST':
        username = request.POST.get('username')
        password = request.POST.get('password')
        md5 = hashlib.md5()
        md5.update(password.encode('utf-8'))
        obj = models.UserProfile.objects.filter(username=username, password=md5.hexdigest(), is_active=True).first()
        if obj:
            return redirect('index')
    return render(request,'login.html')

def reg(request):
    form_obj = RegForm()
    
    if request.method =="POST":
        form_obj = RegForm(request.POST)
        if form_obj.is_valid():
            # models.UserProfile.objects.create(**form_obj.cleaned_data)  # 方式一
            obj = form_obj.save()
            models.Department.objects.filter(userprofile=obj).update(count=F('count')+1) # 计数
            return redirect('login')            
    return render(request,'reg.html',{'form_obj':form_obj})


def index(request):
    return render(request,'index.html')

def customer(request):
    customer_list = models.Customer.objects.all()
    print(customer_list)
    return render(request, 'customer_list.html',{'list':customer_list})    
```



#### 后期整理为父类继承版本：

- 在应用的form.py文件中新建一个父类用于给生成的字段添加样式

  ```python
  class BSModelForm(forms.ModelForm):
      def __init__(self, *args, **kwargs):
          super().__init__(*args, **kwargs)
          
          for field in self.fields.values():
              if isinstance(field, (MultiSelectFormField, forms.BooleanField)):  # 排除几种特殊情况
                  continue
              field.widget.attrs['class'] = 'form-control'
  
  class ConsultRecordForm(BSModelForm):  # 使用继承，少写代码
      class Meta:
          model = models.ConsultRecord
          fields = '__all__'
      
      def __init__(self, request, customer_id, *args, **kwargs):
          super().__init__(*args, **kwargs)  # 不要传入父类
          if customer_id and customer_id != '0':  # 跳过我们自制的0
              self.fields['customer'].choices = [('', '----------'), ] + [(i.pk, str(i)) for i in               models.Customer.objects.filter(pk=customer_id)]
          else:
              self.fields['customer'].choices = [('', '----------'), ] + [(i.pk, str(i)) for i in
  			request.user_obj.customers.all()]
          self.fields['consultant'].choices = [(request.user_obj.pk, request.user_obj), ]
  
  class ClassListForm(BSModelForm):
      class Meta:
          model = models.ClassList
          fields = '__all__'
  
  ```

  