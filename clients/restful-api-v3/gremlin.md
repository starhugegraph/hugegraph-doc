## 4.10.Gremlin

#### 4.10.1.向HugeGraphServer发送gremlin语句，同步执行，压缩返回结果

##### 功能介绍

向HugeGraphServer发送gremlin语句（POST），同步执行，返回结果采用 Gzip 压缩

##### URI

```
POST /gremlin
```

##### URI参数

无

##### Body参数

|  名称   | 是否必填  | 类型  | 默认值 | 取值范围 | 说明  |
|  ----  | ----  | ----  | ----  | ----  | ---- |
| gremlin  | 是 | String  |   |  | 要发送给HugeGraphServer执行的gremlin语句  |
| bindings  | 否 | String  |   |  | 用来绑定参数，key是字符串，value是绑定的值（只能是字符串或者数字），功能类似于MySQL的 Prepared Statement，用于加速语句执行  |
| language  | 是 | String  |   |  | 发送语句的语言类型，默认为gremlin-groovy  |
| aliases  | 是 | String  |   |  | 为存在于图空间的已有变量添加别名  |

##### Response

|  名称   | 类型  |  说明  |
|  ----  | ----  | ----  |
| requestId  | String |  请求ID |
| status  | Map |  返回状态 |
| result  | Object |  结果对象 |

表1 status对象

|  名称   | 类型  |  说明  |
|  ----  | ----  | ----  |
| message  | String |  消息 |
| code  | Integer |  返回码 |
| attributes  | Map |  信息 |

表2 result对象

|  名称   | 类型  |  说明  |
|  ----  | ----  | ----  |
| data  | Array |  数据列表 |

表3 data对象(查询类型为顶点类型)

| 名称         | 类型     | 说明      |
|------------|--------|---------|
| id         | Int    | 顶点id    |
| lable      | String | 顶点label |
| type       | String | 类型      |
| properties | Json   | 属性集合    |

表4 data对象(查询类型为边类型)

| 名称         | 类型     | 说明      |
|------------|--------|---------|
 | id         | Int    | 边id |
 | lable      | String | 边label |
 | type       | String | 类型 |
 | outV       | String | 初始顶点 |
 | outVLabel  | String | 初始顶点label |
 | inV         | String | 目标顶点 |
 | inVLabel  | String | 目标顶点label |
 | properties | Json   | 属性集合 |

表5 data对象(查询类型为path类型)

| 名称         | 类型     | 说明                               |
 |--------|----------------------------------|---------|
| id         | Int    | 边/顶点id                           |
| labels      | String | 边/顶点label                        |
| objects       | Json   | 边/顶点集合，objects展开内容即为上述表3和表4的顶底和边 |

##### 使用示例1(查询类型为顶点类型)

###### Method & Url

```
POST http://localhost:8080/gremlin
```

###### Request Body

```json
{
	"gremlin": "graph.traversal().V('1:marko')",
	"bindings": {},
	"language": "gremlin-groovy",
	"aliases": {"graph":"gs1-hugegraph", "g":"__g_gs1-hugegraph"}
}
```

###### Response Status

```json
200
```

###### Response Body

```json
{
	"requestId": "c6ef47a8-b634-4b07-9d38-6b3b69a3a556",
	"status": {
		"message": "",
		"code": 200,
		"attributes": {}
	},
	"result": {
		"data": [{
			"id": "1:marko",
			"label": "person",
			"type": "vertex",
			"properties": {
				"city": "Beijing",
				"name": "marko",
				"age":  29
			}
		}],
		"meta": {}
	}
}
```

##### 使用示例2(查询类型为边类型)

###### Method & Url

```
POST http://localhost:8080/gremlin
```

###### Request Body

```json
{
	"gremlin": "graph.traversal().E('S1:li>2>>S2:lop1')",
	"bindings": {},
	"language": "gremlin-groovy",
	"aliases": {"graph":"gs1-hugegraph", "g":"__g_gs1-hugegraph"}
}
```

###### Response Status

```json
200
```

###### Response Body

```json
{
     "requestId": "3de127e8-6ef6-4d6e-b08c-d75c096682e2",
     "status": {
          "message": "",
          "code": 200,
          "attributes": {}
     },
     "result": {
          "data": [
               {
                    "id": "S1:li>2>>S2:lop1",
                    "label": "created",
                    "type": "edge",
                    "outV": "1:li",
                    "outVLabel": "person",
                    "inV": "2:lop1",
                    "inVLabel": "software",
                    "properties": {
                         "city": "Shenzhen",
                         "date": "2021-12-10 00:00:00.000"
                    }
               }
          ],
          "meta": {}
     }
}
```

