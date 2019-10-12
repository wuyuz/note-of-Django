## csrf、Ajax

**csrf**：Cross Site Request Forgery,跨站点伪造请求。

![1561621029068](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1561621029068.png)

- **举个例子**：

  ​		假如用户abc登录了银行的网站，并且向abc2进行了转账，对银行发送的请求是 http://bank.example/withdraw?account=abc&amount=1000000&for=abc2. 通常情况下，请求发送到服务器后，服务器会首先验证是否是合法的session，如果是则转账成功。假设黑客也有同样银行的账号。他知道转账的时候会生成如上的请求链接。黑客也可以发送同样的请求给服务器要求转账给自己。但是服务器校验他的这个请求不是合法的session。因此黑客想到了CSRF的方式。他自己做一个网站，在网站中发下如下链接：http://bank.example/withdraw?account=abc&amount=1000000&for=heike 并且通过广告或其他的方式诱使abc点击这个链接，上述 url 就会从 abc的浏览器发向银行，而这个请求会附带 abc浏览器中的 cookie 一起发向银行服务器。大多数情况下，该请求会失败，因为他要求 abc 的认证信息。但是，如果 abc 当时恰巧刚访问他的银行后不久，他的浏览器与银行网站之间的 session 尚未过期，浏览器的 cookie 之中含有 abc 的认证信息。这时，悲剧发生了，这个 url 请求就会得到响应，钱将从 abc 的账号转移到黑客的账号，而 abc 当时毫不知情。

  ```
  解决办法：
  	CSRF 攻击之所以能够成功，是因为黑客可以完全伪造用户的请求，该请求中所有的用户验证信息都是存在于cookie中，因此黑客可以在不知道这些验证信息的情况下直接利用用户自己的 cookie来通过安全验证。要抵御CSRF，关键在于在请求中放入黑客所不能伪造的信息(隐藏的input标签)，并且该信息不存在于 cookie 之中。可以在 HTTP 请求中以参数的形式加入一个随机产生的 token（在请求银行网页时就Response了），并在服务器端建立一个拦截器来验证这个 token，如果请求中没有 token 或者 token 内容不正确，则认为可能是 CSRF 攻击而拒绝该请求。
  ```

- 在Django中我们最初已经了解过csrf: 

  ```
  我们在setting.py文件中，如果不注释中间件中的：'django.middleware.csrf.CsrfViewMiddleware',那么一般的form表单就不能提交数据，只能在form中添加 {% csrf_token %} 才能通过。换句话说，中间件的csrf没注释就要在所有提交信息中添加 {% csrf_token %}。
  ```

  

#### csrf的装饰器

- **csrf_exempt** ：某个视图不需要进行csrf校验，也就是说即使中间件的csrf没有注释，一样放行form的提交信息，即使你form有{% csrf_token %}。

  ```python
  from django.views.decorators.csrf import csrf_exempt, csrf_protect
  from django.utils.decorators import method_decorator # 用于CBV加装饰器
  
  @method_decorator(csrf_exempt, name='dispatch') # 第二种方式 加在类上，就不用写继承dispatch的函数代码， 其实就是给它加
  class Login(View):
      @method_decorator(csrf_exempt) # 第一种方法 csrf_exempt只能加在dispatch方法上
      def dispatch(self, request, *args, **kwargs):
          ret = super().dispatch(request, *args, **kwargs)
          return ret
      
      def get(self,request,*args,**kwargs):
          return render(request,'login.html')
      
      def post(self,request,*args,**kwargs):
          pass
          return render(request,'login.html',{'error':'用户名密码错误'})
          
   -------------------------以上是CBV的模式------------------ 
  from django.views.decorators.csrf import csrf_exempt, csrf_protect
  @csrf_exempt  # FBV 模式
  def ok(request):
      return render(request,'login.html')
  ```
  
- **csrf_protect** ： 某个视图需要进行csrf校验，即使你中间件的csrf注释了，一样需要form添加{% csrf_token %}提交信息。

  ```python
  from django.views.decorators.csrf import csrf_exempt, csrf_protect
  class Login(View):
      def get(self,request,*args,**kwargs):
          return render(request,'login.html')
  
      @method_decorator(csrf_protect)  # form中必须有{% csrf_token %}
      def post(self,request,*args,**kwargs):
          pass
          return render(request,'login.html',{'error':'用户名密码错误'})
   -------------------------CBV以上----------------------   
  
  @csrf_protect
  def ok(request):
      return render(request,'login.html')
  ```

