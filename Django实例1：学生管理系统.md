## Django实例1：学生管理系统

#### 总路由：

- 已经从总路由中分发了路由到app中：

  ```python
  from django.conf.urls import url
  from app1 import views
  
  urlpatterns = [
      url(r'^student_list/$',views.student_list),
      url(r'^add_student/$',views.add_student),
      url(r'^del_student/$',views.del_student),
      url(r'^edit_student/$',views.edit_student),
      
      url(r'^class_list/$',views.class_list),
      url(r'^add_class/$',views.add_class),
      url(r'^edit_class/$',views.edit_class),
      url(r'^del_class/$',views.del_class),
      
      url(r'^teacher_list/$',views.teacher_list),
      url(r'^add_teacher/$',views.add_teacher),
      url(r'^del_teacher/$',views.del_teacher),
      url(r'^edit_teacher/$',views.edit_teacher),
  ]
  ```



#### 模型表

- 注意表与表的关系

  ```python
  from django.db import models
  
  class Class(models.Model):
      id = models.AutoField(primary_key=True)
      name = models.CharField(max_length=12)
          
  class Student(models.Model):
      id = models.AutoField(primary_key=True)
      name = models.CharField(max_length=32)
      classes = models.ForeignKey('Class')
          
  class Teacher(models.Model):
      name = models.CharField(max_length=32)
      classes = models.ManyToManyField('Class')
  ```



#### 总视图函数：

```python
from django.shortcuts import render,redirect,HttpResponse
from app1 import  models

def student_list(request):
    student_obj = models.Student.objects.all()
    return render(request,'student_list.html',{'list':student_obj})

def add_student(request):
    error_msg = ''
    class_obj_list = models.Class.objects.all()
    if request.method == 'POST':
        student_name = request.POST.get('student')
        class_id = request.POST.get('class_id')
        if student_name:
            models.Student.objects.create(name = student_name,classes_id=class_id)
            return  redirect('/app1/student_list/')            
        else:
            error_msg = '信息不能为空'                     
    return render(request,'add_student.html',{'error_msg':error_msg,'list':class_obj_list})

def del_student(request):   
    student_id = request.GET.get('id')
    del_list = models.Student.objects.filter(id = student_id)
    if del_list:
        del_list.delete()
        return  redirect('/app1/student_list/')
    else:
        return HttpResponse('删除错误')
        
def edit_student(request):
    error = ''
    student_id = request.GET.get('id')
    student_obj = models.Student.objects.filter(id = student_id).first()
    class_list= models.Class.objects.all()
    if request.method == 'POST':
        student_name = request.POST.get('student')
        class_id = request.POST.get('class_id')
        if student_name:
            if models.Student.objects.filter(name =  student_name):
                error = '班级信息已创建，请重新输入'
            else:
                student_obj.name = student_name
                student_obj.classes_id = class_id
                student_obj.save()
                return redirect('/app1/student_list/')            
        else: error = '班级信息不能为空'       
    return render(request, 'edit_student.html', {'error':error, 'list':class_list,'student_obj':student_obj})
    
    
 -------------------------------------- 教室 ---------------------------
def class_list(request):
    class_obj = models.Class.objects.all()
    return render(request, 'class_list.html',{'list':class_obj})

def add_class(request):
    error = ''
    if request.method == 'POST':
        class_name = request.POST.get('class')
        if class_name:
            if models.Class.objects.filter(name = class_name):
                error = '班级信息已经存在'      
            else:
                models.Class.objects.create(name = class_name)
                return redirect('/app1/class_list/')      
        else:
            error = '班级信息不能为空'            
    return render(request, 'add_class.html', {'error':error})

def edit_class(request):
    error = ''
    class_id = request.GET.get('id')
    class_obj = models.Class.objects.filter(id = class_id).first()   
    if request.method == 'POST':
        class_name = request.POST.get('class')
        if class_name:
            if models.Class.objects.filter(name = class_name):
                error = '班级信息已创建'
            else:
                class_obj.name = class_name
                class_obj.save()
                return redirect('/app1/class_list/')       
        else:
            error = '信息不能为空'        
    return render(request, 'edit_class.html', {'list':class_obj,'error':error})
        
def del_class(request):
    class_id = request.GET.get('id')
    del_list = models.Class.objects.filter(id = class_id)    
    if del_list:
        del_list.delete()
        return redirect('/app1/class_list')
    else:
        return HttpResponse('删除错误')
    
 ----------------------------------------老师-----------------------------------
def teacher_list(request):
    teacher_obj_list = models.Teacher.objects.all()
    return render(request,'teacher_list.html',{'list':teacher_obj_list})

def add_teacher(request):
    error = ''
    class_obj_list = models.Class.objects.all()
    if request.method == 'POST':
        teacher_name = request.POST.get('teacher')
        class_ids = request.POST.getlist('class_id')
        if teacher_name:
            new_teacher_obj = models.Teacher.objects.create(name=teacher_name)
            new_teacher_obj.classes.set(class_ids) # set加列表，给这个关系管理表重新赋值
            return redirect('/app1/teacher_list/')
        else:            
            error = '老师姓名不能为空'
    return render(request,'add_teacher.html',{'list':class_obj_list,'error':error})
    
def del_teacher(request):
    teacher_id = request.GET.get('id')
    del_list = models.Teacher.objects.filter(id = teacher_id)
    if del_list:
        del_list.delete()
        return redirect('/app1/teacher_list/')
    else:
        return HttpResponse('删除错误')
      
def edit_teacher(request):
    teacher_id = request.GET.get('id')
    error = ''
    teacher_obj = models.Teacher.objects.filter(id = teacher_id).first()
    classes_list = models.Class.objects.all()
    if request.method == 'POST':
        teacher_name = request.POST.get('teacher_name')
        class_ids = request.POST.getlist('class_ids')
        if teacher_name:
            teacher_obj.name = teacher_name
            teacher_obj.classes.set(class_ids)
            teacher_obj.save()
            return redirect('/app1/teacher_list/')
        else:
            error = '老师姓名不能为空'        
    return render(request,'edit_teacher.html',{'list':classes_list,'error':error,'teacher_obj':teacher_obj})
        
```



