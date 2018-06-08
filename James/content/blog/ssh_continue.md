+++
date = "2018-05-08"
title = "SSH终端卡死"
comments = true
slug = 2
+++

>工作中经常用ssh连接到服务器,但是过一段时间ssh终端会自动卡死,以下教程解决ssh终端卡死问题

ssh协议属于应用层，底层是通过tcp协议连接的，我们在客户端通过ssh协议访问服务器，服务器的防火墙会监听所有的tcp连接.

>- 服务器存在防火墙，会关闭超时空闲连接，或设置了关闭超时空闲连接。
>- 客服端和服务器之间存在路由器，路由器也可能带有防火墙，会关闭超时空闲连接。
>- 客服端存在防火墙，会关闭超时空闲连接。

当我们ssh到服务器空闲之后，防火墙会侦测到当前tcp连接是空闲状态，那么防火墙过一段时间会自动把这个tcp连接断开，这样就造成了我们不在终端操作
命令一段时间后，发现终端有卡死的现象，所以解决这个问题的核心就是不要让防火墙把当前tcp连接当做空闲状态给杀掉。
>通过ssh连接后，客户端和服务端长时间没响应时，在两方机器设置中均没任何限制，但在各自的防火墙，或是中转网络连接路由的防火墙中，出现了「闲置超时断开」的缺省机制！

### 解决办法如下：
##### 一、修改服务器端配置
在sshd_config里(我是centos7系统，此文件位于etc/ssh/sshd_config，记得用sudo命令编辑)。编辑

    TCPKeepAlive yes
    ClientAliveInterval 300
    ClientAliveCountMax 3
把这三个选项配置好之后，重启ssh服务，centos7默认是在**systemctl**做统一管理，重启命令如下：

    sudo systemctl restart sshd
    sudo systemctl status sshd
查看sshd的日志：

    sudo journalctl -u sshd -f -e

里面查看是否重启成功，
**TCPKeepAlive** yes保持tcp连接、**ClientAliveInterval** 300 指定服务端向客户端请求消息的时间间隔，单位是秒，默认是0，不发送。设置个300表示5分钟发送一次（注意，这里是服务端主动发起），然后等待客户端响应，成功，则保持连接。
**ClientAliveCountMax** 3 最大客户端重连次数，就是指服务端发出请求后客户端无响应则自动断开的最大次数。使用默认给的3即可。

    注意：TCPKeepAlive必须打开，否则直接影响后面的设置，原理就是ssh底层是tcp协议，防火墙那里肯定对tcp连接做监听了。ClientAliveInterval设置的值要小于各层防火墙的最小值，不然，也就没用了。

##### 二、修改客户端配置
在你的客户端用户目录下(必须是存放你公钥和私钥的用户目录下)

    vi ~/.ssh/config
编辑如下：

    Host *
    ServerAliveInterval 60

*这个修改append到文件首部即可*，表示对你配置的所有ssh连接都表示每60秒去给服务端发起一次请求消息（这个设置好就行了）

##### 三、修改连接工具的配置

    通过改变连接工具的一些默认配置，把keepalive的配置打开起来即可：

- secureCRT：会话选项 - 终端 - 反空闲 - 发送NO-OP每xxx秒，设置一个非0值。
- putty：Connection - Seconds between keepalive(0 to turn off)，设置一个非0值。
- iTerm2：profiles - sessions - When idle - send ASCII code.
- XShell：session properties - connection - Keep Alive - Send keep alive message while this session connected. Interval [xxx] sec.

>当然，用这个办法的副作用也是有的，比如iTerm2会出现一些并不想输入的字符、vim会有些多余字符插入等等，这些情况就按个人的需要酌情取舍了。我个人只是试了试，然后就改回来了。


##### 四、修改连接参数-o

    ssh -o ServerAliveInterval=30 user@host

### 总结：

我个人觉得修改服务端最方便，所有的客户端都不用修改，一劳永逸，但是代价就是不安全，我个人的服务器也没什么东西，做好ssh防爆破应该不会有啥问题，
所以对个人服务器采用第一种方法就行了，但是公司的生产服务器别这么弄，会产生莫名其妙的问题在以后的工作中，所以采用四应该是没啥问题听安全的。
效果如下图：
![Example image](/static/img/post/WechatIMG3181.jpeg)



