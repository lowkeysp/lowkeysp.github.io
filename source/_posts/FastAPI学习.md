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






## 多个模型
```python
from typing import Optional

from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()

class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr
    full_name: Optional[str] = None

class UserOut(BaseModel):
    username: str
    email: EmailStr
    full_name: Optional[str] = None

class UserInDB(BaseModel):
    username: str
    hashed_password: str
    email: EmailStr
    full_name: Optional[str] = None

def fake_password_hasher(raw_password: str):
    return "supersecret" + raw_password

def fake_save_user(user_in: UserIn):
    hashed_password = fake_password_hasher(user_in.password)
    user_in_db = UserInDB(**user_in.dict(), hashed_password=hashed_password)
    print("User saved! ..not really")
    return user_in_db

@app.post("/user/", response_model=UserOut)
async def create_user(user_in: UserIn):
    user_saved = fake_save_user(user_in)
    return user_saved
```

其上是包含多个模型，有用户输入模型，用户输出模型，数据库模型等，用户输入模型需要拥有密码属性，输出模型不应该包含密码，数据库模型需要保存密码的哈希值。

### 关于**user_in.dict()

user_in是UserIn类的Pydantic模型。

Pydantic 模型具有 .dict（） 方法，该方法返回一个拥有模型数据的 dict。

因此，如果我们像下面这样创建一个 Pydantic 对象 user_in：
```python
user_in = UserIn(username="john", password="secret", email="john.doe@example.com")
```
然后我们调用：
```python
user_dict = user_in.dict()
```
现在我们有了一个数据位于变量 user_dict 中的 dict（它是一个 dict 而不是 Pydantic 模型对象）。

如果我们调用：
```python
print(user_dict)
```
我们将获得一个这样的 Python dict：
```
{
    'username': 'john',
    'password': 'secret',
    'email': 'john.doe@example.com',
    'full_name': None,
}
```
如果我们将 user_dict 这样的 dict 以 `**user_dict`形式传递给一个函数（或类），Python将对其进行 **解包**。它会将 user_dict 的键和值作为关键字参数直接传递。

因此，从上面的 user_dict 继续，编写：
```python
UserInDB(**user_dict)
```
会产生类似于以下的结果：
```python
UserInDB(
    username="john",
    password="secret",
    email="john.doe@example.com",
    full_name=None,
)
```
或者更确切地，直接使用 user_dict 来表示将来可能包含的任何内容：
```python
UserInDB(
    username = user_dict["username"],
    password = user_dict["password"],
    email = user_dict["email"],
    full_name = user_dict["full_name"],
)
```
如上例所示，我们从 user_in.dict（） 中获得了 user_dict，此代码：
```python
user_dict = user_in.dict()
UserInDB(**user_dict)
```
等同于
```python
UserInDB(**user_in.dict())
```
因为 user_in.dict() 是一个 dict，然后我们通过以`**`开头传递给 UserInDB 来使 Python**解包**它。

这样，我们获得了一个来自于其他 Pydantic 模型中的数据的 Pydantic 模型。

然后添加额外的关键字参数 hashed_password=hashed_password，例如：
```python
UserInDB(**user_in.dict(), hashed_password=hashed_password)
```
最终结果如下:
```python
UserInDB(
    username = user_dict["username"],
    password = user_dict["password"],
    email = user_dict["email"],
    full_name = user_dict["full_name"],
    hashed_password = hashed_password,
)
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
# Response状态码

## 通过参数status_code自定义状态码

```python
from fastapi import FastAPI

app = FastAPI()

@app.post("/items/", status_code=201)
async def create_item(name: str):
    return {"name": name}
```
支持任意路径操作
```
@app.get()
@app.post()
@app.put()
@app.delete()
```
status_code 参数接收一个表示 HTTP 状态码的数字,status_code 是装饰器方法（get，post 等）的一个参数。不像之前的所有参数和请求体，它不属于路径操作函数。

可以从`fastapi.status`导入状态码常量，便于使用和记忆
```python
from fastapi import FastAPI, status

