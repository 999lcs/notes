### Django2 By Example笔记

[TOC]

#### 第一章 创建博客应用

##### **时区及语言**

```python
# LANGUAGE_CODE = 'en-us'
LANGUAGE_CODE = 'zh-hans'

# TIME_ZONE = 'UTC'
TIME_ZONE = 'Asia/Shanghai'

USE_I18N = True

USE_L10N = True

USE_TZ = True
# USE_TZ = False

# 数据库返回时区值错误，os执行：mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root -p mysql

https://blog.csdn.net/qq_27361945/article/details/80580795
看了官方文档，https://docs.djangoproject.com/en/2.1/topics/i18n/timezones/
Time zones¶
Overview¶
When support for time zones is enabled, Django stores datetime information inUTC in the database, uses time-zone-aware datetime objects internally, andtranslates them to the end user’s time zone in templates and forms.
翻译一下：当使用时区时，Django存储在数据库中的所有日期时间信息都以UTC时区为准，在后台使用有时区的datetime，前台用户使用时，在网页上翻译成用户所在的时区。
```

##### **django安装**

```python
#python
» python 
Python 3.8.3 (v3.8.3:6f8c8320e9, May 13 2020, 16:29:34) 

#python3 path
» which python3 
/Library/Frameworks/Python.framework/Versions/3.8/bin/python3

#安装django(在虚拟环境内安装)
pip install Django==3.1.1
>>> import django
>>> django.get_version()

#django path
» which django-admin                               
/Library/Frameworks/Python.framework/Versions/3.8/bin/django-admin
```

##### **virtualenv安装**

```bash


#安装virtualenv
pip install virtualenv

#python3,django-admin,virtualenv path
export PATH="/Library/Frameworks/Python.framework/Versions/3.8/bin:$PATH"

#创建虚拟环境
$ virtualenv my_env

#指定python3创建虚拟环境
$ virtualenv my_env -p /Library/Frameworks/Python.framework/Versions/3.6/bin/python3  

#虚拟环境的包路径
/Users/liuchuanshi/PycharmProjects/blog/my_env/lib/python3.8/site-packages

#激活虚拟环境
➜  ~ source activate                                                                     
(my_env) ➜  ~ 

#退出虚拟环境
(my_env) ➜  ~ deactivate

#packages迁移,处在虚拟环境内：
pip freeze > requirements.txt # 环境迁出，txt文件可以任意命名
pip install -r requirements.txt # 环境迁入

#virtualenv，virtualenvwrapper安装及使用
#链接：https://www.cnblogs.com/zixinyu/p/11308659.html
mkvirtualenv 环境名：创建环境

workon：当前存在环境列表

workon 环境名：选择环境

rmvirtualenv 环境名：删除环境

mkproject mic：创建mic项目和运行环境mic
	
mktmpenv：创建临时运行环境

lsvirtualenv：列出可用的运行环境

cdvirtualenv：进入虚拟环境目录

cdsitepackages：进入虚拟环境的site-packages目录

lssitepackages: 列出当前环境安装了的包

deactivate：退出环境
```

##### **创建项目和应用**

```bash
#建立项目目录，并创建项目
django-admin startproject mysite

#项目设置
settings.py

#迁移库表
python manage.py migrate

#运行站点
python manage.py runserver

#创建应用
python manage.py startapp blog

pycharm打开项目，并选择已存在的虚拟环境

#项目设置
settings.py

#设计数据结构
定义一个Post类，在blog应用下的models.py

#激活应用
编辑settings.py文件，添加blog.apps.BlogConfig到INSTALLED_APPS设置中

#创建和执行迁移
python manage.py makemigrations blog
python manage.py migrate
```

##### **更换sqlite为mysql**

```shell
#安装mysql后，才能安装mysqlclient，并且不能删除mysql
$ brew install mysql  
$ pip install mysqlclient

#settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'testblog',
        'USER': 'root',
        'PASSWORD': 'root',
        'HOST': '127.0.0.1',
        'PORT': '3306',
    },
}
```

