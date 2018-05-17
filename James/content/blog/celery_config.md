+++
date = "2018-05-15"
title = "celery集群部署"
slug = hugo-integrated-gitment-plugin
gitment = true
+++

>最近的工作中，经常用到celery做异步任务，但是把整个业务逻辑写在一个celery worker里是一个很不理想的方案，尤其现在是一个微服务盛行的年代
，所以很多给前端的api最好都原子化，这样方便以后我们组合封装业务逻辑，并且把整个项目的结构梳理的更清晰易懂，我们写的代码都有可能是给别人看的，
所以我们应该尽可能把整个架构做到最优。

##### 一、celery基本概念

celery的概念我就不详细介绍了，这个是在python里做异步最好的了吧（我个人觉得）。celery主要是又三部分组成，**broker**, **worker**, **backend**,
这三个概念我们一定要清楚，尤其是前两个，这是我们做celery集群必须要懂得概念。

1. **broker**是celery的消息队列，这个在celery官方推荐用rabbitmq这个队列做我们的broker，说到rabbitmq，其实是有一段历史的，这个大家可以自行
百度，正是历史原因，才有了rabbitmq这个开源的消息队列（amqp协议），rabbitmq的好处就是队列不会丢我们的celery任务（相较于django orm和redis）。

2. **worker**是celery执行代码的消费者，这个是业务逻辑的执行方，我们写的代码实际上都是worker实例在执行。启动worker实例需要单独的进程，
一般都是使用**celery cli**执行命令使用，比较好的实现是把worker作为守护进程，这里推荐大家使用docker封装，或者使用supervisor来使得celery worker变为守护进程。

3. **backed**就是worker执行完之后的result的存储端，我一般都是在django里使用django-celery-results这个包，可以查看文档把result存入Django orm里，很
方便。

##### 二、celery集群

*celery集群是我主要介绍的，本篇也主要介绍这个内容。*

>为什么要celery集群，其实celery的集群主要就是worker的集群，一般我们的集群方案都是共享一个消息队列（rabbitmq），这个rabbitmq我们一般单独放在
我们的内网服务器上，我们在做测试的时候为了大家环境变量一致，都会用到一个rabbitmq, 数据库, redis，我们公司就是把这三个放在了内网服务器上，配置为
内网可访问，防火墙没有限制内网网卡，这样我们大家只需要在Django settings里配置好统一的env就可以愉快的开始项目了。

celery集群的第一步就是需要在你的celery config里配置好各自的消息队列，这样我们的worker就可以指定队列消费了。配置如下：

    CELERY_QUEUES = (
        Queue('default', Exchange('default'), routing_key='default'),
        Queue('add_notice_async', Exchange('add_notice_async'), routing_key='add_notice_async'),
        Queue('send_tts', Exchange('send_tts'), routing_key='send_tts'),
        Queue('voice_recognize', Exchange('voice_recognize'), routing_key='voice_recognize'),
        Queue('play_voice', Exchange('play_voice'), routing_key='play_voice'),
        Queue('stop_play_voice', Exchange('stop_play_voice'), routing_key='stop_play_voice'),
    )
这样我们就指定了6个消息队列，这里需要注意的是我们最好启用一个default消息队列，这个是针对没有指定队列的worker的队列，所以相应的我会在我的服务上启用5个celery worker实例。

启动celery worker的方法很简单，在我们配置好我们的celery基本配置后，需要在项目里启动如下命令：

    celery worker -A config -l INFO --hostname=add_notice_async --queues=add_notice_async -c 1
    celery worker -A config -l INFO -n send_tts -Q send_tts -c 1
这里需要说明一下参数的意义，**—A --app** 指定配置的app目录，celery会从这个app里读取配置信息，**-l --log info**
配置日志级别，一般设为info, **--hostname** 指定你启动的celery实例的名称，应为是集群，所以你的每一个celery实例都应该有一个特殊的名字方便broker把消息
传递过去， **--queues=** 这个参数很重要，这个是根据你设定的queue，来让worker消费指定的queue, **-c**这个是指定每个worker实例启动多少个并发，我这里设定
为一个，一般是cpu的数量。

如果把以上都配置好了之后，celery集群基本就可以运行了，注意我们要写好日志，异步任务debug不方便，所以我们就需要日志给我们提示了。是不是很方便，celery的异步框架真的很方便，如果你想
进阶的了解celery，我建议大家看看celery源码，方便我们理解。






