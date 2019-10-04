## 关联管理对象的方法、分组聚合、F和Q查询、事务



#### 关联管理对象的方法：

- **概念**：通过之前的学习我们知道，在涉及到表与表之间关联的时候就要用到关系管理对象，但是需要注意的是，在一对一关系中，我们并没有用到关系管理对象，而是直接通过获得其中一张表中的记录，然后.另一张表名(小写),即可拿到与之对应的另一张表的对应记录值；所以关联管理对象存在于：

  ```python
  1、外键关系的反向查询，因为正向直接获得的关联管理对象是一对一的，只有反向才是一对多
  2、多对多关联关系
  \# 换句话说，关联管理对象就是针对一对多关系的一个桥梁，也可以说成跨表的一个桥梁，多对多也可以想成一对多，因为多对多实则是一对多的集合体
  ```

- **对应的表结构**

  ```python
  from django.db import models
  
  class Book(models.Model):
      title = models.CharField(max_length=32)
      pub = models.ForeignKey('Publisher',related_name='books',on_delete=models.CASCADE,null=True) # on_delete 表示连接的外键被删除后，此记录怎么办，CASCADE表示一起删除
      
      def __str__(self):
          return self.title
  
  class Publisher(models.Model):
      name = models.CharField(max_length=32,verbose_name='名称')
      
      def __str__(self):
          return  self.name
      
  class Author(models.Model):
      name = models.CharField(max_length=32)
      # 如果是通过作者查书的次数多，我们就把关联管理对象放Author表中（正向方便），反之
      books = models.ManyToManyField('Book')
      
      def __str__(self):
          return self.name
      
  class AuthorDetail(models.Model):
      addr = models.CharField(max_length=64,null=True)
      author = models.OneToOneField('Author')
      
      def __str__(self):
          return self.addr  
  ```
  

  
- **方法：** 获得关系管理对象后的方法

  ```python
  import os
  os.environ.setdefault("DJANGO_SETTINGS_MODULE", "mysite.settings")
  import django
  
  django.setup()
  from app import models
  
  \# ---------------------多对多关联管理对象的方法------------------
   # all()  获取所关联的所有对象
  author_obj = models.Author.objects.get(pk = 1)
  print(author_obj.books.all())  # 关联管理对象
  
   # set()  重置多对多的关系,自带save(): set([id, id…])、set([对象, 对象])或则set(Queryset[])
  author_obj = models.Author.objects.get(pk = 1) # 获得一个作者
  author_obj.books.set([1,3])  # 要设置此作者对应的书，就要重新设置，凡是和他有关的又要重写录入，当配合前端的select的多选框时，前端要使用set(request.POST.getlist('class_ids'))
  author_obj.books.set(models.Book.objects.filter(pk__in=[1,2]))
   # 注解：set会根据你给的对象，去访问它的主键id（需要关联的属性），将其添加到数据表中
  
   # add() 添加多对多的关系,自带save(): add(1,3)、set(*[])
  author_obj = models.Author.objects.get(pk = 1) # 获得一个作者
  author_obj.books.add(4,3) # 给这个关联对象添加记录，即对应的多的一方添加记录，此添加4，5号书
  author_obj.books.add(4,3) # 重复无效果
  author_obj.books.add(*models.Book.objects.filter(pk__in=[1,4]))
  
   # remove() 删除多对多关系，自带save(): remove(1,3)、remove(*[])
  author_obj.books.remove(5,3) # 删除该作者对应的写的4、5号书
  author_obj.books.remove(*models.Book.objects.filter(pk__in=[1,4]))
  
   # clear()  清除所有的多对多关系
  author_obj = models.Author.objects.get(pk = 1)
  author_obj.books.clear()
  
   #  create() 根据关系管理对象创建该管理对象对应的 一条数据，可create(**dic)
   # 给一个作者添加一本书
  author_obj = models.Author.objects.get(pk = 1)
  obj = author_obj.books.create(title='跟Mjj学前端',pub_id=1) # 注意这里创建的是作者的新书，所以需要书的各个属性值包括pub_id
  print(obj)
  
   # 给一本书添加一个作者
  obj = models.Book.objects.get(pk = 1)
  new_obj = obj.author_set.create(name = '萧帮主') # 注意这里没有配置related_name,否则要替换author
  print(new_obj) # 萧帮主
  
  
  \#---------------------------一对多关联管理对象的方法---------------------
  	#正向获得的是一对一的关系
  
   # 反向：
  pub_obj = models.Publisher.objects.get(pk=1)  # 获得id为1的出版社
   # all() 获取所有对象的QuerySet
  print(pub_obj.books.all()) # 设置了related_name = 'books',没有设置则用book_set
  
   # set() 重置一对多记录集合,注意：不能用id:set([1,2])，只能设置对象set(QuerySet[对象,对象])
  pub_obj.books.set(models.Book.objects.filter(pk__in=[4,5])) # 将4，5号书的出版社设置为1号出版社
  
   # add() 添加一对多记录，注意：不能用id: add(1,2),只能设置对象：set(*QuerySet[对象,对象])
  pub_obj.books.add(*models.Book.objects.filter(pk__in=[1,5])) #将1，5 的出版社设置为1号出版社
  
   # remove() 删除，原本关联管理器是没有这个方法的，因为如果删除了记录的值，而原本数据库中的关联字段有没有设置可为空，就出错了
   # 所以要使用此方法，需要设置关联管理对象 null= True;再迁移数据库，就可以使用remove了
  pub_obj.books.remove(*models.Book.objects.filter(pk__in=[1,5,4]))
  
   # clear() 和remove的要求是一样的，都要关联管理器字段可为空
   # pub_obj.books.clear()
  
   # create() 创建关联管理器对应的表的一条记录
  pub_obj.books.create(title='用Python养猪')  # 因为此pub_obj，自带了自己出版社的id，所以直接就给他了
  ```

  

