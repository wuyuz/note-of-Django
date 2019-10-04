## Django 导入积累

- ##### CSRF导入相关

  ```
  from django.utils.decorators import method_decorator # 用于装饰CBV
  
  from django.views.decorators.csrf import csrf_exempt,csrf_protect,ensure_csrf_cookie  # csrf放行、强制要求csrf以及确保生成cookie
  ```

- ##### 返回HttpResonse对象相关

  ```
  from django.http.response import JsonResponse  # 返回JsonResponse
  from django.shortcuts import render,HttpResponse,redirect,reverse # 三剑客
  ```

- ##### 导入全局设置

  ```
  from django.conf import global_settings  # 导入全局变量
  from django.conf import settings  # 导入settings文件中的大写变量
  ```

- ##### URL路径相关

  ```
  from django.urls import reverse
  from django.conf.urls import url,include # 路由和分发函数
  from django.urls import path
  ```

- ##### 自定义标签、过滤器、inclusion_tag

  ```python
  from django import template
  from django.utils.safestring import mark_safe  # 这个相当于django自带的safe,因为自定义的标签不能使用自带的safe，只能用他
  register = template.Library()
  
  @register.filter
  def func(value, arg):pass
  
  @register.simple_tag   #表示将函数转换为自定义标签
  def multi_tag(x,y,z):
      return x*y*z
  
  @register.inclusion_tag('result.html')  #将result.html里面的内容用下面函数的返回值渲染，然后作为一个组件一样，加载到使用这个函数的html文件里面
  def show_results(n): #参数可以传多个进来
      n = 1 if n < 1 else int(n)
      data = ["第{}项".format(i) for i in range(1, n+1)]
      return {"data": data}    #这里可以穿多个值，和render的感觉是一样的{'data1':data1,'data2':data2....}
  ```

- ##### 中间件相关

  ```
  from django.utils.deprecation import MiddlewareMixin  # 导入中间件父类
  ```

- ##### Models相关

  ```
  from django.db import models   # 导入模型models
  ```

- ##### 表查询相关

  ```
  from django.db.models import Avg,Count,Max,Min
  from django.db.models import F,Q  # 表中字段比较更改/filter查询或添加
  ```

- ##### Python脚本调用Django环境

  ```
  import os
  
  if __name__ == '__main__':
  	os.environ.setdefault(DJANGO_SETTINGS_MODULE", "BMS.settings")  # 第二个参数时setting文件名，可在manage.py中找到此句
  	import django
  	django.setup()
  	
  	from app import models
  	...
  ```

- ##### 视图函数相关

  ```
  from django.view import View  # 导入CBV的父类
  ```

- ##### form组件相关

  ```python
  from django import forms  # 导入forms组件
  
  class ReForm(forms.Form): pass # 手写字段
  def register2(request,pk=None):
      obj = models.xxx.object.filter().first()
      form_obj = RegForm(instance=obj)
      if request.method == "POST":
          form_obj = RegForm(data=request.POST,instance=obj) 
          if form_obj.is_valid():
              return HttpResponse("注册成功") 
      return render(request, "register2.html", {"form_obj": form_obj})
  --------------------------------------------
  
  from django.core.exceptions import ValidationError # 导入错误类
  ```

- ##### 事务相关

  ```python
  from django.db import transaction  # 导入事务
  
   try:
       with transaction.atomic():
           generate_relationships()
   except IntegrityError:
         handle_exception()
  ```

- ##### admin相关

  ```python
  from django.contrib import admin # 在admin.py中
  from rbac import models
  
  class PermissionConfig(admin.ModelAdmin):
      list_display = ['id', 'url', 'title','menu']
      list_editable = ['url', 'title','menu']
  
  admin.site.register(models.Permission, PermissionConfig) # 配置可编辑的网页
  admin.site.register(models.Role)
  
  ------------------------------------------------
   # 在url.py文件中
  from django.contrib import admin    
  ```

- ##### forms相关进阶

  ```python
  from django import forms  # 导入forms
  from django.forms import modelformset_factory  # 导入可编辑的modelform
  
  ModeFormSet = modelformset_factory(models.StudyRecord, StudyRecordForm,extra=0)
      form_set_obj = ModeFormSet(queryset=models.StudyRecord.objects.filter(course_record_id=course_record_id))
      
      if request.method == 'POST':
          form_set_obj = ModeFormSet(queryset=models.StudyRecord.objects.filter(course_record_id=course_record_id),data=request.POST)
          if form_set_obj.is_valid():
              form_set_obj.save()
              return redirect(reverse('study_record',args=(course_record_id,)))
              
      return render(request,'teacher/study_record_list.html',{'form_set_obj':form_set_obj})
  ```

  