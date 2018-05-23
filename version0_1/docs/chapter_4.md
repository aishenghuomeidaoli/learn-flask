# 第10-11行 启动服务

```
if __name__ == '__main__':
    app.run(debug=True)
```


```
def run(self, host='localhost', port=5000, **options):
    from werkzeug import run_simple
    if 'debug' in options:
        self.debug = options.pop('debug')
    options.setdefault('use_reloader', self.debug)
    options.setdefault('use_debugger', self.debug)
    return run_simple(host, port, self, **options)
```

根据`Flask.run`的源码，我们看到，这里