## Django模型进阶2



#### 一、创建模型对象

​	显然了解ORM模型后我们知道，创建模型对象就是创建一条记录，也就是说：class Model的一次实例化。在最初章节我们介绍了多种创建对象的方法，现在我们再进行一些补充和扩展：

- 1、在模型类中使用：classmethod，添加一个添加对象的类方法:

  ```python
  class Book(models.Model):
      title = models.CharField(max_length=100)
  	#创建类方法
      @classmethod
      def create(cls, title):
          book = cls(title=title)
          # do something with the book
          return book
      
      #创建实例方法
  	def create_book(self, title):
          book = self.create(title=title)
          # do something with the book
          return book
  book = Book.create("Pride and Prejudice")
  ```

- 2、在模型类中添加一个关系管理器（自定义的），官方推荐

  ```python
  class BookManager(models.Manager):
      def create_book(self, title):
          book = self.create(title=title)
          # do something with the book
          return book
  
  class Book(models.Model):
      title = models.CharField(max_length=100)
      objects = BookManager()  #这里相当于实例化了一个Manager对象，这就是我们平时使用的objects，也就是说我们平时使用的objects也是Manager的对象，里面自带了all、filter等方法。
  
  book = Book.objects.create_book("Pride and Prejudice")
  ```

  

#### 二、自定义模型加载

- classmethodModel.from_db（*db*，*field_names*，*values*)

  ```python
  from_db()从数据库加载时，该方法可用于自定义模型实例创建。
     1、 db：参数包含加载模型的数据库的数据库别名，
     2、 field_names：包含所有已加载字段的名称，包含每个field_names字段的加载值values 。field_names与values顺序相同。
     3、 values：保证按__init__()预期顺序排列 。
     
   #如果延迟任何字段，它们将不会出现 field_names。在这种情况下，django.db.models.DEFERRED 为每个缺少的字段分配值。除了创建新模型之外，该from_db()方法还必须在新实例的属性中设置 adding和db标志_state。 
  ```

- 下面是一个示例，说明如何记录从数据库加载的字段的初始值： 

  ```python
  from django.db.models import DEFERRED
  #cls是models.Model
  @classmethod
  def from_db(cls, db, field_names, values):
    #from_db()的默认实现(可能会更改，可以用super()代替)。
      if len(values) != len(cls._meta.concrete_fields):
          values = list(values)
          values.reverse()
          # 找到值就用值找不到就用默认值
          values = [
              values.pop() if f.attname in field_names else DEFERRED
              for f in cls._meta.concrete_fields
          ]
      #除了创建新模型之外，该from_db()方法还必须在新实例的属性中设置 adding和db标志_state。
      instance = cls(*values)
      instance._state.adding = False
      instance._state.db = db
      # 自定义以在实例上存储原始字段值
      instance._loaded_values = dict(zip(field_names, values))
      return instance
  
  def save(self, *args, **kwargs):
      # 检查当前值与._loaded_values的区别。例如，防止更改模型的creator_id。(本例不支持'creator_id'被延迟的情况)。
      if not self._state.adding and (
              self.creator_id != self._loaded_values['creator_id']):
          raise ValueError("Updating the value of creator isn't allowed")
      super().save(*args, **kwargs)
  ```

  

#### 三、刷新数据库中的对象

- 如果从模型实例中删除字段，则再次访问该字段会重新加载数据库中的值：

  ```
  >>> obj = MyModel.objects.first()
  >>> del obj.field
  >>> obj.field  # Loads the field from the database
  ```

- `Model.refresh_from_db`（*using = None*，*fields = None*）

  ```
  如果需要从数据库重新加载模型的值，则可以使用该 refresh_from_db()方法。在没有参数的情况下调用此方法时，将执行以下操作：
  	1、模型的所有非延迟字段都将更新为当前存在于数据库中的值。
  	2、从重新加载的实例中清除任何缓存的关系。
  ```

- 仅从数据库重新加载模型的字段。其他与数据库相关的值（如注释）不会重新加载。任何 @cached_property属性也不会被清除。
  **重新加载发生在加载实例的数据库中，如果未从数据库加载实例，则从默认数据库中重新加载**。该 using参数可用于强制用于重新加载的数据库。可以使用fields 参数强制加载字段集。
  例如，要测试update()调用是否导致了预期的更新，您可以编写类似于此的测试：

  ```python
  def test_update_result(self):
      obj = MyModel.objects.create(val=1)
      MyModel.objects.filter(pk=obj.pk).update(val=F('val') + 1)
     #在这一点上obj.val仍然是1，但是数据库中的值被更新为2。对象的更新值需要从数据库中重新加载。
      obj.refresh_from_db()
      self.assertEqual(obj.val, 2)
      
  #请注意，访问延迟字段时，通过此方法加载延迟字段的值。因此，可以自定义延迟加载的方式。下面的示例显示了在重新加载延迟字段时如何重新加载所有实例的字段：
  class ExampleModel(models.Model):
      def refresh_from_db(self, using=None, fields=None, **kwargs):
          # 字段包含要加载的延迟字段的名称。
          if fields is not None:
              fields = set(fields)
              deferred_fields = self.get_deferred_fields()
              # 如果要加载任何延迟字段
              if fields.intersection(deferred_fields):
                  # 然后把它们都装上
                  fields = fields.union(deferred_fields)
          super().refresh_from_db(using, fields, **kwargs)
  ```



#### 四、当你保存时，发生了什么？

当你保存一个对象时，Django 执行以下步骤：

​	1\. 发出一个`pre-save` 信号。 发送一个`django.db.models.signals.pre_save` 信号，以允许监听该信号的函数完成一些自定义的动作。

​	2\. 预处理数据。 如果需要，对对象的每个字段进行自动转换。

大部分字段不需要预处理 —— 字段的数据将保持原样。预处理只用于具有特殊行为的字段。例如，如果你的模型具有一个`auto_now=True` 的`DateField`，那么预处理阶段将修改对象中的数据以确保该日期字段包含当前的时间戳。（我们的文档还没有所有具有这种“特殊行为”字段的一个列表。）

​	3\. 准备数据库数据。 要求每个字段提供的当前值是能够写入到数据库中的类型。

大部分字段不需要数据准备。简单的数据类型，例如整数和字符串，是可以直接写入的Python 对象。但是，复杂的数据类型通常需要一些改动。

例如，`DateField` 字段使用Python 的 `datetime` 对象来保存数据。数据库保存的不是`datetime` 对象，所以该字段的值必须转换成ISO兼容的日期字符串才能插入到数据库中。

​	4\. 插入数据到数据库中。 将预处理过、准备好的数据组织成一个SQL 语句用于插入数据库。

​	5\. 发出一个post-save 信号。 发送一个`django.db.models.signals.post_save` 信号，以允许监听听信号的函数完成一些自定义的动作。