- **ensure_csrf_cookie** : 确保生成csrf的cookie

  ```python
    # 有时后我们通过{% csrf_token %} 在网页生成input标签，但是并没有设置相应的cookie。此时我们就需要ensure_csrf_cookie方法加在get方法上(CBV)
    
  from django.views.decorators.csrf import  ensure_csrf_cookie
  from django.utils.decorators import method_decorator # 用于CBV
  
  class Login(View):
      @method_decorator(ensure_csrf_cookie) # 确保生成csrf_token
      def get(self,request,*args,**kwargs):
          return render(request,'login.html')
      
      def post(self,request,*args,**kwargs):pass
      
    # 总结： 必须要要生成cookie，再让{% csrf_token %}生成input标签    
  ```

  

#### csrf_token实现原理：

​		我们可以通过：`from django.middleware.csrf import CsrfViewMiddleware` 中的CsrfViewMiddleware去查看源码，它是一个类且有以下方法：`_accept、 _reject、 _get_token、 _get_token 、_set_token 、process_request 、process_view 、process_response` , 显然_单下划线的方法都是私有方法，所以请求来的时候最先触发`process_request`函数：

- csrf中间件中执行`process_request`：此函数用于给每个正常访问网页的cookie，中添加一个csrftoken,放入请求中，每次来都认得。

  ```python
  def _get_token(self, request):
       if settings.CSRF_USE_SESSIONS: # 默认是False
         ...
        else:
            try: # 获取cookie值，setting中的CSRF_COOKIE_NAME是csrftoken
                cookie_token = request.COOKIES[settings.CSRF_COOKIE_NAME]
            except KeyError:
                return None
            csrf_token = _sanitize_token(cookie_token) # 验证长度，清洗
            if csrf_token != cookie_token: #清洗后不一致，需重置
                request.csrf_cookie_needs_reset = True
            return csrf_token
   
  #from django.conf import global_settings 查看源码全局配置
  
  def process_request(self, request):
      csrf_token = self._get_token(request)  # 1从cookie中获得csrf_token
      if csrf_token is not None:
              # Use same token next time.
            request.META['CSRF_COOKIE'] = csrf_token # 将从cookie中获得的值放在请求中
  
   # 总结：
  	1. 从cookie中获取到csrftoken的值
      2. csrftoken的值放入到request.META
  ```

- **执行process_view**：此函数用于比较来访问的cookie中是否有合格的csrftoken

  ```
  1. 查询视图函数是否使用csrf_exempt装饰器，使用了就不进行csrf的校验
  2. 判断请求方式：
     1. 如果是GET', 'HEAD', 'OPTIONS', 'TRACE'不进行csrf校验
     2. 其他的请求方式（post，put）,进行csrf校验：
     		1.获取cookie中csrftoken的值,获取csrfmiddlewaretoken的值
  		2.能获取到  ——》 request_csrf_token
  		3.获取不到   ——》 获取请求头中X-csrftoken的值 ——》request_csrf_token
  比较上述request_csrf_token和cookie中csrftoken的值，比较成功接收请求，比较不成功拒绝请求。    
  ```

- **总结**： 

  ```
  1、{% csrf_token %} 生成input标签，并且存储里面的生成的csrfmiddlewaretoken对应得值，查看源码，默认传入两个参数触发它，里面有一个列表记录值，然后返回加密值给用户，同时在CsrfViewMiddleware中将修改cookie携带得值，这个值就是csrf_token,以后的请求都会携带一阵csrf_token,后端存有关于这个csrf_token对应的值，最后用网页上csrfmiddlewaretoken对应的值去盐，和后端存的值进行比较。
  2、综上，cookie就是一个桥梁，csrfmiddlewaretoken就一把钥匙，后端数据和csrfmiddlewaretoken对应的值就是比较对象。
  ```

  

#### Ajax

- 什么是JSON？

  ```js
  JSON 指的是 JavaScript 对象表示法（JavaScript Object Notation）
  JSON 是轻量级的文本数据交换格式
  JSON 独立于语言 *
  JSON 具有自我描述性，更易理解
  
   # JSON 使用 JavaScript 语法来描述数据对象，但是 JSON 仍然独立于语言和平台。JSON 解析器和 JSON 库支持许多不同的编程语言。
   
   #合格的json：
  ["one", "two", "three"]
  { "one": 1, "two": 2, "three": 3 }
  {"names": ["张三", "李四"] }
  [ { "name": "张三"}, {"name": "李四"} ]
  
   #不合格的json
  { name: "张三", 'age': 32 }  // 属性名必须使用双引号
  [32, 64, 128, 0xFFF] // 不能使用十六进制值
  { "name": "张三", "age": undefined }  // 不能使用undefined
  { "name": "张三",
    "birthday": new Date('Fri, 26 Aug 2011 07:13:10 GMT'),
    "getName":  function() {return this.name;}  // 不能使用函数和日期对象
  } 
   
  ```

