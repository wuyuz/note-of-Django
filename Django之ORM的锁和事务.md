## Django之ORM的锁和事务



#### 一、锁

- **行级锁：select_for_update(nowait=False, skip_locked=False)** 

  - 注意必须用在事务里面，至于如何开启事务，我们看下面的事务一节。返回一个锁住行直到事务结束的查询集，如果数据库支持，它将生成一个 SELECT ... FOR UPDATE 语句。

    ```python
    entries = Entry.objects.select_for_update().filter(author=request.user)  
    #加互斥锁，由于mysql在查询时自动加的是共享锁，所以我们可以手动加上互斥锁。create、update、delete操作时，mysql自动加行级互斥锁
    ```

  - 所有匹配的行将被锁定，直到事务结束。这意味着可以通过锁防止数据被其它事务修改。

    　　一般情况下如果其他事务锁定了相关行，那么本查询将被阻塞，直到锁被释放。 如果这不想要使查询阻塞的话，使用select_for_update(nowait=True)。 如果其它事务持有冲突的锁，互斥锁, 那么查询将引发 DatabaseError 异常。你也可以使用select_for_update(skip_locked=True)忽略锁定的行。 nowait和skip_locked是互斥的，同时设置会导致ValueError。

    > ```
    > 目前，postgresql，oracle和mysql数据库后端支持select_for_update()。 但是，MySQL不支持nowait和skip_locked参数。使用不支持这些选项的数据库后端（如MySQL）将nowait=True或skip_locked=True转换为select_for_update()将导致抛出DatabaseError异常，这可以防止代码意外终止。
    > ```

    

- **表级锁**：

  ```python
  class LockingManager(models.Manager):
      """ Add lock/unlock functionality to manager.
  
      Example::
  
          class Job(models.Model): #其实不用这么负载，直接在orm创建表的时候，给这个表定义一个lock和unlock方法，借助django提供的connection模块来发送锁表的原生sql语句和解锁的原生sql语句就可以了，不用外层的这个LckingManager(model.Manager)类
  
              manager = LockingManager()
  
              counter = models.IntegerField(null=True, default=0)
  
              @staticmethod
              def do_atomic_update(job_id)
                  ''' Updates job integer, keeping it below 5 '''
                  try:
                      # Ensure only one HTTP request can do this update at once.
                      Job.objects.lock()
  
                      job = Job.object.get(id=job_id)
                      # If we don't lock the tables two simultanous
                      # requests might both increase the counter
                      # going over 5
                      if job.counter < 5:
                          job.counter += 1                                        
                          job.save()
  
                  finally:
                      Job.objects.unlock()
  
  
      """    
  
      def lock(self):
          """ Lock table. 
  
          Locks the object model table so that atomic update is possible.
          Simulatenous database access request pend until the lock is unlock()'ed.
  
          Note: If you need to lock multiple tables, you need to do lock them
          all in one SQL clause and this function is not enough. To avoid
          dead lock, all tables must be locked in the same order.
  
          See http://dev.mysql.com/doc/refman/5.0/en/lock-tables.html
          """
          cursor = connection.cursor()
          table = self.model._meta.db_table
          logger.debug("Locking table %s" % table)
          cursor.execute("LOCK TABLES %s WRITE" % table)
          row = cursor.fetchone()
          return row
  
      def unlock(self):
          """ Unlock the table. """
          cursor = connection.cursor()
          table = self.model._meta.db_table
          cursor.execute("UNLOCK TABLES")
          row = cursor.fetchone()
          return row  
  ```

  

#### 二、事务

​		关于MySQL的事务处理，我的mysql博客已经说的很清楚了，那么我们来看看Django是如果做事务处理的。django1.8版本之前是有很多种添加事务的方式的，中间件的形式（全局的）、函数装饰器的形式，上下文管理器的形式等，但是很多方法都在1.8版之后给更新了，下面我们只说最新的：

