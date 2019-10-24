## DRF + Vue配置

```
python3    ：Python代码解释器

uwsgi wsgi(web服务网关接口，就是一个实现了python web应用的协议)：作用就是启动Django项目

virtualenvwrapper  ：虚拟环境隔离

Django路飞的代码

vue的代码

nginx (一个是nginx对静态文件处理的优秀性能，一个是nginx的反向代理功能，以及nginx的默认80端口，访问nginx的80端口，就能反向代理到应用的8000端口)

mysql

redis ：缓存，购物车订单信息

supervisor 进程管理工具
```



#### 项目逻辑架构的介绍

![img](https://img2018.cnblogs.com/blog/1269599/201901/1269599-20190105221923146-160865937.png)

web服务器

```
传统的c/s架构，请求的过程是
客户端 > 服务器 
服务器 > 客户端
服务器就是：1.接收请求 2.处理请求 3.返回响应
```

web框架层

```
HTTP的动态数据交给web框架，例如django遵循MTV模式处理请求。
HTTp协议使用url定位资源，urls.py将路由请求交给views视图处理，然后返回一个结果，完成一次请求。
web框架使用者只需要处理业务的逻辑即可。
```



### 项目部署环境步骤 



### 一、环境准备

　　确保准备好python3的环境，和虚拟解释器vitrualenvwrapper

　　然后新建一个虚拟环境 ***mkvirtualenv s15vuedrf*** (其中，虚拟环境的包是创建在 ***/root/Envs/s15vuedrf*** 中)

　　将项目部署在该虚拟环境中

　　前面的博文中已做了详细的解释：

　　　　[python3的安装](https://www.cnblogs.com/cyycyhcbw/articles/10193946.html)

　　　　[virtualenvwrapper的安装](https://www.cnblogs.com/cyycyhcbw/articles/10193946.html#_label1_1)



#### 二、准备前后端的代码

```
1.准备前后端代码

    下载前后端的代码，这里将项目下载到 /opt/s15vuedrf中   
    mkdir /opt/s15vuedrf
    cd /opt/s15vuedrf
    wget https://files.cnblogs.com/files/pyyu/luffy_boy.zip 
    wget https://files.cnblogs.com/files/pyyu/07-luffy_project_01.zip

    如果代码在本地，传到服务器 使用 lrzsz 或者 xftp工具，进行传输


2.解压缩代码
    由于是.zip压缩格式，所以用unzip解压即可，解压到当前文件夹
    unzip luffy_boy.zip 
    unzip 07-luffy_project_01.zip
```



#### 三、从vue前端代码搞起

```
(1) 准备node打包环境，下载到 /opt中

    cd /opt
    wget https://nodejs.org/download/release/v8.6.0/node-v8.6.0-linux-x64.tar.gz
    
(2) 解压缩node包，配置环境变量，使用npm和node命令
    系统环境变量配置好之后，就可以直接使用
    
    tar -zxvf node-v8.6.0-linux-x64.tar.gz
    vim /etc/profile        
    PATH = "xxxxx:/opt/node-v8.6.0-linux-x64/bin"   #进入源码中的bin,查看命令
    source /etc/profile
    
(3) 检测node和npm命令是否正常,查看版本号

    node -v    # v8.6.0
    npm  -v    # 5.3.0
    
(4) 安装vue项目所需的包
    cd /root/Envs/s15vuedrf/07-luffy_project_01   # 这里虚拟包位置根据自己创建位置确定
    输入
        npm install      # 下载依赖包，package.json中的依赖包
        npm run build    # 打包编译前端代码到 dist 文件夹
    这两条都正确配置之后，就会生成一个 dist 静态文件目录，整个项目的前端内容和index.html都在这里了
    如果打包编译不成功的话，可以使用pycharm打包之后，将dist文件直接传输到vue项目文件中即可。
    
(5) 之后等待nginx加载这个 dist文件夹即可。
```



### 四、部署后端代码所需的环境

#### 1、部署 uwsgi 启动后端项目

```
(1) 激活虚拟环境并进入

    workon s15vuedrf 
    
(2) 通过一条命令，导出本地的所有软件包依赖 
    在windows终端cmd中导出：
    
    pip3 freeze >  requirements.txt 
    
(3) 下载django所依赖的模块
    将这个requirements.txt 传至到服务器，在服务器的新虚拟环境中，安装这个文件，就能安装所有的软件包了
    
    pip3 install -r  requirements.txt    # 下载文件中后端的依赖包，
    
    注意：可能安装不完全，然后手动启动项目，查看缺少什么依赖包，然后单独再进行安装
    python3 manage.py runserver
    
    这个文件内容如下：项目所需的软件包都在这里了
        [root@web02 opt]# cat requirements.txt 这个是老师电脑上的
        certifi==2018.11.29
        chardet==3.0.4
        crypto==1.4.1       # pycrypto
        Django==2.1.4
        django-redis==4.10.0
        django-rest-framework==0.1.0
        djangorestframework==3.9.0
        idna==2.8
        Naked==0.1.31
        pycrypto==2.6.1
        pytz==2018.7
        PyYAML==3.13
        redis==3.0.1
        requests==2.21.0
        shellescape==3.4.1
        urllib3==1.24.1
        uWSGI==2.0.17.1
        
        # 这个是自己电脑上的
        (s15vuedrf) [root@localhost s15vuedrf]# cat requirements.txt 
        asn1crypto==0.24.0     # pycrypto
        beautifulsoup4==4.6.3
        certifi==2018.8.24
        cffi==1.11.5
        chardet==3.0.4
        Click==7.0
        cryptography==2.3.1
        Django==2.1.2
        django-multiselectfield==0.1.8
        django-redis==4.10.0
        djangorestframework==3.9.0
        Flask==1.0.2
        gevent==1.3.6
        greenlet==0.4.15
        idna==2.7
        itsdangerous==0.24
        Jinja2==2.10
        MarkupSafe==1.0
        Naked==0.1.31
        Pillow==5.3.0
        pycparser==2.18
        pycryptodome==3.7.2
        PyMySQL==0.9.2
        pytz==2018.5
        PyYAML==3.13
        redis==3.0.1
        requests==2.19.1
        shellescape==3.4.1
        six==1.11.0
        urllib3==1.23
        Werkzeug==0.14.1

    
(4) 准备uwsgi 支持高并发的启动python项目(注意uwsgi不支持静态文件的解析，必须用nginx去处理静态文件)
    因此这里用 uwsgi 代替项目中的 wsgi 来启动项目，因为要在虚拟环境中启动项目,需要将uwsgi安装在虚拟环境里。
    1. 安装uwsgi，要确保进入到虚拟环境中，再进行安装
    
        workon s15vuedrf
        pip3 install -i https://pypi.douban.com/simple uwsgi
        
        which uwsgi   # 查看是否来自虚拟环境,才能启动在虚拟环境中启动项目

    2. 学习uwsgi的使用方法
        通过uwsgi启动单个的python web文件
        
         uwsgi --http :8000 --wsgi-file   s15testuwsgi.py    # uwsgi启动单个py文件的命令
        
                --http 指定http协议 
                --wsgi-file  指定一个python文件
        
        通过uwsgi启动django项目，并且支持热加载项目，不重启项目，自动生效 新的 后端代码
        django 项目名为s15drf,需要进入项目中：
        
            cd s15drf            
            uwsgi --http  :8000 --module s15drf.wsgi   --py-autoreload=1   # uwsgi启动django项目命令
                
                --s15drf.wsgi，      #由于wsgi文件在内层的s15drf中            
                --module             #指定找到django项目的wsgi.py文件
                --py-autoreload=1    #热加载,表示
        
        
        这种启动方式比较麻烦，因为可以写在配置文件 uwsgi.ini 中，通过配置文件启动项目
                
                
(5) 使用uwsgi的配置文件，启动项目
        1) 创建一个uwsgi.ini配置文件，写入参数信息
        
            在项目的外层目录中创建配置文件，与manage.py在同一目录即可，
            
            cd /opt/s15vuedrf/luffy_boy
            touch uwsgi.ini 
            
                [uwsgi]                    
                chdir           = /opt/s15vuedrf/luffy_boy/                    
                module          = luffy_boy.wsgi                    
                home            = /root/Envs/s15vuedrf                    
                master          = true                                    
                processes       = 1                    
                socket          = 0.0.0.0:9000     # 与nginx连用，启动项目时使用 socket
                # http            = 0.0.0.0:9000   # 只有uwsgi启动的时候使用 http
                vacuum          = true
                
                
                ++++++++++++++  详解见以下文件  ++++++++++++++++++++++
```

uwsgi.ini

```
[uwsgi]
# Django-related settings
# the base directory (full path)
#指定项目的绝对路径的第一层路径！！！！！！！！！！！！！！！！！！！！！！！！
chdir           = /opt/s15vuedrf/luffy_boy/

# Django's wsgi file
#  指定项目的 wsgi.py文件！！！！！！！！！！！！
#  写入相对路径即可，这个参数是以  chdir参数为相对路径
module          = luffy_boy.wsgi

# the virtualenv (full path)
# 写入虚拟环境解释器的 绝对路径！！！！！！
home            = /root/Envs/s15vuedrf
# process-related settings
# master
master          = true
# maximum number of worker processes
#指定uwsgi启动的进程个数                
processes       = 1

#这个参数及其重要！！！！！！
#这个参数及其重要！！！！！！
#这个参数及其重要！！！！！！
#这个参数及其重要！！！！！！
# the socket (use the full path to be safe
#socket指的是，uwsgi启动一个socket连接，当你使用nginx+uwsgi的时候，使用socket参数
socket          = 0.0.0.0:8000

#这个参数是uwsgi启动一个http连接，当你不用nginx只用uwsgi的时候，使用这个参数
#这个参数是uwsgi启动一个http连接，当你不用nginx只用uwsgi的时候，使用这个参数
#这个参数是uwsgi启动一个http连接，当你不用nginx只用uwsgi的时候，使用这个参数
#这个参数是uwsgi启动一个http连接，当你不用nginx只用uwsgi的时候，使用这个参数
#这个参数是uwsgi启动一个http连接，当你不用nginx只用uwsgi的时候，使用这个参数

#http  =  0.0.0.0:8000   #。。。。。。。

# ... with appropriate permissions - may be needed
# chmod-socket    = 664
# clear environment on exit
vacuum          = true

```



```
(6)使用uwsgi配置文件启动项目

    cd /opt/s15vuedrf/luffy_boy/    
    
    uwsgi --ini  uwsgi.ini       # 确保在项目中，执行以下命令，启动项目    
```



#### 2、supervisor进程管理工具，启动项目

```
supervisor进程管理工具

1.将linux进程运行在后台的方法有哪些

    1)命令后面加上 &  符号
        python  manage.py  runserver &
    2)使用nohup命令
    3)使用进程管理工具supervisor
    
    这里采用第三种方式


2.安装supervisor，使用python2的包管理工具 easy_install ，注意，此时要退出虚拟环境！！！！
    注意，此时要退出虚拟环境！！！！
    注意，此时要退出虚拟环境！！！！
    注意，此时要退出虚拟环境！！！！
    
    # 如果没有命令，使用以下命令，安装：
    
    yum install python-setuptools
    easy_install supervisor
    
    # 然后,输入 echo tab键查看是否有echo_supervisord_conf命令


3.通过命令，生成一个配置文件，这个文件就是写入你要管理的进程任务

    echo_supervisord_conf > /etc/supervisor.conf

    
4.编辑这个配置文件，写入操作  django项目的 命令 

    vim /etc/supervisor.conf
    
    # 直接到最底行，写入以下配置  uwsgi启动项目的命令
    
    [program:s15luffy]
    command=/root/Envs/s15vuedrf/bin/uwsgi  --ini  /opt/s15vuedrf/luffy_boy/uwsgi.ini   
    
    #command= uwsgi路径 --ini 项目中uwsgi.ini配置文件路径


5.启动supervisord服务端，指定配置文件启动
    启动之前要确保，没有杀死 supervisor进程和 uwsgi 进程

    supervisord -c  /etc/supervisor.conf

    
6.通过supervisorctl管理任务

    supervisorctl -c /etc/supervisor.conf 

    
7.supervisor管理django进程的命令如下
    supervisorctl直接输入命令会进入交互式的操作界面
    >  stop s15luffy 
    >  start s15luffy 
    >  status s15luffy 


8.完成了启动luffy的后端代码
```



#### 3、配置 nginx 步骤

```
配置nginx步骤如下
1.确保安装了nginx
    前面的博文中有安装步骤，查看即可
    nginx的安装

2.配置文件/opt/nginx112/conf/nginx.conf如下：

    #第一个server虚拟主机是为了找到vue的dist文件， 找到路飞的index.html
    server {
            listen       80;
            server_name  192.168.13.210;
            
            #当请求来自于 192.168.13.210/的时候默认80端口，直接进入以下location，然后找到vue的dist/index.html 
            location / {
                root   /opt/s15vuedrf/07-luffy_project_01/dist;
                index  index.html;
            }
        }
        
    #由于vue发送的接口数据地址是 192.168.13.210:8000  我们还得再准备一个入口server如下
    server {
        listen 8000;
        server_name  192.168.13.210;
        
        #当接收到接口数据时，请求url是 192.168.13.210:8000 就进入如下location
        location /  {
            #这里是nginx将请求转发给  uwsgi启动的 9000端口，uwsgi.ini文件中端口9000，的项目将被启动
            uwsgi_pass  192.168.13.210:9000;
            # include  就是一个“引入的作用”，就是将外部一个文件的参数，导入到当前的nginx.conf中生效
            include /opt/nginx112/conf/uwsgi_params;
        }
    }


3.启动nginx 

    nginx    # 直接启动服务，如果没有添加系统环境变量就用绝对路径启动，/opt/nginx112/sbin/nginx
    
    此时可以访问 192.168.13.210  查看页面结果即可，显示正常则完成项目的部署


    
启动路飞项目，这个项目用的是sqllite，因此安装mysql自行选择

redis必须安装好，存放购物车的数据
```



### 五、vue+nginx页面刷新404错误解决方案

```
vue+nginx页面刷新404错误解决方案

参考地址:https://router.vuejs.org/zh/guide/essentials/history-mode.html

1.修改nginx.conf,给前端server添加一行try_files $uri $uri/ /index.html;
# 这里的try_files相当于在服务端增加一个覆盖所有情况的候选资源：如果 URL 匹配不到任何静态资源，则应该返回同一个 index.html 页面
   
  server{
       listen 80;
       server_name 192.168.119.128;
       location / {
            root /opt/vuedrf/07-luffy_project_01/dist;
            index index.html ;
            try_files $uri $uri/ /index.html;      
        }
  }

2.此时，你的服务器就不再返回 404 错误页面，因为对于所有路径都会返回 index.html 文件;如果路由不存在则会返回空页面,不会报错.
  为了避免这种情况，你应该在 Vue-router里面覆盖所有的路由情况，然后再给出一个NotFound页面

    (1)vue components里创建一个NotFound页面:

          <template>
              <h1>页面不存在</h1>
          </template>

          <script>
              export default {
                  name: "NotFound"
              }
          </script>
          <style scoped>
          </style>

    (2)vue-router里:
    
          # 引入NotFound页面
          import NotFound from '@/components/NotFound/NotFound'
          # 在routes列表最后添加NotFound捕获所有未定义页面
          {
            path: "*",
            name:'NotFound',
            component:NotFound,
          }
      
3.重新生成dist文件,启动nginx
```

