### jinja2 和 Django初步

#### 一、Django的安装

- 以python解释器安装：

  ```python
  pip3 install django==1.11.9
  ```

- 以anaconda命令行安装:(这里我们下载1.11.9版本)

  ```
  pip install django==1.11.9  
  ```

- 卸载django

  ```
  pip uninstall django
  ```

  ![1558058933630](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558058933630.png)

- 如果安装时，弹出网址给出最新安装源，我们可以配置：在安装了anaconda后，我们也可以使用anaconda来进行Python库的安装，同样的也需要进行源的配置。（其实使用pip，anaconda来进行Python库的安装都是差不多，不过个人比较喜欢用anaconda）这个配置方法就很简单了，你只需要在配置了anaconda的pycharm中的终端（Terminal）逐条输入以下两条命令即可：

  ![1558059272582](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558059272582.png)

  ```
  conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
  conda config --set show_channel_urls yes
  ```

  也可以再pycharm中进行安装，也可以配置安装源：

  ![1558059430791](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558059430791.png)

  

#### 二、多版本Python和Anaconda

- 在Pycharm中装载django的时候，总是无法创建，原因是Pyhton3.7版本在Pycharm中不兼容，所以我选择降低Anaconda的版本（我使用的是Anaconda的解释Python解释器）。Anaconda是支持选择版本的（在降低版本操作后，Anaconda的目录下会有一个env的文件，里面有降低后的版本。但是总感觉很麻烦

- 最后我在Python官网上去重新下载了3.6的版本，然后在环境变量中，配置的 python3（重命名了，这就可以完全规避多版本的冲突）

  ![1558077278792](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558077278792.png)



#### 三、模块渲染jinja2

###### jinja2介绍和简单示例