##### 使用示例3(查询类型为path类型)

###### Method & Url

```
POST http://localhost:8080/gremlin
```

###### Request Body

```json
{
	"gremlin": "graph.traversal().E('graph.traversal().V().hasLabel('software').has('name','lop').bothE().otherV().path()')",
	"bindings": {},
	"language": "gremlin-groovy",
	"aliases": {"graph":"gs1-hugegraph", "g":"__g_gs1-hugegraph"}
}
```

###### Response Status

```json
200
```

###### Response Body

```json
{
    "requestId": "027120b7-bc68-4a41-98c1-d8f4ddc5926e",
    "status": {
         "message": "",
         "code": 200,
         "attributes": {}
    },
    "result": {
         "data": [
              {
                   "labels": [
                        [],
                        [],
                        []
                   ],
                   "objects": [
                        {
                             "id": "2:lop",
                             "label": "software",
                             "type": "vertex",
                             "properties": {
                                  "name": "lop",
                                  "lang": "java",
                                  "price": 328
                             }
                        },
                        {
                             "id": "S1:marko>2>>S2:lop",
                             "label": "created",
                             "type": "edge",
                             "outV": "1:marko",
                             "outVLabel": "person",
                             "inV": "2:lop",
                             "inVLabel": "software",
                             "properties": {
                                  "city": "Shanghai",
                                  "date": "2017-12-10 00:00:00.000"
                             }
                        },
                        {
                             "id": "1:marko",
                             "label": "person",
                             "type": "vertex",
                             "properties": {
                                  "name": "marko",
                                  "age": 29,
                                  "city": "Beijing"
                             }
                        }
                   ]
              },
              {
                   "labels": [
                        [],
                        [],
                        []
                   ],
                   "objects": [
                        {
                             "id": "2:lop",
                             "label": "software",
                             "type": "vertex",
                             "properties": {
                                  "name": "lop",
                                  "lang": "java",
                                  "price": 328
                             }
                        },
                        {
                             "id": "S1:peter>2>>S2:lop",
                             "label": "created",
                             "type": "edge",
                             "outV": "1:peter",
                             "outVLabel": "person",
                             "inV": "2:lop",
                             "inVLabel": "software",
                             "properties": {
                                  "city": "Beijing",
                                  "date": "2017-12-10 00:00:00.000"
                             }
                        },
                        {
                             "id": "1:peter",
                             "label": "person",
                             "type": "vertex",
                             "properties": {
                                  "name": "peter",
                                  "age": 29,
                                  "city": "Shanghai"
                             }
                        }
                   ]
              }
         ],
         "meta": {}
    }
}
```

注意：

> 这里是直接使用图对象（hugegraph），先获取其遍历器（traversal()），再获取顶点。
不能直接写成`graph.traversal().V()`或`g.V()`，可以通过`"aliases": {"graph": "gs1-hugegraph", "g": "__g_gs1-hugegraph"}`
为图和遍历器添加别名后使用别名操作。其中，`hugegraph`是原生存在的变量，`__g_gs1-hugegraph`是`HugeGraphServer`额外添加的变量，
每个图都会存在一个对应的这样格式（__g_${graphspace}-${graph}）的遍历器对象。

> 响应体的结构与其他 Vertex 或 Edge 的 RESTful API的结构有区别，用户可能需要自行解析。

#### 4.10.2.向HugeGraphServer发送gremlin语句，同步执行，不压缩返回结果

##### 功能介绍

向HugeGraphServer发送gremlin语句（POST），同步执行，不压缩返回的结果

##### URI

```
POST /gremlinunzip
```

##### URI参数

无

##### Body参数

|  名称   | 是否必填  | 类型  | 默认值 | 取值范围 | 说明  |
|  ----  | ----  | ----  | ----  | ----  | ---- |
| gremlin  | 是 | String  |   |  | 要发送给HugeGraphServer执行的gremlin语句  |
| bindings  | 否 | String  |   |  | 用来绑定参数，key是字符串，value是绑定的值（只能是字符串或者数字），功能类似于MySQL的 Prepared Statement，用于加速语句执行  |
| language  | 是 | String  |   |  | 发送语句的语言类型，默认为gremlin-groovy  |
| aliases  | 是 | String  |   |  | 为存在于图空间的已有变量添加别名  |

##### Response

