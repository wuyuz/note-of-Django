## ORM进阶！！！！！

**注**：数据库迁移的时候出现一个警告

```python
WARNINGS: 
?: (mysql.W002) MySQL Strict Mode is not set for database connection 'default'
HINT: MySQL's Strict Mode fixes many data integrity problems in MySQL, such as data truncation upon insertion, by escalating warnings into errors. It is strongly recommended you activate it.（原因是数据库没有配置严格模式）

# 解决方法，在配置数据库的时候添加一个OPTIONS的参数
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.mysql",
        "NAME": "你的数据库名称",  # 需要自己手动创建数据库
        "USER": "数据库用户名",
        "PASSWORD": "数据库密码",
        "HOST": "数据库IP",
        "POST": 3306,
         'OPTIONS': {
    		'init_command': "SET sql_mode='STRICT_TRANS_TABLES'"},
    }
}
```



#### 常用字段详解：

- **AtuoField** ：自增的整形字段，必填参数primary_key=True，则成为数据库的主键。无该字段时，django自动创建。一个model不能有两个AutoField字段。

- **IntegerField**： 一个整数类型。数值的范围是 -2147483648 ~ 2147483647。

- **CharField：** 字符类型，必须提供max_length参数。max_length表示字符的长度。

- **DateField:**  日期类型，日期格式为YYYY-MM-DD，相当于Python中的datetime.date的实例。
  - **参数**：auto_now :每次修改时修改当前时间；auto_now_add: 新创建对象时自动添加当前日期时间。
  
  - **注意**：1、DateTimeField也是继承它；
  
    ​		   2、注意默认的时间时UTC（比我们当地少8小时），如果想用当地时间，修改setting的USE_TZ = False;      		   3、auto_now 和 add_now_add 互斥，不能同时设置。
- **DateTimeField**：日期时间字段，格式为YYYY-MM-DD HH:MM[:ss[.uuuuuu]][TZ]，相当于Python中的datetime.datetime的实例。
- **DecimalField**：表示小数
  
  - 参数：max_digits，小数总长度； decimal_places，小数位长度。如：DecimalField(max_digits = 5,decimal_palces = 2)。
- **BinaryField**： 二进制，比如保存一些图片上传的字节流

- 其他的字段请查看上一章



#### 自定义字段

- 我们不难发现，Django并没有为我们提供varchar类型（这只是个引子，其实CharField字段在某个角度也是varchar)，但总的来说就是我们可以自定义字段，根据mysql认识的去写字段类型。

  ```python
  基本语法：
  class UnsignedIntegerField(models.IntegerField):
      def db_type(self, connection):
          return 'integer UNSIGNED'  #返回值为字段在数据库中的属性
      
   -----------------------------------------------------------------   
   #具体用法：自定义一个char类型字段：
  class MyCharField(models.Field):  # 都要继承models.Field
      def __init__(self, max_length, *args, **kwargs):
          self.max_length = max_length 
          super(MyCharField, self).__init__(max_length=max_length, *args, **kwargs)
  
      def db_type(self, connection):
          return 'char(%s)' % self.max_length #限定生成数据库表的字段类型为char，长度为max_length指定的值
          #当然我们可以设置 return varchar(%s)%self.max_length ,这就是个varchar字段
        
      
  ----------------------使用自定义字段-------------------------------
  from django.db import models
  class MyCharField(models.Field):
      def __init__(self, max_length, *args, **kwargs): #这个init完全可以不写，只是方便理解
          # self.max_length = max_length  # 不需要这个也是可以的，会去父类找
          super(MyCharField,self).__init__(max_length=max_length, *args, **kwargs)
          
      def db_type(self, connection):
          return 'char(%s)'%self.max_length
  
  class Person(models.Model):
      name = models.CharField(max_length=32)
      age = models.IntegerField()
      birth = models.DateTimeField(auto_now_add=True)
      phone = MyCharField(max_length=11)   
      
  # 这里执行迁移饿时候会问我们要默认值，这之后选择1，然后输入11个字符串即可    
  ```

  

#### 字段参数详解（结合上例）

- **null** ：数据库字符串是否可以为空；记得上面我们的phone字段要我们设置默认值吗，我们就可以设置

  ```python
  phone = MyCharField(max_length=11, null=True) 
  ```

- **db_column** : 数据库中字段的列名

  ```python
   # 一般情况下，一张表中的字段名为我们定义这个字段时 '=' 左边的变量名，如下列的'name'，但有时我们想取一个详细的名字，这时候我们就用db_column = 'username' 此时表中的字段名是'usrname'，但是我们具体操作时还是使用name
  name = models.CharField(max_length=32, db_column='username')   # 具体操作还是name
  ```

- **default** ： 数据库中字段的默认值

  ```python
  age = models.IntegerField(default=18)  # 设置默认值，没有时用它，有就覆盖
  ```

- **primary_key** : 数据库中字段是否为主键

- **db_index** :   数据库中字段是否可以建立索引; 使查询速度变快，但是存的时候很慢

  ```python
  age = models.IntegerField(default=18, db_index=True)  # 使age字段为索引
  ```

- **unique** ：数据库中字段是否可以建立唯一索引

  ```python
  name = models.CharField(max_length=32, db_column='username', unique=True) # 字段唯一
  ```

