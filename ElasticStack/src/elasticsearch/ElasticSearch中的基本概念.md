## ElasticSearch中的基本概念

### 索引

- 索引（index）是Elasticsearch对逻辑数据的逻辑存储，所以它可以分为更小的部分。
- 可以把索引看成关系型数据库的表，索引的结构是为快速有效的全文索引准备的，特别是它不存储原始值。
- Elasticsearch可以把索引存放在一台机器或者分散在多台服务器上，每个索引有一或多个分片（shard），每个分片可以有多个副本（replica）。

### 文档

- 存储在Elasticsearch中的主要实体叫文档（document）。用关系型数据库来类比的话，一个文档相当于数据库表中的一行记录。
- Elasticsearch和MongoDB中的文档类似，都可以有不同的结构，但Elasticsearch的文档中，相同字段必须有相同类型。
- 文档由多个字段组成，每个字段可能多次出现在一个文档里，这样的字段叫多值字段（multivalued）。
  每个字段的类型，可以是文本、数值、日期等。字段类型也可以是复杂类型，一个字段包含其他子文档或者数
  组。

### 映射

所有文档写进索引之前都会先进行分析，如何将输入的文本分割为词条、哪些词条又会被过滤，这种行为叫做
映射（mapping）。一般由用户自己定义规则。

### 文档类型

- 在Elasticsearch中，一个索引对象可以存储很多不同用途的对象。例如，一个博客应用程序可以保存文章和评
  论。
- 每个文档可以有不同的结构。
- 不同的文档类型不能为相同的属性设置不同的类型。例如，在同一索引中的所有文档类型中，一个叫title的字段必须具有相同的类型。

## RESTful API

在Elasticsearch中，提供了功能丰富的RESTful API的操作，包括基本的CRUD、创建索引、删除索引等操作。

### 创建非结构化索引

在Lucene中，创建索引是需要定义字段名称以及字段的类型的，在Elasticsearch中提供了非结构化的索引，就是不需要创建索引结构，即可写入数据到索引中，实际上在Elasticsearch底层会进行结构化操作，此操作对用户是透明的。

### 创建空索引

```bash
PUT /haoke
{
    "settings": {
        "index": {
        "number_of_shards": "2", #分片数
        "number_of_replicas": "0" #副本数
        }
    }
}
```

### 删除索引

```bash
#删除索引
DELETE /haoke
{
	"acknowledged": true
}
```

### 插入数据

>URL规则：
>POST /{索引}/{类型}/{id}

```bash
POST /haoke/user/1001
#数据
{
"id":1001,
"name":"张三",
"age":20,
"sex":"男"
}
```

使用postman操作成功后

![img_1](../../../day08/images/img_1.png)

我们通过ElasticSearchHead进行数据预览就能够看到我们刚刚插入的数据了

![img_2](../../../day08/images/img_2.png)

说明：非结构化的索引，不需要事先创建，直接插入数据默认创建索引。不指定id插入数据：

![img_2](../../../day08/images/img_2.png)

### 更新数据

在Elasticsearch中，文档数据是不为修改的，但是可以通过覆盖的方式进行更新。

```bash
PUT /haoke/user/1001
{
"id":1001,
"name":"张三",
"age":21,
"sex":"女"
}
```

更新结果如下：

![img_3](../../../day08/images/img_3.png)

可以看到数据已经被覆盖了。问题来了，可以局部更新吗？ -- 可以的。前面不是说，文档数据不能更新吗？ 其实是这样的：在内部，依然会查询到这个文档数据，然后进行覆盖操作，步骤如下：

1. 从旧文档中检索JSON
2. 修改它
3. 删除旧文档
4. 索引新文档

```bash
#注意：这里多了_update标识
POST /haoke/user/1001/_update
{
"doc":{
"age":23
}
}
```


![img_4](../../../day08/images/img_4.png)

可以看到，数据已经是局部更新了

### 删除索引

在Elasticsearch中，删除文档数据，只需要发起DELETE请求即可，不用额外的参数

```bash
DELETE /haoke/user/1001
```

