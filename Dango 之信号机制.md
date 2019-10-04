## Django 之信号机制

#### 概念

Django 提供一个“信号分发器”，允许解耦的应用在框架的其它地方发生操作时会被通知到。
也就是说在特定事件发生时，可以发送一个信号去通知注册了这个信号的一个或者多个回调，在回调里进行逻辑处理。**在发生特定情况时触发的一个或者一系列事件**。

#### 如何监听信号

- 使用Django为我们提供的Single模块，且为我们提供了足够多的信号机制，当然我们可以自定义信号，通过继承模块

  ```python
  from django.dispatch import Signal, receiver
  ```

- 注册信号，使用Single模块中的特定函数注册一个信号，然后通过connect函数将事件注册到函数中（或者装饰器@receiver)

  ```python
  signalAllen = Signal(providing_args=['allen'])  
  	#注册一个信号，注意providing_args表示触发时，除了sender参数外，还要的参数名，这里时allen
  ```

- 触发函数，在特定的位置触发信号中的函数，当然这都是在一个请求与相应之中完成的；而内置的Django信号机制已经将触发函数的机制写入了源码中

  ```python
  signalAllen.send(sender=None, allen='test')
   #在内置的Django信号中，会默认设置sender，我们只需要使用即可，这里是我们自定义的
  ```



#### Django的内置信号函数

已经为我们写好了信号，我们只需要使用装饰器/connect将，信号需要的逻辑函数写好，在相应的时刻就会触发函数

```python
Model signals
    pre_init                    # django的modal执行其构造方法前，自动触发
    post_init                   # django的modal执行其构造方法后，自动触发
    pre_save                    # django的modal对象保存前，自动触发
    post_save                   # django的modal对象保存后，自动触发
    pre_delete                  # django的modal对象删除前，自动触发
    post_delete                 # django的modal对象删除后，自动触发
    m2m_changed                 # django的modal中使用m2m字段操作第三张表（add,remove,clear）前后，自动触发
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



#### 简单自定义信号实例

```python
from django.shortcuts import render,HttpResponse
from django.dispatch import Signal, receiver

 #生成一个信号
signalAllen = Signal(providing_args=['allen'])

 #方式一：通过receiver装饰器注册
@receiver(signalAllen)
def signal_callback(sender, **kwargs):
    print(sender, kwargs)
    print('signal_callback called')

# 可以定义多个
@receiver(signalAllen)
def signal_callback1(sender, **kwargs):
    print(sender, kwargs)
    print('signal_callback1 called')


def signal_callback2(sender,**kwargs): #自定义sender必须写，表示作用于谁
    print("发送者",sender)
    print('参数',kwargs)

 #方式二：通过 信号.connect() 函数注册到信号中
signalAllen.connect(signal_callback2)


def index(request):
    # 通过 信号.disconnect() 删除注册的函数
    signalAllen.disconnect(signal_callback)  # 将signal_callback函数从注册信号中删除

     # 通过 信号.send() 函数执行信号中的函数
    signalAllen.send(sender=None,allen='test') # 同时会触发三个注册到signalAllen信号中的函数
    return HttpResponse('ok')

```



#### Django内置信号演示

```python
from app import models
from django.forms import ModelForm

class RegForm(ModelForm):
    class Meta:
        model = models.Users
        fields = "__all__"


class RegForm1(ModelForm):
    class Meta:
        model = models.Users1
        fields = "__all__"

from django.db.models.signals import pre_save, post_save

# sender 表示针对那一张表进行信号验证，如果不写，表示对每一张表都进行触发
# 比如 我的post_save写了sender，这里没写，那么只会触发这一个
@receiver(pre_save,sender=models.Users)
def pre_save_callback(sender, **kwargs):
    print(sender,kwargs)
    print('pre_save_callback')


@receiver(post_save,sender=models.Users) #可指定是那张表
def post_save_callback(sender, **kwargs):
    print(sender,kwargs) #<class 'app.models.Users'> {'signal': <django.db.models.signals.ModelSignal object at 0x000001D1D387DD30>, 'instance': <Users: Users object>, 'created': True, 'update_fields': None, 'raw': False, 'using': 'default'}

    print('post_save_callback')

def reg(request):
    form_obj = RegForm()
    if request.method == "POST":
        form_obj = RegForm(request.POST)
        if form_obj.is_valid():
            form_obj.save()   #在save中自动触发信号的send函数
    return  render(request,'reg.html',{'form_obj':form_obj})

def reg1(request):
    form_obj = RegForm1()
    if request.method == "POST":
        form_obj = RegForm1(request.POST)
        if form_obj.is_valid():  
            form_obj.save()
    return  render(request,'reg.html',{'form_obj':form_obj})


--------------------------------------------------------
 #models.py中
from django.db import models

class Users(models.Model):
    name = models.CharField(max_length=12,verbose_name='名字')
    pwd = models.CharField(max_length=12, verbose_name='密码')

class Users1(models.Model):
    name = models.CharField(max_length=12,verbose_name='名字')
    pwd = models.CharField(max_length=12, verbose_name='密码')    
 
-----------------------------------------------------------
 #url.py   
urlpatterns = [
    url(r'reg/',views.reg),
    url(r'reg2/',views.reg1)
]
```

