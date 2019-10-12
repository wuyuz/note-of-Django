### 静态文件和ORM

#### 静态文件

- 当我们要在html中引用js、css、img……（静态文件时），就需要进行静态文件的规范并使用。下面我们就讲django的静态文件进行配置

- **第一步**：在setting进行修改

  ```python
  STACIC_URL = '/xxx/' #别名随便写，在下面我们拼接后路径后，这个别名就代表下面拼接的路径了，但是前端html、和外界以后都是通过这个别名来访问我们拼接的路径，这样对外就隐藏了我们静态文件的名字。
  
  STATICFILES_DIRS = [
      os.path.join(BASE_DIR,'jingtai'), #注意别忘了写逗号,第二个参数就是项目中你存放静态文件的文件夹名称（与应用同级）
  ]
  ```

  ![1558512331577](C:\Users\wanglixing\AppData\Roaming\Typora\typora-user-images\1558512331577.png)

- **第二步**：具体的使用，在static.html文件中（**第一种方式**）

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <link rel="stylesheet" href="/static/css/1.css">   //引用css文件
  {# <link rel="stylesheet" href="/jintai/css/1.css"> 失效#}
  
  </head>
  <body>
  <div class="c1">
      <p >如果引用了别名，通过jintai引入css，反而不行（失效）</p>
  </div>
  <script src="/static/js/1.js">   //引用js文件
  
  </script>
  </body>
  </html>
  ```

  配置URL和视图函数后：

  ![1558512352291](C:\Users\wanglixing\AppData\Roaming\Typora\typora-user-images\1558512352291.png)

  

- **第二种引入方式**：上面，我们通过在setting中设置别名，将所有静态文件路径与之拼接。其实，之前拼接路径的列表可以存储多条路径，这样我们可以实现，讲多个静态文件夹整合在一个别名下，然后通过具体的下一级目录进行拼接查找，即可找到相应的静态文件。下面我们将使用：

  **{% load static %}** 来动态加载别名（常用），也就是说即使你的配置的static的名字随便变化了，此方法会自动找别名

  ```html
  {% load static %}
  <img src="{% static "/images/hi.jpg" %}" alt="Hi!" />  //引入img
  
  #引用js文件
  {% load static %}
  <script src="{% static "/mytest.js" %}"></script>
  
  #某个文件多处被用到可以存为一个变量，使用as取别名
  {% load static %}
  {% static "/images/hi.jpg" as myphoto %}
  <img src="{{ myphoto }}"></img>
  ```

  修改static.html文件：注意{% load static %} 必须在引用之前就可以

  ```html
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      {% load static %}
      <link rel="stylesheet" href="{% static "/css/1.css" %}">
  </head>
  ```

- **方式三**: 使用{% get_static_prefix %} 来获取别名前缀，再拼接路径，和第二种方法类似

  ```html
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      {% load static %}
      <link rel="stylesheet" href="{% get_static_prefix%}css/1.css">  //这主意中间没有空格
  </head>
  
  //或则取别名
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      {% load static %}
      {% get_static_prefix as STATIC_PREFIX %}
      <link rel="stylesheet" href="{{ STATIC_PREFIX  }}css/1.css">
  </head>
  ```




##### 多个静态文件的配置和查找顺序

- 当我们需要多个静态文件时，需要注意统一的别名只能时static，这个别名代表了所有 你的已经写在列表里（配置）的文件名，相当于给你的静态文件装进一个黑箱中，用的时候django给你取。

- 静态文件是有执行顺序的，依次查找黑箱中的静态文件，如果我用到一个图片名，多个静态文件都有，此时就会触发最前一个静态文件里的此图片。

  ![1560306877214](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1560306877214.png)



------



#### ORM （Object relation mapping)

- **ORM**，全称是object relation mapping。翻译过来，就是对象关系映射。

  要知道：再程序开发一个领域可以分为：应用开发人员/数据库管理人员；前者通过编程语言实现应用，而后者是通过sql语句进行数据库操作，那么要求一个应用开发人员去写sql语句，就会显得很吃力，但是我们可以写程序啊，那么ORM就是一个映射数据库的一个模型。

  ```
  官方：MVC框架中包括一个重要的部分，就是ORM，它实现了数据模型与数据库的解耦，即数据模型的设计不需要依赖于特定的数据库，通过简单的配置就可以轻松更换数据库，这极大的减轻了开发人员的工作量，不需要面对因数据库变更而导致的无效劳动
  
  ORM是“对象-关系-映射”的简称。（Object Relational Mapping，简称ORM）(将来会学一个sqlalchemy，是和他很像的，但是django的orm没有独立出来让别人去使用，虽然功能比sqlalchemy更强大，但是别人用不了)
  ```

- **ORM的作用**：

  ```
  ORM可以实现不写sql语句，就可以实现操作数据库。
  
  通过pymysql，就是Mysql给Python提供的接口。早期的接口叫mysqldb （python3.4后就没有者模块了）
  ```

- **ORM的核心**：用python类操作表，用对象操作记录。

  ```
  ORM把相应的类和属性操作，转换为sql语句，来操作数据库。它做了一个翻译的过程！学习ORM，就是用什么语法，能翻译成什么样的sql语句。
  ```
  

![1558517844026](C:\Users\wanglixing\Desktop\知识点复习\Django\静态文件和ORM.assets\1558517844026.png)

- ##### ORM的优点

  ```
  1.符合python语法。
  
  2.自己写的sql语句，效率不够高。因为它直接传参，就可以使用了。
  
  3. 不需要自己写SQL，对于类的操作，会转换成相应的SQL语句，来操作数据库。(核心功能)
  
  4.它不属于硬编码，它不针对于MySQL。比如公司发展了，需要换成oracle数据库。（可以对多种数据库进行操作）
  	如果项目中的sql语句写死了（或者数据库迁移），那么项目中的所有的sql语句，都得更换。如果用了ORM，只需要改一处，
  直接修改setting.py中的数据库引擎，换成oracle，就可以了。这样，方便数据库迭代，这是它最大的优点。
  ```

- ##### ORM的缺点

  ```
  1.执行效率低。看上图，它是从上层到下层的操作。它有一个翻译的过程。所以不如直接写SQL（原生sql），操作的执行效率低。
  但是影响不大。真遇到这种请情况，优化SQL就可以了。效率问题，跟SQL有很大的关系。
  2、不用担心：ORM还提供了，执行原生SQL的接口。可以直接执行原生SQL。
  ```

- **建议**：刚开始写ORM时，一定要对应SQL语句，去写ORM

  ![1558518548625](C:\Users\wanglixing\Desktop\知识点复习\Django\静态文件和ORM.assets\1558518548625.png)

  从上图中可以知道，class的类属性和字段是对应的。 [decimal(10,2)表示最大长度为10位。小数点2位。那么整数部分，最大只能有8位。]

  ```
              ORM引擎
  python -------   --------> sql
  
  类名                        表名
  属性变量                    字段
  属性值                      约束条件
  
  对象                        一条记录
  
  比如save()，就可以增加一条记录。所以是对象对应一条记录。
  ```

- 名言警句：

  ```
  早期，会经历很多碰壁的过程，这样你就能成长！
  先创建表，分模块来创建表。
  到了写代码的时候，反而是最简单的。
  比如一个商城项目，设计表，梳理流程，花了2个月。写代码，几个星期就完成了。
  ```

  

##### 具体操作

- **第一步**：配置环境：Pycharm的Mysql环境 + setting的数据库配置

  - Pycharm的Mysql环境 ：

    ![1558519351421](C:\Users\wanglixing\Desktop\知识点复习\Django\静态文件和ORM.assets\1558519351421.png)

  - 之后点击Mysql，进入输入连接的库、密码、用户名……（就可以连接了）

    ![1558519461138](C:\Users\wanglixing\Desktop\知识点复习\Django\静态文件和ORM.assets\1558519461138.png)

    

  - 设置好Pycharm后，修改setting文件的databases，默认django使用的是sqlite3数据库，我们现在要设置为Mysql数据库。

    ```python
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',   # 数据库引擎mysql
            'NAME': 'db1',       # 你要存储数据的库名，事先要创建之
            'USER': 'root',      # 数据库用户名
            'PASSWORD': '123',      # 密码
            'HOST': 'localhost', # 主机
            'PORT': '3306',      # 数据库使用的端口
        }
    }
    ```

  - 安装pymsql模块：需要安装两个模块

    **首先是注册app**：修改mysite下的settings.py文件里面的**`INSTALLED_APPS`**,添加自己的项目名字到里面，否则数据库不知道给哪个项目添加表。

    ```python
     #django连接MySQL，使用的是pymysql模块，必须得安装2个模块。否则后面会创建表不成功！或者提示no module named MySQLdb 
    pip3 install pymysql
    pip3 install mysqlclient
    ```

  - 在应用目录下的\__init__.py文件中添加以下代码（我们这里是在应用 app文件下，设置的）

    ```python
    import pymysql
    pymysql.install_as_MySQLdb()
    
    #启动项目，会报错：no module named MySQLdb 。这是因为django默认你导入的驱动是MySQLdb，可是MySQLdb 对于py3有很大问题，所以我们需要的驱动是PyMySQL。（也就是收到设置驱动式mysql）
    ```

- **第二步**：在应用下找到主角---models.py，在里面创建模型

  ```python
  from django.db import models
  
  class Book(models.Model):  #注销此类，在makemigrations，再提交就会删除ci'biao
       id=models.AutoField(primary_key=True) # 如果表里面没有写主键，表里面会自动生成一个自增主键字段，叫做id字段，orm要求每个表里面必须要写一个主键
       title=models.CharField(max_length=32)  # 和varchar(32)是一样的，32个字符
       state=models.BooleanField()
       pub_date=models.DateField()     # 必须存这种格式"2018-12-12"
       price=models.DecimalField(max_digits=8,decimal_places=2) # max_digits最大位数，decimal_places小数部分占多少位
       publish=models.CharField(max_length=32)
  #ORM没法创建数据库，它只能操作表！  
  ```
  
- 接下来要在pycharm的teminal中通过命令创建数据库的表了。有2条命令，分别是：
  
  ```python
    > python manage.py makemigrations   #生成记录，每次修改了models里面的内容或者添加了新的app，新的app里面写了models里面的内容，都要执行这两条,这将在应用migrations文件下生成：0001_initial.py文件，这里面就是记录你本次操作。
    
    > python manage.py migrate     #执行上面这个语句的记录来创建表，生成的表名字前面会自带应用的名字，例如：你的book表在mysql里面叫做app01_book表,提交执行数据库创建
  ```
  
- 关于同步指令的执行简单原理:
  
  ```
       在执行python manager.py makemigrations 时django 会在相应的 app 的migration文件夹下面生成 一个python脚本文件，在执行 python manager.py migrate 时 django才会生成数据库表，那么django是如何生成数据库表的呢？
       django是根据 migration下面的脚本文件来生成数据表的
       每个migration文件夹下面有多个脚本，那么django是如何知道该执行那个文件的呢，django有一张django-migrations表，表中记录了，已经执行的脚本，那么表中没有的就是还没执行的脚本，则执行migrate的时候就只执行表中没有记录的那些脚本。
        1、有时在执行 migrate 的时候如果发现没有生成相应的表，可以看看在 django-migrations表中看看 脚本是否已经执行了，
        2、可以删除 django-migrations 表中的记录 和 数据库中相应的 表 ， 然后重新 执行
  ```
  
- 通过pycharm提供的功能来执行manage.py相关的指令：
  
  ![1558522964544](C:\Users\wanglixing\Desktop\知识点复习\Django\静态文件和ORM.assets\1558522964544.png)
  
  在这个终端里，你就可以不用输入：python manage.py -opt了，直接输入：makemigrations/migrate就行了
  
  
  
- **注意:** 如果想打印orm转换过程中的sql，需要在settings中进行如下配置：
  
  ```python
    #在setting中添加这一段，就会让所有的提交数据库时，出现sql语句
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
  
