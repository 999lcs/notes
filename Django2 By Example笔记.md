### Django2 By Example笔记

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
#filter&publish__year
Post.objects.filter(publish__year=2017)
#filter&author__username
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

到`mysite`目录下编辑`urls.py`：

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('blog/', include('blog.urls', namespace='blog')),
]
```

##### **反向解析url**

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