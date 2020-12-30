

### 一、什么是Elasticsearch?

[官网](https://www.elastic.co/cn/what-is/elasticsearch)

> Elasticsearch 是一个分布式的开源搜索和分析引擎，适用于所有类型的数据，包括文本、数字、地理空间、结构化和非结构化数据。Elasticsearch 在 Apache Lucene 的基础上开发而成，由 Elasticsearch N.V.（即现在的 Elastic）于 2010 年首次发布。Elasticsearch 以其简单的 REST 风格 API、分布式特性、速度和可扩展性而闻名，是 Elastic Stack 的核心组件；Elastic Stack 是适用于数据采集、充实、存储、分析和可视化的一组开源工具。人们通常将 Elastic Stack 称为 ELK Stack（代指 Elasticsearch、Logstash 和 Kibana），目前 Elastic Stack 包括一系列丰富的轻量型数据采集代理，这些代理统称为 Beats，可用来向 Elasticsearch 发送数据

着重功能就是用来做**数据的检索和分析**

- **应用程序搜索**
- 网站搜索
- 企业搜索
- **日志处理和分析**
- 基础设施**指标和容器监测**
- 应用程序性能监测
- 地理空间**数据分析和可视化**
- 安全分析
- 业务分析



### 二、mysql也能实现Elasticsearch的功能 为什么还需要Elasticsearch

mysql当然也能实现数据的搜索和分析  例如求年龄的平均值 avg  分组group by 等等 但是我们说术业有专攻，

而mysql专攻于持久化的存储与管理 ，也就是crud 。如果真的使用mysql做海量数据的检索和分析，Elasticsearch更在行，能在秒级给我们响应我们感兴趣的数据，而mysql单表达到百万数据，我们需要进行一些检索和查询都是一些比较慢的操作，比较浪费性能。例如在一些电商系统里面，需要对商品的不同属性按照不同的关键字来检索商品，我们如果用mysql来做，mysql肯定承受不了那么大的压力，而Elasticsearch可以帮我们做到这些。



### 三、底层

Elastisearch的底层是开元库Lucene。但是你没法直接调用lucene,必须自己写代码去调用他的接口。Elasticsearch是lucene的封装，提供了REST API的操作接口 开箱即用。

[官方文档]: https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html

### 四、基本概念

#### 1、Index（索引）

动词，相当于mysql中的insert

名词，相当于mysql的Database

#### 2、Type（类型）~~7.0废弃~~

在index（索引）中，可以定义一个或多个类型，类似于mysql中的table，同一种类型的数据放在一起

####  3、 Document（文档）

保存在某个索引（Index）下、某种类型（Type）的一个数据（Document），文档是json格式的数据，Document1相当于mysql中某个table数据

![elasticsearch概念](http://hinzzz.oss-cn-shenzhen.aliyuncs.com/elasticsearch%E6%A6%82%E5%BF%B5.png?Expires=32500886400&OSSAccessKeyId=LTAI4G9rkBZLb3G51wiGr2sS&Signature=EUv2ETlTQvW1TlA3bQ67%2FoQIX5E%3D)

#### 4、倒排索引

分词：将整句拆分为单词

保存的记录

1. 红海行动
2. 探索红海行动
3. 红海特别行动
4. 红海纪录片
5. 特工红海特别探索

| 词     | 得分      |
| ------ | --------- |
| 红海   | 1 2 3 4 5 |
| 行动   | 1 2 3     |
| 探索   | 2 5       |
| 特别   | 3         |
| 纪录片 | 4         |
| 特工   | 5         |

检索：

1. 红海特工行动
2. 红海行动



### 五、安装

注意elasticsearch和kibana的版本最好一致 不然容易出现错误

1、下载

```shell
docker pull elasticsearch:7.4.2 #存储和检索数据
docker pull kibana:7.4.2 #可视化检索工具
```

2、配置

```shell
mkdir -p /home/elasticsearch/config
mkdir -p /home/elasticsearch/data
echo "http.host: 0.0.0.0" >/home/elasticsearch/config/elasticsearch.yml
chmod -R 777 /home/elasticsearch/
```

3、运行

9200：http协议 外部通讯 外部访问

9300：tcp协议 内部通讯 用于集群

```shell
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
-e  "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms64m -Xmx256m" \
-v /home/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /home/elasticsearch/data:/usr/share/elasticsearch/data \
-v  /home/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:7.4.2 
```

开机自启动

```shell
docker update elasticsearch --restart=always
```

4、启动kibana

```shell
docker run --name kibana -e ELASTICSEARCH_HOSTS=http://localhost:9200 -p 5601:5601 -d kibana:7.4.2
```

开机自启动

```shell
docker update kibana  --restart=always
```

5、安装完成 访问http://ip:9200/，http://ip:5601/

版本信息

```json
{
  "name" : "244bb2701c87",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "4pa8UebPT4ye9hFNy2vNUA",
  "version" : {
    "number" : "7.4.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "2f90bbf7b93631e52bafb59b3b049cb44ec25e96",
    "build_date" : "2019-10-28T20:40:44.881551Z",
    "build_snapshot" : false,
    "lucene_version" : "8.2.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

### 六、初步检索

#### 1、_cat

GET _cat/nodes 查看所有节点

GET _cat/health 查看es健康状态

Get _cat/master 查看主节点

GET _cat/indices 查看所有索引

| 参数名  | 例子                                            | 作用                                                         |
| ------- | ----------------------------------------------- | ------------------------------------------------------------ |
| Verbose | GET /_cat/XXX/?v                                | 开启详细输出                                                 |
| Help    | GET /_cat/XXX/?help                             | 输出可用的列                                                 |
| Headers | GET /_cat/XXX/?h=column1,column2                | 指定输出的列                                                 |
| Sort    | GET /_cat/XXX/?v&s=column1,column2:desc,column3 | 指定输出的列进行排序，默认按照升序排序                       |
| Format  | GET /_cat/XXX?format=json                       | 指定响应返回的数据格式：text（默认）,json,yaml,smile,cbor（通过设置 Accept的HTTP头部的多媒体格式的优先级更高） |

### 七、索引一个文档

#### 1、Post

```shell
POST /mall/user/1
{
  "name":"hinzzz",
  "age":18
}
#在mall索引user类型下保存id为1的数据xxx，id可以不指定
#id已存在  执行修改操作 id不存在 新增 ，常用于新增
```

Post文档更新：

```shell
POST /mall/user/1/_update
{
  "doc":{
    "name":"hinzzz1",
    "age":18
  }
}

POST /mall/user/1/
{
  "doc":{
    "name":"hinzzz1",
    "age":18
  }
}
#1）带_update的时候会比较doc里面的值 和已存在的值是否一样 如果不一样执行修改操作 如果一样 不进行任何操作。重复执行更新操作，数据不会更新
#2）不带_update，重复执行更新操作，数据也能更新成功。
#应用场景：
#对于大并发更新，不带update
#对于大并发查询少更新，带update,对比更新，重新计算分配规则
```



#### 2、Put

```shell
PUT /mall/user/1
{
  "name":"wlq",
  "age":18
}
#mall索引user类型下保存或者修改id为1的数据xxx ，**id必须制定**
#id已存在  执行修改操作 id不存在 新增 ，常用于修改操作
PUT /mall/user/1?if_seq_no=5&if_primary_term=1 
#可带条件 修改seq_no=5 primary_term=1的数据
```

执行结果

“_index”: “mall” 表明该数据在哪个数据库下；

“_type”: “user” 表明该数据在哪个类型下；

“_id”: “1” 表明被保存数据的id；

“_version”: 1, 被保存数据的版本

“result”: “created” 这里是创建了一条数据，如果重新put一条数据，则该状态会变为updated，并且版本号也会发生变化。

#### 3、Get

```shell
GET /mall/user/1
#获取索引为mall类型为user并且id为1的数据
```

#### 4、Delete

```shell
DELETE mall/user/1 #根据id删除
DELETE mall #删除整个索引
```



#### 5、bulk(批量操作)

语法格式

```shell
{action:{metadata}}\n
{request body  }\n

{action:{metadata}}\n
{request body  }\n

#action：(行为)，包含create（文档不存在时创建）、update（更新文档）、index（创建新文档或替换已用文档）、delete（删除一个文档）。
#create和index的区别：如果数据存在，使用create操作失败，会提示文档已存在，使用index则可以成功执行。
#metadata：(行为操作的具体索引信息)，需要指明数据的_index、_type、_id。

POST /mall/user/_bulk
{"delete":{"_index":"mall","_type":"user","_id":"4"}} /#删除的批量操作不需要请求体
{"create":{"_index":"mall","_type":"user","_id":"100"}}
{"name":"hinz100"} #请求体
{"index":{"_index":"mall","_type":"user"}} #没有指定_id，elasticsearch将会自动生成_id
{"name":"hinz"} #请求体
{"update":{"_index":"mall","_type":"user","_id":"1"}} #/更新动作不能缺失_id，文档不存在更新将会失败
{"doc":{"name":"hinz1"}} #请求体

bluk一次最大处理多少数据量
bulk会将要处理的数据载入内存中，所以数据量是有限的，最佳的数据量不是一个确定的数据，它取决于你的硬件，你的文档大小以及复杂性，你的索引以及搜索的负载。

一般建议是1000-5000个文档，大小建议是5-15MB，默认不能超过100M，可以在es的配置文件（即$ES_HOME下的config下的elasticsearch.yml）中，bulk的线程池配置是内核数+1。

bulk批量操作的json格式解析
bulk的格式：
{action:{metadata}}\n
{requstbody}\n (请求体)

不用将其转换为json对象，直接按照换行符切割json，内存中不需要json文本的拷贝。
对每两个一组的json，读取meta，进行document路由。
直接将对应的json发送到node上。
为什么不使用如下格式：
[{"action":{},"data":{}}]
这种方式可读性好，但是内部处理就麻烦；耗费更多内存，增加java虚拟机开销：

将json数组解析为JSONArray对象，在内存中就需要有一份json文本的拷贝，宁外好友一个JSONArray对象。
解析json数组里的每个json，对每个请求中的document进行路由。
为路由到同一个shard上的多个请求，创建一个请求数组。
将这个请求数组序列化。

```

### 八、检索

- 通过REST request uri 发送搜索参数 （uri +检索参数）；
- 通过REST request body 来发送它们（uri+请求体）；



#### 1、Query DSL

elasticsearch提供了一个可以执行查询的Json风格的DSL，这个被称为Query DSL，查询语句非常全面

```shell
#基本语法格式
QUERY_NAME:{
   ARGUMENT:VALUE,
   ARGUMENT:VALUE,...
}
```

```shell
GET mall/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 5,
  "sort": {
      "age": {
        "order": "desc"
      }
    },
  "_source": ["name","age"]
}
#match_all 查询类型：意思是查询所有的所有,
#from,size 分页查询，
#sort排序 可组合多字段
#_source 指定查询结果字段 可支持多个
```

- match 匹配查询

- 基本类型  精确匹配

```shell
#查询返回age=18的数据
GET /mall/_search
{
  "query":{
    "match": {
      "age": "18"
    }
  }
}
```

- 文本类型 全文检索：最终会按照评分进行排序 会对检索条件进行分词匹配

```shell
#查询name 包含 "hinzzz"或者name 包含 "wlq"或者name 包含 "hinz100"的数据
GET /mall/_search
{
  "query":{
    "match": {
      "name": "hinzzz wlq hinz100"
    }
  }
}
```

- match_phrase	短句匹配：不会对查询条件进行分词检索

```shell
#查询name 包含 "hinzzz wlq"的结果
GET /mall/_search
{
  "query":{
    "match_phrase": {
      "name": "hinzzz wlq"
    }
  }
}
```

- keyworkd 精确匹配：匹配的条件就是要显示字段的全部值，要进行精确匹配的。

```shell
#查询name="hinzzz wlq"
GET /mall/_search
{
  "query":{
    "match_phrase": {
      "name.keyword": "hinzzz wlq"
    }
  }
}
```

- multi_math 多字段匹配

```shell
# 查询name和address 匹配sz的结果
GET /mall/_search
{
  "query":{
    "multi_match": {
      "query": "sz",
      "fields": ["name","address"]
    }
  }
}
```

- bool 用来做符合查询

复合语句可以合并，任何其他查询语句，包括符合语句。这也就意味着，复合语句之间
可以互相嵌套，可以表达非常复杂的逻辑。

```
must：必须达到must所列举的所有条件
must_not：必须不匹配must_not所列举的所有条件。
should：应该满足should所列举的条件。 类似or
```

```shell
GET /mall/_search
{
  "query":{
    "bool": {
      "should": [
        {
          "match": {
            "name": "hinzzz"
          }
        },
        {
          "match": {
            "age": "18"
          }
        }
      ]
    }
  }
}
```

- filter 结果过滤

```shell
#先查询name匹配hinzzz的结果 再从结果中获取 age>10 并且 age < 20
GET /mall/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
          "name": "hinzzz"
          }
        }
      ],
      "filter": {
        "range": {
          "age": {
            "gte": 10,
            "lte": 20
          }
        }
      }
    }
  }
}
```

