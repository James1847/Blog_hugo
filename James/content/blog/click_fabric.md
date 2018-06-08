+++
date = "2018-06-08"
title = "一键实现django的makemigrations和migrate"
comments = true
slug = 5
+++

>我们使用Django的项目时，免不了使用makemigratios和migrate对数据库的操作命令，虽然这两个命令分开有利于多数据库的
迁移，但是大多时候我们都是在一个数据库使用这俩命令，没有必要先makemigrations之后再migrate，每次输入这些命令的时候
都很繁琐，今天我教大家自己创建一个命令，名字就叫makemigrate（实现了刚才的两步），而且代码十分简单。

话不多说直接上代码：

    import djclick as click
    from django.conf import settings

    from fabric.api import lcd
    from fabric.operations import local


    def abort_if_false(ctx, param, value):
        if not value:
            click.secho('close operation successful with no harm', fg='green')
            ctx.abort()


    @click.command()
    @click.option('--yes', is_flag=True, callback=abort_if_false, expose_value=False, prompt='你确定要继续吗？')
    def command():
        with lcd(settings.BASE_DIR):
            click.secho('makemigrations for your project', fg='red')
            local('env/bin/python src/manage.py makemigrations')
            click.secho('migrate for your project', fg='red')
            local('env/bin/python src/manage.py migrate')
            click.secho('successfully makemigrations and migrate for your project')

这里有两个概念：

一、django-click

1. django-click包是对python click包的封装，这个适用于一般的django项目，我们可以在**/app/management/commands/xx.py**新建你的命令，其中
xx.py中的**xx**就是最后你的命令名字，执行时就是**python manage.py xx**这样执行。

2. @click.command()装饰器是必须的，加入这个后，你的*command()*方法就可以写调用逻辑了。

3. @click.option()装饰器是增加你的命令参数的，加入这个装饰器以后，你可以自定义你的参数和逻辑。具体参考click官方文档。

4. command()方法是你的命令的实际执行逻辑。

二、fabric

1. 我们要在代码里执行命令，最简洁的就是使用fabric包，他可以很方便的在本机或者通过ssh连接的主机上执行命令，很方便。

2. local()方法就是在本机执行命令，**env/bin/python**是激活你的环境变量，这里使用的是**virtualenv**，如果你使用的是**pipenv**的话，那么
你可以使用**pipenv run** 后加指令。


三、我们的逻辑就是click增加django的cli事件，fabric执行逻辑代码。最后执行我们的cli时，只需要**Python manage.py makemigrate**效果如下图：
![Example image](/img/post/WechatIMG3181.jpeg)