- ##### json将python转换json后的数据对照表：

  | Python           | JSON   |
  | ---------------- | ------ |
  | dict             | object |
  | list, tuple      | array  |
  | str, unicode     | string |
  | int, long, float | number |
  | True             | true   |
  | False            | false  |
  | None             | null   |

  

- **预备知识**：JavaScript中关于JSON对象和字符串转换的两个方法：

  ```js
   # JSON.parse(): 用于将一个JSON字符串转换为JavaScript对象　前端反序列化
  JSON.parse('{"name":"alex"}');
  JSON.parse('{name:"alex"}') ;      // 错误
  JSON.parse('[18,undefined]') ;     // 错误
  
   # JSON.stringify(): 用于将 JavaScript 值转换为 JSON字符串 前端序列化
  JSON.stringify({"name":"alex"}) 
  
   # 后端如果序列化的两种方式
   #法一
  from django.http.response import JsonResponse；
  return JsonResponse({name:'xxx',}) #全自动，后端也全自动
   #法二
  return HttpResponse(json.dumps({name:'xxx',}))  #手动,前端:JSON.parse(data)
  ```

- **介绍**：AJAX（Asynchronous Javascript And XML）翻译成中文就是“异步的Javascript和XML”。即使用Javascript语言与服务器进行异步交互，传输的数据为XML（当然，传输的数据不只是XML）

- 和XML比较：XML和JSON都使用结构化方法来标记数据；下面是两种标记数据的格式

  ```json
  <?xml version="1.0" encoding="utf-8"?>
  <country>
      <name>中国</name>
      <province>
          <name>黑龙江</name>
          <cities>
              <city>哈尔滨</city>
              <city>大庆</city>
          </cities>
      </province>
      <province>
          <name>广东</name>
          <cities>
              <city>广州</city>
              <city>深圳</city>
              <city>珠海</city>
          </cities>
      </province>
  </country>
  
   -----------------------上面是XML------------------------
   {
      "name": "中国",
      "province": [{
          "name": "黑龙江",
          "cities": {
              "city": ["哈尔滨", "大庆"]
          }
      }, {
          "name": "广东",
          "cities": {
              "city": ["广州", "深圳", "珠海"]
          }
      }
  }
   //显然下面json的数据结构更清晰                 
  ```

  

- **特性**：

  ```
  1、AJAX 不是新的编程语言，而是一种使用现有标准的新方法。
  2、AJAX 最大的优点是在不重新加载整个页面的情况下，可以与服务器交换数据并更新部分网页内容。（这一特点给用户的感受是在不知不觉中完成请求和响应过程）
  3、AJAX 不需要任何浏览器插件，但需要用户允许JavaScript在浏览器上执行。
  
  同步交互：客户端发出一个请求后，需要等待服务器响应结束后，才能发出第二个请求；
  异步交互：客户端发出一个请求后，无需等待服务器响应结束，就可以发出第二个请求。
  ```

- **简单实例**：实现一个简单的加法，当然一般的方法也能实现，但是每次都要刷新页面才能得到结果，并且都是同步的，所以我们考虑使用ajax

  ```python
   # html文件中：
  <!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="x-ua-compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>AJAX局部刷新实例</title>
  </head>
  <body>
  <input type="text" id="i1">+
  <input type="text" id="i2">=
  <input type="text" id="i3">
  <input type="button" value="AJAX提交" id="b1">
  
  <script src="/static/jquery-3.2.1.min.js"></script>
  <script>
    $("#b1").on("click", function () {
      $.ajax({
        url:"/ajax_add/",
        type:"GET",
        data:{"i1":$("#i1").val(),"i2":$("#i2").val()}, # 必须传入字符串
        success:function (data) {
          $("#i3").val(data);
        }
      })
    })
  </script>
  </body>
  </html> 
  
  --------------------------------------------------
   # views.py文件中
  def ajax_demo1(request):
      return render(request, "ajax_demo1.html")
  
  def ajax_add(request):
      i1 = int(request.GET.get("i1"))
      i2 = int(request.GET.get("i2"))
      ret = i1 + i2
      return JsonResponse(ret, safe=False)
      
  ----------------------------------------------------
   # url.py文件
  urlpatterns = [
      ...
      url(r'^ajax_add/', views.ajax_add),
      url(r'^ajax_demo1/', views.ajax_demo1),
      ...   
  ] 
  ```
  


#### Ajax常见应用情景

- 搜索引擎根据用户输入的关键字，自动提示检索关键字。

