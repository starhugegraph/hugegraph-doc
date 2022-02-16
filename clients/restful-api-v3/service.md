## 4.2.服务

在 HugeGraph 中，在线计算（OLTP）、离线计算（OLAP）和存储都是作为服务（service）存在的。目前仅支持在线计算服务的动态创建。

#### 4.2.1.创建一个服务

##### 功能介绍

创建一个服务

##### URI

```
POST /graphspaces/${graphspace}/services
```


##### URI参数

|  名称   | 是否必填  | 类型  | 默认值 | 取值范围 | 说明  |
|  ----  | ----  | ----  | ----  | ----  | ---- |
| graphspace  | 是 | String  |   |   | 图空间名称  |

##### Body参数

|  名称   | 是否必填  | 类型  | 默认值  | 取值范围  | 说明  |
|  ----  | ----  | ----  | ----  | ----  | ----  |
| name  | 是 | String  |   | 小写字母、数字和下划线组成，首字符必须是小写字母，长度不超过48  |  service 的名字 |
| type  | 是 | String  |   | OLTP, OLAP, STORAGE  |  目前只支持OLTP |
| deployment_type  | 是 | String  |   | K8S、MANUAL  |  service 的部署类型，K8S 指通过K8S集群启动服务，MANUAL 指通过手动部署的方式启动服务 |
| count  | 是 | Int |   | > 0  |  HugeGraphServer 的数目 |
| cpu_limit  | 是 | Int  |   | > 0  |  HugeGraphServer 的 CPU 核数 |
| memory_limit  | 是 | Int  |   | > 0  |  HugeGraphServer 的内存大小，单位 GB |
| storage_limit  | 是 | Int  |   | > 0  |  HugeGraphServer 的存储大小，单位 GB |
| route_type  | 是 | String  |   | ClusterIP，LoadBalancer，NodePort  |  只有在 deployment_type 为 K8S 时需要设置 |
| port  | 是 | Int  |   |   |  只有在 deployment_type 为 K8S 时需要设置 |
| urls  | 是 | Array  |   |   |  只有在 deployment_type 为 MANUAL 时需要设置 |
| description  | 是 | String  |   |   |  service 的描述信息 |

##### Response

|  名称   | 类型 |  说明  |
|  ----  | ---|  ----  |
| name  | String | service 的名字 |
| type  | String | 服务类型 |
| port  | Int | 端口 |
| route_type  | String | 路由类型 |
| count  | Int |  HugeGraphServer 的总数目 |
| running | Int | 运行的 HugeGraphServer 的数目
| cpu_limit  | Int  |  HugeGraphServer 的 CPU 核数 |
| memory_limit  | Int  |  HugeGraphServer 的内存大小，单位 GB |
| storage_limit  | Int  |  HugeGraphServer 的存储大小，单位 GB |
| deployment_type  | String  |  service 的部署类型 |
| urls | Array | service 地址列表 |
| pd_service_id  | String |  pd service id |
| description  | String |  service 的描述信息 |

##### 使用示例

###### Method & Url

```
POST http://localhost:8080/graphspaces/gs1/services
```

###### Request Body

```json
{
  "name": "sv1",
  "type": "OLTP",
  "port": 0,
  "count": 2,
  "cpu_limit": 4,
  "memory_limit": 6,
  "storage_limit": 4096,
  "deployment_type": "MANUAL",
  "urls": ["http://xx.xx.xx.xx:8080", "http://xx.xx.xx.xx:8080" ],
  "description": "test oltp service"
}
```

###### Response Status

```json
201
```

###### Response Body

```json
{
  "name": "sv1",
  "type": "OLTP",
  "deployment_type": "MANUAL",
  "description": "test oltp service",
  "count": 2,
  "running": 0,
  "cpu_limit": 4,
  "memory_limit": 6,
  "storage_limit": 4096,
  "route_type": null,
  "port": 0,
  "urls": [
    "http://10.27.154.54:8080",
    "http://10.27.154.55:8080"
  ],
  "pd_service_id": "895e0c269e07130200f26bd1a747d599"
}
```

#### 4.2.2.列出某个图空间的所有服务

##### 功能介绍

列出某个图空间的所有服务

##### URI

```
GET /graphspaces/${graphspace}/services
```

##### URI参数

|  名称   | 是否必填  | 类型  | 默认值 | 取值范围 | 说明  |
|  ----  | ----  | ----  | ----  | ----  | ---- |
| graphspace  | 是 | String  |   |   | 图空间名称  |

##### Body参数

无

##### Response

|  名称   | 类型 |  说明  |
|  ----  | ---|  ----  |
| services  |Array| service 的名字列表 |

##### 使用示例


###### Method & Url

```
GET http://localhost:8080/graphspaces/gs1/services
```


###### Request Body

无

###### Response Status

```json
200
```

###### Response Body

```json
{
    "services":["sv1"]
}
```

#### 4.2.3.查看某个服务

##### 功能介绍

查看某个服务

##### URI

```
GET /graphspaces/${graphspace}/services/${service}
```

##### URI参数

|  名称   | 是否必填  | 类型  | 默认值 | 取值范围 | 说明  |
|  ----  | ----  | ----  | ----  | ----  | ---- |
| graphspace  | 是 | String  |   |   | 图空间名称  |
| service  | 是 | String  |   |   | 服务名称  |

##### Body参数

无

##### Response

|  名称   | 类型 |  说明  |
|  ----  | ---|  ----  |
| name  | String | service 的名字 |
| type  | String | 服务类型 |
| port  | Int | 端口 |
| route_type  | String | 路由类型 |
| count  | Int |  HugeGraphServer 的总数目 |
| running | Int | 运行的 HugeGraphServer 的数目
| cpu_limit  | Int  |  HugeGraphServer 的 CPU 核数 |
| memory_limit  | Int  |  HugeGraphServer 的内存大小，单位 GB |
| storage_limit  | Int  |  HugeGraphServer 的存储大小，单位 GB |
| deployment_type  | String  |  service 的部署类型 |
| urls | Array | service 地址列表 |
| pd_service_id  | String |  pd service id |
| description  | String |  service 的描述信息 |

##### 使用示例

###### Method & Url

```
GET http://127.0.0.1:8080/graphspaces/gs1/services/sv1
```

###### Request Body

无

###### Response Status

```json
200
```

###### Response Body

```json
{
  "name": "sv1",
  "type": "OLTP",
  "deployment_type": "MANUAL",
  "description": "test oltp service",
  "count": 2,
  "running": 0,
  "cpu_limit": 4,
  "memory_limit": 6,
  "storage_limit": 4096,
  "route_type": null,
  "port": 0,
  "urls": [
    "http://10.27.154.54:8080",
    "http://10.27.154.55:8080"
  ],
  "pd_service_id": "895e0c269e07130200f26bd1a747d599"
}
```

#### 4.2.4.删除某个服务

##### 功能介绍

删除某个服务

##### URI

```
DELETE /graphspaces/${graphspace}/services/${service}
```


##### URI参数

|  名称   | 是否必填  | 类型  | 默认值 | 取值范围 | 说明  |
|  ----  | ----  | ----  | ----  | ----  | ---- |
| graphspace  | 是 | String  |   |   | 图空间名称  |
| service  | 是 | String  |   |   | 服务名称  |

##### Body参数

无

##### Response

无

##### 使用示例

###### Method & Url

```
DELETE http://localhost:8080/graphspaces/gs1/services/sv1
```


###### Request Body

无

###### Response Status

```json
204
```

> 注意：删除图空间，会导致图空间的全部资源被释放。
