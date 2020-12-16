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

#### 2、Type（类型）废弃了

在index（索引）中，可以定义一个或多个类型，类似于mysql中的table，同一种类型的数据放在一起

####  3、 Document（文档）

保存在某个索引（Index）下、某种类型（Type）的一个数据（Document），文档是json格式的数据，Document1相当于mysql中某个table数据