app = FastAPI()

@app.post("/items/", status_code=status.HTTP_201_CREATED)
async def create_item(name: str):
    return {"name": name}
```

## 通过Response参数自定义状态码
可以在路径操作函数中声明Response参数，然后给这个临时的Response对象设置status_code信息

FastAPI通过这个临时的Response对象解析出status_code信息(以及header、cookie信息等)，然后放入到最终返回的Response对象中

```python
from fastapi import FastAPI, Response, status

app = FastAPI()

tasks = {"foo": "Listen to the Bar Fighters"}

@app.put("/get-or-create-task/{task_id}", status_code=200)
def get_or_create_task(task_id: str, response: Response):
    if task_id not in tasks:
        tasks[task_id] = "This didn't exist before" 
        response.status_code = status.HTTP_201_CREATED
    return tasks[task_id]
```

## 通过直接返回Response自定义状态码
FastAPI默认情况下会通过JSONResponse返回请求结果，返回的状态码是默认状态码或者是路径操作中设置的状态码

如果我们想自定义状态码，可以通过直接返回Response对象来设置状态码，比如直接返回JSONResponse对象。

如下示例，我们需要先导入JSONResponse，然后自定义状态码，直接返回数据对象
```python
from typing import Optional
from fastapi import Body, FastAPI, status
from fastapi.responses import JSONResponse

app = FastAPI()

items = {"foo": {"name": "Fighters", "size": 6}, "bar": {"name": "Tenders", "size": 3}}

@app.put("/items/{item_id}")
async def upsert_item(item_id: str, name: Optional[str] = Body(None), size: Optional[int] = Body(None)):
    if item_id in items:
        item = items[item_id]
        item["name"] = name
        item["size"] = size
        return item
    else:
        item = {"name": name, "size": size}
        items[item_id] = item
        return JSONResponse(status_code=status.HTTP_201_CREATED, content=item)
```
>100 及以上状态码用于「消息」响应。你很少直接使用它们。具有这些状态代码的响应不能带有响应体。
>
>200 及以上状态码用于「成功」响应。这些是你最常使用的。200 是默认状态代码，它表示一切「正常」。另一个例子会是 201，「已创建」。它通常在数据库中创建了一条新记录后使用。一个特殊的例子是 204，「无内容」。此响应在没有内容返回给客户端时使用，因此该响应不能包含响应体。
>
>300 及以上状态码用于「重定向」。具有这些状态码的响应可能有或者可能没有响应体，但 304「未修改」是个例外，该响应不得含有响应体。
>
>400 及以上状态码用于「客户端错误」响应。这些可能是你第二常使用的类型。一个例子是 404，用于「未找到」响应。对于来自客户端的一般错误，你可以只使用 400。
>
>500 及以上状态码用于服务器端错误。你几乎永远不会直接使用它们。当你的应用程序代码或服务器中的某些部分出现问题时，它将自动返回这些状态代码之一。

# 表单数据
接收的不是 JSON，而是表单字段时，要使用 Form

要使用表单，需预先安装 python-multipart
```
pip install python-multipart
```

声明表单参数的方式与Query或者Path相同
```python
from fastapi import Form

from fastapi import FastAPI

app = FastAPI()


@app.post("/login/")
async def login(*, username: str = Form(...), password: str = Form(...)):
     return {"username": username}
```

# 请求文件

File用于定义客户端的上传文件，上传文件以「表单数据」形式发送

## 导入文件


```python
from fastapi import FastAPI, File, UploadFile

app = FastAPI()

@app.post("/files/")
async def create_file(file: bytes = File(...)):
    return {"file_size": len(file)}


@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile = File(...)):
    return {"filename": file.filename}