- **unique_for_date** ：数据库中字段【日期】部分是否可以建立唯一索引；响应的还有月和日期唯一：略

  

- **verbose_name** :   Admin中显示的字段名称; 也就是在form表单中要显示的

  ```python
  name = models.CharField(max_length=32, db_column='username', unique=True, verbose_name='姓名')   # 在后台管理中显示的名字
  
  # 其实查看CharFeid源码(其他很多字段也是，因为都是继承得Field，可以看看它里面得参数)，它的对应得参数第一个就是verbose_name,所以上面得写法等价于：
  name = models.CharField('姓名',max_length=32, db_column='username', unique=True)
  ```

- **blank**:  Admin中是否允许用户输入为空。

  ```python
   #在admin中修改或添加数据时models中有一层校验，即使该字段数据库中是允许为空的，但是blank = False（默认）会要求admin的form表单不能为空。 相应得form表单中得选项会变得浅，不会强调
  
  phone = MyCharField(max_length=11, blank = True, null = True) # 表示admin中的form表单可以为空，数据库数据也可以为空，blank和null两者多连用
  ```

- **editable** ：Admin中是否可以编辑，默认都是可编辑得（True）

  ```python
  age = models.IntegerField(default=18, db_index=True, editable=False) # 这样admin中就不会出现这个age修改框了
  ```

- **help_text** ：Admin中该字段的提示信息

  ```python
      name = models.CharField(max_length=32, db_column='username', unique=True, verbose_name='姓名',help_text='不能是乳名')  # 作用于admin中得form表单
  
  ```

- **choices** ：Admin中显示选择框的内容，用不变动的数据放在内存中从而避免跨表操作

  ```python
      gander = models.IntegerField(verbose_name='性别', choices=[(0, '男'), (1, '女'),],default=1) # 最好都设置默认值，
      hobby = models.BooleanField('兴趣', choices=((0, '篮球'),(1, '足球')),default=0) #会为form表单中添加一个多选框
  ```

- **error_messages**  ： 自定义错误信息（字典类型），从而定制想要显示的错误信息；

  ```python
  字典健：null, blank, invalid, invalid_choice, unique, and unique_for_date，如：{'null': "不能为空.", 'invalid': '格式错误'}
  
  # 举例 
  from django import forms
  class TestForm1(forms.Form):
      username = forms.CharField(
          min_length=3,
          max_length=20,
          error_messages={'required':'用户名不能为空', # 用户名为空时显示的错误信息
                          'min_length':'大于3个字符',
                          'max_length':'小于20个字符'}
          )
  ```

- **validators** ：自定义错误验证（列表类型），从而定制想要的验证规则

  ```python
  from django.core.validators import RegexValidator # 导入校验
  from django.core.validators import EmailValidator,URLValidator,DecimalValidator, MaxLengthValidator,MinLengthValidator,MaxValueValidator,MinValueValidator
   
  如：
     test = models.CharField(
                             max_length=32,
                             error_messages={
                                      'c1': '优先错信息1',
                                      'c2': '优先错信息2',
                                      'c3': '优先错信息3',
                                  },
                             validators=[
                                      RegexValidator(regex='root_\d+', message='错误了', code='c1'),
                                      RegexValidator(regex='root_112233\d+', message='又错误了', code='c2'),
                                      EmailValidator(message='又错误了', code='c3'), ]
                              ) 
  ```
  


#### Model Meta 参数

- 主要用于篇日志数据表的一些元信息

  ```python
  class UserInfo(models.Model):
      nid = models.AutoField(primary_key=True)
      username = models.CharField(max_length=32)
   
      class Meta:
          # 数据库中生成的表名称 默认 app名称 + 下划线 + 类名，db_column是admin显示字段
          db_table = "table_name"  #自改，作用于数据库
   
          # admin中显示的表名称，表中的字段也有verbose_name,显示的是字段名
          verbose_name = '个人信息'  # 作用于admin
   
          # verbose_name加s  效果和verbose_name 类式，只是英文是表中有多个值会加s
          verbose_name_plural = '所有用户信息'
   
          # 联合索引 
          index_together = [
              ("pub_date", "deadline"),   # 应为两个存在的字段，添加了一起的索引，最左原则
          ]
   
          # 联合唯一索引
          unique_together = (("driver", "restaurant"),)   # 应为两个存在的字段，但是加了一个约束唯一
  ```



##### 简单的了解admin：

- 我们在setting.py文件中的 INSTALLED_APPS  列表中可以找到一个admin应用，这是django自带的一个后台管理应用，如果不是使用django打开，那么就需要配置admin应用（自带的）：添加`'django.contrib.admin'` 到INSTALLED_APPS中，并且注销url.py中的路径。然后我们就可以访问这个路径了。

- 创建超级管理员：`python manage.py createsuperuser`  再输入密码：超过八位，且数字英文混合。

- 在后台中进行添加表格或数据, 在应用中的admin.py中进行修改

  ```python
  from django.contrib import admin
  from app import models
  
  # Register your models here.
  admin.site.register(models.Person)  # 这样就可以注册一张表了
  ```



#### 必知必会13条

##### 测试文件：

**方法一**：在pycharm中的运行栏（Django项目中）中有一个Python Console的按钮，点击即可打开Django的shell脚本。或则在项目目录下在cmd中输入：`python manage.py shell`  也可以打开测试。

![1561363446082](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1561363446082.png)



