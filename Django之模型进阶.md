## Django之模型进阶



#### 官网知识点补充

**需要使用的数据**

```python
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def __str__(self):
        return self.name

class Author(models.Model):
    name = models.CharField(max_length=200)
    email = models.EmailField()

    def __str__(self):
        return self.name

class Entry(models.Model):
    blog = models.ForeignKey(Blog, on_delete=models.CASCADE)
    headline = models.CharField(max_length=255)
    body_text = models.TextField()
    pub_date = models.DateField()
    mod_date = models.DateField()
    authors = models.ManyToManyField(Author)
    n_comments = models.IntegerField()
    n_pingbacks = models.IntegerField()
    rating = models.IntegerField()

    def __str__(self):
        return self.headline
```

- **创建对象**

  ```python
  >>> from blog.models import Blog
  >>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
  >>> b.save()
  ```

- **针对ForeignKey和ManyToManyField字段的保存：**

  ```python
  ForeignKey添加记录
  #法一：
  >>> from blog.models import Blog, Entry 
  >>> entry = Entry.objects.get(pk=1)  #去拿到要被添加记录的对象
  >>> cheese_blog = Blog.objects.get(name="Cheddar Talk") # 要添加的记录
  >>> entry.blog = cheese_blog  #使用关联管理对象绑定
  >>> entry.save()
  
  #法二：
  >>> from blog.models import Author
  >>> joe = Author.objects.create(name="Joe") #创建一个记录
  >>> entry.authors.add(joe)  # 使用关系管理对象添加
  
  ------------------------------------------------
  ManyToManyFeild添加数据
  >>> john = Author.objects.create(name="John")
  >>> paul = Author.objects.create(name="Paul")
  >>> george = Author.objects.create(name="George")
  >>> ringo = Author.objects.create(name="Ringo")
  >>> entry.authors.add(john, paul, george, ringo)  #使用关系管理对象一次性添加多个记录
  ```

- **跨越关系的查找**：

  ```python
   #要跨越关系，只需使用跨模型的相关字段的字段名称，用双下划线分隔，直到到达所需的字段。
      
   #练习：检索所有Entry与对象Blog，其name 为：'Beatles Blog'
   #正向查询，使用关联管理对象__name
  >>> Entry.objects.filter(blog__name='Beatles Blog') 
  
   #练习：使用检索所有Blog具有至少一个对象Entry ，其headline包含'Lennon'
   #反向查询，使用小写的表名__字段(甚至可以深度查询)
  >>>Blog.objects.filter(entry__headline__contains='Lennon') 
  ```

- ##### 跨越多值关系

  ```python
   #练习：要选择所有包含标题中包含“Lennon”且2008年发布的条目（同一条目满足两个条件）的博客。
  Blog.objects.filter(entry__headline__contains='Lennon', entry__pub_date__year=2008)
  
   #练习：要选择标题 中包含“Lennon”条目的所有博客以及 2008年发布的条目。
  Blog.objects.filter(entry__headline__contains='Lennon').filter(entry__pub_date__year=2008) 
  
  #假设只有一个博客的两个条目都包含“Lennon”和2008年的条目，但是2008年的条目都没有包含“Lennon”。第一个查询不会返回任何博客，但第二个查询将返回该博客。
  
  ----------------------------------------------------
  如上所述filter()，跨越多值关系的查询的行为不是等效实现的exclude()。相反，单个exclude() 调用中的条件不一定是指同一个项目。
  
  #练习:下面的查询将排除包含博客都 与条目“侬”的标题，并在2008年出版的条目
  Blog.objects.exclude(
      entry__headline__contains='Lennon',
      entry__pub_date__year=2008,
  )
  #但是，与使用时的filter()的行为不同 ，这不会限制基于满足这两个条件的条目的博客。为了做到这一点，即选择所有不包含 2008年发布的“Lennon”条目的博客，您需要进行两次查询：
  Blog.objects.exclude(
      entry__in=Entry.objects.filter(
          headline__contains='Lennon',
          pub_date__year=2008,
      ),
  )
  ```

