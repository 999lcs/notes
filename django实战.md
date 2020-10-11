### django学习笔记

#### 20200804

##### login表单

###### view函数GET请求

```python
if request.method == "GET":
  login_form = LoginForm()
  print(login_form)
  # 如果客户端对服务器发出GET请求，则将login_form = loginForm()传给指定模板，从而在前端呈现相应的输入框。
  return render(request, "account/account.html", {"form":login_form})
```

通过contexts（{"form":login_form}）--字典对象，返回给指定模板（account/account.html）的HTML代码

```html
<tr><th><label for="id_username">Username:</label></th><td><input type="text" name="username" value="lcs" required id="id_username" /></td></tr>
<tr><th><label for="id_password">Password:</label></th><td><input type="password" name="password" required id="id_password" /></td></tr>
```

###### contexts（上下文）



###### 带有输入框和确认按钮的表单

```html
<p>第一个单选按钮在表单之外,但它仍属于该表单的一部分。尝试点击文本标签切换单选按钮。</p>
<form action="demo-form.php">
First name: <input type="text" name="FirstName" value="Mickey"><br>
Last name: <input type="text" name="LastName" value="Mouse"><br>
<input type="submit" value="提交">
</form>
```

###### 带有两个输入字段和相关标记的简单 HTML 表单

```html
<p>点击其中一个文本标签选中选项：</p>
<form action="demo_form.php">
  <label for="male">Male</label>
  <input type="radio" name="sex" id="male" value="male"><br>
  <label for="female">Female</label>
  <input type="radio" name="sex" id="female" value="female"><br><br>
  <input type="submit" value="提交">
</form>
```

###### request对象

method：`if request.method == "POST":`；返回类字典对象：`column_name = request.POST['column']`