![img_5](../../../day08/images/img_5.png)

需要注意的是，result表示已经删除，version也增加了。

如果删除一条不存在的数据，会响应404

![img_6](../../../day08/images/img_6.png)
> 删除一个文档也不会立即从磁盘上移除，它只是被标记成已删除。Elasticsearch将会在你之后添加更多索引的时候才会在后台进行删除内容的清理。【相当于批量操作】

### 搜索数据

#### 根据id搜索数据

```bash
GET /haoke/user/_uKggXsBWk1QZxnzOi4y
#返回的数据如下
{
    "_index": "haoke",
    "_type": "user",
    "_id": "_uKggXsBWk1QZxnzOi4y",
    "_version": 1,
    "_seq_no": 5,
    "_primary_term": 1,
    "found": true,
    "_source": {
        "id": 4,
        "name": "张三",
        "age": 20,
        "sex": "男"
    }
}
```

#### 搜索全部数据

```bash
GET  /haoke/user/_search
```

注意，使用查询全部数据的时候，默认只会返回10条
````
{
"took": 9,
"timed_out": false,
"_shards": {
"total": 2,
"successful": 2,
"skipped": 0,
"failed": 0
},
"hits": {
"total": {
"value": 3,
"relation": "eq"
},
"max_score": 1.0,
"hits": [
{
"_index": "haoke",
"_type": "user",
"_id": "1002",
"_score": 1.0,
"_source": {
"id": 1002,
"name": "张三",
"age": 20,
"sex": "男"
}
},
{
"_index": "haoke",
"_type": "user",
"_id": "_uKggXsBWk1QZxnzOi4y",
"_score": 1.0,
"_source": {
"id": 4,
"name": "张三",
"age": 20,
"sex": "男"
}
},
{
"_index": "haoke",
"_type": "user",
"_id": "1003",
"_score": 1.0,
"_source": {
"id": 3,
"name": "张三",
"age": 23,
"sex": "女"
}
}
]
}
}
````
#### 关键字搜索数据

```bash
#查询年龄等于20的用户
GET /haoke/user/_search?q=age:20
```

结果如下：

![img_7](../../../day08/images/img_7.png)

### DSL搜索

Elasticsearch提供丰富且灵活的查询语言叫做DSL查询(Query DSL),它允许你构建更加复杂、强大的查询。
DSL(Domain Specific Language特定领域语言)以JSON请求体的形式出现。

```bash
POST /haoke/user/_search
#请求体
{
    "query" : {
        "match" : { #match只是查询的一种
        	"age" : 20
        }
    }
}
```

实现：查询年龄大于30岁的男性用户。

现有数据
![img_8](../../../day08/images/img_8.png)

```bash
POST /haoke/user/_search
#请求数据
{
    "query": {
        "bool": {
            "filter": {
                    "range": {
                        "age": {
                        "gt": 30
                    }
                }
            },
            "must": {
                "match": {
                	"sex": "男"
                }
            }
        }
    }
}
```

查询出来的结果

![img_9](../../../day08/images/img_9.png)

#### 全文搜索

```bash
POST /haoke/user/_search
#请求数据
{
    "query": {
        "match": {
        	"name": "张三 李四"
        }
    }
}
```

![img_10](../../../day08/images/img_10.png)

高亮显示，只需要在添加一个 highlight即可

```bash
POST /haoke/user/_search
#请求数据
{
    "query": {
        "match": {
        	"name": "张三 李四"
        }
    }
    "highlight": {
        "fields": {
        	"name": {}
        }
    }
}
```

![img_11](../../../day08/images/img_11.png)

#### 聚合

在Elasticsearch中，支持聚合操作，类似SQL中的group by操作。

```bash
POST /haoke/user/_search
{
    "aggs": {
        "all_interests": {
            "terms": {
                "field": "age"
            }
        }
    }
}
```

结果如下，我们通过年龄进行聚合

![img_12](../../../day08/images/img_12.png)

从结果可以看出，年龄30的有2条数据，20的有一条，40的一条。