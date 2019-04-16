---
title: Django笔记
date: 2019-04-16 14:24:09
tags:
- Python
- Django
categories:
- Python
---

随手记录一些`Django`发现的知识点。

<!-- more -->

## `as_view`

当在`view`中采用`class-based-view`方法时，必须要在`url`中使用`as_view()`。例如：

```python 
# views.py
from django.views.generic import View
from django.http import HttpResponse

class Hello(View):
    def get(self, request):
        return HttpResponse("Hello")
      
# urls.py
from fun.views import Hello

urlpatterns = [
    ...
    path('hello/', Hello.as_view()),
]
```

查看`as_views`源码发现，它是返回一个可以被调用的`view`，这个`view`可以接受`request`，并且返回一个`response`：

```python 
>>> Hello
<class 'fun.views.Hello'>
>>> my_callable_view = Hello.as_view()
>>> my_callable_view
<function Hello at 0x114b9ff28>
>>> request
<WSGIRequest: GET '/hello/'>
>>> response = my_callable_view(request)
>>> response
<HttpResponse status_code=200, "text/html; charset=utf-8">
>>> response.content
b'Hello'
```

##  `dispatch`

提到`as_view`就不得不提到`dispatch`方法，因为即使使用`as_view`，最终调用的仍然是实例的`dispatch`方法。

```python 

def dispatch(self, request, *args, **kwargs):
  # Try to dispatch to the right method; if a method doesn't exist,
  # defer to the error handler. Also defer to the error handler if the
  # request method isn't on the approved list.
  if request.method.lower() in self.http_method_names:
    handler = getattr(self, request.method.lower(), self.http_method_not_allowed)
    else:
      handler = self.http_method_not_allowed
      return handler(request, *args, **kwargs)
```

它会查找实例中是否存在一个同名的`request.method`方法，如果没有找到对应的方法，将会返回一个`HttpResponseNotAllowed`。

如果重载了`dispatch`方法，将会影响到全部的视图类方法函数，所以我通常选择在上面加`method_decorator`装饰器，例如，添加缓存：

```python 
from django.views.generic import View
from django.views.decorators.cache import cache_page
from django.http import HttpResponse
from django.utils.decorators import method_decorator


class Hello(View):
    def get(self, request):
        # 模拟耗时操作
        import time
        time.sleep(1)
        return HttpResponse("Hello")

    # 增加15分钟的缓存
    @method_decorator(cache_page(15*60))
    def dispatch(self, request, *args, **kwargs):
        return super(Hello, self).dispatch(request, *args, **kwargs)
```

这个方法和在`urls.py`添加缓存是一样的。

## `reverse`

因为以前一直使用前后端分离，基本没有遇到过在`view`中需要进行重定向的问题，所以对`reverse`的用法一直略显生疏。它可以防止当一个`url`更名后，全局所有的引用都得跟着改变：

```python 
# urls.py
...
urlpatterns = [
    path('hello/', Hello.as_view(), name='hello'),
]

# 测试结果
>>> from django.urls import reverse
>>> reverse('hello')
'/hello/'
```

## 中间件(`Middlewares`)

它有五个接口：

- `process_request(self, request)`：该方法在请求被决定使用哪个`view`之前调用，会返回`None`或者`HttpResponse`对象。

- `process_view(self , request, callback , callback_args,  callback_kwargs)`：`callback`是请求被决定使用的`view`函数，`callback_args`和`callback_kwargs`是`view`锁需要接受的参数，它返回`None`或者`HttpResponse`。
- `process_template_response(self, request, response):`response`是一个由`Django view`或者中间件返回的`TemplateResponse `对象, `process_template_response()`在`view`使用`render`渲染一个模版对象完成之后被调用，它必须返回一个`render`方法执行后的`response`对象。

- `process_response`(`self`, `request`, `response`)：`response` 是一个`django view`或者中间件返回的`HttpResponse`或者`StreamingHttpResponse`对象，`process_response`会在所有响应到达浏览器之前被调用

- `process_exception`(`self`, `request`, `exception`)：`exception`是`view`函数中`raise`的`Exception`对象，当`view`函数`raise`一个`exception`的时候调用`process_exception`，它会返回`None`或`HttpResponse`对象。

### `process_request`

```python 
# fun/middlewares.py
from django.utils.deprecation import MiddlewareMixin
from django.http import HttpResponse


class TestMiddleware(MiddlewareMixin):
    def process_request(self, request):
        if request.META['HTTP_USER_AGENT'] == 'PostmanRuntime/7.6.1':
            request.META['user_name'] = 'Postman'
        else:
            request.META['user_name'] = 'Chrome'
            
