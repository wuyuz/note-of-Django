## Django之错误收集

- ##### 1、AttributeError: 'QuerySet' object has no attribute '_meta'

  ```python
   # 此原因出现在将实例传入Form类中时，如果传入的时QuerySet对象，就会报错。
   # 解决办法：使用first()/get() 取出具体的某一个元素
   
  def consultant_change(request, pk=None):
      obj = models.ConsultRecord.objects.filter(pk=pk).first()
      form_obj = ConsultantForm(instance=None)    
      title = '编辑更进记录' if pk else '新增跟进记录'
      return render(request,'form.html',{'form_obj':form_obj,'title':title})
   
  ```

- ##### 2、NOT NULL constraint failed: （某一张表的字段）

  ```
   # 此原因出现在数据保存时，数据库中的非空约束添加的问题，查看数据库字段
   # 解决方案： 1、设置该表的自增字段，或检查字段长度
   			2、确认是否保存数据的表不是该表
  ```

- 3、