- 还有一个很重要的应用场景就是注册时候的用户名的查重。

其实这里就使用了AJAX技术！当文件框发生了输入变化时，使用AJAX技术向服务器发送一个请求，然后服务器会把查询到的结果响应给浏览器，最后再把后端返回的结果展示出来。

```
1、整个过程中页面没有刷新，只是刷新页面中的局部位置而已！
2、当请求发出后，浏览器还可以进行其他操作，无需等待服务器的响应！
```

![1561631857975](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1561631857975.png)

- **Ajax的优点**：

  ```
  1.AJAX使用JavaScript技术向服务器发送异步请求；
  2.AJAX请求无须刷新整个页面；
  3.因为服务器响应内容不再是整个页面，而是页面中的部分内容，所以AJAX性能高；
  4.一般来说ajax在向后端放送请求后得到响应，在success中应该做出相应的变化使页面
  
  注意点： url后的路径必须/xxx/，不然会在本路径下拼接xxx/(坑)，
  ```

- 写一个登陆认证页面，登陆失败不刷新页面，提示用户登陆失败，登陆成功自动跳转到网站首页。

  ```python
   # login.html文件
  {% load static %}
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
  
  <div>
      用户名:<input type="text" id="username">
      密码:<input type="text" id="pwd">
      {% csrf_token %}
      <button id="sub">提交</button>
      <span style="color: red;font-size: 12px;" id="error"></span>
  </div>
  
  <script src="{% static 'jquery.js' %}"></script>
  <script>
      $('#sub').click(function () {
          $.ajax({
              url:"{% url 'login' %}",
              type:'post',
              data:{username:$('#username').val(),pwd:$('#pwd').val(),csrfmiddlewaretoken:$('[name=csrfmiddlewaretoken]').val()},
              success:function (data) {
                  data = JSON.parse(data);
                  console.log(data,typeof data);
                  if (data['status']){
                      location.href=data['home_url'];
                  }
                  else {
                      $('#error').text('用户名或者密码错误！')
                  }
              }
          })
      })
  </script>
  </body>
  </html> 
  
   ------------------------------------------------------
    # home.html文件
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
  <h1>欢迎来到xxx官网</h1>
  </body>
  </html>
  
  ---------------------------------------------------------
   #urls.py文件
  url(r'^login/', views.login,name='login'),
  url(r'^home/', views.home,name='home'),
  
  ---------------------------------------------------------
   #views.py文件中
  def login(request):
      res_dict = {'status':None,'home_url':None}
      if request.method == 'GET':
          return render(request,'login.html')
      else:
          uname = request.POST.get('username')
          pwd = request.POST.get('pwd')
  
          user_obj = models.UserInfo.objects.filter(name=uname,password=pwd).exists()
          import json
          if user_obj:
              res_dict['status'] = True
              res_dict['home_url'] = reverse('home')
              res_json_dict = json.dumps(res_dict)
  
              return HttpResponse(res_json_dict) #直接回复字典格式是不可以的，必须转换成json字符串，转换成普通字符串也是不行的，因为前端需要对json进行反序列获得这个字典，在通过字典的形式来操作数据。
          else:
              res_dict['status'] = False
              res_json_dict = json.dumps(res_dict)
              return HttpResponse(res_json_dict)
  　　　　　　# 如果你就是不使用JsonResponse的话，也可以给HttpResponse添加一个参数，content_type='application/json'，那么前端ajax拿到数据之后，也是不需要反序列化的，ajax的回调函数就收到的就是一个反序列化之后的一个对象，因为ajax接受到数据后，通过这个data_type或者content_type发现你发送来的是个json格式的数据，那么ajax内容就自动将这个数据反序列化得到了js的数据对象，然后通过对象可以直接操作数据。
  　　　　　　# return HttpResponse(res_json_dict,data_type='application/json') #方式一
  　　　　　　# return JsonResponse(res_dict) #方式二
  def home(request):
      return render(request,'base.html') 
  ```

- **注意点**：

  -  1、ajax里面写$(this)时要注意的问题：还有一点注意，如果你添加某些dom对象的时候，如果你想在不刷新页面的情况下来添加这个对象，那么你要注意，如果这个对象也需要绑定事件的话，你需要用on来给和他相同的标签对象来绑定事件。**可以使用箭头函数来避免this指向变化**

  ![1561635206506](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1561635206506.png)



#### Ajax的使用：

