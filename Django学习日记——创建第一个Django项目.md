## 使用Docker安装Django
经过之前对Docker的运用，目前我们对创建容器已经有了一定了解，搭配Dockerfile以及docker-compose.yml我们可以很方便的生成一个我们需要的容器环境。 
接下来我们的目的是在创建好的容器上进行Django框架的学习。
先给出Dockerfile以及docker-compose.yml的配置： 
使用python环境来做基础镜像生成容器： 
```
FROM python:3.6
WORKDIR /mnt/data/mysite
RUN sed -i s@/archive.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list
RUN sed -i s@/deb.debian.org/@/mirrors.aliyun.com/@g /etc/apt/sources.list
RUN apt-get clean && apt-get update && apt-get install -y \
	vim \
	openssh-server \
	net-tools \
	inetutils-ping
RUN pip install \
	django \
	uwsgi
```
我们已经使用环境在对应目录下生成过django的项目，因此这里我们可以直接使用docker-compose.yml生成并启动容器环境： 
```
version: '3'

services:
 mydjango:
    image: mydjango:v1
    build: .
    restart: always
    command:
      - /bin/bash 
      - -c 
      - |
        cd /mnt/data/mysite 
        python manage.py runserver 0.0.0.0:1234 
    ports:
    - "1234:1234"
    volumes:
    - ./data:/mnt/data
    container_name: mydjango
```
完成后，我们就可以进行Django的学习了。
## 创建Django项目
这里我们参照的是[Django 文档](https://docs.djangoproject.com/zh-hans/3.2/ "Django 文档")。
如果这是你第一次使用 Django 的话，你需要一些初始化设置。也就是说，你需要用一些自动生成的代码配置一个 Django project —— 即一个 Django 项目实例需要的设置项集合，包括数据库配置、Django 配置和应用程序配置。
打开命令行，cd 到一个你想放置你代码的目录，然后运行以下命令：
```
$ django-admin startproject mysite
```
让我们看看 startproject 创建了些什么:
```
mysite/
    manage.py
    db.sqlite3
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
```
这些目录和文件的用处是：
- 最外层的 mysite/ 根目录只是你项目的容器， 根目录名称对 Django 没有影响，你可以将它重命名为任何你喜欢的名称。 
- manage.py: 一个让你用各种方式管理 Django 项目的命令行工具。(用处很多，但是还需要进一步学习才能了解) 
- 里面一层的 mysite/ 目录包含你的项目，它是一个纯 Python 包。它的名字就是当你引用它内部任何东西时需要用到的 Python 包名。 (比如 mysite.urls)。 
- db.sqlite3：一个空的数据库文件，感觉是测试生成，暂时没什么意义。 
- mysite/__init__.py：一个空文件，告诉 Python 这个目录应该被认为是一个 Python 包。如果你是 Python 初学者，阅读官方文档中的 更多关于包的知识。 
- mysite/settings.py：Django 项目的配置文件。 
- mysite/urls.py：Django 项目的 URL 声明，就像你网站的“目录”。 
- mysite/asgi.py：作为你的项目的运行在 ASGI 兼容的 Web 服务器上的入口。（暂时不使用） 
- mysite/wsgi.py：作为你的项目的运行在 WSGI 兼容的Web服务器上的入口。（暂时不使用） 

生成好项目文件后，我们在mysite目录下执行： 
```
$ python manage.py runserver
```
这一步如果是docker环境下，必须要直接写到docker-compose.yml中，否则无法访问到对应的浏览器页面！
在命令行下我们可以看到如下打印： 
```
mydjango  | System check identified no issues (0 silenced).
mydjango  | August 04, 2021 - 11:31:44
mydjango  | Django version 3.2.5, using settings 'mysite.settings'
mydjango  | Starting development server at http://0.0.0.0:1234/
mydjango  | Quit the server with CONTROL-C.

```
仅需看到这一部分就知道我们的部署是成功的，并且可以进行下一步操作了。 
通过127.0.0.1：1234我们可以访问到一个火箭起飞的页面，说明启动服务成功，端口也返回了数据。
## 创建投票应用
目前对于各个视图的功能还不了解，这里我们仍然按原文档进行学习，主要保证对框架有一个基本的了解。 
在处于 manage.py 所在的目录下，然后运行这行命令来创建一个应用： 
```
$ python manage.py startapp polls
```
这将会创建一个 polls 目录，它的目录结构大致如下： 
```
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
```
这个目录结构包括了投票应用的全部内容。 
## 编写第一个视图
让我们开始编写第一个视图吧。打开 polls/views.py，把下面这些 Python 代码输入进去：
```
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```
很明显这里调用HttpResponse方法，来返回了一句话。那么这里我们也可以了解到，views的作用就是页面，就是展示。我的一切效果展示通过views来进行输出。 
这是 Django 中最简单的视图。如果想看见效果，我们需要将一个 URL 映射到它——这就是我们需要 URLconf 的原因了。 
为了创建 URLconf，请在 polls 目录里新建一个 urls.py 文件。你的应用目录现在看起来应该是这样： 
```
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    urls.py
    views.py
```
在 polls/urls.py 中，输入如下代码： 
```
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```
这个界面看起来是用来实现url组建，但是实现方式不详，path方法需要再确认一下。
下一步是要在根 URLconf 文件中指定我们创建的 polls.urls 模块。在 mysite/urls.py 文件的 urlpatterns 列表里插入一个 include()， 如下：
```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```
这里在Django生成下是有说明的： 
```
The `urlpatterns` list routes URLs to views. For more information please see:
    https://docs.djangoproject.com/en/3.2/topics/http/urls/
Examples:
Function views
    1. Add an import:  from my_app import views
    2. Add a URL to urlpatterns:  path('', views.home, name='home')
Class-based views
    1. Add an import:  from other_app.views import Home
    2. Add a URL to urlpatterns:  path('', Home.as_view(), name='home')
Including another URLconf
    1. Import the include() function: from django.urls import include, path
    2. Add a URL to urlpatterns:  path('blog/', include('blog.urls'))
```
这里列举了集中页面以及相应的配置方法，与我看到的path方法一致，具体每个views对应什么功能页，还不太了解。

函数 include() 允许引用其它 URLconfs。每当 Django 遇到 include() 时，它会截断与此项匹配的 URL 的部分，并将剩余的字符串发送到 URLconf 以供进一步处理。 
我们设计 include() 的理念是使其可以即插即用。因为投票应用有它自己的 URLconf( polls/urls.py )，他们能够被放在 "/polls/" ， "/fun_polls/" ，"/content/polls/"，或者其他任何路径下，这个应用都能够正常工作。 
这个时候我们创建完index页，重启容器环境，尝试访问http://127.0.0.1:1234/polls/ 就能够看见能够看见 "Hello, world. You're at the polls index." 
函数 path() 具有四个参数，两个必须参数：route 和 view，两个可选参数：kwargs 和 name。现在，是时候来研究这些参数的含义了。 
### path() 参数： route
route 是一个匹配 URL 的准则（类似正则表达式）。当 Django 响应一个请求时，它会从 urlpatterns 的第一项开始，按顺序依次匹配列表中的项，直到找到匹配的项。 
这里明显匹配的是polls这串字符。
### path() 参数： view
当 Django 找到了一个匹配的准则，就会调用这个特定的视图函数，并传入一个 HttpRequest 对象作为第一个参数，被“捕获”的参数以关键字参数的形式传入。稍后，我们会给出一个例子。 
这个说明不太能理解，猜测是从第一段的` include('polls.urls')`处跳转到polls目录下的urls.py，并以path('', views.index, name='index')为响应体，进入polls.index。 
另外两个可选参数在这里没有体现，暂时不做考虑。
