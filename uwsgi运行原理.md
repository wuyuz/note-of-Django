 最近在看uWSGI配置启动服务，有几个关于uWSGI的配置的疑问，想请教一下大家。当使用uWSGI启动服务的时候可以配置启动多个进程，线程。但是我们使用的python web应用服务框架例如django flask不都是单进程运行的吗？uWSGI是怎么将这样的一个服务通过多进程和多线程的方式启动提供服务的呢？uWSGI多进程，多线程情况下，python web框架中的缓存又是怎么同步的？百度了一下，关于为什么使用uWSGI，uWSGI做了什么，都说的很笼统。关于弄明白uWSGI是什么，为什么，做了什么的文章或者博客，大家有好的推荐没？ 

 https://uwsgi-docs-zh.readthedocs.io/zh_CN/latest/ThingsToKnow.html 