##### **管理后台**

```python
#为数据模型创建管理后台站点
python manage.py createsuperuser
http://127.0.0.1:8000/admin/

#向管理后台内添加模型，自定义显示（编辑blog应用的admin.py）
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
```

##### **QuerySet**

```python
#get
user = User.objects.get(username='admin')
#create
post = Post(title='Another post', slug='another-post', body='Post body', author = user)
#save
post.save()
#create
Post.objects.create(title='One more post', slug='One more post', body='Post body', author=user)
#update
post.title = 'New title'
#all
all_posts = Post.objects.all()
#filter&publish__year,条件查询采用双下划线写法
Post.objects.filter(publish__year=2017)
#filter&author__username,双下划线还一个用法是从关联的模型中取其字段
Post.objects.filter(author__username='admin')
#链式调用
Post.objects.filter(publish__year=2017).filter(author__username='admin')
#exclude
Post.objects.filter(publish__year=2017).exclude(title__startswith='Why')
#order_by
Post.objects.order_by('-title')
#delete
Post.objects.get(id=39).delete()
#切片
Post.objects.all()[:3]
```

##### **模型管理器**

一是给默认的管理器增加新的方法：`Post.objects.my_manager()`

二是修改默认的管理器：`Post.my_manager.all()`

```python
class PublishedManager(models.Manager):
    def get_queryset(self):
        return super(PublishedManager, self).get_queryset().filter(status='published')

class Post(models.Model):
    # ......
    objects = models.Manager()  # 默认的管理器
    published = PublishedManager()  # 自定义管理器
    
#取得所有标题开头是Who，而且已经发布的文章
Post.published.filter(title__startswith="Who")    
```

##### **视图函数**

`blog`应用的`views.py`文件：

```python
from django.shortcuts import render, get_object_or_404
from .models import Post

def post_list(request):
    posts = Post.published.all()
    return render(request, 'blog/post/list.html', {'posts': posts})
```

```python
def post_detail(request, year, month, day, post):
    post = get_object_or_404(Post, slug=post, status="published", publish__year=year, publish__month=month, publish__day=day)
    return render(request, 'blog/post/detail.html', {'post': post})
```

##### **为视图配置URL**

<u>URL pattern的作用是将URL映射到视图上。Django接到对于某个URL的请求时，按照顺序从上到下试图匹配URL，停在第一个匹配成功的URL处，将`HttpRequest`类的一个实例和其他参数传给对应的视图并调用视图处理本次请求。</u>

在`blog`应用下目录下边新建一个`urls.py`文件，然后添加如下内容：

```python
from django.urls import path
from . import views

app_name = 'blog'
urlpatterns = [
    # post views
    path('', views.post_list, name='post_list'),
    path('<int:year>/<int:month>/<int:day>/<slug:post>/', views.post_detail, name='post_detail'),
]
```

到`mysite`目录下编辑`urls.py`，之后我们方便的通过使用命名空间来快速指向具体的URL，例如`blog:post_list`和`blog:post_detail`：

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('blog/', include('blog.urls', namespace='blog')),
]
```

##### **反向解析URL**

为Post模型的每一个数据对象创建规范化的URL，添加一个`get_absolute_url()`方法，该方法返回对象的URL。我们将使用`reverse()`方法通过名称和其他参数来构建URL。之后在模板中，就可以使用`get_absolute_url()`创建超链接到具体数据对象。编辑`models.py`文件

```python
from django.urls import reverse

class Post(models.Model):
    # ......
    def get_absolute_url(self):
        return reverse('blog:post_detail', args=[self.publish.year, self.publish.month, self.publish.day, self.slug])
