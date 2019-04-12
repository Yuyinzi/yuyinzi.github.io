---
title: Django源码分析
date: 2019-04-11 13:41:00
tags: 
    - Python 
    - Django
categories:
    - Python
---
主要分析了运行`python manage.py runserver`所发生的事。
<!-- more -->
## `WSGI`,`uwsgi`和`uWSGI`
`WSGI`：全称是`Web Server Gateway Interface`，是一种规范，只适用于`Python`语言。要实现`WSGI`协议，必须同时实现`web server`和`web application`，当前运行在`WSGI`协议之上的web框架有`Bottle`, `Flask`, `Django`。
`uwsgi`：与`WSGI`一样是一种通信协议，是`uWSGI`服务器的独占协议，用于定义传输信息的类型(`type of information`)，每一个`uwsgi packet`前`4byte`为传输信息类型的描述，与`WSGI`协议是两种东西，据说该协议是`fcgi`协议的`10`倍快。
`uWSGI`：是一个`web`服务器，实现了`WSGI`协议、`uwsgi`协议、`http`协议等。
## 入口
入口函数在`manage.py`中，从`execute_from_command_line(sys.argv)`开始，这时候会传入`[manage.py文件在的位置，command(runserver)， 端口号]`：
```python
def execute_from_command_line(argv=None):
    """
    A simple method that runs a ManagementUtility.
    """
    # 使用argv进行实例化
    utility = ManagementUtility(argv)
    utility.execute()
```
接下来调用`execute()`方法，根据注释，这个方法根据`subcommand`解析出需要的操作：
```python
def execute(self):
    """
    Given the command-line arguments, this figures out which subcommand is
    being run, creates a parser appropriate to that command, and runs it.
    """
    if settings.configured:
        # Start the auto-reloading dev server even if the code is broken.
        # The hardcoded condition is a code smell but we can't rely on a
        # flag on the command class because we haven't located it yet.
        if subcommand == 'runserver' and '--noreload' not in self.argv:
            # check_errors是一个闭包，中间执行的`django.setup`进行了一系列的导包操作
            # 包括`INSTALLED_APPS`，检查是否有重复的
            # 会有apps_ready，models_ready, ready三个状态
               autoreload.check_errors(django.setup)()

# 略去对help version命令的处理代码
    self.fetch_command(subcommand).run_from_argv(self.argv)
```
关键在于最后一句，首先看`fetch_command`(`subcommand`此处为`runserver`)，用来获取执行命令所需要的类:
```python
    def fetch_command(self, subcommand):
        """
        Tries to fetch the given subcommand, printing a message with the
        appropriate command called from the command line (usually
        "django-admin" or "manage.py") if it can't be found.
        """
        # Get commands outside of try block to prevent swallowing exceptions
        commands = get_commands()
        try:
            app_name = commands[subcommand]
        except KeyError:
            # ...
        if isinstance(app_name, BaseCommand):
            # If the command is already loaded, use it directly.
            klass = app_name
        else:
            klass = load_command_class(app_name, subcommand)
        return klass
```
接下来是`run_from_argv`：
```python
def run_from_argv(self, argv):
    self._called_from_command_line = True
    parser = self.create_parser(argv[0], argv[1])
    options = parser.parse_args(argv[2:])  # 返回一个NameSpace
    cmd_options = vars(options)   # 解为字典
    # Move positional args out of options to mimic legacy optparse
    args = cmd_options.pop('args', ())
    handle_default_options(options)
    try:
        self.execute(*args, **cmd_options)
    except Exception as e:
        # ...
    finally:
       # ...
```
主要看`execute`：
```python
def execute(self, *args, **options):
  # 略去进行一些基本的输出设置
  try:
      if self.requires_system_checks and not options.get('skip_checks'):
          self.check()
      if self.requires_migrations_checks:
          self.check_migrations()
      output = self.handle(*args, **options)

  finally:
      if saved_locale is not None:
          translation.activate(saved_locale)
  return output
```
通过调用`handle`，对地址端口进行检查：
```python
def handle(self, *args, **options):
    # 对DEBUG和ALLOWED_HOSTS进行检查
    # ...
        
        # 对端口 地址合法性进行检查
        m = re.match(naiveip_re, options['addrport'])
        if m is None:
            raise CommandError('"%s" is not a valid port number '
                               'or address:port pair.' % options['addrport'])
        self.addr, _ipv4, _ipv6, _fqdn, self.port = m.groups()
        if not self.port.isdigit():
            raise CommandError("%r is not a valid port number." % self.port)
            
    if not self.addr:
        self.addr = '::1' if self.use_ipv6 else '127.0.0.1'
        self._raw_ipv6 = self.use_ipv6
    self.run(**options)
```
在`Pycharm`中，最后调用`run`，`user_reloader`为`True`时，会通过`restart_with_reloader`开启一个新的进程，这个进程再次重复上面的调用过程，当再次调用到`python_reloader`时，开启一个新的线程：
```python
def run(self, **options):
    # 如果use_reloader为True,则会在`autoreload.main`中开启一个新的线程
    if use_reloader:
        autoreload.main(self.inner_run, None, options)
    else:
        self.inner_run(None, **options)
```
新开的线程运行`inner_run`，输出我们常见到的信息：
```python
def inner_run(self, *args, **options):
   try:
       # 通过调用get_internal_wsgi_application获得一个WSGIHandler Object
       # django.core.handlers.wsgi.py中进行WSGIHandler实例化
       handler = self.get_handler(*args, **options)
       run(self.addr, int(self.port), handler,
           ipv6=self.use_ipv6, threading=threading, server_cls=self.server_cls)
   except socket.error as e:
      # ...
```
最终调用`run`方法(`django.core.servers.basehhtp.py`)：
```python
def run(addr, port, wsgi_handler, ipv6=False, threading=False, server_cls=WSGIServer):
    server_address = (addr, port)
    if threading:
        # httpd_cls的类是wsgiserver
        httpd_cls = type(str('WSGIServer'), (socketserver.ThreadingMixIn, server_cls), {})
    else:
        httpd_cls = server_cls
    # 实例化一个WSGIServer, django.core.servers.basehttp.py
    httpd = httpd_cls(server_address, WSGIRequestHandler, ipv6=ipv6)
    if threading:
        httpd.daemon_threads = True
    # wsgi_handler实际上就是application
    httpd.set_app(wsgi_handler)
    httpd.serve_forever()
```

