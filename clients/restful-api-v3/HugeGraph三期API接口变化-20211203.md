# HugeGraph三期API接口变化

> 注意：三期设计和开发正在进行中，此文档只描述当前情况，后续可能还会有所调整。

HugeGraph三期有一些API发生变化，但会尽可能与二期API环境保持兼容。保持兼容的方式包括：

1. 通过Jersey的filter将原有的schema和graph操作重定向到默认图空间
2. 改造hugegraph-client，尽可能保证基于hugegraph-client实现的二期业务应用不用改动

## 1 API整体变化

为了支持多租户，API在三期出现了较大变动，主要包括：

1. 增加多租户API
2. 改造权限API
3. 改造全部schema和graphAPI
4. 改造动态创建图API



## 2 增加多租户API

### 2.1 增加graphspace相关API

HugeGraph里的租户的概念叫做图空间（graphspace），以图空间为单位分配CPU、Memory和存储资源。图空间中可以创建多个服务（service）和多个图。

#### 2.1.1 创建graphspace

URL:

```
POST http://localhost:8080/graphspaces
```

Request body:

```json
{
  "name": "gs1",
  "description": "1st graph space",
  "cpu_limit": 1000,
  "memory_limit": 1024,
  "storage_limit": 1000,
  "oltp_namespace": "hugegraph-server",
  "olap_namespace": "hugegraph-server",
  "storage_namespace": "hugegraph-server",
  "max_graph_number": 100,
  "max_role_number": 10,
  "configs": {}
}
```

Response status:

```json
201
```

Response body:

```json
{
    "name": "gs1",
    "description": "1st graph space",
    "cpu_limit": 1000,
    "memory_limit": 1024,
    "storage_limit": 1000,
    "oltp_namespace": "hugegraph-server",
    "olap_namespace": "hugegraph-server",
    "storage_namespace": "hugegraph-server",
    "max_graph_number": 100,
    "max_role_number": 10,
    "cpu_used": 0,
    "memory_used": 0,
    "storage_used": 0,
    "graph_number_used": 0,
    "role_number_used": 0
}
```



#### 2.1.2 列出全部graphspace

URL

```
GET http://localhost:8080/graphspaces
```

Response status:

```json
200
```

Response body:

```json
{
    "graphSpaces":[
        "gs1",
        "DEFAULT"
    ]
}
```



#### 2.1.3 查看某个graphspace

URL:

```
GET http://localhost:8080/graphspaces/gs1
```

Response status:

```json
200
```

Response body:

```json
{
    "name": "gs1",
    "description": "1st graph space",
    "cpu_limit": 1000,
    "memory_limit": 1024,
    "storage_limit": 1000,
    "oltp_namespace": "hugegraph-server",
    "olap_namespace": "hugegraph-server",
    "storage_namespace": "hugegraph-server",
    "max_graph_number": 100,
    "max_role_number": 10,
    "cpu_used": 0,
    "memory_used": 0,
    "storage_used": 0,
    "graph_number_used": 0,
    "role_number_used": 0
}
```



#### 2.1.4 更改某个graphspace

URL：

```
PUT http://localhost:8080/graphspaces/gs1
```

Request body:

```json
{
  "description": "updated 1st graph space",
  "cpu_limit": 10000,
  "memory_limit": 10240,
  "storage_limit": 10000,
  "max_graph_number": 1000,
  "max_role_number": 100,
  "configs": {}
}
```



Response status:

```json
200
```



Response body:

```json
{
    "name": "gs1",
    "description": "updated 1st graph space",
    "cpu_limit": 10000,
    "memory_limit": 10240,
    "storage_limit": 10000,
    "oltp_namespace": "hugegraph-server",
    "olap_namespace": "hugegraph-server",
    "storage_namespace": "hugegraph-server",
    "max_graph_number": 1000,
    "max_role_number": 100,
    "cpu_used": 0,
    "memory_used": 0,
    "storage_used": 0,
    "graph_number_used": 0,
    "role_number_used": 0
}
```



