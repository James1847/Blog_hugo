+++
date = "2018-05-30"
title = "Celery集群问题小记"
comments = true
slug = 4
+++

>最近在工作中遇到过一些问题，这里记录一下

#### celery集群worker怎么消费队列的

这个问题一定要好好说说，我们有这样一个需求，两个进程怎么通信，大多数都是通过消息中间件的方式，比如**rabbitmq**或者**java**的**activemq**,
这些消息中间件有个很好的作用，可以让我们在各个进程间通过消息队列的方式通信。场景是这样的：
A主机需要让B主机做处理，那么他就发送到rabbitmq里面很多message，等待B去开通celery实例来消费这些message，但是真实情况是A确实发送到队列了，
B主机也确实能收到这些消息，但是接受到之后总是报KeyError,大概报错就是这个worker不是你注册的worker,你没有权利来接受参数的意思。
这个时候问题怎么解决，后来问了问大神，给出的答案就在队列里的message上，每个message都是一个协议消息，rabbitmq就是amqp协议，我们可以打开rabbitmq
的监控端口，可以很直观的看到amqp消息中存了消息ID， langeuage, task, eta_time还有路由等等，这几个就是最重要的信息，其他可以略过了。
根据这些重要信息，我们基本能知道worker取信息也是根据这些字段来从队列取走的。工作中出现的问题就是worker取消息的时候看到对应的task字段没有匹配上，
导致出现了KeyError。

#### celery各个worker是怎么通信的

看过一点源码，主要是koumb包的mailbox类，可以实现各个worker广播和单播，源码不在赘述。