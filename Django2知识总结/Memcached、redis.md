## Memcached



#### 什么是memcached：

Memcached之前是danga的一个项目，最早是为LiveJournal服务的，Memcached是一个高性能的分布式内存对象缓存系统，全世界不少公司采用这个缓存构建大负载的网站，来分担数据库压力，Memcached是通过在内存里维护一个统一的巨大的hash表，其能存储各种数据：图像、视频、文件、数据库索引等

- 简单的说就是将数据调用到内存中，然后从内存中读取，从而大大提高读取速度
- Memcached实用场景：存储验证码（图像、短信验证码），登陆session等所有重要的数据



#### 安装和启动Memcached

- windows：https://www.runoob.com/memcached/window-install-memcached.html

  ```shell
  安装：memcached.exe -d install  # 要以管理员身份启动
  启动：memcached.exe -d start
  
  PS D:\memcache\memcached> .\memcached.exe -d install
  PS D:\memcache\memcached> .\memcached.exe -d start
  
  # 如果启动失败，将pthreadGC2.dll复制到C盘windows下的window32目录下，再试一次；再查看任务管理器的-> 服务 -> 看看启动没
  
  常用命令：
  c:\memcached\memcached.exe -d start
  c:\memcached\memcached.exe -d stop
  c:\memcached\memcached.exe -d uninstall
  ```

- linux:

  ```
  yum install memcachaed
  systemctl start memcached
  
  [root@MyHost ~]# ps aux |grep memcached
  memcach+  8766  0.0  0.0 325552  1188 ? Ssl  20:40   0:00 /usr/bin/memcached -u memcached -p 11211 -m 64 -c 1024
  ```

- 常见参数: 因为systemctl 命令是按默认方式启动，所以我们可以通过下面方式自定制启动

  ```shell
  -d: 让memcached再后台运行
  -m: 指定占用neicun，以m为单位，默认64m
  -p: 指定端口，默认11211
  -l：那些ip可以连接
  -u: 指定用户启动
  
  [root@MyHost ~]# /usr/bin/memcached -u root -d start
  [root@MyHost ~]# ps aux |grep memcached
  root      8819  0.0  0.0 325552   912 ?        Ssl  20:48   0:00 /usr/bin/memcached -u root -d start
  
  [root@MyHost ~]# /usr/bin/memcached -m 1024 -u root -d start
  
  [root@MyHost ~]# /usr/bin/memcached -m 1024 -u root -p 12222 -d start
  
  [root@MyHost ~]# /usr/bin/memcached -m 1024 -u root -d -l 0.0.0.0 start  # 如果想要别的机器能连接就必须配置-l参数
  ```

- 使用telnet操作memcached

  ```shell
  [root@MyHost ~]# systemctl start memcached
  [root@MyHost ~]# yum install telnet
  
  [root@MyHost ~]# telnet 127.0.0.1 11211
  Trying 127.0.0.1...
  Connected to 127.0.0.1.
  Escape character is '^]'.
  set username 0 60 3   # 设置
  abc
  STORED
  get username   # 获取
  VALUE username 0 3
  abc
  END
  
  ```