#### 总导航模块

```html
#base.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <link rel="stylesheet" href="/static/bootstrap-3.3.7-dist/css/bootstrap.min.css">
    <link rel="stylesheet" href="/static/css/dashboard.css">
    <style>
        table {
            text-align: center;
        }

        th {
            text-align: center;
        }
    </style>
</head>
<body>
<body>
<nav class="navbar navbar-inverse navbar-fixed-top">
    <div class="container-fluid">
        <div class="navbar-header">
            <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#navbar"
                    aria-expanded="false" aria-controls="navbar">
                <span class="sr-only">Toggle navigation</span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
            </button>
            <a class="navbar-brand" href="#">学校管理系统</a>
        </div>
        <div id="navbar" class="navbar-collapse collapse">
            <ul class="nav navbar-nav navbar-right">
                <li><a href="#">Dashboard</a></li>
                <li><a href="#">Settings</a></li>
                <li><a href="#">Profile</a></li>
                <li><a href="#">Help</a></li>
            </ul>
            <form class="navbar-form navbar-right">
                <input type="text" class="form-control" placeholder="Search...">
            </form>
        </div>
    </div>
</nav>

<div class="container-fluid">
    <div class="row">
        <div class="col-sm-3 col-md-2 sidebar">
            {% block bar %}
                <ul class="nav nav-sidebar">
                    <li class="active"><a href="/app1/student_list/">学生管理 <span class="sr-only">(current)</span></a>
                    </li>
                    <li><a href="/app1/class_list/">教室管理</a></li>
                    <li><a href="/app1/teacher_list/">老师管理</a></li>
                    {#                <li><a href="#">Export</a></li>#}
                </ul>
            {% endblock %}
        </div>
        <div class="col-sm-6 col-sm-offset-4 col-md-6 col-md-offset-3 main">

            {% block content %}

            {% endblock %}
        </div>
    </div>
</div>

<!-- Placed at the end of the document so the pages load faster -->
<script src="https://cdn.bootcss.com/jquery/1.12.4/jquery.min.js"></script>
<script>window.jQuery || document.write('<script src="../../assets/js/vendor/jquery.min.js"><\/script>')</script>
<script src="https://cdn.bootcss.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
<script>
    $("li > a").on('click', function () {
        $(this).parent().addClass('active').siblings().removeClass('active');
    });

</script>

</body>
</body>
</html>
```