#### 聚合和分组

- **聚合**：对某个字段进行整合得出一个结论，和SQL中的聚合函数一样的思想。

- **注意**：`aggregate()`是`QuerySet` 的一个终止子句，意思是说，它返回一个包含一些键值对的字典；返回一个字典，键的名称是聚合值的标识符，值是计算出来的聚合值。键的名称是按照字段和聚合函数的名称自动生成出来的。

  用到的**聚合函数**：

  ```python
  from django.db.models import Avg, Sum, Max, Min, Count
  ```

- 具体实例

  ```python
   对应的models.py中
  class Book(models.Model):
      title = models.CharField(max_length=32)
      price = models.DecimalField(max_digits=6, decimal_places=2)
      pub = models.ForeignKey('Publisher',related_name='books',on_delete=models.CASCADE,null=True) # on_delete 表示连接的外键被删除后，此记录怎么办，CASCADE表示一起删除
      
      def __str__(self):
          return self.title
  
      -------------------------测试文件--------------------
  import os
  os.environ.setdefault("DJANGO_SETTINGS_MODULE", "mysite.settings")
  import django
  
  django.setup()
  from app import models
  from django.db.models import Max, Min, Avg, Sum, Count
  
  # 聚合
  ret = models.Book.objects.all().aggregate(Max('price')) # aggregate 聚合
  print(ret) #{'price__max': Decimal('199.00')}
  
  ret = models.Book.objects.all().aggregate(avg = Avg('price')) # avg = 可重命名（相当于关键字传参，但是要注意位置参数必须在前）
  print(ret)  # {'avg': 100.0}
  
  # 多组聚合
  ret = models.Book.objects.all().aggregate(Avg('price'),max = Max('price')) # Avg()是位置参数，所以必须再前面
  print(ret) # {'max': Decimal('199.00'), 'price__avg': 100.0}
  
  # 聚合前添加条件
  ret = models.Book.objects.filter(pk__gt=3).aggregate(Avg('price'),max = Max('price')) # 大于3的书籍的平均和最大值
  # 注意aggregate是一个终止子句，不能再链式编程了，后面不能再.了
  ```

  