```
创建文件（File）参数的方式与 Body 和 Form 一样，File 是直接继承自 Form 的类。

如果把路径操作函数参数的类型声明为 bytes，FastAPI 将以 bytes 形式读取和接收文件内容

这种方式把文件的所有内容都存储在内存里，适用于小型文件

不过，很多情况下，UploadFile 更好用，UploadFile 与 bytes 相比有更多优势：
* 使用 spooled 文件：存储在内存的文件超出最大上限时，FastAPI 会把文件存入磁盘；
* 这种方式更适于处理图像、视频、二进制文件等大型文件，好处是不会占用所有内存；
* 可获取上传文件的元数据；
* 自带 file-like async 接口；
* 暴露的 Python SpooledTemporaryFile 对象，可直接传递给其他预期「file-like」对象的库。

UploadFile 的属性如下:
* filename：上传文件名字符串（str），例如， myimage.jpg；
* content_type：内容类型（MIME 类型 / 媒体类型）字符串（str），例如，image/jpeg；
* file： SpooledTemporaryFile（ file-like 对象）。其实就是 Python文件，可直接传递给其他预期 file-like 对象的函数或支持库。

UploadFile 支持以下 async 方法，（使用内部 SpooledTemporaryFile）可调用相应的文件方法。
* write(data)：把 data （str 或 bytes）写入文件；
* read(size)：按指定数量的字节或字符（size (int)）读取文件内容；
* seek(offset)：移动至文件 offset （int）字节处的位置；例如，await myfile.seek(0) 移动到文件开头；执行 await myfile.read() 后，需再次读取已读取内容时，这种方法特别好用；
* close()：关闭文件。

因为上述方法都是 async 方法，要搭配「await」使用c

例如，在 async 路径操作函数 内，要用以下方式读取文件内容：
```python
contents = await myfile.read()
```
在普通 def 路径操作函数 内，则可以直接访问 UploadFile.file，例如：
```python
contents = myfile.file.read()
```
## 多文件上传
可用同一个「表单字段」发送含多个文件的「表单数据」

上传多个文件时，要声明含 bytes 或 UploadFile 的列表（List）

```python
from typing import List

from fastapi import FastAPI, File, UploadFile
from fastapi.responses import HTMLResponse

app = FastAPI()

#使用List实现多文件上传
@app.post("/files/")
async def create_files(files: List[bytes] = File(...)):
    return {"file_sizes": [len(file) for file in files]}


@app.post("/uploadfiles/")
async def create_upload_files(files: List[UploadFile] = File(...)):
    return {"filenames": [file.filename for file in files]}


@app.get("/")
async def main():
    content = """
<body>
<form action="/files/" enctype="multipart/form-data" method="post">
<input name="files" type="file" multiple>
<input type="submit">
</form>
<form action="/uploadfiles/" enctype="multipart/form-data" method="post">
<input name="files" type="file" multiple>
<input type="submit">
</form>
</body>
    """
    return HTMLResponse(content=content)
```

# 请求表单与文件
FastAPI 支持同时使用 File 和 Form 定义文件和表单字段
```python
from fastapi import FastAPI, File, Form, UploadFile

app = FastAPI()

@app.post("/files/")
async def create_file(
    file: bytes = File(...), fileb: UploadFile = File(...), token: str = Form(...)
):
    return {
        "file_size": len(file),
        "token": token,
        "fileb_content_type": fileb.content_type,
    }
```
在同一个请求中接收数据和文件时，应同时使用 File 和 Form

# 错误处理
某些情况下，需要向客户端返回错误提示


## HTTPException
向客户端返回 HTTP 错误响应，可以使用 HTTPException
```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

items = {"foo": "The Foo Wrestlers"}

@app.get("/items/{item_id}")
async def read_item(item_id: str):
    if item_id not in items:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item": items[item_id]}
```
## 添加自定义响应头
```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

items = {"foo": "The Foo Wrestlers"}