- 还有一种可以查看执行时的sql语句
  
  ```python
    from app01 import models
    
    def add_book(request):
        book_obj = models.Book(title='python',price=123,pub_date='2012-12-12',publish='人民出版社')
        book_obj.save()
        from django.db import connection  #通过这种方式也能查看执行的sql语句
        print(connection.queries)
        return HttpResponse('ok')
  ```
  
  ![1558689016927](C:\Users\wanglixing\Desktop\知识点复习\Django\静态文件和ORM.assets\1558689016927.png)
  

  
- ##### 更多字段和参数

  - 每个字段有一些特有的参数，例如，CharField需要max_length参数来指定`VARCHAR`数据库字段的大小。还有一些适用于所有字段的通用参数。 这些参数在文档中有详细定义，这里我们只简单介绍一些最常用的。

  - ##### 常用字段

    ```
    <1> CharField
            字符串字段, 用于较短的字符串.
            CharField 要求必须有一个参数 max_length, 用于从数据库层和Django校验层限制该字段所允许的最大字符数.
     
    <2> IntegerField
           #用于保存一个整数.范围是-2147483648 ~ 2147483648
     
    <3> FloatField
            一个浮点数. 必须 提供两个参数:
             
            参数    描述
            max_digits    总位数(不包括小数点和符号)
            decimal_places    小数位数
                    举例来说, 要保存最大值为 999 (小数点后保存2位),你要这样定义字段:
                     
                    models.FloatField(..., max_digits=5, decimal_places=2)
                    要保存最大值一百万(小数点后保存10位)的话,你要这样定义:
                     
                    models.FloatField(..., max_digits=17, decimal_places=10) #max_digits大于等于17就能存储百万以上的数
                    
                    admin 用一个文本框(<input type="text">)表示该字段保存的数据.
     
    <4> AutoField
            一个 IntegerField, 添加记录时它会自动增长. 你通常不需要直接使用这个字段;
            自定义一个主键：my_id=models.AutoField(primary_key=True)
            如果你不指定主键的话,系统会自动添加一个主键字段到你的 model，也就是id.
            一个models类中不能有两个AutoField
     
    <5> BooleanField
            A true/false field. admin 用 checkbox 来表示此类字段.
     
    <6> TextField
            一个容量很大的文本字段.
            admin 用一个 <textarea> (文本区域)表示该字段数据.(一个多行编辑框).
     
    <7> EmailField
            一个带有检查Email合法性的 CharField,不接受 maxlength 参数.
      
    <8> DateField
            一个日期字段. 共有下列额外的可选参数:
            Argument    描述
            auto_now    当对象被保存时,自动将该字段的值设置为当前时间.通常用于表示 "last-modified" 时间戳.
            auto_now_add    当对象首次被创建时,自动将该字段的值设置为当前时间.通常用于表示对象创建时间.都是设置为True（仅仅在admin中有意义...)
     
    <9> DateTimeField
             一个日期时间字段. 类似 DateField 支持同样的附加选项.它是继承DateField，也就是说也可以使用auto_now、auto_now_add
     
    <10> ImageField
            类似 FileField, 不过要校验上传对象是否是一个合法图片.#它有两个可选参数:height_field和width_field,
            如果提供这两个参数,则图片将按提供的高度和宽度规格保存.  
            
    <11> FileField
         一个文件上传字段.
         要求一个必须有的参数: upload_to, 一个用于保存上载文件的本地文件系统路径. 这个路径必须包含 strftime #formatting,
         该格式将被上载文件的 date/time
         替换(so that uploaded files don't fill up the given directory).
         admin 用一个<input type="file">部件表示该字段保存的数据(一个文件上传部件) .
     
         注意：在一个 model 中使用 FileField 或 ImageField 需要以下步骤:
                （1）在你的 settings 文件中, 定义一个完整路径给 MEDIA_ROOT 以便让 Django在此处保存上传文件.
                (出于性能考虑,这些文件并不保存到数据库.) 定义MEDIA_URL 作为该目录的公共 URL. 要确保该目录对
                 WEB服务器用户帐号是可写的.
                （2） 在你的 model 中添加 FileField 或 ImageField, 并确保定义了 upload_to 选项,以告诉 Django
                 使用 MEDIA_ROOT 的哪个子目录保存上传文件.你的数据库中要保存的只是文件的路径(相对于 MEDIA_ROOT).
                 出于习惯你一定很想使用 Django 提供的 get_<#fieldname>_url 函数.举例来说,如果你的 ImageField
                 叫作 mug_shot, 你就可以在模板中以 {{ object.#get_mug_shot_url }} 这样的方式得到图像的绝对路径.
     
    <12> URLField
          用于保存 URL. 若 verify_exists 参数为 True (默认), 给定的 URL 会预先检查是否存在( 即URL是否被有效装入且
          没有返回404响应).
          admin 用一个 <input type="text"> 文本框表示该字段保存的数据(一个单行编辑框)
     
    <13> NullBooleanField
           类似 BooleanField, 不过允许 NULL 作为其中一个选项. 推荐使用这个字段而不要用 BooleanField 加 null=True 选项
           admin 用一个选择框 <select> (三个可选择的值: "Unknown", "Yes" 和 "No" ) 来表示这种字段数据.
     
    <14> SlugField
           "Slug" 是一个报纸术语. slug 是某个东西的小小标记(短签), 只包含字母,数字,下划线和连字符.#它们通常用于URLs
           若你使用 Django 开发版本,你可以指定 maxlength. 若 maxlength 未指定, Django 会使用默认长度: 50.  #在
           以前的 Django 版本,没有任何办法改变50 这个长度.
           这暗示了 db_index=True.
           它接受一个额外的参数: prepopulate_from, which is a list of fields from which to auto-#populate
           the slug, via JavaScript,in the object's admin form: models.SlugField
           (prepopulate_from=("pre_name", "name"))prepopulate_from 不接受 DateTimeFields.
     
    <13> XMLField
            一个校验值是否为合法XML的 TextField,必须提供参数: schema_path, 它是一个用来校验文本的 RelaxNG schema #的文件系统路径.
     
    <14> FilePathField
            可选项目为某个特定目录下的文件名. 支持三个特殊的参数, 其中第一个是必须提供的.
            参数    描述
            path    必需参数. 一个目录的绝对文件系统路径. FilePathField 据此得到可选项目.
            Example: "/home/images".
            match    可选参数. 一个正则表达式, 作为一个字符串, FilePathField 将使用它过滤文件名. 
            注意这个正则表达式只会应用到 base filename 而不是
            路径全名. Example: "foo.*\.txt^", 将匹配文件 foo23.txt 却不匹配 bar.txt 或 foo23.gif.
            recursive可选参数.要么 True 要么 False. 默认值是 False. 是否包括 path 下面的全部子目录.
            这三个参数可以同时使用.
            match 仅应用于 base filename, 而不是路径全名. 那么,这个例子:
            FilePathField(path="/home/images", match="foo.*", recursive=True)
            ...会匹配 /home/images/foo.gif 而不匹配 /home/images/foo/bar.gif
     
    <15> IPAddressField
            一个字符串形式的 IP 地址, (i.e. "24.124.1.30").
    <16> CommaSeparatedIntegerField
            用于存放逗号分隔的整数值. 类似 CharField, 必须要有maxlength参数.
    ```
  
  
  