#### 总编辑模块

```html
#base1.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <link rel="stylesheet" href="/static/bootstrap-3.3.7-dist/css/bootstrap.min.css">
    <link rel="stylesheet" href="/static/css/dashboard.css">
</head>
<body>
<div class="container">
    <class class="row">
        <class class="col-lg-8 col-lg-offset-2 col-sm-8 col-sm-offset-2">
            <div class="panel panel-primary">
                <div class="panel-heading">
                    {% block head %}

                    {% endblock %}
                </div>
                <div class="panel-body">
                    {% block content %}

                    {% endblock %}
                </div>
            </div>
        </class>
    </class>
</div>
</body>
</html>
```



#### 班级表模块综合

```html
#add_class.html文件
{% extends 'base1.html' %}
{% block head %}
    <h3 class="panel-title">新增班级</h3>
{% endblock %}

{% block content %}
    <div class="panel-body">
        <form class="form-horizontal" action="" method="post">
            <div class="form-group">
                <label for="inputEmail3" class="col-sm-2 control-label">班级</label>
                <div class="col-sm-10">
                    <input type="text" class="form-control" name="class" placeholder="班级名">
                </div>
            </div>

            <div class="form-group">
                <div class="col-sm-offset-2 col-sm-10">
                    {{ error}}
                    <button type="submit" class="btn btn-primary center-block">提交</button>
                </div>
            </div>
        </form>
    </div>
{% endblock %}

-----------------------------------------------------------------------
#class_list.html文件
{% extends 'base.html' %}
{% block bar %}
    <ul class="nav nav-sidebar">
        <li><a href="/app1/student_list/">学生管理 <span class="sr-only">(current)</span></a></li>
        <li class="active"><a href="/app1/class_list/">课程管理</a></li>
        <li><a href="/app1/teacher_list/">老师管理</a></li>
    </ul>
{% endblock %}

{% block content %}
    <div class="panel panel-primary">
        <div class="panel-heading" style="height:60px;line-height: 60px;">
            <h2 class="panel-title" style="font-size: 20px;">教室管理系统</h2>
        </div>
        <div class="panel-body">
            <a href="/app1/add_class/">
                <button class="btn btn-warning btn-group-sm">新增</button>
            </a>
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
                            <a href="/app1/del_class/?id={{ class.id }}">
                                <button class="btn btn-danger sm">删除</button>
                            </a>
                        </td>
                    </tr>

                {% endfor %}
                </tbody>
            </table>
        </div>
    </div>
{% endblock %}

-------------------------------------------------------------
#edit_class.html文件
{% extends 'base1.html' %}
{% block head %}
    <h3 class="panel-title">新增学生</h3>
{% endblock %}

{% block content %}

    <div class="panel-body">
        <form class="form-horizontal" action="" method="post">
            <div class="form-group">
                <label for="inputEmail3" class="col-sm-2 control-label">班级</label>
                <div class="col-sm-10">
                    <input type="text" class="form-control" name="class" placeholder="{{ list.name }}">

                </div>
            </div>

            <div class="form-group">
                <div class="col-sm-offset-2 col-sm-10">
                    {{ error }}
                    <button type="submit" class="btn btn-primary center-block">提交</button>
                </div>
            </div>
        </form>
    </div>
{% endblock %}
```



#### 学生表