- **分组**：

  - 我们在这里先复习一下SQL语句的分组。假设现在有一张公司职员表：（单表）

    ![1561455565724](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1561455565724.png)

    我们使用原生SQL语句，按照部门分组求平均工资：求每个部门的平均工资

    ```sql
    select dept,AVG(salary) from employee group by dept;
    
    # ORM查询的分组:
    from django.db.models import Avg
    Employee.objects.values("dept").annotate(avg=Avg("salary").values("dept", "avg")
    ```

  - 连表查询的分组：

    ![1561455661045](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1561455661045.png)

    我们使用SQL和ORM查询：求每个部门的平均工资

    ```python
    select dept.name,AVG(salary) from employee inner join dept on (employee.dept_id=dept.id) group by dept_id;
    
    # ORM查询
    from django.db.models import Avg
    models.Dept.objects.annotate(avg=Avg("employee__salary")).values("name", "avg")
    ```

  - 更多实例：

    ```python
     # 表结构
    from django.db import models
    
    class Book(models.Model):
        title = models.CharField(max_length=32)
        price = models.DecimalField(max_digits=6, decimal_places=2)
        pub = models.ForeignKey('Publisher',related_name='books',on_delete=models.CASCADE,null=True) # on_delete 表示连接的外键被删除后，此记录怎么办，CASCADE表示一起删除
        
        def __str__(self):
            return self.title
    
    class Publisher(models.Model):
        name = models.CharField(max_length=32,verbose_name='名称')
        
        def __str__(self):
            return  self.name
        
    class Author(models.Model):
        name = models.CharField(max_length=32)
        # 如果是通过作者查书的次数多，我们就把关联管理对象放Author表中（正向方便），反之
        books = models.ManyToManyField('Book',related_name='author')
        
        def __str__(self):
            return self.name
              
    
    ---------------------测试文件------------------------------
    import os
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "mysite.settings")
    import django
    django.setup()
    from app import models
    from django.db.models import Max, Min, Avg, Sum, Count
    
    
     # 分组
     # 注意：无论分组还是聚合都是针对查询表的字段进行的，如果要跨表，正向需要关联管理器_id(或直接管理器),反向如果没有配置related_query_name,就看
     # related_name ,如果还是没有就用类名小写（ps:凡是用到''都是字段查询，即要用related_query_name,优先)
     #1、统计一本书的作者个数
    ret = models.Book.objects.annotate(Count('author'))  #annotate 注释
    print(ret) #<QuerySet [<Book: 挑花侠大战菊花怪……] 并没有什么变化？
    print(ret.values('author__count'))  # <QuerySet [{'author__count': 3}, {'author__count': 2}, ……] 封装再这里面，当然可以重命名 count=Count('')
    
    ret = models.Book.objects.annotate(count=Count('author')).values() # 这样就可以查看里面的所有值了，其实annotate表示注释的意思，他是进行了封装再我们的每个对象中
    for obj in ret:
        print(obj)  # 分组的结果当成注解添加到对象中，可通过values来获取
        
     # 2、统计出每个出版社的最便宜的书的价格
     # 方式一：  反向：使用表名__字段 跨表（类似于字段反向查询的表名_set)
    ret = models.Publisher.objects.annotate(min=Min('books__price')).values('min')   # 这里显然出版社没有价格，所以跨表（此时没有条件，所以以每个出版社为一组），此时book中有related_name,所以不用
     # book（表名），同时注意这里也是字段查询，老大还是related_query_name(没设置)
    print(ret)  # value('min')表示取得min字段，取值
    
     # 方式二：正向：使用value('')来规定分组条件（按照谁分组），当然可以value('pub_name'),但是名字可能重复，就会合并；
    ret = models.Book.objects.values('pub_id').annotate(min=Min('price')).values('min')
    print(1,ret)
     # 注意：value(),annotate默认它内部字段为分组条件，所以如果后面还有values，只能取前面出现过得字段，如min、pub_id……，切记不能value(),其意味着所有字段都参与分组条件，也就是说如果我取别没有出现过的字段，默认回去取分组的第一条的数据，和我们想要的对不上
     
     #3、统计不止一个作者得图书
    ret = models.Book.objects.annotate(count=Count('author__name')).filter(count__gt=1) # 这里Book中没有管理对象，所以要反向，也就通过反向跨表得到一个一对多的关系管理对象表，因为设置了related_name，所以用它；如果没有设置related_name,也没有设置related_query_name(优先级最高），就用表名（小写）
    print(ret)
    
     #4、根据一本图书作者数量得多少对查询集QuerySet进行排序
    ret = models.Book.objects.annotate(count=Count('author')).order_by('-count')
    print(ret)
    
     #5、查询各个作者出得书得总价格
    ret = models.Author.objects.annotate(sum=Sum('books__price')).values() # 这里有关联管理对象所以是正向查找，所以 books__price
    print(ret)
    ```

    

#### F查询和Q查询

​		在上面所有的例子中，我们构造的过滤器都只是将字段值与某个常量做比较。如果我们要对两个字段的值做比较，那该怎么做呢？Django 提供 F() 来做这样的比较。F() 的实例可以在查询中引用字段，来比较同一个 model 实例中两个不同字段的值。

