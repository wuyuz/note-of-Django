## Django2 内置方法



#### 限制请求装饰器

Django内置的视图装饰器可以给视图提供一些限制，比如这个视图只能通过GET的method访问等，以下将介绍一些常用的内置视图装饰器

- django.views.decorators.http.require_http_methods: 这个装饰器需要传递一个允许访问的方法泪飙，比如只能通过GET方式的访问：

  ```python
  from django.shortcuts import render,HttpResponse
  from django.views.decorators.http import require_http_methods,require_GET
  ,request_POST
  @require_http_methods(['GET'])  # 也就是说我们可以不用使用request.method == 'GET'了，只针对get的函数
  def index(request):
      return HttpResponse('ok')
  
  # 上面类似写法
  @require_GET
  def index(request):
      return HttpResponse('ok')
  
  --------------------------------
  @require_http_methods(['POST'])   # 只能以post访问，不用if判断了
  def add_stu(request):
      return HttpResponse('ok')
  
  ---------------------------------
  @require_http_methods(['POST','GET'])
  def add_stu(request):
      if request.method='GET':
          return 'OK'
      return HttpResponse('ok')
  ```



#### 重定向

- 永久重定向：http状态码301，多用于旧网址迁移

- 暂时重定向：http状态码302，多用于暂时跳转，如登陆注册

- Django中的重定向 redirect(to,*args permanent=False,**kwargs) 来实现

  ```
  to: url跳转
  permanent: 默认为False,为True的话表示永久重定向
  ```

  

#### WSGIRequest对象

Django在接受http请求之后，会根据http请求携带的参数以及报文信息创建一个WSGIRequest对象，并且视为视图函数第一个参数传给视图函数，也就是我们看到的reqeust参数：django.core.handlers.wsgi.WSGIRequest

- 对应的属性：

  - path：完整路径，不包括域名/xx/yy/

  - method: 代表当前http方法

  - GET：返回一个 django.http.request.QueryDict对象，里面存储了url参数?name=wang

  - POST:返回一个 django.http.request.QueryDict对象,返回一个post方式提交的参数（满足：urlencode/form-data编码，content-type:json)

  - FILES：返回一个 django.http.request.QueryDict对象,返回所有文件

  - COOKIES: 获取标准的字典

  - session: 一个标准字典，用来操作服务器session

  - META：存储客户端发送上的所有header信息

    ```
    content_length: 请求正文长度
    content_type:请求正文的mime类型
    http_accept: 响应可接受的编码
    http_host:客户端发送的host值
    http_referer: 在访问这一个页面前访问的url
    remote_addr: 客户端的ip地址，如果服务器用的ngix反向代理，显示的为127.0.0.1，这个时候可以使用http_x_forwarded_for来获取
    
    	if request.META.has_key('HTTP_X_FORWARDED_FOR'):
    		ip = request.META("HTTP_X_FORWARDED_FOR")
    	else:
    		ip = request.META('HOST_ADDR')
    ```

- 常用方法：

  - is_secure:是否采用https协议
  - get_host(): 获取服务器域名
  - get_full_path(): 获取全路径



#### QueryDict对象

我们平时用的request.GET 和 require.POST 都是QueryDict对象，这个对象继承dict

- get()：返回一个字典中的值
- getlist():  根据键返回多个值



#### HttpResponse 对象

Django服务器收到请求后，会将提交上来的数据进行封装成一个HttpRequst对象给视图，视图处理逻辑后，也需要返回一个响应给浏览器，这个响应就是HttpResponse对象

- content ：返回内容

- status_code: 返回的状态码

- content_type: 默认时text/html(默认html文件)

  ```
  1、text/plain  纯文本
  2、text/css、text/js、
  3、multipart/form-data 文件
  4、application/json  json文件
  ```

- 设置相应头： response['xxx'] ='xxx'



#### JsonResponse类

用来对象dump成json字符串，然后返回json字符串封装成Response对象返回给浏览器

```python
from django.http import JsonResponse

def index(request):
	persons = ['张三','李四']
	return JsonResponse(person)
	
# 如果时HttpRespons时需要添加参数safe
def index(request):
	persons = ['张三','李四']
	return HttpResponse(person,safe=False)
```



#### 生成CSV文件

有时候我们做的网站，需要将一些数据，生成一个csv文件给浏览器，并且时作为附件的形式下载下来，如何生成csv文件