**方法二**：为了方便我们测试大量models数据，我们在项目下创建一个test文件夹，再其中写一个py文件专门用于测试代码。

```python
import os
# 可在manage.py中找到下面这句话，赋值粘贴
# os.environ.setdefault('DJANGO_SETTING_MODULE', 'mysite.settings') # 给系统的环境变量中设置了一个键值对，第一个是Django的全局配置文件，第二个事settings文件

os.environ['DJANGO_SETTINGS_MODULE'] = 'mysite.settings'  # 如果前一种方式报错就用这也种 
import django
django.setup()  # 起django项目，就相当于 python manage.py shell
from app import models
ret = models.Person.objects.all()
print(ret)


-----------models准备工作，设置返回的值--------------
from django.db import models

class MyCharField(models.Field):
    def __init__(self, max_length, *args, **kwargs):
        # self.max_length = max_length
        super(MyCharField,self).__init__(max_length=max_length, *args, **kwargs)
        
    def db_type(self, connection):
        return 'char(%s)'%self.max_length

class Person(models.Model):
    name = models.CharField(max_length=32, db_column='username', unique=True, verbose_name='姓名',help_text='不能是乳名')
    age = models.IntegerField(default=18, db_index=True)
    birth = models.DateTimeField(auto_now_add=True)
    phone = MyCharField(max_length=11, blank=True, null=True)
    gander = models.IntegerField(verbose_name='性别', choices=[(0, '男'), (1, '女'),],default=1, blank=True) # 最好都设置默认值，
    hobby = models.BooleanField('兴趣', choices=((0, '篮球'),(1, '足球')),default=0) #会为form表单中添加一个多选框
        
    def __str__(self):
        return f'{self.pk} - {self.name}'
    
    class Meta:
        db_table = 'person_table'
```

- **具体用法**：在测试文件中

  ```python
  import os
  # 可在manage.py中找到这句话，赋值粘贴
  # os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'mysite.settings') # 给系统的环境变量中设置了一个键值对，第一个是Django的全局配置文件，第二个事settings文件
  
  os.environ['DJANGO_SETTINGS_MODULE'] = 'mysite.settings'
  import django
  
  django.setup()  # 起django项目，就相当于 python manage.py shell
  from app import models
  
  # all()  获取所有数据 -- > QuerySet[……] 对象列表
  ret = models.Person.objects.all()
  print(ret)  #<QuerySet [<Person: 6 - 立兴>, …… <Person: 9 - taibai>]>
  
  # get() 获取满足条件的一个数据 --> 一个对象 （获取多个，或没有都会报错）
  ret = models.Person.objects.get(pk=6)
  print(ret)  # 6 - 立兴
  
  #filter()  获取满足条件的所有对象 -- > QuerySet[……] 对象列表
  ret = models.Person.objects.filter(pk=6)
  print(ret) # <QuerySet [<Person: 6 - 立兴>]>
  
  #exclude() 获取不满足条件的所有数据 -- > QuerySet[……] 对象列表
  ret = models.Person.objects.exclude(pk=6)
  print(ret) #<QuerySet [<Person: 7 - 小伍>…, <Person: 9 - taibai>]>
  
  #values()  获取对象所有的字段和字段的值 --> QuerySet[{},{}]
  #values('字段','字段')  获取对象某些字段和字段的值 --> QuerySet[{},{}]
  ret = models.Person.objects.values()
  print(ret)  #<QuerySet [{'id': 6, 'name': '立兴', 'age': 24, 'birth': datetime.datetime(2019, 6, 24, 15, 36, 11, 332315), 'phone': '12345678911', 'gander': 1, 'hobby': False}, {'id': 7, 'name': '小伍'…… },]>
  ret1 = models.Person.objects.values('name') #<QuerySet [{'name': 'taibai'}, {'name': '小伍'}, {'name': '帅哥'}, {'name': '王立兴'}]>
  print(ret1) #<QuerySet [{'name': 'taibai'}, {'name': '小伍'}, {'name': '帅哥'}, {'name': '立兴'}]>
  
  #values_list() 获取对象的所有数据，返回的事元组，作用于QuerySet
  #values_list('字段','字段') 获取对象的某些数据，返回的事元组
  ret = models.Person.objects.values_list()
  ret1 = models.Person.objects.values_list('name','id')  #
  print(ret) # <QuerySet [(6, '立兴', 24, ……, '12345678911', 1, False), (7, '小伍',…… )]>
  print(ret1) # <QuerySet [('taibai', 9), ('小伍', 7), ('帅哥', 8), ('王立兴', 6)]>
  
  # order_by()  # 排序，还可以设置多个排序顺序order_by('age','name') age相同的按照name排序
  ret = models.Person.objects.all().order_by('-pk')  # 降序加 '-'
  print(ret) #<QuerySet [<Person: 9 - taibai>, <Person: 8 - 帅哥>, <Person: 7 - 小伍>, <Person: 6 - 王立兴>]>
  
  #reverse()  # 反转必须针对已经排了序，也就是连用order_by,同时可以在Meta中设置ording = ('pk',)，但是不建议，有坑（和distinct冲突）
  ret = models.Person.objects.all().order_by('-pk').reverse()
  print(ret)  # <QuerySet [<Person: 6 - 立兴>, <Person: 7 - 小伍>, <Person: 8 - 帅哥>, <Person: 9 - taibai>]>
  
  # distinct() # 去重，不支持字段去重，表示完完全全相同才能去重，但是每个记录id肯定不同，但是我们可以针对values中返回的QuerySet进行去重
  ret = models.Person.objects.values('phone').distinct()
  print(ret) #<QuerySet [{'phone': '12345678911'}, {'phone': '12334456711'}, {'phone': None}]>
  
  # count()  计数QuerySet中的个数，当然可以使用len
  ret = models.Person.objects.all().count()
  print(ret, len( models.Person.objects.all())) #4 4
  
  #first() 作用于QuerySet返回一个对象  last() 取最后一个
  ret = models.Person.objects.all().first()  # 查不到返回None
  print(ret) # 6 - 立兴
  
  #exist()  查取数据是否存在，filter查询很慢而且多的话，会很慢
  ret = models.Person.objects.filter(pk=6).exists()
  print(ret) # True
  
  ----------------------------总结------------------------
  返回QuerySet的方法：
  all()
  filter()
  exclude()
  values()
  values_list()
  order_by()
  reverse()
  distinct()
  
  返回一个对象的方法:
  get()
  first()
  last()
  
  返回一个数字的方法：
  count()
  
  返回布尔值的方法：
  exist()
  ```

  

