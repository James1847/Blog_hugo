+++
date = "2018-10-10"
title = "Django的csrf原理"
comments = true
slug = 6
+++

网上看了很多Django的CSRF防御解析,发现很多都是解答的不详细,网上大多都是说在cookie里传入一个csrftoken进行验证,那么问题来了，如果只在cookie里存入
token,那么实际上不就跟没有防御csrf一样吗，因为csrf攻击就是假装client的cookie里的token来欺骗服务器的,实际上经过我抓包之后，发现，Django会在POST的
表单里还会传入一个csrfmiddlewaretoken,然后一起发送给后端，后端收到cookie里的csrftoken和表单里的csrfmiddlewaretoken，然后经过一个算法比较这两个值
是否相等来判断是否遭到了csrf攻击,注意后端检查是在中间件里面检查的,大家去django的settings文件里看看你的中间件默认会开启csrfmiddleware的,这样做的好处
就是后端不用持久化存储token了,比对只是通过这两个值互相比对,大大减少了后端的压力。