@app.get("/items-header/{item_id}")
async def read_item_header(item_id: str):
    if item_id not in items:
        raise HTTPException(
            status_code=404,
            detail="Item not found",
            headers={"X-Error": "There goes my error"},
        )
    return {"item": items[item_id]}
```

## 自定义异常处理器
使用Starlette的异常工具（https://www.starlette.io/exceptions/），可以自定义异常处理器

假设要触发的自定义异常叫作 UnicornException，且需要 FastAPI 实现全局处理该异常，此时，可以用 @app.exception_handler() 添加自定义异常控制器
```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

class UnicornException(Exception):
    def __init__(self, name: str):
        self.name = name

app = FastAPI()

@app.exception_handler(UnicornException)
async def unicorn_exception_handler(request: Request, exc: UnicornException):
    return JSONResponse(
        status_code=418,
        content={"message": f"Oops! {exc.name} did something. There goes a rainbow..."},
    )

@app.get("/unicorns/{name}")
async def read_unicorn(name: str):
    if name == "yolo":
        raise UnicornException(name=name)
    return {"unicorn_name": name}
```
请求 `/unicorns/yolo` 时，路径操作函数就会抛出异常 UnicornException，这个异常会被我们的异常处理器unicorn_exception_handler捕获到,收到的HTTP状态码就是418，内容如下：
```
{"message": "Oops! yolo did something. There goes a rainbow..."}
```

## 覆盖缺省异常处理器
FastAPI有一些缺省的异常处理器，触发 HTTPException 或请求无效数据时，这些处理器返回缺省的 JSON 响应结果。

可以重写这些缺省异常处理器

当一个请求包含非法数据的时候，FastAPI内部会抛出RequestValidationError异常，并且有默认的异常处理器来处理。

可以用`@app.exception_handler(RequestValidationError)`来重写这个异常处理器
```python
from fastapi import FastAPI, HTTPException
from fastapi.exceptions import RequestValidationError
from fastapi.responses import PlainTextResponse

app = FastAPI()

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    return PlainTextResponse(str(exc), status_code=400)

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id == 3:
        raise HTTPException(status_code=418, detail="Nope! I don't like 3.")
    return {"item_id": item_id}
```
当请求/items/foo时，由于item_id要求是int，foo是str，则抛出异常，返回结果不是默认的下面内容：
```
{
    "detail": [
        {
            "loc": [
                "path",
                "item_id"
            ],
            "msg": "value is not a valid integer",
            "type": "type_error.integer"
        }
    ]
}
```
而是变成了
```
1 validation error
path -> item_id
  value is not a valid integer (type=type_error.integer)
```

RequestValidationError 包含其接收到的无效数据请求的 body ,开发时，可以用这个请求体生成日志、调试错误，并返回给用户
```python
from fastapi import FastAPI, Request, status
from fastapi.encoders import jsonable_encoder
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
from pydantic import BaseModel

app = FastAPI()

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content=jsonable_encoder({"detail": exc.errors(), "body": exc.body}),
    )

class Item(BaseModel):
    title: str
    size: int

@app.post("/items/")
async def create_item(item: Item):
    return item
```
如果传递了不合法的数据
```
{
  "title": "towel",
  "size": "XL"
}
```
那么我们收到的返回结果如下：
```
{
  "detail": [
    {
      "loc": [
        "body",
        "item",
        "size"
      ],
      "msg": "value is not a valid integer",
      "type": "type_error.integer"
    }
  ],
  "body": {
    "title": "towel",
    "size": "XL"
  }
}
```
FastAPI HTTPException vs Starlette HTTPException

FastAPI 也提供了自有的 HTTPException。

FastAPI 的 HTTPException 继承自 Starlette 的 HTTPException 错误类。

它们之间的唯一区别是，FastAPI 的 HTTPException 可以在响应中添加响应头。

OAuth 2.0 等安全工具需要在内部调用这些响应头。

因此你可以继续像平常一样在代码中触发 FastAPI 的 HTTPException 。

但注册异常处理器时，应该注册到来自 Starlette 的 HTTPException。

这样做是为了，当 Starlette 的内部代码、扩展或插件触发 Starlette HTTPException 时，处理程序能够捕获、并处理此异常。

注意，本例代码中同时使用了这两个 HTTPException，此时，要把 Starlette 的 HTTPException 命名为 StarletteHTTPException：



## 覆盖 HTTPException 错误处理器
例如，只为错误返回纯文本响应，而不是返回 JSON 格式的内容：
```python
from fastapi import FastAPI, HTTPException
from fastapi.responses import PlainTextResponse
from starlette.exceptions import HTTPException as StarletteHTTPException