#### 单表的双下划线

- **用于多数据进行筛选**：

  ```python
  import os
  # 可在manage.py中找到这句话，赋值粘贴
  
  os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'mysite.settings') # 给系统的环境变量中设置了一个键值对，第一个是Django的全局配置文件，第二个事settings文件
  # os.environ['DJANGO_SETTINGS_MODULE'] = 'mysite.settings'
  import django
  django.setup()  # 起django项目，就相当于 python manage.py shell
  from app import models
  
  # 大于小于
  ret = models.Person.objects.filter(pk__gt = 7)   # gt  greater than
  ret1 = models.Person.objects.filter(pk__lt = 7)  # gt  less than
  print(ret, ret1) # <QuerySet [<Person: 8 - 帅哥>, <Person: 9 - taibai>]> <QuerySet [<Person: 6 - 王立兴>]>
  
  # 大于等于、小于等于
  ret = models.Person.objects.filter(pk__gte = 7)   # gt  greater than equal
  ret1 = models.Person.objects.filter(pk__lte = 7)  # gt  less than equal
  print(ret, ret1) #<QuerySet [<Person: 7 - 小伍>, <Person: 8 - 帅哥>, <Person: 9 - taibai>]> <QuerySet [<Person: 6 - 王立兴>, <Person: 7 - 小伍>]>
  
  # 区间
  ret = models.Person.objects.filter(pk__gt = 7, pk__gte =8)   # 大于7, 同是大于取最左
  ret1 = models.Person.objects.filter(pk__gt = 7, pk__lte =8)   # 小于等于8 大于7
  ret2 = models.Person.objects.filter(pk__range = [7,8])  # 双边都包含
  print(ret, ret1, ret2) #<QuerySet [<Person: 8 - 帅哥>, <Person: 9 - taibai>]> <QuerySet [<Person: 8 - 帅哥>]> <QuerySet [<Person: 7 - 小伍>, <Person: 8 - 帅哥>]>
  
  #成员判断,没有返回None
  ret = models.Person.objects.filter(pk__in = [1,8])
  print(ret)  #<QuerySet [<Person: 8 - 帅哥>]>
  
  #包含
  ret = models.Person.objects.filter(name__contains = '王')
  print(ret) #<QuerySet [<Person: 6 - 王立兴>]>]
  
  # 包含中忽略大小写
  ret = models.Person.objects.filter(name__icontains = 'A')     # i ignore不理会
  ret1 = models.Person.objects.filter(name__startswith = 'A')   # 以什么开头
  ret2 = models.Person.objects.filter(name__istartswith = 'A')  # 以什么开头,忽略大小写
  print(ret)
  
  #时间筛选,ORM中只支持year，如果想查birth__contains = '2019-06'
  ret = models.Person.objects.filter(birth__year = '2019')
  print(ret) #<QuerySet [<Person: 6 - 王立兴>, <Person: 7 - 小伍>, <Person: 8 - 帅哥>, <Person: 9 - taibai>]>
  
  #筛选出为空的
  ret = models.Person.objects.filter(phone__isnull=True)
  print(ret) #<QuerySet [<Person: 9 - taibai>]>
  ```



#### ForeignKey外键操作 (一对多)