- **F查询用于同一模型中字段的比较**：在到目前为止给出的例子中，我们构造了过滤器，用于比较模型字段的值和常量。但是，如果要将模型字段的值与同一模型上的另一个字段进行比较，该怎么办？

  ```python
   #练习1：要查找包含比pingback更多注释的所有博客条目的列表，我们构造一个F()对象来引用pingback计数，并F()在查询中使用该对象。
  >>> from django.db.models import F
  >>> Entry.objects.filter(n_comments__gt=F('n_pingbacks'))
  
  ------------------------------------------------------------
   #可以使用加减乘除
   #练习2：要查找所有博客条目的评论数量是pingback的两倍以上 ，我们会修改查询
  >>> Entry.objects.filter(n_comments__gt=F('n_pingbacks') * 2) 
  
   #练习3：要查找条目评级小于pingback计数和评论计数总和的所有条目，我们将发出查询 
  >>>Entry.objects.filter(rating__lt=F('n_comments') + F('n_pingbacks'))
  
  ---------------------------------------------------------
   #跨表F查询：F()具有双下划线的对象将引入访问相关对象所需的任何连接，正反都可以
   #练习1：要检索作者姓名与博客名称相同的所有条目，我们可以发出查询
  >>> Entry.objects.filter(authors__name=F('blog__name'))
   #练习2：内容将返回在发布后超过3天内修改的所有条目
  >>> from datetime import timedelta
  >>> Entry.objects.filter(mod_date__gt=F('pub_date') + timedelta(days=3))
  
  ----------------------------------------------------------
   #该F()对象通过支持位运算.bitand()，.bitor()， .bitrightshift()，和.bitleftshift()。例如：
  >>> F('somefield').bitand(16)       
  ```

- **PK查询快捷方式**：Django提供了一个`pk`查找快捷方式，代表“主键”。

  ```python
   #以下三条在id为主键下是等效的
  >>> Blog.objects.get(id__exact=14) # Explicit form
  >>> Blog.objects.get(id=14) # __exact is implied
  >>> Blog.objects.get(pk=14) # pk implies id__exact
  
   #使用pk不仅限于__exact查询 - 可以组合任何查询术语以pk对模型的主键执行查询：
   # Get blogs entries with id 1, 4 and 7
  >>> Blog.objects.filter(pk__in=[1,4,7])
  
  # Get all blog entries with id > 14
  >>> Blog.objects.filter(pk__gt=14)
  
  --------------------------------------------------
  #pk查找也适用于连接。例如，这三个陈述是等价的
  >>> Entry.objects.filter(blog__id__exact=3) # Explicit form
  >>> Entry.objects.filter(blog__id=3)        # __exact is implied
  >>> Entry.objects.filter(blog__pk=3)        # __pk implies __id__exact
  ```