```python
import csv
from django.http import HttpResponse

def csv_view(request):
	response=HttpResponse(content_type='text/csv') # 指定格式
	# 如何处理文件的参数，attachment表示附件下载，filename表示文件名
	response['Content-Disposition'] ='attachement;filename="xxx.csv"'
	
	writer = csv.write(response)
	writer.writerow(['username','age','height'])
	writer.writerow(['wang','10','159'])
	return response


# 大型csv读取
from django.template import loader
def template_view(request):
   	response=HttpResponse(content_type='text/csv') # 指定格式
	# 如何处理文件的参数，attachment表示附件下载，filename表示文件名
	response['Content-Disposition'] ='attachement;filename="xxx.csv"'
	
    context={
        'rows':{
            ['username','age'],
            ['zhilian',90],
        }
    }
	template = loader.get_template('abc.txt')  # 传入模板进行渲染
    csv_template = template.render(context)  # 传入数据
    response.content = csv_template  # 将数据赋值
	return response 

-----------------------------------
 # 对应的abc.txt
{% for row in rows %} {{row.0}},{{row.1}} 
{%endfor%}
```



#### CBV视图类

django.views.generic.base.View 是主要的类视图，所有的视图都是继承他，我们可以针对不同的请求方式定义不同的视图函数，类中结合 闭包+反射 来实现CBV => FBV开发模式

```python
urlpatterns = [
    path('detail/<book_id>',views.BookDetail.as_view())
]

----------------------------------------
class BookDetail(View):
    # 如果不接受book_id,就会放入kwargs参数中，request出现在args中
    def get(self, *args, **kwargs):
        print(kwargs,args)
        return HttpResponse('xxxx')
```

如果用户访问了View中的没有定义的方法，比如你的视图函数只支持get方法，而有post访问，此时就可以把这个请求转发给 http_method_not_allowed

```python
jango.views.generic.base.View from django.views import View  # 其实就是在jango.views.generic.base下的View
class BookDetail(View):
    # 如果不接受book_id,就会放入kwargs参数中，request出现在args中
    def get(self, *args, **kwargs):
        print(kwargs,args)
        return HttpResponse('xxxx')
    
    def http_method_not_allowed(self, request, *args, **kwargs):
        return HttpResponse('您访问的方式没有对应的逻辑处理')
```



#### TemplateView模板类

django.views.generic.base.TemplateView ，这个类视图是专门用来返回模板的，这个类中有两个属性需要用到：template_name,用来存储模板路径，TemplateView会自动渲染这个变量的模板；另一个：get_context_date，用来返回上下文数据，也就是传入的参数

```python
1、方式一：使用到了传递参数，需要重写get_context_date函数

urlpatterns = [
    path('detail/<book_id>',views.BookDetail.as_view()),
    path('',views.HomePageView.as_view())
]

-----------------------------------------------------
class HomePageView(TemplateView):
    template_name='index.html'

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['username'] = '王立'
        return context
        
        
2、方式二：直接使用模板
from django.views.generic.base import TemplateView
urlpatterns = [
    path('',views.TemplateView.as_view('index.html'))
]
```



#### ListView列表视图

在网站开发中，经常会出现需要列出某个表中的一些数据作为列表展示出来，比如文章列表，图书列表等，在Django中可以使用ListView来帮助我们快速实现这个需求

```python
# url.py文件
from django.urls import path
from . import views

app_name = 'app'
urlpatterns = [
    path('add/',views.add_article),
    path('list/',views.ArticleListView.as_view()),
]
-------------------------------------------------

#views.py
from django.shortcuts import render,HttpResponse
from django.views.generic import ListView
from .models import Article

# 添加数据
def add_article(request):
    articles = []
    for x in range(0,102):
        article = Article(title='标题：%s'%x,content='内容：%s'%x)
        articles.append(article)

    Article.objects.bulk_create(articles)
    return HttpResponse('文章保存成功')

# list数据列表
class ArticleListView(ListView):
    models= Article  # 针对的模型
    template_name = 'list.html'  # 针对的模板html
    context_object_name = 'articles'  # 这个是传入模板的变量名，也就是说在模板中使用传入的上下文变量名字
    paginate_by = 10  # 每页显示数据条目
    ordering = 'create_time' # 排序
    page_kwarg = 'page'   # 用于跳转的url参数关键字

    def get_context_data(self, *, object_list=None, **kwargs):
        context = super(ArticleListView,self).get_context_data(**kwargs)
        print(context)  # 可对数据进行操作，object_list中存储的是对象列表，可以发现里面存在'pagintor'和'page_obj'键
        
        return context

    def get_queryset(self):
        # 必须写这个参数，用于限制要返回的数据限制，显示小于80 的数据，并不是一次性全返回
        return Article.objects.filter(id__lte=80)

-------------------------------------------------------

 # models.py文件
from django.db import models

class Article(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
    create_time = models.DateTimeField(auto_now_add=True)
```



