## Django2 的介绍



#### 环境：

​	Django2.0 ，主要是想加深Django的认识，也是对之前的知识进行一次补充和复习



#### 项目启动

- path函数的演示

  ```python
  from django.urls import path
  from django.http import HttpResponse
  from book import views
  
  def index(request):
      return HttpResponse('首页')
  
  urlpatterns = [
      # 注意^$的用法可能不是之前1.1.1的版本了
      path('admin/', admin.site.urls),
      path(r'index/',index),
      
      # 匹配参数,book_id 与视图函数里面的第二个参数一致
      path('book/detail/<book_id>',views.book_detail)
      #path('publish/<int:publish_id>',views.publish)
      
  ]
  ```

- 当我我们使用Pycharm时，想创建多个app时，结合终端来创建

  ```shell
  1、终端：python3 manange.py startapp app2
  2、settings.py 文件中：添加 INSTALLED_APPS = []
  ```

- debug模式： 在settings.py文件中如果设置了DEBUG=True，默认是开启了调试模式，但是线上是要设置为False的，且要更改 ALLOW_HOST=['*']



#### URL映射

- 视图一般写在app的views.py中，并且视图的第一个参数永远是request(HttpRequest对象)，这个对象存储了请求过来的所有信息，Flask的request是使用了请求上下文的模式，只要导入request，就会触发全局的偏函数，去Local() 中根据线程id取出对象的request。而Django中的request是想加工厂一样通过wsgif、中间件的加工得来的

  ```python
  from django.http import HttpResponse
  def book_list(request):
  	return HttpResponse('列表：')
  ```

  ​		Django默认加载的URL地址是：setting.py文件中的 ROOT_URLCONF = 'Djano.urls' 配置了url根目录，在urls.py文件中所有的映射都应该写在 'urlpatterns' 变量中，所有的映射不是随便写的，而是使用的 'path' 或者 're_path' 函数进行包装，只不过re_path添加了正则表达式规则



#### Django 内置的转换器

- 我们通过下面的代码，可以打开coverters.py文件，里面存放着所有的转换器

  ```python
  from django.urls import converters
  
  --------------------------------------------------
  coverters.py
  ...
  DEFAULT_CONVERTERS = {
      'int': IntConverter(),
      'path': PathConverter(),
      'slug': SlugConverter(),
      'str': StringConverter(),  # 默认是使用str转换器<python>
      'uuid': UUIDConverter(),
  }
  
  REGISTERED_CONVERTERS = {}
  ```



#### URL的urls模块化

在我们的项目中，不可能只有一个app，如果把所有的app的views中的视图都放在urls.py文件中进行映射，肯定会让代码非常乱， 因此django提供了一个方法，可以将app内部包含自己url匹配规则，而在项目urls.py中再统一包含这个app的urls。使用这个技术需要借助include函数

```python
 #项目/urls.py
from django.urls import path,include

urlpatterns = [
    path('book/',include('books.urls'),namespace='book'),
]

-----------------------------------------------------
 #项目/books/urls.py
from . import views
from django.urls import path

app_name = 'books'
urlpatterns = [
    path(r'index/',index),   
    path('book/detail/<book_id>',views.book_detail)  
]

 # 注意：
    报错：Specifying a namespace in include() without providing an app_name ，表示没有指定一个应用命名空间，即使你注册了INSTALL_APP,也需要注册app_name
```

- include函数的使用

  - include(module, namespace=None):

    ```
    1、module:子url的模板位置字符串
    2、namespace: 实例命名空间，注意，如果指定实例命名空间，那么必须先指定应用命名空间（在子urls.py中添加'app_name'这个参数
    ```

  - include((pattern_list, app_namespace),namespace=None ): 上面的应用命名空间是在子urls模板中指定，我们也可以在include函数中指定，且传入的可以是一个元组

    ```python
     #项目/urls.py
    from django.urls import path,include
    
    urlpatterns = [
        path('book/',include(('books.urls','books'),namespace='book'),
    ]
    
    -----------------------------------------------------
     #项目/books/urls.py
    from . import views
    from django.urls import path
    
    urlpatterns = [
        path(r'index/',index),   
        path('book/detail/<book_id>',views.book_detail)  
    ]
    ```

  - include(pattern_list): include中也可以进行路由映射

    ```python
    urlpatterns = [
        path('movie/',include([
        	path('',views.movie),  # 这样访问/move也可以使用
        	path('list/',views.movie_list)
        ])),
    ]
    ```

- re_path 函数： 有时候我们在写url匹配的时候，想要写使用正则表达式来实现一些复杂的需求，那么这时候我们可以使用re_path来实现

  ```python
  from . import views
  from django.urls import path, re_path
  
  urlpatterns = [ 
      path('book/detail/<book_id>',views.book_detail),
      re_path(r'articles/(?p<year>[0-9]{4})',views.year_article)
  ]
   #注意： 所有的route字符串前面都要加上一个'r',表示这个字符的原生字符串
  ```

  

#### 自定义URL(path)转换器

- 需求： 实现一个获取文章列表的demo，要求给一个url，根据url如：/articles/python+django+flask/来返回路径中3种书籍的列表，显然我们可以获得最后的字符串，通过切割来实现，那么这里我们使用自定义转换器来实现这个功能。

  ```python
  from django.urls import path,re_path
  from . import views
  from django.urls import converters,register_converter
  
  app_name = 'app'
  
  class CategoryConverter:
      regex = r'\w+|(\w+\+\w+)+'
  
      def  to_python(self,value):
          # 类型转换，通过正则拿到的字符串：value
          result = value.split('+')
          return result
  
      def to_url(self,value):
          # 当涉及到反向解析时会调用，此时的value为[]列表
          if isinstance(value,list):
              result = '+'.join(value)
              return result
          else:
              raise RuntimeError('分类参数出错')
  
  # 注册进转换器
  register_converter(CategoryConverter,'cate')
  
  urlpatterns = [
      path('index/',views.index),
      # 自定义转换器
      path('new/<cate:categories>/',views.article_list),
      # re_path使用
      re_path(r'list/(?P<categories>(\w+\+\w+)+|\w+)/',views.article_list)
  ]
  ```

  