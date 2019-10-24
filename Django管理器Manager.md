## Django管理器Manager

### 管理器

什么叫管理器(管理器是Django内置的Model.Manager类)

那么管理器是做什么的呢？

> A `Manager` is the interface through which database query operations are provided to Django models. At least one `Manager` exists for every model in a Django application.
>
> 管理器是Django提供的查询数据库操作的接口，Django 的app 的每个model都最少有一个管理器

### 默认管理器对象objects

```
objects = models.Manager()
```

objects 是默认的管理器对象

> By default, Django adds a `Manager` with the name `objects` to every Django model class
>
> 默认情况下，Django会为model的每个类class定义一个管理器对象，名字默认为objects
>
> 注意：是每个models.py 的每个class 都有一个默认的objects管理器对象

我们之前通过python manage.py shell 可以调用objects管理器对象

```python
假设我们的models.py有以下两个class，分别为BookInfo 与 HeroInfo,那么按照上面的说法，这两个类都有一个叫做objects的默认管理器

class BookInfo(models.Model):
    btitle = models.CharField(max_length=20)
    bpub_date = models.DateTimeField()
    def __str__(self):
        return self.btitle

class HeroInfo(models.Model):
    hname = models.CharField(max_length=20)
    hgender = models.BooleanField(default=True)
    hbook = models.ForeignKey(BookInfo)
    def __str__(self):
        return self.hname

python manage.py shell
>>> from booktest.models import * 

>>> BookInfo.objects.all()   #使用BookInfo类的默认管理器
<QuerySet [<BookInfo: abc>]>

>>> HeroInfo.objects.all()   #使用HeroInfo类的默认管理器
<QuerySet [<HeroInfo: h1>, <HeroInfo: h2>]>
```

### 重命名默认管理器对象

> However, if you want to use `objects` as a field name, or if you want to use a name other than `objects` for the `Manager`, you can rename it on a per-model basis. To rename the `Manager` for a given class, define a class attribute of type `models.Manager()` on that model. For example:
>
> 如果你不想使用objects这个管理器对象的名字，你可以重命名默认管理器对象的名字

官网实例：

```python
from django.db import models

class Person(models.Model):
    #...
    people = models.Manager()
```

Using this example model, `Person.objects` will generate an `AttributeError` exception, but `Person.people.all()` will provide a list of all `Person` objects.

重命名默认管理器对象的例子：

```python
修改models.py
from django.db import models

class BookInfo(models.Model):
    btitle = models.CharField(max_length=20)
    bpub_date = models.DateTimeField()
    New_Manager = models.Manager()              #BookInfo类定义了新的管理器对象New_Manager
    
    def __str__(self):
        return self.btitle

class HeroInfo(models.Model):                  #HeroInfo类使用默认管理器对象objects
    hname = models.CharField(max_length=20)
    hgender = models.BooleanField(default=True)
    hbook = models.ForeignKey(BookInfo)
    def __str__(self):
        return self.hname




$ python manage.py shell

>>> from booktest.models import * 

>>> BookInfo.objects.all()    #BookInfo类没有objects管理器对象了
Traceback (most recent call last):
  File "<console>", line 1, in <module>
AttributeError: type object 'BookInfo' has no attribute 'objects'


>>> BookInfo.New_Manager.all() #BooKInfo类有New_Manager管理器对象
<QuerySet [<BookInfo: abc>]>


>>> HeroInfo.New_Manager.all()  
Traceback (most recent call last):
  File "<console>", line 1, in <module>
AttributeError: type object 'HeroInfo' has no attribute 'New_Manager'

>>> HeroInfo.objects.all()
<QuerySet [<HeroInfo: h1>, <HeroInfo: h2>]>
>>>
```

### 自定义管理器对象

> You can use a custom `Manager` in a particular model by extending the base `Manager` class and instantiating your custom `Manager` in your model.

通过扩展基本Manager类 并在模型中实例化自定义Manager，可以在特定模型中使用自定义Manager。

> There are two reasons you might want to customize a Manager: to add extra Manager methods, and/or to modify the initial QuerySet the Manager returns.

您可能希望自定义Manager有两个原因：添加额外的Manager方法，和/或修改Manager返回的初始QuerySet。

#### Adding extra manager methods

默认管理器返回的是queryset

```python
>>> BookInfo.objects.all()   #使用BookInfo类的默认管理器,返回queryset
<QuerySet [<BookInfo: abc>]>
```

A custom `Manager` method can return anything you want. It doesn’t have to return a `QuerySet`.

但是自定义的管理器对象可以返回任何你想要的类

例如下面的例子：

首先修改models.py

```python
from django.db import models

#定义了一个名字是New_Manager的管理器对象，继承自models.Manager,并重写了models.Manager的get_queryset方法

class New_Manager(models.Manager):
    def get_queryset(self):
        return super(New_Manager,self).get_queryset().filter(IsDelete=False)

#默认情况下有如下使用方式BookInfo.objects.filter() 

class BookInfo(models.Model):
    btitle = models.CharField(max_length=20)
    bpub_date = models.DateTimeField()
    IsDelete = models.NullBooleanField()

    New_Manager1 = models.Manager() #重命名objects管理器对象
    New_Manager2 = New_Manager()    #自定义了New_Manager2管理器对象

    def __str__(self):
        return self.btitle
```

结果：

```
>>> from booktest.models import * 
>>> BookInfo.New_Manager1.all()
<QuerySet [<BookInfo: book2>, <BookInfo: book3>, <BookInfo: book4>, <BookInfo: book5>, <BookInfo: book1>]>
>>> BookInfo.New_Manager2.all()
<QuerySet [<BookInfo: book3>, <BookInfo: book4>]>
```