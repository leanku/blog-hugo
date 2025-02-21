---
title: "ElasticSearch介绍"
date: 2023-07-19T20:46:01+08:00
draft: false
categories: ["ElasticSearch"]
tags: ["ElasticSearch"]
keywords: ["ElasticSearch","微服务"]
---

# ElasticSearch

## 1. ElasticSearch介绍
Elasticsearch 是一个分布式、可扩展、实时的搜索和分析引擎，基于 Apache Lucene 构建。它能够快速存储、搜索和分析大量数据，广泛应用于全文搜索、日志分析、业务指标监控等场景。
  

### Lucene 介绍
  Lucene是开源、免费、高性能、纯Java编写的全文检索工具包

  他是一个全文检索的工具包，是一个全文检索框架，并不是一个全文检索引擎

  它非常复杂，并且需要Java集成使用

### Lucene 和 ElasticSearch

* ElasticSearch和solr是基于lucene的开源项目
* ElasticSearch通过简单易用的restful api接口，隐藏了lucene的复杂性
* ElasticSearch自带分布式管理，并且可以跨语言使用

## Lucene 和 Solr
* ElasticSearch自带分布式管理而Solr需要借助Zookpeeper实现分布式管理
* Solr支持多格式的数据，在传统搜索中表现好于ES,但是它的更新效率比较低
* ElasticSearch只支持json格式的数据，在处理实时索引搜索时明显好于Solr

## 2. 核心概念
### 1. 文档（Document）
* Elasticsearch 中的基本数据单元，类似于关系数据库中的一行记录。
* 文档以 JSON 格式存储，包含多个字段。
* 示例：
```
{
  "id": 1,
  "title": "Elasticsearch Guide",
  "content": "Elasticsearch is a distributed search engine.",
  "tags": ["search", "distributed"]
}
```

### 2.索引（Index）
* 索引是文档的集合，类似于关系数据库中的表。
* 每个索引有一个唯一的名称，用于标识和操作数据。
* 示例：books 索引存储所有书籍相关的文档。

### 3.类型（Type）（已弃用）
在早期版本中，索引可以包含多个类型（类似于表结构），但在 Elasticsearch 7.x 及更高版本中已被弃用。

### 4.分片（Shard）
* 索引可以被分成多个分片，每个分片是一个独立的 Lucene 索引。
* 分片允许数据水平拆分，支持分布式存储和并行处理。
* 分片分为主分片（Primary Shard）和副本分片（Replica Shard）。

### 5.节点（Node）
* 节点是 Elasticsearch 集群中的一个服务器实例，负责存储数据和执行操作。
* 节点可以扮演不同的角色（如主节点、数据节点、协调节点）。

###  6.集群（Cluster）
* 集群由一个或多个节点组成，共同存储数据并提供搜索服务。
* 集群通过唯一的名称标识。

## 3.架构与工作原理

### 1.分布式架构
* Elasticsearch 采用分布式设计，数据存储在多个节点上。
* 数据被分成多个分片，每个分片可以有多个副本，确保高可用性和容错性。

### 2.数据写入流程
1. 客户端发送写入请求到协调节点。
2. 协调节点根据文档 ID 计算目标分片，并将请求转发到主分片所在的节点。
3. 主分片写入数据后，同步到副本分片。
4. 写入成功后，返回响应给客户端。

### 3.数据搜索流程
1. 客户端发送搜索请求到协调节点。
2. 协调节点将请求广播到所有相关分片（主分片或副本分片）。
3. 每个分片执行搜索操作，返回结果给协调节点。
4. 协调节点合并结果，排序后返回给客户端。

### 4.倒排索引
* Elasticsearch 使用倒排索引（Inverted Index）实现快速全文搜索。
* 倒排索引将文档中的每个词映射到包含该词的文档列表。
* 示例：
  * 文档 1：{"content": "Elasticsearch is fast"}
  * 文档 2：{"content": "Elasticsearch is distributed"}
  * 倒排索引：
  ```
  "Elasticsearch" -> [文档1, 文档2]
  "fast" -> [文档1]
  "distributed" -> [文档2]
  ```

## 3.主要特性

### 1.高性能
* 支持实时搜索和分析，响应时间通常在毫秒级。
* 通过分布式架构和倒排索引，能够快速处理大规模数据。

### 2.可扩展性
* 支持水平扩展，可以通过增加节点来提升存储和计算能力。
* 自动分片和副本机制，确保数据分布均衡。

### 3.高可用性
* 通过副本分片实现数据冗余，确保节点故障时数据不丢失。
* 主节点选举机制，确保集群的高可用性。

### 4.丰富的查询功能
* 支持全文搜索、结构化搜索、模糊搜索、范围搜索等多种查询方式。
* 提供聚合（Aggregation）功能，支持数据统计和分析。

### 5.插件生态
* 支持丰富的插件，扩展 Elasticsearch 的功能。
* 例如：IK 分词插件（中文分词）、Elasticsearch-Hadoop（与 Hadoop 集成）。
  * ik分词器