​	在上一章中我们为了上网页动态起来(也就是让固定的html文件，在不同时候表现出不同的内容，不受局限)，我们使用了字符串的方法`replace` ，显得很麻烦，且麻烦，这里我们就开始学习新的模块**`jinja2`**来时刻动态返回html文件。 [官方网址](http://docs.jinkan.org/docs/jinja2/intro.html#api)

```
安装：pip install jinja2
```

​		一个简单的动态页面(字符串替换)，我完全可以从数据库中查询数据，然后去替换我html中的对应内容（专业名词叫做模板渲染，你先渲染一下，再给浏览器进行渲染），然后再发送给浏览器完成渲染。 这个过程就相当于HTML模板渲染数据。 本质上就是HTML内容中利用一些特殊的符号来替换要展示的数据。 我这里用的特殊符号是我定义的，其实模板渲染有个现成的工具： `jinja2`　(注意，其实Django中有自己的渲染模块，和jinja2一样，但是不是jinja2)

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="x-ua-compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Title</title>
</head>
<body>
    <h1>姓名：{{name}}</h1>
    <h1>爱好：</h1>
    <ul>
        {% for hobby in hobby_list %}
        <li>{{hobby}}</li>
        {% endfor %}
    </ul>
</body>
</html>
```

使用jinja2渲染上面的文件，创建一个pythonn文件，代码如下

```python
from wsgiref.simple_server import make_server
from jinja2 import Template

def index():
    with open("index2.html", "r",encoding='utf-8') as f:
        data = f.read()
    template = Template(data)  # 生成模板文件
    ret = template.render({"name": "于谦", "hobby_list": ["烫头", "泡吧"]})  # 把数据填充到模板里面
    return [bytes(ret, encoding="utf8"), ]  #注意jinja2返回的还是个字符串，需要转化


# 定义一个url和函数的对应关系
URL_LIST = [
    ("/index/", index),
]

def run_server(environ, start_response):
    print(start_response)  #<bound method BaseHandler.start_response of <wsgiref.simple_server.ServerHandler object at 0x0000015DFFA3E278>>
    print(environ)   # 一些配置信息，是对所有的本地信息和请求信息的整合
    start_response('200 OK', [('Content-Type', 'text/html;charset=utf8'), ])  # 设置HTTP响应的状态码和头信息,注意不设置Contenn-Type编码格式，就会出现编码错误在浏览器中
    url = environ['PATH_INFO']  # 取到用户输入的url
    func = None  # 将要执行的函数
    for i in URL_LIST:
        if i[0] == url:
            func = i[1]  # 去之前定义好的url列表里找url应该执行的函数
            break
    if func:  # 如果能找到要执行的函数
        return func()  # 返回函数的执行结果
    else:
        return [bytes("404没有该页面", encoding="utf8"), ]

if __name__ == '__main__':
    httpd = make_server('', 8000, run_server)  # 在调用在这个函数时，会默认给他environ参数
    print("Serving HTTP on port 8000...")
    httpd.serve_forever()
```



#### 四、jinja2详细的使用方法

- 简单传参，使页面动态

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
      <p>你的名字是什么{{name}}</p>  //使用{{ }} 传递
      <p>A value from a list: {{ mylist[3] }}.</p>  //数字索引
      <p>A value from a dictionary: {{ mydict['key'] }}.</p>  //字典
  </body>
  </html>
  ```

- 服务器文件

  ```python
  from jinja2 import Template
  from wsgiref.simple_server import  make_server
  
  def index():
      with open('2.html','r',encoding='utf-8') as f:
          data = f.read()
      templte = Template(data)  #得到一堆字符串，再进行实例化出templte
      ret = templte.render({'name':'王立','mylist':[1,2,3,4],'mydict':{'key':'ok'}})   # 这里使用render加字典传递
      return [bytes(ret,encoding='utf-8')]
  
  URL=[   # 路由
      ('/',index)
  ]
  
  def app(environ,start_respinse):
      start_respinse("200 OK",[('Content-Type','text/html;charset=utf8'),])   # 消息头部，专门用来设置消息头的函数
      path = environ['PATH_INFO']
      func =None
      for url in URL:
          if url[0]==path:
              func = url[1]
              break
      if func: return func()
      else: return [bytes('404没有该页面',encoding='utf-8'),]
      
  httped = make_server('',8080,app)
  print('Serving HTTP on port 8080……')
  httped.serve_forever()
  ```

  

##### 变量

- ​    在模板中使用的 `{{ name }}` 结构表示一个变量，它是一种特殊的占位符，告诉模板引擎这个位置的值从渲染模板时使用的数据中获取。Jinja2 能识别所有类型的变量，甚至是一些复杂的类型，例如列表、字典和对象。在模板 中使用变量的一些示例如下:

  ```html
  <p>A value from a dictionary: {{ mydict['key'] }}.</p>  //字典
  <p>A value from a list: {{ mylist[3] }}.</p>  //数字索引
  <p>A value from a list, with a variable index: {{ mylist[myintvar] }}.</p>  //变量索引
  <p>A value from an object's method: {{ myobj.somemethod() }}.</p> //对象
  ```

- ​      可以使用过滤器修改变量，过滤器名添加在变量名之后，中间使用竖线分隔。例如，下述 模板以首字母大写形式显示变量 name 的值:

  ```html
  Hello, {{ name|capitalize }}   //使用将变量名首字母大写
  ```

  ![1558081209914](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558081209914.png)



##### jinja2的控制结构

- if-else的用法

  ```html
  {% if name %}
      Hello,{{name}}
  {% else %}
       Hello ,你没名字一看就是菜啊!!!
  {% endif %}     //直接用就是了
  ```

- for循环

  ```html
      {% for title in ["黄金4","黄金3","黄金2"] %}  //也可以传一个变量过来
          <li>{{title}}</li>
      {% endfor %}
  ```

- jinja2中的宏，类似于函数：

  ```html
  {% macro render_comment(comment) %} 
         <li>{{ comment }}</li>
  {% endmacro %}   //定义一个宏，用于返回结构体，下次直接使用render_comment就可以了
  <ul>
        {% for comment in comments %}
        {{ render_comment(comment) }} 
        {% endfor %}
  </ul>
  ```




#### 五、Django起第一个项目

- **第一步**创建第一个自己的项目

  ```python
  语法：
  django-admin startproject +项目名字  #前提在script文件中找到django-admin，设置环境变量
  django-admin startproject mysite   
  ```

- **第二步**你已经有了自己的项目文件夹了，如下

  ![1558084800168](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558084800168.png)

  ```
  manage.py ----- Django项目里面的工具，通过它可以调用django shell和数据库，启动关闭项目与				项目交互等，不管你将框架分了几个文件，必然有一个启动文件，其实他们本身就是一个				  文件。
  settings.py ---- 包含了项目的默认设置，包括数据库信息，调试标志以及其他一些工作的变量。
  urls.py ----- 负责把URL模式映射到应用程序。URL配置(URLconf)就像Django 所支撑网站的目录。				它的本质是URL与要为该URL调用的视图函数之间的映射表；你就是以这种方式告诉					  Django，对于客户端发来的某个URL调用哪一段逻辑代码对应执行。
  wsgi.py ---- runserver命令就使用wsgiref模块做简单的web server，后面会看到renserver命				令，所有与socket相关的内容都在这个文件里面了，目前不需要关注它。
  ```

  　   你会发现，上面没有什么view视图函数的文件啊，这里我们说一个应用与项目的关系，上面我们只是创建了一个项目，并没有创建应用，以微信来举例，微信是不是一个大的项目，但是微信里面是不是有很多个应用，支付应用、聊天应用、朋友圈、小程序等这些在一定程度上都是相互独立的应用，也就是说一个大的项目里面可以有多个应用。所以下一步，我们要创建一个我们的应用逻辑。

  

- 在mysite目录下（都是和大项目重名）创建应用

  ```python
  python manage.py startapp blog   #通过执行manage.py文件来创建应用，执行这句话一定要注意，你应该						在这个manage.py的文件所在目录下执行这句话，因为其他目录里面没有这个文件
  python manage.py startapp blog2  #每个应用都有自己的目录，每个应用的目录下都有自己的views.py视图						函数和models.py数据库操作相关的文件
  ```

  ![1558085033704](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558085033704.png)

  ```
  我们现在只需要看其中两个文件
  　　　　models.py ：之前我们写的那个名为model的文件就是创建表用的，这个文件就是存放与该app(应用)相关					的表结构的
  　　　　views.py  ：存放与该app相关的视图函数的
  ```

- **启动django项目**：

  ```python
  python manage.py runserver 8080   # python manage.py runserver 127.0.0.1:8080，本机就不用写ip地址了 如果连端口都没写，默认是本机的8000端口
  ```

  ![1558085141115](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558085141115.png)





##### 基于Django实现一个简单动态的返回一个网页

![1558082953801](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558082953801.png)

- 第一步，浏览器请求进入服务器，被wsgi.py进行整理接收，返回一个request对象（源码中的WSGIRequest返回一个request对象，总之在wsgi.py中进行了很大的整合，我们只需要返回使用就是了。
- 第二步，在urls.py（总路由中）导入django.conf.urls的url，用来做路由分发和分组;  导入django.contrib的admin的admin.site.urls,这应该是用来控制管理平台的路由。
- 第三步，通过urls的分派，进入views.py中去查找函数，让函数在model，中去操作数据库，返回数据，整合到template中的html文件中，在最后将最终的数据返回wsgi.py中将消息返回个浏览器。

**简单应用**：现在实现一个用户输入一个timer路径，返回一个含有当前时间的页面，想想怎么做？用户输入网址-->路径-->函数-->返回数据(文件)

- 1、urls.py控制器中（第一步就找它）

  ```python
  from django.conf.urls import url
  from django.contrib import admin
  
  #找对应的函数，是哪个app里面的函数
  from app01 import views
  
  urlpatterns = [
      path('admin/', admin.site.urls), #这个先不用管，后面会学
      path('index/',views.index),
  ]
  ```

- 2、views.py 视图

  ```python
  from django.shortcuts import render,HttpResponse
  
  def index(request):
      import datetime
      now = datetime.datetime.now()
      ctime = now.strftime("%Y-%m-%d %X")
      return render(request,'index.html',{"ctime":ctime})  # 类似于jinja2中的模块渲染，使模块动态起来
  ```
  
- 3、template模块中的index.html

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
  
  <h4>当前时间:{{ ctime }}</h4>
  </body>
  </html>
  ```

  ![1558084278800](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558084278800.png)

- 启动服务，可以通过终端，也可以通过pycharm中的运行按钮

  ```
  cmd: python mamage.py runserver 8080     #默认是本机回环地址
  ```

  

##### 需要注意的一点：

- 在settings配置文件里面将这一行注释掉，这是django给你加的一个csrf的认证，现在不需要，它用于通过post提交时，的一些安全防护。

  ```python
  MIDDLEWARE = [
      'django.middleware.security.SecurityMiddleware',
      'django.contrib.sessions.middleware.SessionMiddleware',
      'django.middleware.common.CommonMiddleware',
      # 'django.middleware.csrf.CsrfViewMiddleware',
      'django.contrib.auth.middleware.AuthenticationMiddleware',
      'django.contrib.messages.middleware.MessageMiddleware',
      'django.middleware.clickjacking.XFrameOptionsMiddleware',
  ]
  #否则在post后会报错
  ```

  ​		还记得django写视图函数的时候，有一个参数是必须要给的吗，叫做request，如果你是post请求，那么就用request.POST，就能拿到post请求提交过来的所有数据（一个字典，然后再通过字典取值request.POST.get('username')，取出来的就是个字符串，你在那个字典里面看到的是{'username':['chao']}，虽然看着是列表，但是request.POST.get('username')取出来的就是个字符串），通过request.GET就能拿到提交过来的所有数据，而且记着，每一个视图函数都要给人家返回一些内容，用render或者HttpResponse等，其实render里面也是通过HttpResponse来返回内容，不然会报错，错误是告诉你没有返回任何内容：

  ![1558085509082](C:\Users\RootUser\AppData\Roaming\Typora\typora-user-images\1558085509082.png)

  

##### 静态文件配置

- 网页数据的动态性是在后端完成的，通过render实现，因为浏览器不认识{{}},只加载css,js……，

  所以我们需要设置settings.py

  ```python
  STATIC_URL = '/static/'
  
  STATICFILES_DIRS = (
      os.path.join(BASE_DIR,"static"),
  )
  ```

  **STATIC_URL和STATICFILES_DIRS 这2个参数，缺一不可。django必须用别名，代指物理路径。也就是STATIC_URL 代指STATICFILES_DIRS 路径, 不能直接用物理路径引用**，比如html引用jquery，代码如下：

  ```html
  <script src="../static/jquery.js"></script>  //这是可以的但是不推荐
  <script src="/static/jquery.js"></script>
  
  //访问statics里面的静态文件，可以直接访问的。它在django源码里面定义好了
  所以urls.py里面不需要定义statics，就可以直接访问
  ```

  

- **起Django项目步骤总结**：

  ```python
  1、创建一个Django项目名：django-admin startproject mysite  
  
  2、进入项目中，创建一个app：python manage.py startapp blog
  	2.1、在setting.py中的INSTALLED_APP添加app的路	  	径：'app1.apps.App1Config'
  
  3、配置静态文件：
  	STATIC_URL = '/static/'
  	STATICFILES_DIRS=[   # 静态文件
      os.path.join(BASE_DIR,'jintai'), ]
      
  4、修正数据库：
      DATABASES = {
          'default': {
              'ENGINE': 'django.db.backends.mysql',
              'NAME': 'db2',
              'HOST': '127.0.0.1',
              'PORT': '3306',
              'USER': 'root',
              'PASSWORD': '123',
              'OPTIONS': {
                  "init_command": "SET default_storage_engine='INNODB'",
              }
          }
      }
      
      4.1在与Django项目同名的目录下的__init__.py文件中写如下代码，告诉Django使用pymysql模块连接MySQL数据库:
          import pymysql
  		pymysql.install_as_MySQLdb()
      
   5、分发路由： 
      from django.conf.urls import url,include
      urlpatterns = [
          url('^index/', include('app1.url')),
      ]
      #在自己的app中新建url.py，写自己的路径，剔除父级路径部分
      
  ```




##### 如何更新models文件，以及清洗迁移文件

```
1、在每个应用app下的migrations文件下会及时记录：makemigrations命令后的models操作，等待migrate命令的触发
2、在数据库中有一个迁移文件记录表：django_migrations,它会记录所有执行migrate后的文件。
3、如果想及时更改models文件结构，需要删除数据库中原有的表，如现在数据库中有book表、出版社表，此时我更新了book表，那么我需要删除book表和django_migrations中最后一次生成book的记录。

#注意：一般情况下，migrations和migrate联合执行是很少报错的。
```



##### 通过在数据库中修改以及更新字段，怎么作用到models文件中

```
1、在pycharm中的database中添加字段，然后进行填充数据，命令：右键 modify Table 修改 excute
2、此时在views中是不能使用到我们新添加的字段的，因为在models中的没有对应的字段：
	addr = models.CharField(max_length = 12)
	注意:这里是不能执行makemigrations了，但是在本地migrations文件中和刚刚修改的models又存在差异，此时我们可以修改initial文件
在fields中添加新的addr元组，这样就不会存在差异了。

#还可以通过数据库使用manage.py自动生成数据库
```