|  名称   | 类型  |  说明  |
|  ----  | ----  | ----  |
| requestId  | String |  请求ID |
| status  | Map |  返回状态 |
| result  | Object |  结果对象 |

表1 status对象

|  名称   | 类型  |  说明  |
|  ----  | ----  | ----  |
| message  | String |  消息 |
| code  | Integer |  返回码 |
| attributes  | Map |  信息 |

表2 result对象

|  名称   | 类型  |  说明  |
|  ----  | ----  | ----  |
| data  | Array |  数据列表 |

表3 data对象

|  名称   | 类型  |  说明  |
|  ----  | ----  | ----  |
| id         | Int    | 顶点id    |
| lable      | String | 顶点label |
| type       | String | 类型      |
| properties | Json   | 属性集合    |


##### 使用示例

###### Method & Url

```
POST http://localhost:8080/gremlin
```

###### Request Body

```json
{
	"gremlin": "graph.traversal().V('1:marko')",
	"bindings": {},
	"language": "gremlin-groovy",
	"aliases": {"graph":"gs1-hugegraph", "g":"__g_gs1-hugegraph"}
}
```

###### Response Status

```json
200
```

###### Response Body

```json
{
	"requestId": "c6ef47a8-b634-4b07-9d38-6b3b69a3a556",
	"status": {
		"message": "",
		"code": 200,
		"attributes": {}
	},
	"result": {
		"data": [{
			"id": "1:marko",
			"label": "person",
			"type": "vertex",
			"properties": {
				"city": "Beijing",
				"name": "marko",
				"age": 29
			}
		}],
		"meta": {}
	}
}
```

注意：

> 这里是直接使用图对象（hugegraph），先获取其遍历器（traversal()），再获取顶点。
不能直接写成`graph.traversal().V()`或`g.V()`，可以通过`"aliases": {"graph": "gs1-hugegraph", "g": "__g_gs1-hugegraph"}`
为图和遍历器添加别名后使用别名操作。其中，`hugegraph`是原生存在的变量，`__g_gs1-hugegraph`是`HugeGraphServer`额外添加的变量，
每个图都会存在一个对应的这样格式（__g_${graphspace}-${graph}）的遍历器对象。

> 响应体的结构与其他 Vertex 或 Edge 的 RESTful API的结构有区别，用户可能需要自行解析。

#### 4.10.3.向HugeGraphServer发送gremlin语句，异步执行

##### 功能介绍

向HugeGraphServer发送gremlin语句（POST），异步执行

##### URI

```
POST /graphspaces/${graphspace}/graphs/${graph}/jobs/gremlin
```

##### URI参数

|  名称   | 是否必填  | 类型  | 默认值 | 取值范围 | 说明  |
|  ----  | ----  | ----  | ----  | ----  | ---- |
| graphspace  | 是 | String  |   |  | 图空间  |
| graph  | 是 | String  |   |  | 图  |

##### Body参数

|  名称   | 是否必填  | 类型  | 默认值 | 取值范围 | 说明  |
|  ----  | ----  | ----  | ----  | ----  | ---- |
| gremlin  | 是 | String  |   |  | 要发送给HugeGraphServer执行的gremlin语句  |
| bindings  | 否 | String  |   |  | 用来绑定参数，key是字符串，value是绑定的值（只能是字符串或者数字），功能类似于MySQL的 Prepared Statement，用于加速语句执行  |
| language  | 是 | String  |   |  | 发送语句的语言类型，默认为gremlin-groovy  |
| aliases  | 是 | String  |   |  | 为存在于图空间的已有变量添加别名  |

##### Response

|  名称   | 类型  |  说明  |
|  ----  | ----  | ----  |
| task_id  | Integer | 任务ID  |

##### 使用示例

###### Method & Url

```
POST http://localhost:8080/graphspaces/gs1/graphs/hugegraph/jobs/gremlin
```

###### Request Body

```json
{
	"gremlin": "g.V('1:marko')",
	"bindings": {},
	"language": "gremlin-groovy",
	"aliases": {}
}
```

###### Response Status

```json
201
```

###### Response Body

```json
{
	"task_id": 1
}
```

注意：

> 异步执行Gremlin语句暂不支持aliases，可以使用 `graph` 代表要操作的图，也可以直接使用图的名字， 例如 `hugegraph`;
另外`g`代表 traversal，等价于 `graph.traversal()` 或者 `hugegraph.traversal()`

注：

> 可以通过`GET http://localhost:8080/graphspaces/gs1/graphs/hugegraph/tasks/1`（其中"1"是task_id） 来查询异步任务的执行状态
