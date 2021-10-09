#### 1 Windows下启动Kibana

```shell
# Kibana的版本为7.8.0
bin/kibana.bat
```

#### 2 访问主页面

```shell
http://localhost:5601
```

#### 3 查看Elasticsearch集群健康情况

```shell
# 进入到Dev Tools界面
GET /_cluster/health
```

#### 4 Kibana操作Elasticsearch过程

**注：针对Elasticsearch-7.10.0操作，不同大版本的操作有些许不同。**

##### 4.1 创建数据

``` json
# 创建document：PUT /_index/_type/_id
# _type：_doc与_create
# _doc：若_id已存在，则覆盖原有数据
# _cretae：若_id已存在，则报错误
PUT /customer/_doc/1
{
  "name": "John Doe"
}

# 批量创建document
# 1、创建需要导入的jsons文件：accounts.json，注意最后一行要有换行符(Enter)
# 2、在accounts.json所在的目录打开CMD窗口，执行以下命令：
curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_bulk?pretty&refresh" --data-binary "@accounts.json"

# 创建索引
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "age":    { "type": "integer" },  
      "email":  { "type": "keyword"  }, 
      "name":   { "type": "text"  }     
    }
  }
}

# 指定索引增加字段
PUT /my-index-000001/_mapping
{
  "properties": {
    "employee-id": {
      "type": "keyword",
      "index": false
    }
  }
}
```

**accounts.json数据格式：**

```json
{"index":{"_id":"1"}}
{"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
{"index":{"_id":"6"}}
{"account_number":6,"balance":5686,"firstname":"Hattie","lastname":"Bond","age":36,"gender":"M","address":"671 Bristol Street","employer":"Netagy","email":"hattiebond@netagy.com","city":"Dante","state":"TN"}
{"index":{"_id":"13"}}
{"account_number":13,"balance":32838,"firstname":"Nanette","lastname":"Bates","age":28,"gender":"F","address":"789 Madison Street","employer":"Quility","email":"nanettebates@quility.com","city":"Nogal","state":"VA"}
```

##### 4.2 删除数据

```json
# 删除index：DELETE /_index
DELETE /my-index-000001
```

##### 4.3 更新数据

```json
# 更新_id为1的address
# POST /_index/_type/_id
# _type：_doc与_update
# _doc：覆盖原有数据
# _update：更新文档指定字段
POST /bank/_update/1
{
  "doc":{
    "address":"陕西渭南"
  }
}
```

##### 4.4 检索数据

```json
# 检索ID为1的document
GET /bank/_doc/1

# 按account_number升序排列检索位置在10-19(从0开始)的数据，默认只查出前十条数据
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ],
  "from": 10,
  "size": 10
}

# 根据地址进行检索
# 匹配重视：match表示mill或lane；match_phrase表示mill和lane，不区分大小写
GET /bank/_search
{
  "query": { "match_phrase": { "address": "mill lane" } }
}

# 组合查询，其中should表示可匹配，也可不匹配，增大匹配因素_score
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ],
      "should": [
        { "match": { "city": "Nord" } },
        { "match": { "gender": "M" } }
      ]  
    }
  }
}

# 组合查询，开启过滤条件
GET /bank/_search
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}

# 批量根据_id检索document
GET /_mget
{
  "docs":[
    {"_index":"bank","_id":"1"},
    {"_index":"bank","_id":"2"},
    {"_index":"bank","_id":"3"},
    {"_index":"bank","_id":"4"},
    {"_index":"bank","_id":"5"}
  ]
}
# 查看_index信息
curl "localhost:9200/_cat/indices?v"

# 查询索引详情和索引配置
GET /my-index-000001

# 查询索引详情
GET /my-index-000001/_mapping

# 查询索引中指定字段详情
GET /my-index-000001/_mapping/field/employee-id
```