- ##### 参数说明
  
  ```
    (1)null  如果为True，Django 将用NULL 来在数据库中存储空值。 默认值是 False.
     
    (1)blank    如果为True，该字段允许不填(admin管理表单中，表单允许不填)。默认为False。
    	要注意，这与 null 不同。null纯粹是数据库范畴的，而 blank 是数据验证范畴的。
    如果一个字段的blank=True，表单的验证将允许该字段是空值。如果字段的blank=False，该字段就是必填的。
     
    (2)default
    	字段的默认值。可以是一个值或者可调用对象。如果可调用 ，每有新对象被创建它都会被调用，如果你的字段没有设置可以为空，那么将来如果我们后添加一个字段，这个字段就要给一个default值
     
    (3)primary_key
    	如果为True，那么这个字段就是模型的主键。如果你没有指定任何一个字段的primary_key=True，
    Django 就会自动添加一个IntegerField字段做为主键，所以除非你想覆盖默认的主键行为，
    否则没必要设置任何一个字段的primary_key=True。
     
    (4)unique   如果该值设置为 True, 这个数据字段的值在整张表中必须是唯一的
     
    (5)choices   由二元组组成的一个可迭代对象（例如，列表或元组），用来给字段提供选择项。 如果设置了choices ，默认的表单将是一个选择框而不是标准的文本框，<br>而且这个选择框的选项就是choices 中的选项。
    
    (6)db_index
    　　如果db_index=True 则代表着为此字段设置数据库索引。DatetimeField、DateField、TimeField这个三个时间字段，都可以设置如下属性。
    
    (7)auto_now_add
        配置auto_now_add=True，创建数据记录的时候会把当前时间添加到数据库。
    
    (8)auto_now
        配置上auto_now=True，每次更新数据记录的时候会更新该字段，标识这条记录最后一次的修改时间。
        
    (9)  db_column           数据库中字段的列名
    (10) unique_for_date     数据库中字段【日期】部分是否可以建立唯一索引
    (11) unique_for_month    数据库中字段【月】部分是否可以建立唯一索引
  ```
  