- 基于jquery实现Ajax：

  ```js
  <button class="send_Ajax">send_Ajax</button>
  <script>
         $(".send_Ajax").click(function(){
             $.ajax({
                 url:"/handle_Ajax/",
                 type:"POST",
                 data:{username:"chao",password:123},
                 success:function(data){
                     console.log(data)
                 },        　　　　　　
                 error: function (jqXHR, textStatus, err) { //错误信息
                          console.log(arguments);
                      },
                 complete: function (jqXHR, textStatus) {
                          console.log(textStatus);
                  },
                 statusCode: {
                      '403': function (jqXHR, textStatus, err) {
                            console.log(arguments);
                       },
                      '400': function (jqXHR, textStatus, err) {
                          console.log(arguments);
                      }
                  }
             })
  
         })
  </script>
  
   //各项参数说明：
  1、url: 要求为String类型的参数，（默认为当前页地址）发送请求的地址。
  2、type: 要求为String类型的参数，请求方式（post或get）默认为get。注意其他http请求方法，例如put和delete也可以使用，但仅部分浏览器支持。
  3、timeout: 要求为Number类型的参数，设置请求超时时间（毫秒）。此设置将覆盖$.ajaxSetup()方法的全局设置。
  
  4、async: 要求为Boolean类型的参数，默认设置为true，所有请求均为异步请求。如果需要发送同步请求，请将此选项设置为false。注意，同步请求将锁住浏览器，用户其他操作必须等待请求完成才可以执行。
  5、cache: 要求为Boolean类型的参数，默认为true（当dataType为script时，默认为false），设置为false将不会从浏览器缓存中加载请求信息。
  
  6、data: 要求为Object或String类型的参数，发送到服务器的数据。如果已经不是字符串，将自动转换为字符串格式。get请求中将附加在url后。防止这种自动转换，可以查看　　processData选项。对象必须为key/value格式，例如{foo1:"bar1",foo2:"bar2"}转换为&foo1=bar1&foo2=bar2。如果是数组，JQuery将自动为不同值对应同一个名称。例如{foo:["bar1","bar2"]}转换为&foo=bar1&foo=bar2。
  
  7、.dataType: 要求为String类型的参数，预期服务器返回的数据类型。如果不指定，JQuery将自动根据http包mime信息返回responseXML或responseText，并作为回调函数参数传递。可用的类型如下：
  	xml：返回XML文档，可用JQuery处理。
  	html：返回纯文本HTML信息；包含的script标签会在插入DOM时执行。
  	script：返回纯文本JavaScript代码。不会自动缓存结果。除非设置了cache参数。注意             在远程请求时（不在同一个域下），所有post请求都将转为get请求。
  	json：返回JSON数据。
  	jsonp：JSONP格式。使用SONP形式调用函数时，例如myurl?callback=?，JQuery将自			  动替换后一个“?”为正确的函数名，以执行回调函数。
  	text：返回纯文本字符串。
  8、complete：要求为Function类型的参数，请求完成后调用的回调函数（请求成功或失败时均调用）。参数：XMLHttpRequest对象和一个描述成功请求类型的字符串。
            function(XMLHttpRequest, textStatus){
               this;    //调用本次ajax请求时传递的options参数
            }
  9、processData：声明当前的data数据是否进行转码或预处理，默认为true，即预处理；if为false，那么对data：{a:1,b:2}会调用json对象的toString()方法，即{a:1,b:2}.toString(),最后得到一个［object，Object］形式的结果。
  
  10、contentType：默认值: "application/x-www-form-urlencoded"。发送信息至服务器时内容编码类型。用来指明当前请求的数据编码格式；urlencoded:?a=1&b=2；如果想以其他方式提交数据，比如contentType:"application/json"，即向服务器发送一个json字符串：
                 $.ajax("/ajax_get",{
                    data:JSON.stringify({  //提前转换
                         a:22,
                         b:33
                     }),
                     contentType:"application/json", 
                     type:"POST",          
                 });                          //{a: 22, b: 33}
  
               注意：contentType:"application/json"一旦设定，data必须是json字符串，不能是json对象
   // 相应的views.py: json.loads(request.body.decode("utf8"))
  ```

  

- 基于原生js实现：

  ```js
  var b2 = document.getElementById("b2");
    b2.onclick = function () {
      // 原生JS
      var xmlHttp = new XMLHttpRequest();
      xmlHttp.open("POST", "/ajax_test/", true);
      xmlHttp.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
      xmlHttp.send("username=chao&password=123456");
      xmlHttp.onreadystatechange = function () {
        if (xmlHttp.readyState === 4 && xmlHttp.status === 200) {
          alert(xmlHttp.responseText);
        }
      };
    };
  ```

- **Ajax 实现流程**：

  ![1561636142177](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1561636142177.png)

  

#### Ajax请求设置csrf_token