# fun/views.py
class Hello(View):
    def get(self, request):
        user_name = request.META.get('user_name', None)
        return HttpResponse("Hello " + user_name)
```

采用`Postman`和浏览器访问分别能得到不同的结果。

### `process_view`

```python 
# 新增process_view，打印需要调用的view的名称
class TestMiddleware(MiddlewareMixin):
    def process_request(self, request):
        if request.META['HTTP_USER_AGENT'] == 'PostmanRuntime/7.6.1':
            request.META['user_name'] = 'Postman'
        else:
            request.META['user_name'] = 'Chrome'

    def process_view(self, request, callback, callback_args, callback_kwargs):
        print(callback.__name__)
```

输出结果都是：`Hello`

### `process_template_response`

```python 

class TestMiddleware(MiddlewareMixin):
		...

    def process_template_response(self, request, response):
        print(response)
        return response
      
 # views中需要有TemplateResponse
from django.template.response import TemplateResponse

class Hello(View):
    def get(self, request):
        user_name = request.META.get('user_name', None)
        # return HttpResponse("Hello " + user_name
        print("Hello"+user_name)
        return TemplateResponse(request, 'hello.html')
```

`render`会直接将模板渲染，并返回一个`HttpResponse`，所以使用`TemplateResponse`可以拦截一下渲染后的`Response`。

### `process_response`

可以基于此接口，再修改返回的`response`：

```python 
class TestMiddleware(MiddlewareMixin):
		...
    def process_response(self, request, response):
        print(response)
        return HttpResponse("Hi " + response.content.decode('utf-8').split()[-1])
      
      
# views.py
class Hello(View):
    def get(self, request):
        user_name = request.META.get('user_name', None)
        return HttpResponse("Hello " + user_name)
```

输出会从`Hello Postman`变成`Hi Postman`

### `procee_exception`

这个接口可以很方便的拦截异常：

```python 

class TestMiddleware(MiddlewareMixin):
		...
    def process_exception(self, request, exception):
        print(exception)
        return HttpResponse("啊哦，服务器开小差了~")
      
# views.py

class Hello(View):
    def get(self, request):
      	# 出现一个KeyError
        user_name = request.META.get('user_names')
        return HttpResponse("Hello " + user_name)
```

异常会打印在终端，而浏览器访问会返回`Hi 啊哦，服务器开小差了~`，可见它是先处理`process_exception`(在有异常的情况下)，并将其`response`返回给`process_response`处理。

### 执行顺序

执行顺序如下图所示：

![执行顺序](/assets/middlewares.png)

在`1.10`版本后，如果`process_request`没有返回`None`，这个请求将会返回给同一层的中间件。

首先添加一个新的中间件，在`settings`中放在`TestMiddleware`后面：

```python 
# fun/middlewares.py
...
class AnotherTestMiddleware(MiddlewareMixin):
    def process_request(self, request):
        if request.META['HTTP_USER_AGENT'] == 'PostmanRuntime/7.6.1':
            request.META['user_name'] = 'NewPostman'
        else:
            request.META['user_name'] = 'NewChrome'

    def process_view(self, request, callback, callback_args, callback_kwargs):
        print('New', callback.__name__)

    def process_template_response(self, request, response):
        print('New', response)
        return response

    def process_response(self, request, response):
        print('New', response)
        return HttpResponse("Hi " + response.content.decode('utf-8').split()[-1])

    def process_exception(self, request, exception):
        print('New', exception)
        return HttpResponse("啊哦，服务器开小差了~")
```

输出结果：

```shell
Hello
New Hello
New <HttpResponse status_code=200, "text/html; charset=utf-8">
<HttpResponse status_code=200, "text/html; charset=utf-8">
```

如果为原本的`TestMiddleware`的`process_request`添加个返回，那么`AnotherMiddleware`中的函数都不会执行：

```python 
class TestMiddleware(MiddlewareMixin):
    def process_request(self, request):
        if request.META['HTTP_USER_AGENT'] == 'PostmanRuntime/7.6.1':
            request.META['user_name'] = 'Postman'
        else:
            request.META['user_name'] = 'Chrome'
        return HttpResponse("StopTest")
```

输出结果：

```shell
<HttpResponse status_code=200, "text/html; charset=utf-8">
```

而当出现了异常时，是由后面的`AnotherMiddleware`所处理的：

```shell
Quit the server with CONTROL-C.
Hello
New Hello
New can only concatenate str (not "NoneType") to str
New <HttpResponse status_code=200, "text/html; charset=utf-8">
<HttpResponse status_code=200, "text/html; charset=utf-8">
```