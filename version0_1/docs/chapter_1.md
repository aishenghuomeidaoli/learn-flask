# 第1行 引入 Flask 类

```
from flask import Flask
```

这行代码会执行 flask 模块内的函数，并将引入 Flask 类。观察 flask.py 文件，可以把整个代码分为三个部分：

* 引入依赖：

```
from __future__ import with_statement
import os
import sys

from threading import local
from jinja2 import Environment, PackageLoader, FileSystemLoader
from werkzeug import Request as RequestBase, Response as ResponseBase, \
     LocalStack, LocalProxy, create_environ, cached_property, \
     SharedDataMiddleware
from werkzeug.routing import Map, Rule
from werkzeug.exceptions import HTTPException, InternalServerError
from werkzeug.contrib.securecookie import SecureCookie

from werkzeug import abort, redirect
from jinja2 import Markup, escape

try:
    import pkg_resources
    pkg_resources.resource_stream
except (ImportError, AttributeError):
    pkg_resources = None
```


* 定义函数与类，此处省略函数体：

```
class Request(RequestBase):
    ...

class Response(ResponseBase):
    ...

class _RequestGlobals(object):
    pass

class _RequestContext(object):
    ...

def url_for(endpoint, **values):
    ...

def flash(message):
    ...

def get_flashed_messages():
    ...

def render_template(template_name, **context):
    ...

def render_template_string(source, **context):
    ...

def _default_template_ctx_processor():
    ...

def _get_package_path(name):
    ...

class Flask(object):
    ...
```

* 定义变量

```
_request_ctx_stack = LocalStack()
current_app = LocalProxy(lambda: _request_ctx_stack.top.app)
request = LocalProxy(lambda: _request_ctx_stack.top.request)
session = LocalProxy(lambda: _request_ctx_stack.top.session)
g = LocalProxy(lambda: _request_ctx_stack.top.g)
```

大家都知道，引入作用是为了使用，我们到具体使用的时候，再说明具体功能，第一部分略过。

第二部分是函数及`Flask`类的定义，我们下一部分实例化会讲到。

第三部分是变量的定义，主要使用了`werkzeug.local`模块，我们在[1.1-werkzeug](./1_1werkzeug.md)中介绍了一些`werkzeug.local`模块的内容，总结起来它的作用是：

* `Flask`借助`LocalStack()`实现了线程隔离，无论并发的实现是通过`协程(gevent)`还是`线程(thread)`，都以当前的`线程唯一标识符`作为键来进行存储，实现了线程隔离。

* 借助`LocalProxy()`, `Flask`的四个全局变量`current_app`, `request`, `session`, `g`在每一次调用时都是实时获取，而不是一个固定的对象。举一个极端的例子，第一行引入`Flask`时，`__request_ctx_stack.top`为`None`。