- **前提**：

  ```
  1、确保有csrftoken的cookie
  2、在页面中使用{% csrf_token %}，加装饰器 ensure_csrf_cookie
  3、JsonResponse常常要求回复字典类型，否则会报safe类型错误，所以需要设置safe=True：JsonResponse(ret,safe=True)
  ```

- **方式一**：通过获取隐藏的input标签中的csrfmiddlewaretoken值，放置在data中发送。

  ```js
  $.ajax({
    url: "/cookie_ajax/",
    type: "POST",
    data: {
      "username": "chao",
      "password": 123456,
      "csrfmiddlewaretoken": $("[name = 'csrfmiddlewaretoken']").val()  // 使用jQuery取出csrfmiddlewaretoken的值，拼接到data发送
    },
    success: function (data) {
       console.log(data);
    }
  })
  ```

- **方式二**：通过获取返回的cookie中的字符串 放置在请求头中发送。注意：需要引入一个jquery.cookie.js插件。

  ```js
  <script src="{% static 'js/jquery.cookie.js' %}"></script>
  $.ajax({
    url: "/cookie_ajax/",
    type: "POST",
    headers: {"X-CSRFToken": $.cookie('csrftoken')},  // 从Cookie取csrftoken，并设置到请求头中
    data: {"username": "Q1mi", "password": 123456},
    success: function (data) {
      console.log(data);
    }
  })
  
  ---------------------------------------------------------------
   #总结
  <script src="{% static 'js/jquery.cookie.js' %}"></script>
  $.ajax({
  	headers:{"X-CSRFToken":$.cookie('csrftoken')}, //其实在ajax里面还有一个参数是headers，自定制请求头，可以将csrf_token加在这里，我们发contenttype类型数据的时候，csrf_token就可以这样加
   
  }) 
  ```

  - 或则可以自己写一个getCookie方法：

    ```js
    function getCookie(name) {
        var cookieValue = null;
        if (document.cookie && document.cookie !== '') {
            var cookies = document.cookie.split(';');
            for (var i = 0; i < cookies.length; i++) {
                var cookie = jQuery.trim(cookies[i]);
                // Does this cookie string begin with the name we want?
                if (cookie.substring(0, name.length + 1) === (name + '=')) {
                    cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                    break;
                }
            }
        }
        return cookieValue;
    }
    var csrftoken = getCookie('csrftoken');
    ```

  - 每一次都这么写太麻烦了，可以使用$.ajaxSetup()方法为ajax请求统一设置。

    ```js
    function csrfSafeMethod(method) {
      // these HTTP methods do not require CSRF protection
      return (/^(GET|HEAD|OPTIONS|TRACE)$/.test(method));
    }
    
    $.ajaxSetup({
      beforeSend: function (xhr, settings) {
        if (!csrfSafeMethod(settings.type) && !this.crossDomain) {
          xhr.setRequestHeader("X-CSRFToken", csrftoken);
        }
      }
    });
    ```

  - **注意点**：

    ```python
    如果使用从cookie中取csrftoken的方式，需要确保cookie存在csrftoken值。
    如果你的视图渲染的HTML文件中没有包含 {% csrf_token %}，Django可能不会设置CSRFtoken的cookie。
    这个时候需要使用ensure_csrf_cookie()装饰器强制设置Cookie。
    
    django.views.decorators.csrf import ensure_csrf_cookie
    @ensure_csrf_cookie
    def login(request):
        pass
    ```

    

#### Ajax 文件上传

- **请求头**：ContentType指的是请求体的编码类型，常见的类型共有3种

  - `application/x-www-form-urlencoded`：这应该是最常见的 POST 提交数据的方式了。浏览器的原生 <form> 表单，如果不设置 `enctype` 属性，那么最终就会以 默认格式application/x-www-form-urlencoded 方式提交数据，ajax默认也是这个。请求类似于下面这样（无关的请求头在本文中都省略掉了）

    ```python
    POST http://www.example.com HTTP/1.1
    Content-Type: application/x-www-form-urlencoded;charset=utf-8
    
     #此时如果在form中使用get或则post提交数据，后端得到的都是类于user=yuan&age=22  这就是上面这种contenttype规定的数据格式，后端对应这个格式来解析获取数据，不管是get方法还是post方法，都是这样拼接数据，大家公认的一种数据格式，但是如果你contenttype指定的是urlencoded类型，但是post请求体里面的数据是下面那种json的格式，那么就出错了，服务端没法解开数据。
    ```

    我们当然可以使用Network查看拼接数据：

    ![1561639434084](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1561639434084.png)

    

  - `multipart/form-data`：　这又是一个常见的 POST 数据提交的方式。我们使用表单上传文件时，必须让 <form> 表单的 `enctype` 等于 multipart/form-data，form表单不支持发json类型的contenttype格式的数据，而ajax什么格式都可以发，也是ajax应用广泛的一个原因。直接来看一个请求示例：（了解）

  - `application/json`:application/json 这个 Content-Type 作为响应头大家肯定不陌生。实际上，现在越来越多的人把它作为请求头，用来告诉服务端消息主体是序列化后的 JSON 字符串。由于 JSON 规范的流行，除了低版本 IE 之外的各大浏览器都原生支持 JSON.stringify，服务端语言也都有处理 JSON 的函数，使用 JSON 不会遇上什么麻烦。

    ![1561639802257](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1561639802257.png)