```html
#student_list.html文件
{% extends 'base.html' %}
{% block content %}
    <div class="panel panel-primary">
        <div class="panel-heading" style="height:60px;line-height: 60px;">
            <h2 class="panel-title" style="font-size: 20px;">学生管理系统</h2>
        </div>
        <div class="panel-body">
            <a href="/app1/add_student/"><button class="btn btn-warning btn-group-sm">新增</button></a>
            <table class="table table-striped table-hover table-bordered">
                <thead>
                <tr>
                    <th>序号</th>
                    <th>姓名</th>
                    <th>班级</th>
                    <th>操作</th>
                </tr>
                </thead>
                <tbody>
                {% for student in list %}
                    <tr>
                        <td> {{ forloop.counter }} </td>
                        <td> {{ student.name }} </td>
                        <td> {{ student.classes.name }} </td>
                        <td>
                            <a href="/app1/edit_student/?id={{ student.id }}">
                                <button class="btn btn-primary sm">编辑</button>
                            </a>
                            <a href="/app1/del_student/?id={{ student.id }}">
                                <button class="btn btn-danger sm">删除</button>
                            </a>
                        </td>
                    </tr>

                {% endfor %}
                </tbody>
            </table>
        </div>
    </div>
{% endblock %}

----------------------------------------------------------------------
#add_student.html文件
{% extends 'base1.html' %}
{% block head %}
    <h3 class="panel-title">修改学生</h3>
{% endblock %}

{% block content %}

    <div class="panel-body">
        <form class="form-horizontal" action="" method="post">
            <div class="form-group">
                <label for="1" class="col-sm-2 control-label">姓名</label>
                <div class="col-sm-10">
                    <input type="text" id='1' class="form-control" name="student" placeholder="{{ student_obj.name }}">
                </div>
                <label for="class" class="col-sm-2 control-label">班级</label>
                <div class="col-sm-10" style="margin-top:10px;">
                    <select name="class_id" id="class">
                        {% for class in list %}
                            <option value="{{ class.id }}">{{ class.name }}</option>
                        {% endfor %}
                    </select>
                </div>
            </div>

            <div class="form-group">
                <div class="col-sm-offset-2 col-sm-10 text-danger">
                    {{ error_msg }}
                    <button type="submit" class="btn btn-primary center-block">提交</button>
                </div>
            </div>
        </form>
    </div>
{% endblock %}

---------------------------------------------------------------------------
#edit_student.html文件
{% extends 'base1.html' %}
{% block head %}
    <h3 class="panel-title">修改学生</h3>
{% endblock %}

{% block content %}

    <div class="panel-body">
        <form class="form-horizontal" action="" method="post">
            <div class="form-group">
                <label for="1" class="col-sm-2 control-label">姓名</label>
                <div class="col-sm-10">
                    <input type="text" id='1' class="form-control" name="student" placeholder="{{ student_obj.name }}">
                </div>
                <label for="class" class="col-sm-2 control-label">班级</label>
                <div class="col-sm-10" style="margin-top:10px;">
                    <select name="class_id" id="class">
                        {% for class in list %}
                            {% if student_obj.classes_id == class.id %}
                                <option selected value="{{ class.id }}">{{ class.name }}</option>
                            {% else %}
                                <option value="{{ class.id }}">{{ class.name }}</option>
                            {% endif %}
                        {% endfor %}
                    </select>
                </div>
            </div>

            <div class="form-group">
                <div class="col-sm-offset-2 col-sm-10">
                    {{ error }}
                    <button type="submit" class="btn btn-primary center-block">提交</button>
                </div>
            </div>
        </form>
    </div>
{% endblock %}
```



#### 教师表