- **重点理解部分**

  ```python
   \#表结构：一对多的关系
  class Publisher(models.Model):
      name = models.CharField(max_length=32, verbose_name="名称")
  
      def __str__(self):
          return self.name
  
  class Book(models.Model):
      title = models.CharField(max_length=32)
      pub = models.ForeignKey(Publisher, related_name='books',related_query_name='xxx',on_delete=models.CASCADE)
  
      def __str__(self):
          return self.title    
  
  -----------------------测试文件----------------------------
  
  import os
  os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'mysite.settings') # 给系统的环境变量中设置了一个键值对，第一个是Django的全局配置文件，第二个事settings文件
  
  import django
  django.setup()  # 起django项目，就相当于 python manage.py shell
  from app import models
  
  ---------------------基于对象查询（记录get）----返回一条记录---------------------
  
  # 正向查询，通过获取到的关联管理对象去拿对应的记录值，拿到管理对象就是一对一结构了
  # 基于对象的查询, 拿到书名对象，该对象中有一个外键外联，我们拿到这个外联对象，该对象默认返回出版社名字
  book_obj = models.Book.objects.get(title='钢铁侠')
  print(book_obj.pub)  # 路飞学城出版社
  print(book_obj.pub.name)  # 等同于上面的方法
  print(type(models.Book.pub))  #<class 'django.db.models.fields.related_descriptors.ForwardManyToOneDescriptor'>
  
  #补充：在多对多的模型时，get获得到一条记录后，反向也是一对多，此时正向直接 关联管理对象.all()
  
  
  #反向查询，通过被关联(关联管理对象管理的表）的某一条记录来反查，含有关联管理对象的表的记录
  # 这里我们通过被关联的表拿到一条记录即pub_obj,此纪录在'一对多'模型中对应'多',所以我们使用关 记录.联表名_set 来获得一个'一'的集合(set)，当再关系管理对象中设置了related_name，优先使用它，反向查询就用它
  pub_obj = models.Publisher.objects.get(pk=1)
  print(pub_obj.book_set,type(pub_obj.book_set)) #app.Book.None  <class 'django.db.models.fields.related_descriptors.create_reverse_many_to_one_manager.<locals>.RelatedManager'>
  # 不难发现打印的类型也是一个关联关联对象，回想我们之前正向查询时我们早就有了关联管理对象(正向），在上面正向查询时，它涉及的是一对一，多的一方在出版社这边
  # 所以直接就可以从每条book记录中拿到唯一的管理对象，但是反向就不行，一个出版社队形多个书，所以这里就需要设立一个集合(set),来管理一个出版社对应的书，
  print(pub_obj.book_set.all())  #<QuerySet [<Book: 桃花仙大战菊花怪>]>
  
  #注意：我们可以不用book_set，这就需要我们在创建关联管理对象pub时配置：related_naem = 'books', 就可以不使用book_set了
  pub_obj = models.Publisher.objects.get(pk=1)
  print(pub_obj.books.all())  #<QuerySet [<Book: 桃花仙大战菊花怪>]>
  
  # 补充：在多对多关系时，使用get拿到被关联表，此时该表没有关联管理对象，就要是使用 表名_set.all()，同样可以设置 related_name;后面具体会介绍
  
  
  --------------- 基于字段查询（跨表filter）----返回一个QuerySet[]-------------------
  
  # 基于字段的查询,使用过滤filter查询，正向
  # 查询 '钢铁侠'对应的出版社
  ret = models.Book.objects.filter(title='钢铁侠').first()
  print(ret.pub.name)
  
  # 基于字段的查询,使用过滤filter查询，正向跨表查询(查询关联关联对象表的记录，多为集合，使用关联管理对象）：在下例子中，我们要拿出版社的书，而我又想从Book表入手，我们现在有的只有
  # 出版社的名字 '老男孩出版社',换句话说在filter中我们能给出的条件是：name = '老男孩出版社',但是这只能在publisher表中这么filter
  #为了解决这个问题，django使用了一个跨表查询机制：pub__name (关联管理对象__name='')
  # 查询老男孩出版社出版的书
  ret1 = models.Book.objects.filter(pub__name='老男孩出版社')
  print(ret1) #<QuerySet [<Book: Mjj>]>
  
  # 基于字段的查询,使用过滤filter查询，反向跨表查询(查询关联关联对象表的记录，多为单个）；与正向不同的是，在下例中
  #我们只知道 出版社的名字 也就是说要使用filter,能给出的条件是 title = '钢铁侠';但是我们要从Publisher出发，这时候
  #我们只能跨表到book中，相应的条件是 books__title = '钢铁侠'（默认情况下是 关联管理对象所在表表名小写__筛选字段，即book_title，但是上面我们能
  #已经设置了关联关联对象的别名related_name = 'books'，所以books_title）
  # 查询出版钢铁侠的出版社
  ret2 = models.Publisher.objects.filter(books__title='钢铁侠')
  print(ret2) #<QuerySet [<Publisher: 路飞学城出版社>]>
  
  # related_query_name = 'xxx' 在关联管理对象中设置这个参数会是跨表查询时以此名取代表名，优先级高于related_name
  ret3 = models.Publisher.objects.filter(xxx__title='钢铁侠')
  print(ret3)  #<QuerySet [<Publisher: 路飞学城出版社>]>
  
  ---------------------------------------------------------
  总结：
  	# 1、当一个对象是一对一的时候，关联管理对象就是只针对我们要的记录,直接 管理对象.属性 就ok； 当对象是一对多的时候就要使用关联关联对象了，此时往往对应一个集合，具体还要看多对多
  
  	# 2、related_name 和 related_query_name 的区别： 前者针对的时反向查询时查找多条记录的表的别名；后者是对跨表查询时用于查询单条记录的表别名
      # 3、注意总的来说 类名_set 和 pub都是关联管理对象
      # 4、每个关联管理对象都有：关联管理对象_id 这个属性，表示其id，等价于关联管理对象.id
  ```