- ##### 自定义一个char类型字段
  
  ```python
    #自定义有char类型
    class MyCharField(models.Field):
        def __init__(self, max_length, *args, **kwargs):
            self.max_length = max_length
            super(MyCharField, self).__init__(max_length=max_length, *args, **kwargs)
     
        def db_type(self, connection):
            """
            限定生成数据库表的字段类型为char，长度为max_length指定的值
            """
            return 'char(%s)' % self.max_length  # 返回的是自己写的名字char(25),这里可以也可以返回return 'varchar(%s)'%self.max_length
        
    #使用自定义char类型在字段
    class Class(models.Model):
        id = models.AutoField(primary_key=True)
        title = models.CharField(max_length=25)
        # 使用自定义的char类型的字段
        cname = MyCharField(max_length=25)
  ```
  
  ![1560342583487](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1560342583487.png)
  
  
  
- **Model Meta参数**：
  
  ```python
    class UserInfo(models.Model):
        nid = models.AutoField(primary_key=True)
        username = models.CharField(max_length=32)
     
        class Meta:
            # 数据库中生成的表名称 默认 app名称 + 下划线 + 类名
            db_table = "table_name"
     
            # admin中显示的表名称
            verbose_name = '个人信息'
     
            # verbose_name加s
            verbose_name_plural = '所有用户信息'
     
            # 联合索引 
            index_together = [
                ("pub_date", "deadline"),   # 应为两个存在的字段
            ]
     
            # 联合唯一索引
            unique_together = (("driver", "restaurant"),)   # 应为两个存在的字段
            
     #补充：配置元信息的重要性？当我们在别的项目中需要使用到我们之前项目的数据库时，知道了数据库密码、接口等还不都，在models中的类默认在执行迁移的时候会生成的项目名_表名，显然和我们数据库中的要使用的表名不一致，会自动创建新表，此时，我们就需要使用元信息了，让我们的表名和数据库中的一致即可。（不需要迁移，就可以使用）       
  ```
  
  
  
- **使用choices参数**：该参数接收一个可迭代的列表或元组（基本单位为二元组）。如果指定了该参数，在实例化该模型时，该字段只能取选项列表中的值。且需要注意的是这个可选列表的第一个值会被显示，要想显示第二个值，需要使用 `get_FOO_display()`
  
  ```python
     #models.py文件
    from django.db import models
    class Person(models.Model):
        SHIRT_SIZES = (
            ('S', 'Small'),
            ('M', 'Medium'),
            ('L', 'Large'),
        )
        name = models.CharField(max_length=60)
        shirt_size = models.CharField(max_length=1, choices=SHIRT_SIZES)
    
        --------------------------------------------
    >>> p = Person(name="Fred Flintstone", shirt_size="L")
    >>> p.save()
    >>> p.shirt_size
    'L'
    >>> p.get_shirt_size_display()
    'Large'    
  ```
  