app = FastAPI()


@app.exception_handler(StarletteHTTPException)
async def http_exception_handler(request, exc):
    return PlainTextResponse(str(exc.detail), status_code=exc.status_code)

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id == 3:
        raise HTTPException(status_code=418, detail="Nope! I don't like 3.")
return {"item_id": item_id}
```
如果请求 /items/3，这时候返回的错误信息为：
```
Nope! I don't like 3.
```
如果没有自定义异常处理器http_exception_handler，返回的错误信息为：
```
{
    "detail": "Nope! I don't like 3."
}
```


# 复用 FastAPI 异常处理器
FastAPI 支持先对异常进行某些处理，然后再使用 FastAPI 中处理该异常的默认异常处理器。从 fastapi.exception_handlers 中导入要复用的默认异常处理器：
```python
from fastapi import FastAPI, HTTPException
# 从fastapi.exception_handlers导入缺省异常处理器
from fastapi.exception_handlers import (
    http_exception_handler,
    request_validation_exception_handler,
)
from fastapi.exceptions import RequestValidationError
from starlette.exceptions import HTTPException as StarletteHTTPException

app = FastAPI()

@app.exception_handler(StarletteHTTPException)
async def custom_http_exception_handler(request, exc):
    print(f"OMG! An HTTP error!: {repr(exc)}")
    #在使用缺省异常处理器之前添加了一条日志输出
    return await http_exception_handler(request, exc)

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    print(f"OMG! The client sent invalid data!: {exc}")
    #在使用缺省异常处理器之前添加了一条日志输出
    return await request_validation_exception_handler(request, exc)

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id == 3:
        raise HTTPException(status_code=418, detail="Nope! I don't like 3.")
    return {"item_id": item_id}
```

# 路径操作配置
可以将几个参数传递给**路径操作装饰器**来配置它，这些参数直接传递给路径操作装饰器，而不是路径操作函数
## 响应状态码
可以定义status_code要在路径操作的响应中使用的 (HTTP) 。可以直接传递int代码，例如404，也可以使用快捷常量status：
```python
from typing import Optional, Set
from fastapi import FastAPI, status
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None
    tags: Set[str] = []


@app.post("/items/", response_model=Item, status_code=status.HTTP_201_CREATED)
async def create_item(item: Item):
    return item
```
## 标签
```python
from typing import Optional, Set
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None
    tags: Set[str] = []

@app.post("/items/", response_model=Item, tags=["items"])
async def create_item(item: Item):
    return item

@app.get("/items/", tags=["items"])
async def read_items():
    return [{"name": "Foo", "price": 42}]


@app.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "johndoe"}]
```

## 总结和描述(summary和description)
```python
from typing import Optional, Set
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None
    tags: Set[str] = []

@app.post(
    "/items/",
    response_model=Item,
    summary="Create an item",
    description="Create an item with all the information, name, description, price, tax and a set of unique tags",
)
async def create_item(item: Item):
    return item
```
## 文档字符串的描述
由于描述往往很长并且涵盖多行，可以在函数docstring 中声明路径操作描述，FastAPI将从那里读取它。
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None
    tags: Set[str] = []


@app.post("/items/", response_model=Item, summary="Create an item")
async def create_item(item: Item):
    """
    Create an item with all the information:

    - **name**: each item must have a name
    - **description**: a long description
    - **price**: required
    - **tax**: if the item doesn't have tax, you can omit this
    - **tags**: a set of unique tag strings for this item
    """
    return item
```

