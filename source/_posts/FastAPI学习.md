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