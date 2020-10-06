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

#pycharm打开项目，并选择已存在的虚拟环境

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

