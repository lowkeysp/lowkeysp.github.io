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

如果函数里的参数不是路径参数的一部分，那么这样的参数就自动被解释为请求参数.

请求参数就是URL中问号('?')后面以'&'间隔开的键值对，它们是URL的一部分，并且参数类型都是字符串类型，比如：
```
http://127.0.0.1:8000/items/?skip=0&limit=10
```
在这个URL中，skip和limit为请求参数

```python
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]

@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```

## 请求参数缺省值

```python
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]

@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```
在上述例子中，skip和limit都是有缺省值的，其中，skip的缺省值为0，limit的缺省值为10，那么，
```
http://127.0.0.1:8000/items/
```
等同于
```
http://127.0.0.1:8000/items/?skip=0&limit=10
```

## 请求参数可选
将请求参数的缺省值设置为None，则该请求参数是可选的，非必须的

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: str, q: str = None):
    if q:
        return {"item_id": item_id, "q": q}
    return {"item_id": item_id}
```
在这个例子中，参数q就是可选的，缺省为None.

同时在这个例子中，我们可以注意到，FastAPI可以非常智能的识别参数种类，这里参数item_id是一个路径参数，而参数q是一个请求参数

## 多路径参数，多请求参数
可以同时声明多个路径参数、多个请求参数，并且不用考虑声明顺序。FastAPI可以准确无误的识别参数类型

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}/items/{item_id}")
async def read_user_item(
    user_id: int, item_id: str, q: str = None, short: bool = False
):
    item = {"item_id": item_id, "owner_id": user_id}
    if q:
        item.update({"q": q})
    if not short:
        item.update(
            {"description": "This is an amazing item that has a long description"}
        )
    return item
```
在这个例子中，user_id和item_id都是路径参数，q和short为请求参数，且q是可选的，short的缺省值是False。

## 请求参数必选
如果一个请求参数没有被设置任何缺省值(包括None)，那么它就是必选的
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_user_item(item_id: str, needy: str):
    item = {"item_id": item_id, "needy": needy}
    return item
```
如果我们没有携带请求参数needy，而是直接访问：
```
http://127.0.0.1:8000/items/foo-item
```
那么我们就会看到如下的错误信息：
```
{
    "detail": [
        {
            "loc": [
                "query",
                "needy"
            ],
            "msg": "field required",
            "type": "value_error.missing"
        }
    ]
}
```
# Request Body
Request Body是从客户端发送到API端的数据内容

## 单个Request Body

```python
#导入Pydantic BaseModel
from pydantic import BaseModel

# 声明需要的数据模型，并且继承自BaseModel
# 与请求参数类似，如果缺省值是None，则意味着是可选的；
class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None

```
可以直接访问数据模型的属性
```python

app = FastAPI()

@app.post("/items/")
# 指定为Item类型
async def create_item(item: Item):

    item_dict = item.dict()
    if item.tax:
        price_with_tax = item.price + item.tax
        item_dict.update({"price_with_tax": price_with_tax})
    return item_dict
```

通过下面数据进行访问，会自动识别成item类型
```
{
    "name":"Foo",
    "description":"An optional description",
    "price":45.2,
    "tax":3.5
}
```

## 同时使用Request Body参数、路径参数、请求参数
可以同时声明Request Body参数、路径参数、请求参数这几种不同类型的参数，FastAPI会自动识别不同类型的参数，并且赋予参数对象正确的数据内容

```python
from fastapi import FastAPI
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None

app = FastAPI()

@app.put("/items/{item_id}")
async def create_item(item_id: int, item: Item, q: str = None):
    result = {"item_id": item_id, **item.dict()}
    if q:
        result.update({"q": q})
    return result
```
函数参数按照如下的顺序进行识别匹配：

(1)、如果这个参数已经在路径中被声明过，那么它就是一个路径参数。

(2)、如果这个参数的类型是单类型的(如str、float、int、bool等)，那么它就是一个请求参数。

(3)、如果这个参数的类型是Pydantic数据模型，那么它就被认为是Request Body参数。

## 多个Request Body
可同时声明多个Request Body参数
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None

class User(BaseModel):
    username: str
    full_name: str = None

@app.put("/items/{item_id}")
async def update_item(*, item_id: int, item: Item, user: User):
    results = {"item_id": item_id, "item": item, "user": user}
    return results
```
这两个参数(item、user)类型都是Pydantic数据模型

