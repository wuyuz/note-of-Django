## Django 面试题精选

1. 技术点

   1.1 使用Django自带的认证系统
   1.2 使用到cookie session来解决 http协议的无连接,无状态的的弊端
   1.3 使用form,modelform来验证用户输入的信息
   1.4 使用modelset来完成用户可以批量实现修改数据库的信息
   1.5 使用ajax,实现页面局部的实现
   1.6 使用rbac来实现的用户的权限的控制和用户页面的菜单的生成.
   1.7 验证码
   1.8 自定义标签
   1.9 中间件
   1.10 模板
   1.11 json 使用,在登录中使用json返回登录状态的信息

2. rbac的流程

   2.1 表结构的设计
   			用户表 用户表和角色表是多对多
   			权限表 权限表和角色表是多对多
   			角色表 
   			菜单表 菜单表和权限表是一对多

   ​			在用户登录成功后将用户的权限和菜单的信息注入session中

   ​			session注入
   ​			在session输入的时候组成两个数据结构

   ```
   persission_list = [{
   'url':item['url'],
   'title':item['title'],
   'pk':item['pk'],
   'pid':item['pid']
   }]
   
   menu_dic菜单的信息 
   if item['menu__pk']: 
   menu_dic = menu_dic[item['menu__pk']]={
   'title':item['menu__title'],
   'icon':item['menu__icon'],
   'children':[
   {
   'title':item['title'],
   'url': item['url'],
   'pid':item['pid']
   }
   ]
   } 
   ```

   



中间件
自定义的标签得到菜单的信息,用inclusion_tag来渲染到模板中

项目的描述:
客户管理系统
网咨
销售
客户到学生(缴费)
学生的管理系统
学生信息的管理



中间件
process_request 在url分发之前

process_response 在视图函数执行之后

process_view 在url分发之后,视图之前

process_exception 视图函数中出现异常

process_template_response 是在视图函数执行完成后立即执行，但是它有一个前提条件，那就是视图函数返回的对象有一个render()方法（或者表明该对象是一个TemplateResponse对象或等价方法）。


在setting中 配置添加应用
INSTALLED_APPS 


修改的连接的数据库:
DATABASES 的配置 


modelformset
modelform不能实现我们批量修改的需求
使用了modelformset

1 在写了一modelform的类
2 model=models.StudentStudyRecord 批量修改的表
form=form.StudyRecordDeialModelForm 是写modelform中类

form_set_obj = modelformset_factory(model=models.StudentStudyRecord,
form=form.StudyRecordDeialModelForm,extra=0) 


3 formset = form_set_obj(queryset=all_study_recored) 

4 html表中循环 instance 加这个不可以修改
form.instance.student 

Django的生命周期:

模板的继承

自己写好模板,在继承需要修改的地方,用block快括起来并起名字
在继承的时候{%extends 模板名%}

{% block content %}
修改的内容
{% endblock content %}



多筛选条件

将用户的所有信息返回给前端

1 使用cookie session,将用户的信息放在session中




公户和私户的转换

私户转公户 1销售自己的能力不行 2销售离职了 3 客户不想和这个销售聊了 




常见的状态码:
2开头 （请求成功） 
200 （成功）
201 （已创建）
3开头 （请求被重定向）
4开头 （请求错误）

5开头（服务器错误）


session默认的周期,是14天

可以在setting中配置session的配置


ajax 是用户的提前

ajax 请求前做一些事情



from组件 
1 在模板中渲染
2 用户信息的校验
3 在用户输入错误时,将用户输入错误的信息,返回在前端



为什么不买? 



上线问题?
开发的时间


mysql的5.6


上传EXE表,将用户上传的文件,写进数据库.使用xlrd 模块


RBAC 基于角色的权限访问控制