### 笔记和摘录



##### **切换到root用户**

使用命令：sudo su -；命令执行后输入密码

或

使用命令：sudo su；su - ；不需要输入密码



##### **zsh及oh-my-zsh安装配置**

查看当前使用的 shell

```bash
echo $SHELL
/bin/bash
```

查看安装的 shell

```undefined
cat /etc/shells

/bin/bash
/bin/csh
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
```

使用 brew 更新 zsh

```php
brew install zsh

==> Downloading https://homebrew.bintray.com/bottles/zsh-5.5.1.high_sierra.bottle.tar.gz
######################################################################## 100.0%
==> Pouring zsh-5.5.1.high_sierra.bottle.tar.gz
/usr/local/Cellar/zsh/5.5.1: 1,444 files, 12MB
```

切换为 zsh

```undefined
chsh -s /bin/zsh
```

安装oh-my-zsh

Github 网址是：
https://github.com/robbyrussell/oh-my-zsh

准备工作: 你的mac上需要装curl 或者 wget, 如果你的电脑上没有安装curl 或者 wget, 请安装后再继续。

你可以通过curl 或者 wget的命令行安装

curl

```
curl -L https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh | sh
```

 wget

```
wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O - | sh
```

准备工作: 你的mac上需要安装git。

1、克隆代码库

```
git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
```

2、备份已有的.zshrc [可选步骤]

```
cp ~/.zshrc ~/.zshrc.orig
```

3、新建一个zsh的配置文件

可以拷贝一份已有的模板文件来创建zsh的配置文件

```
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
```

执行从 oh-my-zsh 的 GitHub 下载的安装脚本

```csharp
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

打开 oh-my-zsh 配置文件

```undefined
vim ~/.zshrc
```

配置项 `ZSH_THEME` 即为 oh-my-zsh 的主题配置，oh-my-zsh 的 GitHub Wiki 页面提供了 [主题列表](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Frobbyrussell%2Foh-my-zsh%2Fwiki%2Fthemes)
当设置为 `ZSH_THEME=random` 时，每次打开终端都会使用一种随机的主题。

更新配置

```bash
source ~/.zshrc
```

自动补全插件（自动补全插件zsh-autosuggestions

1、安装zsh-autosuggestions

```bash
git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```

2、编辑~/.zshrc文件
-------- 打开命令行工具进入~/.zshrc文件

```undefined
  sudo vim ~/.zshrc
```

plugins=(git)这一行添加：plugins=(git zsh-autosuggestions)

```bash
source ~/.zshrc
```

> Homebrew：
> [https://brew.sh/index_zh-cn](https://links.jianshu.com/go?to=https%3A%2F%2Fbrew.sh%2Findex_zh-cn)
> [https://github.com/Homebrew/brew](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FHomebrew%2Fbrew)
> oh-my-zsh：
> [https://ohmyz.sh/](https://links.jianshu.com/go?to=https%3A%2F%2Fohmyz.sh%2F)
> [https://github.com/robbyrussell/oh-my-zsh](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Frobbyrussell%2Foh-my-zsh)
> incr：
> [http://mimosa-pudica.net/zsh-incremental.html](https://links.jianshu.com/go?to=http%3A%2F%2Fmimosa-pudica.net%2Fzsh-incremental.html)

链接：https://www.jianshu.com/p/d372ef933453



##### **解决github图片不显示的问题**

修改C:\Windows\System32\drivers\etc\hosts，在文件末尾添加：

```shell
# GitHub Start 
192.30.253.112    Build software better, together 
192.30.253.119    gist.github.com
151.101.184.133    assets-cdn.github.com
151.101.184.133    raw.githubusercontent.com
151.101.184.133    gist.githubusercontent.com
151.101.184.133    cloud.githubusercontent.com
151.101.184.133    camo.githubusercontent.com
151.101.184.133    avatars0.githubusercontent.com
151.101.184.133    avatars1.githubusercontent.com
151.101.184.133    avatars2.githubusercontent.com
151.101.184.133    avatars3.githubusercontent.com
151.101.184.133    avatars4.githubusercontent.com
151.101.184.133    avatars5.githubusercontent.com
151.101.184.133    avatars6.githubusercontent.com
151.101.184.133    avatars7.githubusercontent.com
151.101.184.133    avatars8.githubusercontent.com

 # GitHub End
```

##### **mac忘记mysql密码**

```shell
#01-在系统偏好设置中停止mysql服务 或者使用指令
sudo /usr/local/mysql/support-files/mysql.server stop

#02-输入指令：
cd /usr/local/mysql/bin 

#03-使用管理员权限登录
sudo su

#输入以下命令来禁止mysql验证功能 　
./mysqld_safe --skip-grant-tables &

#04-输入 ./mysql/ 回车，输入
FLUSH PRIVILEGES;

#05-修改密码
ALTER USER 'root'@'localhost' IDENTIFIED BY '你的新密码';
```

#####  

##### **Django设置中国时区问题**

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

```python
#如果settings.py使用：
TIME_ZONE = 'Asia/Shanghai'
USE_TZ = True

#将会数据库是UTC时间，前端是北京时间(UTC+8),造成问题是，如果以日期为参数查询，同时又正好碰到前后端日期不一致，将会出错，如：
    def get_absolute_url(self):
        return reverse('blog:post_detail', args=[self.publish.year, self.publish.month, self.publish.day, self.slug])
    
#所以如果不跨时区使用，设置如下即可：
TIME_ZONE = 'Asia/Shanghai'
USE_TZ = False
```