```

##### **模板**

`post/list.html`：

```django
{% extends "blog/base.html" %}
{% block title %}My Blog{% endblock %}
{% block content %}
    <h1>My Blog</h1>
    {% for post in posts %}
        <h2>
            <a href="{{ post.get_absolute_url }}">
                {{ post.title }}
            </a>
        </h2>
        <p class="date">
        Published {{ post.publish }} by {{ post.author }}
        </p>
        //{{ post.body|truncatewords:30|linebreaks }}
				{{ post.body|slice:70|linebreaks }}
    {% endfor %}
{% endblock %}
```

`post/detail.html`：

```django
{% extends 'blog/base.html' %}
{% block title %}
{{ post.title }}
{% endblock %}

{% block content %}
    <h1>{{ post.title }}</h1>
    <p class="date">
    Published {{ post.publish }} by {{ post.author }}
    </p>
    {{ post.body|linebreaks }}
{% endblock %}
```

##### **分页**

**单独分页：**

编辑`blog`应用的`views.py`文件，修改`post_list`视图：

```python
from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger

def post_list(request):
    object_list = Post.published.all()
    paginator = Paginator(object_list, 3)  # 每页显示3篇文章
    page = request.GET.get('page')
    try:
        posts = paginator.page(page)
    except PageNotAnInteger:
        # 如果page参数不是一个整数就返回第一页
        posts = paginator.page(1)
    except EmptyPage:
        # 如果页数超出总页数就返回最后一页
        posts = paginator.page(paginator.num_pages)
    return render(request, 'blog/post/list.html', {'page': page, 'posts': posts})
```

**通用分页模板：**

在`blog`应用的`templates/`目录中新建`pagination.html`

```django
<div class="pagination">
    <span class="step-links">
        {% if page.has_previous %}
        <a href="?page={{ page.previous_page_number }}">Previous</a>
        {% endif %}
    <span class="current">
        Page {{ page.number }} of {{ page.paginator.num_pages }}.
    </span>
    {% if page.has_next %}
        <a href="?page={{ page.next_page_number }}">Next</a>
    {% endif %}
    </span>
</div>
```

`blog/post/list.html`文件，在`{% content %}`中的最下边增加一行：

```django
{% block content %}
    # ......
    {% include 'pagination.html' with page=posts %}
{% endblock %}
```

##### **基于类的视图**

**CBV相比FBV有如下优点：**

- 可编写单独的方法对应不同的HTTP请求类型如GET，POST，PUT等请求，不像FBV一样需要使用分支
- 使用多继承创建可复用的类模块（也叫做*mixins*）

**ListView（列出任意类型的数据）：**

编辑`blog`应用的`views.py`文件，添加下列代码：

```python
from django.views.generic import ListView
class PostListView(ListView):
    queryset = Post.published.all()
   # model = Post
    context_object_name = 'posts'  # 如不设置context_object_name，默认变量名是object_list
    paginate_by = 3
    template_name = 'blog/post/list.html'  # 如果不指定，默认使用blog/post_list.html
```

打开`blog`应用的`urls.py`文件，注释掉刚才的`post_list` URL pattern，为PostListView类增加一行：

```python
urlpatterns = [
    # post views
    # path('', views.post_list, name='post_list'),
    path('',views.PostListView.as_view(),name='post_list'),
    path('<int:year>/<int:month>/<int:day>/<slug:post>/', views.post_detail, name='post_detail'),
]
```

Django内置的`ListView`返回的变量名称叫做`page_obj`,所以必须修改`post/list.html`中导入分页模板的那行代码：

```django
{% include 'pagination.html' with page=page_obj %}
```

##### **django应用架构图**



![20201007175701](https://tva1.sinaimg.cn/large/007S8ZIlly1gjlq6zse42j30lj0g6dgd.jpg)



#### 第二章 增强博客功能

##### **通过邮件分享文章**

**创建表单：**

- `Form`：用于生成标准的表单
- `ModelForm`：用于从模型生成表单

在`forms.py`文件，然后编写：

```python
from django import forms

class EmailPostForm(forms.Form):
    name = forms.CharField(max_length=25)
    email = forms.EmailField()
    to = forms.EmailField()
    comments = forms.CharField(required=False, widget=forms.Textarea)
