[TOC]

# flask 0.1

我们先安装 [learn-flask/version0_1/requirements.txt](../requirements.txt) 中的 Jinja2==2.4 和 Werkzeug==0.6.1。

为了方便查看 flask 源码，在源码中打印日志，我们已经把 flask 复制到项目内了，无需单独安装，位于 [learn-flask/version0_1/flask.py](../flask.py)。

## 1. 由最简单的 Flask 应用入手

[version0_1/app1.py](../app1.py)中写有最简单的 flask 示例。我们逐行进行分析：


```
from flask import Flask

app = Flask(__name__)


@app.route('/')
def index():
    return '<p>Hello, World</p>'

if __name__ == '__main__':
    app.run(debug=True)
```


* [第1行 引入 Flask 类](./chapter_1.md)

* [第3行 实例化 Flask](./chapter_2.md)

* [第6-8行 注册视图函数](./chapter_3.md)

* [第10-11行 启动服务](./chapter_4.md)

* [访问 “http://localhost:5000/” 发生了什么](./chapter_5.md)