- 总结操作

  ```
  4.Memcached set命令
      Memcached set 命令用于将 value(数据值) 存储在指定的 key(键) 中。
      如果set的key已经存在，该命令可以更新该key所对应的原来的数据，也就是实现更新的作用。
      语法：
    	set key flags exptime bytes [noreply]
      value
  
      参数说明如下：
      key：键值 key-value 结构中的 key，用于查找缓存值。
      flags：可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息 （压缩是否）。
      exptime：在缓存中保存键值对的时间长度（以秒为单位，0 表示永远）
      bytes：在缓存中存储的字节数
      noreply（可选）： 该参数告知服务器不需要返回数据
      value：存储的值（始终位于第二行）（可直接理解为key-value结构中的value）
  
      例如：
      set foo 0 900 9
      memcached
      get foo
  
  5.Memcached add 命令
      Memcached add 命令用于将 value(数据值) 存储在指定的 key(键) 中。
      如果 add 的 key 已经存在，则不会更新数据(过期的 key 会更新)，之前的值将仍然保持相同，并且您将获得响应 NOT_STORED。
  
      语法：
      add key flags exptime bytes [noreply]
      value
  
      例如：
      add new_key 0 900 10
      data_value
  
  6.Memcached replace 命令
      Memcached replace 命令用于替换已存在的 key(键) 的 value(数据值)。
      如果 key 不存在，则替换失败，并且您将获得响应 NOT_STORED。
      语法：
      replace key flags exptime bytes [noreply]
      value
  
      例如：
      add mykey 0 900 10
      data_value
      get mykey
  	replace mykey 0 900 16
  	some_other_value
  
  7.Memcached append 命令
      Memcached append 命令用于向已存在 key(键) 的 value(数据值) 后面追加数据 。
      语法：
  
      append key flags exptime bytes [noreply]
      value
  
  8.Memcached prepend 命令
      Memcached prepend 命令用于向已存在 key(键) 的 value(数据值) 前面追加数据 。
      用法：
  
      prepend key flags exptime bytes [noreply]
      value
  
      例如：
      set foo 0 900 9
      memcached
      STORED
      get foo
      VALUE foo 0 9
      memcached
      END
      
      prepend foo 0 900 5
      redis
      STORED
      get foo
      VALUE foo 0 14
      redismemcached
      END
  
  9.Memcached get 命令
      Memcached get 命令获取存储在 key(键) 中的 value(数据值) ，如果 key 不存在，则返回空。
      语法：
  
      get key
      多个 key 使用空格隔开，如下:
      get key1 key2 key3
  
  10.Memcached delete 命令
      Memcached delete 命令用于删除已存在的 key(键)。
      语法：
  
      delete key [noreply]
  
  11.Memcached incr 与 decr 命令
      Memcached incr 与 decr 命令用于对已存在的 key(键) 的数字值进行自增或自减操作。
      incr 与 decr 命令操作的数据必须是十进制的32位无符号整数。
      如果 key 不存在返回 NOT_FOUND，如果键的值不为数字，则返回 CLIENT_ERROR，其他错误返回 ERROR。
      incr 命令语法：
      incr key increment_value
  
      例如：
      set no 0 900 2
      10
      incr no 5
      decr 命令语法：
      decr key decrement_value
      例如：
      decr no 5
      5
  ```

  

#### telnet 命令

- windows系统默认有些不开启telnet功能，我们可以通过再控制面板中打开 "关闭或开启windows功能"中勾选telnetclient，重启即可

  ```shell
  >telent ip  端口      # 启动
  >quit              #退出
   
  [root@MyHost ~]# /usr/bin/memcached -l 0.0.0.0 -u root -d start   # linux启动，windows访问，阿里云添加端口
  ```

  windows下连接

  ```
  > telnet 47.95.217.144 11211  # 连接后无反应，需要回车以下
  
  set语法：key->value  如果key存在就覆盖，没有就添加
  	set user[key] 0[是否压缩] 60[过期时间] 7[字符长度]
  	
  get语法：从memcached中获取数据
  	get user[key名称]
  	
  add语法: 添加键值，已经有的会添加失败
  	add usrname[key] 0[是否压缩] 120[过期时间] 7[长度]
  	
  delete语法： 删除键值
  	delete user
  	
  flush_all ：删除所有
  
  stats:查看memcached状态
  ```

  

#### Python操作Memcached

安装 python-memcached

```
pip3 install python-memcached
```

- 建立连接

  ```python
  import memcache
  mc = memecache.Client(['127.0.0.1:11211'],debug=True)
  
  #设置数据
  mc.set('user','hello',time=60*2)
  mc.set_multi({'email':'xxx@163.com','telephone':'12343421'},time=60)
  
  # 获取值
  mc.get('telephone')
  
  # 删除数据
  mc.delete('email')
  ```

  

#### 在Django中配置Memecached

和配置Mysql数据库一样，不过可以设置分布式：添加CACHES

```python
CACHES = {
    'default': {
        'BACKEND':'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION':[
        	'127.0.0.1:11211',   # 分布式存储
        	'127.0.0.3:11211',
        ]
    }
}
```

配置好memcached后，之后代码中可以如下操作

```python
from django.core.cache import cache

def index(reqeust):
	cache.set('key','xxx',69)
	print(cache.get('key'))
	return HttpResponse('success')
```

需要注意的是，django在存储数据到memcached中的时候，不会讲指定的key存储进去，而是对key进行一些处理，比如一个前缀，会加上版本号，如果想要自己加上前缀，那可以settings.CACHES中添加KEY_FUNCTION参数：

```
CACHES = {
    'default': {
        'BACKEND':'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION':[
        	'127.0.0.1:11211',   # 分布式存储
        	'127.0.0.3:11211',
        ],
        'KEY_FUNCTION':lambda key,prefix_key,version:"django:%s"%key
    }
}
```

