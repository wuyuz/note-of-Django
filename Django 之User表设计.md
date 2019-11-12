## Django 之User表设计



#### AbstractUser

- 我们想使用Django自带的User表结构就需要继承AbstractUser类，点击查看源码流程

  ```python
  1、AbstractUser 继承了AbstractBaseUser以及PermissionMixin，总的来说就是为我们创建了很多字段
  2、在AbstractBaseUser中默认给我们创建好了password、last_login、is_active三个字段，并使用了 
  	class Meta:
          abstract = True  # 表示强制实现了这三个字段，我们就不用写了
          
  3、注意点，在AbstractUser中的object不再是一般表中继承的Manager()管理方式了，而是使用的自定义的UserManger()
  	objects = UserManager()  # 点击查看
      ...
      EMAIL_FIELD = 'email'  
      USERNAME_FIELD = 'username'  #以那个字段作为登陆名
      REQUIRED_FIELDS = ['email'] # 必须的字段
  	
  4、UserManger() 为我们提供了三个方法：
  	class UserManager(BaseUserManager):
          use_in_migrations = True
  
          def _create_user(self, username, email, password, **extra_fields):pass
              """
              Creates and saves a User with the given username, email and password.
              """
          def create_user(self, username, email=None, password=None, **extra_fields):pass
  
          def create_superuser(self, username, email, password, **extra_fields):pass
  
  
  ```

- 创建用户表的注意事项

  ```python
  from django.db import models
  from django.contrib.auth.models import AbstractUser
  class UserProfile(AbstractUser):
      mobile = models.CharField(max_length=11,verbose_name='手机号码')
      icon = models.ImageField(upload_to='uploads/%Y/%m/%d/')
      # 按照年月日再media文件中生成文件夹，再static中创建media，
      # 同时配置setting中设置MEDIA_URL='/static/media/'和MEDIA_ROOT=os.path.join(BASE_DIR,'static')
      # 如果想再模板中使用上传的图片，还需要再TEMPLATES下的OPTION中添加'django.template.context_processors.media'
      # 迁移时需要添加setting中：AUTH_USER_MODEL = 'user.UserProfile' # 应用.表名
      
      class Meta:
          db_table = 'userprofile'
          verbose_name='用户表'
          verbose_name_plural = verbose_name
  ```

  

#### 确定 User Model

 我们推荐一下方式来确定某一django项目使用的user model: 

```python
    # 使用默认User model时
    >>> from django.contrib.auth import get_user_model
    >>> get_user_model()
    <class 'django.contrib.auth.models.User'>

    # 使用自定义User model时
    >>> from django.contrib.auth import get_user_model
    >>> get_user_model()
    <class 'xxx.models.UserProfile'>
```

#### 方法一：使用settings.AUTH_USER_MODEL 

自从django 1.5之后, 用户可以自定义User model了, 如果需要外键使用user model, 官方推荐的方法如下，在settings中设置AUTH_USER_MODEL:

```python
# settings.py
# 格式为 "<django_app名>.<model名>"
AUTH_USER_MODEL = "myapp.NewUser"
```

在models.py中使用

```python
# models.py
from django.conf import settings
from django.db import models

class Article(models.Model):
    author = models.ForeignKey(settings.AUTH_USER_MODEL)
    title = models.CharField(max_length=255)
```

不要忘了在settings.py中设置:

```
 AUTH_USER_MODEL = "myapp.NewUser"
```

#### 方法2: 扩展 AbstractBaseUser类

AbstractBaseUser中只含有3个field: password, last_login和is_active. 如果你对django user model默认的first_name, last_name不满意, 或者只想保留默认的密码储存方式, 则可以选择这一方式.

#### 方法3: 使用OneToOneField

如果你想建立一个第三方模块发布在PyPi上, 这一模块需要根据用户储存每个用户的额外信息. 或者我们的django项目中希望不同的用户拥有不同的field, 有些用户则需要不同field的组合, 且我们使用了方法1或方法2:

更多知识： https://www.jianshu.com/p/b993f4feff83 