![image-20200805080524143](https://raw.githubusercontent.com/999lcs/img/main/md/007S8ZIlly1gjfqqq2kufj30hf0e8jui-20201011172945368.jpg?token=AK4FYCZP4HXYBLRREG2FRKC7QLIMQ)

###### 登录后调用session

![image-20200804233025592](https://raw.githubusercontent.com/999lcs/img/main/md/007S8ZIlly1gjfqqv75i9j30h104j41t-20201011172929384.jpg?token=AK4FYC5QK2KHF3HJLPGPHTC7QLILO)



#### 20200805

##### 新增栏目功能

###### view

```python
def article_column(request):
    if request.method == "GET":
        columns = ArticleColumn.objects.filter(user=request.user)
        column_form = ArticleColumnForm()
        print(column_form)
        print("***def article_column(request)--print(column_form)***")
        return render(request, "article/column/article_column.html", {"columns": columns, "column_form": column_form})

    if request.method == "POST":
        column_name = request.POST['column1'] //前端传过来的
        columns = ArticleColumn.objects.filter(user_id=request.user.id, column=column_name)
        if columns:
            return HttpResponse('2')
        else:
            ArticleColumn.objects.create(user=request.user, column=column_name)
            return HttpResponse('1')
```

###### html

```javascript
<div>
    <p class="text-right">
        <button id="add_column" onclick="add_column()" class="btn btn-primary">add column</button>
    </p>
</div>
```

###### javascritp

```javascript
<script type="text/javascript" src='{% static "js/layer.js" %}'></script>
<script type="text/javascript">

    function add_column() {
        var index = layer.open({
            type: 1,
            skin: "layui-layer-rim",
            area: ["400px", "200px"],
            title: "新增栏目",
            content: '<div class="text-center" style="margin-top:20px"><p>请输入新的栏目名称</p>' +
                //'<p><input type="text"></p></div>',
                '<p>{{ column_form.column }}</p></div>',  
          //输入框，继承自父页面request返回的context,
          //{{ column_form.column }}等于：
          //<input type="text" name="column" required id="id_column" maxlength="200" />
            btn: ['确定', '取消'],
            yes: function (index, layero) {
                column_name = $('#id_column').val();  
              // 输入框的id：<input type="text" name="column" required id="id_column"/>
                $.ajax({
                    url: '{% url "article:article_column" %}',
                    type: 'POST',
                    data: {"column1": column_name},  //变量column1是和后端约定好的
                    success:function(e) {
                        if (e == "1") {
                            parent.location.reload();
                            layer.msg("good");
                        } else {
                            layer.msg("此栏目已有，请更换名称");
                        }
                    },
                });
            },
            btn2: function (index, layero) {
                layer.close(index);
            }
        });
    }
    </script>
```

###### 重点代码：

```javascript
#view返回表单对象：column_form：
return render(request, "article/column/article_column.html", {"columns": columns, "column_form": column_form})

#js新开窗口获取后台传送的表单对象{{ column_form.column }}：
content: '<div class="text-center" style="margin-top:20px"><p>请输入新的栏目名称</p>' + '<p>{{ column_form.column }}</p></div>', 

#column_form（表单对象）：
<tr><th><label for="id_column">Column:</label></th><td><input type="text" name="column" required id="id_column" maxlength="200" /></td></tr>
  
#column_form.column（没有标签的表单对象）：
<input type="text" name="column" required id="id_column" maxlength="200" />
```

###### **总结：**

JavaScript弹窗可以直接引用父页面的模板变量并获取后台数据，同时JavaScript新打开的页面实际已经渲染好了，只是等着用户点击加载而已；本质上JavaScript弹窗是继承父页面之前同步交互，除非另外再使用AJAX进行异步交互。如下，弹窗中引用父页面模板变量{{ columns }}的效果：

![image-20200805153055605](https://raw.githubusercontent.com/999lcs/img/main/md/007S8ZIlly1gjfqswpjt6j30bh093dg8-20201011172955002.jpg?token=AK4FYC67I3EXBLMQV2HETBC7QLINC)



#### 20200813

##### 表单类

![截屏2020-08-14 上午12.17.55](https://raw.githubusercontent.com/999lcs/img/main/md/007S8ZIlly1gjfqt1afpvj30ei0khdkq-20201011173000812.jpg?token=AK4FYCZGFC36PTXZ7JGNXVC7QLINQ)



#### 20200814

##### 三种login

```python
url(r'^login/$', views.user_login, name="user_login"), 
# 自定义login函数

from django.contrib.auth import views as auth_views
url(r'^login/$', auth_views.login, name='user_login'),
# 使用django内置login函数；
# 因内置longin函数：template_name='registration/login.html'，将该模板从django目录复制到项目模板；
# 因内置longin函数：redirect_field_name=REDIRECT_FIELD_NAME，修改settings.py文件,增加LOGIN_REDIRECT_URL = '/blog/'，使登录后重定向到blog页面。

url(r'^new-login/$', auth_views.login, {"template_name": "account/login.html"}), 
# 使用django内置login函数，并自定义模板文件位置：account/login.html
```

#### 20200816

##### redis

###### 下载、解压、编译Redis:

```shell
$ wget http://download.redis.io/releases/redis-5.0.5.tar.gz
$ tar xzf redis-5.0.5.tar.gz
$ cd redis-5.0.5
$ make
```

进入到解压后的 `/opt/redis-5.0.5/src` 目录，通过如下命令启动Redis:

```shell
$ src/redis-server
```

您可以使用内置的客户端与Redis进行交互:

```shell
$ src/redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```

###### 安装redis库:

`sudo pip install redis`

###### python交互:

```python
import redis
r = redis.StrictRedis(host="localhost", port=6379, db=0)
r.set("foo", "bar")
True
r
Redis<ConnectionPool<Connection<host=localhost,port=6379,db=0>>>
r.get("foo")
'bar'
```

###### 数据库配置:

mysite/settings.py

```python
REDIS_HOST = 'localhost'
REDIS_PORT = 6379
REDIS_DB = 0
```

#### 20200817

##### redis

```shell
#启动客户端
$ ./redis-cli
#设置键
127.0.0.1:6379> set foo bar
#获取键值
127.0.0.1:6379> get foo
"bar"
#获取所有键
127.0.0.1:6379> KEYS *
#获取特定键
127.0.0.1:6379> KEYS *views
 1) "article:62:views"
 2) "article:64:views"

#article:(article.id).views -- 对象类型：对象id：对象属性
total_views = r.incr("article:{}:views".format(article.id))
#获取ariticle.id=192的文章阅读次数
127.0.0.1:6379> GET article:192:views
"110"

#zincrby()函数原型为zincrby(self, name, amount, value)，其中name为有序集合名，amount为数值，value为键名
r.zincrby('article_ranking', 1, article.id)
```

#### 20200818

##### print中文乱码问题：

```python
# -*- coding: utf-8 -*-
from __future__ import print_function
```

##### redis删除键和清空所有键：

```
127.0.0.1:6379> DEL article_ranking

(integer) 1

127.0.0.1:6379> FLUSHALL

OK

127.0.0.1:6379> 
```

##### [使用redis实现【统计文章阅读量】及【最热文章】功能](https://www.cnblogs.com/gcgc/p/10670528.html)

###### 1.视图函数

```python
# 不需要登录装饰器，匿名用户也可访问<br>def article_detail(request, id, slug):
# print(slug,id)
article = get_object_or_404(ArticlePost, id=id, slug=slug)
# 连接redis
r = redis.StrictRedis(host=settings.REDIS_HOST, port=settings.REDIS_PORT, db=settings.REDIS_DB)
# 总的访问次数，访问一次就+1，一般命名规则为"对象类型：对象ID：对象属性"
total_views = r.incr('article:{}:views'.format(article.id))
# zincrby(name, amount, value)方法:根据amount设定的步长增加有序集合name中的value的分值（类似于权重）
# 实现了每访问一次文章就会将article_ranking中的article.id分值增加1
# article_ranking中存放的是文章的id用来代表文章，每访问一次该文章就会增加文章的分值
r.zincrby('article_ranking', 1, article.id)
# 获取分值排名前十的对象
article_ranking = r.zrange('article_ranking', 0, -1, desc=True)[:10]
# 获取排名前十文章的id列表,使用的是列表推导式，先进行for循环，再将每次的的值带入int()方法运算，将结果放在新的列表中
article_ranking_ids = [int(id) for id in article_ranking]
print('文章浏览量对应的id：%s' % article_ranking_ids)
# 查询出排名在前十的文章对象,并放在list中。注意id__in用法：id在article_ranking_ids列表中
most_viewed = list(ArticlePost.objects.filter(id__in=article_ranking_ids))
print('文章未排序：%s' % most_viewed)
# 将获得的列表按照下表索引进行排序，lamda为匿名函数，先运算后面表达式，冒号前的x相当于参数，代表most_viewed列表中文章对象
# 按照文章的id得到对应的下标,再按照下标进行排序
most_viewed.sort(key=lambda x: article_ranking_ids.index(x.id))
print('文章已经排序：%s' % most_viewed)
return render(request, 'article/column/article_detail.html', {'article': article,
                                                              'total_views': total_views,
                                                              'most_viewed': most_viewed})
```

###### 2.前端页面

```html
{% extends 'article/base.html' %}
{% load staticfiles %}
{% block title %}article detail{% endblock %}

{% block content %}
<div class="container">
    <div class="col-md-9">
        <header>
            <h1>{{ article.title }}</h1>
            <p>{{ user.username }} {{ total_views }}次阅读</p>
        </header>

        <link rel="stylesheet" href="{% static 'editor/css/editormd.preview.css' %}">

        <div id="editormd-view">
            <textarea id="append-test" style="display: none">
    {{ article.body }}
            </textarea>
        </div>
    </div>
    <div class="col-md-3">
        <p class="text-center">最受欢迎文章</p>
        <ol>
            {% for article_rank in most_viewed %}
            <li>
                <a href="{{ article_rank.get_url_path }}">{{ article_rank.title }}</a>
            </li>
            {% endfor %}
        </ol>
    </div>

</div>
    <script src="{% static 'js/jquery.js' %}"></script>
    <script src="{% static 'editor/lib/marked.min.js' %}"></script>
    <script src="{% static 'editor/lib/prettify.min.js' %}"></script>
    <script src="{% static 'editor/lib/raphael.min.js' %}"></script>
    <script src="{% static 'editor/lib/underscore.min.js' %}"></script>
    <script src="{% static 'editor/lib/sequence-diagram.min.js' %}"></script>
    <script src="{% static 'editor/lib/flowchart.min.js' %}"></script>
    <script src="{% static 'editor/lib/jquery.flowchart.min.js' %}"></script>
    <script src="{% static 'editor/editormd.js' %}"></script>

    <script type="text/javascript">
            $(function () {
                editormd.markdownToHTML("editormd-view",{
                    htmlDecode:"style,script,iframe",
                    emoji:true,
                    tasklist:true,
                    flowChart:true,
                    tex:true,
                    sequenceDiagram:true,
                });
            });
    </script>
{% endblock %}
```

##### 个人网站view函数：

```python
def article_detail(request, id, slug):
    article = get_object_or_404(ArticlePost, id=id, slug=slug)

    total_views = r.incr("article:{}:views".format(article.id))  # article:(article.id).views -- 对象类型：对象id：对象属性

    a=r.zincrby('article_ranking', 1, article.id)  #访问1次，文章分数加1;zincrby(self, name, amount, value)
    print('1文章id:', article.id, '2文章title:', article.title, '3文章分数', a)

    article_ranking = r.zrange('article_ranking', 0, -1, desc=False)[:10]  #获取分值排名前十的文章对象
    print('top10:',article_ranking)

    article_ranking_ids = [int(id) for id in article_ranking]   #获取排名前十的文章的id列表,使用的是列表推导式，先进行for循环，再将每次的的值带入int()方法运算，将结果放在新的列表中
    print("文章分数排名前10的文章id列表:", article_ranking_ids)

    most_viewed = list(ArticlePost.objects.filter(id__in=article_ranking_ids)) #查询出排名在前十的文章对象,并放在list中。注意id__in用法：id在article_ranking_ids列表中
    print("未排序的文章对象列表:", most_viewed)
    most_viewed.sort(key=lambda x: article_ranking_ids.index(x.id))  #x.id为文章的id，article_ranking_ids.index(x.id)得到id列表的索引，按索引对文章对象进行排序
    print('已排序的文章对象列表：', most_viewed)
    #取出文章id；找到文章id列表的索引位置，按索引位置对文章对象进行排序；比如文章id=193，在ids列表中索引位置为2，就把文章排序到索引位置2
    #总结：ids是分值前10的文章id列表；文章对象再按照ids列表进行排序;等于先id排序，再文章排序。

    return render(request, "article/list/article_detail.html", {"article":article, "total_views":total_views, "most_viewed":most_viewed})
```



##### Lambda表达式

Lambda表达式是Python中一类特殊的定义函数的形式，使用它可以定义一个匿名函数。与其它语言不同，Python的Lambda表达式的函数体只能有单独的一条语句，也就是返回值表达式语句。其语法如下： [2] 

**lambda 形参列表 : 函数返回值表达式语句**

下面是个Lambda表达式的例子：

```python
#!/usr/bin/envpython
li=[{"age":20,"name":"def"},{"age":25,"name":"abc"},{"age":10,"name":"ghi"}]
li=sorted(li, key=lambda x:x["age"])
print(li)
```

如果不用Lambda表达式，而要写成常规的函数，那么需要这么写：

```python
#!/usr/bin/envpython
def comp(x):
    return x["age"]
li=[{"age":20,"name":"def"},{"age":25,"name":"abc"},{"age":10,"name":"ghi"}]
li=sorted(li, key=comp)
print(li)
```



#### 20200821

##### pymysql配置

mac环境安装MySQLdb非常困难，且MySQLdb只支持Python2.x，还不支持3.x
 可以用PyMySQL代替。安装方法：pip install PyMySQL
 然后在需要的项目中，在`__init__.py`中添加两行：

```python
import pymysql
pymysql.install_as_MySQLdb()
```

就可以用 import MySQLdb了。其他的方法与MySQLdb一样。



##### 聚合计数

```python
#每位作者文章数
ArticlePost.objects.all().values('author_id').annotate(count=Count('author')).values('author_id', 'count').order_by('author_id')
<QuerySet [{'count': 7, 'author_id': 20}, {'count': 3, 'author_id': 21}, {'count': 14, 'author_id': 23}]>
#每篇文章的评论数
Comment.objects.all().values('article_id').annotate(count=Count('article')).values('article_id', 'count').order_by('article_id')
<QuerySet [{'count': 17, 'article_id': 188}, {'count': 3, 'article_id': 192}, {'count': 2, 'article_id': 198}]>
```



#### 20200822

##### [返回列表中字典值的列表](https://www.it1352.com/1807425.html)

```python
>>> array = [{'a':0,'b':1,'c':2}, {'a':3,'b':4,'c':5}, {'a':6,'b':2,'c':3},]
>>> [y for x in array for y in x.values()]
[0, 1, 2, 3, 4, 5, 6, 2, 3]
>>> [x['a'] for x in array]  # Assuming that all dicts have an 'a' key
[0, 3, 6]
>>>
```

```python
#sql：select article_id,count(*) as counts from article_comment group by article_id order by counts desc
aid=Comment.objects.all().values('article_id').annotate(count=Count('article')).values('article_id').order_by('article_id')
print(aid)
<QuerySet [{'article_id': 188}, {'article_id': 192}, {'article_id': 198}]>

a_list = [y for x in aid for y in x.values()]
print("list:", a_list)
('list:', [188, 192, 198])

#values 获取字典形式的结果
aid.values('article_id')
<QuerySet [{'article_id': 188}, {'article_id': 192}, {'article_id': 198}]>

#values_list 获取元组形式结果
aid.values_list('article_id')
<QuerySet [(188,), (192,), (198,)]>
```



#### 20200823

##### 三种方式渲染表格

##### 1、使用原生模版数据展示

- views

```python
def myhost(request):  # 1-使用原生模版数据展示
    myhost = MyHost.objects.all()
    myhost_form = myhost
    return render(request, "account/test.html", {"myhost_form":myhost_form, "myhost":myhost})
```

- html

```html
 <table id="test" class="display" style="width:100%">
    <thead>
        <tr>
            <th>序号</th>
            <th>主机名</th>
            <th>IP</th>
            <th>用途</th>
            <th>应用</th>
            <th>操作</th>
        </tr>
    </thead>
{#      <tbody></tbody>#}
     <tbody>
    {% for host in myhost %}
        <tr>
            <td>{{ host.id }}</td>
            <td>{{ host.hostname }}</td>
            <td>{{ host.ip }}</td>
            <td>{{ host.usage }}</td>
            <td>{{ host.app_name }}</td>
        </tr>
    {% endfor %}
</tbody>
 </table>
```

- url

```python
url(r'^myhost/$', views.myhost, name="myhost"),  #test!!!  先调用myhost视图函数，
```



##### 2、使用DataTable表格插件展示数据（最基本的展示）

- 引入js包：

```javascript
<link rel="stylesheet" type="text/css" href="http://cdn.datatables.net/1.10.15/css/jquery.dataTables.css">
<link rel="stylesheet" type="text/css" href="https://cdn.datatables.net/1.10.21/css/jquery.dataTables.min.css">
<link rel="stylesheet" type="text/css" href="https://cdn.datatables.net/buttons/1.6.2/css/buttons.dataTables.min.css">
<!-- jQuery -->
<script type="text/javascript" charset="utf8" src="http://code.jquery.com/jquery-1.10.2.min.js"></script>
<!-- DataTables -->
<script type="text/javascript" charset="utf8" src="http://cdn.datatables.net/1.10.15/js/jquery.dataTables.js"></script>
```

- 最简单的js，手动编写tbody表格（test），不从后台取数：

```javascript
<!-- BASE JS -->
 <script type="text/javascript">
     $(document).ready(function() {
        $('#test').DataTable();
    } );
 </script>
```

##### 3、使用DataTable表格插件展示数据（js处理数据）

- views

```python
def myhost(request):   # 2-使用js数据展示
    host_list = []
    for host_info in MyHost.objects.all():
        host_list.append({
            'id': host_info.id,
            'name': host_info.hostname,
            'ip': host_info.ip,
            'usage': host_info.usage,
            'appname': host_info.app_name,
        })
    return render(request, "account/test.html", {'myhost':json.dumps(host_list)})   #渲染test模板页面
```

- html

```html
 <table id="test" class="display" style="width:100%">
    <thead>
        <tr>
            <th>序号</th>
            <th>主机名</th>
            <th>IP</th>
            <th>用途</th>
            <th>应用</th>
            <th>操作</th>
        </tr>
    </thead>
      <tbody></tbody>
 </table>

<!-- JS -->
<script type="text/javascript">
    $(document).ready(function () {
        $("#test").DataTable({
            data: {{ myhost|safe }},
            columns: [
                {data: 'id'},
                {data: "name"},
                {data: "ip"},
                {data: "usage"},
                {data: "appname"}
            ]
        })
    });
</script>
```

- url

```python
url(r'^myhost/$', views.myhost, name="myhost"),  #test!!!  先调用myhost视图函数，
```

##### 4、使用DataTable表格插件展示数据（ajax处理数据，返回dict）

- views （返回dict，键名默认：host_dic['data'] ）

```python
def myhost(request):   # 3-使用ajax请求数据展示2
    print(request.POST)
    host_list = []
    for host_info in MyHost.objects.all():
        host_list.append({   # 组装成dic数据的列表
            'id': host_info.id,
            'name': host_info.hostname,
            'ip': host_info.ip,
            'usage': host_info.usage,
            'appname': host_info.app_name,
        })
    host_dic = {}
    host_dic['data'] = host_list  # 将整个host_list列表作为一个元素，组装成dic，并返回字典
    return 
HttpResponse(json.dumps(host_dic,ensure_ascii=False),content_type="application/json,charset=utf-8")
```

- html（ajax等页面加载后去调用myhost数据接口）

```html
 <table id="test" class="display" style="width:100%">
    <thead>
        <tr>
            <th>序号</th>
            <th>主机名</th>
            <th>IP</th>
            <th>用途</th>
            <th>应用</th>
            <th>操作</th>
        </tr>
    </thead>
      <tbody></tbody>
 </table>

<!-- AJAX DICT -->
<script type="text/javascript">
    $(document).ready(function () {
        $("#test").DataTable({
            ajax: "{% url 'account:myhost' %}",
            "columns": [
                {'data': 'id'},
                {"data": "name"},
                {"data": "ip"},
                {"data": "usage"},
                {"data": "appname"}
            ],
            "language": {url:"http://cdn.datatables.net/plug-ins/9dcbecd42ad/i18n/Chinese.json"}
        });
    });
</script>
```

- url

```python
url(r'^myhost/$', views.myhost, name="myhost"),  #test!!!  先调用myhost视图函数，
url(r'^test/$', views.test, name="test"),  #test!!!  先请求test页面，再从该页面发起ajax请求，由myhost视图函数返回json数据，渲染表格
```



##### 5、使用DataTable表格插件展示数据（ajax处理数据，使用dataSrc）

- views （返回dict，键名默认：host_dic['data1'] ）

```python
def myhost(request):   # 3-使用ajax请求数据展示2
    print(request.POST)
    host_list = []
    for host_info in MyHost.objects.all():
        host_list.append({   # 组装成dic数据的列表
            'id': host_info.id,
            'name': host_info.hostname,
            'ip': host_info.ip,
            'usage': host_info.usage,
            'appname': host_info.app_name,
        })
    host_dic = {}
    host_dic['data1'] = host_list  # 将整个host_list列表作为一个元素，组装成dic，并返回字典
    # return HttpResponse(json.dumps(host_dic,ensure_ascii=False),content_type="application/json,charset=utf-8")
    # return HttpResponse(json.dumps(host_list,ensure_ascii=False),content_type="application/json,charset=utf-8")
    return HttpResponse(json.dumps(host_dic),content_type="application/json")
```

- html

```html
 <table id="test" class="display" style="width:100%">
    <thead>
        <tr>
            <th>序号</th>
            <th>主机名</th>
            <th>IP</th>
            <th>用途</th>
            <th>应用</th>
            <th>操作</th>
        </tr>
    </thead>
      <tbody></tbody>
 </table>

<!-- AJAX LIST-->
<script type="text/javascript">
    $(document).ready(function () {
        $("#test").DataTable({

            dom: 'Bfrtip', // 导出文件
            button: [
                'copy', 'csv', 'excel', 'pdf', 'print'
            ],

            bProcessing: true, //DataTables载入数据时，是否显示‘进度’提示
            bautoWidth: true,//自动计算宽度
            {#ajax: "{% url 'account:myhost' %}",  //ajax默认模式，使用appname，返回字典#}
            {#ajax: '/account/myhost',   //默认模式，使用绝对路径，返回字典#}

        ajax: {    //ajax的dataSrc模式
            url: '/account/myhost',
            {#dataSrc: "",  // 为空默认返回列表#}
            {#dataSrc: "{{ host_list }}",  //返回列表#}
            {#dataSrc: "{{ host_dic }}",  //返回字典#}
            dataSrc: 'data1' //返回字典
        },
            "columns": [
                {'data': 'id',
                    width: "5%",
                    // 若想前端显示的不一样，则需要"render"函数
                    'render': function (data, type, full, meta) {
                        return meta.row + 1 + meta.settings._iDisplayStart;
                    }
                },
                { "data": "name",
                    'render': function (data, type, full, meta) {
                        return '<a class="text-warning" style="color:#008bff" title="该主机所属应用系统为：'+ full.appname +'">'+ data +'</a>';
                    }
                },
                {"data": "ip"},
                {"data": "usage"},
                {"data": "appname"}
            ],
            "language": {url:"http://cdn.datatables.net/plug-ins/9dcbecd42ad/i18n/Chinese.json"}
        })
    });
</script>
```

- url

```python
url(r'^myhost/$', views.myhost, name="myhost"),  #test!!!  先调用myhost视图函数，
url(r'^test/$', views.test, name="test"),  #test!!!  先请求test页面，再从该页面发起ajax请求，由myhost视图函数返回json数据，渲染表格
```

##### 6、使用DataTable表格插件展示数据完整版（增删改查）

- views （返回dict，键名默认：host_dic['data1'] ）

```python
def myhost(request):   # 3-使用ajax请求数据展示2
    print(request.POST)
    host_list = []
    for host_info in MyHost.objects.all():
        host_list.append({   # 组装成dic数据的列表
            'id': host_info.id,
            'name': host_info.hostname,
            'ip': host_info.ip,
            'usage': host_info.usage,
            'appname': host_info.app_name,
        })
    host_dic = {}
    host_dic['data1'] = host_list  # 将整个host_list列表作为一个元素，组装成dic，并返回字典
    return HttpResponse(json.dumps(host_dic,ensure_ascii=False),content_type="application/json,charset=utf-8")
    # return HttpResponse(json.dumps(host_list,ensure_ascii=False),content_type="application/json,charset=utf-8")
    # return HttpResponse(json.dumps(host_dic),content_type="application/json")
```

- html

```javascript
<script type="text/javascript">
$.ajaxSetup({
  data: {csrfmiddlewaretoken: '{{ csrf_token }}' },
});

$(function(){
       var test = $('#test').DataTable({
           dom: 'Bfrtip',
           button: ['copy', 'csv', 'excel', 'pdf', 'print'],
           "ajax": {
               {#"url": "{% url 'account:myhost' %}",#}
               url: '/account/myhost/',
               dataSrc: "data1",//默认为data
               "type": "post",
               "error":function(){alert("服务器未正常响应，请重试");}
           },
           "columns": [
               { "data": "id", "title":"序号","defaultContent":""},
               { "data": "name", "title":"主机名","defaultContent":""},
               { "data": "ip", "title":"IP","defaultContent":""},
               { "data": "usage", "title":"用途","defaultContent":""},
               { "data": "appname", "title":"应用","defaultContent":""},
               { "data": null, "title":"操作","defaultContent": "<button class='edit-btn' type='button'>编辑</button>"}
           ],
           "language": {
               "sProcessing": "处理中...",
               "sLengthMenu": "显示 _MENU_ 项结果",
               "sZeroRecords": "没有匹配结果",
               "sInfo": "显示第 _START_ 至 _END_ 项结果，共 _TOTAL_ 项",
               "sInfoEmpty": "显示第 0 至 0 项结果，共 0 项",
               "sInfoFiltered": "(由 _MAX_ 项结果过滤)",
               "sInfoPostFix": "",
               "sSearch": "搜索:",
               "sUrl": "",
               "sEmptyTable": "表中数据为空",
               "sLoadingRecords": "载入中...",
               "sInfoThousands": ",",
               "oPaginate": {
                   "sFirst": "首页",
                   "sPrevious": "上页",
                   "sNext": "下页",
                   "sLast": "末页"
               },
               "oAria": {
                   "sSortAscending": ": 以升序排列此列",
                   "sSortDescending": ": 以降序排列此列"
               }
           }
       });
       $("#test tbody").on("click",".edit-btn",function(){
           var tds=$(this).parents("tr").children();
           $.each(tds, function(i,val){
               var jqob=$(val);
               if(i < 1 || jqob.has('button').length ){return true;}//跳过第1项 序号,按钮
               var txt=jqob.text();
               var put=$("<input type='text'>");
               put.val(txt);
               jqob.html(put);
           });
           $(this).html("保存");
           $(this).toggleClass("edit-btn");
           $(this).toggleClass("save-btn");
       });

       $("#test tbody").on("click",".save-btn",function(){
           var row=test.row($(this).parents("tr"));
           var tds=$(this).parents("tr").children();
           $.each(tds, function(i,val){
               var jqob=$(val);
               //把input变为字符串
               if(!jqob.has('button').length){
                   var txt=jqob.children("input").val();
                   jqob.html(txt);
                   test.cell(jqob).data(txt);//修改DataTables对象的数据
               }
           });
           var data=row.data();
           $.ajax({
              {#url: "{% url 'account:myhost_edit' %}",#}
              url: '/account/myhost-edit/',
               "data":data,
               "type":"post",
               "error":function(){
                   alert("服务器未正常响应，请重试");
               },
               "success":function(e){
                   {#alert("保存成功");#}
                   {#window.confirm("sometext");#}
                   {#window.prompt("sometext","defaultvalue");#}
                   if(e == "1") {
                       parent.location.reload();
                   }else {
                       alert("sorry, you are not lucky. the picture can't been uploaded.")
                   }
               }
           });
           $(this).html("编辑");
           $(this).toggleClass("edit-btn");
           $(this).toggleClass("save-btn");
       });

       //批量点击编辑按钮
       $("#batch-edit-btn").click(function(){
           $(".edit-btn").click();
       });
       $("#batch-save-btn").click(function(){
           $(".save-btn").click();
       });
   });
</script>
```

- url

```python
url(r'^myhost/$', views.myhost, name="myhost"),  #test!!!  先调用myhost视图函数，
url(r'^test/$', views.test, name="test"),  #test!!!  先请求test页面，再从该页面发起ajax请求，由myhost视图函数返回json数据，渲染表格
```



#### 20200824

##### format 格式化函数

Python2.6 开始，新增了一种格式化字符串的函数 **str.format()**，它增强了字符串格式化的功能。

基本语法是通过 **{}** 和 **:** 来代替以前的 **%** 。x

- format 函数可以接受不限个参数，位置可以不按顺序。

```python
>>>"{} {}".format("hello", "world")    # 不设置指定位置，按默认顺序
'hello world'
 
>>> "{0} {1}".format("hello", "world")  # 设置指定位置
'hello world'
 
>>> "{1} {0} {1}".format("hello", "world")  # 设置指定位置
'world hello world'
```

- 也可以设置参数：

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
print("网站名：{name}, 地址 {url}".format(name="菜鸟教程", url="www.runoob.com"))
 
# 通过字典设置参数
site = {"name": "菜鸟教程", "url": "www.runoob.com"}
print("网站名：{name}, 地址 {url}".format(**site))
 
# 通过列表索引设置参数
my_list = ['菜鸟教程', 'www.runoob.com']
print("网站名：{0[0]}, 地址 {0[1]}".format(my_list))  # "0" 是必须的

网站名：菜鸟教程, 地址 www.runoob.com
网站名：菜鸟教程, 地址 www.runoob.com
网站名：菜鸟教程, 地址 www.runoob.com
```

- 也可以向 **str.format()** 传入对象:

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
class AssignValue(object):
    def __init__(self, value):
        self.value = value
my_value = AssignValue(6)
print('value 为: {0.value}'.format(my_value))  # "0" 是可选的

value 为: 6
```

- 查询对象所有属性

```python
article = ArticlePost.objects.filter(id = 192)
article
<QuerySet [<ArticlePost: 霍格沃茨特快列车>]>   #文章对象
article.values()
<QuerySet [{'body': u'# \u970d     #字典所有键值
```

##### 文章统计

- 最多阅读

```python
total_views = r.incr("article:{}:views".format(article.id)) #阅读数 
r.zincrby('article_ranking', 1, article.id) #分值
article_ranking = r.zrange('article_ranking', 0, -1, desc=True)[:10] #排序切片 
article_ranking_ids = [int(id) for id in article_ranking] #id列表
most_viewed = list(ArticlePost.objects.filter(id__in=article_ranking_ids)) #文章列表 
most_viewed.sort(key=lambda x: article_ranking_ids.index(x.id)) #排序
```

- 最多评论

```python
aid = Comment.objects.all().values('article_id').annotate(count=Count('article')).values('article_id').order_by('-count')[:10]
#print str(Comment.objects.all().values('article_id').annotate(count=Count('article')).values('article_id').order_by('-count')[:10].query)
#SELECT `article_comment`.`article_id` FROM `article_comment` GROUP BY `article_comment`.`article_id` ORDER BY COUNT(`article_comment`.`article_id`) DESC LIMIT 10

a_list = [y for x in aid for y in x.values()]
#x迭代aid对象{'article_id': 188}；y迭代x的values；得到id值列表

most_commed = list(ArticlePost.objects.filter(id__in=a_list)) 
most_commed.sort(key=lambda x: a_list.index(x.id)) 
```

- 最新发表

```python
article_top10 = ArticlePost.objects.all()[:10]
```

##### 列表推导式

```python
z = [[1,2,3], [4,5,6], [7,8,9]]
[y for x in z for y in x]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
#等价
def f(z):
    for y in z:
        for x in y:
           yield x
英文描述：[item for sublist in list for item in sublist]
也就是：子列表中项目的子列表项目

#获取评论最多文章的id列表
aid = Comment.objects.all().values('article_id').annotate(count=Count('article')).values('article_id').order_by('-count')[:10]

aid
<QuerySet [{'article_id': 188}, {'article_id': 192}, {'article_id': 198}, {'article_id': 199}, {'article_id': 185}, {'article_id': 189}, {'article_id': 191}, {'article_id': 194}, {'article_id': 187}, {'article_id': 193}]>

#id列表
[y for x in aid for y in x.values()]
[188, 192, 198, 199, 185, 189, 191, 194, 187, 193]
```

#### 20200827

##### get和filter

```python
#get是用来获取一个对象的，如果需要获取满足条件的对象，就要用到filter
#filter返回对象列表（queryset），只能使用列表对象的属性和方法，除非获取列表元素
user = User.objects.filter(username='lrl')
user
<QuerySet [<User: lrl>]>
type(user)
<class 'django.db.models.query.QuerySet'>

user.article.count()
Traceback (most recent call last):
  File "<input>", line 1, in <module>
AttributeError: 'QuerySet' object has no attribute 'article'

user[0].article.count()
12


#get返回对象，可以直接使用对象的属性和方法。
user = User.objects.get(username='lrl')
user
<User: lrl>
type(user)
<class 'django.contrib.auth.models.User'>

user.article.count()
12
```

##### annotate

```python
#外键,一对多关系，一user对应多article
class ArticlePost(models.Model):
    author = models.ForeignKey(User, related_name="article")
#统计用户文章数    
>>User.objects.annotate(total_article=Count('article'))
#发表文章数最多的前三名用户
>>User.objects.annotate(total_article=Count('article')).order_by('-total_article')[:3]
<QuerySet [<User: bob>, <User: lrl>, <User: mike>]>
bob.total_article = 14



#获取相似文章
article_tag_ids = article.article_tag.values_list("id", flat=True)
similar_articles = ArticlePost.objects.filter(article_tag__in=article_tag_ids).exclude(id=article.id)
#具体相似文章
similar_articles
<QuerySet [<ArticlePost: 3浣溪沙>, <ArticlePost: 2念奴娇·赤壁怀古>, <ArticlePost: 2念奴娇·赤壁怀古>, <ArticlePost: 1江城子·乙卯正月二十日夜记梦>, <ArticlePost: 1江城子·乙卯正月二十日夜记梦>, <ArticlePost: 1江城子·乙卯正月二十日夜记梦>]>
#相似文章及标签
similar_articles.values_list("id","article_tag")
<QuerySet [(240, 19), (239, 19), (239, 20), (238, 19), (238, 20), (238, 21)]>
#
similar_articles2 = similar_articles.annotate(same_tags=Count("article_tag")).order_by('-same_tags', '-created')[:3]
similar_articles2
<QuerySet [<ArticlePost: 1江城子·乙卯正月二十日夜记梦>, <ArticlePost: 2念奴娇·赤壁怀古>, <ArticlePost: 3浣溪沙>]>

similar_articles2[0]
<ArticlePost: 1江城子·乙卯正月二十日夜记梦>
similar_articles2[0].same_tags
3
```



#### 20200831

##### jquery的each方法

```javascript
//each语法
$.each(Object, function(name, value) {
      this;      //this指向当前属性的值
      name;      //name表示Object当前属性的名称
      value;     //value表示Object当前属性的值
 });

//行内编辑 
$("#test tbody").on("click",".edit-btn",function(){
		var tds=$(this).parents("tr").children();
  	$.each(tds, function(i,val){
  	var jqob=$(val);
  	if(i < 1 || jqob.has('button').length ){return true;}//跳过第1项 序号,按钮
  	var txt=jqob.text();
  	var put=$("<input type='text'>");
  	put.val(txt);
  	jqob.html(put);
  	});
  	$(this).html("保存");
  	$(this).toggleClass("edit-btn");
  	$(this).toggleClass("save-btn");
});

//这个each就有更厉害了，能循环每一个属性，输出结果为：1   2  3  4
var obj = { one:1, two:2, three:3, four:4};
$.each(obj, function(key, val) {
alert(obj[key]);
});
```

#### 20200901

##### layer.confirm用法

```javascript
//普通
if (confirm("确定要删除该属性？")) { }

//删除
function deleteUser(id){
    var url = "${ctx }/manage/member/delete.do?museId="+id;
    layer.confirm('您确定要删除该用户吗?',{btn: ['确定', '取消'],title:"提示"}, function(){
        $.ajax({
            type: "post",
            url: url,
            data: null,
            dataType: "json",
            async:false,
            success:function(data) {
                if(data.flag == 1){
                    layer.msg('删除成功', {icon: 1});
                    window.setTimeout("javascript:location.href='${ctx }/manage/member/toPage.do'", 2000); 
                }else{
                    layer.msg('删除失败', {icon: 2});
                }
            }
        });
    });
```

#### 20200902

##### 安装 Homebrew

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"

#运行下面自动脚本（已经全部替换为国内地址）：
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
                初步介绍几个brew命令

        本地软件库列表：brew ls
        查找软件：brew search google（其中google替换为要查找的软件关键字）
        查看brew版本：brew -v  更新brew版本：brew update

现在可以输入命令open ~/.zshrc -e 或者 open ~/.bash_profile -e 整理一下重复的语句(运行 echo $SHELL 可以查看应该打开那一个文件修改)

        https://zhuanlan.zhihu.com/p/111014448  欢迎来给点个赞


查看了下homebrew的脚本，https://raw.githubusercontent.com/Homebrew/install/master/install，发现只要把/usr/local/.git 目录删了即可，就是执行rm -rf /usr/local/.git
```

##### 安装node.js

```javascript
//Mac OS 上安装
//你可以通过以下两种方式在 Mac OS 上来安装 node：
//1、在官方下载网站下载 pkg 安装包，直接点击安装即可。
//2、使用 brew 命令来安装：
brew install node

//在你项目的根目录下创建一个叫 server.js 的文件，并写入以下代码：
var http = require('http');
http.createServer(function (request, response) {

    // 发送 HTTP 头部 
    // HTTP 状态值: 200 : OK
    // 内容类型: text/plain
    response.writeHead(200, {'Content-Type': 'text/plain'});

    // 发送响应数据 "Hello World"
    response.end('Hello World\n');
}).listen(8888);

// 终端打印如下信息
console.log('Server running at http://127.0.0.1:8888/');

//使用 node 命令执行以上的代码：
node server.js
Server running at http://127.0.0.1:8888/
```



#### 20200904

##### annotate（多对多，相似文章），相当于groupby（分组）

```python
#获取文章
article = ArticlePost.objects.get(id=235)
article
<ArticlePost: 0临江仙·夜饮东坡醒复醉>

#文章标签列表
article_tag_ids = article.article_tag.values_list("id", flat=True)
<QuerySet [19, 20, 21]>

#标签相似文章
similar_articles = ArticlePost.objects.filter(article_tag__in=article_tag_ids).exclude(id=235)

similar_articles
<QuerySet [<ArticlePost: 3浣溪沙>, <ArticlePost: 2念奴娇·赤壁怀古>, <ArticlePost: 2念奴娇·赤壁怀古>, <ArticlePost: 1江城子·乙卯正月二十日夜记梦>, <ArticlePost: 1江城子·乙卯正月二十日夜记梦>, <ArticlePost: 1江城子·乙卯正月二十日夜记梦>]>

#标签相似文章及对应标签
similar_articles.values_list("id","article_tag")
<QuerySet [(240, 19), (239, 19), (239, 20), (238, 19), (238, 20), (238, 21)]>

#按文章分组，统计每篇文章的标签数量，相当于238有3个标签（19,20,21)，239有2个标签（19，20），240有1个标签（19）
similar_articles.annotate(same_tags=Count("article_tag")).order_by('-same_tags', '-created')[:3]
<QuerySet [<ArticlePost: 3浣溪沙>, <ArticlePost: 2念奴娇·赤壁怀古>, <ArticlePost: 1江城子·乙卯正月二十日夜记梦>]>
```

作者：andy sin
链接：https://www.zhihu.com/question/26235428/answer/79910310
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



已经一年多了，其实django就只是一个框架，关键是学会web的技巧，那么用其他框架也是一样，现在我也还在用django，近期在做运维后台。多了解前端对你帮助很大的。想ajax、jquery基本是要掌握的。有时间可以看看vue + django restful api 做前后端分离。要做实时的推荐下django-channels websocket库，之前用它做了个实时日志查看，类似tail -F 这样的效果。

这里说下收获：

1. django-channels websocket库
2. pycharm，告别sublime开发django吧，没有一个IDE做debug，你会很痛苦的
3. class view要多看源码，看看ListView、DetailView、UpdateView、Minix等是如何组织的
4. 掌握ajax、jquery, 做web很大部分也是在做交互，交互的话这两个很通用了。
5. vue、django-restful-api 这些前后端分离的方案也要了解
6. 多看看decorator装饰器、templatetags，不要太依赖view，view应该尽量简洁。多借用二者，减少view的臃肿
7. 多看看document，了解form的clean、clean_fieldname等执行顺序
8. 学十个课程，不如动手做一个项目。博客、管理后台、小工具、api接口等等。
9. 思路很重要，web思维对了，剩下的就是组装。（例如我要做运维后台，我需要数据、展示。数据我用zabbix api提供的；模板我使用adminlte；展示用echarts；app我至少会建立monitor、machine、users；涉及远程控制机器的使用saltstack，这里只是初步构想，后面还可以有很多主意）
10. 推荐2个项目[awesome-django](https://link.zhihu.com/?target=https%3A//github.com/rosarior/awesome-django), [awesome-python](https://link.zhihu.com/?target=https%3A//github.com/vinta/awesome-python) ，相当于django和python 库分类，速查。逛下会有收获

比较乱的写了一些，希望帮到一心想学django但还没上车的同学，就如当年迷茫我。



#### 20200906

##### 使用消息框架

导入消息模块并且在视图中使用很简单的语句就可以发送消息，例如：

```python
from django.contrib import messages
messages.error(request, 'Something went wrong')
```

在我们的站点中增加消息内容。由于消息是贯穿整个网站的，所以打算将消息显示的部分设置在母版中，编辑`base.html`，在ID为`header`的`<div>`标签和ID为`content`的`<div>`标签之间增加下列代码：

```html
{% if messages %}
    <ul class="messages">
        {% for message in messages %}
            <li class="{{ message.tags }}">{{ message|safe }}<a href="#" class="close">X</a></li>
        {% endfor %}
    </ul>
{% endif %}
```

在模板中使用了`messages`变量，在后文可以看到视图并未向模板传入该变量。这是因为在`settings.py`中的`TEMPLATES`设置中，`context_processors`的设置中包含`django.contrib.messages.context_processors.messages`这个上下文管理器，从而为模板传入了`messages`变量，而无需经过视图。默认情况下可以看到还有`debug`，`request`和`auth`三个上下文处理器。其中后两个就是我们在模板中可以直接使用`request.user`而无需传入该变量，也无需为`request`对象添加`user`属性的原因。

之后来修改`account`应用的`views.py`文件，导入`messages`，然后编辑`edit`视图：

```python
from django.contrib import messages

@login_required
def edit(request):
    if request.method == "POST":
        user_form = UserEditForm(instance=request.user, data=request.POST)
        profile_form = ProfileEditForm(instance=request.user.profile, data=request.POST, files=request.FILES)
        if user_form.is_valid() and profile_form.is_valid():
            user_form.save()
            profile_form.save()
            messages.success(request, 'Profile updated successfully')
        else:
            messages.error(request, "Error updating your profile")
    else:
        user_form = UserEditForm(instance=request.user)
        profile_form = ProfileEditForm(instance=request.user.profile)

    return render(request, 'account/edit.html', {'user_form': user_form, 'profile_form': profile_form})
```

#### 20200908

##### split()方法和rsplit()方法

```python
#rsplit()--从字符串最后面开始分割
S = "this is string example....wow!!!"
print (S.rsplit( ))
print (S.rsplit('i',1))
print (S.rsplit('w'))

['this', 'is', 'string', 'example....wow!!!']
['this is str', 'ng example....wow!!!']
['this is string example....', 'o', '!!!']

#split() -- 从字符串最前面开始分割
print (S.split('i',1))
['th', 's string example....wow!!!']
print (S.rsplit('i',1))
['this is str', 'ng example....wow!!!']
```



##### python爬虫——urllib2库的安装及使用

https://blog.csdn.net/qq_37616069/article/details/80376576

**urllib2库的基本使用**

所谓网页抓取，就是把URL地址中指定的网络资源从网络流中读取出来，保存到本地。 在Python中有很多库可以用来抓取网页，我们先学习`urllib2`。

> urllib2 是 Python2.7 自带的模块(不需要下载，导入即可使用)
>
> urllib2 官方文档：https://docs.python.org/2/library/urllib2.html
>
> urllib2 源码：https://hg.python.org/cpython/file/2.7/Lib/urllib2.py

**`urllib2` 在 python3.x 中被改为`urllib.request`**

**urlopen**

我们先来段代码：

```python
# urllib2_urlopen.py
 
# 导入urllib2 库
import urllib2
 
# 向指定的url发送请求，并返回服务器响应的类文件对象
response = urllib2.urlopen("http://www.baidu.com")
 
# 类文件对象支持 文件对象的操作方法，如read()方法读取文件全部内容，返回字符串
html = response.read()
 
# 打印字符串
print html
```

#### 20200910

##### ImageField模型

```python
from images.models import Image

#Image对象
image = Image.objects.all()
image
<QuerySet [<Image: 花木兰1>, <Image: 花木兰2>, <Image: 速度与激情5>, <Image: 花木兰3>, <Image: 花木兰4>, <Image: 三体>]>

#ImageField对象
hml4 = image[4]
hml4
<Image: 花木兰4>
hml4.user
<User: lrl>
hml4.image
<ImageFieldFile: images/2020/09/08/Hua-Mu-Lan-3.jpg>
  
#ImageField对象属性  
hml4.image.name
u'images/2020/09/08/Hua-Mu-Lan-3.jpg'
hml4.image.url
'/media/images/2020/09/08/Hua-Mu-Lan-3.jpg'
hml4.image.path
u'/Users/liuchuanshi/PycharmProjects/mysite/media/images/2020/09/08/Hua-Mu-Lan-3.jpg'

#ImageField对象作为File对象的rename操作
import os
from django.conf import settings

old_path = hml4.image.path
old_path
u'/Users/liuchuanshi/PycharmProjects/mysite/media/images/2020/09/08/Hua-Mu-Lan-3.jpg'
hml4.image.name = "images/2020/09/08/Hua-Mu-Lan-3-new.jpg"

new_path = settings.MEDIA_ROOT + hml4.image.name
new_path
'/Users/liuchuanshi/PycharmProjects/mysite/media/images/2020/09/08/Hua-Mu-Lan-3-new.jpg'

settings.MEDIA_ROOT
'/Users/liuchuanshi/PycharmProjects/mysite/media/'
os.rename(old_path, new_path)
hml4.save()
```

#### 20200914

##### 基于类的视图

- 最简类视图

**views.py**

```python
from django.views.generic import TemplateView

class AboutView(TemplateView):
    template_name = "course/about.html"
```

**urls.py**

```python
#from django.views.generic import TemplateView
urlpatterns = [
    # url(r'^about/$', TemplateView.as_view(template_name="course/about.html")),//这种写法不需要views函数
    url(r'^about/$', AboutView.as_view(), name="about"),
```

- ListView

**views.py**

```python
from django.views.generic import ListView

class CourseListView(ListView):
    model = Course  # Course.objects.all()
    # queryset = Course.objects.filter(user=User.objects.filter(username="lrl"))
    context_object_name = "courses"
    template_name = 'course/course_list.html'

#重写get_queryset()方法
    # def get_queryset(self):
    #     qs = super(CourseListView, self).get_queryset()
    #     return qs.filter(user = User.objects.filter(username="lrl"))
```

##### Mixin混合类

**views.py**

```python
class UserMixin(object): #1-根据用户筛选
    def get_queryset(self):
        qs = super(UserMixin, self).get_queryset()
        return qs.filter(user=self.request.user)

class UserCourseMixin(UserMixin, RecentLoginRequiredMixin): #2-课程模型及登录验证
    max_last_login_delta = 600  # Require a login within the last 10 minutes
    model = Course
    login_url = "/account/login/"

class ManageCourseListView(UserCourseMixin, ListView): #3-读取课程
    context_object_name = "courses"
    template_name = 'course/manage/manage_course_list.html'
```

**CreateView**

```python
class CreateCourseView(UserCourseMixin, CreateView): #4-创建课程;继承CreateView，不用写get方法
    fields = ['title', 'overview']
    template_name = 'course/manage/create_course.html'

    def post(self, request, *args, **kwargs): #重写post方法
        form = CreateCourseForm(data=request.POST)
        if form.is_valid():
            new_course = form.save(commit=False)
            new_course.user = self.request.user
            new_course.save()
            return redirect("course:manage_course")
        return self.render_to_response({"form": form})
```

#### 20200916

##### func(*args, **kwargs)

在python的函数中经常能看到输入的参数前面有一个或者两个星号：例如

```python
def foo(param1, *param2):
def bar(param1, **param2):
```

这两种用法其实都是用来将**任意个数**的参数导入到python函数中。

单星号（*）：*agrs
将所以参数以**元组(tuple)**的形式导入：
例如：

```python
>>> def foo(param1, *param2):
        print param1
        print param2
>>> foo(1,2,3,4,5)
1
(2, 3, 4, 5)
```

双星号（**）：**kwargs
将参数以**字典**的形式导入

```python
>>> def bar(param1, **param2):
        print param1
        print param2
>>> bar(1,a=2,b=3)
1
{'a': 2, 'b': 3}
```

此外，单星号的另一个用法是解压参数列表：

```python
>>> def foo(bar, lee):
        print bar, lee
>>> l = [1, 2]
>>> foo(*l)
1 2
```

当然这两个用法可以同时出现在一个函数中：例如

```python
>>> def foo(a, b=10, *args, **kwargs):
        print a
        print b
        print args
        print kwargs
>>> foo(1, 2, 3, 4, e=5, f=6, g=7)
1
2
3 4
{'e': 5, 'g': 7, 'f': 6}
```

参考资料： http://stackoverflow.com/questions/36901/what-does-double-star-and-star-do-for-python-parameters



##### [Macbook升级10.13后没有telnet的解决方法（High Sierra）](https://www.cnblogs.com/ccdc/p/12542735.html)

Macbook的系统到10.13，使用telnet或ftp命令时会提示command not found，这是因为苹果去掉了FTP和Telnet这些工具，究其原因还是因为这两个协议并不安全。

安装步骤：

1、安装brew：

```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"  

#中途会提示回车继续
#Press RETURN to continue or any other key to abort
```

2、安装telnet：

```bash
brew install telnet
```

3、如果要安装FTP，执行下面命令：

```bash
brew install inetutils
brew link --overwrite inetutils
```



#### 20200917

##### python插库

```python
# -*- coding:utf-8 -*-
import time
from pymysql import *

# 装饰器，计算插入50000条数据需要的时间
def timer(func):
    def decor(*args):
        start_time = time.time()
        func(*args)
        end_time = time.time()
        d_time = end_time - start_time
        print("the running time is : ", d_time)
    return decor

@timer
def add_test_users():
    conn = connect(host='localhost', port=3306, user='root', password='rootroot', database='blog', charset='utf8')
    cs = conn.cursor()  #获取游标
    i = 942
    for num in range(0, 10):
        try:
            i = i + 1
            # sql = "INSERT INTO `blog`.`account_myhost`(`hostname`, `ip`, `usage`, `app_name`) VALUES ('1', '1', '1', '1')"
            sql = "INSERT INTO account_myhost(`id`, `hostname`, `ip`, `usage`, `app_name`) VALUES(%s, %s, %s, %s, %s)" %(i,'1','1','1','1')
            cs.execute(sql)
        except Exception as e:
            return
    conn.commit()  #提交
    cs.close()
    conn.close()
    print('OK')

add_test_users()
```



##### memcached

```python
#安装
brew install memcached
memcached -l 127.0.0.1:11211
pip install python-memcached==1.59
pip install django-memcache-status==1.3

#命令
telnet 127.0.0.1 11211
stats items
stats cachedump 4 1
flush_all

#settings.py
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': '127.0.0.1:11211',
}
}
INSTALLED_APPS = [
    # ...
    'memcache_status',
]

#视图增加缓存
subjects = cache.get('all_subjects')
if not subjects:
    subjects = Subject.objects.annotate(total_courses=Count('courses'))
    cache.set('all_subjects', subjects)
    
#缓存视图输出结果
from django.views.decorators.cache import cache_page
url(r'^host-list/$', cache_page(60 * 15)(MyhostListView.as_view()), name="host_list"),
```



#### 20200922

##### AJAX

**GET 请求**

```html
<!-- ajax get -->
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<script>
function loadXMLDoc1()
{
	var xmlhttp;
	xmlhttp=new XMLHttpRequest();

	xmlhttp.onreadystatechange=function()
	{
		if (xmlhttp.readyState==4 && xmlhttp.status==200)
		{
			document.getElementById("getDiv").innerHTML=xmlhttp.responseText;
		}
	}
	xmlhttp.open("GET","/static/1.txt",true);
	xmlhttp.send();
}
</script>
</head>
<body>

<h2>AJAX-GET</h2>
<button type="button" onclick="loadXMLDoc1()">get请求</button>
<div id="getDiv">get请求</div>

</body>
</html>
```

**POST请求**

```html
<!-- ajax post -->
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<script>
function loadXMLDoc2()
{
  var xmlhttp;
  xmlhttp=new XMLHttpRequest();

  xmlhttp.onreadystatechange=function()
  {
    if (xmlhttp.readyState==4 && xmlhttp.status==200)
    {
      document.getElementById("postDiv").innerHTML=xmlhttp.responseText;
    }
  }
  xmlhttp.open("POST","/static/1.txt",true);
  xmlhttp.send();
}
</script>
</head>
<body>

<h2>AJAX-POST</h2>
<button type="button" onclick="loadXMLDoc2()">post请求</button>
<div id="postDiv">post请求</div>

</body>
</html>
```

**responseXML**

```html
<!-- ajax responseXML -->
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<script>
function loadXMLDoc3()
{
  var xmlhttp;
  xmlhttp=new XMLHttpRequest();
  var txt,x,i;

  xmlhttp.onreadystatechange=function()
  {
    if (xmlhttp.readyState==4 && xmlhttp.status==200)
    {
      xmlDoc=xmlhttp.responseXML;
      txt="";
      x=xmlDoc.getElementsByTagName("ARTIST");
      for (i=0;i<x.length;i++)
      {
        txt=txt + x[i].childNodes[0].nodeValue + "<br>";
      }
      document.getElementById("xmlDiv").innerHTML=txt;
    }
  }

  xmlhttp.open("GET","/static/cd_catalog.xml",true);
  xmlhttp.send();
}
</script>
</head>

<body>

<h2>AJAX-responseXML</h2>
<div id="xmlDiv"></div>
<button type="button" onclick="loadXMLDoc3()">获取我的CD</button>

</body>
</html>
```

**回调函数**

```html
<!-- ajax callback -->
<!DOCTYPE html>
<html>
<head>
<script>
var xmlhttp;

function myFunction()
{
	loadXMLDoc4("/static/1.txt",function()
	{
		if (xmlhttp.readyState==4 && xmlhttp.status==200)
		{
			document.getElementById("callDiv").innerHTML=xmlhttp.responseText;
		}
	});
}

function loadXMLDoc4(url,cfunc)
{
xmlhttp=new XMLHttpRequest();
xmlhttp.onreadystatechange=cfunc;
xmlhttp.open("GET",url,true);
xmlhttp.send();
}

</script>
</head>
<body>

<h2>使用 AJAX-callback</h2>
<button type="button" onclick="myFunction()">回调函数</button>
<div id="callDiv">修改内容</div>
</body>
</html>
```

**ajax-get**

```javascript
function loadXMLDoc1()
{
	xmlhttp=new XMLHttpRequest();
	xmlhttp.onreadystatechange=function(){callback}
	xmlhttp.open("GET","/static/1.txt",true);
	xmlhttp.send();
}
```

**ajax-callback**

用一个callback函数创建一个XMLHttpRequest，并从一个TXT文件中检索数据。

```javascript
function loadXMLDoc4(url,cfunc)
{
xmlhttp=new XMLHttpRequest();
xmlhttp.onreadystatechange=cfunc;
xmlhttp.open("GET",url,true);
xmlhttp.send();
}

function myFunction()
{ loadXMLDoc4("/try/ajax/ajax_info.txt", function(){callback}); }
```

**jquery-get**

```html
<!-- jquery get -->
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
<script src="https://cdn.staticfile.org/jquery/1.10.2/jquery.min.js">
</script>
<script>
$(document).ready(function(){
	$("button").click(function(){
		$.post("/static/1.txt",function(data,status){
			alert("数据: " + data + "\n\n状态: " + status);
		});
	});
});
</script>
</head>
<body>

<button>发送一个 HTTP GET 请求并获取返回结果</button>

</body>
</html>
```

**jquery-post**

```html
<!-- jquery post -->
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
<script src="https://cdn.staticfile.org/jquery/1.10.2/jquery.min.js">
</script>
<script>
$(document).ready(function(){
	$("#b2").click(function(){
		$.post("/static/1.txt",{
			name:"菜鸟教程",
			url:"http://www.runoob.com"
		},
		function(data,status){
			alert("数据: \n" + data + "\n状态: " + status);
		});
	});
});
</script>
</head>
<body>

<button id="b2">发送一个 HTTP POST 请求页面并获取返回内容</button>

</body>
</html>
```

**jQuery ajax() 方法**

```html
<!-- jquery ajax()方法 -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>菜鸟教程(runoob.com)</title>
    <script src="https://cdn.staticfile.org/jquery/1.10.2/jquery.min.js">
    </script>
    <script>
        $(document).ready(function(){
            $("button").click(function(){
                $.ajax({url:"/static/1.txt",success:function(result){
                        $("#div1").html(result);
                    }});
            });
        });
    </script>
</head>
<body>

<h2>使用 jQuery AJAX 修改文本内容</h2>
<button>获取其他内容</button>
<div id="div1"></div>

</body>
</html>
```

#### 20200928

##### **mac下python3.8连接mysql驱动**

```shell
macOS (Homebrew)
Install MySQL and mysqlclient:

# Assume you are activating Python 3 venv
$ brew install mysql
$ pip install mysqlclient
If you don't want to install MySQL server, you can use mysql-client instead:

# Assume you are activating Python 3 venv
$ brew install mysql-client
$ echo 'export PATH="/usr/local/opt/mysql-client/bin:$PATH"' >> ~/.bash_profile
$ export PATH="/usr/local/opt/mysql-client/bin:$PATH"
$ pip install mysqlclient
最后pycharm还需要安装mysqlclient
```



##### **python连接mysql的驱动**

对于py2.7的朋友，直接可以用[MySQLdb](https://pypi.python.org/pypi/MySQL-python/1.2.5)去连接，但是[MySQLdb](https://pypi.python.org/pypi/MySQL-python/1.2.5)不支持python3.x。这是需要注意的~

 

那应该用什么python连接mysql的驱动呢，在[stackoverflow](http://stackoverflow.com/questions/23376103/python-3-4-0-with-mysql-database)上有人解答：

（1）可以尝试使用PyMySQL，但它很慢，最新的版本可以支持python 3.4. 地址：http://www.pymysql.org/

（2）还可以尝试使用[mysql-connector-python](http://dev.mysql.com/downloads/connector/python/).地址：http://dev.mysql.com/downloads/connector/python/

（3）还可以尝试使用[mysqlclient](https://pypi.python.org/pypi/mysqlclient) ，地址：https://pypi.python.org/pypi/mysqlclient

当然上述的驱动都是不同人士提供，具体的好处需要自己慢慢摸索，自己反复安装都不成功也是有可能的，这就需要不断的磨砺了。



##### **mac下安装Mysql**

```undefined
brew install mysql
```

设置MySql开机启动

```ruby
$ mkdir -p ~/Library/LaunchAgents   # 首先确认该目录是否存在，若存在则不用执行本命令
$ ln -sfv /usr/local/opt/mysql/*.plist ~/Library/LaunchAgents  
$ launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
```

启动Mysql

```bash
brew list mysql   #查看已经安装的mysql路径
cd /usr/local/Cellar/mysql/${mysql.version}/bin
mysql.server start
```

停止Mysql（设置了开机启动）

```ruby
launchctl unload -w ~/Users/jacky/Library/LaunchAgents/homebrew.mxcl.mysql.plist  
sudo pkill mysqld
```

停止Mysql（未设置开机启动）

```bash
cd /usr/local/Cellar/mysql/${mysql.version}/bin
mysql.server stop
```

使用HomeBrew卸载Mysql

```jsx
brew remove mysql
brew cleanup
launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
rm -Rf ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
```





##### Django删除超级用户

```python
$ from django.contrib.auth.models import User     #引入管理员存放的数据库 
$ user = User.objects.filter()      # 查找该数据库里有哪些用户 
$ user    # 列出用户名
<QuerySet [<User: name>]>
$ User.objects.get(username="name", is_superuser=True).delete()
```



#### Django管理页面保存中文内容出现 ‘ascii’ codec can’t encode characters in position 0-4: 错误解决办法

解决办法：在admin.y添加下面三行代码即可

```python
import sys;

reload(sys);
sys.setdefaultencoding("utf8")
```



#### Django 实现外键去除自动添加的后缀‘_id’

django在使用外键ForeignKey的时候，会自动给当前字段后面添加一个后缀_id。

正常来说这样并不会影响使用。除非你要写原生sql，还有就是这个表是已经存在的，你只是把数据库中的表映射回models。实际上django提供了这样的一个关键字db_colnum来解决这个问题，你只需要：

```python
f = models.ForeignKey(AnotherModel, db_column='f')
```

这样就不会自动添加_id这个后缀了。

**文档中是这么解释的：**

The name of the database column to use for this field. If this isn't given, Django will use the field's name.
If your database column name is an SQL reserved word, or contains characters that aren't allowed in Python variable names – notably, the hyphen – that's OK. Django quotes column and table names behind the scenes.

https://docs.djangoproject.com/en/dev/ref/models/fields/#db-column



#### [《跟老齐学Python Django实战》读后感](https://www.cnblogs.com/kaid/p/8921922.html)

1.说一下这本书，讲解的很细致，内容选取足够入门Django。

2.在学习这本书要注意的几点：

<1>如果你想跟着敲这本书的代码必须要安装：Django版本1.10.1（当然也可以玩玩新版本Django2，中间有些部分需要自己goole来解决）和以下内容：

sudo pip3 install django==1.10.1

sudo pip3 install pytz

sudo pip3 install django-password-reset

sudo pip3 install redis

sudo pip3 install Markdown

sudo pip3 install Pillow

sudo pip3 install sorl-thumbnail

sudo pip3 install django-braces

sudo pip3 install awesome-slugify

<2>这本书源码下载地址：https://github.com/qiwsir/DjangoPracticeProject

再说一下源码：如果按照<1>中安装好，导入的包基本上不会有什么问题，因为这个项目就是针对Django1.10开发的。

说基础错误的地方

\>1 login.html

```python
源码中
<p style="margin-top:10px">Forgot your password? <a href="{% url 'pwd_reset' %}">reset password</a></p>
</div>
改为如下：
<p style="margin-top:10px">Forgot your password? <a href="{% url 'pwd_reset:password_reset_recover' %}">reset password</a></p>
</div>
```

\>2 这个错误是一个数据库问题的错误：在创建多对一的关系的,需要在Foreign的第二参数中加入on_delete=models.CASCADE 这是主外关系键中的**级联删除**，也就是当删除***主表***的数据时候***从表***中的数据也随着一起删除。

account应用中models 

```python
user = models.OneToOneField(User, unique=True, on_delete=models.CASCADE) 
user = models.OneToOneField(User, unique=True, on_delete=models.CASCADE)
```

article应用中的models

```python
user = models.ForeignKey(User, related_name='article_column', on_delete=models.CASCADE)
author = models.ForeignKey(User, related_name="tag", on_delete=models.CASCADE)
```

blog应用中的models

```python
author = models.ForeignKey(User, related_name='blog_posts', on_delete=models.CASCADE)
```

course应用中models

```python
user = models.ForeignKey(User, related_name='lesson_user', on_delete=models.CASCADE)
course = models.ForeignKey(Course, related_name='lesson', on_delete=models.CASCADE)
```

image应用中的models 

```python
user = models.ForeignKey(User, related_name="images", on_delete=models.CASCADE)
```

<3>修改好后运行 python manage.py makemigrations 创建数据表

　　　　　 运行 python manage.py migrate 创建数据库

<4>按照以上修改，源码就可以运行了

Python manage.py runserver

当然，如果你想进入后台管理，就需要自己创建超级管理员

python manage.py createsuperuser



#### save(commit=False)方法

当你通过表单获取你的模型数据，但是需要给模型里null=False字段添加一些非表单的数据，该方法会非常有用。如果你指定commit=False，那么save方法不会理解将表单数据存储到数据库，而是给你返回一个当前对象。这时你可以添加表单以外的额外数据，再一起存储。

```python
def register(request):
    if request.method == "POST":
        user_form = RegistationForm(request.POST)  #user_form含有‘password’和‘password2'字段
        if user_form.is_valid():
            new_user = user_form.save(commit=False)  #不写库，而是生成User对象，该对象没有password2字段
            new_user.set_password(user_form.cleaned_data['password'])
            new_user.save()
            return HttpResponse("successfully")
        else:
            return HttpResponse("sorry, your can not register.")
    else:
        user_form = RegistationForm()
        return render(request, "account/register.html", {"form":user_form})
```



#### Django设置中国时区问题

系统调用为格林威治时间，中国在东八区，需要加八小时得到中国时间。

这时需要在settings.py中进行设置

TIME_ZONE = 'Asia/Shanghai'
但经过测试这时不足以修改前端或数据库存储的时间。查阅文档后发现，还需要修改

USE_TZ = False
此时调用时间即为中国的时间

源码证明

```python
def now():
    """
    Return an aware or naive datetime.datetime, depending on settings.USE_TZ.
    """
    if settings.USE_TZ:
        # timeit shows that datetime.now(tz=utc) is 24% slower
        return datetime.utcnow().replace(tzinfo=utc)
    else:
        return datetime.now()
```


原文链接：https://blog.csdn.net/rudy5348/java/article/details/89847213



##### **Windows环境django项目导入**

pip list

pip 生成和安装requirements.txt

生成文件
pip freeze > requirements.txt

从requirements.txt安装依赖库
pip install -r requirements.txt


redis-cli.exe -h 127.0.0.1 -p 6379




Django连接mysql django.core.exceptions.ImproperlyConfigured: mysqlclient 1.3.13 or newer is required：
https://blog.csdn.net/weixin_43297217/article/details/91128528


明明安装了更高级版本的mysqlclient还是提示错误。
django使用MySQL有两种方法：
方法一：

首先在项目settings.py的文件同目录下的 init.py 文件里输入

import pymysql
pymysql.install_as_MySQLdb()
然后用下载的命令安装安装pymysql驱动：

pip install pymysql
方法二：

如提示 Did you install mysqlclient? 说明缺失这个包(mysql的驱动包)，并且这个包不能通过pip安装，须自己下载，下载地址：

https://www.lfd.uci.edu/~gohlke/pythonlibs/#mysqlclient

下载对应版本软件包

python 3.6 32的下载： mysqlclient?1.3.13?cp36?cp36m?win32.whl
python 3.6 64 的下载：mysqlclient?1.3.13?cp36?cp36m?win_amd64.whl
错误原因：
发现出现错误的原因是在项目settings.py的文件同目录下的 init.py 文件里输入了：
import pymysql
pymysql.install_as_MySQLdb()。
上述两种方法出现了冲突。
解决办法：
pycharm软件下：
1.先去init.py把以下代码删除：import pymysql
pymysql.install_as_MySQLdb()。
2.去设置里面找到项目库，卸载pymysql.
3.项目库里面安装最新版的mysqlclient即可。



Django启动服务器时，报错mysql的2059错误的解决办法。
http://blog.sina.com.cn/s/blog_a60b1c3c0102y9cs.html
环境说明：
Windows 10
python 3.6.5
Django 2.0.5
mysql 8.0.11

当启动django自带的服务器时，报错2059：

> _mysql_exceptions.OperationalError: (2059, )

> django.db.utils.OperationalError: (2059, )

启动方式为如下：

> python manage.py runserver 0.0.0.0:8000

经过一番查询，调试，最终发现了问题所在。主要就是mysql8.0的问题。
目前最新的mysql8.0对用户密码的加密方式为caching_sha2_password, django暂时还不支持这种新增的加密方式。只需要将用户加密方式改为老的加密方式即可。

解决步骤：
1.登录mysql，连接用户为root。

> mysql -u root -p

2.执行命令查看加密方式

> use mysql;
> select user,plugin from user where user='root';

3.执行命令修改加密方式

> alter user 'root'@'localhost' identified with mysql_native_password by 'yourpassword'

4.属性权限使配置生效

> flush privileges



重设mysql8.0的加密方式后，再次启动django服务器就没有任何问题了。