```

**通过视图处理表单：**

编辑`blog`应用的`views.py`文件：

```python
from .forms import EmailPostForm

def post_share(request, post_id):
    # 通过id 获取 post 对象
    post = get_object_or_404(Post, id=post_id, status='published')
    if request.method == "POST":
        # 表单被提交
        form = EmailPostForm(request.POST)
        if form.is_valid():
            # 验证表单数据
            cd = form.cleaned_data
            # 发送邮件......
    else:
        form = EmailPostForm()
    return render(request, 'blog/post/share.html', {'post': post, 'form': form})
```

**SMTP服务器设置：**

在`settings.py`文件中加入SMTP服务器设置：

```python
EMAIL_HOST = 'smtp.sina.com'
EMAIL_HOST_USER = "999lcs@sina.com"
EMAIL_HOST_PASSWORD = "b42de0d8f5f06d8a"
EMAIL_PORT = 587
EMAIL_USE_TLS = True
DEFAULT_FROM_EMAIL = "999lcs@sina.com"
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend' # 将邮件打印在控制台
```

输入`python manage.py shell`，在命令行环境中试验一下发送邮件的指令：

```python
from django.core.mail import send_mail
send_mail('Django mail', 'This e-mail was sent with Django.', 'your_account@gmail.com', ['your_account@gmail.com'], fail_silently=False)
# 参数分别是邮件标题、邮件内容、发件人和收件人地址列表，最后一个参数`fail_silently=False`表示如果发送失败就抛出异常。如果看到返回1，就说明邮件成功发送。
```

**视图中增加发邮件功能：**

把发送邮件的功能加入到视图中，编辑`views.py`中的`post_share`视图函数：

```python
def post_share(request, post_id):
    sent = False
            # ......
            # 发送邮件
            post_url = request.build_absolute_uri(post.get_absolute_url())
            subject = '{} ({}) recommends you reading "{}"'.format(cd['name'], cd['email'], post.title)
            message = 'Read "{}" at {}\n\n{}\'s comments:{}'.format(post.title, post_url, cd['name'], cd['comments'])
            send_mail(subject, message, 'lee0709@vip.sina.com', [cd['to']])
            sent = True
    else:
        form = EmailPostForm()
    return render(request, 'blog/post/share.html', {'post': post, 'form': form, 'sent': sent})
```

还需要给视图配置URL，打开`blog`应用中的`urls.py`，加一条`post_share`的URL pattern：

```python
urlpatterns = [
    # ...
    path('<int:post_id>/share/', views.post_share, name='post_share'),
]
```

**在模板中渲染表单：**

在`blog/templates/blog/post/`目录内创建share.html，添加如下代码：

```python
{% extends "blog/base.html" %}

{% block title %}Share a post{% endblock %}

{% block content %}
    {% if sent %}
        <h1>E-mail successfully sent</h1>
        <p>
            "{{ post.title }}" was successfully sent to {{ form.cleaned_data.to }}.
        </p>
    {% else %}
        <h1>Share "{{ post.title }}" by e-mail</h1>
        <form action="." method="post">
        {{ form.as_p }}
            {% csrf_token %}
            <input type="submit" value="Send e-mail">
        </form>
    {% endif %}
{% endblock %}
```

修改`blog/post/detail.html`将下列链接增加到`{{ post.body|linebreaks }}`之后：

```python
<p>
    <a href="{% url "blog:post_share" post.id %}">Share this post</a>
</p>
```

这里的`{% url %}`标签，其功能和在视图中使用的`reverse()`方法类似，使用URL的命名空间`blog`和URL命名`post_share`，再传入一个ID作为参数，就可以构建出一个URL。在页面渲染时，`{% url %}`就会被渲染成反向解析出的URL。

关闭浏览器验证的方法是给表单添加`novalidate`属性：`<form action="" **novalidate**>`