要注意，由于是多个request body，所以需要key来做区分，这里body里的key就是参数的名称，body内容格式如下示例：
```
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    },
    "user": {
        "username": "dave",
        "full_name": "Dave Grohl"
    }
}
```

## 嵌入单个Request Body

如果是单个Request Body的话，则直接提到，使用的是
```
{
    "name":"Foo",
    "description":"An optional description",
    "price":45.2,
    "tax":3.5
}
```
如果是使用如下格式：
```
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    }
}
```
这种叫做单个Request Body的嵌入，这里"item"是Request Body内部数据内容的key，那么我们就需要利用Body方法的embed参数，才能正确解析出Request Body内容
```
item: Item = Body(..., embed=True)
```
具体代码如下：
```python
from fastapi import Body, FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None


@app.put("/items/{item_id}")
async def update_item(*, item_id: int, item: Item = Body(..., embed=True)):
    results = {"item_id": item_id, "item": item}
    return results
```

## 单个数值
对于Request Body里的单个数值，FastAPI提供了便利的操作方法Body。

Request Body的数据格式如下：
```
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    },
    "user": {
        "username": "dave",
        "full_name": "Dave Grohl"
    },
    "importance": 5
}
```
其中，importance为单个数值，
可通过`Body(...)`获取，具体如下：
```python
from fastapi import Body, FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None

class User(BaseModel):
    username: str
    full_name: str = None

@app.put("/items/{item_id}")
# 如果没有Body(...)方法的话，则只是看成一个请求参数
async def update_item(
    *, item_id: int, item: Item, user: User, importance: int = Body(...)
):
    results = {"item_id": item_id, "item": item, "user": user, "importance": importance}
    return results
```

# 参数附加信息

## 请求参数附加信息
使用Query模块来对**请求参数**添加附加信息

```python

from fastapi import Query

# 可选参数声明
q: str = Query(None)  # 等同于  q: str = None

# 缺省值参数声明
q: str = Query("query")  # 等同于  q: str = "query"

# 必选参数声明
q: str = Query(...)  # 等同于 q: str
```
主要用于字符串参数的附加信息
```
min_length：最小长度
max_length：最大长度
regex：正则表达式
```
实例：
```python
q: str = Query(None, max_length=50)  # 限制q的最大长度不超过50
```
主要用于自动化文档的附加信息
```
title：参数标题
description：参数描述信息
deprecated：表示参数即将过期
```
特殊附加信息
```
alias：参数别名
```
实例如下：
```python
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(
    q: str = Query(
        None,
        alias="item-query",
        title="Query string",
        description="Query string for the items to search in the database that have a good match",
        min_length=3,
        max_length=50,
        regex="^fixedquery$",
        deprecated=True
    )
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

## 请求参数列表
FastAPI基于Query模块可以支持**请求参数**列表，例如请求参数q可以在URL中出现多次：
```
http://localhost:8000/items/?q=foo&q=bar
```
对应代码如下：
```python
from typing import List
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: List[str] = Query(None)):
    query_items = {"q": q}
    return query_items
```
结果返回为：
```
{
  "q": [
    "foo",
    "bar"
  ]
}
```
Query也支持请求参数列表的缺省值设置
```python
from typing import List
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: List[str] = Query(["foo", "bar"])):
    query_items = {"q": q}
    return query_items
```

## 路径参数附加信息
FastAPI通过Path模块实现对**路径参数**附加信息的支持

```python
from fastapi import Path

```
所有适用于请求参数的附加信息同样适用于路径参数
```python
item_id: int = Path(..., title="The ID of the item to get")　
```
注意，路径参数在URL里是必选的，因此Path的第一个参数是...，即使你传递了None或其他缺省值，也不会影响参数的必选性

代码示例：
```python
from fastapi import FastAPI, Path

app = FastAPI()

