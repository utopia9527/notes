# 一、WSGI协议概念

## 1.1 WSGI是什么

- Django 是 Python 语言编写的 Web 框架，而我们使用 Django 编写的项目称为 Web 应用，而一个 Web 项目的运行离不开 Web 服务器，所以就需要一种规范或者协议来定义 Web 应用如何与 Web 服务器之间实现交互以及请求的接受与响应的返回，这就引出了 WSGI 协议

- WSGI（Web Server Gateway Interface）即 Web 服务器网关接口，它是属于一种规范协议，它定义了 Python Web 应用程序与 Web 服务器通信的接口

- 简而言之，**WSGI是一种协议，是一种web应用和web服务器之间实现交互的协议**

## 1.2 Django 内置wsgi服务器

- 在 WSGI 协议中定义了两个角色：一个是 Web 服务器即 server，另一个是应用程序即 application

- server 需要接受来自客户端的请求，然后根据wsgi协议定义调用应用程序（application），应用程序处理请求并返回结果给 server，最终响应给客户端

- Django 框架同时实现了 WSGI 的 server 和 application。其中内置的 WSGI 服务器是**基于 Python 的内置模块 wsgiref** 实现的，主要是添加了一些异常处理和错误记录，**但是没有考虑到运行效率，故不适合在生产环境中使用**

- 实例演示

  - 在manage.py 同级目录之下新建一个web.py文件，编写如下代码

  ```python
  from wsgiref.simple_server import make_server # 使用的是Django内部实现的WSGI服务器
  
  # 定义服务器调用对象application
  # application必须使用start_response(status，headers)，并且返回值是一个可迭的代序列，序列中的每个对象将标准输出
  # WSGI environ是一些键值对，要么是提供给server，要么提供给middleware
  
  # 方法 application由 web服务器调用，参数env，start_response 由 web服务器实现并传入
  # 其中，env是一个字典，包含了类似 HTTP_HOST，HOST_USER_AGENT，SERVER_PROTOCO 等环境变量。
  # start_response则是一个方法，该方法接受两个参数，分别是status，response_headers。
  # application方法的主要作用是，设置 http 响应的状态码和 Content-Type 等头部信息，并返回响应的具体结果。
  
  def application(environ,start_response): 
      """
      :param environ:  #包含所有客户端的请求信息即上下文请求，application从这个参数中获取客户端请求意图
      :param start_response: 一个可调用对象，用于发送http请求状态
      :return: [b'Hello World!\n'] #返回可迭代对象 且必须是字节流，Http是面向字节流协议
       """
      print('environ:',environ)
  
      #  构建http响应
      status='200 OK'
      response_headers=[('Conteny-type','text/plain')] #响应头是一个列表
      start_response(status,response_headers) #返回给server之前调用 start_response
  
      return [b"Hello World!\n"]
  
  #创建WSGI服务器，指定调用application,这里的调用对象也可以是一个类或者实例
  httpeserver=make_server('127.0.0.1',8000,application) 
  #处理请求后退出
  httpeserver.handle_request()
  ```

  - 启动项目后，在命令行输入`curl -i 127.0.0.1:8000`即可得到如下响应

    ```python
    HTTP/1.1 404 Not Found
    Date: Wed, 25 May 2022 06:28:52 GMT
    Server: WSGIServer/0.2 CPython/3.6.4
    Content-Type: text/html
    X-Frame-Options: SAMEORIGIN
    Content-Length: 2025
    ```

- 总结

  - 由上面测试代码可知，wsgi就是一个简单的测试当前Django项目接口的web协议
  - 正常的流程和wsgi流程
    - 正常流程：启动项目，写好url，视图函数，然后在网页之中进行接口的调用
    - wsgi流程：启动项目，写好wsgi文件，然后**再命令行执行对应命令即可获取对应接口的反馈**

# 二、uWSGI生产环境的搭建与配置

## 2.1 uWSGI基本概念

- 上面介绍了 Django 的内置 WSGI 服务器的实现，但是由于 Django 并不专注于实现服务器，所以内置的 WSGI 服务器无法满足生产环境的需求，所以我们在这里要介绍另外一种 WSGI 服务器，即 uWSG
- **uWSGI 是当下最流行的一种 WSGI 服务器，同样遵守 WSGI 协议**
- 它可以与各种 Python Web 框架实现兼容，而且配置过程与使用方式都非常简单
- 我们之前使用 **runserver 命令启动项目，通常只是在开发和测试环境中使用**
- 当开发结束后，**完善的项目代码需要在一个高效稳定的环境中运行，这时候可以使用 uWSGI**，它是 WSGI 服务器的一种，它可以让 Django、Flask 等开发的 Web 应用站点运行其中

## 2.2 安装配置uWSGI服务器

- `sudo pip3 install uwsgi`

- 在项目根目录下(和manage.py同级)，新建uwsgi.ini 配置文件，内容如下

  ```python
  [uwsgi]
  # 套接字方式的 IP地址:端口号
  # socket=127.0.0.1:8000
  # Http通信方式的 IP地址:端口号
  http=127.0.0.1:8000
  #上述两种方式选择其一，在使用Nginx需要使用socket
  
  # 项目当前工作目录,这里需要换为项目文件夹的绝对路径
  chdir=/Users/didi/django/django_demp
  
  # 项目中wsgi.py文件的目录，相对于当前工作目录
  wsgi-file=/Users/didi/django/django_demp/trail
  
  #是否启动主进程来管理其他进程
  master=true
  
  # 进程个数，根据电脑配置设置
  process=4
  
  # 每个进程的线程个数
  threads=2
  
  # 服务的pid记录文件
  pidfile=/Users/didi/django/django_demp/uwsgi.pid
  
  # 服务的日志文件位置
  daemonize=/Users/didi/django/django_demp/uwsgi.log
  ```

  - 然后修改 settings.py 文件将其设置为适合线上生产环境使用

    ```python
    DEBUG=False          #关闭调试模式
    ALLOWED_HOSTS = ['*']  #任何ip都可以访问
    ```

    

- 启动与测试

  ```bash
  启动 uwsgi 命令
  $ cd 项目文件夹
  $ sudo uwsgi --ini uwsgi.ini
  
  停止 uwsgi 命令
  $ cd 项目文件夹
  $ sudo uwsgi --stop uwsgi.pid
  ```

- 说明

  - 当 uWSGI 启动后，当前 Django 项目的程序已变成后台守护进程，在关闭当前终端时此进程也不会停止。执行完我启动命令后，可以看到在项目的主目录下又生成了两个文件分别是 uwsgi.pid 与 uwsgi.log，它们分别用来记录，uWsgi 的进程号码 pid 与 uWSGI 的相关运行日志。启动成功的标志如下所示

    ```bash
    [uWSGI] getting INI configuration from uwsgi.ini
    ```

  - ​	启动之后，就可在游览器通过url可以正常进行访问了

## 2.3 uwsgi

- uWSGI为web服务器，是web server 和 web application 沟通的桥梁
- 通信则需要标准，这个标准就是协议，uWSGI  服务器有很多种协议，而常用的协议就是uwsgi

# 三、uwsgi 拒绝服务

- [uwsgi拒绝服务](https://www.dytttf.com/2021/11/22/%E4%B8%80%E6%AC%A1uWSGI%E6%9C%8D%E5%8A%A1%E5%93%8D%E5%BA%94500%E6%8E%92%E6%9F%A5-worker%E5%8A%A0%E8%BD%BD%E9%9A%8F%E6%9C%BA%E5%A4%B1%E8%B4%A5/)

