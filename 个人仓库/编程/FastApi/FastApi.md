## 第一部分：基础知识
### 一、HTTP协议
#### 1、协议组成
![[Pasted image 20250304195543.png]]
1. 请求首行
	1. 请求方法
	2. 请求资源路径
	3. 协议版本
2. 请求头
	1. Host(必须的)：主机名
	2. User-Agent:发出亲求的客户端
	3. Accept：可接受内容的类型，如果是*/* 标识所有类型
	4. Accept-Encoding：表示接受的数据压缩格式
	5. Connection：keep-alive，是否长连接
	6. Content-Type：请求体数据类型
	7. Content-Length：长度
3. 请求体(POST、PUT、PATCH)，GET没有请求体
	1. 空格
	2. Json数据
4. 响应首行
	1. 协议版本
	2. 返回的状态码
5. 响应头
	1. Content-Type：响应体数据类型
	2. date：响应时间戳
	3. 响应体内容
6. 响应体
#### 2、Url的组成
https://www.baidu.com/s?wd=yuan
1. 协议：https
2. 地址(域名)：www.baidu.com
3. 路径(?之前)：s
4. 参数：wd=yuan
### 二、RestFul规范
1. POST:增加
2. GET:获取
3. PUT:修改
4. DELETE:删除
### 三、所需要的包
```cmd
pip install fastapi
pip install uvicorn  //ASGI服务器,作为前端与后端的中间层服务
```
### 四、创建一个简单的API并启动项目测试
#### 1、使用控制台启动
```python
from fastapi import FastAPI

app = FastAPI()
#通过这个来标识此方法的请求路径为跟路径
@app.get("/")
async def root():
return {"message": "Hello World"}
```
```cmd
#QuickStart为当前的py文件名
uvicorn QuickStart:app --reload
```

#### 2、在程序内部调用uvicorn的接口
```python
from fastapi import FastAPI
import uvicorn
app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello Worlds"}
if __name__ == "__main__":
    uvicorn.run("QuickStart:app", host="0.0.0.0", port=8000, log_level="debug", reload=True) # 使用 uvicorn 运行 FastAPI 应用
```
## 第二部分：具体实现、详细知识
#### 1、路径操作装饰器
- path:请求路径
- tags:API文档中的分组标签，一般把一个类型的api设置为同样的tags
- description:API文档中的描述，展开后显示
- summary：摘要，展开前显示
- deprecated：标记是否弃用
对于GET、POST、PUT、DELETE这些常用的参数都通用

#### 2、路由分发(相当于将API分组成类似于asp.net core中的控制器)
在控制器目录下创建对应的控制器:如user和order
![[Pasted image 20250305200144.png]]
```python
#控制器user中的内容
from fastapi import APIRouter
user_control = APIRouter()
@user_control.get("/getUserByID")
def getOrderByID(user_id: int):
    return {"user_id": user_id}


#控制器order中的内容
from fastapi import APIRouter
order_control = APIRouter()
@order_control.get("/getOrderByID")
def getOrderByID(order_id: int):
    return {"order_id": order_id}



#主程序中的内容
from fastapi import FastAPI
import uvicorn
#当API分组管理之后，在main.py中引入分组进行使用
from Controls.user import user_control
from Controls.order import order_control
app = FastAPI()
# 将user_control和order_control注册到app中，使用prefix指定路由前缀，使用tags指定路由标签
app.include_router(user_control,prefix="/user",tags=["用户管理"])
app.include_router(order_control,prefix="/order",tags=["订单管理"])
if __name__ == "__main__":
    uvicorn.run("main:app", host="127.0.0.1", port=8000, log_level="debug", reload=True)
```


#### 3、路径参数与查询参数
```python
from fastapi import APIRouter
order_control = APIRouter()

#这中方式为查询参数
@order_control.get("/getOrderByID")
def getOrderByID(order_id: int):
    return {"order_id": order_id}
对应的请求体为：
curl -X 'GET' \
  'http://127.0.0.1:8000/order/getOrderByID?order_id=111' \
  -H 'accept: application/json'

#这种方式为路径参数
@order_control.get("/getOrderNameByID/{order_id}")
def getOrderNameByID(order_id: int):
    return {"order_name": "订单名称"}
对应的请求体为：
curl -X 'GET' \
  'http://127.0.0.1:8000/order/getOrderNameByID/111' \
  -H 'accept: application/json'
```
#### 4、请求体参数的校验:pydantic、field_validator自定义验证
```python
#在数据模型中使用如下库完成数据模型的定义，且有类型检测功能，如果传入的函数无法转换为目标类型则返回错误信息至前端
from pydantic import BaseModel，Field,field_validator
from typing import Optional
class User(BaseModel):
	#Field可以用来做一写参数的校验
    id: Field(default=0,gt=0,lt=1000000)
    #也可以使用Optional来限制类型为str或者None，且可以和Field共同使用
    name: Optional[str] = Field(default="",max_length=10)
    age: int
class Order(BaseModel):
    id: int
    name: str
    #自定义验证器，需要引入field_validator
    @field_validator("name")
    def name_validator(cls,value):#value是name的值 cls是类本身
        assert value.isalpha(),"name must be alphabetic"
        if len(value) > 10:
            raise ValueError("name must be less than 10 characters")
        return value
```
#### 5、Form表单数据
```shell
依赖于python-multipart组件
pip install python-multipart
```

```python
#引入Form
from fastapi import APIRouter,Form
from modul import Car
car_control = APIRouter()
@car_control.post("/getCarByID")
#在这里规定数据体类型为Form
def getCarByID(car:Car = Form(...)):
    return {"car_id": car}
```
#### 6、文件上传
```python
#小文件上传需要使用File，大文件上传需要使用UploadFile

from fastapi import APIRouter,File,UploadFile

#路径相关处理库

import os

#文件拷贝库

import shutil

file_control = APIRouter()

@file_control.post("/upload_large")

#上传大文件使用的方式,文件将会分块上传，使用流式处理

def upload(file:UploadFile = File(...)):

    #文件保存

    path = os.path.join("apps","images",file.filename)

    #with类似于using

    with open(path,"wb") as f:

        #file.file是上传的文件流，当文件较大的时候使用流式拷贝

        shutil.copyfileobj(file.file,f)

    return {"filename": file.filename,"fileSize": file.size}

#上传小文件使用的方式,文件将会一次性加载到内存中

@file_control.post("/upload_small")

def upload_small(file:bytes = File(...), filename: str = None):

    path = os.path.join("apps", "images", filename)

    #文件保存

    with open(path,"wb") as f:

        f.write(file)

        print(f"文件保存成功: {path}")

    return {"filename": filename, "fileSize": len(file)}

#上传多个文件使用的方式

@file_control.post("/upload_multiple")

def upload_multiple(files:list[UploadFile] = File(...)):

    #文件保存

    for file in files:

        path = os.path.join("apps","images",file.filename)

        with open(path,"wb") as f:

            shutil.copyfileobj(file.file,f)

    return {"filenames": [file.filename for file in files]}
```
#### 7、静态文件请求
```python
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles
import uvicorn
#当API分组管理之后，在main.py中引入分组进行使用
from apps.Controls.user import user_control
from apps.Controls.order import order_control
from apps.Controls.car import car_control
from apps.Controls.file import file_control
from apps.Controls.requestTest import requestTest_control
app = FastAPI()
#用于为静态资源的外部访问提供一个路径
app.mount("/sourceFile", StaticFiles(directory="statics"))
app.mount("/images", StaticFiles(directory="images"), name="images")
# 将user_control和order_control注册到app中，使用prefix指定路由前缀，使用tags指定路由标签
app.include_router(user_control,prefix="/user",tags=["用户管理"])

app.include_router(order_control,prefix="/order",tags=["订单管理"])

app.include_router(car_control,prefix="/car",tags=["车辆管理"])

app.include_router(file_control,prefix="/file",tags=["文件管理"])

app.include_router(requestTest_control,prefix="/requestTest",tags=["HTTP请求数据获取"])

if __name__ == "__main__":

    uvicorn.run("main:app", host="127.0.0.1", port=8000, log_level="debug", reload=True)
```
然后在浏览器中就可以使用
![[Pasted image 20250306184312.png]]

#### 8、响应模板参数：用于限定响应的内容和格式
```python
from fastapi import APIRouter
from apps.Controls.modul import ResponseValueIn,ResponseValueOut
response_control = APIRouter()
#使用response_model来对响应数据进行过滤
@response_control.post("/response",description="响应数据模板",response_model=ResponseValueOut)
def CreateResponseValue(valueIn:ResponseValueIn):
    return valueIn


#定义的亲求和响应模型
class ResponseValueIn(BaseModel):
    code: int
    password: str
    message: str
    data: dict
class ResponseValueOut(BaseModel):
    code: int
    message: str
    data: dict
#同时可以使用下面的模板来进行响应限制
```

响应模板介绍：

| 参数                                | 功能                     | 说明                                  |
| --------------------------------- | ---------------------- | ----------------------------------- |
| `response_model_include`          | 只返回指定字段                | 过滤返回的字段，只包括指定的字段。                   |
| `response_model_exclude`          | 排除指定字段                 | 从响应中排除指定字段。                         |
| `response_model_exclude_unset`    | 排除未设置的字段               | 排除未显式设置的字段，保留那些在响应中已经设置的字段。         |
| `response_model_exclude_defaults` | 排除默认值字段                | 排除值为默认值的字段。                         |
| `response_model_exclude_none`     | 排除值为 `None` 的字段        | 排除值为 `None` 的字段。                    |
| `response_model_by_alias`         | 使用别名字段名序列化响应数据         | 如果模型中使用别名（`alias`），则使用别名字段名进行序列化。   |
| `include_in_schema`               | 是否将路径操作包含在 OpenAPI 文档中 | 设置为 `False` 会将该路径操作从 OpenAPI 文档中移除。 |

## 第三部分：ORM
```shell
#依赖的ORM库及相关依赖
pip install tortoise-orm
pip install aiomysql

#安装对应的ORM迁移工具机相关依赖
pip install aerich
pip install tomli_w tomlkit
```

```python
"""

模型类

"""

from tortoise import models

from tortoise import fields

  
  

#学生模型

class Student(models.Model):

    id = fields.IntField(pk=True)#设定主键

    name = fields.CharField(max_length=255)

    email = fields.CharField(max_length=255)

    password = fields.CharField(max_length=255)

    created_at = fields.DatetimeField(auto_now_add=True)

    updated_at = fields.DatetimeField(auto_now=True)

  

    # 关联班级,班级对于学生是一对多的关系

    class_id = fields.ForeignKeyField('models.Class', related_name='students')

  

    # 关联课程,课程对于学生是多对多的关系

    courses = fields.ManyToManyField('models.Course', related_name='students')

  

#班级模型

class Class(models.Model):

    id = fields.IntField(pk=True)

    name = fields.CharField(max_length=255)

    created_at = fields.DatetimeField(auto_now_add=True)

    updated_at = fields.DatetimeField(auto_now=True)

  

#课程模型

class Course(models.Model):

    id = fields.IntField(pk=True)

    name = fields.CharField(max_length=255)

    created_at = fields.DatetimeField(auto_now_add=True)

    updated_at = fields.DatetimeField(auto_now=True)

    teacher = fields.ForeignKeyField('models.Teacher', related_name='courses')

  

#教师模型

class Teacher(models.Model):

    id = fields.IntField(pk=True)

    name = fields.CharField(max_length=255)

    email = fields.CharField(max_length=255)

    password = fields.CharField(max_length=255)

    created_at = fields.DatetimeField(auto_now_add=True)

    updated_at = fields.DatetimeField(auto_now=True)
```
```python
#数据库连接配置
Tortoise_config={

        "connections": {

            "default": {

                "engine": "tortoise.backends.mysql",

                "credentials": {

                    "database": "ormtest",

                    "user": "root",

                    "password": "dj19980202..",

                    "host": "localhost",

                    "port": 3306,

                }

            }

        },

        "apps": {

            "models": {

                "models": ["models"],

                "default_connection": "default",

            },

            "aerich": {

                "models": ["aerich.models"],

                "default_connection": "default",

            }

        },

        "use_tz": False,

        "timezone": "Asia/Shanghai"

    }
```

```sql
#进入mysql数据库页面
mysql -uroot -p    #输入密码
#查看当前已经创建的数据库
show databases;
#创建数据库
create database ORMTest charset utf8;
#查看创建数据库使用的语法
show create database ORMTest;



#查看数据库中的表
show ORMTest;
show tables;
```

```shell
#使用aerich初始化数据库，相关的库在开头已经安装,setting.Tortoise_config为配置信息
aerich init -t setting.Tortoise_config
aerich init-db


#数据库的迁移
aerich migrate --name changeModels
#升级
aerich upgrade
#降级
aerich downgrade
```
