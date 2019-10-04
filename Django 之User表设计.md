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

  