@app.get("/items/{item_id}")
async def read_items(
    q: str, item_id: int = Path(..., title="The ID of the item to get")
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```
## 路径参数、请求参数顺序问题
### 不带缺省值的参数应放在前面
如果把带有缺省值的参数放在了不带缺省值的参数前面，Python会发出运行警告。

因此在实际使用时，我们应当把不带缺省值的参数放在前面，无论这个参数是路径参数还是请求参数。
```python
from fastapi import FastAPI, Path

app = FastAPI()

@app.get("/items/{item_id}")

##把q放在前面，因为q是不带缺省值的，如果item_id在前，q在后的话，会直接报错，SyntaxError: non-default argument follows default argument
async def read_items(
    q: str, item_id: int = Path(..., title="The ID of the item to get")
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```
编程过程中一直考虑顺序很讨厌，可以通过传递`*`参数，则不需要考虑参数顺序问题。
```python
from fastapi import FastAPI, Path

app = FastAPI()

@app.get("/items/{item_id}")
async def read_items(
    *, item_id: int = Path(..., title="The ID of the item to get"), q: str
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```
Python并不会对 * 做任何操作。但通过识别 *，Python知道后面所有的参数都是关键字参数(键值对)，通常也叫kwargs，无论参数是否有默认值

## 数字类型参数的校验
借助Query、Path等模块,可实现对数字类型参数的校验功能

```
gt： 大于(greater than)
ge： 大于或等于(greater than or equal)
lt： 小于(less than)
le： 小于或等于( less than or equal)
```
具体实例如下：
```python
from fastapi import FastAPI, Path

app = FastAPI()

@app.get("/items/{item_id}")
async def read_items(
    *,
    item_id: int = Path(..., title="The ID of the item to get", gt=0, le=1000),
    q: str,
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```
float类型也适用
```python
from fastapi import FastAPI, Path, Query

app = FastAPI()

@app.get("/items/{item_id}")
async def read_items(
    *,
    item_id: int = Path(..., title="The ID of the item to get", ge=0, le=1000),
    q: str,
    size: float = Query(..., gt=0, lt=10.5)
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```
# Pydantic复杂模型
通过Filed模块，为Pydantic模型添加附加信息

Field模块的参数与Query、Path、Body等相同

```python
from fastapi import Body, FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str = Field(None, title="The description of the item", max_length=300)
    price: float = Field(..., gt=0, description="The price must be greater than zero")
    tax: float = None

@app.put("/items/{item_id}")
async def update_item(*, item_id: int, item: Item = Body(..., embed=True)):
    results = {"item_id": item_id, "item": item}
    return results
```
## Pydantic嵌套模型
模型的属性可以是数据集合类型
```python

# tags属性是list类型
class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None
    tags: list = []

# tags属性是List类型
from typing import List
class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None
    tags: List[str] = []

# tags属性是Set类型
from typing import Set
class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None
    tags: Set[str] = set()


# tags属性是Dict类型
from typing import Dict
class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None
    tags: Dict[int, float]

```
模型的属性可以是另一个Pydantic模型
```python
from typing import Set
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Image(BaseModel):
    url: str
    name: str

class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None
    tags: Set[str] = []
    image: Image = None

@app.put("/items/{item_id}")
async def update_item(*, item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

则，Request Body为：
```
{
    "name": "Foo",
    "description": "The pretender",
    "price": 42.0,
    "tax": 3.2,
    "tags": ["rock", "metal", "bar"],
    "image": {
        "url": "http://example.com/baz.jpg",
        "name": "The Foo live"
    }
}
```
也可以把Pydantic模型作为list、set等集合类型的元素类型
```python
from typing import List, Set
from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()

class Image(BaseModel):
    url: HttpUrl
    name: str

class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None
    tags: Set[str] = []
    images: List[Image] = None

@app.put("/items/{item_id}")
async def update_item(*, item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results

```
Request Body内容格式如下：
```
{
    "name": "Foo",
    "description": "The pretender",
    "price": 42.0,
    "tax": 3.2,
    "tags": [
        "rock",
        "metal",
        "bar"
    ],
    "images": [
        {
            "url": "http://example.com/baz.jpg",
            "name": "The Foo live"
        },
        {
            "url": "http://example.com/dave.jpg",
            "name": "The Baz"
        }
    ]
}
```

# 复杂数据类型

* UUID

标准的通用唯一标识符(Universally Unique Identifier)，一般在数据库或者系统中表示ID

在requests和responses中被认为是字符串类型

* datetime.datetime

标准的Python datetime.datetime

在requests和responses中被认为是字符串类型，例如"2008-09-15T15:53:00+05:00"

* datetime.date

标准的Python datetime.date。

在requests和responses中被认为是字符串类型，例如"2008-09-15"

* datetime.time

标准的Python datetime.time

在requests和responses中被认为是字符串类型，例如"14:23:55.003"

* datetime.timedelta

标准的Python datetime.timedelta

在requests和responses中被认为是表示秒数的float类型，例如"2008-09-15"

* frozenset

在requests和responses中等同于set

在requests中，列表数据会先进行去重，然后转换成set

在responses中，set会被转换成list

* bytes

标准的Python bytes

在requests和responses中被认为是字符串类型

* Decimal

标准的Python Decimal

在requests和responses中被认为是float类型

```python
from datetime import datetime, time, timedelta
from uuid import UUID
from fastapi import Body, FastAPI

app = FastAPI()

@app.put("/items/{item_id}")
async def read_items(
    item_id: UUID,
    start_datetime: datetime = Body(None),
    end_datetime: datetime = Body(None),
    repeat_at: time = Body(None),
    process_after: timedelta = Body(None),
):
    start_process = start_datetime + process_after
    duration = end_datetime - start_process
    return {
        "item_id": item_id,
        "start_datetime": start_datetime,
        "end_datetime": end_datetime,
        "repeat_at": repeat_at,
        "process_after": process_after,
        "start_process": start_process,
        "duration": duration,
    }
```
测试：
```
127.0.0.1:8000/items/a8098c1a-f86e-11da-bd1a-00112444be1e

Body:

{
    "start_datetime": "2008-09-15T15:53:00+05:00",
    "end_datetime": "2018-09-15T15:53:00+05:00",
    "repeat_at": "14:23:55.003",
    "process_after": "20080915"

}
```

# Cookie参数


利用Cookie模块声明cookie参数，与Query,Path类似，
```python
from typing import Optional
from fastapi import Cookie, FastAPI

app = FastAPI()

@app.get("/items/")
async def read_items(ads_id: Optional[str] = Cookie(None)):
    return {"ads_id": ads_id}
```
则，请求中Cookie包含"ads_id = "xx"的话，则会直接解析出来 


可以在Response中返回Cookie信息给终端

* 使用Response参数

在路径操作函数中声明Response参数，然后给这个临时的Response对象设置cookie信息,FastAPI通过这个临时的Response对象解析出cookie信息，然后放入到最终返回的Response对象中。

```python
from fastapi import FastAPI, Response

app = FastAPI()

@app.post("/cookie-and-object/")
def create_cookie(response: Response):
    response.set_cookie(key="fakesession", value="fake-cookie-session-value")
    return {"message": "Come to the dark side, we have cookies"}
```

* 直接返回Response

也可以在直接返回的Response对象中设置cookie信息
```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()

@app.post("/cookie/")
def create_cookie():
    content = {"message": "Come to the dark side, we have cookies"}
    
    response = JSONResponse(content=content)
    response.set_cookie(key="fakesession", value="fake-cookie-session-value")
    return response
```

# Header参数


## 声明Header参数
使用定义 Query, Path 和 Cookie 参数一样的方法定义 Header 参数

```python
from typing import Optional
from fastapi import FastAPI, Header

app = FastAPI()

@app.get("/items/")
async def read_items(user_agent: Optional[str] = Header(None)):
    return {"User-Agent": user_agent}
```


## 自动转换

Header 在 Path, Query 和 Cookie 提供的功能之上有一点额外的功能。

大多数标准的headers用 "连字符" 分隔，也称为 "减号" (-)。

但是像 user-agent 这样的变量在Python中是无效的。

因此, 默认情况下, Header 将把参数名称的字符从下划线 (_) 转换为连字符 (-) 来提取并记录 headers.

同时，HTTP headers 是大小写不敏感的，因此，因此可以使用标准Python样式(也称为 "snake_case")声明它们。

因此，您可以像通常在Python代码中那样使用 user_agent ，而不需要将首字母大写为 User_Agent 或类似的东西。

如果出于某些原因，你需要禁用下划线到连字符的自动转换，设置Header的参数 convert_underscores 为 False:

```python
from typing import Optional
from fastapi import FastAPI, Header

app = FastAPI()

@app.get("/items/")
async def read_items(
    strange_header: Optional[str] = Header(None, convert_underscores=False)
):
    return {"strange_header": strange_header}
```

在设置 convert_underscores 为 False 之前，请记住，一些HTTP代理和服务器不允许使用带有下划线的headers


## 重复的 headers
有可能收到重复的headers。这意味着，相同的header具有多个值

您可以在类型声明中使用一个list来定义这些情况。

你可以通过一个Python list 的形式获得重复header的所有值。

比如, 为了声明一个 X-Token header 可以出现多次，你可以这样写：
```python
from typing import List, Optional
from fastapi import FastAPI, Header

app = FastAPI()

@app.get("/items/")
async def read_items(x_token: Optional[List[str]] = Header(None)):
    return {"X-Token values": x_token}
```

如果你与路径操作通信时发送两个HTTP headers，就像：
```
X-Token: foo
X-Token: bar
```
响应会是:
```
{
    "X-Token values": [
        "bar",
        "foo"
    ]
}
```

# Response模型
在路径操作中，我们可以用参数response_model来声明Response模型

```python
from typing import List
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None
    tags: List[str] = []

@app.post("/items/", response_model=Item)
async def create_item(item: Item):
    return item
```
注意response_model是装饰器方法(get，post等)的参数

Response模型可以是一个Pydantic模型，也可以是一个Pydantic模型的列表，例如`List[Item]`

支持任意路径操作
* @app.get()
* @app.post()
* @app.put()
* @app.delete()

Response模型最主要的是限制输出数据只能是所声明的Response模型


## 输入输出模型示例

```python
from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()

class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr
    full_name: str = None

class UserOut(BaseModel):
    username: str
    email: EmailStr
    full_name: str = None

@app.post("/user/", response_model=UserOut)
async def create_user(*, user: UserIn):
    return user
```

如上所示，虽然路径操作函数返回的结果是user(包含了password)，但我们声明的Response模型是UserOut(不包含password)。

FastAPI会过滤掉所有不在输出模型中的数据，因此最终的输出结果里并没有password

如果输入内容如下：
```
{
    "username": "user",
    "password": "1234",
    "email": "user@qq.com",
    "full_name": "full_name"
}
```
那么输出结果为：
```
{
    "username": "user",
    "email": "user@qq.com",
    "full_name": "full_name"
}
```

## Response模型参数


```python
from typing import List
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = 10.5
    tags: List[str] = []

items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}

@app.get("/items/{item_id}", response_model=Item, response_model_exclude_unset=True)
async def read_item(item_id: str):
    return items[item_id]
```
响应模型可以具有默认值，比如，
* description: Optional[str] = None 具有默认值 None
* tax: float = 10.5 具有默认值 10.5
* tags: List[str] = [] 具有一个空列表作为默认值： []



### response_model_exclude_unset 参数
但如果它们并没有存储实际的值，你可能想从结果中忽略它们的默认值。

举个例子，当你在 NoSQL 数据库中保存了具有许多可选属性的模型，但你又不想发送充满默认值的很长的 JSON 响应

你可以设置路径操作装饰器的 response_model_exclude_unset=True 参数：

然后响应中将不会包含那些默认值，而是仅有实际设置的值。

则，当你访问：
```# 访问：
http://127.0.0.1:8000/items/foo
```
则返回：
```
{
    "name": "Foo",
    "price": 50.2
}
```

### response_model_include 和 response_model_exclude 参数

实际中尽量少用这两个参数，而是应该声明不同的类表示不同的数据需求，这样更利于数据维护和逻辑清晰
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = 10.5

items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The Bar fighters", "price": 62, "tax": 20.2},
    "baz": {
        "name": "Baz",
        "description": "There goes my baz",
        "price": 50.2,
        "tax": 10.5,
    },
}

#包含name, description
@app.get("/items/{item_id}/name", response_model=Item, response_model_include={"name", "description"})
async def read_item_name(item_id: str):
    return items[item_id]

#排除tax
@app.get("/items/{item_id}/public", response_model=Item, response_model_exclude={"tax"})
async def read_item_public_data(item_id: str):
    return items[item_id]
```

## Response联合（Union）模型
可以声明Response模型是一个Union类型(包含两种类型)，实际返回结果可以是Union其中任何一个

```python
from typing import Union
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class BaseItem(BaseModel):
    description: str
    type: str

class CarItem(BaseItem):
    type = "car"

class PlaneItem(BaseItem):
    type = "plane"
    size: int

items = {
    "item1": {"description": "All my friends drive a low rider", "type": "car"},
    "item2": {
        "description": "Music is my aeroplane, it's my aeroplane",
        "type": "plane",
        "size": 5,
    },
}

@app.get("/items/{item_id}", response_model=Union[PlaneItem, CarItem])
async def read_item(item_id: str):
    return items[item_id]
```

## Response列表模型
Response模型也可以是一个列表
```python
from typing import List
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str

items = [
    {"name": "Foo", "description": "There comes my hero"},
    {"name": "Red", "description": "It's my aeroplane"},
]

@app.get("/items/", response_model=List[Item])
async def read_items():
    return items
```
## Response字典模型

```python
from typing import Dict
from fastapi import FastAPI

app = FastAPI()


@app.get("/keyword-weights/", response_model=Dict[str, float])
async def read_keyword_weights():
    return {"foo": 2.3, "bar": 3.4}
```