## 响应说明(response_description)
```python
from typing import Optional, Set

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None
    tags: Set[str] = []


@app.post(
    "/items/",
    response_model=Item,
    summary="Create an item",
    response_description="The created item",
)
async def create_item(item: Item):
    """
    Create an item with all the information:

    - **name**: each item must have a name
    - **description**: a long description
    - **price**: required
    - **tax**: if the item doesn't have tax, you can omit this
    - **tags**: a set of unique tag strings for this item
    """
    return item
```
response_description特指响应，description泛指路径操作

## 弃用路径操作(deprecated)
如果要弃用某个路径操作，但是并不删除它，可以使用deprecated

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/", tags=["items"])
async def read_items():
    return [{"name": "Foo", "price": 42}]


@app.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "johndoe"}]


@app.get("/elements/", tags=["items"], deprecated=True)
async def read_elements():
    return [{"item_id": "Foo"}]
```

# JSON兼容编码器
在进行数据存储或者传输的时候，有时候需要把数据(比如Pydantic模型)转换成JSON兼容的格式(如dict、list等)。

FastAPI提供了 jsonable_encoder 函数来实现
```python
from datetime import datetime
from typing import Optional

from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel

fake_db = {}

class Item(BaseModel):
    title: str
    timestamp: datetime
    description: Optional[str] = None

app = FastAPI()

@app.put("/items/{id}")
def update_item(id: str, item: Item):
    json_compatible_item_data = jsonable_encoder(item)
    fake_db[id] = json_compatible_item_data
```
假设fake_db只接受JSON 兼容数据的数据库，在此例中，将Pydantic模型转换为一个字典，并将这个datetime转换为一个字符串

# 更新数据
## 用PUT更新数据
更新数据请用`HTTP PUT`操作,PUT用于接收替换现有数据的数据
```python
from typing import List, Optional

from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: Optional[str] = None
    description: Optional[str] = None
    price: Optional[float] = None
    tax: float = 10.5
    tags: List[str] = []

items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}

@app.get("/items/{item_id}", response_model=Item)
async def read_item(item_id: str):
    return items[item_id]

@app.put("/items/{item_id}", response_model=Item)
async def update_item(item_id: str, item: Item):
    update_item_encoded = jsonable_encoder(item)
    items[item_id] = update_item_encoded
    return update_item_encoded
```
## 用PATCH进行部分更新
HTTP PATCH 操作用于更新 部分 数据。即，只发送要更新的数据，其余数据保持不变。

PATCH 没有 PUT 知名，也怎么不常用。很多人甚至只用 PUT 实现部分更新。

FastAPI 对此没有任何限制，可以随意互换使用这两种操作。

```python
from typing import List, Optional

from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: Optional[str] = None
    description: Optional[str] = None
    price: Optional[float] = None
    tax: float = 10.5
    tags: List[str] = []


items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}


@app.get("/items/{item_id}", response_model=Item)
async def read_item(item_id: str):
    return items[item_id]


@app.patch("/items/{item_id}", response_model=Item)
async def update_item(item_id: str, item: Item):

    stored_item_data = items[item_id]
    stored_item_model = Item(**stored_item_data)
    # 使用exclude_unset参数，则返回时，只包含创建 item 模型时显式设置的数据，不包括默认值
    # 只更新用户设置过的值，不用模型中的默认值覆盖已存储过的值
    update_data = item.dict(exclude_unset=True)
    # 为已存储的模型创建副本，用接收的数据更新其属性 （使用 update 参数）
    updated_item = stored_item_model.copy(update=update_data)
    # 把模型副本转换为可存入数据库的形式（比如，使用 jsonable_encoder）
    items[item_id] = jsonable_encoder(updated_item)
    return updated_item
