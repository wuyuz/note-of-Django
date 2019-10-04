## Django 之content-type组件



#### Content-type

- Django内置的一个组件，帮助开发者做连表操作。（混搭）

  

##### 我们先来分析以下表结构的涉及，怎么实现多张表的合并？

- **方式一：**通过追加字段名（有缺陷，追加时需要重写表结构）

  ![1564282202291](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1564282202291.png)

- **方式二：**通过链接表名和表中id进行配对（可无限扩展，且不用重写表结构，适用于一张表和多张表关联）

  ![1564282436308](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1564282436308.png)

- **方式三：**在方式二的基础上更进，如果修改了表名方式二会全部修改数据，通过table表可以改善

  ![1564283402602](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1564283402602.png)



#### 具体应用

- 以我们最初的models的写法来创建表结构

  ```python
  from django.db import models
  class Course(models.Model):
      title = models.CharField(max_length=32)
      
  
  class DegreeCourse(models.Model):
      title = models.CharField(max_length=32)
      
  class PricePolicy(models.Model):
      price = models.IntegerField()
      period = models.IntegerField()
      table_name = models.CharField(verbose_name='关联的表名称')
      object_id = models.CharField(verbose_name='关联的表中的id')
  ```

- 使用content-type表来做第三种方式的table表

  ```python
  from django.db import models
  from django.contrib.contenttypes.fields import GenericForeignKey, GenericRelation
  from django.contrib.contenttypes.models import ContentType
  
  class Course(models.Model):
      title = models.CharField(max_length=32)
      
  class DegreeCourse(models.Model):
      title = models.CharField(max_length=32)
      
  class PricePolicy(models.Model):
      price = models.IntegerField()
      period = models.IntegerField()
      table_name = models.ForeignKey(ContentType,verbose_name='关联的表名称')  # 使用Django为我们提供的content-type表，作为table表
      object_id = models.IntegerField(verbose_name='关联的表中的id')
  ```

  - **小提示：**

    ```
    	在settings中的APP注册列表中，我们可以看到content-type应用表，该表用于记录系统自带的数据表和我们自己子meodels中创建的表机构，我们也可以直接使用
    ```

    ![1564283934261](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1564283934261.png)



##### 设计完表结构后，我们如果想插入一条数据：为学位课"python 全栈"添加一个价格策略：一个月 9.9该如何操作？

- 原始方法

  ```python
  obj = DegreeCourse.object.filter(title='Python 全栈').first()
  cobj = ContentType.objest.filter(model='course').first()
  PricePolicy.objects.create(price='9.9',period='30',content_type_id=cobj.id,object_id=obj.id)
  ```

- 使用ContentType组件

  ```python
  from django.db import models
  from django.contrib.contenttypes.fields import GenericForeignKey, GenericRelation
  from django.contrib.contenttypes.models import ContentType
  
  class Course(models.Model):
      title = models.CharField(max_length=32)
      
  class DegreeCourse(models.Model):
      title = models.CharField(max_length=32)
      
  class PricePolicy(models.Model):
      price = models.IntegerField()
      period = models.IntegerField()
      content_type = models.ForeignKey(ContentType,verbose_name='关联的表名称')  # 使用Django为我们提供的content-type表，作为table表
      object_id = models.IntegerField(verbose_name='关联的表中的id')
      
      # 帮助你快速实现content_type操作,通过传入的对象，本对象的id，以及关联对象的id，并赋值给'content_type','object_id'两个字段
      content_object = GenericForeignKey('content_type','object_id')
      
  ------------------------------------------------------------------------    
   # views.py文件中
  from django.shortcuts import render,HttpResponse
  from api import models
  
  def test(request):
      # 1.为学位课"python 全栈"添加一个价格策略：一个月 9.9
      obj = models.DegreeCourse.objects.filter(title='Python 全栈').first()
      print(obj)
      # 给content_type传值，会自动去查找传入对象的关联表
      models.PricePolicy.objects.create(price=9.9,period=30,content_object=obj)
  
      obj = models.DegreeCourse.objects.filter(title='Python 全栈').first()
      models.PricePolicy.objects.create(price=19.9, period=60, content_object=obj)
      
      obj = models.DegreeCourse.objects.filter(title='Python 全栈').first()
      models.PricePolicy.objects.create(price=29.9, period=90, content_object=obj)
      
      return HttpResponse('OK')
  
  -------------------------------------------------------------------------
   # url中
  urlpatterns = [
      url(r'^test/', views.test),
  ]
  ```

  

- 使用ContentType进行迅速反向查找

  ```python
  from django.contrib.contenttypes.fields import GenericForeignKey, GenericRelation
  from django.contrib.contenttypes.models import ContentType
  # Create your models here.
  
  class Course(models.Model):
      title = models.CharField(max_length=32)
       # 仅用于反向查找,不生成数据库
      price_policy_list = GenericRelation("PricePolicy")
      
  class DegreeCourse(models.Model):
      title = models.CharField(max_length=32)
      
  class PricePolicy(models.Model):
      price = models.IntegerField()
      period = models.IntegerField()
      content_type = models.ForeignKey(ContentType,verbose_name='关联的表名称')  # 使用Django为我们提供的content-type表，作为table表
      object_id = models.IntegerField(verbose_name='关联的表中的id')
      
      # 帮助你快速实现content_type操作,通过传入的对象，本对象的id，以及关联对象的id，并赋值给'content_type','object_id'两个字段
      content_object = GenericForeignKey('content_type','object_id')
      
   --------------------------------------------------
    # views.py文件
  def test(request):
      # 根据课程ID获取课程和课程的所有价格策略
      course = models.Course.objects.filter(id=1).first()
      price_policy = course.price_policy_list.all()
      print(price_policy.values('price'))
      
      return HttpResponse('OK')
  
  ```
  