```html
#teacher_list.html文件
{% extends 'base.html' %}
{% block bar %}
    <ul class="nav nav-sidebar">
        <li><a href="/app1/student_list/">学生管理 <span class="sr-only">(current)</span></a></li>
        <li><a href="/app1/class_list/">课程管理</a></li>
        <li class="active"><a href="/app1/teacher_list/">老师管理</a></li>
    </ul>
{% endblock %}
{% block content %}
    <div class="panel panel-primary">
        <div class="panel-heading" style="height:60px;line-height: 60px;">
            <h2 class="panel-title" style="font-size: 20px;">教师管理系统</h2>
        </div>
        <div class="panel-body">
            <a href="/app1/add_teacher/">
                <button class="btn btn-warning btn-group-sm">新增</button>
            </a>
            <table class="table table-striped table-hover table-bordered">
                <thead>
                <tr>
                    <th>序号</th>
                    <th>老师姓名</th>
                    <th>所教班级</th>
                    <th>操作</th>
                </tr>
                </thead>
                <tbody>
                {% for obj in list %}
                    <tr>
                        <th>{{ obj.id }}</th>
                        <td>{{ obj.name }}</td>
                        <td>
                            {% for class in obj.classes.all %}
                                {% if forloop.last %}
                                      < {{ class.name }} >
                                {% else %}
                                    < {{ class.name }} >、
                                {% endif %}
                            {% endfor %}
                        </td>
                        <td>
                            <a href="/app1/edit_teacher/?id={{ obj.id }}">
                                <button class="btn btn-primary sm">编辑</button>
                            </a>
                            <a href="/app1/del_teacher/?id={{ obj.id }}">
                                <button class="btn btn-danger sm">删除</button>
                            </a>
                        </td>
                    </tr>
                {% endfor %}
                </tbody>
            </table>
        </div>
    </div>
{% endblock %}

-----------------------------------------------------------------------------------
#add_teacher.html文件
{% extends 'base1.html' %}
{% block head %}
    <h3 class="panel-title">添加老师</h3>
{% endblock %}

{% block content %}
    <div class="panel-body">
        <form class="form-horizontal" action="" method="post">
            <div class="form-group">
                <label for="1" class="col-sm-2 control-label">姓名</label>
                <div class="col-sm-10">
                    <input type="text" id='1' class="form-control" name="teacher" placeholder="{{ student_obj.name }}">
                </div>
                <label for="class" class="col-sm-2 control-label">班级</label>
                <div class="col-sm-10" style="margin-top:10px;">
                    <select name="class_id" id="class" multiple style="height:20px;">
                        {% for class in list %}
                            <option value="{{ class.id }}">{{ class.name }}</option>
                        {% endfor %}
                    </select>
                </div>
            </div>

            <div class="form-group">
                <div class="col-sm-offset-2 col-sm-10 text-danger">
                    {{ error }}
                    <button type="submit" class="btn btn-primary center-block">提交</button>
                </div>
            </div>
        </form>
    </div>

<script src="https://cdn.bootcss.com/jquery/1.12.4/jquery.min.js"></script>
<script>
    $('select').hover(
        function () {
            $(this).css('height', '60px')
        },
        function () {
            $(this).css('height', '20px')
        }
    );
</script>
{% endblock %}

----------------------------------------------------------------------------------
#edit_teacher.html文件
{% extends 'base1.html' %}
{% block head %}
    <h3 class="panel-title">添加老师</h3>
{% endblock %}

{% block content %}

    <div class="panel-body">
        <form class="form-horizontal" action="" method="post">
            <div class="form-group">
                <label for="1" class="col-sm-2 control-label">姓名</label>
                <div class="col-sm-10">
                    <input type="text" id='1' class="form-control" name="teacher" placeholder="{{ student_obj.name }}">
                </div>
                <label for="class" class="col-sm-2 control-label">班级</label>
                <div class="col-sm-10" style="margin-top:10px;">
                    <select name="class_id" id="class" multiple style="height:20px;">
                        {% for class in list %}
                            <option value="{{ class.id }}">{{ class.name }}</option>
                        {% endfor %}
                    </select>
                </div>
            </div>

            <div class="form-group">
                <div class="col-sm-offset-2 col-sm-10 text-danger">
                    {{ error }}
                    <button type="submit" class="btn btn-primary center-block">提交</button>
                </div>
            </div>
        </form>
    </div>

<script src="https://cdn.bootcss.com/jquery/1.12.4/jquery.min.js"></script>
<script>
    $('select').hover(
        function () {
            $(this).css('height', '60px')
        },
        function () {
            $(this).css('height', '20px')
        }
    );
</script>
{% endblock %}

```





##### 注意点:

```
1、提交POST请求前注释掉settings.py文件中45行上下的 'django.middleware.csrf.CsrfViewMiddleware', 或者在HTML代码的form标签内加上 {% csrf_token %} 的标签。二者选一即可提交POST请求。另：可在class_list.html的代码中添加一个a标签： <a href="/add_class/">新页面添加</a> 

2、提交POST请求前注释掉settings.py文件中45行上下的 'django.middleware.csrf.CsrfViewMiddleware', 或者在HTML代码的form标签内加上 {% csrf_token %}的标签。二者选一即可提交POST请求。另：可在student_list.html的代码中添加一个a标签： <a href="/add_student/">新页面添加</a> 

```