- **全局开启**：在Web应用中，常用的事务处理方式是将每个请求都包裹在一个事务中。这个功能使用起来非常简单，你只需要将它的配置项ATOMIC_REQUESTS设置为True。它是这样工作的：当有请求过来时，Django会在调用视图方法前开启一个事务。如果请求却正确处理并正确返回了结果，Django就会提交该事务。否则，Django会回滚该事务。

  ```python
  DATABASES = {
      'default': {
          'ENGINE': 'django.db.backends.mysql',
          'NAME': 'mxshop',
          'HOST': '127.0.0.1',
          'PORT': '3306',
          'USER': 'root',
          'PASSWORD': '123',
          'OPTIONS': {
              "init_command": "SET default_storage_engine='INNODB'",
  　　　　　　　#'init_command': "SET sql_mode='STRICT_TRANS_TABLES'", #配置开启严格sql模式
  
          }
          "ATOMIC_REQUESTS": True, #全局开启事务，绑定的是http请求响应整个过程
          "AUTOCOMMIT":False, #全局取消自动提交，慎用
      }，
  　　'other':{
  　　　　'ENGINE': 'django.db.backends.mysql', 
              ......
  　　} #还可以配置其他数据库
  }
  
  ---------------------------------------------------------------------
  #上面这种方式是统一个http请求对应的所有sql都放在一个事务中执行（要么所有都成功，要么所有都失败）。是全局性的配置， 如果要对某个http请求放水（然后自定义事务），可以用non_atomic_requests修饰器，那么他就不受事务的管控了
  from django.db import transaction
  @transaction.non_atomic_requests
  def my_view(request):
      do_stuff()
  
  @transaction.non_atomic_requests(using='other')
  def my_other_view(request):
      do_stuff_on_the_other_database()
  ```

  - 但是Django 文档中说，不推荐这么做。因为如果将事务跟 HTTP 请求绑定到一起的时，然而view 是依赖于应用程序对数据库的查询语句效率和数据库当前的锁竞争情况。当流量上来的时候，性能会有影响，知道一下就行了。所以推荐用下面这种方式，通过 `transaction.atomic` 来更加明确的控制事务。atomic允许我们在执行代码块时，在数据库层面提供原子性保证。 如果代码块成功完成， 相应的变化会被提交到数据库进行commit；如果执行期间遇到异常，则会将该段代码所涉及的所有更改回滚。



**局部使用事务**：`atomic(*using=None, savepoint=True*)[source]`  

