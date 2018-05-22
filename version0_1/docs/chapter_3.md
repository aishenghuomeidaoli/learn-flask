# 第6-8行 注册视图函数

```
@app.route('/')
def index():
    return '<p>Hello, World</p>'
```

通过用法我们知道`app.route`为装饰器函数，除了路径参数外，还可以传入`methods`参数。

`Flask.route`代码如下：

```
def route(self, rule, **options):
    def decorator(f):
        self.add_url_rule(rule, f.__name__, **options)
        self.view_functions[f.__name__] = f
        return f
    return decorator
```

装饰器包含两个操作：

* 根据函数名记录视图函数。

`Flask`实例化时：`self.view_functions = {}`，在此处被修饰的`index`函数以`f.__name__`即'index'为键，函数对象为值存入`self.view_functions`。

我们可以修改`index`视图函数，待服务重启后访问[http://localhost:5000/](http://localhost:5000/)查看控制台输出，修改后的代码与输出如下：

```
...
def index():
    from flask import current_app
    print current_app.view_functions
    return '<p>Hello, World</p>'
...
```

```
127.0.0.1 - - [21/May/2018 18:40:49] "GET / HTTP/1.1" 200 -
{'index': <function index at 0x10e3e42a8>}
```

* 增加路由规则。

`add_url_rule`函数如下：

```
def add_url_rule(self, rule, endpoint, **options):
    options['endpoint'] = endpoint
    options.setdefault('methods', ('GET',))
    self.url_map.add(Rule(rule, **options))
```

这里传入的`options`为一个空字典，经过函数体前两行处理后变为`{'endpoint': 'index', 'methods':  ('GET',)}`。

第三行`self.url_map`来自于在`Flask`初始化时用实例化的`werkzeug.routing.Map`对`self.url_map`的赋值。

此处`werkzeug.routing.Map`的功能暂且放下，后续补上相关文档。

我们只要知道这里`self.url_map`调用了`Map.add`方法，`Flask`实例`app`的路由映射记录了`index`的路由。