```

# 依赖项


下面举一个简单的例子，用于了解依赖注入系统是怎么工作的。


```python
from typing import Optional
from fastapi import Depends, FastAPI

app = FastAPI()

#依赖项，依赖项函数的形式和结构与路径操作函数一样
# 可以把依赖项当作没有「装饰器」（即，没有 @app.get("/some-path") ）的路径操作函数
# 依赖项可以返回各种内容
async def common_parameters(q: Optional[str] = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}

# 与在路径操作函数参数中使用 Body、Query 的方式相同，声明依赖项需要使用 Depends 和一个新的参数
# 这里只能传给 Depends 一个参数
# 且该参数必须是可调用对象，比如函数。该函数接收的参数和路径操作函数的参数一样
@app.get("/items/")
async def read_items(commons: dict = Depends(common_parameters)):
    return commons

@app.get("/users/")
async def read_users(commons: dict = Depends(common_parameters)):
    return commons
```
接收到新的请求时，FastAPI 执行如下操作：
* 用正确的参数调用依赖项函数（「可依赖项」）
* 获取依赖项函数返回的结果
* 把依赖项函数返回的结果赋值给路径操作函数的参数

这样，只编写一次代码，FastAPI 就可以为多个路径操作共享这段代码

## 类作为依赖项
依赖项并不一定非要是函数，只要是**可调用**的即可，比如Python的类

可以将上述的`common_parameters`函数变成类
```python
class CommonQueryParams:
    def __init__(self, q: str = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit
```
注意，这里我们用了`__init__`方法来实现类的初始化。并且类的初始化参数与依赖项函数完全相同

使用依赖项类
```python
from typing import Optional
from fastapi import Depends, FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]

