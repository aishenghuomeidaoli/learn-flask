# 1.1 werkzeug

flask 代码中定义了以下几个变量：

```
_request_ctx_stack = LocalStack()
current_app = LocalProxy(lambda: _request_ctx_stack.top.app)
request = LocalProxy(lambda: _request_ctx_stack.top.request)
session = LocalProxy(lambda: _request_ctx_stack.top.session)
g = LocalProxy(lambda: _request_ctx_stack.top.g)
```

分别是`LocalStack`、`LocalProxy`的实例，这两个类从`werkzeug.local`引入。

我们可以参考`werkzeug.local`的[文档](http://werkzeug.pocoo.org/docs/0.14/local/)，进行了解。

## wekzeug.local 背景介绍

在视图函数及其他函数中，存在调用同一对象的情况，但使用全局变量存在缺点。

在`WSGI`应用中，我们需要保证线程安全，即任何需要对当前线程操作数据的时候，都能进入指定的线程，并检索到对应的数值。
这样，才能保证不会取到其他线程的数据，`Python`中`thread local`可以实现这个目的。

有些项目会使用`greenlet`来解决并发问题，`greenlet`相比于线程消耗的资源更少，但多个`greenlet`可能共用一个线程，不同请求可能存在于一个线程中，这样单纯的线程隔离不能够满足不同请求之间的数据隔离。

基于以上情况，`werkzeug.local`提供了本地数据存储的解决方案，即无论并发使用协程还是线程，都可以实现请求之间的数据隔离。

参考阅读：

1. [Werkzeug(Flask)之Local、LocalStack和LocalProxy](https://www.jianshu.com/p/3f38b777a621)
2. [Greenlet Vs. Threads](https://stackoverflow.com/questions/15556718/greenlet-vs-threads)

## Local()

```
class Local(object):
    __slots__ = ('__storage__', '__ident_func__')

    def __init__(self):
        object.__setattr__(self, '__storage__', {})
        object.__setattr__(self, '__ident_func__', get_ident)

    def __iter__(self):
        return iter(self.__storage__.items())

    def __call__(self, proxy):
        """Create a proxy for a name."""
        return LocalProxy(self, proxy)

    def __release_local__(self):
        self.__storage__.pop(self.__ident_func__(), None)

    def __getattr__(self, name):
        try:
            return self.__storage__[self.__ident_func__()][name]
        except KeyError:
            raise AttributeError(name)

    def __setattr__(self, name, value):
        ident = self.__ident_func__()
        storage = self.__storage__
        try:
            storage[ident][name] = value
        except KeyError:
            storage[ident] = {name: value}

    def __delattr__(self, name):
        try:
            del self.__storage__[self.__ident_func__()][name]
        except KeyError:
            raise AttributeError(name)
```


`Local`对象包含了`__storage__`和`__ident_func__`属性。`__storage__`初始化时为`{}`，用来存储数据。`__ident_func__`为从`gevent`或`thread`引入的函数，调用该函数可以获取当前线程的整数型唯一标志符。

`Local`对象定义了`__setattr__`方法，可以实现对指定`键(name)`设定`值(value)`。当调用`__setattr__`时，首先获取`线程唯一标识符(ident)`，与`存储空间(storage)`，并更新当前线程对应的存储数据，不同的线程在`__storage__`存储的形式为：以线程唯一标识符为键，其值为需要存储的键值对，例如：


```
>>> from werkzeug.local import Local
>>> local = Local()
>>> local.name = 'user1'
>>> print local.__storage__
{140735291326464: {'name': 'user1'}}
```

## LocalProxy()

`LocalProxy`是一种代理，当调用`LocalProxy`实例时，调用被代理对象的`__call__`方法。

举例：

```
>>> import time
>>> from werkzeug.local import LocalProxy
>>> def now():
...     return time.time()
...

>>> time_a = now()  # 普通函数调用，赋值时，调用now()函数，存放至内存中
>>> time_a
1524732300.538734
>>> time_a
1524732300.538734

>>> time_b = LocalProxy(now)  # time_b为LocaProxy实例
>>> time_b  # 当调用time_b时，返回now.__call__()，即now()
1524732320.073784
>>> time_b
1524732322.481673
```


## LocalStack()

`LocalStack`提供了`栈`的操作，可以进行压栈、出栈。源码示例清晰的展示了其功能：

```
>>> ls = LocalStack()
>>> ls.push(42)
>>> ls.top
42
>>> ls.push(23)
>>> ls.top
23
>>> ls.pop()
23
>>> ls.top
42
```
