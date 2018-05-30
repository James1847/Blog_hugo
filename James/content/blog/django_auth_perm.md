+++
date = "2018-05-30"
title = "Django的认证和授权机制"
comments = true
slug = 3
+++


>最近的工作中,需要前端用到小程序，由于微信小程序有自己的登录，session维护机制，那么这个时候我们需要后端去了解
django的认证机制。本章主要介绍django的**authentication**和**permission**的用法。

#### Authentication
##### 1. 这个是django最先走的逻辑，django会根据你的**settings**设置里**AUTHENTICATION_BACKENDS**来从上到下调用每个backend的authenticate()方法，这里我们可以通过打断点单步调试的方法去看看django的运行机制。具体代码不在这里贴出。

##### 2. 理解了上一步的话，那么我们就说说我们常用的django第三方包之**django-restframework**和**django-restframework-jwt**,这两个包是我们做基于restful API的重要第三方包，通过这两个包我们很容易就可以写出rest风格的基于**jwt**认证**API**接口，这里我需要说一下，django_restframwork在settings里可以配置自己的参数，其中django_restframwork也有自己的**AUTHENTICATION_BACKENDS**，这个参数的意思是所有走restframework的接口又得根据自己的AUTHENTICATION_BACKENDS来从上至下做认证。上面这句话有点难理解，意思就是:

###### 1. django原生先从settings里找AUTHENTICATION_BACKENDS从上到下依次认证。

###### 2. 如果是rest接口，那么再从restframework的配置里找AUTHENTICATION_BACKENDS，再从这里从上到下依次认证。

##### 3. 每个认证方法其实就是调用的每个AUTHENTICATION_BACKEND的authenticate()方法，每个方式都有自己的authenticate()方法，如果认证成功，authenticate()方法返回认证的user, 否则返回None.

##### 4. 只要在任何一个AUTHENTICATION_BACKEND认证成功了，那么就不会向下继续认证了，认证成功之后，把得到的user赋值给request.user， 这样我们每个request就能拿到登录的user了。如果认证都失败了，返回的是None, 这个时候django会给request.user赋值为anonymous user：匿名用户。

##### 5. 接下来到了每个请求访问API的时候，这个时候就是Permissions机制了，这个机制其实就是从request.user里判断各种逻辑，举个例子，rest_framework.permissions.IsAuthenticated这个permission其实就是查看request.user.is_authenticated这个属性是否为True,如果为Ture,说明可以调用这个API.

#### Permission

##### - 这里需要说明几点permission机制，我们的API可以脱离authentication机制，只不过就是request.user是个匿名用户罢了，如果我们想快速的写一些permission的东西，我们可以继承BasePermission，然后复写has_permission()方法，这样就实现了自己的业务实现。

#### 总结

django的认证权限是分为这两大块的，只有我们深入了解django的原理，我们才能写出优雅的认证机制，官方文档只是教我们写什么参数，但是没有告诉我们为什么这么用，这个只能看源码和博客了。


Author: James

Reference:

- http://www.django-rest-framework.org/api-guide/permissions/#api-reference/

- http://polyglot.ninja/django-rest-framework-authentication-permissions/