- **使用Q对象进行复杂查找**：关键字参数查询 - 输入[`filter()`](https://docs.djangoproject.com/zh-hans/2.1/ref/models/querysets/#django.db.models.query.QuerySet.filter)等 - 是“AND”编辑在一起的。如果需要执行更复杂的查询（例如，带`OR`语句的查询），则可以使用。[`Qobjects`](https://docs.djangoproject.com/zh-hans/2.1/ref/models/querysets/#django.db.models.Q) 也就是说在filter中的条件只能同时满足才能查询，不能进行满足其中一个即可。

  ```python
  用法1：封装查询对象， Q（）是用于封装关键字参数集合的对象。这些关键字参数在上面的“字段查找”中指定,例如下面的Q，此Q对象封装了一个LIKE查询:
      
  from django.db.models import Q
  Q(question__startswith='What')
  
  -------------------------------------------------------------
  用法2：合并Q对象，Q可以使用&和|运算符组合对象。当在两个Q对象上使用运算符时，它会生成一个新Q对象。例如，此语句生成一个Q表示两个"question__startswith"查询的“OR”的对象：
  
  Q(question__startswith='Who') | Q(question__startswith='What')
  #相当于：WHERE question LIKE 'Who%' OR question LIKE 'What%'
  
  -----------------------------------------------------------
  用法3：Q 可以使用~运算符取消对象，允许组合查找结合常规查询和negated（NOT）查询。
  Q(question__startswith='Who') | ~Q(pub_date__year=2005)
  
  ------------------------------------------------------------
  用法4：每个查找函数，采用关键字参数（例如filter()， exclude()， get()）也可以通过一个或多个 Q对象作为位置（未命名的）参数。如果为Q查找函数提供多个 对象参数，则参数将“AND”编辑在一起。
  
  Poll.objects.get(
      Q(question__startswith='Who'),
      Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6))
  )
  -------------------------------------------------------------
  用法5：Q提供了对象，则它必须位于任何关键字参数的定义之前。
  Poll.objects.get(
      Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6)),
      question__startswith='Who',
  )
  ```

- **selected_related()** ：返回一个跟随”外键关系的`QuerySet`“，在执行查询时选择其他相关对象数据。这是一个性能增强器，它会导致单个更复杂的查询，但意味着以后使用外键关系不需要数据库查询。

  ```python
  以下示例说明了普通查找和select_related()查找之间的区别 。这是标准查找：
  #普通查找
  # Hits the database.
  e = Entry.objects.get(id=5)
  
  # Hits the database again to get the related Blog object.
  b = e.blog
  
  -----------------------------------------------------------
  #select_related查找
  # Hits the database.
  e = Entry.objects.select_related('blog').get(id=5)
  
  # Doesn't hit the database, because e.blog has been prepopulated
  # in the previous query.
  b = e.blog
  
  ---------------------------------------------------------
  #链式传递不考虑顺序：链接filter()和select_related()链接的顺序并不重要。这些查询集是等效的，如下：
  Entry.objects.filter(pub_date__gt=timezone.now()).select_related('blog')
  Entry.objects.select_related('blog').filter(pub_date__gt=timezone.now())
  ```

  - 案例讲解：

    ```python
    from django.db import models
    class City(models.Model):
        # ...
        pass
    
    class Person(models.Model):
        # ...
        hometown = models.ForeignKey(
            City,
            on_delete=models.SET_NULL,
            blank=True,
            null=True,
        )
    
    class Book(models.Model):
        # ...
        author = models.ForeignKey(Person, on_delete=models.CASCADE)
        
    ------------------------------------------------------
    #Book.objects.select_related('author__hometown').get(id=4) 将缓存相关Person 和相关的City
    # Hits the database with joins to the author and hometown tables.
    b = Book.objects.select_related('author__hometown').get(id=4)
    p = b.author         # Doesn't hit the database.
    c = p.hometown       # Doesn't hit the database.
    
    # Without select_related()...
    b = Book.objects.get(id=4)  # Hits the database.
    p = b.author         # Hits the database.
    c = p.hometown       # Hits the database.
    ```

    









#### 一、QuerySet

- **可切片：**使用Python 的切片语法来限制`查询集`记录的数目 。它等同于SQL 的`LIMIT` 和`OFFSET` 子句。

  ```python
  Entry.objects.all()[:5]      # (LIMIT 5)
  Entry.objects.all()[5:10]    # (OFFSET 5 LIMIT 5)
  
  #不支持负的索引（例如Entry.objects.all()[-1]）。通常，查询集 的切片返回一个新的查询集 —— 它不会执行查询。
  ```

- **可迭代**：本身就是一个列表集合

  ```python
  articleList=models.Article.objects.all()
  for article in articleList:
      print(article.title)
  ```

- **惰性查询**：`查询集` 是惰性执行的 —— 创建`查询集`不会带来任何数据库的访问。你可以将过滤器保持一整天，直到`查询集` 需要求值时，Django 才会真正运行这个查询。（关于惰性是不是在迭代器的地方听过呀）

  ```python
  queryResult=models.Article.objects.all() # not hits database，通过看到的打印的翻译出来的sql语句记录，你会发现单纯的这句话并没有sql语句打印，也就是这里并没有进入数据库查询
   
  print(queryResult) # hits database 进入 
  for article in queryResult:
      print(article.title)    # hits database
   
  #if判断的时候也会执行，if queryResult:pass、
  #一般来说，只有在“请求”查询集 的结果时才会到数据库中去获取它们。当你确实需要结果时，查询集 通过访问数据库来求值。 关于求值发生的准确时间
  ```

- **缓存机制**：每个`查询集`都包含一个缓存来最小化对数据库的访问。理解它是如何工作的将让你编写最高效的代码。叫做queryset缓存空间

  在一个新创建的`查询集`中，缓存为空。首次对`查询集`进行求值 —— 同时发生数据库查询 ——Django 将保存查询的结果到`查询集(非简单查询的查询结果，简单查询往下看。)`的缓存中并返回明确请求的结果（例如，如果正在迭代`查询集`，则返回下一个结果）。接下来对该`查询集` 的求值将重用缓存的结果。

  请牢记这个缓存行为，因为对`查询集`使用不当的话，它会坑你的。例如，下面的语句创建两个`查询集`，对它们求值，然后扔掉它们：

  ```python
  print([a.title for a in models.Article.objects.all()])
  print([a.create_time for a in models.Article.objects.all()])
  
  #这意味着相同的数据库查询将执行两次，显然倍增了你的数据库负载。同时，还有可能两个结果列表并不包含相同的数据库记录，因为在两次请求期间有可能有Article被添加进来或删除掉。为了避免这个问题，只需保存查询集并重新使用它：
  queryResult=models.Article.objects.all()
  print([a.title for a in queryResult])
  print([a.create_time for a in queryResult])
  ```

  - **何时查询集不会被缓存?**  查询集不会永远缓存它们的结果。当只对查询集的部分进行求值时会检查缓存， 如果这个部分不在缓存中，那么接下来查询返回的记录都将不会被缓存。所以，这意味着使用切片或索引来限制查询集将不会填充缓存。

    例如，重复获取查询集对象中一个特定的索引将每次都查询数据库：

    ```python
    >>> queryset = Entry.objects.all()
    >>> print queryset[5] # Queries the database
    >>> print queryset[5] # Queries the database again
    
    #然而，如果已经对全部查询集求值过，则将检查缓存：
    >>> queryset = Entry.objects.all()
    >>> [entry for entry in queryset] # Queries the database
    >>> print queryset[5] # Uses cache
    >>> print queryset[5] # Uses cache
    
    -----------------------------------------------------------
    #下面是一些其它例子，它们会使得全部的查询集被求值并填充到缓存中：
    >>> [entry for entry in queryset]
    >>> bool(queryset)
    >>> entry in queryset
    >>> list(queryset)
    -----------------------------------------------------------
    #注意：简单地打印查询集不会填充缓存。
    queryResult=models.Article.objects.all()
    print(queryResult) #  hits database
    print(queryResult) #  hits database
    ```

- ##### exists() 与 iterater() 方法

  - **exists**：简单的使用if语句进行判断也会完全执行整个queryset并且把数据放入cache，虽然你并不需要这些 数据！为了避免这个，可以用exists()方法来检查是否有数据：

    ```python
    if queryResult.exists():
       #SELECT (1) AS "a" FROM "blog_article" LIMIT 1; args=()
          print("exists...")
    ```

  - **iterator**:当queryset非常巨大时，cache会成为问题。处理成千上万的记录时，将它们一次装入内存是很浪费的。更糟糕的是，巨大的queryset可能会锁住系统 进程，让你的程序濒临崩溃。要避免在遍历数据的同时产生queryset cache，可以使用iterator()方法 来获取数据，处理完数据就将其丢弃。

    ```python
    objs = Book.objects.all().iterator()  #objs变成了一个生成器，生成器也是迭代器，但是生成器有个特点，就是取完值就不能再取了
    # iterator()可以一次只从数据库获取少量数据，这样可以节省内存
    for obj in objs:
        print(obj.title)
    #BUT,再次遍历没有打印,因为迭代器已经在上一次遍历(next)到最后一次了,没得遍历了
    for obj in objs:
        print(obj.title)
    ```

    
    

#### 二、中介模型

​	处理类似搭配 pizza 和 topping 这样简单的多对多关系时，使用标准的`ManyToManyField`  就可以了。但是，有时你可能需要关联数据到两个模型之间的关系上。例如，有这样一个应用，它记录音乐家所属的音乐小组。我们可以用一个`ManyToManyField` 表示小组和成员之间的多对多关系。但是，有时你可能想知道更多成员关系的细节，比如成员是何时加入小组的。

​	对于这些情况，Django 允许你指定一个中介模型来定义多对多关系。 你可以将其他字段放在中介模型里面。源模型的`ManyToManyField` 字段将使用`through` 参数指向中介模型。对于上面的音乐小组的例子，代码如下：

```python
from django.db import models
 
class Person(models.Model):
    name = models.CharField(max_length=128)
 
    def __str__(self):              # __unicode__ on Python 2
        return self.name
 
class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(Person, through='Membership')  #这里使用through表示将使用Membership来当第三张多表
 
    def __str__(self):              # __unicode__ on Python 2
        return self.name
 
class Membership(models.Model):
    person = models.ForeignKey(Person)
    group = models.ForeignKey(Group)
    date_joined = models.DateField()
    invite_reason = models.CharField(max_length=64)
```

​	既然你已经设置好`ManyToManyField` 来使用中介模型（在这个例子中就是`Membership`），接下来你要开始创建多对多关系。你要做的就是创建中介模型的实例：

```python
>>> ringo = Person.objects.create(name="Ringo Starr")
>>> paul = Person.objects.create(name="Paul McCartney")  #实例出一条Person记录
>>> beatles = Group.objects.create(name="The Beatles")   #实例出一个group记录
>>> m1 = Membership(person=ringo, group=beatles,  #两条关联管理对象，相互关联各自的一条记录
...     date_joined=date(1962, 8, 16),
...     invite_reason="Needed a new drummer.")
>>> m1.save()
>>> beatles.members.all() #使用members关联管理对象也就是正向查询
[<Person: Ringo Starr>]

>>> ringo.group_set.all()
[<Group: The Beatles>]
>>> m2 = Membership.objects.create(person=paul, group=beatles,
...     date_joined=date(1960, 8, 1),
...     invite_reason="Wanted to form a band.")
>>> beatles.members.all()
[<Person: Ringo Starr>, <Person: Paul McCartney>]

#注意：与普通的多对多字段不同，你不能使用add、 create和赋值语句（比如，beatles.members = [...]）来创建关系
# THIS WILL NOT WORK
>>> beatles.members.add(john)  #单独的加入一条记录，显然没有beatles中的关联管理字段，所以不行
# NEITHER WILL THIS
>>> beatles.members.create(name="George Harrison")  #光创建一个姓名跟不行
# AND NEITHER WILL THIS
>>> beatles.members = [john, paul, ringo, george]  #更不行

#为什么不能这样做？ 这是因为你不能只创建 Person和 Group之间的关联关系，你还要指定 Membership模型中所需要的所有信息；而简单的add、create 和赋值语句是做不到这一点的。所以它们不能在使用中介模型的多对多关系中使用。此时，唯一的办法就是创建中介模型的实例。 remove()方法被禁用也是出于同样的原因。但是clear() 方法却是可用的。它可以清空某个实例所有的多对多关系：
>>> # Beatles have broken up
>>> beatles.members.clear()
>>> # Note that this deletes the intermediate model instances
>>> Membership.objects.all()
[]
```



#### 三、查询优化

##### 表数据及准备：

```python
class UserInfo(AbstractUser):
    """
    用户信息
    """
    nid = models.BigAutoField(primary_key=True)
    nickname = models.CharField(verbose_name='昵称', max_length=32)
    telephone = models.CharField(max_length=11, blank=True, null=True, unique=True, verbose_name='手机号码')
    avatar = models.FileField(verbose_name='头像',upload_to = 'avatar/',default="/avatar/default.png")
    create_time = models.DateTimeField(verbose_name='创建时间', auto_now_add=True)
 
    fans = models.ManyToManyField(verbose_name='粉丝们',
                                  to='UserInfo',
                                  through='UserFans',
                                  related_name='f',
                                  through_fields=('user', 'follower'))
 
    def __str__(self):
        return self.username
 
class UserFans(models.Model):
    """
    互粉关系表
    """
    nid = models.AutoField(primary_key=True)
    user = models.ForeignKey(verbose_name='博主', to='UserInfo', to_field='nid', related_name='users')
    follower = models.ForeignKey(verbose_name='粉丝', to='UserInfo', to_field='nid', related_name='followers')
 
class Blog(models.Model):
 
    """
    博客信息
    """
    nid = models.BigAutoField(primary_key=True)
    title = models.CharField(verbose_name='个人博客标题', max_length=64)
    site = models.CharField(verbose_name='个人博客后缀', max_length=32, unique=True)
    theme = models.CharField(verbose_name='博客主题', max_length=32)
    user = models.OneToOneField(to='UserInfo', to_field='nid')
    def __str__(self):
        return self.title
 
class Category(models.Model):
    """
    博主个人文章分类表
    """
    nid = models.AutoField(primary_key=True)
    title = models.CharField(verbose_name='分类标题', max_length=32)
 
    blog = models.ForeignKey(verbose_name='所属博客', to='Blog', to_field='nid')
 
class Article(models.Model):
 
    nid = models.BigAutoField(primary_key=True)
    title = models.CharField(max_length=50, verbose_name='文章标题')
    desc = models.CharField(max_length=255, verbose_name='文章描述')
    read_count = models.IntegerField(default=0)
    comment_count= models.IntegerField(default=0)
    up_count = models.IntegerField(default=0)
    down_count = models.IntegerField(default=0)
    category = models.ForeignKey(verbose_name='文章类型', to='Category', to_field='nid', null=True)
    create_time = models.DateField(verbose_name='创建时间')
    blog = models.ForeignKey(verbose_name='所属博客', to='Blog', to_field='nid')
    tags = models.ManyToManyField(
        to="Tag",
        through='Article2Tag',
        through_fields=('article', 'tag'),
)
 
 
class ArticleDetail(models.Model):
    """
    文章详细表
    """
    nid = models.AutoField(primary_key=True)
    content = models.TextField(verbose_name='文章内容', )
 
    article = models.OneToOneField(verbose_name='所属文章', to='Article', to_field='nid')
 
 
class Comment(models.Model):
    """
    评论表
    """
    nid = models.BigAutoField(primary_key=True)
    article = models.ForeignKey(verbose_name='评论文章', to='Article', to_field='nid')
    content = models.CharField(verbose_name='评论内容', max_length=255)
    create_time = models.DateTimeField(verbose_name='创建时间', auto_now_add=True)
 
    parent_comment = models.ForeignKey('self', blank=True, null=True, verbose_name='父级评论')
    user = models.ForeignKey(verbose_name='评论者', to='UserInfo', to_field='nid')
 
    up_count = models.IntegerField(default=0)
 
    def __str__(self):
        return self.content
 
class ArticleUpDown(models.Model):
    """
    点赞表
    """
    nid = models.AutoField(primary_key=True)
    user = models.ForeignKey('UserInfo', null=True)
    article = models.ForeignKey("Article", null=True)
    models.BooleanField(verbose_name='是否赞')
 
class CommentUp(models.Model):
    """
    点赞表
    """
    nid = models.AutoField(primary_key=True)
    user = models.ForeignKey('UserInfo', null=True)
    comment = models.ForeignKey("Comment", null=True)
 
 
class Tag(models.Model):
    nid = models.AutoField(primary_key=True)
    title = models.CharField(verbose_name='标签名称', max_length=32)
    blog = models.ForeignKey(verbose_name='所属博客', to='Blog', to_field='nid')
 
 
 
class Article2Tag(models.Model):
    nid = models.AutoField(primary_key=True)
    article = models.ForeignKey(verbose_name='文章', to="Article", to_field='nid')
    tag = models.ForeignKey(verbose_name='标签', to="Tag", to_field='nid')
```



#### select_related

- **简单使用：**对于一对一字段（OneToOneField）和外键字段（ForeignKey），可以使用select_related 来对QuerySet进行优化。

  select_related 返回一个`QuerySet`，当执行它的查询时它沿着外键关系查询关联的对象的数据。它会生成一个复杂的查询并引起性能的损耗，但是在以后使用外键关系时将不需要数据库查询。简单说，在对QuerySet使用select_related()函数后，Django会获取相应外键对应的对象，从而在之后需要的时候不必再查询数据库了。

  下面的例子解释了普通查询和`select_related()` 查询的区别。查询id=2的文章的分类名称,下面是一个标准的查询：

  ```python
  # Hits the database.
  article=models.Article.objects.get(nid=2)
   
  # Hits the database again to get the related Blog object.
  print(article.category.title)
  
  '''
  SELECT
      "blog_article"."nid",
      "blog_article"."title",
      "blog_article"."desc",
      "blog_article"."read_count",
      "blog_article"."comment_count",
      "blog_article"."up_count",
      "blog_article"."down_count",
      "blog_article"."category_id",
      "blog_article"."create_time",
       "blog_article"."blog_id",
       "blog_article"."article_type_id"
               FROM "blog_article"
               WHERE "blog_article"."nid" = 2; args=(2,)
   
  SELECT
       "blog_category"."nid",
       "blog_category"."title",
       "blog_category"."blog_id"
                FROM "blog_category"
                WHERE "blog_category"."nid" = 4; args=(4,)
  '''
  
  #如果我们使用select_related()函数
  articleList=models.Article.objects.select_related("category").all()
   
      for article_obj in articleList:
          #  Doesn't hit the database, because article_obj.category
          #  has been prepopulated in the previous query.
          print(article_obj.category.title)
  '''
  SELECT
       "blog_article"."nid",
       "blog_article"."title",
       "blog_article"."desc",
       "blog_article"."read_count",
       "blog_article"."comment_count",
       "blog_article"."up_count",
       "blog_article"."down_count",
       "blog_article"."category_id",
       "blog_article"."create_time",
       "blog_article"."blog_id",
       "blog_article"."article_type_id",
   
       "blog_category"."nid",
       "blog_category"."title",
       "blog_category"."blog_id"
   
  FROM "blog_article"
  LEFT OUTER JOIN "blog_category" ON ("blog_article"."category_id" = "blog_category"."nid");
  '''            
  ```

  

- **多外键查询**：这是针对category的外键查询，如果是另外一个外键呢？让我们一起看下：

  ```python
  article=models.Article.objects.select_related("category").get(nid=1)
  print(article.articledetail)
  
  # 观察logging结果，发现依然需要查询两次，所以需要改为：
  article=models.Article.objects.select_related("category","articledetail").get(nid=1)
  print(article.articledetail)
  #或则：
  article=models.Article.objects
  　　　　　　　　　　　　　.select_related("category")
  　　　　　　　　　　　　　.select_related("articledetail")
  　　　　　　　　　　　　　.get(nid=1)  # django 1.7 支持链式操作
  print(article.articledetail)
  ```

  

- **深度查询**：

  ```python
  # 查询id=1的文章的用户姓名
   
      article=models.Article.objects.select_related("blog").get(nid=1)
      print(article.blog.user.username)
      
  #依然需要查询两次：
  SELECT
      "blog_article"."nid",
      "blog_article"."title",
      ......
   
       "blog_blog"."nid",
       "blog_blog"."title",
   
     FROM "blog_article" INNER JOIN "blog_blog" ON ("blog_article"."blog_id" = "blog_blog"."nid")
     WHERE "blog_article"."nid" = 1;
    
  SELECT
      "blog_userinfo"."password",
      "blog_userinfo"."last_login",
      ......
   
  FROM "blog_userinfo"
  WHERE "blog_userinfo"."nid" = 1;
  ------------------------------------------------------------
  
  #这是因为第一次查询没有query到userInfo表，所以，修改如下：
  article=models.Article.objects.select_related("blog__user").get(nid=1)
  print(article.blog.user.username)
  
  SELECT
   
  "blog_article"."nid", "blog_article"."title",
  ......
   
   "blog_blog"."nid", "blog_blog"."title",
  ......
   
   "blog_userinfo"."password", "blog_userinfo"."last_login",
  ......
   
  FROM "blog_article"
   
  INNER JOIN "blog_blog" ON ("blog_article"."blog_id" = "blog_blog"."nid")
   
  INNER JOIN "blog_userinfo" ON ("blog_blog"."user_id" = "blog_userinfo"."nid")
  WHERE "blog_article"."nid" = 1;
  ```

- **总结**：

  ```
  1、select_related主要针一对一和多对一关系进行优化。
  2、select_related使用SQL的JOIN语句进行优化，通过减少SQL查询的次数来进行优化、提高性能。
  3、可以通过可变长参数指定需要select_related的字段名。也可以通过使用双下划线“__”连接字段名来实现指定的递归查询。
  4、没有指定的字段不会缓存，没有指定的深度不会缓存，如果要访问的话Django会再次进行SQL查询。
  5、也可以通过depth参数指定递归的深度，Django会自动缓存指定深度内所有的字段。如果要访问指定深度外的字段，Django会再次进行SQL查询。
  6、也接受无参数的调用，Django会尽可能深的递归查询所有的字段。但注意有Django递归的限制和性能的浪费。
  7、Django >= 1.7，链式调用的select_related相当于使用可变长参数。Django < 1.7，链式调用会导致前边的select_related失效，只保留最后一个。
  ```

  

#### prefetch_related()

​		对于多对多字段（ManyToManyField）和一对多字段，可以使用prefetch_related()来进行优化。prefetch_related()和select_related()的设计目的很相似，都是为了减少SQL查询的数量，但是实现的方式不一样。后者是通过JOIN语句，在SQL查询内解决问题。但是对于多对多关系，使用SQL语句解决就显得有些不太明智，因为JOIN得到的表将会很长，会导致SQL语句运行时间的增加和内存占用的增加。若有n个对象，每个对象的多对多字段对应Mi条，就会生成Σ(n)Mi 行的结果表。

- prefetch_related()的解决方法是，分别查询每个表，然后用Python处理他们之间的关系。

  ```python
  # 查询所有文章关联的所有标签
      article_obj=models.Article.objects.all()
      for i in article_obj:
          print(i.tags.all())  #4篇文章: hits database 5
          
  # 改为prefetch_related：查询所有文章关联的所有标签
      article_obj=models.Article.objects.prefetch_related("tags").all()
      for i in article_obj:
          print(i.tags.all())  #4篇文章: hits database 2
          
  '''
  SELECT "blog_article"."nid",
                 "blog_article"."title",
                 ......
   
  FROM "blog_article";
   
  SELECT
    ("blog_article2tag"."article_id") AS "_prefetch_related_val_article_id",
    "blog_tag"."nid",
    "blog_tag"."title",
    "blog_tag"."blog_id"
     FROM "blog_tag"
    INNER JOIN "blog_article2tag" ON ("blog_tag"."nid" = "blog_article2tag"."tag_id")
    WHERE "blog_article2tag"."article_id" IN (1, 2, 3, 4);'''        
  ```

  

#### 四、extra

- **语法**：

  ```python
   extra(select=None, where=None, params=None, tables=None, order_by=None, select_params=None)
      
      #有些情况下，Django的查询语法难以简单的表达复杂的 WHERE 子句，对于这种情况, Django 提供了 extra() QuerySet修改机制 — 它能在 QuerySet生成的SQL从句中注入新子句。extra可以指定一个或多个 参数,例如 select, where or tables. 这些参数都不是必须的，但是你至少要使用一个!要注意这些额外的方式对不同的数据库引擎可能存在移植性问题.(因为你在显式的书写SQL语句),除非万不得已,尽量避免这样做
  ```

- **参数之select**：

  ```python
  #The select 参数可以让你在 SELECT 从句中添加其他字段信息，它应该是一个字典，存放着属性名到 SQL 从句的映射。
  queryResult=models.Article
  　　　　　　　　　　　.objects.extra(select={'is_recent': "create_time > '2017-09-05'"})
  #结果集中每个 Entry 对象都有一个额外的属性is_recent, 它是一个布尔值，表示 Article对象的create_time 是否晚于2017-09-05.　
  
  #练习：
  article_obj=models.Article.objects
  　　　　　　　　　　　　　　.filter(nid=1)
  　　　　　　　　　　　　　　.extra(select={"standard_time":"strftime('%%Y-%%m-%%d',create_time)"})
  　　　　　　　　　　　　　　.values("standard_time","nid","title")
  print(article_obj)  # <QuerySet [{'title': 'MongoDb 入门教程', 'standard_time': '2017-09-03', 'nid': 1}]>
  ```

  

#### 