- **F查询：**也就是在同一张表中得不同字段进行比较,以及对某字段批量更新

  ```python
  import os
  os.environ.setdefault("DJANGO_SETTINGS_MODULE", "mysite.settings")
  import django
  
  django.setup()
  from app import models
  
  # F查询
  from django.db.models import F
  #比较两个字段的值
  ret = models.Book.objects.filter(sale__gt=F('kucun')) # 比较sale字段 大于 kucun 的对象集合；也就是说F()会拿出想要比较的字段然后于前者进行比较
  print(ret) # <QuerySet [<Book: 跟Mjj学前端>, <Book: 用Python养猪>]>
  
  #补充：
  #我们之前修改一个对象，用的方法(最老的办法: 此办法低效，因为他会更新所有的这条记录的字段）
  obj = models.Book.objects.get(pk=1)
  obj.sale = 121
  obj.save()
  
  # 批量更新 QuerySet[].update(), 自带save()，此方法较为高效，只更新修改字段
  models.Book.objects.filter(pk__in=[1,3]).update(sale=100)
  
  #Django 支持 F() 对象之间以及 F() 对象和常数之间的加减乘除和取模的操作。
  models.Book.objects.filter(pk__in=[2,3]).update(sale=F('sale')*2+10)  # 支持乘除等更新：配合update
  ```



- **Q 查询**：`filter()` 等方法中的关键字参数查询都是一起进行“AND” 的。 如果你需要执行更复杂的查询（例如`OR`语句），你可以使用`Q对象`。

  - 实例1：查询作者名是小仙女或小魔女的

    ```python
    models.Book.objects.filter(Q(authors__name="小仙女")|Q(authors__name="小魔女"))
    ```

  - 能配合的功能符号：

    ```
    | :或
    & :与，等于','号
    ~ :非
    ```

  - 详细实例讲解：

    ```python
    import os
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "mysite.settings")
    import django
    django.setup()
    from app import models
    from django.db.models import Q
    ret = models.Book.objects.filter(Q(pk__gt=3),Q(pk__lt=2)) # 但是目前这两个Q对象还是用,号连接，所以还是and条件
    print(ret) #<QuerySet []>，因为现在还是and条件
    
    ret = models.Book.objects.filter(Q(pk__lt=2)|Q(pk__gt=5)) # 小于2 大于5
    print(ret) #<QuerySet [<Book: 挑花侠大战菊花怪>, <Book: 跟alex学Python>, <Book: 跟Mjj学前端>, <Book: 用Python养猪>]>
    
    ret = models.Book.objects.filter(Q(Q(pk__lt=2)|Q(pk__gt=5))&Q(price__gt=150)) # 多个对象的重叠且多and连接: & 等于,
    print(ret)
    
    ret = models.Book.objects.filter(~Q(pk__gt=4)|Q(pk__lt=2)) # 小于4 或则小于2 ，可知Q查询遵循贪婪匹配
    print(ret) #<QuerySet [<Book: 挑花侠大战菊花怪>, <Book: 挑花侠大战菊花怪续集>, <Book: 挑花侠大战菊花怪终章>, <Book: Mjj自传>]>
    ```

    

#### 事务：

- 事务里面一定要保持原子性操作，事务中的所有执行，只要有出错，就要回滚，防止错误记录被存储

- **具体实现**：

  ```python
  import os
  
  if __name__ == '__main__':
      os.environ.setdefault("DJANGO_SETTINGS_MODULE", "BMS.settings")
      import django
      django.setup()
  
      import datetime
      from app01 import models
  
      try:
          from django.db import transaction
          with transaction.atomic():
              new_publisher = models.Publisher.objects.create(name="火星出版社")
              models.Book.objects.create(title="橘子物语", publish_date=datetime.date.today(), publisher_id=10)  # 指定一个不存在的出版社id
      except Exception as e:
          print(str(e))
          
   # 一旦出错，将数据库中的数据删除，并打印报错信息 ，只有正常结束才能保存在数据库中。      
  ```

  

#### Django ORM 执行原生SQL