#### Paginator和Page类

都是用作分页的，他们在Django中的路径为 django.core.paginator 和 django.core.paginator.Page。常用于ListView类结合使用，因为在

Pageinator 常用属性和方法：

- count：总共有多少条数
- num_pages: 总共有多少页 
- page_range: 页面的区间，比如有三页，那么就range(1,4)



Page常用属性和方法：

- has_next: 是否有下一页

- has_previous: 是否有上一页

- next_page_number:下一页的页码

- previous_page_number:上一页的页码

- number: 当前页

- start_index:当前这一页的第一条数据和索引值

- end_index: 当前这一页的最后一条数据的索引

  ```python
  from django.shortcuts import render,HttpResponse
  from django.views.generic import ListView
  from .models import Article
  
  class ArticleListView(ListView):
      models= Article  # 针对的模型
      template_name = 'list.html'  # 针对的模板html
      context_object_name = 'articles'  # 这个是传入模板的变量名，也就是说在模板中使用传入的上下文变量名字
      paginate_by = 10  # 每页显示数据条目
      ordering = 'create_time' # 排序
      page_kwarg = 'page'   # 用于跳转的url参数关键字
  
  
      def get_context_data(self, *, object_list=None, **kwargs):
          context = super(ArticleListView,self).get_context_data(**kwargs)
          
          paginator = context.get('paginator')
          page_obj = context.get('page_obj')
          
          print(paginator.count)  # 打印总共条数，经过筛选的80条
          print(paginator.num_pages)  # 8
          print(paginator.page_range)  # range(1,9)
          print(page_obj.has_previous())  # 第一页时返回False，没有上一页了
  
          return context
  
      def get_queryset(self):
          # 必须写这个参数，用于限制要返回的数据限制，显示小于80 的数据
          return Article.objects.filter(id__lte=80)
  ```

  