- **外键的总结**：

  ```python
   \#基于对象查询 （返回的是一个或多个对象，多用get）
  book_obj = models.Book.objects.get(pub__name='xxxxx')
  正向(通过关联管理对象)
  	book_obj.pub    ——》  所关联的对象
  	book_obj.pub_id    ——》  所关联的对象id
  
  反向(通过表名)
  	1、没有指定related_name
  		pub_obj.book_set     ——》  关系管理对象    （类名小写_set）
  		pub_obj.book_set.all()   ——》  所关联的所有对象
  
  	2、指定related_name='books'
  		pub_obj.books   ——》  关系管理对象 
  		pub_obj.books.all()   ——》  所关联的所有对象 
  ---------------------------------------------------------------		
  		
  \#基于字段查询 (返回的是一个QuerySet列表，多用filter)   
  正向查询：(通过关联管理对象所在的表，关联管理对象__字段='')
  models.Book.objects.filter(pub__name='xxxxx')
  
  反向查询：(通过要跨的表__字段='')
  	1、没有指定related_name
  		models.Publisher.objects.filter(book__title='xxxxx')
  
  	2、指定related_name=‘books’
  		models.Publisher.objects.filter(books__title='xxxxx')
  
  	3、指定related_query_name='book‘
  		models.Publisher.objects.filter(book__title='xxxxx')
  ```
  
  

#### 多对多关系

​		通过上面我们学习的一对多关系，为我们学习多对多做了铺垫，因为在多对多关系中，每个一记录就是一个一对多关系，不是吗？所以大同小异，只是我们之前的一对多模型中，使用的 `类名_set` 来获取多的一方的集合，在多对多关系中，有一方是可以使用关联管理对象来获得集合的，如：`pub.all()`。

- 有下列表结构

  ```python
  from django.db import models
  
  class Book(models.Model):
      title = models.CharField(max_length=32)
      pub = models.ForeignKey('Publisher',related_name='books',on_delete=models.CASCADE)
      
      def __str__(self):
          return self.title
  
  class Publisher(models.Model):
      name = models.CharField(max_length=32,verbose_name='名称')
      
      def __str__(self):
          return self.name
      
  class Author(models.Model):
      name = models.CharField(max_length=32)
      # 如果是通过作者查书的次数多，我们就把关联管理对象放Author表中（正向方便），反之
      books = models.ManyToManyField('Book')
      
      def __str__(self):
  		return self.name
      
  class AuthorDetail(models.Model):
      addr = models.CharField(max_length=64,null=True)
      author = models.OneToOneField('Author')  # 写在任意一方既可
      
  ```

- **测试数据**（在应用同级目录的test文件下的'多表操作.py'）

  ```python
  import os
  
  os.environ.setdefault("DJANGO_SETTINGS_MODULE", "mysite.settings")
  import django
  
  django.setup()
  from app import models
  
   --------------------------跨表查询------------------------------
   # 多对多的查询
   # 基于对象的查询，获得一个记录，通过记录的关联管理器获得另一张表的记录
   # 正向，通过获得关联管理器.all()
  author_obj = models.Author.objects.get(pk=1)
  print(author_obj.books) #app.Book.None 从一对多的方向拿都是一个关联管理对象
  print(author_obj.books.all()) #关联管理对象的常用方法：all()
  
   #反向（通过获得被关联表的一条记录（可用get、filter），再用 表名_set 来获得管理管理对象）
  book_obj = models.Book.objects.filter(title='Mjj自传').first()  # 使用get(title='Mjj自传') 也是一样的
  print(book_obj.author_set) # app.Author.None 反向也是拿到一个关联管理器，只是用到表名_set(这里没有设置related_name）
  print(book_obj.author_set.all()) # <QuerySet [<Author: Author object>]>
   # 注意：在Author的关联管理器中指定了related_name后就代替author_set
  
  
   # 基于字段的查询，通过在filter中是使用管理器__字段，或表名__字段 筛选获得一个QuerySet列表，
   # 正向，通过在filter中 关联管理器__字段 = '' 去查询另一张表的记录筛选
   # 查询书名为'Mjj自传'的作者
  ret = models.Author.objects.filter(books__title='Mjj自传') # 我们只知道书名，但是现在从作者表出发，显然没有title属性，所以我们通过管理器去另一张表查
  print(ret)  # <QuerySet [<Author: Author object>]>
  
   # 反向，在filter中通过 表名__字段 = ''去查询另一张表的记录
   # 查找Mjj写的书
  ret = models.Book.objects.filter(author__name='Mjj') # 从书出发显然没有name属性，所以只能跨表，但是book表没有管理器，所以只能表名__name
  print(ret) # <QuerySet [<Book: 挑花侠大战菊花怪>, <Book: Mjj自传>]>
  
   # 注意：如果设置了related_name,在反向查找时就要替换author；如果设置了related_query_name,其优先级最高，且要替换之前的author
  
   # 注意，当使用到values时，再没有设置query_related_name时，默认使用表名跨表
  queryResult=Publish.objects
  　　　　　　　　　　　　　　.filter(name="苹果出版社")
  　　　　　　　　　　　　　　.values_list("book__title","book__price")   #查到出版社对象后，可通过values_list() 中使用__来取值
              
   ----------------------------------------------------------
   一对一查询
   # 注意在一对一关系中，只需要获得任意一方都可以，且直接通过对象.关联表名(小写)即可
  ret = models.Author.objects.get(pk=1)
print(ret.authordetail.addr)  # 北京沙河汇德
  
  ret = models.AuthorDetail.objects.get(pk=2)
  print(ret.author, ret.author.name) # Author object alex
  ```
  
  