#### 2.1.5 删除某个graphspace

URL:

```
DELETE http://localhost:8080/graphspaces/gs1?confirm_message=I%27m+sure+to+drop+the+graph+space
```

Response status:

```json
204
```



### 2.2 增加service相关API

增加服务（service）接口，服务分为OLTP、OLAP、STORAGE三种。以OLTP为例，一个OLTP service，可以对应1-n个HugeGraphServer。

#### 2.2.1 创建service

URL:

```
POST http://localhost:8080/graphspaces/gs1/services
```

Request body:

```json
{
  "name": "sv1",
  "type": "OLTP",
  "description": "test oltp service",
  "count": 1,
  "cpu_limit": 2,
  "memory_limit": 4,
  "storage_limit": 10
}
```

Response status:

```json
201
```

Response body:

```json
{
    "name": "sv1",
    "type": "OLTP",
    "description": "test oltp service",
    "count": 1,
    "cpu_limit": 2,
    "memory_limit": 4,
    "storage_limit": 10,
    "urls": [
        "10.244.4.92:8080"
    ]
}
```



#### 2.2.2 列出全部service

URL

```
GET http://localhost:8080/graphspaces/gs1/services
```

Response status:

```json
200
```

Response body:

```json
{
    "services":[
        "sv2",
        "sv1"
    ]
}
```



#### 2.2.3 查看某个service

URL:

```
GET http://localhost:8080/graphspaces/gs1/services/sv1
```

Response status:

```json
200
```

Response body:

```json
{
    "name": "sv1",
    "type": "OLTP",
    "description": "test oltp service",
    "count": 1,
    "cpu_limit": 2,
    "memory_limit": 4,
    "storage_limit": 10,
    "urls": [
        "10.244.4.92:8080"
    ]
}
```

#### 2.2.4 删除某个service

URL:

```
DELETE http://localhost:8080/graphspaces/gs1/services/sv1?confirm_message=I%27m+sure+to+drop+the+service
```

Response status:

```json
204
```



### 2.3 改造其他API

由于增加了graphspace，所以其他的API也要相应的改动，具体内容参见第3、第4和第5部分



## 3 改造权限API

### 3.1 user相关的API提升到全局

- request body和response不变

- URL变化提升到全局，以创建用户API为例：

```
# 旧版
POST http://localhost:8080/graphs/auth/users
```

```
# 新版
POST http://localhost:8080/auth/users
```

### 3.2 其他权限API path增加graphspace信息

- request body和response不变

- URL增加graphspace信息，以创建用户组为例：

```
# 旧版
POST http://localhost:8080/graphs/auth/groups
```

```
# 新版
POST http://localhost:8080/graphspaces/{graphspace}/auth/groups
```



## 4 改造全部schema API和graph API

schema API包括:

- PropertyKey
- VertexLabel
- EdgeLabel
- IndexLabel

graph API包括

- Vertex
- Edge

API改动如下：

- Request body和Response status和Response body不变

- URL增加graphspace信息，以创建VertexLabel为例：

```
# 旧版
POST http://localhost:8080/graphs/hugegraph/schema/vertexlabels
```

```
# 新版
POST http://localhost:8080/graphspaces/gs1/graphs/hugegraph/schema/vertexlabels
```



## 5 改造动态创建图API

### 5.1 增加schema config接口

#### 5.1.1 创建schemaconfig

URL:

```
POST http://localhost:8080/graphspaces/gs1/schemaconfigs
```

Request body:

```json
{
  "name": "config1",
  "schema": "
     schema.propertyKey('name').asText().ifNotExist().create();
     schema.propertyKey('age').asInt().ifNotExist().create();
     schema.propertyKey('city').asText().ifNotExist().create();
     schema.propertyKey('date').asText().ifNotExist().create();
     schema.propertyKey('price').asDouble().ifNotExist().create();
     schema.vertexLabel('person').properties('name', 'age', 'city').primaryKeys('name').ifNotExist().create();
     schema.vertexLabel('software').properties('name', 'price').primaryKeys('name').ifNotExist().create();
     schema.edgeLabel('knows').sourceLabel('person').targetLabel('person').ifNotExist().create();
     schema.edgeLabel("created").sourceLabel('person').targetLabel('software').ifNotExist().create();
  "
}
```