- 结合 Bootstrap 完成分页

  - [打开官网](https://v3.bootcss.com/components/#pagination)，找到组件中的分页；在html中导入

    ```html
    # list.html文件
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
        <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">
    </head>
    <body>
        <ul>
            {% for article in articles %}
                <li>{{ article.title}}</li>
            {% endfor %}
        
            <ul class="pagination">
                {#   上一页     #}
                {% if page_obj.has_previous %}
                    <li><a href="{% url 'app:article_list' %}?page={{ page_obj.previous_page_number }}">上一页</a></li>
                {% else %}
                    <li class="disabled"><a href="javascript:void(0)">上一页</a></li>
                {% endif %}
    
                {#   中间页     #}
                {% for page in paginator.page_range %}
                    {% if page == page_obj.number %}
                        <li class="active"><a href="javascript:void(0);">{{ page }}</a></li>
                    {% else %}
                        {#  这里的page在创建的时候已经生成的  #}
                        <li><a href="{% url 'app:article_list' %}?page={{ page }}">{{ page }}</a></li>
                    {% endif %}
                {% endfor %}
    
                {#   下一页     #}
                {% if page_obj.has_next %}
                    <li><a href="{% url 'app:article_list' %}?page={{ page_obj.next_page_number }}">下一页</a></li>
                {% else %}
                    <li class="disabled"><a href="javascript:void(0)">下一页</a></li>
                {% endif %}
            </ul>
        </ul>
    </body>
    ```

  - 后端代码

    ```python
    from django.shortcuts import render,HttpResponse
    from django.views.generic import ListView
    
    from .models import Article
    
    def add_article(request):
        articles = []
        for x in range(0,102):
            article = Article(title='标题：%s'%x,content='内容：%s'%x)
            articles.append(article)
    
        Article.objects.bulk_create(articles)
        return HttpResponse('文章保存成功')
    
    class ArticleListView(ListView):
    
        models= Article  # 针对的模型
        template_name = 'list.html'  # 针对的模板html
        context_object_name = 'articles'  # 这个是传入模板的变量名，也就是说在模板中使用传入的上下文变量名字
        paginate_by = 10  # 每页显示数据条目
        ordering = 'create_time' # 排序
        page_kwarg = 'page'   # 用于跳转的url参数关键字
    
    
        def get_context_data(self, *, object_list=None, **kwargs):
            context = super(ArticleListView,self).get_context_data(**kwargs)
    
            paginator = context.get('paginator')
            page_obj = context.get('page_obj')
            print(paginator.count)  # 打印总共条数，经过筛选的80条
            print(paginator.num_pages)  # 8
            print(paginator.page_range)  # range(1,9)
            print(page_obj.has_previous())  # 第一页时返回False，没有上一页了
    
            return context
    
        def get_queryset(self):
            # 必须写这个参数，用于限制要返回的数据限制，显示小于80 的数据
            return Article.objects.filter(id__lte=80)
    ```

- 分页跟进版本，截取页

  ```python
  list.html
  
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">
  </head>
  <body>
      <ul>
          {% for article in articles %}
              <li>{{ article.title}}</li>
          {% endfor %}
      
          <ul class="pagination">
              {#   上一页     #}
              {% if page_obj.has_previous %}
                  <li><a href="{% url 'app:article_list' %}?page={{ page_obj.previous_page_number }}">上一页</a></li>
              {% else %}
                  <li class="disabled"><a href="javascript:void(0)">上一页</a></li>
              {% endif %}
  
              {% if left_has_more %}
                  <li><a href="{% url 'app:article_list' %}?page=1">1</a></li>
                  <li><a href="javascript:void(0)">...</a></li>
              {% endif %}
  
              {# 左边的页码  #}
              {% for left_page in left_pages %}
                  <li><a href="{% url 'app:article_list' %}?page={{ left_page }}">{{ left_page }}</a></li>
              {% endfor %}
  
              {#   当前页    #}
                  <li class="active"><a href="{% url 'app:article_list' %}?page={{ current_page }}">{{ current_page }}</a></li>
  
              {#    右边页    #}
              {% for right_page in right_pages %}
                   <li><a href="{% url 'app:article_list' %}?page={{ right_page }}">{{ right_page }}</a></li>
              {% endfor %}
  
              {% if right_has_more %}
                  <li><a href="javascript:void(0)">...</a></li>
                  <li><a href="{% url 'app:article_list' %}?page={{ num_pages }}">{{ num_pages }}</a></li>
              {% endif %}
  
              {#   下一页     #}
              {% if page_obj.has_next %}
                  <li><a href="{% url 'app:article_list' %}?page={{ page_obj.next_page_number }}">下一页</a></li>
              {% else %}
                  <li class="disabled"><a href="javascript:void(0)">下一页</a></li>
              {% endif %}
          </ul>
      </ul>
  </body>
  
  -----------------------------------------------------------
   # views.py文件
  from django.shortcuts import render,HttpResponse
  from django.views.generic import ListView
  from .models import Article
  
  
  class ArticleListView(ListView):
  
      models= Article  # 针对的模型
      template_name = 'list.html'  # 针对的模板html
      context_object_name = 'articles'  # 这个是传入模板的变量名，也就是说在模板中使用传入的上下文变量名字
      paginate_by = 10  # 每页显示数据条目
      ordering = 'create_time' # 排序
      page_kwarg = 'page'   # 用于跳转的url参数关键字
  
  
      def get_context_data(self, *, object_list=None, **kwargs):
          context = super(ArticleListView,self).get_context_data(**kwargs)
  
          paginator = context.get('paginator')
          page_obj = context.get('page_obj')
          print(paginator.count)  # 打印总共条数，经过筛选的80条
          print(paginator.num_pages)  # 8
          print(paginator.page_range)  # range(1,9)
          print(page_obj.has_previous())  # 第一页时返回False，没有上一页了
  
          pagination_data = self.get_pagination_data(paginator,page_obj)
          context.update(pagination_data)
          return context
  
      def get_queryset(self):
          # 必须写这个参数，用于限制要返回的数据限制，显示小于80 的数据
          return Article.objects.filter(id__lte=100)
  
      # 页码截取
      def get_pagination_data(self,paginator,page_obj,around_count=2):
          # 环绕当前页的前后为2
          current_page = page_obj.number
          num_pages = paginator.num_pages
          left_has_more = False
          right_has_more = False
  
          # 左边部分
          if current_page <= around_count+2:
              left_pages = range(1,current_page)
          else:
              left_has_more=True
              left_pages = range(current_page-around_count,current_page)
  
          # 右边部分
          if current_page >= num_pages-around_count-1:
              right_pages = range(current_page+1,num_pages+1)
          else:
              right_has_more = True
              right_pages = range(current_page+1,current_page+around_count+1)
  
          return {
              'left_pages':left_pages,
              'right_pages':right_pages,
              'current_page':current_page,
              'left_has_more':left_has_more,
              'right_has_more':right_has_more,
              'num_pages':num_pages
          }
  ```

  ![1570779968425](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1570779968425.png)