## 请求与响应
在`serve_forever`中通过`select`来监听是否有请求到达：
```python
   def serve_forever(self, poll_interval=0.5):
   # ... 
      with _ServerSelector() as selector:
          selector.register(self, selectors.EVENT_READ)

          while not self.__shutdown_request:
              ready = selector.select(poll_interval)
              if ready:
                  self._handle_request_noblock()

              self.service_actions()
```
一旦有请求到达，则执行`_handle_request_noblock`：
```python
def _handle_request_noblock(self):
    try:
        request, client_address = self.get_request()
    except OSError:
        return
    if self.verify_request(request, client_address):
        try:
            self.process_request(request, client_address)
        except Exception:
            self.handle_error(request, client_address)
            self.shutdown_request(request)
        except:
            self.shutdown_request(request)
            raise
    else:
        self.shutdown_request(request)
```
主要在`process_request`中处理请求：
```python
def process_request(self, request, client_address):
    self.finish_request(request, client_address)
    self.shutdown_request(request)
```
`finish_request`会直接实例化`BaseRequestHandler`：
```python
class BaseRequestHandler:
    def __init__(self, request, client_address, server):
        self.request = request
        self.client_address = client_address
        self.server = server
        self.setup()
        try:
            self.handle()
        finally:
            self.finish()
```
主要是通过`handle()`中的`run()`：
```python
def handle(self):  
    handler = ServerHandler(
        self.rfile, self.wfile, self.get_stderr(), self.get_environ()
    )
    handler.request_handler = self      # backpointer for logging
    handler.run(self.server.get_app())
```
因为`WSGIHandler`实现了`__call__`方法，所以可以直接调用：
```python
class WSGIHandler(base.BaseHandler)：
    def __call__(self, environ, start_response):
        set_script_prefix(get_script_name(environ))
        signals.request_started.send(sender=self.__class__, environ=environ)
        request = self.request_class(environ)
        response = self.get_response(request)

        response._handler_class = self.__class__

        status = '%d %s' % (response.status_code, response.reason_phrase)
        response_headers = [(str(k), str(v)) for k, v in response.items()]
        for c in response.cookies.values():
            response_headers.append((str('Set-Cookie'), str(c.output(header=''))))
        start_response(force_str(status), response_headers)
        if getattr(response, 'file_to_stream', None) is not None and environ.get('wsgi.file_wrapper'):
            response = environ['wsgi.file_wrapper'](response.file_to_stream)
        return response
```
这就是一个熟悉的`application`格式了。流程如图：
![Alt text](/assets/django_wsgi.png)
> 通过`run`方法，会创建`WSGIServer`实例，通过`set_app`和`get_app`方法设置和获取`wsgi_handler`
> 当来新请求时，调用`handler_request_noblock`方法，创建`WSGIRequestHandler`实例处理请求(`finish_request`)
> `WSGIRequestHandler`在实例化的时候，会调用自身的`handle`方法，这个方法会创建一个`ServerHandler`实例，调用其`run`方法
> 在`run`方法中使用`get_app`，获得`WSGIHandler`，获取`response`，传入`start_response`回调，用来处理返回的`header`和`status`，使用`finish_response()`返回`response`


Reference: [做python Web开发你要理解：WSGI & uwsgi](https://www.jianshu.com/p/679dee0a4193)
