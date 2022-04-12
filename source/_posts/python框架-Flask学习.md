---
title: python框架-Flask学习
date: 2022-02-28 15:39:52
tags:
---

# 安装
Flask 依赖 Jinja 模板引擎和 Werkzeug WSGI 套件

当安装 Flask 时，以下配套软件会被自动安装
* Werkzeug 用于实现 WSGI ，应用和服务之间的标准 Python 接口。
* Jinja 用于渲染页面的模板语言。
* MarkupSafe 与 Jinja 共用，在渲染页面时用于避免不可信的输入，防止注入攻击。
* ItsDangerous 保证数据完整性的安全标志数据，用于保护 Flask 的 session cookie.
* Click 是一个命令行应用的框架。用于提供 flask 命令，并允许添加自定义 管理命令。

建议在开发环境和生产环境下都使用虚拟环境来管理项目的依赖

为什么要使用虚拟环境？随着你的 Python 项目越来越多，你会发现不同的项目会需要不同的版本的 Python 库。同一个 Python 库的不同版本可能不兼容。

虚拟环境可以为每一个项目安装独立的 Python 库，这样就可以隔离不同项目之间的 Python 库，也可以隔离项目与操作系统之间的 Python 库。

Python 内置了用于创建虚拟环境的 venv 模块。

创建一个项目文件夹，然后创建一个虚拟环境。创建完成后项目文件夹中会有一个 venv 文件夹

```shell
对于macOS/Linux

mkdir myproject
cd myproject
python3 -m venv venv

对于Windows
mkdir myproject
cd myproject
python -3 -m venv venv
```

在开始工作前，先要激活相应的虚拟环境：
```shell
对于macOS/Linux
. venv/bin/activate


对于Windows
venv\Scripts\activate
```


激活后，你的终端提示符会显示虚拟环境的名称。

在已激活的虚拟环境中可以使用如下命令安装 Flask
```shell
pip install Flask
```

# 快速上手

一个最小的 Flask 应用如下：
```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello_world():
    return "<p>Hello, World!</p>"
```

那么，这些代码是什么意思呢？

1、首先我们导入了 Flask 类。该类的实例将会成为我们的 WSGI 应用。

2、接着我们创建一个该类的实例。第一个参数是应用模块或者包的名称。 __name__ 是一个适用于大多数情况的快捷方式。有了这个参数， Flask 才能知道在哪里可以找到模板和静态文件等东西。

3、然后我们使用 route() 装饰器来告诉 Flask 触发函数 的 URL 。

4、函数返回需要在用户浏览器中显示的信息。默认的内容类型是 HTML ，因此字 符串中的 HTML 会被浏览器渲染。

把它保存为 hello.py 或其他类似名称。请不要使用 flask.py 作为应用名称，这会与 Flask 本身发生冲突。

可以使用 flask 命令或者 python 的 -m 开关来运行这个应 用。在运行应用之前，需要在终端里导出 FLASK_APP 环境变量：

```bash
#Bash
$ export FLASK_APP=hello
$ flask run
 * Running on http://127.0.0.1:5000/


 #cmd
 > set FLASK_APP=hello
> flask run
 * Running on http://127.0.0.1:5000/


 #Powershell
 > $env:FLASK_APP = "hello"
> flask run
 * Running on http://127.0.0.1:5000/
```
作为一个捷径，如果文件名为 app.py 或者 wsgi.py ，那么您不 需要设置 FLASK_APP 环境变量。

这样就启动了一个非常简单的内建的服务器。这个服务器用于测试应该是足够了， 但是用于生产可能是不够的。

现在在浏览器中打开 http://127.0.0.1:5000/ ，应该可以看到 Hello World! 字样。

运行服务器后，会发现只有您自己的电脑可以使用服务，而网络中的其他电脑却 不行。缺省设置就是这样的，因为在调试模式下该应用的用户可以执行您电脑中 的任意 Python 代码。

如果您关闭了调试器或信任您网络中的用户，那么可以让服务器被公开访问。 只要在命令行上简单的加上 --host=0.0.0.0 即可:
```
$ flask run --host=0.0.0.0
```

这行代码告诉您的操作系统监听所有公开的 IP 。
