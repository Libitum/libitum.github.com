+++
title = 'Django Api 服务之实现默认的 login_required'
date = 2017-06-20T23:23:00+08:00
categories = ['Tech']
tags = ['Python']
draft = false
+++

## Background
我们在使用 Django 做 Api 开发时，基本所有接口都需要认证。这种情况下，如果使用 Django 
原生的 login_required 就会显得非常麻烦，每个接口上都需要添加这个装饰器。

因此，需要能够实现一个默认的 『login_required』，默认将所有接口都加上登录校验，而只将一些特殊的接口例外。

<!--more-->

## Solution
1. 使用 Django Middleware 来实现登录校验

    ``` python
    from django.http.response import HttpResponseForbidden
    from django.conf import settings

    class LoginRequiredMiddleWare(object):
        def __init__(self, get_response):
            self.get_response = get_response

        def __call__(self, request):
            # 判断是否已经登录。这里依赖于 SessionMiddleware 
            # 和 AuthenticationMiddleWare 做自动认证
            if hasattr(request, 'user') and not request.user.is_authenticated:
                ignore = False
                for prefix in settings.AUTH_FREE_URL_PREFIX:
                    if request.path.startswith(prefix):
                        ignore = True
                        break
                if ignore is False:
                    # 如果未登录且没有设置忽略，则直接返回 503
                    return HttpResponseForbidden()

            # 正常处理请求
            response = self.get_response(request)
            return response

        def process_exception(self, request, exception):
            """ automatically handle exception in get response """
            pass

    ```

1. 修改配置文件 settings.py，配置需要忽略的 url
``` python
AUTH_FREE_URL_PREFIX = [
    '/account/user/login',
]
```
从 Middleware 的实现可以看出，装饰器是在 get\_response 里面的，因此无法像 login\_reqired 那样实现
一个 login_free。所以可以选择这种配置文件的形式。


## Summary
1. Dango Middleware 可以实现对 Request 的统一处理
1. Middleware 的 process_exception 方法，可以实现异常的统一处理。这样在 Django App 的逻辑代码中，
    就可以直接 raise exception，避免每个地方都单独处理异常 Response

## Reference
1. [Dango Middleware](https://docs.djangoproject.com/en/1.11/topics/http/middleware/)