Response status:

```json
201
```

Response body:

```json
{
  "name": "config1",
  "schema": "
     schema.propertyKey('name').asText().ifNotExist().create();
     schema.propertyKey('age').asInt().ifNotExist().create();
     schema.propertyKey('city').asText().ifNotExist().create();
     schema.propertyKey('date').asText().ifNotExist().create();
     schema.propertyKey('price').asDouble().ifNotExist().create();
     schema.vertexLabel('person').properties('name', 'age', 'city').primaryKeys('name').ifNotExist().create();
     schema.vertexLabel('software').properties('name', 'price').primaryKeys('name').ifNotExist().create();
     schema.edgeLabel('knows').sourceLabel('person').targetLabel('person').ifNotExist().create();
     schema.edgeLabel("created").sourceLabel('person').targetLabel('software').ifNotExist().create();
  "
}
```



#### 5.1.2 列出全部schema config

URL

```
GET http://localhost:8080/graphspaces/gs1/schemaconfigs
```

Response status:

```json
200
```

Response body:

```json
{
    "services":[
        "config1",
        "config2"
    ]
}
```



#### 5.1.3 查看某个schemaconfig

URL:

```
GET http://localhost:8080/graphspaces/gs1/schemaconfigs/config1
```

Response status:

```json
200
```

Response body:

```json
{
  "name": "config1",
  "schema": "
     schema.propertyKey('name').asText().ifNotExist().create();
     schema.propertyKey('age').asInt().ifNotExist().create();
     schema.propertyKey('city').asText().ifNotExist().create();
     schema.propertyKey('date').asText().ifNotExist().create();
     schema.propertyKey('price').asDouble().ifNotExist().create();
     schema.vertexLabel('person').properties('name', 'age', 'city').primaryKeys('name').ifNotExist().create();
     schema.vertexLabel('software').properties('name', 'price').primaryKeys('name').ifNotExist().create();
     schema.edgeLabel('knows').sourceLabel('person').targetLabel('person').ifNotExist().create();
     schema.edgeLabel("created").sourceLabel('person').targetLabel('software').ifNotExist().create();
  "
}
```

#### 5.1.4 删除某个schemaconfig

URL:

```
DELETE http://localhost:8080/graphspaces/gs1/schemaconfigs/config1?confirm_message=I%27m+sure+to+drop+the+config
```

Response status:

```json
204
```



### 5.2 动态创建图API

URL:

```
POST http://localhost:8080/graphspaces/gs1/graphs/
```

Request body:

```json
{
  "name": "hugegraph", 
  "auth": false,
  "schema.init_template": "config1"
  "store": "hugegraph",
  "store.config1": "xxx",
  ...
  "store.configx": "xxx",
}
```

参数说明：

- auth控制是否开启权限，选填，默认为false

- schema指定要提前加载的schema config，选填，默认为空

- store指定后端存储，必填，指定三期分布式存储

- stor.config后端存储配置，选填



Response status:

```json
201
```

Response body:

```json
{
  "name": "hugegraph", 
  "auth": false,
  "store": "hugegraph",
  "schema.init_template": "schema_name"
  "store.config1": "xxx",
  ...
  "store.configx": "xxx",
}
```

### 5.3 清空图API

URL

```
PUT http://localhost:8080/graphspaces/gs1/graphs/hugegraph
```

Request body:

```json
{
  "action": "clear",
  "clear_schema": false,
}
```

- clear_schema是否清空元数据，可选项，默认为false，表示只清空顶点、边和索引信息，保留schema；为true时，清空全部数据



Response status：

```json
200
```