#### ORM练习
- 表结构

  ```python
  # 书
  class Book(models.Model):
      title = models.CharField(max_length=32)
      publish_date = models.DateField(auto_now_add=True)
      price = models.DecimalField(max_digits=5, decimal_places=2)
      memo = models.TextField(null=True)
      # 创建外键，关联publish
      publisher = models.ForeignKey(to="Publisher")
      # 创建多对多关联author
      author = models.ManyToManyField(to="Author")
  
      def __str__(self):
          return "<Book object: {} {}>".format(self.id, self.title)
  
  
  # 出版社
  class Publisher(models.Model):
      name = models.CharField(max_length=32)
      city = models.CharField(max_length=32)
  
      def __str__(self):
          return "<Publisher object: {} {}>".format(self.id, self.name)
  
  
  # 作者
  class Author(models.Model):
      name = models.CharField(max_length=32)
      age = models.IntegerField()
      phone = models.CharField(max_length=11)
  
      def __str__(self):
          return "<Author object: {} {}>".format(self.id, self.name)
  ```



- **测试数据**

  ```python
  -- ----------------------------
  -- Records of app01_author
  -- ----------------------------
  INSERT INTO `app01_author` VALUES ('1', '金老板', '18', '15512351234');
  INSERT INTO `app01_author` VALUES ('2', '小哪吒', '20', '15312341234');
  INSERT INTO `app01_author` VALUES ('3', 'Alex', '73', '15512341234');
   
  -- ----------------------------
  -- Records of app01_publisher
  -- ----------------------------
  INSERT INTO `app01_publisher` VALUES ('1', '沙河出版社', '北京');
  INSERT INTO `app01_publisher` VALUES ('2', '西二旗出版社', '北京');
  INSERT INTO `app01_publisher` VALUES ('3', '张江出版社', '上海');
  INSERT INTO `app01_publisher` VALUES ('4', '沙河出版社', '上海');
   
  -- ----------------------------
  -- Records of app01_book
  -- ----------------------------
  INSERT INTO `app01_book` VALUES ('1', '跟金老板学开车', '2018-08-03', '12.90', null, '1');
  INSERT INTO `app01_book` VALUES ('2', '跟金老板学开潜艇', '2017-08-10', '9.99', null, '1');
  INSERT INTO `app01_book` VALUES ('3', '跟老男孩学思想', '2018-09-03', '39.99', null, '2');
  INSERT INTO `app01_book` VALUES ('4', '跟egon学喊麦', '2018-06-12', '0.99', null, '4');
   
  -- ----------------------------
  -- Records of app01_book_author
  -- ----------------------------
  INSERT INTO `app01_book_author` VALUES ('3', '1', '1');
  INSERT INTO `app01_book_author` VALUES ('4', '1', '2');
  INSERT INTO `app01_book_author` VALUES ('5', '2', '1');
  INSERT INTO `app01_book_author` VALUES ('2', '2', '2');
  INSERT INTO `app01_book_author` VALUES ('6', '3', '3');
  INSERT INTO `app01_book_author` VALUES ('7', '4', '3');
  
  # 在pycharm中database选项中右键打开数据库的Open Console，在里面从上至下的生成一个个表，run@host……
  ```