```python
extra 方法介绍
在QuerySet的基础上继续执行子语句
extra(self, select=None, where=None, params=None, tables=None, order_by=None, select_params=None)

 #select和select_params是一组，where和params是一组，tables用来设置from哪个表
Entry.objects.extra(select={'new_id': "select col from sometable where othercol > %s"}, select_params=(1,))
Entry.objects.extra(where=['headline=%s'], params=['Lennon'])
Entry.objects.extra(where=["foo='a' OR bar = 'a'", "baz = 'a'"])
Entry.objects.extra(select={'new_id': "select id from tb where id > %s"}, select_params=(1,), order_by=['-nid'])

举个例子：
models.UserInfo.objects.extra(
                    select={'newid':'select count(1) from app01_usertype where id>%s'},
                    select_params=[1,],
                    where = ['age>%s'],
                    params=[18,],
                    order_by=['-age'],
                    tables=['app01_usertype']
                )
                """
                select 
                    app01_userinfo.id,
                    (select count(1) from app01_usertype where id>1) as newid
                from app01_userinfo,app01_usertype
                where 
                    app01_userinfo.age > 18
                order by 
                    app01_userinfo.age desc
                """

----------------------------------------------------------------
执行原生SQL，更高灵活度的方式执行原生SQL语句
from django.db import connection, connections
cursor = connection.cursor()  # cursor = connections['default'].cursor()
cursor.execute("""SELECT * from auth_user where id = %s""", [1])
row = cursor.fetchone()
```



#### Django终端打印SQL语句

- 在Django项目的settings.py文件中，在最后复制粘贴如下代码：

  ```python
  LOGGING = {
      'version': 1,
      'disable_existing_loggers': False,
      'handlers': {
          'console':{
              'level':'DEBUG',
              'class':'logging.StreamHandler',
          },
      },
      'loggers': {
          'django.db.backends': {
              'handlers': ['console'],
              'propagate': True,
              'level':'DEBUG',
          },
      }
  }
  ```

  

#### 考试题精选

- **大题**：

  ![1561870042614](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1561870042614.png)

- **对应的答案**：

  ```python
  import os
  os.environ.setdefault("DJANGO_SETTINGS_MODULE", "mysite.settings")
  import django
  django.setup()
  from app import models
  
   #1、查询三年级三班所有同学的名字  (一对多实例 ）
  ret = models.Classes.objects.get(c_name = '三年级三班').students.all().values('s_name')
  print(ret)
  
  ret = models.Student.objects.filter(my_class__c_name='三年级三班').values('s_name')
  print(ret)
  
  ret = models.Student.objects.filter(my_class__c_name='三年级三班').values('s_name')
  print(ret)
  
  ret = models.Classes.objects.filter(c_name='三年级三班').values_list('students__s_name')
  print(ret)
  
   #2、查询分数小于等于60分的同学的成绩和姓名
  ret = models.Student.objects.filter(score__lte = 60).values('s_name')
  print(ret)
  
   #3、查询每个班级的名称和学生人数
  from django.db.models import Count
  ret = models.Classes.objects.annotate(count = Count('students__s_name')).values('c_name','count')
  print(ret)
  
   #4、查询年纪最大的老师的姓名和年纪
  from django.db.models import Max,Avg
  ret= models.Teacher.objects.filter(age = models.Teacher.objects.all().aggregate(max=Max('age')).get('max')).values('t_name','age')
  print(ret)
   #5、分别查询出男女老师的个数
  
  ret = models.Teacher.objects.values('sex').annotate(count=Count('t_name')).values('sex','count')
  print(ret)
  
   #6、查询名字中有'小'的学生的姓名和成绩
  ret = models.Student.objects.filter(s_name__contains='小').values('s_name','score')
  print(ret)
  
   #7、查询二年级二班的平均成绩
  from django.db.models import Avg
  ret = models.Student.objects.filter(my_class = models.Classes.objects.get(c_name = '二年级二班')).aggregate(avg = Avg('score'))
  print(ret)
  
   #8、新增一个名为'四年级四班'的班级
  obj = models.Classes.objects.create(c_name='四年级四班')
  obj.teachers.set([1,2])
  
   #9、给二年级二班新增一个名为'小红'的学生
  obj = models.Student.objects.create(s_name='小红',score=45,my_class=models.Classes.objects.get(c_name='二年级二班'))
   
   #10、给二年级二班，新增一个名为'苑局'的30岁男老师
  t_name=models.Teacher.objects.create(t_name='苑局',sex=1,age=30)
models.Classes.objects.get(c_name='二年级二班').teachers.add(7)
  
   # 11、小红转班了， 转到三年级三班
  models.Student.objects.filter(s_name='小红').update(my_class=3)
  
   # 12、给所有学生都加2分
  from django.db.models import F
  models.Student.objects.all().update(score=F('score')+2)
  ```
  
  