- **基于form表单的文件上传**：图片等

  ```js
  <form> #用不用form没关系，这里就是个盒子的作用，一般写form标签是为了提示别人，这个地方的内容是要提交的
  　　　　　　{% csrf_token %}
        用户名 <input type="text" id="user">
        头像 <input type="file" id="avatar">
       <input type="button" id="ajax-submit" value="ajax-submit">
  </form>
  <script>
      $("#ajax-submit").click(function(){
          var formdata=new FormData(); #ajax上传文件的时候，需要这个类型，它会将添加给它的键值对加工成formdata的类型
          formdata.append("user",$("#user").val());  #添加键值的方法是append，注意写法，键和值之间是逗号
  　　　　 formData.append("csrfmiddlewaretoken", $("[name='csrfmiddlewaretoken']").val()); #别忘了csrf_token
          formdata.append("avatar_img",$("#avatar")[0].files[0]);
          $.ajax({
              url:"",
              type:"post",
              data:formdata, #将添加好数据的formdata放到data这里
              processData: false ,    // 不处理数据
              contentType: false,    // 不设置内容类型
              success:function(data){
                  console.log(data)
              }
          })
      })
  </script>
  
  -----------------------------------------------------
   #或则使用：
  var form = document.getElementById("form1");
  var fd = new FormData(form);
  
  这样也可以直接通过ajax 的 send() 方法将 fd 发送到后台。
  注意：由于 FormData 是 XMLHttpRequest Level 2 新增的接口，现在 低于IE10 的IE浏览器不支持 FormData。
  ------------------------------------------------------
  检查请求头：Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryaWl9k5ZMiTAzx3FT
  ------------------------------------------------------
  
   # 注意： 当ajax传的数据是个列表的时候，最好进行字符串转化：
   data:{hobby:JSON.stringify(['唱','跳'])} //相应的视图函数需要反序列化
  ```

- 解析request.boby数据：由于它是json数据，那么使用json模块，进行反序列化，就可以了！修改ajax_handle视图函数：

  ```python
  def ajax_handle(request):
      print("body",request.body)
      print("POST",request.POST)
      #由于是一个bytes类型，需要解码。再用json反序列化才行
      data = json.loads(request.body.decode("utf-8"))
      print(data) #打印json
      print(data["a"])  # 取key为a的值
      return HttpResponse('ok')
   ------------------------------------------------------
    # 结果
  body b'{"a":1,"b":2}'
  POST <QueryDict: {}>
  {'b': 2, 'a': 1}
  1  
  ```