- 参数：using='other',就是当你操作其他数据库的时候，这个事务才生效，看上面我们的数据库配置，除了default，还有一个other，默认的是default。savepoint的意思是开启事务保存点，推荐看一下我[数据库博客](https://www.cnblogs.com/clschao/articles/10065275.html)里面的事务部分关于保存点的解释。
- 原子性是数据库事务的一个属性。使用atomic，我们就可以创建一个具备原子性的代码块。一旦代码块正常运行完毕，所有的修改会被提交到数据库。反之，如果有异常，更改会被回滚。　　　

- 被atomic管理起来的代码块还可以内嵌到方法中。这样的话，即便内部代码块正常运行，如果外部代码块抛出异常的话，它也没有办法把它的修改提交到数据库中。



- **用法一**：给函数做装饰器来使用　

  ```python
  from django.db import transaction
  
  @transaction.atomic
  def viewfunc(request):
      # This code executes inside a transaction.
      do_stuff()
  ```

- **用法二**：作为上下文管理器来使用，其实就是设置事务的保存点

  ```python
  from django.db import transaction
  
  def viewfunc(request):
      # This code executes in autocommit mode (Django's default).
      do_stuff()
  
      with transaction.atomic():   #保存点
          # This code executes inside a transaction.
          do_more_stuff()
  
      do_other_stuff()
      
  ------------------------------------------------------
  #一旦把atomic代码块放到try/except中，完整性错误就会被自然的处理掉了，比如下面这个例子：
  
  from django.db import IntegrityError, transaction
  @transaction.atomic
  def viewfunc(request):
      create_parent()
  
      try:
          with transaction.atomic():
              generate_relationships()
      except IntegrityError:
          handle_exception()
  
      add_children()
  ```

  

- 用法三**：还可以嵌套使用，函数的事务嵌套上下文管理器的事务，上下文管理器的事务嵌套上下文管理器的事务等。下面的是函数嵌套上下文的例子：

  ```python
  from django.db import IntegrityError, transaction
  
  @transaction.atomic
  def viewfunc(request):
      create_parent()
  
      try:
          with transaction.atomic():
              generate_relationships()
  　　　　　　　#other_task()  #还要注意一点，如果你在事务里面写了别的操作，只有这些操作全部完成之后，事务才会commit，也就是说，如果你这个任务是查询上面更改的数据表里面的数据，那么看到的还是事务提交之前的数据。
      except IntegrityError:
          handle_exception()
  
      add_children()
  ```

  - 这个例子中，即使generate_relationships()中的代码打破了数据完整性约束，你仍然可以在add_children()中执行数据库操作，并且create_parent()产生的更改也有效。需要注意的是，在调用handle_exception()之前，generate_relationships()中的修改就已经被安全的回滚了。因此，如果有需要，你照样可以在异常处理函数中操作数据库。

    ```
    尽量不要在atomic代码块中捕获异常
    
       因为当atomic块中的代码执行完的时候，Django会根据代码正常运行来执行相应的提交或者回滚操作。如果在atomic代码块里面捕捉并处理了异常，就有可能隐盖代码本身的错误，从而可能会有一些意料之外的不愉快事情发生。
    
       担心主要集中在DatabaseError和它的子类（如IntegrityError）。如果这种异常真的发生了，事务就会被破坏掉，而Django会在代码运行完后执行回滚操作。如果你试图在回滚前执行一些数据库操作，Django会抛出TransactionManagementError。通常你会在一个ORM相关的信号处理器抛出异常时遇到这个行为。
    
    	捕获异常的正确方式正如上面atomic代码块所示。如果有必要，添加额外的atomic代码块来做这件事情，也就是事务嵌套。这么做的好处是：当异常发生时，它能明确地告诉你那些操作需要回滚，而那些是不需要的。  
    ```

  - 为了保证原子性，atomic还禁止了一些API。像试图提交、回滚事务，以及改变数据库连接的自动提交状态这些操作，在atomic代码块中都是不予许的，否则就会抛出异常。

    ```
    下面是Django的事务管理代码：
       1、 进入最外层atomic代码块时开启一个事务；
       2、 进入内部atomic代码块时创建保存点；
       3、 退出内部atomic时释放或回滚事务；注意如果有嵌套，内层的事务也是不会提交的，可以释放（正常结束）或者回滚
        4、退出最外层atomic代码块时提交或者回滚事务；
    ```

  - 你可以将保存点参数设置成False来禁止内部代码块创建保存点。如果发生了异常，Django在退出第一个父块的时候执行回滚，如果存在保存点，将回滚到这个保存点的位置，否则就是回滚到最外层的代码块。外层事务仍然能够保证原子性。然而，这个选项应该仅仅用于保存点开销较大的时候。毕竟它有个缺点：会破坏上文描述的错误处理机制。

    ```python
    #注意：transaction只对数据库层的操作进行事务管理,不能理解为python操作的事务管理
    def example_view(request):
        tag = False
        with transaction.atomic():
            tag = True
            change_obj() # 修改对象变量
            obj.save()
            raise DataError
        print("tag = ",tag) #结果是True，也就是说在事务中的python变量赋值，即便是事务回滚了，这个赋值也是成功的
    ```

  - **还要注意**：如果你配置了全局的事务，它和局部事务可能会产生冲突，你可能会发现你局部的事务完成之后，如果你的函数里面其他的sql除了问题，也就是没在这个上下文管理器的局部事务包裹范围内的函数里面的其他的sql出现了问题，你的局部事务也是提交不上的，因为全局会回滚这个请求和响应所涉及到的所有的sql，所以还是建议以后的项目尽量不要配置全局的事务，通过局部事务来搞定，当然了，看你们的业务场景。

  - **transaction的其他方法**：

    ```python
    @transaction.atomic
    def viewfunc(request):
    
      a.save()
      # open transaction now contains a.save()
      sid = transaction.savepoint()  #创建保存点
    
      b.save()
      # open transaction now contains a.save() and b.save()
    
      if want_to_keep_b:
          transaction.savepoint_commit(sid) #提交保存点
          # open transaction still contains a.save() and b.save()
      else:
          transaction.savepoint_rollback(sid)  #回滚保存点
          # open transaction now contains only a.save()
    
      transaction.commit() #手动提交事务，默认是自动提交的，也就是说如果你没有设置取消自动提交，那么这句话不用写，如果你配置了那个AUTOCOMMIT=False，那么就需要自己手动进行提交。
    ```

  - 为保证事务的隔离性，我们还可以结合上面的锁来实现，也就是说在事务里面的查询语句，咱们使用select_for_update显示的加锁方式来保证隔离性，事务结束后才会释放这个锁，例如：（了解）

    ```python
    @transaction.atomic ## 轻松开启事务
    def handle(self):
        ## 测试是否存在此用户
        try:
            ## 锁定被查询行直到事务结束
            user = 
        User.objects.select_for_update().get(open_id=self.user.open_id)
            #other sql 语句
        except User.DoesNotExist:
            raise BaseError(-1, 'User does not exist.')
    ```

  - 通过Django外部的python脚本来测试一下事务：

    ```
    import os
    
    if __name__ == '__main__':
        os.environ.setdefault("DJANGO_SETTINGS_MODULE", "BMS.settings")
        import django
        django.setup()
    
        import datetime
        from app01 import models
    
        try:
            from django.db import transaction
            with transaction.atomic():
                new_publisher = models.Publisher.objects.create(name="火星出版社")
                models.Book.objects.create(title="橘子物语", publish_date=datetime.date.today(), publisher_id=10)  # 指定一个不存在的出版社id
        except Exception as e:
            print(str(e))
    ```

    

- 下面再说一些设置事务的小原则：

  ```
  	  1.保持事务短小 
  　　　　2.尽量避免事务中rollback 
  　　　　3.尽量避免savepoint 
  　　　　4.默认情况下，依赖于悲观锁 
  　　　　5.为吞吐量要求苛刻的事务考虑乐观锁 
  　　　　6.显示声明打开事务 
  　　　　7.锁的行越少越好，锁的时间越短越好
  ```

  