- **练习题**

  ```python
  import os
  os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'mysite.settings')
  
  import django
  django.setup()
  from app import models
  
   # 查找所有书名里包含金老板的书（不涉及跨表，且符合字符串查询）
  ret = models.Book.objects.filter(title__contains='金老板')
  print(ret)  #<QuerySet [<Book: <Book object: 1 跟金老板学开车>>, <Book: <Book object: 2 跟金老板学开潜艇>>]>
  
   # 查找出版日期是2018年的书
  ret = models.Book.objects.filter(publish_date__year = '2018')
  print(ret) # <QuerySet [<Book: <Book object: 1 跟金老板学开车>>, <Book: <Book object: 3 跟老男孩学思想>>, <Book: <Book object: 4 跟egon学喊麦>>]>
  
   # 查找出版日期是2017年的书名
  ret = models.Book.objects.filter(publish_date__year='2017').values('title')
  print(ret)  #<QuerySet [<Book: <Book object: 2 跟金老板学开潜艇>>]>
  
   # 查找价格大于10元的书
  ret = models.Book.objects.filter(price__gt=10)
  print(ret)  #<QuerySet [<Book: <Book object: 1 跟金老板学开车>>, <Book: <Book object: 3 跟老男孩学思想>>]>
  
   # 查找价格大于10元的书名和价格
  ret = models.Book.objects.filter(price__gt=10).values('title','price')
  print(ret) #<QuerySet [{'title': '跟金老板学开车', 'price': Decimal('12.90')}, {'title': '跟老男孩学思想', 'price': Decimal('39.99')}]>
  
   # 查找memo字段是空的书 (注意空字符串也是空，但是isnull不能查询，所以使用Q)
  from django.db.models import Q
  ret = models.Book.objects.filter(Q(memo__isnull=True)|Q(memo=''))
  print(ret)
  

  # 查找在北京的出版社
  ret = models.Publisher.objects.filter(city='北京')
  print(ret)
  
  
   # 查找名字以沙河开头的出版社
  ret = models.Publisher.objects.filter(name__startswith='沙河')
  print(ret)
  
   # 查找“沙河出版社”出版的所有书籍  (注意出版社中有两个'沙河出版社',但是字段查时只能查一个
   # 法一：字段，正向查询
  ret = models.Book.objects.filter(publisher__name = '沙河出版社')
  print(ret)
  
   # 法二: 反向查询
  ret = models.Publisher.objects.filter(name='沙河出版社') # 多个对象，用for
  for obj in ret:
      print(obj.book_set.all())
  
   # 查找每个出版社出版价格最高的书籍价格
  ret = models.Publisher.objects.all()
  for obj in ret:
      new_obj = obj.book_set.all().order_by('-price').first()
      if new_obj:
          print(new_obj.title, new_obj.price)
          
   # 法二：
  from django.db.models import Max, Min, Avg, Sum, Count
  ret = models.Publisher.objects.annotate(max=Max('book__price')).values('max')
  print(ret)
      
   # 查找每个出版社的名字以及出的书籍数量
  ret = models.Publisher.objects.all()
  for obj in ret:
      print(obj.name, obj.book_set.all().count())
  
   # 法二
  ret = models.Publisher.objects.annotate(count=Count('book__id')).values('name','count')
  print(ret)
  
   # 查找作者名字里面带“小”字的作者
  ret = models.Author.objects.filter(name__contains='小')
  print(ret)
  
   # 查找年龄大于30岁的作者
  ret = models.Author.objects.filter(age__gt=30)
  print(ret)
  
   # 查找手机号是155开头的作者
  ret = models.Author.objects.filter(phone__startswith='155')
  print(ret)
  
   # 查找手机号是155开头的作者的姓名和年龄
  ret = models.Author.objects.filter(phone__startswith='155').values_list('name', 'age')
  print(ret)
  
  
   # 查找每个作者写的价格最高的书籍价格
  ret = models.Author.objects.all()
  for obj in ret:
      new_ret = obj.book_set.all().order_by('-price').first()
      print(obj.name, new_ret.price)
      
   # 法二
  ret = models.Author.objects.annotate(max=Max('book__price')).values('max')
  
   # 法三
  ret = models.Book.objects.values('author').annotate(max=Max('price'))
      
   # 查找每个作者的姓名以及出的书籍数量
  ret = models.Author.objects.all()
  for obj in ret:
      new_ret = obj.book_set.all().count()
      print(obj.name, new_ret)
  
  
   # 查找书名是“跟金老板学开车”的书的出版社
  ret = models.Book.objects.filter(title='跟金老板学开车').first()
  print(ret.publisher.name)
  
   # 查找书名是“跟金老板学开车”的书的出版社所在的城市
  ret = models.Book.objects.filter(title='跟金老板学开车').first()
  print(ret.publisher.city)
  
   # 查找书名是“跟金老板学开车”的书的出版社的名称
  ret = models.Book.objects.filter(title='跟金老板学开车').first()
  print(ret.publisher.name)
  
   # 查找书名是“跟金老板学开车”的书的出版社出版的其他书籍的名字和价格
  ret = models.Publisher.objects.get(book__title='跟金老板学开车')
  print(ret.book_set.exclude(title='跟金老板学开车').values('title','price'))
  
   # 法二
  ret = models.Book.objects.filter(publisher=models.Publisher.objects.filter(book__title='跟金老板学开车')).values('title','price').exclude(title='跟金老板学开车')
  
   # 查找书名是“跟金老板学开车”的书的所有作者
  ret = models.Book.objects.filter(title = '跟金老板学开车').first()
  print(ret.author.all().values('name'))
  
  
   # 查找书名是“跟金老板学开车”的书的作者的年龄
  ret = models.Book.objects.filter(title = '跟金老板学开车').first()
  print(ret.author.all().values('name','age'))
  
   # 查找书名是“跟金老板学开车”的书的作者的手机号码
  ret = models.Book.objects.filter(title = '跟金老板学开车').first()
  print(ret.author.all().values('name','age','phone'))
  
   # 查找书名是“跟金老板学开车”的书的作者们的姓名以及出版的所有书籍名称和价钱
  ret = models.Book.objects.get(title = '跟金老板学开车').author.all()
  for obj in ret:
      print(obj.book_set.all().values('price','title'))
      
   # 法二
  ret = models.Book.objects.filter(title='跟金老板学开车').values('author__name','author__book__title','author__book__price')
   #等价于: ret = models.Book.objects.filter(title='跟金老板学开车').values('author__name','title','price')
  print(ret)
  
   # 法三
  models.Author.objects.values('name','book__title','book__price').filter(book__title='跟金老板学开车')
  ```
  
  

- **总结**：

  ```
  基于对象的跨表查询
  
      正向查询：关联属性在A表中，所以A对象找关联B表数据，正向查询
      反向查询：关联属性在A表中，所以B对象找A对象，反向查询
      
      一对多：  
             按字段：xx
      book  ------------------ >  publish
            <--------------------
            按表名小写__字段名。比如publish__name
      
  
      多对多：
            正 按字段：xx
      book  ------------------------- >  author
            <-------------------------
            反 按表名小写__字段名
            
      一对一 
              正 按字段：.ad
      author  ------------------------- >  authordetail
            <-------------------------
               反 按表名小写  authordetail_obj.author
  ```

  