一个标准的中文分词器。可以根据定义的字典对域进行粉刺，并且支持用户配置自己的字典，所以它除了可以按通用的习惯分词外，还可以定制化分词。 可以使用插件的方式将他接入到ES。

    ik分词器有两种分词方式：ik_smart最粗粒度的拆分和ik_max_word最细粒度的拆分。


##  4.使用场景

###  1.全文搜索
* 适用于搜索引擎、电商网站的商品搜索、内容管理系统的文档搜索等场景。
* 示例：在电商网站中搜索商品名称、描述、分类等信息。

###  2.日志分析
* 与 Logstash 和 Kibana 结合，构建日志管理和分析系统（ELK Stack）。
* 示例：分析服务器日志，监控系统状态，排查故障。

### 3.业务指标监控
* 实时分析业务数据，生成可视化报表。
* 示例：监控网站访问量、用户行为、交易量等指标。

### 4.数据挖掘与分析
* 使用聚合功能，对大规模数据进行统计分析。
* 示例：分析用户的地理分布、购买偏好等。

## 5.优缺点

### 1.优点
* **高性能**：支持实时搜索和分析。
* **可扩展性**：支持水平扩展，适应大规模数据。
* **高可用性**：通过副本机制确保数据安全。
* **易用性**：提供 RESTful API，易于集成和使用。

### 2.缺点
* **资源消耗**：对内存和 CPU 要求较高。
* **复杂性**：分布式系统的部署和维护较为复杂。
* **数据一致性**：在极端情况下可能出现数据不一致问题。

## 5.示例：使用 Elasticsearch 构建商品搜索系统

1. 数据准备
   1. 创建 products 索引，定义字段映射：
   ``` jSON
   PUT /products
    {
      "mappings": {
        "properties": {
          "name": { "type": "text" },
          "price": { "type": "float" },
          "category": { "type": "keyword" }
        }
      }
    }
    ```

2. 写入数据
   1. 插入商品数据：
   ``` jSON
   POST /products/_doc/1
    {
      "name": "Smartphone X",
      "price": 599.99,
      "category": "Electronics"
    }
    ```

3. 搜索数据
   1. 搜索名称包含 "Smartphone" 的商品：
   ``` jSON
   GET /products/_search
    {
      "query": {
        "match": {
          "name": "Smartphone"
        }
      }
    }
    ```

4. 聚合分析
   1. 按类别统计商品数量：
   ``` jSON
   GET /products/_search
    {
      "size": 0,
      "aggs": {
        "categories": {
          "terms": {
            "field": "category"
          }
        }
      }
    }
    ```
5. 更新
   1. 完全替换文档
   
      通过指定文档 ID，直接覆盖旧文档。若文档不存在，则会创建新文档。

      示例：更新商品价格
      ``` JSON
      PUT /products/_doc/1
      {
        "name": "Smartphone X",
        "price": 549.99,  // 价格从 599.99 更新为 549.99
        "category": "Electronics"
      }
      ```
      操作说明：

      * Elasticsearch 会先删除旧文档（ID=1），再写入新文档。
      * 即使只修改部分字段，也需要传递完整的文档内容。

     2. 部分更新（Partial Update）
     
        使用 _update API，仅更新文档的指定字段，无需传递完整文档。

        语法:
        ``` JSON
        POST /<index>/_update/<doc_id>
        {
          "doc": {
            "field1": "new_value1",
            "field2": "new_value2"
          }
        }
        ```
        示例：仅更新商品价格
        ``` JSON
        POST /products/_update/1
        {
          "doc": {
            "price": 499.99
          }
        }
        ```
        操作说明：

        * Elasticsearch 内部会执行以下操作：

          1. 获取旧文档。

          2. 合并新旧文档的字段。

          3. 删除旧文档，写入新文档。

        * 优点：减少网络传输数据量，适合仅更新少量字段的场景。

    3. 使用脚本更新（Scripted Update）
   
        通过 Painless 脚本动态更新文档字段，支持复杂的逻辑（如条件更新、计算字段值等）。

        语法：
        ``` JSON
        POST /<index>/_update/<doc_id>
        {
          "script": {
            "source": "ctx._source.<field> = <value>",
            "lang": "painless"
          }
        }
        ```
        示例：将商品价格打 9 折
        ``` JSON
        POST /products/_update/1
        {
          "script": {
            "source": "ctx._source.price *= 0.9",
            "lang": "painless"
          }
        }
        ```
        示例：条件更新（仅当价格高于 500 时打折）
        ``` JSON
        POST /products/_update/1
        {
          "script": {
            "source": """
              if (ctx._source.price > 500) {
                ctx._source.price *= 0.8;
              }
            """,
            "lang": "painless"
          }
        }
        ```
    4.批量更新

    使用 _bulk API 批量更新文档，提升效率。
    ``` jSON
    POST /_bulk
    { "update": { "_index": "products", "_id": "1" } }
    { "doc": { "price": 499.99 } }
    { "update": { "_index": "products", "_id": "2" } }
    { "doc": { "price": 299.99 } }
    ```
   