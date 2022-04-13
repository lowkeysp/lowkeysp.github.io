---
title: FastAPI学习
date: 2022-04-12 10:59:48
tags:
---

# FastAPI安装

安装Fast API，通常需要Starlette和Pydantic
* Python3.6+
* Starlette：web部分
* Pydantic：数据部分

其中，Starlette需要的依赖项为：
* requests - TestClient需要
* aiofiles - FileResponse或者StaticFiles需要
* jinja2 - 缺省模板配置需要
* python-multipart - 表单解析需要
* itsdangerous - SessionMiddleware支持需要
* pyyaml - Starlette's SchemaGenerator 支持需要
* graphene - GraphQLApp 支持需要
* ujson - UJSONResponse 需要

Starlette是一个轻量级ASGI框架/工具包。它非常适合用来构建高性能的asyncio 服务，并支持HTTP和WebSockets。

官网地址：https://www.starlette.io/

> ASGI解释
> 
> CGI，通用网关接口(Common Gateway Interface)是一个Web服务器主机提供信息服务的标准接口，通过CGI接口，Web服务器就能够获取客户端提交的信息，转交给服务器端的CGI程序进行处理，最后返回结果给客户端
>
>WSGI,Python Web Server Gateway Interface,指定了web服务器和Python web应用或web框架之间的标准接口，以提高web应用在一系列web服务器间的移植性。
>
>ASGI,异步网关协议接口,介于网络服务和python应用之间的标准接口，能够处理多种通用的协议类型，包括http，http2和websocket
>
>WSGI是基于http协议模式开发的，不支持websocket，而ASGI的诞生解决了python中的WSGI不支持当前的web开发中的一些新的协议标准，同时ASGI支持原有模式和Websocket的扩展,，即ASGI是WSGI的扩展。

Pydantic需要的依赖为：
* ujson - 比较快的JSON解析
* email_validator - email校验

同时还需要：
* uvicorn - 加载和服务程序需要,是一个ASGI服务器
* orjson - ORJSONResponse 需要

可以通过`pip install fastapi[all]`安装以上所有依赖。

> Tips:可以使用python的虚拟环境，在虚拟环境中搭建FastAPI可能会更好一些。
>
>https://blog.csdn.net/qq_44643484/article/details/123251333


测试FastAPI环境是否搭建成功

新建main.py

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"Hello": "World"}
```

之后，运行
```
uvicorn main:app --reload
```
其中，
* main：表示main.py文件
* app：main.py创建的实例 app = FastAPI()
* --reload:代码有改动时服务会自动重启（仅适用于开发环境）

我们访问以下两个地址，可获取自动生成的交互式API文档，并且当代码改动时文档会自动更新。方便我们的开发调试
* http://127.0.0.1:8000/docs (基于 Swagger UI)
* http://127.0.0.1:8000/redoc (基于 ReDoc)

# 路径参数

## 声明URL路径参数
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id):
    return {"item_id": item_id}
```
**路径参数**`item_id`的值会直接作为**实参**`item_id`传递给**路径函数**`read_item`

## 路径参数的变量类型

可以在**路径函数**中声明路径参数的变量类型，下面的例子，item_id被指定为是int类型，如果访问`/items/aa`的话，则会类型错误。如果访问`/items/3`，会进行数据转换，传递给read_item函数时，会从字符串转成int类型。
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```
所有的数据校验能力都是在Pydantic的底层支持下得以实现的。

## 路径参数的预定义值
使用Python Enum来声明路径参数的预定义值

```python
from enum import Enum
from fastapi import FastAPI

#声明了一个类ModelName，它继承自str(这样限制了值的类型必须是字符串类型)和Enum
class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"

app = FastAPI()

@app.get("/model/{model_name}")
#声明了model_name的类型为枚举的ModelName
async def get_model(model_name: ModelName):
    if model_name == ModelName.alexnet:
        return {"model_name": model_name, "message": "Deep Learning FTW!"}
    if model_name.value == "lenet":
        return {"model_name": model_name, "message": "LeCNN all the images"}
    return {"model_name": model_name, "message": "Have some residuals"}
```

## 路径参数中含有文件路径
如果需要在路径参数中包含文件路径时，`/files/{file_path}`,比如这个file_path需要是`home/johndoe/myfile.txt`,也就是整个URL要变成`/files/home/johndoe/myfile.txt`

可以声明为path类型，`/files/{file_path:path}`,`:path`指出了file_path可以匹配任何文件路径
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    return {"file_path": file_path}
```

如果file_path需要是`/home/johndoe/myfile.txt`,则URL就要变成`/files//home/johndoe/myfile.txt`

# 请求参数