- 基于Ajax的上传文件详细版：利用ajax和FormData实现页面无刷新的文件上传效果，主要用到了jQuery的ajax()方法和XMLHttpRequest Level 2的`FormData接口。关于FormData，大家可以看MDN文档：

  ```js
   #修改file_put视图函数
  def file_put(request):
      if request.method == "POST":
          print(request.POST)  # 打印POST信息
          print(request.FILES)  # 打印文件信息
  
          file_obj = request.FILES.get("img")  # 获取img
          print('type',type(file_obj))
          print(file_obj.__dict__)  # 打印img对象属性
          print(file_obj.name)  # 打印文件名
  
          response = {"state":False}
          with open("static/images/"+file_obj.name,"wb") as f:  # 打开文件
              for line in file_obj:
                  ret = f.write(line)  # 写入文件
                  print(ret)  # 返回的是写入的字符长度
                  if ret:  # 判断返回值
                      response["state"] = True
          return HttpResponse(json.dumps(response))  # 返回json
      return render(request, "file_put.html")  # 渲染页面file_put.html 
      
  ---------------------------------------------------------
   #修改file_put.html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
  {% csrf_token %}
  <h3>form表单文件上传</h3>
  <form>
        用户名 <input type="text" id="user"><br/>
        头像 <input type="file" id="avatar"><br/><br/>
       <input type="button" id="ajax-submit" value="ajax-submit">
  </form>
  
  <script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>
  <script>
      $("#ajax-submit").click(function(){
          var csrf = $("[name=csrfmiddlewaretoken]").val();  //csrf
          var formdata=new FormData();  //实例化了一个空的FormData对象
          formdata.append("csrfmiddlewaretoken",csrf);  //给当前FormData对象添加一个键/值对.
          formdata.append("user",$("#user").val());
          formdata.append("img",$("#avatar")[0].files[0]);
          $.ajax({
  
              url:"",  //表示为当前url,
              type:"post",
              data:formdata,  //发送一个FormData对象
              processData: false ,    // 不处理数据
              contentType: false,    // 不设置内容类型
              success:function(data){
                  var data = JSON.parse(data);  //反序列化数据
                  console.log(data);
                  if (data.state){ //判断返回值
                      //弹出提示框，并刷新整个页面
                      alert('上传成功');window.location.href="/file_put/";
                  }else {
                      alert('上传失败');
                  }
              }
          })
      })
  </script>
  </body>
  </html> 
  //代码很简单，需要注意的是页面中没有用到form表单，那么怎么提交数据呢，答案是用FormData来模拟表单中的<input type="file" id="avatar">控件
  ```
  
- ##### 基于图片上传

  ```
  <input type="file" name="file" id="file">
  <input type="submit" value="提交" onclick="ajaxupload()">
  
  <script>
      function ajaxupload() {
          var xhr = new XMLHttpRequest();                            // 创建XMLHttpRequest 对象
          var fm = new FormData()                                    // 创建表单
          var file_obj = document.getElementById('file').files[0]    // 获取上传的文件对象
          console.log(file_obj)                                      // 打印文件对象
          fm.append('file',file_obj)                                //  将文件对象添加到form 里面
          xhr.open('post', '/upload/', true);                       //  创建post 请求
          xhr.onreadystatechange = function () {                    //  请求成功执行回调函数
              if (xhr.readyState == 4) {                            //  服务期返回 状态码 4的时候  
                  console.log(xhr.responseText)                     //   打印服务器回调信息
              };
          };
          xhr.send(fm)                                              //   发送数据，请求中包含 文件
      }
  </script>
  ```

  views.py

  ```
  def upload(request):
      if request.method =='GET':
          return render(request,'upload.html')
      if request.method == 'POST':
          file = request.FILES.get('file')              # 获取文件信息用 request.FILES.get
          print(file)                                   # 这里的get('file') 相当于 name = file
          # print(file) 可以直接显示文件名，是因为django FILES内部 重写了 __repr__ 方法 
          if file:                                      # 如果文件存在
              with open(file.name,'wb') as f:               #  新建1张图片 ，图片名称为 上传的文件名
                  for temp in file.chunks():                #  往图片添加图片信息
                      f.write(temp)
          return HttpResponse('ok')
  ```



**实例**：使用sweetalert插件和Ajax动态删除数据

```js
 # 我使用的是模块继承，所以将js部分写在父级模块中，大概内容如下：
<script>
    $('.btn-danger').click(function () {
        var name = $(this).parent().attr('name');
        var id = $(this).parent().attr('pk');
        var title = name + id;
        var thi = $(this)[0];
        swal({
            title: "Are you sure?",
            text: "Once deleted, you will not be able to recover this imaginary file!",
            icon: "warning",
            buttons: true,
            dangerMode: true,
        })
            .then((willDelete) => {
                if (willDelete) {
                    $.ajax({
                        url: title,
                        success: function (res) {
                            if (res.result===0){
                                    console.log(thi)
                                    $(thi).parent().parent().parent().remove();
                                    swal("Poof! Your imaginary file has been deleted!", {
                                    icon: "success",
                                });
                            }
                        }
                    })
                } else {
                    swal("Your imaginary file is safe!");
                }
            });
    })
</script> 
```

对应的删除模块部分：

```html
<table class="table table-striped table-hover table-bordered">
                <thead>
                	<tr>
                    	<th>序号</th>
                    	<th>年级</th>
                    	<th>操作</th>
                	</tr>
                </thead>
                <tbody>
                {% for class in list %}
                    <tr>
                        <td> {{ forloop.counter }} </td>
                        <td> {{ class.name }} </td>
                        <td>
                            <a href="/app1/edit_class/?id={{ class.id }}">
                                <button class="btn btn-primary sm">编辑</button>
                            </a>
                            <a name="/app1/del_class/" pk ={{ class.id }}>
                                <button class="btn btn-danger sm">删除</button>
                            </a>
                        </td>
                    </tr>
                {% endfor %}
              </tbody>
</table>
```