- **备注名**：(verbose_name:常用于admin中显示字段)
  
  - 除了 [`ForeignKey`](https://docs.djangoproject.com/zh-hans/2.1/ref/models/fields/#django.db.models.ForeignKey) ， [`ManyToManyField`](https://docs.djangoproject.com/zh-hans/2.1/ref/models/fields/#django.db.models.ManyToManyField) 和 [`OneToOneField`](https://docs.djangoproject.com/zh-hans/2.1/ref/models/fields/#django.db.models.OneToOneField) ，任何字段类型都接收一个可选的参数 [`verbose_name`](https://docs.djangoproject.com/zh-hans/2.1/ref/models/fields/#django.db.models.Field.verbose_name) ，如果未指定该参数值， Django 会自动使用该字段的属性名作为该参数值，并且把下划线转换为空格。
  
    ```
    first_name = models.CharField("person's first name", max_length=30)  #在该例中，备注名为 "person's first name",这个备注名也就是之前每个字段配置的'verbose_name'，可以查看Field的源码的init,默认在第一个位置上传参
      
    first_name = models.CharField(max_length=30) # 在该例中：备注名为 "first name"
    ```
  
  - [`ForeignKey`](https://docs.djangoproject.com/zh-hans/2.1/ref/models/fields/#django.db.models.ForeignKey), [`ManyToManyField`](https://docs.djangoproject.com/zh-hans/2.1/ref/models/fields/#django.db.models.ManyToManyField) and [`OneToOneField`](https://docs.djangoproject.com/zh-hans/2.1/ref/models/fields/#django.db.models.OneToOneField) 接收的第一个参数为模型的类名，后面可以添加一个 [`verbose_name`](https://docs.djangoproject.com/zh-hans/2.1/ref/models/fields/#django.db.models.Field.verbose_name) 参数：
  
    ```python
      poll = models.ForeignKey(
          Poll,
          on_delete=models.CASCADE,
          verbose_name="the related poll",
      )
      sites = models.ManyToManyField(Site, verbose_name="list of sites")
      place = models.OneToOneField(
          Place,
          on_delete=models.CASCADE,
          verbose_name="related place",
      )
    ```
  
  
  
- ##### 多表关系和参数
  
  ```python
    ForeignKey(ForeignObject) # ForeignObject(RelatedField)
        to,                 # 要进行关联的表名
        to_field=None,      # 要关联的表中的字段名称
        on_delete=None,     # 当删除关联表中的数据时，当前表与其关联的行的行为
                            - models.CASCADE，删除关联数据，与之关联也删除
                            - models.DO_NOTHING，删除关联数据，引发错误IntegrityError
                            - models.PROTECT，删除关联数据，引发错误ProtectedError
                            - models.SET_NULL，删除关联数据，与之关联的值设置为null（前提FK字段需要设置为可空）
                            - models.SET_DEFAULT，删除关联数据，与之关联的值设置为默认值（前提FK字段需要设置默认值）
                            - models.SET，删除关联数据，
                                   a. 与之关联的值设置为指定值，设置：models.SET(值)
                                   b. 与之关联的值设置为可执行对象的返回值，设置：models.SET(可执行对象)
     
                                        def func():
                                            return 10
     
                                        class MyModel(models.Model):
                                            user = models.ForeignKey(
                                                to="User",
                                                to_field="id"
                                                on_delete=models.SET(func),)
        related_name=None,          # 反向操作时，使用的字段名，用于代替 【表名_set】 如： obj.表名_set.all()
        related_query_name=None,    # 反向操作时，使用的连接前缀，用于替换【表名】     如： models.UserGroup.objects.filter(表名__字段名=1).values('表名__字段名')
        limit_choices_to=None,      # 在Admin或ModelForm中显示关联数据时，提供的条件：
                                    # 如：
                            - limit_choices_to={'nid__gt': 5}
                            - limit_choices_to=lambda : {'nid__gt': 5}
     
                            from django.db.models import Q
                            - limit_choices_to=Q(nid__gt=10)
                            - limit_choices_to=Q(nid=8) | Q(nid__gt=10)
                            - limit_choices_to=lambda : Q(Q(nid=8) | Q(nid__gt=10)) & Q(caption='root')
        db_constraint=True          # 是否在数据库中创建外键约束
        parent_link=False           # 在Admin中是否显示关联数据
     
     
    OneToOneField(ForeignKey)
        to,                 # 要进行关联的表名
        to_field=None       # 要关联的表中的字段名称
        on_delete=None,     # 当删除关联表中的数据时，当前表与其关联的行的行为
     
                            ###### 对于一对一 ######
                            # 1. 一对一其实就是 一对多 + 唯一索引
                            # 2.当两个类之间有继承关系时，默认会创建一个一对一字段
                            # 如下会在A表中额外增加一个c_ptr_id列且唯一：
                                    class C(models.Model):
                                        nid = models.AutoField(primary_key=True)
                                        part = models.CharField(max_length=12)
     
                                    class A(C):
                                        id = models.AutoField(primary_key=True)
                                        code = models.CharField(max_length=1)
     
    ManyToManyField(RelatedField)
        to,                         # 要进行关联的表名
        related_name=None,          # 反向操作时，使用的字段名，用于代替 【表名_set】 如： obj.表名_set.all()
        related_query_name=None,    # 反向操作时，使用的连接前缀，用于替换【表名】     如： models.UserGroup.objects.filter(表名__字段名=1).values('表名__字段名')
        limit_choices_to=None,      # 在Admin或ModelForm中显示关联数据时，提供的条件：
                                    # 如：
                                        - limit_choices_to={'nid__gt': 5}
                                        - limit_choices_to=lambda : {'nid__gt': 5}
     
                                        from django.db.models import Q
                                        - limit_choices_to=Q(nid__gt=10)
                                        - limit_choices_to=Q(nid=8) | Q(nid__gt=10)
                                        - limit_choices_to=lambda : Q(Q(nid=8) | Q(nid__gt=10)) & Q(caption='root')
        symmetrical=None,           # 仅用于多对多自关联时，symmetrical用于指定内部是否创建反向操作的字段
                                    # 做如下操作时，不同的symmetrical会有不同的可选字段
                                        models.BB.objects.filter(...)
     
                                        # 可选字段有：code, id, m1
                                            class BB(models.Model):
     
                                            code = models.CharField(max_length=12)
                                            m1 = models.ManyToManyField('self',symmetrical=True)
     
                                        # 可选字段有: bb, code, id, m1
                                            class BB(models.Model):
     
                                            code = models.CharField(max_length=12)
                                            m1 = models.ManyToManyField('self',symmetrical=False)
     
        through=None,               # 自定义第三张表时，使用字段用于指定关系表
        through_fields=None,        # 自定义第三张表时，使用字段用于指定关系表中那些字段做多对多关系表
                                        from django.db import models
     
                                        class Person(models.Model):
                                            name = models.CharField(max_length=50)
     
                                        class Group(models.Model):
                                            name = models.CharField(max_length=128)
                                            members = models.ManyToManyField(
                                                Person,
                                                through='Membership',
                                                through_fields=('group', 'person'),
                                            )
     
                                        class Membership(models.Model):
                                            group = models.ForeignKey(Group, on_delete=models.CASCADE)
                                            person = models.ForeignKey(Person, on_delete=models.CASCADE)
                                            inviter = models.ForeignKey(
                                                Person,
                                                on_delete=models.CASCADE,
                                                related_name="membership_invites",
                                            )
                                            invite_reason = models.CharField(max_length=64)
        db_constraint=True,         # 是否在数据库中创建外键约束
        db_table=None,              # 默认创建第三张表时，数据库中表的名称
  ```
  
  
  
- ##### auto_new 你需要知道的事
  
  ```python
    #当需要更新时间的时候，我们尽量通过datetime模块来创建当前时间，并保存或者更新到数据库里面，看下面的分析：
    假如我们的表结构是这样的
    
    class User(models.Model):
        username = models.CharField(max_length=255, unique=True, verbose_name='用户名')
        is_active = models.BooleanField(default=False, verbose_name='激活状态')
    
    #那么我们修改用户名和状态可以使用如下两种方法：
    
    #方法一：update() 批量修改表记录
    	User.objects.filter(id=1).update(username='nick',is_active=True)
    
    #方法二：原始办法，相对慢，因为会重新赋值该记录每个字段值  
        _t = User.objects.get(id=1)
        _t.username='nick'
        _t.is_active=True
        _t.save()
    
    #方法一适合更新一批数据，类似于mysql语句update user set username='nick' where id = 1 
    #方法二适合更新一条数据，也只能更新一条数据，当只有一条数据更新时推荐使用此方法，另外此方法还有一个好处，我们接着往下看
    
    具有auto_now属性字段的更新
    我们通常会给表添加三个默认字段 
    - 自增ID，这个django已经默认加了，就像上边的建表语句，虽然只写了username和is_active两个字段，但表建好后也会有一个默认的自增id字段 
    - 创建时间，用来标识这条记录的创建时间，具有auto_now_add属性，创建记录时会自动填充当前时间到此字段 
    - 修改时间，用来标识这条记录最后一次的修改时间，具有auto_now属性，当记录发生变化时填充当前时间到此字段
    
    就像下边这样的表结构：
    class User(models.Model):
        create_time = models.DateTimeField(auto_now_add=True, verbose_name='创建时间')
        update_time = models.DateTimeField(auto_now=True, verbose_name='更新时间')
        username = models.CharField(max_length=255, unique=True, verbose_name='用户名')
        is_active = models.BooleanField(default=False, verbose_name='激活状态')
    
    当表有字段具有auto_now属性且你希望他能自动更新时，必须使用上边方法二的更新，不然auto_now字段不会更新，也就是：
    
    _t = User.objects.get(id=1)
    _t.username='nick'
    _t.is_active=True
    _t.save()
    
    json/dict类型数据更新字段
    目前主流的web开放方式都讲究前后端分离，分离之后前后端交互的数据格式大都用通用的jason型，那么如何用最少的代码方便的更新json格式数据到数据库呢？同样可以使用如下两种方法：
    
    方法一：
        data = {'username':'nick','is_active':'0'}
        User.objects.filter(id=1).update(**data)
    
    同样这种方法不能自动更新具有auto_now属性字段的值
    通常我们再变量前加一个星号(*)表示这个变量是元组/列表，加两个星号表示这个参数是字典
    方法二：
        data = {'username':'nick','is_active':'0'}
        _t = User.objects.get(id=1)
        _t.__dict__.update(**data)
        _t.save()
    
    方法二和方法一同样无法自动更新auto_now字段的值
    注意这里使用到了一个__dict__方法
    方法三：
        _t = User.objects.get(id=1)
        _t.role=Role.objects.get(id=3)
        _t.save()
  ```
  
- ##### 附ORM与数据库（sql）实际字段对应关系

  ```
  	'AutoField': 	  				'integer AUTO_INCREMENT',
      'BigAutoField':					'bigint AUTO_INCREMENT',
      'BinaryField': 					'longblob',
      'BooleanField': 				'bool',
      'CharField': 					'varchar(%(max_length)s)',
      'CommaSeparatedIntegerField': 	'varchar(%(max_length)s)',
      'DateField': 					'date',
      'DateTimeField': 				'datetime',
      'DecimalField': 				'numeric(%(max_digits)s, %(decimal_places)s)',
      'DurationField': 				'bigint',
      'FileField': 					'varchar(%(max_length)s)',
      'FilePathField': 				'varchar(%(max_length)s)',
      'FloatField': 					'double precision',
      'IntegerField': 				'integer',
      'BigIntegerField':				'bigint',
      'IPAddressField': 				'char(15)',
      'GenericIPAddressField': 		'char(39)',
      'NullBooleanField': 			'bool',
      'OneToOneField': 				'integer',
      'PositiveIntegerField': 		'integer UNSIGNED',
      'PositiveSmallIntegerField': 	'smallint UNSIGNED',
      'SlugField': 					'varchar(%(max_length)s)',
      'SmallIntegerField': 			'smallint',
      'TextField': 					'longtext',
      'TimeField': 					'time',
      'UUIDField': 					'char(32)',
  ```

- ##### 注意

  ```
  我们通过与mysql建立连接，操作mysql数据时，其实是创建了一个临时对话框，就比如我们通过Pycharm来操作数据库（migrate操作等时，会打印一个警告：
  ```

  ![1558595235078](C:\Users\wanglixing\Desktop\知识点复习\Django\静态文件和ORM.assets\1558595235078.png)

  这涉及到mysql的sql_mode设置的相关知识：[详细解释](https://www.cnblogs.com/clschao/articles/9962347.html)

  ```
   sql_mode是个很容易被忽视的变量,默认值是空值,在这种设置下是可以允许一些非法操作的,比如允许一些非法数据的插入。在生产环境必须将这个值设置为严格模式,所以开发、测试环境的数据库也必须要设置,这样在开发测试阶段就可以发现问题.
   
   sql model 常用来解决下面几类问题：
  　　(1) 通过设置sql mode, 可以完成不同严格程度的数据校验，有效地保障数据准备性。
  
  　　(2) 通过设置sql model 为宽松模式，来保证大多数sql符合标准的sql语法，这样应用在不同数据库之间进行迁移时，则不需要对业务sql 进行较大的修改。
  
  　　(3) 在不同数据库之间进行数据迁移之前，通过设置SQL Mode 可以使MySQL 上的数据更方便地迁移到目标数据库中。
  　　
  解决模式设置和修改(以解决上述问题为例)：
  	方式一：先执行select @@sql_mode,复制查询出来的值并将其中的NO_ZERO_IN_DATE,NO_ZERO_DATE删除，然后执行set sql_mode = '修改后的值'或者set session sql_mode='修改后的值';，例如：set session sql_mode='STRICT_TRANS_TABLES';改为严格模式
          此方法只在当前会话中生效，关闭当前会话就不生效了。
  
      方式二：先执行select @@global.sql_mode,复制查询出来的值并将其中的NO_ZERO_IN_DATE,NO_ZERO_DATE删除，然后执行set global sql_mode = '修改后的值'。
          此方法在当前服务中生效，重新MySQL服务后失效
  
   
      方法三：在mysql的安装目录下，或my.cnf文件(windows系统是my.ini文件)，新增 sql_mode = ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION，
          添加my.cnf如下：
          [mysqld]
          sql_mode=ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER
          然后重启mysql。从方法永久有效
  ```

  修改前：

  ![1558595895208](C:\Users\wanglixing\Desktop\知识点复习\Django\静态文件和ORM.assets\1558595895208.png)

  

  修改后：

  ![1558595860794](C:\Users\wanglixing\Desktop\知识点复习\Django\静态文件和ORM.assets\1558595860794.png)

  
  - 在Django中也可以通过设置数据库配置来完成解决上述问题：setting中

    ```python
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': 'mxshop',
            'HOST': '127.0.0.1',
            'PORT': '3306',
            'USER': 'root',
            'PASSWORD': '123',
            'OPTIONS': {  #设置严格模式
                "init_command": "SET default_storage_engine='INNODB'",
    　　　　　　　#'init_command': "SET sql_mode='STRICT_TRANS_TABLES'",
    
            }
        }
    }
    DATABASES['default']['OPTIONS']['init_command'] = "SET sql_mode='STRICT_TRANS_TABLES'"
    ```




#### ORM 操作

- **基本操作**

  ```python
  # 增
  models.Tb1.objects.create(c1='xx', c2='oo')   # 增加一条数据，可以接受字典类型数据 **kwargs
  
  obj = models.Tb1(c1='xx', c2='oo')  #法二
  obj.save()
   
   
  # 查
  models.Tb1.objects.get(id=123)  # 获取单条数据，不存在则报错（不建议）
  models.Tb1.objects.all()  # 获取全部
  models.Tb1.objects.filter(name='seven')  # 获取指定条件的数据
  models.Tb1.objects.exclude(name='seven')  # 去除指定条件的数据
   
   
  # 删
  # models.Tb1.objects.filter(name='seven').delete()  # 删除指定条件的数据
   
   
  # 改
  models.Tb1.objects.filter(name='seven').update(gender='0')   # 将指定条件的数据更新，均支持 **kwargs
  
  obj = models.Tb1.objects.get(id=1)
  obj.c1 = '111'
  obj.save()   # 修改单条数据
  
  
  --------------------------------------进阶------------------------------------
  # 获取个数
  models.Tb1.objects.filter(name='seven').count()
  
  # 大于，小于
  models.Tb1.objects.filter(id__gt=1)               # 获取id大于1的值
  models.Tb1.objects.filter(id__gte=1)              # 获取id大于等于1的值
  models.Tb1.objects.filter(id__lt=10)              # 获取id小于10的值
  models.Tb1.objects.filter(id__lte=10)             # 获取id小于10的值
  models.Tb1.objects.filter(id__lt=10, id__gt=1)    # 获取id大于1 且 小于10的值
  
  # 成员判断in
  models.Tb1.objects.filter(id__in=[11, 22, 33])   # 获取id等于11、22、33的数据
  models.Tb1.objects.exclude(id__in=[11, 22, 33])  # not in
  
  # 是否为空 isnull
  models.Tb1.objects.filter(pub_date__isnull=True)
  
  # 包括contains
  models.Tb1.objects.filter(name__contains="ven")
  models.Tb1.objects.filter(name__icontains="ven") # icontains大小写不敏感
  models.Tb1.objects.exclude(name__icontains="ven")
  
  # 范围range
  models.Tb1.objects.filter(id__range=[1, 2])   # 范围bettwen and
  
  # 其他类似
  startswith，istartswith, endswith, iendswith,
  
  # 排序order by
  models.Tb1.objects.filter(name='seven').order_by('id')    # asc
  models.Tb1.objects.filter(name='seven').order_by('-id')   # desc
  
  # 分组group by
  from django.db.models import Count, Min, Max, Sum
  models.Tb1.objects.filter(c1=1).values('id').annotate(c=Count('num'))
  #SELECT "app01_tb1"."id", COUNT("app01_tb1"."num") AS "c" FROM "app01_tb1" WHERE "app01_tb1"."c1" = 1 GROUP BY "app01_tb1"."id"   筛选出c1=1的按照id进行分组，最后求出有多少个'num'
  
  # limit 、offset
  models.Tb1.objects.all()[10:20]
  
  # regex正则匹配，iregex 不区分大小写
  models.Tb1.objects.get(title__regex=r'^(An?|The) +')
  models.Tb1.objects.get(title__iregex=r'^(an?|the) +')
  
  # date
  models.Tb1.objects.filter(pub_date__date=datetime.date(2005, 1, 1))
  models.Tb1.objects.filter(pub_date__date__gt=datetime.date(2005, 1, 1))
  
  # year
  models.Tb1objects.filter(pub_date__year=2005)
  models.Tb1objects.filter(pub_date__year__gte=2005)
  
  # month
  models.Tb1.objects.filter(pub_date__month=12)
  models.Tb1.filter(pub_date__month__gte=6)
  
  # day
  models.Tb1.objects.filter(pub_date__day=3)
  models.Tb1.objects.filter(pub_date__day__gte=3)
  
  # week_day
  models.Tb1.objects.filter(pub_date__week_day=2)
  models.Tb1.objects.filter(pub_date__week_day__gte=2)
  
  # hour
  models.Tb1.objects.filter(timestamp__hour=23)
  models.Tb1.objects.filter(time__hour=5)
  models.Tb1.objects.filter(timestamp__hour__gte=12)
  
  # minute
  models.Tb1.objects.filter(timestamp__minute=29)
  models.Tb1.objects.filter(time__minute=46)
  models.Tb1.objects.filter(timestamp__minute__gte=29)
  
  # second
  models.Tb1.objects.filter(timestamp__second=31)
  models.Tb1.objects.filter(time__second=2)
  models.Tb1.objects.filter(timestamp__second__gte=31)
  ```

- ##### 高级操作

  ```python
  # extra
  # 在QuerySet的基础上继续执行子语句：
  extra(self, select=None, where=None, params=None, tables=None, order_by=None, select_params=None)
  
  # select和select_params是一组，where和params是一组，tables用来设置from哪个表
  Entry.objects.extra(select={'new_id': "select col from sometable where othercol > %s"}, select_params=(1,))
  Entry.objects.extra(where=['headline=%s'], params=['Lennon'])
  Entry.objects.extra(where=["foo='a' OR bar = 'a'", "baz = 'a'"])
  Entry.objects.extra(select={'new_id': "select id from tb where id > %s"}, select_params=(1,), order_by=['-nid'])
  
  #举个例子：
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
  
  
  # 执行原生SQL
  # 更高灵活度的方式执行原生SQL语句
  # from django.db import connection, connections
  # cursor = connection.cursor()  # cursor = connections['default'].cursor()
  # cursor.execute("""SELECT * from auth_user where id = %s""", [1])
  # row = cursor.fetchone()
  ```

- QuerySet相关方法

  ```python
  ##################################################################
  # PUBLIC METHODS THAT ALTER ATTRIBUTES AND RETURN A NEW QUERYSET #
  ##################################################################
  
  def all(self)
      # 获取所有的数据对象
  
  def filter(self, *args, **kwargs)
      # 条件查询
      # 条件可以是：参数，字典，Q
  
  def exclude(self, *args, **kwargs)
      # 条件查询
      # 条件可以是：参数，字典，Q
  
  def select_related(self, *fields)
      性能相关：表之间进行join连表操作，一次性获取关联的数据。
  
      总结：
      1. select_related主要针一对一和多对一关系进行优化。
      2. select_related使用SQL的JOIN语句进行优化，通过减少SQL查询的次数来进行优化、提高性能。
  
  def prefetch_related(self, *lookups)
      性能相关：多表连表操作时速度会慢，使用其执行多次SQL查询在Python代码中实现连表操作。
  
      总结：
      1. 对于多对多字段（ManyToManyField）和一对多字段，可以使用prefetch_related()来进行优化。
      2. prefetch_related()的优化方式是分别查询每个表，然后用Python处理他们之间的关系。
  
  def annotate(self, *args, **kwargs)
      # 用于实现聚合group by查询
  
      from django.db.models import Count, Avg, Max, Min, Sum
  
      v = models.UserInfo.objects.values('u_id').annotate(uid=Count('u_id'))
      # SELECT u_id, COUNT(ui) AS `uid` FROM UserInfo GROUP BY u_id
  
      v = models.UserInfo.objects.values('u_id').annotate(uid=Count('u_id')).filter(uid__gt=1)
      # SELECT u_id, COUNT(ui_id) AS `uid` FROM UserInfo GROUP BY u_id having count(u_id) > 1
  
      v = models.UserInfo.objects.values('u_id').annotate(uid=Count('u_id',distinct=True)).filter(uid__gt=1)
      # SELECT u_id, COUNT( DISTINCT ui_id) AS `uid` FROM UserInfo GROUP BY u_id having count(u_id) > 1
  
  def distinct(self, *field_names)
      # 用于distinct去重
      models.UserInfo.objects.values('nid').distinct()
      # select distinct nid from userinfo
  
      注：只有在PostgreSQL中才能使用distinct进行去重
  
  def order_by(self, *field_names)
      # 用于排序
      models.UserInfo.objects.all().order_by('-id','age')
  
  def extra(self, select=None, where=None, params=None, tables=None, order_by=None, select_params=None)
      # 构造额外的查询条件或者映射，如：子查询
  
      Entry.objects.extra(select={'new_id': "select col from sometable where othercol > %s"}, select_params=(1,))
      Entry.objects.extra(where=['headline=%s'], params=['Lennon'])
      Entry.objects.extra(where=["foo='a' OR bar = 'a'", "baz = 'a'"])
      Entry.objects.extra(select={'new_id': "select id from tb where id > %s"}, select_params=(1,), order_by=['-nid'])
  
   def reverse(self):
      # 倒序
      models.UserInfo.objects.all().order_by('-nid').reverse()
      # 注：如果存在order_by，reverse则是倒序，如果多个排序则一一倒序
  
  
   def defer(self, *fields):
      models.UserInfo.objects.defer('username','id')
      或
      models.UserInfo.objects.filter(...).defer('username','id')
      #映射中排除某列数据
  
   def only(self, *fields):
      #仅取某个表中的数据
       models.UserInfo.objects.only('username','id')
       或
       models.UserInfo.objects.filter(...).only('username','id')
  
   def using(self, alias):
       指定使用的数据库，参数为别名（setting中的设置）
  
  
  ##################################################
  # PUBLIC METHODS THAT RETURN A QUERYSET SUBCLASS #
  ##################################################
  
  def raw(self, raw_query, params=None, translations=None, using=None):
      # 执行原生SQL
      models.UserInfo.objects.raw('select * from userinfo')
  
      # 如果SQL是其他表时，必须将名字设置为当前UserInfo对象的主键列名
      models.UserInfo.objects.raw('select id as nid from 其他表')
  
      # 为原生SQL设置参数
      models.UserInfo.objects.raw('select id as nid from userinfo where nid>%s', params=[12,])
  
      # 将获取的到列名转换为指定列名
      name_map = {'first': 'first_name', 'last': 'last_name', 'bd': 'birth_date', 'pk': 'id'}
      Person.objects.raw('SELECT * FROM some_other_table', translations=name_map)
  
      # 指定数据库
      models.UserInfo.objects.raw('select * from userinfo', using="default")
  
      ################### 原生SQL ###################
      from django.db import connection, connections
      cursor = connection.cursor()  # cursor = connections['default'].cursor()
      cursor.execute("""SELECT * from auth_user where id = %s""", [1])
      row = cursor.fetchone() # fetchall()/fetchmany(..)
  
  
  def values(self, *fields):
      # 获取每行数据为字典格式
  
  def values_list(self, *fields, **kwargs):
      # 获取每行数据为元祖
  
  def dates(self, field_name, kind, order='ASC'):
      # 根据时间进行某一部分进行去重查找并截取指定内容
      # kind只能是："year"（年）, "month"（年-月）, "day"（年-月-日）
      # order只能是："ASC"  "DESC"
      # 并获取转换后的时间
          - year : 年-01-01
          - month: 年-月-01
          - day  : 年-月-日
  
      models.DatePlus.objects.dates('ctime','day','DESC')
  
  def datetimes(self, field_name, kind, order='ASC', tzinfo=None):
      # 根据时间进行某一部分进行去重查找并截取指定内容，将时间转换为指定时区时间
      # kind只能是 "year", "month", "day", "hour", "minute", "second"
      # order只能是："ASC"  "DESC"
      # tzinfo时区对象
      models.DDD.objects.datetimes('ctime','hour',tzinfo=pytz.UTC)
      models.DDD.objects.datetimes('ctime','hour',tzinfo=pytz.timezone('Asia/Shanghai'))
  
      """
      pip3 install pytz
      import pytz
      pytz.all_timezones
      pytz.timezone(‘Asia/Shanghai’)
      """
  
  def none(self):
      # 空QuerySet对象
  
  
  ####################################
  # METHODS THAT DO DATABASE QUERIES #
  ####################################
  
  def aggregate(self, *args, **kwargs):
     # 聚合函数，获取字典类型聚合结果
     from django.db.models import Count, Avg, Max, Min, Sum
     result = models.UserInfo.objects.aggregate(k=Count('u_id', distinct=True), n=Count('nid'))
     ===> {'k': 3, 'n': 4}
  
  def count(self):
     # 获取个数
  
  def get(self, *args, **kwargs):
     # 获取单个对象
  
  def create(self, **kwargs):
     # 创建对象
  
  def bulk_create(self, objs, batch_size=None):
      # 批量插入
      # batch_size表示一次插入的个数
      objs = [
          models.DDD(name='r11'),
          models.DDD(name='r22')
      ]
      models.DDD.objects.bulk_create(objs, 10)
  
  def get_or_create(self, defaults=None, **kwargs):
      # 如果存在，则获取，否则，创建
      # defaults 指定创建时，其他字段的值
      obj, created = models.UserInfo.objects.get_or_create(username='root1', defaults={'email': '1111111','u_id': 2, 't_id': 2})
  
  def update_or_create(self, defaults=None, **kwargs):
      # 如果存在，则更新，否则，创建
      # defaults 指定创建时或更新时的其他字段
      obj, created = models.UserInfo.objects.update_or_create(username='root1', defaults={'email': '1111111','u_id': 2, 't_id': 1})
  
  def first(self):
     # 获取第一个
  
  def last(self):
     # 获取最后一个
  
  def in_bulk(self, id_list=None):
     # 根据主键ID进行查找
     id_list = [11,21,31]
     models.DDD.objects.in_bulk(id_list)
  
  def delete(self):
     # 删除
  
  def update(self, **kwargs):
      # 更新
  
  def exists(self):
     # 是否有结果
  
  ```

  