class CommonQueryParams:
    def __init__(self, q: Optional[str] = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit

@app.get("/items/")
async def read_items(commons: CommonQueryParams = Depends(CommonQueryParams)):
    response = {}
    if commons.q:
        response.update({"q": commons.q})
    items = fake_items_db[commons.skip : commons.skip + commons.limit]
    response.update({"items": items})
    return response
```
## 子依赖项
FastAPI 支持创建含子依赖项的依赖项，可以按需声明任意深度的子依赖项嵌套层级，FastAPI 负责处理解析不同深度的子依赖项。

先声明第一个依赖项函数
```python
def query_extractor(q11: str, q12: str):
    return q11 + q12
```

第二个依赖项函数
```python
def query_or_cookie_extractor(q2: str = Depends(query_extractor), last_query: str = Cookie(None)):
    if not q2:
        return last_query
    return q2
```
使用这两个依赖项函数
```python
from fastapi import Cookie, Depends, FastAPI

app = FastAPI()

def query_extractor(q11: str, q12: str):
    return q11 + q12

def query_or_cookie_extractor(q2: str = Depends(query_extractor), last_query: str = Cookie(None)):
    if not q2:
        return last_query
    return q2

@app.get("/items/")
async def read_query(query_or_default: str = Depends(query_or_cookie_extractor)):
    return {"q_or_cookie": query_or_default}
```
在路径操作函数` read_query`中，仅声明了依赖项函数`query_or_cookie_extractor`，但Fast API只要需要先调用第一个依赖项函数`query_extractor`,然后把结果传给`query_or_cookie_extractor`


依赖项的多次调用

如果某个依赖项在同一个路径操作中被声明了多次，例如，多个依赖项都有一个共同的子依赖项，那么FastAPI默认在每一次请求中只会调用这个依赖项一次，FastAPI会把这个依赖项的返回值缓存起来，然后把这个值传递给需要的依赖项，而不是在同一个请求中多次调用这个依赖项。

在有些场景下，我们并不需要缓存这个依赖项的返回值，而是需要多次调用，那么我们可以使用参数`use_cache=False`来禁止依赖项的缓存

```python
async def needy_dependency(fresh_value: str = Depends(get_value, use_cache=False)):
return {"fresh_value": fresh_value}
```
## 路径操作装饰器依赖项
有些时候，我们不需要在路径操作函数中使用依赖项的返回值，或者，有些依赖项并不返回值，但仍要执行或解析该依赖项。对这种情况，不必在声明路径操作函数的参数时使用 Depends，而是可以在路径操作装饰器中添加一个由 dependencies 组成的 list


在路径操作装饰器中添加 dependencies 参数,该参数的值是由 Depends() 组成的 list
```python
from fastapi import Depends, FastAPI, Header, HTTPException

app = FastAPI()

async def verify_token(x_token: str = Header(...)):
    if x_token != "fake-super-secret-token":
        raise HTTPException(status_code=400, detail="X-Token header invalid")

async def verify_key(x_key: str = Header(...)):
    if x_key != "fake-super-secret-key":
        raise HTTPException(status_code=400, detail="X-Key header invalid")
    return x_key

@app.get("/items/", dependencies=[Depends(verify_token), Depends(verify_key)])
async def read_items():
    return [{"item": "Foo"}, {"item": "Bar"}]
```
它们的执行方或解析方式和普通依赖项一样，但就算这些依赖项会返回值，它们的值也不会传递给路径操作函数

## 全局依赖项
有时，我们要为整个应用添加依赖项
```python
from fastapi import Depends, FastAPI, Header, HTTPException


async def verify_token(x_token: str = Header(...)):
    if x_token != "fake-super-secret-token":
        raise HTTPException(status_code=400, detail="X-Token header invalid")


async def verify_key(x_key: str = Header(...)):
    if x_key != "fake-super-secret-key":
        raise HTTPException(status_code=400, detail="X-Key header invalid")
    return x_key


app = FastAPI(dependencies=[Depends(verify_token), Depends(verify_key)])


@app.get("/items/")
async def read_items():
    return [{"item": "Portal Gun"}, {"item": "Plumbus"}]


@app.get("/users/")
async def read_users():
    return [{"username": "Rick"}, {"username": "Morty"}]
```
 在本例中，这些依赖项可以用于应用中的所有路径操作

 ##  与yield的依赖关系

 （没理解，先跳过）
 FastAPI支持依赖项在请求结束后做一些额外的工作,要实现这个功能，我们需要用yield代替return，然后其后添加一些额外的工作。

 比如，

 # 安全

## 安全简介
* OAuth2

OAuth2是一个规范，它定义了几种处理身份认证和授权的方法。它是一个相当广泛的规范，涵盖了一些复杂的使用场景。它包括了使用「第三方」进行身份认证的方法。Facebook，Google等都使用这种安全机制

* OpenID Connect

OpenID Connect 是另一个基于 OAuth2 的规范。它只是扩展了 OAuth2，并明确了一些在 OAuth2 中相对模糊的内容，以尝试使其更具互操作性。例如，Google 登录使用 OpenID Connect（底层使用OAuth2）。但是 Facebook 登录不支持 OpenID Connect。它具有自己的 OAuth2 风格。

* OpenAPI

OpenAPI（以前称为 Swagger）是用于构建 API 的开放规范（现已成为 Linux Foundation 的一部分）。FastAPI 基于 OpenAPI。

OpenAPI 定义了以下安全方案：

1）apiKey：一个特定于应用程序的密钥，可以来自：查询参数。请求头。cookie。

2）http：标准的 HTTP 身份认证系统，包括：bearer: 一个值为 Bearer 加令牌字符串的 Authorization 请求头。这是从 OAuth2 继承的。HTTP Basic 认证方式。HTTP Digest，等等。

3）oauth2：所有的 OAuth2 处理安全性的方式（称为「流程」）

4）openIdConnect：提供了一种定义如何自动发现 OAuth2 身份认证数据的方法。此自动发现机制是 OpenID Connect 规范中定义的内容。

## 