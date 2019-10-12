## Django模型层之关联管理器



#### class RelatedManager

- **概念**："关联管理器"是在一对多或则多对多的关联上下文使用的管理器，也就是一种映射字段，映射到关联表的一条或多条记录上。

- 先看一个**实列**：

  ```python
  #models.py文件中
  from django.db import models
  
  class Class(models.Model):
      name = models.CharField(max_length=12)
      
  class Student(models.Model):
      name = models.CharField(max_length=12)
      classes = models.ForeignKey('CLass')   #  使用字符串形式讲采用反射获取，不加字符串则要放在后面，不然找不到
      studnetdeil = models.OneToOneField('StudentDeil')
  
  class StudentDeil(models.Model):
      addr = models.CharField(max_length=32)
      tel = models.IntegerField(max_length=12)
  
      
  ----------------------------------------------------------------------------------
  #在views.py文件中
  def show(request):
      
      # 一对一，外键关系，studnetdeil是一个对象，但不是关联管理对象，但是一条记录.studnetdeil
      # 却可以拿到一个对应其他表的一条记录，其他关系（一对多……）却不行，他们拿到的是有一个管理对象，拿不到其中的数据，因为本来就不是一条记录
      stu = models.Student.objects.filter(id = 1).first()
      print(stu)  # Student object
      print(stu.studnetdeil.id)  # 1
      print(stu.studnetdeil, type(stu.studnetdeil))  # StudentDeil object  <class 'app.models.StudentDeil'>
      
      --------------------------------------------------------------
      #一对多的情况
      print(stu.classes,type(stu.classes))  # Class object  <class 'app.models.Class'>
      print(stu.classes.id)  # 2
      
      print(models.Student.classes)  # <django.db.models.fields.related_descriptors.ForwardManyToOneDescriptor object at 0x000001F7897EF860>
      print(models.Student.studnetdeil) # <django.db.models.fields.related_descriptors.ForwardOneToOneDescriptor object at 0x000001F7897EF940>
      
      cls = models.Class.objects.filter(id = 1).first()
      print(cls.student_set.all())  # 注意all只用于'多' 的一方，这里就是用的反向查询，多的一方查起
      
      print(stu.classes.name) # 注意在一对多的情况，只能反向时使用all，在多对多的情况可以使用all()
  
      return HttpResponse('ok')
  ```
  
  

- "关联管理器"我们主要分析存在的以下两种对象：一对多、多对多、（一对一不考虑）

  - ForeignKey关系：

    ```python
    from django.db import models
     
    class Reporter(models.Model):  # 报道者
        # ...
        pass
     
    class Article(models.Model):  # 文章
        reporter = models.ForeignKey(Reporter)
    ```

    在上面的例子中，管理器 `reporter.article_set` 拥有下面的方法。反向查询出来"一"，让其可以使用以下的所有功能

  - ManyToManyField关系：

    ```python
    class Topping(models.Model):
        # ...
        pass
     
    class Pizza(models.Model):
        toppings = models.ManyToManyField(Topping)
    ```

    这个例子中，topping.pizza_set 和pizza.toppings都拥有下面的方法。在多对多情况下，反向和正向查询都可以使用以下方法，所以不难总结出，以下方法针对的是一些对象集合。

- **各种方法**：

  - add(obj1[, obj2, ...]) : 把指定的模型对象添加到关联对象集中。

    ```python
    例如：
    
    >>> b = Blog.objects.get(id=1)
    >>> e = Entry.objects.get(id=234)
    >>> b.entry_set.add(e) # Associates Entry e with Blog b.
    在上面的例子中，对于ForeignKey关系，e.save()由关联管理器调用，执行更新操作。然而，在多对多关系中使用add()并不会调用任何 save()方法，而是由QuerySet.bulk_create()创建关系。
    
    延伸：
    # 1 *[]的使用
    >>> book_obj = Book.objects.get(id=1)
    >>> author_list = Author.objects.filter(id__gt=2)
    >>> book_obj.authors.add(*author_list)
    
    
    # 2 直接绑定主键
    book_obj.authors.add(*[1,3])  # 将id=1和id=3的作者对象添加到这本书的作者集合中
                                  # 应用: 添加或者编辑时,提交作者信息时可以用到.  
    ```

  - create(\**kwargs)  :创建一个新的对象，保存对象，并将它添加到关联对象集之中。返回新创建的对象：

    ```python
    >>> b = Blog.objects.get(id=1)
    >>> e = b.entry_set.create(
    ...     headline='Hello',
    ...     body_text='Hi',
    ...     pub_date=datetime.date(2005, 1, 1)
    ... )
    
    # No need to call e.save() at this point -- it's already been saved.
    这完全等价于（不过更加简洁于）：
    
    >>> b = Blog.objects.get(id=1)
    >>> e = Entry(
    ...     blog=b,
    ...     headline='Hello',
    ...     body_text='Hi',
    ...     pub_date=datetime.date(2005, 1, 1)
    ... )
    >>> e.save(force_insert=True)
    要注意我们并不需要指定模型中用于定义关系的关键词参数。在上面的例子中，我们并没有传入blog参数给create()。Django会明白新的 Entry对象blog 应该添加到b中。
    ```

  - remove(obj1[, obj2, ...]) : 从关联对象集中移除执行的模型对象

    ```python
    >>> b = Blog.objects.get(id=1)
    >>> e = Entry.objects.get(id=234)
    >>> b.entry_set.remove(e) # Disassociates Entry e from Blog b.
    对于ForeignKey对象，这个方法仅在null=True时存在。
    ```

  - clear() : 从关联对象集中移除一切对象。

    ```python
    >>> b = Blog.objects.get(id=1)
    >>> b.entry_set.clear()
    注意这样不会删除对象 —— 只会删除他们之间的关联。
    
    就像 remove() 方法一样，clear()只能在 null=True的ForeignKey上被调用。
    ```

  - set() : 先清空，在设置，编辑书籍时即可用到(对一条关联记录进行清空，再填充)

    ![1560927504033](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1560927504033.png)

    ```
    注意：对于所有类型的关联字段，add()、create()、remove()和clear(),set()都会马上更新数据库。换句话说，在关联的任何一端，都不需要再调用save()方法。
    ```

  - **直接赋值**

    ```python
    通过赋值一个新的可迭代的对象，关联对象集可以被整体替换掉。
    >>> new_list = [obj1, obj2, obj3]
    >>> e.related_set = new_list
    
    #如果外键关系满足null=True，关联管理器会在添加new_list中的内容之前，首先调用clear()方法来解除关联集中一切已存在对象的关联。否则， new_list中的对象会在已存在的关联的基础上被添加。　　
    ```

    