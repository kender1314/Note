#         Elasticsearch 学习笔记

————笔记参考elasticsearch 2.x

## 1     Elasticsearch概述

### 1.1      Elasticsearch是什么

\1.   Elasticsearch 是一个分布式、可扩展、实时的搜索与数据分析引擎。

\2.   Elasticsearch 是一个开源的搜索引擎，建立在一个全文搜索引擎库 Apache Lucene™ 基础之上。

### 1.2      Elasticsearch优势

\1.   在处理大规模的数据的时候，以很快的速度进行数据搜索。

\2.   革命性的成果：将单独的，有用的组件融合到一个单一的、一致的、实时的应用中。如全文搜索、 分析系统和分布式数据库

\3.   一个分布式的实时文档存储，每个字段 可以被索引与搜索

\4.   一个分布式实时分析搜索引擎

\5.   能胜任上百个服务节点的扩展，并支持 PB 级别的结构化或者非结构化数据

### 1.3      Elasticsearch请求方式

#### 1.3.1    GET 查询与搜索

##### 1.3.1.1    请求过程中使用的参数

1）     pretty：将会调用 Elasticsearch 的 pretty-print 功能，该功能 使得 JSON 响应体更加可读。（这个参数可以用于很多地方）

```
GET /_count?pretty
{
  "query": {
    "match_all": {}
  }
}
```

2）     _source：可以显示指定字段

```
GET /website/blog/123?_source=title,text
```

3）     timeout：响应时间

如果低响应时间比完成结果更重要，你可以指定 timeout 为 10 或者 10ms（10毫秒），或者 1s（1秒）：

```
GET /_search?timeout=10ms
```

在请求超时之前，Elasticsearch 将会返回已经成功从每个分片获取的结果。

**注**：*应当注意的是* *timeout* *不是停止执行查询，它仅仅是告知正在协调的节点返回到目前为止收集的结果并且关闭连接。在后台，其他的分片可能仍在执行查询即使是结果已经被发送了。*

##### 1.3.1.2    响应体中的字段

​          ![image-20200704000042896](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704000042896.png)                     

图 1—1

###### 1.3.1.2.1    hits

返回结果中最重要的部分是 hits ，它包含 total 字段来表示匹配到的文档总数，并且一个 hits 数组包含所查询结果的前十个文档。

在 hits 数组中每个结果包含文档的 _index 、 _type 、 _id ，加上 _source 字段。这意味着我们可以直接从返回的搜索结果中使用整个文档。这不像其他的搜索引擎，仅仅返回文档的ID，需要你单独去获取文档。

###### 1.3.1.2.2    took

took 值告诉我们执行整个搜索请求耗费了多少毫秒。

###### 1.3.1.2.3    shards

_shards 部分告诉我们在查询中参与分片的总数，以及这些分片成功了多少个失败了多少个。

##### 1.3.1.3    分页

让查询到的数据分页显示。

**size****：**显示应该返回的结果数量，默认是 10

**from****：**显示应该跳过的初始结果数量，默认是 0

如果每页展示 5 条结果，可以用下面方式请求得到 1 到 3 页的结果：

```
GET /_search?size=5
GET /_search?size=5&from=5
GET /_search?size=5&from=10
```

#### 1.3.2    PUT 更新与插入

写、更新入操作

PUT /{index}/{type}/{id}

{

 "field": "value",

 ...

}

acknowledged 表示是否在集群成功创建索引, 同时 shards_acknowledged 表示是否在超时之前成功创建运行必要的分片。

 

#### 1.3.3    DELETE 删除

1）     正如已经在更新整个文档中提到的，删除文档不会立即将文档从磁盘中删除，只是将文档标记为已删除状态。随着你不断的索引更多的数据，Elasticsearch 将会在后台清理标记为已删除的文档。

2）     即使删除失败，_version 值仍然会增加。

#### 1.3.4    HEAD 确认文档是否存在

curl -i -XHEAD http://localhost:9200/website/blog/123

#### 1.3.5    POST 实现ID自增

写、更新入操作，如果插入信息的时候，不指定id，则使用POST.

#### 1.3.6    POST和PUT的区别

l 更新：PUT会将新的json值完全替换掉旧的；而POST方式只会更新相同字段的值，其他数据不会改变，新提交的字段若不存在则增加。

l PUT和DELETE操作是幂等的。所谓幂等是指不管进行多少次操作，结果都一样。比如用PUT修改一篇文章，然后在做同样的操作，每次操作后的结果并没有什么不同，DELETE也是一样。

l POST操作不是幂等的，比如常见的POST重复加载问题：当我们多次发出同样的POST请求后，其结果是创建了若干的资源。

l 创建操作可以使用POST，也可以使用PUT，区别就在于POST是作用在一个集合资源(/articles)之上的，而PUT操作是作用在一个具体资源之上的(/articles/123)。

### 1.4      Elasticsearch解决冲突的方法

#### 1.4.1    悲观并发控制

这种方法被关系型数据库广泛使用，它假定有变更冲突可能发生，因此阻塞访问资源以防止冲突。 一个典型的例子是读取一行数据之前先将其锁住，确保只有放置锁的线程能够对这行数据进行修改。

#### 1.4.2    乐观并发控制

Elasticsearch 中使用的这种方法假定冲突是不可能发生的，并且不会阻塞正在尝试的操作。 然而，如果源数据在读写当中被修改，更新将会失败。应用程序接下来将决定该如何解决冲突。 例如，可以重试更新、使用新的数据、或者将相关情况报告给用户。

Elasticsearch 是分布式的。当文档创建、更新或删除时， 新版本的文档必须复制到集群中的其他节点。Elasticsearch 也是异步和并发的，这意味着这些复制请求被并行发送，并且到达目的地时也许 顺序是乱的 。 Elasticsearch 需要一种方法确保文档的旧版本不会覆盖新的版本。

当我们之前讨论 index ， GET 和 delete 请求时，我们指出每个文档都有一个 _version （版本）号，当文档被修改时版本号递增。 Elasticsearch 使用这个 _version 号来确保变更以正确顺序得到执行。如果旧版本的文档在新版本之后到达，它可以被简单的忽略。

我们可以利用 _version 号来确保 应用中相互冲突的变更不会导致数据丢失。我们通过指定想要修改文档的 version 号来达到这个目的。 如果该版本不是当前版本号，我们的请求将会失败。

让我们创建一个新的博客文章：

PUT /website/blog/1/_create

{

 "title": "My first blog entry",

 "text": "Just trying this out..."

}

响应体告诉我们，这个新创建的文档 _version 版本号是 1 。现在假设我们想编辑这个文档：我们加载其数据到 web 表单中， 做一些修改，然后保存新的版本。

GET /website/blog/1

响应体包含相同的 _version 版本号 1 ：

 ![image-20200704000142066](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704000142066.png)

图 1—2

现在，当我们尝试通过重建文档的索引来保存修改，我们指定 version 为我们的修改会被应用的版本：

PUT /website/blog/1?version=1 

{

 "title": "My first blog entry",

 "text": "Starting to get the hang of this..."

}

注：*我们想这个在我们索引中的文档只有现在的* *_version* *为* *1* *时，本次更新才能成功*

此请求成功，并且响应体告诉我们 _version 已经递增到 2 ：

 ![image-20200704000147729](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704000147729.png)

图 1—3

然而，如果我们重新运行相同的索引请求，仍然指定 version=1 ， Elasticsearch 返回 409 ConflictHTTP 响应码，和一个如下所示的响应体：

 ![image-20200704000151585](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704000151585.png)

图 1—4

这告诉我们在 Elasticsearch 中这个文档的当前 _version 号是 2 ，但我们指定的更新版本号为 1 。

 

##### 1.4.2.1    通过外部系统使用版本控制

一个常见的设置是使用其它数据库作为主要的数据存储，使用 Elasticsearch 做数据检索， 这意味着主数据库的所有更改发生时都需要被复制到 Elasticsearch ，如果多个进程负责这一数据同步，你可能遇到类似于之前描述的并发问题。

如果你的主数据库已经有了版本号 — 或一个能作为版本号的字段值比如 timestamp — 那么你就可以在 Elasticsearch 中通过增加 version_type=external 到查询字符串的方式重用这些相同的版本号， 版本号必须是大于零的整数， 且小于 9.2E+18 — 一个 Java 中 long 类型的正值。

外部版本号的处理方式和我们之前讨论的内部版本号的处理方式有些不同， Elasticsearch 不是检查当前 _version 和请求中指定的版本号是否相同， 而是检查当前 _version 是否 小于 指定的版本号。 如果请求成功，外部的版本号作为文档的新 _version 进行存储。

外部版本号不仅在索引和删除请求是可以指定，而且在 创建 新文档时也可以指定。

PUT /website/blog/2?version=5&version_type=external

{

 "title": "My first external blog entry",

 "text": "Starting to get the hang of this..."

}

 

 

 

 

 

 

 

## 2     基本知识

### 2.1      Elasticsearch 交互

#### 2.1.1    Java API

使用 Java，在代码中可以使用 Elasticsearch 内置的两个客户端：

##### 2.1.1.1    节点客户端（Node client）

节点客户端作为一个非数据节点加入到本地集群中。换句话说，它本身不保存任何数据，但是它知道数据在集群中的哪个节点中，并且可以把请求转发到正确的节点。

##### 2.1.1.2    传输客户端（Transport client）

轻量级的传输客户端可以将请求发送到远程集群。它本身不加入集群，但是它可以将请求转发到集群中的一个节点上。

注：

*1.*    *两个* *Java* *客户端都是通过* *9300* *端口并使用* *Elasticsearch* *的原生传输协议和集群交互。集群中的节点通过端口* *9300* *彼此通信。如果这个端口没有打开，节点将无法形成一个集群。*

*2.*    *Java* *客户端作为节点必须和* *Elasticsearch* *有相同的主要版本；否则，它们之间将无法互相理解。*

#### 2.1.2    RESTful API with JSON over HTTP

1）     所有其他语言可以使用 RESTful API 通过端口 9200 和 Elasticsearch 进行通信，可以用 web 客户端访问 Elasticsearch ，甚至可以使用 curl 命令来和 Elasticsearch 交互。

2）     Elasticsearch 为以下语言提供了官方客户端--Groovy、JavaScript、.NET、 PHP、 Perl、 Python 和 Ruby—还有很多社区提供的客户端和插件，所有这些都可以在 Elasticsearch Clients 中找到。

### 2.2      面向文档（存储方式）

#### 2.2.1    原理

1）     Elasticsearch 是面向文档的，意味着它存储整个对象或文档。

2）  Elasticsearch 不仅存储文档，而且索引每个文档的内容，使之可以被检索。在 Elasticsearch 中，我们对文档进行索引、检索、排序和过滤—而不是对行列数据。

3）  文档指最顶层或者根对象, 这个根对象被序列化成 JSON 并存储到 Elasticsearch 中，指定了唯一 ID。

4）  文档不能被修改，只能被替换。

#### 2.2.2    文档定义

通常情况下，我们使用的术语 对象 和 文档 是可以互相替换的。不过，有一个区别： 一个对象仅仅是类似于 hash 、 hashmap 、字典或者关联数组的JSON对象，对象中也可以嵌套其他的对象。 对象可能包含了另外一些对象。在 Elasticsearch 中，术语文档有着特定的含义。它是指最顶层或者根对象, 这个根对象被序列化成 JSON 并存储到 Elasticsearch中，指定了唯一ID。

#### 2.2.3    优势

能支持复杂的全文检索。

#### 2.2.4    分布式文档存储

**1****）** **Elasticsearch** **如何知道一个文档应该存放到哪个分片中呢？**

答：是根据以下公式存放

```
shard = hash(routing) % number_of_primary_shards
```

routing 是一个可变值，默认是文档的 _id ，也可以设置成一个自定义的值。 routing 通过 hash 函数生成一个数字，然后这个数字再除以 number_of_primary_shards （主分片的数量）后得到 余数 。这个分布在 0 到 number_of_primary_shards-1 之间的余数，就是我们所寻求的文档所在分片的位置。

**Routing****如何自定义设置？**

```
PUT /forums/post/1?routing=2 
{
  "forum_id": "baking", 
  "title":    "Easy recipe for ginger nuts",
  ...
}
```

 

**注**：*所有的文档* *API**（* *get* *、* *index* *、* *delete* *、* *bulk* *、* *update* *以及* *mget* *）都接受一个叫做* *routing* *的路由参数* *，通过这个参数我们可以自定义文档到分片的映射。一个自定义的路由参数可以用来确保所有相关的文档——例如所有属于同一个用户的文档——都被存储到同一个分片中。*

#### 2.2.5    文档元数据

##### 2.2.5.1    文档标识

文档标识包含以下四元素

###### 2.2.5.1.1    _index

1）     代表文档在哪存放

2）     索引名必须小写，不能以下划线开头，不能包含逗号

###### 2.2.5.1.2    _type

1）     文档表示的对象类别

2）     类型相当于索引下数据的子分区，将数据分成不同类别

3）     类型下文档共享一种相同的（或非常相似）的模式：他们有一个标题、描述、产品代码和价格。他们只是正好属于“产品”（假设该类型所处索引是存储产品信息）下的一些子类。

4）     _type 命名可以是大写或者小写，但是不能以下划线或者句号开头，不应该包含逗号， 并且长度限制为256个字符

###### 2.2.5.1.3    _id

1）     文档唯一标识

2）     不再使用 PUT 谓词， 而是使用 POST 谓词，可以实现id的自增。

现在该 URL 只需包含 _index 和 _type :

```
POST /website/blog/
{
  "title": "My second blog entry",
  "text":  "Still trying this out...",
  "date":  "2014/01/01"
}
```

结果：

 ![image-20200704000250921](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704000250921.png)

图 2—1

 

注：

*自动生成的* *ID* *是* *URL-safe**、* *基于* *Base64* *编码且长度为**20**个字符的* *GUID* *字符串。*

###### 2.2.5.1.4    _uid

_uid实际上是index下的type与_id拼接，格式为_uid=type#_id假设我们有一个accout的index，其中有一个account的类型，则对其中_id为1的doc，它的_uid为: account#1

##### 2.2.5.2    _version

1）     在 Elasticsearch 中每个文档都有一个版本号（内部版本）。

2）     当每次对文档进行修改时（包括删除）， _version 的值会递增。

##### 2.2.5.3    _count

返回对应节点下满足条件的文档数量。

##### 2.2.5.4    op_type

查询-字符串参数（用于判断是否创建文档）

```
PUT /website/blog/123?op_type=create
{ ... }
```

 

如果创建新文档的请求成功执行，Elasticsearch 会返回元数据和一个 201 Created 的 HTTP 响应码。

另一方面，如果具有相同的 _index 、 _type 和 _id 的文档已经存在，Elasticsearch 将会返回 409 Conflict 响应码

##### 2.2.5.5    _create （用于判断是否创建文档）

```
PUT /website/blog/123/_create
{ ... }
```

 

如果创建新文档的请求成功执行，Elasticsearch 会返回元数据和一个 201 Created 的 HTTP 响应码。

另一方面，如果具有相同的 _index 、 _type 和 _id 的文档已经存在，Elasticsearch 将会返回 409 Conflict 响应码

##### 2.2.5.6    _update

update 请求最简单的一种形式是接收文档的一部分作为 doc 的参数， 它只是与现有的文档进行合并。对象被合并到一起，覆盖现有的字段，增加新的字段及内容。

```
POST /website/blog/1/_update
{
   "doc" : {
      "tags" : [ "testing" ],
      "views": 0
   }
}
```

**使用脚本部分更新文档**

```
POST /website/blog/1/_update
{
   "script" : "ctx._source.views+=1"
}
```

 

```
POST /website/blog/1/_update
{
   "script" : "ctx._source.tags+=new_tag",
   "params" : {
      "new_tag" : "search"
   }
}
```

 

**更新的文档可能不存在**

```
POST /website/pageviews/1/_update
{
   "script" : "ctx._source.views+=1",
   "upsert": {
       "views": 1
   }
}
```

 

设置参数 retry_on_conflict， 这个参数规定了失败之前 update 应该重试的次数，它的默认值为 0

```
POST /website/pageviews/1/_update?retry_on_conflict=5 
{
   "script" : "ctx._source.views+=1",
   "upsert": {
       "views": 0
   }
}
```

注：*解决更新中发生的冲突。*

##### 2.2.5.7    _mget

可以将多个请求合并成一个，加快检索速度。

```
GET /_mget
{
   "docs" : [
      {
         "_index" : "website",
         "_type" :  "blog",
         "_id" :    2
      },
      {
         "_index" : "website",
         "_type" :  "pageviews",
         "_id" :    1,
         "_source": "views"
      }
   ]
}
```



结果

 ![image-20200704000319611](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704000319611.png)

图 2—2

另一种书写方法：

```
GET /website/blog/_mget
{
   "ids" : [ "2", "1" ]
}
```

注：*即使有某个文档没有找到，上述请求的* *HTTP* *状态码仍然是* *200* *。事实上，即使请求* *没有* *找到任何文档，它的状态码依然是* *200 --**因为* *mget* *请求本身已经成功执行。* *为了确定某个文档查找是成功或者失败，你需要检查* *found* *标记。*

##### 2.2.5.8    _source

Elasticsearch 在 _source 字段存储代表文档体的JSON字符串。字段edit

如果你不需要具体文档内容，可以用下面的映射禁用 _source 字段：

PUT /my_index

{

  "mappings": {

​    "my_type": {

​      "_source": {

​        "enabled": false

​      }

​    }

  }

}

进而可以通过在请求体中指定 _source 参数，来达到只获取特定的字段的效果：

GET /_search

{

  "query":  { "match_all": {}},

  "_source": [ "title", "created" ]

}

##### 2.2.5.9    _all

_all 字段：一个把其它字段值当作一个大字符串来索引的特殊字段。

在所用字段中搜索举例：

GET /_search

{

  "match": {

​    "_all": "john smith marketing"

  }

}



不再需要 _all 字段，可以通过下面的映射来禁用：

PUT /my_index/_mapping/my_type

{

  "my_type": {

​    "_all": { "enabled": false }

  }

}

你可能想要保留 _all 字段作为一个只包含某些特定字段的全文字段，例如只包含 title。 相对于完全禁用 _all 字段，你可以为所有字段默认禁用 include_in_all 选项，仅在你选择的字段上启用：

PUT /my_index/my_type/_mapping

{

  "my_type": {

​    "include_in_all": false,

​    "properties": {

​      "title": {

​        "type":      "string",

​        "include_in_all": true

​      },

​      ...

​    }

  }

}



注：*include_in_all* *设置来逐个控制字段是否要包含在* *_all* *字段中，默认值是* *true**。*

##### 2.2.5.10  _bulk

引用实例：

POST /_bulk

{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }} 

{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}

{ "title":  "My first blog post" }

{ "index": { "_index": "website", "_type": "blog" }}

{ "title":  "My second blog post" }

{ "update": { "_index": "website", "_type": "blog", "_id": "123", "_retry_on_conflict" : 3} }

{ "doc" : {"title" : "My updated blog post"} } 



 

**注：***bulk* *请求不是原子的：* *不能用它来实现事务控制。每个请求是单独处理的，因此一个请求的成功或失败不会影响其他的请求。*

##### 2.2.5.11  _routing

在Elasticsearch中，为了支持分布式，增加了一个系统字段_routing（路由），通过_routing将Doc分发到不同的Shard，不同的Shard可以位于不同的机器上，这样就能实现简单的分布式了。

#### 2.2.6    JSON

Elasticsearch 使用 JavaScript Object Notation（或者 JSON）作为文档的序列化格式。

### 2.3      索引（version 2.X）

**索引（名词）：**

1）     存储数据到 Elasticsearch 的行为叫做 索引

2）     一个 索引 类似于传统关系数据库中的一个 数据库 ，是一个存储关系型文档的地方。 索引 (index) 的复数词为 indices 或 indexes 。

3）     一个 Elasticsearch 集群可以 包含多个索引（数据库） ，相应的每个索引可以包含多个类型（库）。 这些不同的类型存储着多个 文档（数据库中的表） ，每个文档又有多个属性（表中的属性） 。

**索引（动词）：**

1）     索引一个文档 就是存储一个文档到一个 索引 （名词）中以便被检索和查询。这非常类似于 SQL 语句中的 INSERT 关键词，除了文档已存在时，新文档会替换旧文档情况之外。

**倒排索引（**后面有倒排索引的详细讲解**）：**

关系型数据库通过增加一个 索引 比如一个 B树（B-tree）索引 到指定的列上，以便提升数据检索速度。Elasticsearch 和 Lucene 使用了一个叫做 倒排索引 的结构来达到相同的目的。

### 2.4      查询（重要）

#### 2.4.1    轻量查询

**_search**

搜索对应文档中的所有信息。

GET /megacorp/employee/_search

根据名字搜索信息

```
GET /megacorp/employee/_search?q=last_name:Smith
```

#### 2.4.2    使用查询表达式搜索

**Query-string** **搜索**

```
GET /megacorp/employee/_search
```



```
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
```

注：*产生效果与名字搜索信息相同。*

#### 2.4.3    更复杂的搜索

搜索姓氏为 Smith 的员工，但这次我们只需要年龄大于 30 的。查询需要稍作调整，使用过滤器 filter ，它支持高效地执行一个结构化查询。

```
GET /megacorp/employee/_search
{
    "query" : {
        "bool": {
            "must": {
                "match" : {
                    "last_name" : "smith" 
                }
            },
            "filter": {
                "range" : {
                    "age" : { "gt" : 30 } 
                }
            }
        }
    }
}
```



 

注：*这部分是一个* *range* *过滤器* *，* *它能找到年龄大于* *30* *的文档，其中* *gt* *表示**_**大于**_(great than)**。*

#### 2.4.4    全文搜索

搜索下所有喜欢攀岩（rock climbing）的员工：

```
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}
```

显示结果

 ![image-20200704000457409](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704000457409.png)

图 2—3

注：*蓝色**1**标注相关性得分，得分越高，则匹配率越高。*

#### 2.4.5    短语搜索

匹配同时包含 “rock” 和 “climbing” ，并且 二者以短语 “rock climbing” 的形式紧挨着的雇员记录。

```
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
```

显示结果

 ![image-20200704000700562](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704000700562.png)

图 2—4

#### 2.4.6    高亮搜索

许多应用都倾向于在每个搜索结果中高亮部分文本片段，以便让用户知道为何该文档符合查询条件。在 Elasticsearch 中检索出高亮片段也很容易。

```
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    },
    "highlight": {
        "fields" : {
            "about" : {}
        }
    }
}
```



当执行该查询时，结果中会返回一个叫做 highlight 的部分。这个部分包含了 about 属性匹配的文本片段，并以 HTML 标签 <em></em> 封装：

 ![image-20200704000713263](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704000713263.png)

图 2—5

#### 2.4.7    分析（聚合）

elasticsearch 有一个功能叫聚合（aggregations），允许我们基于数据生成一些精细的分析结果。聚合与 SQL 中的 GROUP BY 类似但更强大。

```
GET /megacorp/employee/_search
{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}
```



匹配结果是有哪些interests

 ![image-20200704000733622](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704000733622.png)

图 2—6

```
GET /megacorp/employee/_search
{
  "query": {
    "match": {
      "last_name": "smith"
    }
  },
  "aggs": {
    "all_interests": {
      "terms": {
        "field": "interests"
      }
    }
  }
}
```

all_interests 聚合已经变为只包含匹配查询的文档：

 ![image-20200704000746075](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704000746075.png)

图 2—7

**聚合还支持分级汇总（结果分组）**

比如，查询特定兴趣爱好员工的平均年龄：

```
GET /megacorp/employee/_search
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}
```

结果

 ![image-20200704000904034](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704000904034.png)

图 2—8

### 2.5      分布式特性

Elasticsearch 尽可能地屏蔽了分布式系统的复杂性。这里列举了一些在后台自动执行的操作：

l 分配文档到不同的容器或分片中，文档可以储存在一个或多个节点中。

l 按集群节点来均衡分配这些分片，从而对索引和搜索过程进行负载均衡。

l 复制每个分片以支持数据冗余，从而防止硬件故障导致的数据丢失。

l 将集群中任一节点的请求路由到存有相关数据的节点。

l 集群扩容时无缝整合新节点，重新分配分片以便从离群节点恢复。

## 3     集群原理

### 3.1      集群扩容

1）     扩容可以通过购买性能更强大（ 垂直扩容 ，或 纵向扩容 ） 或者数量更多的服务器（ 水平扩容 ，或 横向扩容 ）来实现。

2）     ElasticSearch 的主旨是随时可用和按需扩容。

3）     虽然 Elasticsearch 可以获益于更强大的硬件设备，但是垂直扩容是有极限的。 真正的扩容能力是来自于水平扩容—为集群添加更多的节点，并且将负载压力和稳定性分散到这些节点中。

### 3.2      集群健康

集群健康：在 status 字段中展示为 green 、 yellow 或者 red 。

```
GET /_cluster/health
```

返回结果：

 ![image-20200704000919850](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704000919850.png)

图 3—1

status 字段指示着当前集群在总体上是否工作正常。它的三种颜色含义如下：

l green：所有的主分片和副本分片都正常运行。

l yellow：所有的主分片都正常运行，但不是所有的副本分片都正常运行。

l red：有主分片没能正常运行。

### 3.3      节点（node）

节点与分片的关系：一个节点中有一个主分片和多个副分片。

#### 3.3.1    分片

##### 3.3.1.1    分片设置

自定义分配分片

```
PUT /blogs
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}
```

##### 3.3.1.2    分片特性

1）     索引实际上是指向一个或者多个物理分片的逻辑命名空间 。

2）     一个分片是一个底层的工作单元 ，它仅保存了全部数据中的一部分，一个分片是一个 Lucene 的实例，以及它本身就是一个完整的搜索引擎。

3）     文档被 存储和索引到分片内，应用程序是直接与索引而不是与分片进行交互。

4）     Elasticsearch 是利用分片将数据分发到集群内各处的。分片是数据的容器，文档会自动的在各节点中迁移分片保存在分片内，分片又被分配到集群内的各个节点里。 当你的集群规模扩大或者缩小时， Elasticsearch，使得数据仍然均匀分布在集群里。

5）     索引内任意一个文档都归属于一个主分片，所以主分片的数目决定着索引能够保存的最大数据量。

技术上来说，一个主分片最大能够存储 Integer.MAX_VALUE - 128 个文档，但是实际最大值还需要参考你的使用场景：包括你使用的硬件， 文档的大小和复杂程度，索引和查询文档的方式以及你期望的响应时长。

6）     一个副本分片只是一个主分片的拷贝。副本分片作为硬件故障时保护数据不丢失的冗余备份，并为搜索和返回文档等读操作提供服务。

7）     在索引建立的时候就已经确定了主分片数，但是副本分片数可以随时修改。

8）     分片是一个功能完整的搜索引擎，它拥有使用一个节点上的所有资源的能力。

#### 3.3.2    性能与安全性

##### 3.3.2.1    consistency

consistency，即一致性。在默认设置下，即使仅仅是在试图执行一个_写_操作之前，主分片都会要求必须要有规定数量(quorum)（或者换种说法，也即必须要有大多数）的分片副本处于活跃可用状态，才会去执行_写_操作(其中分片副本可以是主分片或者副本分片)。这是为了避免在发生网络分区故障（network partition）的时候进行_写_操作，进而导致数据不一致。_规定数量_即：

int( (primary + number_of_replicas) / 2 ) + 1

consistency 参数的值可以设为 one （只要主分片状态 ok 就允许执行_写_操作）,all（必须要主分片和所有副本分片的状态没问题才允许执行_写_操作）, 或 quorum 。默认值为 quorum , 即大多数的分片副本状态没问题就允许执行_写_操作。

注意，规定数量 的计算公式中 number_of_replicas 指的是在索引设置中的设定副本分片数，而不是指当前处理活动状态的副本分片数。如果你的索引设置中指定了当前索引拥有三个副本分片，那规定数量的计算结果即：

int( (primary + 3 replicas) / 2 ) + 1 = 3

如果此时你只启动两个节点，那么处于活跃状态的分片副本数量就达不到规定数量，也因此您将无法索引和删除任何文档。

##### 3.3.2.2    timeout

如果没有足够的副本分片会发生什么？ Elasticsearch会等待，希望更多的分片出现。默认情况下，它最多等待1分钟。 如果你需要，你可以使用 timeout 参数 使它更早终止： 100 100毫秒，30s 是30秒

注：*新索引默认有* *1* *个副本分片，这意味着为满足规定数量应该需要两个活动的分片副本。* *但是，这些默认的设置会阻止我们在单一节点上做任何事情。为了避免这个问题，要求只有当* *number_of_replicas* *大于**1**的时候，规定数量才会执行。*

## 4     搜索

#### 4.1.1    映射（Mapping）

描述数据在每个字段内如何存储

#### 4.1.2    分析（Analysis）

全文是如何处理使之可以被搜索的

#### 4.1.3    领域特定查询语言（Query DSL）

Elasticsearch 中强大灵活的查询语言

#### 4.1.4    多种搜索方式

**/_search**

在所有的索引中搜索所有的类型

**/gb/_search**

在 gb 索引中搜索所有的类型

**/gb,us/_search**

在 gb 和 us 索引中搜索所有的文档

**/g\*,u\*/_search**

在任何以 g 或者 u 开头的索引中搜索所有的类型

**/gb/user/_search**

在 gb 索引中搜索 user 类型

**/gb,us/user,tweet/_search**

在 gb 和 us 索引中搜索 user 和 tweet 类型

**/_all/user,tweet/_search**

在所有的索引中搜索 user 和 tweet 类型

## 5     映射和分析

### 5.1      倒排索引（重要）

#### 5.1.1    原理讲解

假设我们有两个文档，每个文档的 content 域包含如下内容：

\1.   The quick brown fox jumped over the lazy dog

\2.   Quick brown foxes leap over lazy dogs in summer

将词条规范为标准模式，可以找到与用户搜索的词条不完全一致，但具有足够相关性的文档。例如：

l Quick 和 quick 以独立的词条出现，然而用户可能认为它们是相同的词。

l fox 和 foxes 非常相似, 就像 dog 和 dogs ；他们有相同的词根。

l jumped 和 leap, 尽管没有相同的词根，但他们的意思很相近。他们是同义词。

进一步延伸

l Quick 可以小写化为 quick 。

l foxes 可以 *词干提取* --变为词根的格式-- 为 fox 。类似的， dogs 可以为提取为 dog 。

l jumped 和 leap 是同义词，可以索引为相同的单词 jump 。

现在索引看上去像这样：

 ![image-20200704000941652](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704001011544.png)

图 5—1

现在，我们进行指定单词的搜索

例1：quick brown

 ![image-20200704001011544](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704001011544.png)

图 5—2

两个文档都匹配，但是第一个文档比第二个匹配度更高。

例2：Foxes leap

 ![image-20200704001016879](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704001016879.png)

图 5—3

两个文档都匹配，但是第二个文档比第一个匹配度更高。

#### 5.1.2    不可变性

倒排索引被写入磁盘后是 不可改变 的:它永远不会修改。 不变性有重要的价值：

\1.   不需要锁。如果你从来不更新索引，你就不需要担心多进程同时修改数据的问题。

\2.   一旦索引被读入内核的文件系统缓存，便会留在哪里，由于其不变性。只要文件系统缓存中还有足够的空间，那么大部分读请求会直接请求内存，而不会命中磁盘。这提供了很大的性能提升。

\3.   其它缓存(像filter缓存)，在索引的生命周期内始终有效。它们不需要在每次数据改变时被重建，因为数据不会变化。

\4.   写入单个大的倒排索引允许数据被压缩，减少磁盘 I/O 和 需要被缓存到内存的索引的使用量。

一个不变的索引也有不好的地方。主要是它是不可变的!

### 5.2      分析与分析器

分析包含下面的过程：

1）     首先，将一块文本分成适合于倒排索引的独立的词条 

2）     之后，将这些词条统一化为标准格式以提高它们的“可搜索性”，或者 recall

#### 5.2.1    字符过滤器

首先，字符串按顺序通过每个字符过滤器 。他们的任务是在分词前整理字符串。一个字符过滤器可以用来去掉HTML，或者将 & 转化成 and。

#### 5.2.2    分词器

其次，字符串被分词器分为单个的词条。一个简单的分词器遇到空格、tabs、换行符等等的时候，可能会将文本拆分成词条。

#### 5.2.3    Token 过滤器

最后，词条按顺序通过每个 token 过滤器 。这个过程可能会改变词条（例如，小写化 Quick ），删除词条（例如， 像 a， and， the 等无用词），或者增加词条（例如，像 jump 和 leap 这种同义词）。

 

#### 5.2.4    内置分析器

词条 "Set the shape to semi-transparent by calling set_trans(5)"

##### 5.2.4.1    标准分析器

标准分析器是Elasticsearch默认使用的分析器。它是分析各种语言文本最常用的选择。它根据 Unicode 联盟 定义的 单词边界 划分文本。删除绝大部分标点。最后，将词条小写。它会产生

 

set, the, shape, to, semi, transparent, by, calling, set_trans, 5

##### 5.2.4.2    简单分析器

简单分析器在任何不是字母的地方分隔文本，将词条小写。它会产生

set, the, shape, to, semi, transparent, by, calling, set, trans

##### 5.2.4.3    空格分析器

空格分析器在空格的地方划分文本。它会产生

Set, the, shape, to, semi-transparent, by, calling, set_trans(5)

##### 5.2.4.4    语言分析器

特定语言分析器可用于很多语言。它们可以考虑指定语言的特点。例如， 英语 分析器附带了一组英语无用词（常用单词，例如 and 或者 the ，它们对相关性没有多少影响），它们会被删除。 由于理解英语语法的规则，这个分词器可以提取英语单词的 词干 。

英语 分词器会产生下面的词条：

set, shape, semi, transpar, call, set_tran, 5

注意看 transparent、 calling 和 set_trans 已经变为词根格式。

#### 5.2.5    测试分析器

analyze API 看文本是如何被分析的，在消息体里，指定分析器和要分析的文本：

```
GET /_analyze
{
  "analyzer": "standard",
  "text": "Text to analyze"
```



```
}
```



结果：

 ![image-20200704001034181](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704001034181.png)

图 5—4

**注**：token 是实际存储到索引中的词条。 position 指明词条在原始文本中出现的位置。 start_offset 和 end_offset 指明字符在原始字符串中的位置。

### 5.3      什么时候使用分析器

1）     当你查询一个 全文 域时， 会对查询字符串应用相同的分析器，以产生正确的搜索词条列表。

2）     当你查询一个 精确值 域时，不会分析查询字符串，而是搜索你指定的精确值。

3）      

```
GET /_search?q=2014              # 12 results
GET /_search?q=2014-09-15        # 12 results !
GET /_search?q=date:2014-09-15   # 1  result
GET /_search?q=date:2014         # 0  results !
```

（1） date 域包含一个精确值：单独的词条 2014-09-15。

（2） _all 域是一个全文域，所以分词进程将日期转化为三个词条： 2014， 09， 和 15。

 

### 5.4      映射

#### 5.4.1    核心简单域类型

字符串: string

整数 : byte, short, integer, long

浮点数: float, double

布尔型: boolean

日期: date

#### 5.4.2    自定义域映射

##### 5.4.2.1    index

index 属性控制怎样索引字符串。它可以是下面三个值：

**analyzed****：**首先分析字符串，然后索引它。换句话说，以全文索引这个域。

**not_analyzed****：**索引这个域，所以它能够被搜索，但索引的是精确值。不会对它进行分析。

**no****：**不索引这个域。这个域不会被搜索到。

string 域 index 属性默认是 analyzed 。如果我们想映射这个字段为一个精确值，我们需要设置它为 not_analyzed ：

```
PUT /gb/_mapping/tweet
{
  "properties" : {
    "tag" : {
      "type" :    "string",
      "index":    "not_analyzed"
    }
  }
}
```



使用自定义域类型

```
GET /gb/_analyze
{
  "field": "tag",
  "text": "Black-cats" 
}
```

重点：在5.0以后，string类型的not_analyzed被keyword类型代替了，而string方法的analyzed被text类型替换了。

 

### 5.5      复杂核心域类型

#### 5.5.1    多值域

tag 域包含多个标签。我们可以以数组的形式索引标签：

{ "tag": [ "search", "nosql" ]}

 

#### 5.5.2    空域

当然，数组可以为空。这相当于存在零值。 事实上，在 Lucene 中是不能存储 null 值的，所以我们认为存在 null 值的域为空域。

 

下面三种域被认为是空的，它们将不会被索引：

"null_value":        null,

"empty_array":       [],

"array_with_null_value":  [ null ]

 

#### 5.5.3    多层级对象

内部对象 经常用于嵌入一个实体或对象到其它对象中。例如，与其在 tweet 文档中包含 user_name 和 user_id 域，我们也可以这样写：

{

  "tweet":      "Elasticsearch is very flexible",

  "user": {

​    "id":      "@johnsmith",

​    "gender":    "male",

​    "age":     26,

​    "name": {

​      "full":   "John Smith",

​      "first":  "John",

​      "last":   "Smith"

​    }

  }

}

 

#### 5.5.4    内部对象的映射

Elasticsearch 会动态监测新的对象域并映射它们为 对象 ，在 properties 属性下列出内部域：

{

 "gb": {

  "tweet": { 

   "properties": {

​    "tweet":      { "type": "string" },

​    "user": { 

​     "type":       "object",

​     "properties": {

​      "id":      { "type": "string" },

​      "gender":    { "type": "string" },

​      "age":     { "type": "long"  },

​      "name":  { 

​       "type":     "object",

​       "properties": {

​        "full":   { "type": "string" },

​        "first":  { "type": "string" },

​        "last":   { "type": "string" }

​       }

​      }

​     }

​    }

   }

  }

 }

}

user 和 name 域的映射结构与 tweet 类型的相同。事实上， type 映射只是一种特殊的 对象 映射，我们称之为 根对象

 

#### 5.5.5    内部对象是如何索引的

为了能让 Elasticsearch 有效地索引内部类，它把我们的文档转化成这样：

{

  "tweet":      [elasticsearch, flexible, very],

  "user.id":     [@johnsmith],

  "user.gender":   [male],

  "user.age":     [26],

  "user.name.full":  [john, smith],

  "user.name.first": [john],

  "user.name.last":  [smith]

}

 

#### 5.5.6    内部对象数组

最后，考虑包含内部对象的数组是如何被索引的。 假设我们有个 followers 数组：

{

  "followers": [

​    { "age": 35, "name": "Mary White"},

​    { "age": 26, "name": "Alex Jones"},

​    { "age": 19, "name": "Lisa Smith"}

  ]

}

这个文档会像我们之前描述的那样被扁平化处理，结果如下所示：

{

  "followers.age":  [19, 26, 35],

  "followers.name":  [alex, jones, lisa, smith, mary, white]

}

 

## 6     请求体查询

### 6.1      空查询

空查询的三种方式

**第一种**

让我们以最简单的 search API 的形式开启我们的旅程，空查询将返回所有索引库（indices)中的所有文档：

```
GET /_search
{} 
```

注：

\1.   空的大括号代表空的请求体

\2.   _search只显示前十个查询结果

**第二种**

只用一个查询字符串，你就可以在一个、多个或者 _all 索引库（indices）和一个、多个或者所有types中查询：

```
GET /index_2014*/type1,type2/_search
{}
```

**第三种**

同时你可以使用 from 和 size 参数来分页：

```
GET /_search
{
  "from": 30,
  "size": 10
}
```

 

### 6.2      查询表达式(Query DSL)

#### 6.2.1    空查询的本质

空查询（empty search） —{}— 在功能上等价于使用 match_all 查询， 正如其名字一样，匹配所有文档：

```
GET /_search
{
    "query": {
        "match_all": {}
    }
}
```

 

#### 6.2.2    完整的query查询

```
GET /_search
{
    "query": {
        "match": {
            "tweet": "elasticsearch"
        }
    }
}
```

 

### 6.3      查询与过滤

Elasticsearch 使用的查询语言（DSL）拥有一套查询组件，这套组件在两种情况下使用：过滤情况（filtering context）和查询情况（query context）。

#### 6.3.1    过滤情况

当使用于过滤情况时，查询被设置成一个“不评分”或者“过滤”查询。即这个查询只是简单的问一个问题：“这篇文档是否匹配？”。回答也是非常的简单，yes 或者 no ，二者必居其一。

例：

l created 时间是否在 2013 与 2014 这个区间？

l status 字段是否包含 published 这个单词？

l lat_lon 字段表示的位置是否在指定点的 10km 范围内？

#### 6.3.2    查询情况

当使用于 查询情况 时，查询就变成了一个“评分”的查询。和不评分的查询类似，也要去判断这个文档是否匹配，同时它还需要判断这个文档匹配的有多好（匹配程度如何），同时将这个相关程度分配给表示相关性的字段 _score。 此查询的典型用法是用于查找以下文档：

l 查找与 full text search 这个词语最佳匹配的文档

l 包含 run 这个词，也能匹配 runs   、 running 、 jog 或者 sprint

l 包含 quick 、 brown 和 fox 这几个词 — 词之间离的越近，文档相关性越高

l 标有 lucene 、 search 或者 java 标签 — 标签越多，相关性越高

#### 6.3.3    两者性能差异

\1.   过滤查询（Filtering queries）只是简单的检查包含或者排除，这就使得计算起来非常快，结果会被缓存到内存中以便快速读取。

\2.   评分查询（scoring queries）不仅仅要找出匹配的文档，还要计算每个匹配文档的相关性，计算相关性使得它们比不评分查询费力的多。同时，查询结果并不缓存。

### 6.4      主要的查询方式

完整格式如下，本小节所有举例以省略部分代码

```
GET /website/blog/_search
{
  "query" : { 
      "multi_match": {
        "query":    "Smith",
        "fields":   [ "last_name", "about" ]
      }
    }
}
```

 

#### 6.4.1    match_all 查询

match_all 查询简单的匹配所有文档。在没有指定查询方式时，它是默认的查询：

```
{ "match_all": {}}
```

#### 6.4.2    match 查询

match 查询，在执行查询前，它将用正确的分析器去分析查询字符串：

```
{ "match": { "tweet": "About Search" }}
```

#### 6.4.3    multi_match 查询

multi_match 查询可以在多个字段上执行相同的 match 查询：

```
{
    "multi_match": {
        "query":    "full text search",
        "fields":   [ "title", "body" ]
    }
}
```

#### 6.4.4    range 查询

range 查询找出那些落在指定区间内的数字或者时间：

```
{
    "range": {
        "age": {
            "gte":  20,
            "lt":   30
        }
    }
}
```

被允许的操作符如下：

 gt-------大于

 gte-----大于等于

 lt-------小于

 lte------小于等于

#### 6.4.5    term 查询

term 查询被用于精确值匹配，这些精确值可能是数字、时间、布尔或者那些 not_analyzed 的字符串：

```
{ "term": { "age":    26           }}
```

注：*term* *查询对于输入的文本不分析* *，所以它将给定的值进行精确查询。*

#### 6.4.6    terms 查询

terms 查询和 term 查询一样，但它允许你指定多值进行匹配。如果这个字段包含了指定值中的任何一个值，那么这个文档满足条件：

```
{ "terms": { "tag": [ "search", "full_text", "nosql" ] }}
```

注：*和* *term* *查询一样，**terms* *查询对于输入的文本不分析。它查询那些精确匹配的值（包括在大小写、重音、空格等方面的差异）。*

#### 6.4.7    exists 查询和 missing 查询

exists 查询和 missing 查询被用于查找那些指定字段中有值 (exists) 或无值 (missing) 的文档。这与SQL中的 IS_NULL (missing) 和 NOT IS_NULL (exists) 在本质上具有共性：

```
{
    "exists":   {
        "field":    "title"
    }
}
```

这些查询经常用于某个字段有值的情况和某个字段缺值的情况，结果显示包含这个属性的整个文档内容。

### 6.5      组合多查询

#### 6.5.1    bool 查询

bool 查询来实现你的需求。这种查询将多查询组合在一起，成为用户自己想要的布尔查询。它接收以下参数：

**1.**   **must****：**文档 必须 匹配这些条件才能被包含进来。

**2.**   **must_not****：**文档 必须不 匹配这些条件才能被包含进来。

**3.**   **should**：如果满足这些语句中的任意语句，将增加 _score ，否则，无任何影响。它们主要用于修正每个文档的相关性得分。

**4.**   **filter**：必须 匹配，但它以不评分、过滤模式来进行。这些语句对评分没有贡献，只是根据过滤标准来排除或包含文档。

```
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }},
            { "range": { "date": { "gte": "2014-01-01" }}}
        ]
    }
}
```

注：*如果没有* *must* *语句，那么至少需要能够匹配其中的一条* *should* *语句。如果存在至少一条* *must* *语句，则对* *should* *语句的匹配没有要求。*

#### 6.5.2    bool查询增加带过滤器（filtering）

如果我们不想因为文档的时间而影响得分，可以用 filter 语句来重写前面的例子：

```
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }}
        ],
        "filter": {
          "range": { "date": { "gte": "2014-01-01" }} 
        }
    }
}
```

range 查询已经从 should 语句中移到 filter 语句

**优点：**通过将 range 查询移到 filter 语句中，我们将它转成不评分的查询，提升性能。

 

如果你需要通过多个不同的标准来过滤你的文档，bool 查询本身也可以被用做不评分的查询。简单地将它放置到 filter 语句中并在内部构建布尔逻辑：

```
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }}
        ],
        "filter": {
          "bool": { 
              "must": [
                  { "range": { "date": { "gte": "2014-01-01" }}},
                  { "range": { "price": { "lte": 29.99 }}}
              ],
              "must_not": [
                  { "term": { "category": "ebooks" }}
              ]
          }
        }
    }
}
```

#### 6.5.3    constant_score 查询

它被经常用于你只需要执行一个 filter 而没有其它查询（例如，评分查询）的情况下。可以使用它来取代只有 filter 语句的 bool 查询。在性能上是完全相同的。

```
{
    "constant_score":   {
        "filter": {
            "term": { "category": "ebooks" } 
        }
    }
}
```

#### 6.5.4    validate-query验证查询合法性

validate-query API 可以用来验证查询是否合法

```
GET /website/blog/_validate/query
{
   "query": {
      "matcsh" : {
         " tweet" : "really powerful"
      }
   }
}
```

结果

 ![image-20200704001148687](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704001148687.png)

图 6—1

#### 6.5.5    explain理解错误信息

在validate-query之后加上explain，可以显示错误信息

```
GET /website/blog/_validate/query?explain 
{
   "query": {
      "about" : {
         "match" : "really powerful"
      }
   }
}
```

结果

 ![image-20200704001206579](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704001206579.png)

图 6—2

查询合法举例：

```
GET /website/blog/_validate/query?explain 
{
   "query": {
      "match" : {
         "about" : "love"
      }
   }
}
```

结果

 ![image-20200704001213335](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704001213335.png)

图 6—3

## 7     排序与相关性

### 7.1      排序

#### 7.1.1    按照相关性排序

为了按照相关性来排序，需要将相关性表示为一个数值。在 Elasticsearch 中，相关性得分由一个浮点数进行表示，并在搜索结果中通过 _score 参数返回，默认排序是 _score 降序。

#### 7.1.2    按照字段的值排序

在这个案例中，通过获取tweets创建的时间，最新创建的 tweets 排在最前。 我们可以使用 sort 参数进行实现：

```
GET /_search
{
    "query" : {
        "bool" : {
            "filter" : { "term" : { "user_id" : 1 }}
        }
    },
    "sort": { "date": { "order": "desc" }}
}
```

结果

 ![image-20200704001238077](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704001250534.png)

图 7—1

注：

\1.   _score 不被计算, 因为它并没有用于排序。

\2.   date 字段的值表示为自 epoch (January 1, 1970 00:00:00 UTC)以来的毫秒数，通过 sort 字段的值进行返回。

#### 7.1.3    多级排序

假定我们想要结合使用 date 和 _score 进行查询，并且匹配的结果首先按照日期排序，然后按照相关性排序：

```
GET /website/blog/_search
{
    "query" : {
        "bool" : {
            "must":   { "match": { "last_name": "Smith" }},
            "filter" : { "term" : { "_id" : 123 }}
        }
    },
    "sort": [
      { "age": { "order": "desc" }},
      {"_score": { "order": "desc" }}
    ]
}
```

结果

 ![image-20200704001250534](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704001250534.png)

图 7—2

#### 7.1.4    多值字段的排序

一种情形是字段有多个值的排序， 需要记住这些值并没有固有的顺序；一个多值的字段仅仅是多个值的包装，这时应该选择哪个进行排序呢？

对于数字或日期，你可以将多值字段减为单值，这可以通过使用 min 、 max 、 avg 或是 sum 排序模式 。 例如你可以按照每个 date 字段中的最早日期进行排序，通过以下方法：

```
"sort": {
    "dates": {
        "order": "asc",
        "mode":  "min"
    }
}
```

### 7.2      字符串排序与多字段

被解析的字符串字段也是多值字段， 但是很少会按照你想要的方式进行排序。如 fine old art ， 这包含 3 项。我们很可能想要按第一项的字母排序，然后按第二项的字母排序，诸如此类，但是 Elasticsearch 在排序过程中没有这样的信息。

可以为所有的 _core_field 类型 (strings, numbers, Booleans, dates) 接收一个 fields 参数

该参数允许你转化一个简单的映射如：

```
"tweet": {
    "type":     "string",
    "analyzer": "english"
}
```

为一个多字段映射如：

```
"tweet": { 
    "type":     "string",
    "analyzer": "english",
    "fields": {
        "raw": { 
            "type":  "string",
            "index": "not_analyzed"
        }
    }
}
```

注：

*1.*    *tweet* *主字段与之前的一样**:* *是一个* *analyzed* *全文字段。*

*2.*   *新的* *tweet.raw* *子字段是* *not_analyzed.*

现在，只要我们重新索引了我们的数据，使用 tweet 字段用于搜索，tweet.raw 字段用于排序：

```
GET /_search
{
    "query": {
        "match": {
            "tweet": "elasticsearch"
        }
    },
    "sort": "tweet.raw"
}
```

### 7.3      什么是相关性?

#### 7.3.1    相关性概述

每个文档都有相关性评分，用一个正浮点数字段 _score 来表示 。 _score 的评分越高，相关性越高。

评分的计算方式取决于查询类型 不同的查询语句用于不同的目的： 

\1.   fuzzy 查询会计算与关键词的拼写相似程度。

\2.   terms 查询会计算找到的内容与关键词组成部分匹配的百分比。

\3.   通常我们说的 relevance 是我们用来计算全文本字段的值相对于全文本检索词相似程度的算法。

Elasticsearch 的相似度算法被定义为检索词频率/反向文档频率， TF/IDF ，包括以下内容：

##### 7.3.1.1    检索词频率

检索词在该字段出现的频率？出现频率越高，相关性也越高。 字段中出现过 5 次要比只出现过 1 次的相关性高。

词频的计算方式如下：

tf(t in d) = √frequency

注：*词* *t* *在文档* *d* *的词频（* *tf* *）是该词在文档中出现次数的平方根。*



在字段映射中禁用词频统计：

```
PUT /my_index
{
  "mappings": {
    "doc": {
      "properties": {
        "text": {
          "type":          "string",
          "index_options": "docs" 
        }
      }
    }
  }
}
```



注：*将参数* *index_options* *设置为* *docs* *可以禁用词频统计及词频位置，这个映射的字段不会计算词的出现次数，对于短语或近似查询也不可用。要求精确查询的* *not_analyzed* *字符串字段会默认使用该设置。*

 

##### 7.3.1.2    反向文档频率

又叫逆向文档率，每个检索词在索引中出现的频率？频率越高，相关性越低。检索词出现在多数文档中会比出现在少数文档中的权重更低。

逆向文档频率的计算公式如下：

idf(t) = 1 + log ( numDocs / (docFreq + 1))

注：*词* *t* *的逆向文档频率（* *idf* *）是：索引中文档数量除以所有包含该词的文档数，然后求其对数。*

##### 7.3.1.3    字段长度准则

又叫字段长度归一值，字段的长度是多少？长度越长，相关性越低。 检索词出现在一个短的 title 要比同样的词出现在一个长的 content 字段权重更大。

字段长度的归一值公式如下：

norm(d) = 1 / √numTerms

注：*字段长度归一值（* *norm* *）是字段中词数平方根的倒数。*

##### 7.3.1.4    向量空间模型

设想如果查询 “happy hippopotamus” ，常见词 happy 的权重较低，不常见词 hippopotamus 权重较高，假设 happy 的权重是 2 ， hippopotamus 的权重是 5 ，可以将这个二维向量—— [2,5] ——在坐标系下作条直线，线的起点是 (0,0) 终点是 (2,5)

 ![image-20200704001326184](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704001326184.png)

图 7—3

现在，设想我们有三个文档：

l I am happy in summer 。

l After Christmas I’m a hippopotamus 。

l The happy hippopotamus helped Harry 。

文档 1： (happy,____________) —— [2,0]

文档 2： ( ___ ,hippopotamus) —— [0,5]

文档 3： (happy,hippopotamus) —— [2,5]

 ![image-20200704001339510](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704001339510.png)

图 7—4

向量之间是可以比较的，只要测量查询向量和文档向量之间的角度就可以得到每个文档的相关度，文档 1 与查询之间的角度最大，所以相关度低；文档 2 与查询间的角度较小，所以更相关；文档 3 与查询的角度正好吻合，完全匹配。

#### 7.3.2    理解文档是如何被匹配到的

```
GET /website/blog/66grLnABnwFX5xMmNfpB/_explain
{
   "query" : {
      "bool" : {
         "must" :  { "match" : { "title" :   "second" }}
      }
   }
}
 
```

结果

 ![image-20200704001351589](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704001351589.png)

图 7—5

### 7.4      Doc Values 介绍

在 Elasticsearch 中，Doc Values 就是一种列式存储结构，默认情况下每个字段的 Doc Values 都是激活的，Doc Values 是在索引时创建的，当字段索引时，Elasticsearch 为了能够快速检索，会把字段的值加入倒排索引中，同时它也会存储该字段的 Doc Values。

Elasticsearch 默认给 大多数 字段启用 doc values，所以在一些搜索场景大大的节省了内存使用量，但是需要注意的是只有不分词的 string 类型的字段才能使用这种特性。

Elasticsearch 中的 Doc Values 常被应用到以下场景：

l 对一个字段进行排序

l 对一个字段进行聚合

l 某些过滤，比如地理位置过滤

l 某些与字段相关的脚本计算

普通的倒排索引，搜索brown

 ![image-20200704001405077](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704001405077.png)

图 7—6

对于聚合部分，我们需要找到 Doc_1 和 Doc_2 里所有唯一的词项。 用倒排索引做这件事情代价很高： 我们会迭代索引里的每个词项并收集 Doc_1 和 Doc_2 列里面 token。这很慢而且难以扩展：随着词项和文档的数量增加，执行时间也会增加。

Doc values 通过转置两者间的关系来解决这个问题。倒排索引将词项映射到包含它们的文档，doc values 将文档映射到它们包含的词项，如下：

 ![image-20200704001416090](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704001416090.png)

图 7—7

当数据被转置之后，想要收集到 Doc_1 和 Doc_2 的唯一 token 会非常容易。获得每个文档行，获取所有的词项，然后求两个集合的并集。

 

 

 

## 8     执行分布式检索

### 8.1      在分片上索引和 文档

在主副上成功创建文档后，索引和删除文档所需要的步骤顺序：

 ![image-20200704001621314](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704001621314.png)

图 8—1

1）     客户端向 Node 1 发送新建、索引或者删除请求。

2）     节点使用文档的 _id 确定文档属于分片 0 。请求会被转发到 Node 3，因为分片 0 的主分片目前被分配在 Node 3 上。

3）     Node 3 在主分片上面执行请求。如果成功了，它将请求并行转发到 Node 1 和 Node 2 的副本分片上。一旦所有的副本分片都报告成功, Node 3 将向协调节点报告成功，协调节点向客户端报告成功。

### 8.2      取回一个文档

 ![image-20200704001630991](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704001630991.png)

图 8—2

\1.   协调节点辨别出哪些文档需要被取回并向相关的分片提交多个 GET 请求。

\2.   每个分片加载并 丰富 文档，如果有需要的话，接着返回文档给协调节点。

\3.   一旦所有的文档都被取回了，协调节点返回结果给客户端。

### 8.3      局部更新文档

 ![image-20200704001637124](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704001637124.png)

图 8—3

\1.   客户端向 Node 1 发送更新请求。

\2.   它将请求转发到主分片所在的 Node 3 。

\3.   Node 3 从主分片检索文档，修改 _source 字段中的 JSON ，并且尝试重新索引主分片的文档。 如果文档已经被另一个进程修改，它会重试步骤 3 ，超过 retry_on_conflict 次后放弃。

\4.   如果 Node 3 成功地更新文档，它将新版本的文档并行转发到 Node 1 和 Node 2 上的副本分片，重新建立索引。 一旦所有副本分片都返回成功， Node 3 向协调节点也返回成功，协调节点向客户端返回成功。

### 8.4      多文档模式

#### 8.4.1    使用 mget 取回多个文档

 ![image-20200704001646206](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704001646206.png)

图 8—4

\1.   客户端向 Node 1 发送 mget 请求。

\2.   Node 1 为每个分片构建多文档获取请求，然后并行转发这些请求到托管在每个所需的主分片或者副本分片的节点上。一旦收到所有答复， Node 1 构建响应并将其返回给客户端。

 

#### 8.4.2    使用 bulk 修改多个文档

 ![image-20200704001651701](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704001651701.png)

图 8—5

 

bulk API 按如下步骤顺序执行：；

1）     客户端向 Node 1 发送 bulk 请求。

2）     Node 1 为每个节点创建一个批量请求，并将这些请求并行转发到每个包含主分片的节点主机。

3）     主分片一个接一个按顺序执行每个操作。当每个操作成功时，主分片并行转发新文档（或删除）到副本分片，然后执行下一个操作。 一旦所有的副本分片报告所有操作成功，该节点将向协调节点报告成功，协调节点将这些响应收集整理并返回给客户端。

bulk API 还可以在整个批量请求的最顶层使用 consistency 参数，以及在每个请求中的元数据中使用 routing 参数。

 

### 8.5      查询过程分布式搜索

 ![image-20200704001658122](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200704001658122.png)

图 8—6

\1.   客户端发送一个 search 请求到 Node 3 ， Node 3 会创建一个大小为 from + size 的空优先队列。

\2.   Node 3 将查询请求转发到索引的每个主分片或副本分片中。每个分片在本地执行查询并添加结果到大小为 from + size 的本地有序优先队列中。

\3.   每个分片返回各自优先队列中所有文档的 ID 和排序值给协调节点，也就是 Node 3 ，它合并这些值到自己的优先队列中来产生一个全局排序后的结果列表。

### 8.6      游标查询 Scroll

\1.   scroll 查询 可以用来对 Elasticsearch 有效地执行大批量的文档查询。

\2.   游标查询允许我们 先做查询初始化，然后再批量地拉取结果。

\3.   游标查询用字段 _doc 来排序。 这个指令让 Elasticsearch 仅仅从还有结果的分片返回下一批结果

```
GET /old_index/_search?scroll=1m 
{
    "query": { "match_all": {}},
    "sort" : ["_doc"], 
    "size":  1000
}
```

注：

*1.*    *1m**保持游标查询窗口一分钟。*

*2.*   *关键字* *_doc* *是最有效的排序顺序。*

这个查询的返回结果包括一个字段 _scroll_id， 它是一个base64编码的长字符串 。 现在我们能传递字段 _scroll_id 到 _search/scroll 查询接口获取下一批结果：

```
GET /_search/scroll
{
    "scroll": "1m", 
    "scroll_id" : "cXVlcnlUaGVuRmV0Y2g7NTsxMDk5NDpkUmpiR2FjOFNhNnlCM1ZDMWpWYnRROzEwOTk1OmRSamJHYWM4U2E2eUIzVkMxalZidFE7MTA5OTM6ZFJqYkdhYzhTYTZ5QjNWQzFqVmJ0UTsxMTE5MDpBVUtwN2lxc1FLZV8yRGVjWlI2QUVBOzEwOTk2OmRSamJHYWM4U2E2eUIzVkMxalZidFE7MDs="
}
```

## 9     索引管理

### 9.1      创建一个索引

PUT /my_index

{

   "settings": { ... any settings ... },

  "mappings": {

​    "type_one": { ... any mappings ... },

​    "type_two": { ... any mappings ... },

​    ...

  }

}

如果需要禁止自动创建索引，可以通过在 config/elasticsearch.yml 的每个节点下添加下面的配置：

action.auto_create_index: false

### 9.2      删除一个索引

用以下的请求来 删除索引:

DELETE /my_index

也可以这样删除多个索引：

DELETE /index_one,index_two

DELETE /index_*

甚至可以这样删除 全部 索引：

DELETE /_all

DELETE /*

注：

有些时候，用单个命令来删除所有数据可能会导致可怕的后果。如果你想要避免意外的大量删除, 你可以在你的 elasticsearch.yml 做如下配置：

action.destructive_requires_name: true

### 9.3      索引设置

**number_of_shards****：**每个索引的主分片数，默认值是 5 。这个配置在索引创建后不能修改。

**number_of_replicas****：**每个主分片的副本数，默认值是 1 。对于活动的索引库，这个配置可以随时修改。

创建只有一个主分片，没有副本的小索引：

PUT /my_temp_index

{

  "settings": {

​    "number_of_shards" :  1,

​    "number_of_replicas" : 0

  }

}

用 update-index-settings API 动态修改副本数：

PUT /my_temp_index/_settings

{

  "number_of_replicas": 1

}

### 9.4      配置分析器

standard 分析器是用于全文字段的默认分析器，对于大部分西方语系来说是一个不错的选择。 它包括了以下几点：

l standard 分词器，通过单词边界分割输入的文本。

l standard 语汇单元过滤器，目的是整理分词器触发的语汇单元（但是目前什么都没做）。

l lowercase 语汇单元过滤器，转换所有的语汇单元为小写。

l stop 语汇单元过滤器，删除停用词—对搜索相关性影响不大的常用词，如 a ， the ， and ， is 。

#### 9.4.1    停用词过滤器

创建了一个新的分析器，叫做 es_std ， 并使用预定义的西班牙语停用词列表：

PUT /spanish_docs

{

  "settings": {

​    "analysis": {

​      "analyzer": {

​        "es_std": {

​          "type":   "standard",

​          "stopwords": "_spanish_"

​        }

​      }

​    }

  }

}

### 9.5      创建一个自定义分析器

在 analysis 下的相应位置设置字符过滤器、分词器和词单元过滤器:

PUT /my_index

{

  "settings": {

​    "analysis": {

​      "char_filter": {

​        "&_to_and": {

​          "type":    "mapping",

​          "mappings": [ "&=> and "]

​      }},

​      "filter": {

​        "my_stopwords": {

​          "type":    "stop",

​          "stopwords": [ "the", "a" ]

​      }},

​      "analyzer": {

​        "my_analyzer": {

​          "type":     "custom",

​           "char_filter": [ "html_strip", "&_to_and" ],

​          "tokenizer":  "standard",

​          "filter":    [ "lowercase", "my_stopwords" ]

​      }}

}}}

注：

*1.*   *char_filter**中**&_to_and**：将**&* *替换为* *" and "*

*2.*   *filter**中**my_stopwords**：移除自定义的停止词列表中包含的词*

*3.*   *analyzer* *中**custom**：定制*

*4.*   *analyzer* *中**html_strip**：移除掉所有的**HTML**标签*

*5.*   *analyzer* *中**standard**：使用标准的分词器*

*6.*   *analyzer* *中**lowercase**：将大写字母转换成小写*

可以把上面定义的分析器应用在一个 string 字段上：

PUT /my_index/_mapping/my_type

{

  "properties": {

​    "title": {

​      "type":   "string",

​      "analyzer": "my_analyzer"

​    }

  }

}

### 9.6      类型和映射

#### 9.6.1    什么是类型

类型 在 Elasticsearch 中表示一类相似的文档。 类型由 名称 —比如 user 或 blogpost —和 映射 组成。

#### 9.6.2    什么是映射

映射描述了文档可能具有的字段或 属性 、每个字段的数据类型—比如 string, integer 或 date —以及Lucene是如何索引和存储这些字段的。

#### 9.6.3    类型陷阱

**思考：**在同一个索引中，如果有两个不同的类型，每个类型都有同名的字段，但映射不同（例如：一个是字符串一个是数字），将会出现什么情况？

**答案：**Elasticsearch 不会允许你定义这个映射。当你配置这个映射时，将会出现异常。

以data 索引中两种类型的映射为例：

 ![image-20200704002642672](../../../Typora/Picture/image-20200704002642672.png)

图 9—1

每个类型定义两个字段 (分别是 "name"/"address" 和 "timestamp"/"message" )。它们看起来是相互独立的，但在后台 Lucene 将创建一个映射，如:

 ![image-20200704002648261](../../../Typora/Picture/image-20200704002648261.png)

图 9—2

对于整个索引，映射在本质上被 扁平化 成一个单一的、全局的模式。这就是为什么两个类型不能定义冲突的字段：当映射被扁平化时，Lucene 不知道如何去处理。

### 9.7      动态映射

当 Elasticsearch 遇到文档中以前 未遇到的字段，它用 dynamic mapping 来确定字段的数据类型并自动把新的字段添加到类型映射。

#### 9.7.1    dynamic 配置

三种配置方式：

\1.   true动态添加新的字段—缺省

\2.   false忽略新的字段

\3.   strict如果遇到新字段抛出异常

举例：

PUT /my_index

{

  "mappings": {

​    "my_type": {

​      "dynamic":   "strict", 

​      "properties": {

​        "title": { "type": "string"},

​        "stash": {

​          "type":   "object",

​          "dynamic": true 

​        }

​      }

​    }

  }

}

注：

*1.*   *dynamic-strict**：如果遇到新字段，对象* *my_type* *就会抛出异常。*

*2.*   *dynamic-true**：* *而内部对象* *stash* *遇到新字段就会动态创建新字段。*

#### 9.7.2    自定义动态映射

##### 9.7.2.1    日期检测

当 Elasticsearch 遇到一个新的字符串字段时，它会检测这个字段是否包含一个可识别的日期，比如 2014-01-01 。如果它像日期，这个字段就会被作为 date 类型添加。否则，它会被作为 string 类型添加。

关闭日期自动检测，可以通过在根对象上设置 date_detection 为 false 来关闭：

PUT /my_index

{

  "mappings": {

​    "my_type": {

​      "date_detection": false

​    }

  }

}

##### 9.7.2.2    动态模板

例如，我们给 string 类型字段定义两个模板：

l es ：以 _es 结尾的字段名需要使用 spanish 分词器。

l en ：所有其他字段使用 english 分词器。

PUT /my_index

{

  "mappings": {

​    "my_type": {

​      "dynamic_templates": [

​        { "es": {

​           "match":       "*_es", 

​           "match_mapping_type": "string",

​           "mapping": {

​             "type":      "string",

​             "analyzer":    "spanish"

​           }

​        }},

​        { "en": {

​           "match":       "*", 

​           "match_mapping_type": "string",

​           "mapping": {

​             "type":      "string",

​             "analyzer":    "english"

​           }

​        }}

​      ]

}}}

注：

*1.*   *match_mapping_type* *应用模板到特定类型的字段上，如上面例子中应用到**String**类型上*

*2.*   *match* *参数只匹配字段名称，* *path_match* *参数匹配字段在对象上的完整路径*

### 9.8      缺省映射

使用 _default_ 映射为所有的类型禁用 _all 字段， 而只在 blog 类型启用：

PUT /my_index

{

  "mappings": {

​    "_default_": {

​      "_all": { "enabled": false }

​    },

​    "blog": {

​      "_all": { "enabled": true }

​    }

  }

}



 

### 9.9      重新索引和索引别名

#### 9.9.1    索引别名

两种方式管理别名： _alias 用于单个操作， _aliases 用于执行多个原子级操作。

创建索引 my_index_v1 ，然后将别名 my_index 指向它：

PUT /my_index_v1 

PUT /my_index_v1/_alias/my_index 

注：

*1.*   *创建索引* *my_index_v1**。*

*2.*   *设置别名* *my_index* *指向* *my_index_v1* *。*

可以检测这个别名指向哪一个索引：

GET /*/_alias/my_index

或哪些别名指向这个索引：

GET /my_index_v1/_alias/*

两个返回结果相同：

 ![image-20200704002713341](../../../Typora/Picture/image-20200704002713341.png)

图 9—3

#### 9.9.2    重新索引

一个别名可以指向多个索引，所以我们在添加别名到新索引的同时必须从旧的索引中删除它。这个操作需要原子化，这意味着我们需要使用 _aliases 操作：

POST /_aliases

{

  "actions": [

​    { "remove": { "index": "my_index_v1", "alias": "my_index" }},

​    { "add":  { "index": "my_index_v2", "alias": "my_index" }}

  ]

}

## 10   分片内部原理

### 10.1   使文本可被搜索

倒排索引相比特定词项出现过的文档列表，会包含更多其它信息。它会保存每一个词项出现过的文档总数， 在对应的文档中一个具体词项出现的总次数，词项在文档中的顺序，每个文档的长度，所有文档的平均长度，等等。

为了能够实现预期功能，倒排索引需要知道集合中的 所有 文档。早期的全文检索会为整个文档集合建立一个很大的倒排索引并将其写入到磁盘。 一旦新的索引就绪，旧的就会被其替换，这样最近的变化便可以被检索到。

### 10.2   动态更新索引

下一个需要被解决的问题是怎样在保留不变性的前提下实现倒排索引的更新？答案是: 用更多的索引。

通过增加新的补充索引来反映新近的修改，而不是直接重写整个倒排索引。每一个倒排索引都会被轮流查询到—从最早的开始—查询完后再对结果进行合并。

Elasticsearch 基于 Lucene, 这个 java 库引入了 按段搜索 的概念。 每一 段 本身都是一个倒排索引。

#### 10.2.1  索引与分片的比较

被混淆的概念是，一个 Lucene 索引 我们在 Elasticsearch 称作 分片 。 一个 Elasticsearch 索引 是分片的集合。 当 Elasticsearch 在索引中搜索的时候， 他发送查询到每一个属于索引的分片(Lucene 索引)，然后像 执行分布式检索 提到的那样，合并每个分片的结果到一个全局的结果集。

#### 10.2.2  删除和更新

段是不可改变的，所以既不能从把文档从旧的段中移除，也不能修改旧的段来进行反映文档的更新。 取而代之的是，每个提交点会包含一个 .del 文件，文件中会列出这些被删除文档的段信息。

当一个文档被 “删除” 时，它实际上只是在 .del 文件中被 标记 删除。一个被标记删除的文档仍然可以被查询匹配到， 但它会在最终结果被返回前从结果集中移除。

文档更新也是类似的操作方式：当一个文档被更新时，旧版本文档被标记删除，文档的新版本被索引到一个新的段中。 可能两个版本的文档都会被一个查询匹配到，但被删除的那个旧版本文档在结果集返回前就已经被移除。

### 10.3   近实时搜索

随着按段（per-segment）搜索的发展，一个新的文档从索引到可被搜索的延迟显著降低了。新文档在几分钟之内即可被检索，但这样还是不够快。

磁盘在这里成为了瓶颈。提交（Commiting）一个新的段到磁盘需要一个 fsync 来确保段被物理性地写入磁盘，这样在断电的时候就不会丢失数据。 但是 fsync 操作代价很大; 如果每次索引一个文档都去执行一次的话会造成很大的性能问题。

我们需要的是一个更轻量的方式来使一个文档可被搜索，这意味着 fsync 要从整个过程中被移除。

在Elasticsearch和磁盘之间是文件系统缓存。像之前描述的一样， 在内存索引缓冲区中的文档会被写入到一个新的段中。但是这里新段会被先写入到文件系统缓存—这一步代价会比较低，稍后再被刷新到磁盘—这一步代价比较高。不过只要文件已经在缓存中， 就可以像其它文件一样被打开和读取了。

#### 10.3.1  refresh API

在 Elasticsearch 中，写入和打开一个新段的轻量的过程叫做 refresh 。 默认情况下每个分片会每秒自动刷新一次。这就是为什么我们说 Elasticsearch 是 近 实时搜索: 文档的变化并不是立即对搜索可见，但会在一秒之内变为可见。

这些行为可能会对新用户造成困惑: 他们索引了一个文档然后尝试搜索它，但却没有搜到。这个问题的解决办法是用 refresh API 执行一次手动刷新:

```
POST /_refresh 
POST /blogs/_refresh 
```

注：

*1.*    *刷新（**Refresh**）所有的索引。*

*2.*   *只刷新（**Refresh**）* *blogs* *索引。*

并不是所有的情况都需要每秒刷新。可能你正在使用 Elasticsearch 索引大量的日志文件， 你可能想优化索引速度而不是近实时搜索， 可以通过设置 refresh_interval ， 降低每个索引的刷新频率：

```
PUT /my_logs
{
  "settings": {
    "refresh_interval": "30s" 
  }
}
```

注：*每**30**秒刷新* *my_logs* *索引。*

refresh_interval 可以在既存索引上进行动态更新。 在生产环境中，当你正在建立一个大的新索引时，可以先关闭自动刷新，待开始使用该索引时，再把它们调回来：

```
PUT /my_logs/_settings
{ "refresh_interval": -1 } 
 
PUT /my_logs/_settings
{ "refresh_interval": "1s" } 
```

注：  

*1.*   *第一个关闭自动刷新。*

*2.*   *第二个每秒自动刷新。*

### 10.4   持久化变更

#### 10.4.1  translog事务日志

如果没有用 fsync 把数据从文件系统缓存刷（flush）到硬盘，我们不能保证数据在断电甚至是程序正常退出之后依然存在。为了保证 Elasticsearch 的可靠性，需要确保数据变化被持久化到磁盘。

Elasticsearch 增加了一个 translog ，或者叫事务日志，在每一次对 Elasticsearch 进行操作时均进行了日志记录。translog流程：

l 当一个文档被索引之后，就会被添加到内存缓冲区，并且 追加到了 translog。

l 刷新（refresh）使分片处于刷新（refresh）完成后, 缓存被清空但是事务日志不会。

l 这个进程继续工作，更多的文档被添加到内存缓冲区和追加到事务日志。

l 每隔一段时间—例如 translog 变得越来越大—索引被刷新（flush）；一个新的 translog 被创建，老的 translog 被删除，并且一个全量提交被执行。.

translog 提供所有还没有被刷到磁盘的操作的一个持久化纪录。当 Elasticsearch 启动的时候， 它会从磁盘中使用最后一个提交点去恢复已知的段，并且会重放 translog 中所有在最后一次提交后发生的变更操作。

#### 10.4.2  flush API

这个执行一个提交并且截断 translog 的行为在 Elasticsearch 被称作一次 flush 。 分片每30分钟被自动刷新（flush），或者在 translog 太大的时候也会刷新。

flush API 可以被用来执行一个手工的刷新（flush）:

```
POST /blogs/_flush 
 
POST /_flush?wait_for_ongoing 
```

注：

*1.*   *刷新（**flush**）* *blogs* *索引。*

*2.*   *刷新（**flush**）所有的索引并且并且等待所有刷新在返回前完成。*

### 10.5   段合并

由于自动刷新流程每秒会创建一个新的段 ，这样会导致短时间内的段数量暴增。而段数目太多会带来较大的麻烦。 每一个段都会消耗文件句柄、内存和cpu运行周期。更重要的是，每个搜索请求都必须轮流检查每个段；所以段越多，搜索也就越慢。

Elasticsearch通过在后台进行段合并来解决这个问题。小的段被合并到大的段，然后这些大的段再被合并到更大的段。

 

段合并的时候会将那些旧的已删除文档从文件系统中清除。被删除的文档（或被更新文档的旧版本）不会被拷贝到新的大段中。

#### 10.5.1  optimize API

optimize API大可看做是 强制合并 API。它会将一个分片强制合并到 max_num_segments 参数指定大小的段数目。 这样做的意图是减少段的数量（通常减少到一个），来提升搜索性能。

在特定情况下，使用 optimize API 颇有益处。例如在日志这种用例下，每天、每周、每月的日志被存储在一个索引中。 老的索引实质上是只读的；它们也并不太可能会发生变化。

在这种情况下，使用optimize优化老的索引，将每一个分片合并为一个单独的段就很有用了；这样既可以节省资源，也可以使搜索更加快速：

```
POST /logstash-2014-10/_optimize?max_num_segments=1 
```

注：*合并索引中的每个分片为一个单独的段。*

请注意，使用 optimize API 触发段合并的操作不会受到任何资源上的限制。这可能会消耗掉你节点上全部的I/O资源, 使其没有余裕来处理搜索请求，从而有可能使集群失去响应。 如果你想要对索引执行 optimize，你需要先使用分片分配（查看 迁移旧索引）把索引移到一个安全的节点，再执行。

## 11   深入搜索

### 11.1   结构化搜索

结构化搜索（Structured search） 是指有关探询那些具有内在结构数据的过程。比如日期、时间和数字都是结构化的：它们有精确的格式，我们可以对这些格式进行逻辑操作。比较常见的操作包括比较数字或时间的范围，或判定两个值的大小。

#### 11.1.1  精确值查找

当进行精确值查找时，我们会使用过滤器（filters）。过滤器很重要，因为它们执行速度非常快，不会计算相关度（直接跳过了整个评分阶段）而且很容易被缓存，所以尽可能多的使用过滤式查询。

##### 11.1.1.1  term 查询数字

建立索引

POST /my_store/products/_bulk

{ "index": { "_id": 1 }}

{ "price" : 10, "productID" : "XHDK-A-1293-#fJ3" }

{ "index": { "_id": 2 }}

{ "price" : 20, "productID" : "KDKE-B-9947-#kL5" }

{ "index": { "_id": 3 }}

{ "price" : 30, "productID" : "JODL-X-1937-#pV7" }

{ "index": { "_id": 4 }}

{ "price" : 30, "productID" : "QQPX-R-3956-#aD8" }

当查找一个精确值的时候，不希望对查询进行评分计算，使用 constant_score 查询以非评分模式来执行 term 查询并以一作为统一评分。

GET /my_store/products/_search

{

  "query" : {

​    "constant_score" : { 

​      "filter" : {

​        "term" : { 

​          "price" : 20

​        }

​      }

​    }

  }

}



结果

 ![image-20200704002742468](../../../Typora/Picture/image-20200704002742468.png)

图 11—1

注：  

*查询置于* *filter* *语句内不进行评分或相关度的计算，所以所有的结果都会返回一个默认评分* *1* *。*

##### 11.1.1.2  term 查询文本

GET /my_store/products/_search

{

  "query" : {

​    "constant_score" : {

​      "filter" : {

​        "term" : {

​          "productID" : "XHDK-A-1293-#fJ3"

​        }

​      }

​    }

  }

}

这样是查不出想要的结果的，term是not_analyzed查询，而执行标准化查询的时候，需要注意以下几点：

l Elasticsearch 用 4 个不同的 token 而不是单个 token 来表示这个 UPC 。

l 所有字母都是小写的。

l 丢失了连字符和哈希符（ # ）。

因此无法直接查询。

解决思路：

\1.   使用match代替

\2.   将productID字段设置成为not_analyzed

##### 11.1.1.3  内部过滤器的操作

在内部，Elasticsearch 会在运行非评分查询的时执行多个操作：

**1.**   **查找匹配文档****.**

term 查询在倒排索引中查找 XHDK-A-1293-#fJ3 然后获取包含该 term 的所有文档。本例中，只有文档 1 满足我们要求。

**2.**   **创建** **bitset.**

过滤器会创建一个 bitset （一个包含 0 和 1 的数组），它描述了哪个文档会包含该 term 。匹配文档的标志位是 1 。本例中，bitset 的值为 [1,0,0,0] 。在内部，它表示成一个 "roaring bitmap"，可以同时对稀疏或密集的集合进行高效编码。

**3.**   **迭代** **bitset(s)**

一旦为每个查询生成了 bitsets ，Elasticsearch 就会循环迭代 bitsets 从而找到满足所有过滤条件的匹配文档的集合。执行顺序是启发式的，但一般来说先迭代稀疏的 bitset （因为它可以排除掉大量的文档）。

**4.**   **增量使用计数****.**

Elasticsearch 能够缓存非评分查询从而获取更快的访问，但是它也会不太聪明地缓存一些使用极少的东西。非评分计算因为倒排索引已经足够快了，所以我们只想缓存那些我们 知道 在将来会被再次使用的查询，以避免资源的浪费。

为了实现以上设想，Elasticsearch 会为每个索引跟踪保留查询使用的历史状态。如果查询在最近的 256 次查询中会被用到，那么它就会被缓存到内存中。当 bitset 被缓存后，缓存会在那些低于 10,000 个文档（或少于 3% 的总索引数）的段（segment）中被忽略。这些小的段即将会消失，所以为它们分配缓存是一种浪费。

#### 11.1.2  组合过滤器

SELECT product

FROM  products

WHERE (price = 20 OR productID = "XHDK-A-1293-#fJ3")

 AND (price != 30)

为达到以上效果

GET /my_store/products/_search

{

  "query" : {

   "filtered" : { 

​     "filter" : {

​      "bool" : {

​       "should" : [

​         { "term" : {"price" : 20}}, 

​         { "term" : {"productID" : "XHDK-A-1293-#fJ3"}} 

​       ],

​       "must_not" : {

​         "term" : {"price" : 30} 

​       }

​      }

​     }

   }

  }

}

 

SELECT document

FROM  products

WHERE productID   = "KDKE-B-9947-#kL5"

 OR (   productID = "JODL-X-1937-#pV7"

​    AND price   = 30 )

为达到以上效果

GET /my_store/products/_search

{

  "query" : {

   "filtered" : {

​     "filter" : {

​      "bool" : {

​       "should" : [

​        { "term" : {"productID" : "KDKE-B-9947-#kL5"}}, 

​        { "bool" : { 

​         "must" : [

​          { "term" : {"productID" : "JODL-X-1937-#pV7"}}, 

​          { "term" : {"price" : 30}} 

​          ]

​        }}

​       ]

​      }

​     }

   }

  }

}

#### 11.1.3  查找多个精确值

查找价格字段值为 $20 或 $30 的文档，使用terms，同时将price字段的值改为数组

GET /my_store/products/_search

{

  "query" : {

​    "constant_score" : {

​      "filter" : {

​        "terms" : { 

​          "price" : [20, 30]

​        }

​      }

​    }

  }

}

#### 11.1.4  查询范围

##### 11.1.4.1  查询数字范围

SELECT document

FROM  products

WHERE price BETWEEN 20 AND 40

为达到以上效果

GET /my_store/products/_search

{

  "query" : {

​    "constant_score" : {

​      "filter" : {

​        "range" : {

​          "price" : {

​            "gte" : 20,

​            "lt" : 40

​          }

​        }

​      }

​    }

  }

}

##### 11.1.4.2  查询日期范围

**例****1****：**查询指定时间段的所有文档

"range" : {

  "timestamp" : {

​    "gt" : "2014-01-01 00:00:00",

​    "lt" : "2014-01-07 00:00:00"

  }

}

**例****2****：**查找时间戳在过去一小时内的所有文档

"range" : {

  "timestamp" : {

​    "gt" : "now-1h"

  }

}

**例****3****：**日期计算还可以被应用到某个具体的时间，只要在某个日期后加上一个双管符号 (||) 并紧跟一个日期数学表达式就能做到：

"range" : {

  "timestamp" : {

​    "gt" : "2014-01-01 00:00:00",

​    "lt" : "2014-01-01 00:00:00||+1M" 

  }

}

注:

*+1M--->2014* *年* *1* *月* *1* *日加* *1* *月（**2014* *年* *2* *月* *1* *日* *零时）*

##### 11.1.4.3  字符串范围

查找从 a 到 b （不包含）的字符串，同样可以使用 range 查询语法：

"range" : {

  "title" : {

​    "gte" : "a",

​    "lt" : "b"

  }

}

 

#### 11.1.5  处理 Null 值

##### 11.1.5.1  存在查询（exists）

生成如下文档

```
POST /my_index/posts/_bulk
{ "index": { "_id": "1"              }}
{ "tags" : ["search"]                }  
{ "index": { "_id": "2"              }}
{ "tags" : ["search", "open_source"] }  
{ "index": { "_id": "3"              }}
{ "other_field" : "some data"        }  
{ "index": { "_id": "4"              }}
{ "tags" : null                      }  
{ "index": { "_id": "5"              }}
{ "tags" : ["search", null]          }  
```

执行如下sql

```
SELECT tags
FROM   posts
WHERE  tags IS NOT NULL
```

为达到以上效果

```
GET /my_index/posts/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "exists" : { "field" : "tags" }
            }
        }
    }
}
```

##### 11.1.5.2  缺失查询（missing）

```
SELECT tags
FROM   posts
WHERE  tags IS NULL
```

为达到以上结果

```
GET /my_index/posts/_search
{
    "query" : {
        "constant_score" : {
            "filter": {
                "missing" : { "field" : "tags" }
            }
        }
    }
}
```

#### 11.1.6  缓存

实际是采用一个 bitset 记录与过滤器匹配的文档。Elasticsearch 积极地把这些 bitset 缓存起来以备随后使用。一旦缓存成功，bitset 可以复用 任何 已使用过的相同过滤器，而无需再次计算整个过滤器。

##### 11.1.6.1  独立的过滤器缓存

属于一个查询组件的 bitsets 是独立于它所属搜索请求其他部分的。这就意味着，一旦被缓存，一个查询可以被用作多个搜索请求。bitsets 并不依赖于它所存在的查询上下文。这样使得缓存可以加速查询中经常使用的部分，从而降低较少、易变的部分所带来的消耗。

找满足以下任意一个条件的电子邮件：

l 在收件箱中，且没有被读过的

l 不在 收件箱中，但被标注重要的

 ![image-20200704002819022](../../../Typora/Picture/image-20200704002819022.png)

图 11—2

尽管其中一个收件箱的条件是 must 语句，另一个是 must_not 语句，但他们两者是完全相同的。这意味着在第一个语句执行后， bitset 就会被计算然后缓存起来供另一个使用。当再次执行这个查询时，收件箱的这个过滤器已经被缓存了，所以两个语句都会使用已缓存的 bitset 。

##### 11.1.6.2  自动缓存行为

Elasticsearch 会基于使用频次自动缓存查询。如果一个非评分查询在最近的 256 次查询中被使用过（次数取决于查询类型），那么这个查询就会作为缓存的候选。但是，并不是所有的片段都能保证缓存 bitset 。只有那些文档数量超过 10,000 （或超过总文档数量的 3% )才会缓存 bitset 。因为小的片段可以很快的进行搜索和合并，这里缓存的意义不大。

一旦缓存了，非评分计算的 bitset 会一直驻留在缓存中直到它被剔除。剔除规则是基于 LRU 的：一旦缓存满了，最近最少使用的过滤器会被剔除。

### 11.2   全文搜索

#### 11.2.1  基于词项与基于全文

##### 11.2.1.1  基于词项的查询

如 term 或 fuzzy 这样的底层查询不需要分析阶段，它们对单个词项进行操作。用 term 查询词项 Foo 只要在倒排索引中查找 准确词项 ，并且用 TF/IDF 算法为每个包含该词项的文档计算相关度评分 _score 。

##### 11.2.1.2  基于全文的查询

像 match 或 query_string 这样的查询是高层查询，它们了解字段映射的信息：

 

l 如果查询 日期（date） 或 整数（integer） 字段，它们会将查询字符串分别作为日期或整数对待。

l 如果查询一个（ not_analyzed ）未分析的精确值字符串字段，它们会将整个查询字符串作为单个词项对待。

l 但如果要查询一个（ analyzed ）已分析的全文字段，它们会先将查询字符串传递到一个合适的分析器，然后生成一个供查询的词项列表。

#### 11.2.2  匹配查询

用一个示例来解释使用 match 查询搜索全文字段中的单个词：

```
GET /my_index/my_type/_search
{
    "query": {
        "match": {
            "title": "QUICK!"
        }
    }
}
```

Elasticsearch 执行上面这个 match 查询的步骤是：

\1.   **检查字段类型：**标题 title 字段是一个 string 类型（ analyzed ）已分析的全文字段，这意味着查询字符串本身也应该被分析。

\2.   **分析查询字符串：**将查询的字符串 QUICK! 传入标准分析器中，输出的结果是单个项 quick 。因为只有一个单词项，所以 match 查询执行的是单个底层 term 查询。

\3.   **查找匹配文档：**用 term 查询在倒排索引中查找 quick 然后获取一组包含该项的文档，本例的结果是文档：1、2 和 3 。

\4.   **为每个文档评分：**用 term 查询计算每个文档相关度评分 _score ，这是种将词频（term frequency，即词 quick 在相关文档的 title 字段中出现的频率）和反向文档频率（inverse document frequency，即词 quick 在所有文档的 title 字段中出现的频率），以及字段的长度（即字段越短相关度越高）相结合的计算方式。

#### 11.2.3  多词查询

例：

```
GET /my_index/my_type/_search
{
    "query": {
        "match": {
            "title": "BROWN DOG!"
        }
    }
}
```

能查询到多个结果

 ![image-20200704002835646](../../../Typora/Picture/image-20200704002835646.png)

图 11—3

##### 11.2.3.1  查询操作符，提高精度

match 查询还可以接受 operator 操作符作为输入参数，默认情况下该操作符（operator）是 or 。我们可以将它修改成 and 让所有指定词项都必须匹配：

```
GET /my_index/my_type/_search
{
    "query": {
        "match": {
            "title": {      
                "query":    "BROWN DOG!",
                "operator": "and"
            }
        }
    }
}
```

查询到的结果中必须包含brown 和 dog

##### 11.2.3.2  设置相关性，控制精度

match 查询支持 minimum_should_match 最小匹配参数，这让我们可以指定必须匹配的词项数用来表示一个文档是否相关。

```
GET /my_index/my_type/_search
{
  "query": {
    "match": {
      "title": {
        "query":                "quick brown dog",
        "minimum_should_match": "75%"
      }
    }
  }
}
```

#### 11.2.4  组合查询

当没有 must 语句的时候，至少有一个 should 语句必须匹配。

就像我们能控制 match 查询的精度一样，我们可以通过 minimum_should_match 参数控制需要匹配的 should 语句的数量，它既可以是一个绝对的数字，又可以是个百分比：

```
GET /my_index/my_type/_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title": "brown" }},
        { "match": { "title": "fox"   }},
        { "match": { "title": "dog"   }}
      ],
      "minimum_should_match": 2 
    }
  }
}
```

#### 11.2.5  如何使用布尔匹配

```
{
    "match": { "title": "brown fox"}
}
```

 

```
{
  "bool": {
    "should": [
      { "term": { "title": "brown" }},
      { "term": { "title": "fox"   }}
    ]
  }
}
```

这两个查询是等价的

```
{
    "match": {
        "title": {
            "query":    "brown fox",
            "operator": "and"
        }
    }
}
```

 

```
{
  "bool": {
    "must": [
      { "term": { "title": "brown" }},
      { "term": { "title": "fox"   }}
    ]
  }
}
```

这两个查询也是等价的

指定参数 minimum_should_match ，它可以通过 bool 查询直接传递，使以下两个查询等价：

```
{
    "match": {
        "title": {
            "query":                "quick brown fox",
            "minimum_should_match": "75%"
        }
    }
}
```

 

```
{
  "bool": {
    "should": [
      { "term": { "title": "brown" }},
      { "term": { "title": "fox"   }},
      { "term": { "title": "quick" }}
    ],
    "minimum_should_match": 2 
  }
}
```

注：*因为只有三条语句，**match* *查询的参数* *minimum_should_match* *值* *75%* *会被截断成* *2* *。即三条* *should* *语句中至少有两条必须匹配。*

#### 11.2.6  查询语句提升权重

##### 11.2.6.1  词权重提升

假设想要查询关于 “full-text search（全文搜索）” 的文档，但我们希望为提及 “Elasticsearch” 或 “Lucene” 的文档给予更高的 权重 ，这里 更高权重 是指如果文档中出现 “Elasticsearch” 或 “Lucene” ，它们会比没有的出现这些词的文档获得更高的相关度评分 _score：

```
GET /_search
{
    "query": {
        "bool": {
            "must": {
                "match": {
                    "content": { 
                        "query":    "full text search",
                        "operator": "and"
                    }
                }
            },
            "should": [ 
                { "match": { "content": "Elasticsearch" }},
                { "match": { "content": "Lucene"        }}
            ]
        }
    }
}
```

如果我们想让包含 Lucene 的有更高的权重，并且包含 Elasticsearch 的语句比 Lucene 的权重更高。可以通过指定 boost 来控制任何查询语句的相对的权重， boost 的默认值为 1 ，大于 1 会提升一个语句的相对权重。所以下面重写之前的查询：

```
GET /_search
{
    "query": {
        "bool": {
            "must": {
                "match": {  
                    "content": {
                        "query":    "full text search",
                        "operator": "and"
                    }
                }
            },
            "should": [
                { "match": {
                    "content": {
                        "query": "Elasticsearch",
                        "boost": 3 
                    }
                }},
                { "match": {
                    "content": {
                        "query": "Lucene",
                        "boost": 2 
                    }
                }}
            ]
        }
    }
}
```

##### 11.2.6.2  索引权重提升

当在多个索引中搜索时，可以使用参数 indices_boost 来提升整个索引的权重，在下面例子中，当要为最近索引的文档分配更高权重时，可以这么做：

```
GET /docs_2014_*/_search 
{
  "indices_boost": { 
    "docs_2014_10": 3,
    "docs_2014_09": 2
  },
  "query": {
    "match": {
      "text": "quick brown fox"
    }
  }
}
```

注：

*1.*   *这个多索引查询涵盖了所有以字符串* *docs_2014_* *开始的索引。*

*2.*   *其中，索引* *docs_2014_10* *中的所有文件的权重是* *3* *，索引* *docs_2014_09* *中是* *2* *，其他所有匹配的索引权重为默认值* *1* *。*

##### 11.2.6.3  权重提升查询

```
GET /_search
{
  "query": {
    "boosting": {
      "positive": {
        "match": {
          "text": "apple"
        }
      },
      "negative": {
        "match": {
          "text": "pie tart fruit crumble tree"
        }
      },
      "negative_boost": 0.5
    }
  }
}
```

它接受 positive 和 negative 查询。只有那些匹配 positive 查询的文档罗列出来，对于那些同时还匹配 negative 查询的文档将通过文档的原始 _score 与 negative_boost 相乘的方式降级后的结果。

为了达到效果， negative_boost 的值必须小于 1.0 。在这个示例中，所有包含负向词的文档评分 _score 都会减半。

#### 11.2.7  控制分析

控制分析的含义：可以为不同的字段按需求配置指定的分析器

```
PUT /my_index/_mapping/my_type
{
    "my_type": {
        "properties": {
            "english_title": {
                "type":     "string",
                "analyzer": "english"
            }
        }
    }
}
```

#### 11.2.8  被破坏的相关度！

设想如果有 5 个 foo 文档存于分片 1 ，而第 6 个文档存于分片 2 ，在这种场景下， foo 在分片1里非常普通（所以不那么重要），但是在分片2里非常出现很少（所以会显得更重要）。这些 IDF 之间的差异会导致不正确的结果。

在实际应用中，这并不是一个问题，本地和全局的 IDF 的差异会随着索引里文档数的增多渐渐消失，在真实世界的数据量下，局部的 IDF 会被迅速均化，所以上述问题并不是相关度被破坏所导致的，而是由于数据太少。

为了测试，我们可以通过两种方式解决这个问题。第一种是只在主分片上创建索引，正如 match 查询 里介绍的那样，如果只有一个分片，那么本地的 IDF 就是 全局的 IDF。

第二个方式就是在搜索请求后添加 ?search_type=dfs_query_then_fetch ， dfs 是指 分布式频率搜索（Distributed Frequency Search） ， 它告诉 Elasticsearch ，先分别获得每个分片本地的 IDF ，然后根据结果再计算整个索引的全局 IDF 。

### 11.3   多字段搜索

#### 11.3.1   多字符串查询

##### 11.3.1.1  调整评分计算方式

```
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":  "War and Peace" }},
        { "match": { "author": "Leo Tolstoy"   }},
        { "bool":  {
          "should": [
            { "match": { "translator": "Constance Garnett" }},
            { "match": { "translator": "Louise Maude"      }}
          ]
        }}
      ]
    }
  }
}
```

为什么将译者条件语句放入另一个独立的 bool 查询中呢？所有的四个 match 查询都是 should 语句，所以为什么不将 translator 语句与其他如 title 、 author 这样的语句放在同一层呢？

答案在于评分的计算方式。 bool 查询运行每个 match 查询，再把评分加在一起，然后将结果与所有匹配的语句数量相乘，最后除以所有的语句数量。处于同一层的每条语句具有相同的权重。在前面这个例子中，包含 translator 语句的 bool 查询，只占总评分的三分之一。如果将 translator 语句与 title 和 author 两条语句放入同一层，那么 title 和 author 语句只贡献四分之一评分。

##### 11.3.1.2  语句优先级

使用boost 参数，为了提升 title 和 author 字段的权重，为它们分配的 boost 值大于 1 ：

```
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { 
            "title":  {
              "query": "War and Peace",
              "boost": 2
        }}},
        { "match": { 
            "author":  {
              "query": "Leo Tolstoy",
              "boost": 2
        }}},
        { "bool":  { 
            "should": [
              { "match": { "translator": "Constance Garnett" }},
              { "match": { "translator": "Louise Maude"      }}
            ]
        }}
      ]
    }
  }
}
```

#### 11.3.2  单字符串查询

##### 11.3.2.1  最佳字段

当搜索词语具体概念的时候，比如 “brown fox” ，词组比各自独立的单词更有意义。像 title 和 body 这样的字段，尽管它们之间是相关的，但同时又彼此相互竞争。文档在 相同字段 中包含的词越多越好，评分也来自于 最匹配字段 。

###### 11.3.2.1.1  非最佳字段举例：

```
PUT /my_index/my_type/1
{
    "title": "Quick brown rabbits",
    "body":  "Brown rabbits are commonly seen."
}
 
PUT /my_index/my_type/2
{
    "title": "Keeping pets healthy",
    "body":  "My quick brown fox eats rabbits on a regular basis."
}
```

运行下面程序

```
{
    "query": {
        "bool": {
            "should": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}
```

用肉眼判断，文档 2 的匹配度更高，因为它同时包括要查找的两个词，但是结果确相反：

 ![image-20200704002856086](../../../Typora/Picture/image-20200704002856086.png)

图 11—4

为了理解导致这样的原因，需要回想一下 bool 是如何计算评分的：

l 它会执行 should 语句中的两个查询。

l 加和两个查询的评分。

l 乘以匹配语句的总数。

l 除以所有语句总数（这里为：2）。

文档 1 的两个字段都包含 brown 这个词，所以两个 match 语句都能成功匹配并且有一个评分。文档 2 的 body 字段同时包含 brown 和 fox 这两个词，但 title 字段没有包含任何词。这样， body 查询结果中的高分，加上 title 查询中的 0 分，然后乘以二分之一，就得到比文档 1 更低的整体评分。

###### 11.3.2.1.2  dis_max 查询（匹配最佳字段）

分离（Disjunction）的意思是 或（or） ，这与可以把结合（conjunction）理解成 与（and） 相对应。分离最大化查询（Disjunction Max Query）指的是： 将任何与任一查询匹配的文档作为结果返回，但只将最佳匹配的评分作为查询的评分结果返回 ：

```
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}
```

得到我们想要的结果为：

 ![image-20200704002912694](../../../Typora/Picture/image-20200704002912694.png)

图 11—5

###### 11.3.2.1.3  （dis_max）最佳字段查询调优

当用户搜索 “quick pets” 时会发生什么呢？在前面的例子中，两个文档都包含词 quick ，但是只有文档 2 包含词 pets ，两个文档中都不具有同时包含 两个词 的 相同字段 。

```
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Quick pets" }},
                { "match": { "body":  "Quick pets" }}
            ]
        }
    }
}
```

结果，两个评分完全一样

 ![image-20200704002920612](../../../Typora/Picture/image-20200704002920612.png)

图 11—6

我们可能会觉得文档2相关度更高，但事实并非如此，因为 dis_max 查询只会简单地使用 单个 最佳匹配语句的评分 _score 作为整体评分

**tie_breaker** **参数**

可以通过指定 tie_breaker 这个参数将其他匹配语句的评分也考虑其中：

```
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Quick pets" }},
                { "match": { "body":  "Quick pets" }}
            ],
            "tie_breaker": 0.3
        }
    }
}
```

结果如下：

 ![image-20200704002930233](../../../Typora/Picture/image-20200704002930233.png)

图 11—7

tie_breaker 参数提供了一种 dis_max 和 bool 之间的折中选择，它的评分方式如下：

l 获得最佳匹配语句的评分 _score 。

l 将其他匹配语句的评分结果与 tie_breaker 相乘。

l 对以上评分求和并规范化。

 

 

 

 

##### 11.3.2.2  多数字段

为了对相关度进行微调，常用的一个技术就是将相同的数据索引到不同的字段，它们各自具有独立的分析链。

主字段可能包括它们的词源、同义词以及 变音词 或口音词，被用来匹配尽可能多的文档。

相同的文本被索引到其他字段，以提供更精确的匹配。一个字段可以包括未经词干提取过的原词，另一个字段包括其他词源、口音，还有一个字段可以提供 词语相似性 信息的瓦片词（shingles）。

其他字段是作为匹配每个文档时提高相关度评分的 信号，匹配字段越多 则越好。

 

##### 11.3.2.3  混合字段

对于某些实体，我们需要在多个字段中确定其信息，单个字段都只能作为整体的一部分：

 

Person： first_name 和 last_name （人：名和姓）

Book： title 、 author 和 description （书：标题、作者、描述）

Address： street 、 city 、 country 和 postcode （地址：街道、市、国家和邮政编码）

#### 11.3.3  multi_match 查询

##### 11.3.3.1  multi_match举例

```
{
  "dis_max": {
    "queries":  [
      {
        "match": {
          "title": {
            "query": "Quick brown fox",
            "minimum_should_match": "30%"
          }
        }
      },
      {
        "match": {
          "body": {
            "query": "Quick brown fox",
            "minimum_should_match": "30%"
          }
        }
      },
    ],
    "tie_breaker": 0.3
  }
}
```

上面这个查询用 multi_match 重写成更简洁的形式：

```
{
    "multi_match": {
        "query":                "Quick brown fox",
        "type":                 "best_fields", 
        "fields":               [ "title", "body" ],
        "tie_breaker":          0.3,
        "minimum_should_match": "30%" 
    }
}
```

注：

*1.*    *best_fields* *类型是默认值，可以不指定。*

*2.*   *如* *minimum_should_match* *或* *operator* *这样的参数会被传递到生成的* *match* *查询中。*

##### 11.3.3.2  查询字段名称的模糊匹配

可以使用以下方式同时匹配 book_title 、 chapter_title 和 section_title （书名、章名、节名）这三个字段：

```
{
    "multi_match": {
        "query":  "Quick brown fox",
        "fields": "*_title"
    }
}
```

##### 11.3.3.3  提升单个字段的权重

可以使用 ^ 字符语法为单个字段提升权重，在字段名称的末尾添加 ^boost ，其中 boost 是一个浮点数：

```
{
    "multi_match": {
        "query":  "Quick brown fox",
        "fields": [ "*_title", "chapter_title^2" ] 
    }
}
```

注：

*chapter_title* *这个字段的* *boost* *值为* *2* *，而其他两个字段* *book_title* *和* *section_title* *字段的默认* *boost* *值为* *1* *。*

#### 11.3.4  多数字段

全文搜索被称作是 召回率（Recall） 与 精确率（Precision） 的战场： 召回率 ——返回所有的相关文档； 精确率 ——不返回无关文档。目的是在结果的第一页中为用户呈现最为相关的文档。

提高全文相关性精度的常用方式是为同一文本建立多种方式的索引，每种方式都提供了一个不同的相关度信号 signal 。主字段会以尽可能多的形式的去匹配尽可能多的文档。举个例子，我们可以进行以下操作：

l 使用词干提取来索引 jumps 、 jumping 和 jumped 样的词，将 jump 作为它们的词根形式。这样即使用户搜索 jumped ，也还是能找到包含 jumping 的匹配的文档。

l 将同义词包括其中，如 jump 、 leap 和 hop 。

l 移除变音或口音词：如 ésta 、 está 和 esta 都会以无变音形式 esta 来索引。

##### 11.3.4.1  多字段映射

首先要做的事情就是对我们的字段索引两次：一次使用词干模式以及一次非词干模式。为了做到这点，采用 multifields 来实现：

```
DELETE /my_index
 
PUT /my_index
{
    "settings": { "number_of_shards": 1 }, 
    "mappings": {
        "my_type": {
            "properties": {
                "title": { 
                    "type":     "string",
                    "analyzer": "english",
                    "fields": {
                        "std":   { 
                            "type":     "string",
                            "analyzer": "standard"
                        }
                    }
                }
            }
        }
    }
}
```

注：

*1.*   *title* *字段使用* *english* *英语分析器来提取词干。*

*2.*   *title.std* *字段使用* *standard* *标准分析器，所以没有词干提取。*

输入值：

```
PUT /my_index/my_type/1
{ "title": "My rabbit jumps" }
 
PUT /my_index/my_type/2
{ "title": "Jumping jack rabbits" }
```

查询匹配：

```
GET /my_index/_search
{
   "query": {
        "multi_match": {
            "query":  "jumping rabbits",
            "type":   "most_fields", 
            "fields": [ "title", "title.std" ]
        }
    }
}
```

 

#### 11.3.5  跨字段实体搜索

在如 person 、 product 或 address （人、产品或地址）这样的实体中，需要使用多个字段来唯一标识它的信息。 person 实体可能是这样索引的：

```
{
    "firstname":  "Peter",
    "lastname":   "Smith"
}
或地址：
{
    "street":   "5 Poland Street",
    "city":     "London",
    "country":  "United Kingdom",
    "postcode": "W1V 3DG"
}
```

我们可能想搜索 “Poland Street W1V” 这个地址：

使用match方式：

```
{
  "query": {
    "bool": {
      "should": [
        { "match": { "street":    "Poland Street W1V" }},
        { "match": { "city":      "Poland Street W1V" }},
        { "match": { "country":   "Poland Street W1V" }},
        { "match": { "postcode":  "Poland Street W1V" }}
      ]
    }
  }
}
```

为每个字段重复查询字符串会使查询瞬间变得冗长，可以采用 multi_match 查询，将 type 设置成 most_fields 然后告诉 Elasticsearch 合并所有匹配字段的评分：

```
{
  "query": {
    "multi_match": {
      "query":       "Poland Street W1V",
      "type":        "most_fields",
      "fields":      [ "street", "city", "country", "postcode" ]
    }
  }
}
```

**most_fields** **方式的问题**

用 most_fields 这种方式搜索也存在某些问题，这些问题并不会马上显现：

\1.   它是为多数字段匹配任意词设计的，而不是在 所有字段 中找到最匹配的。

\2.   它不能使用 operator 或 minimum_should_match 参数来降低次相关结果造成的长尾效应。

\3.   词频对于每个字段是不一样的，而且它们之间的相互影响会导致不好的排序结果。

#### 11.3.6  字段中心式查询

例：

```
GET /_validate/query?explain
{
    "query": {
        "multi_match": {
            "query":       "peter smith",
            "type":        "most_fields",
            "operator":    "and", 
            "fields":      [ "first_name", "last_name" ]
        }
    }
}
```

以上面为例，分别阐述字段中心式（field-centric）和词中心式（term-centric）。

##### 11.3.6.1  字段中心式（field-centric）

对于匹配的文档， peter 和 smith 都必须同时出现在相同字段中，要么是 first_name 字段，要么 last_name 字段：

(+first_name:peter +first_name:smith)

(+last_name:peter +last_name:smith)

注：*most_fields* *、**best_fields* *类型是字段中心式的查询方式。*

##### 11.3.6.2  词中心式（term-centric）

词中心式 会使用以下逻辑：

+(first_name:peter last_name:peter)

+(first_name:smith last_name:smith)

换句话说，词 peter 和 smith 都必须出现，但是可以出现在任意字段中不一定非得是first_name和last_name字段。

注：*cross_fields* *类型是词中心式的查询方式*

#### 11.3.7  自定义 _all 字段

自定义_all字段，可以用copy_to 参数设置，full_name相当于一个_all字段，可以通过检索full_name字段达到同时检索first_name和last_name字段：

```
PUT /my_index
{
    "mappings": {
        "person": {
            "properties": {
                "first_name": {
                    "type":     "string",
                    "copy_to":  "full_name" 
                },
                "last_name": {
                    "type":     "string",
                    "copy_to":  "full_name" 
                },
                "full_name": {
                    "type":     "string"
                }
            }
        }
    }
}
```

注：

*1.*   *first_name* *和* *last_name* *字段中的值会被复制到* *full_name* *字段。*

*2.*   *copy_to* *设置对**multi-field**无效。如果尝试这样配置映射，**Elasticsearch* *会抛异常。*

#### 11.3.8  cross-fields 跨字段查询

**按字段提高权重**

采用 cross_fields 查询与 自定义 _all 字段 相比，其中一个优势就是它可以在搜索时为单个字段提升权重。

如果要用 title 和 description 字段搜索图书，可能希望为 title 分配更多的权重，这同样可以使用前面介绍过的 ^ 符号语法来实现：

```
GET /books/_search
{
    "query": {
        "multi_match": {
            "query":       "peter smith",
            "type":        "cross_fields",
            "fields":      [ "title^2", "description" ] 
        }
    }
}
```

注：*title* *字段的权重提升值为* *2* *，* *description* *字段的权重提升值默认为* *1* 。

### 11.4   近似匹配

#### 11.4.1  短语匹配

##### 11.4.1.1  什么是短语

一个被认定为和短语 quick brown fox 匹配的文档，必须满足以下这些要求：

 

\1.   quick 、 brown 和 fox 需要全部出现在域中。

\2.   brown 的位置应该比 quick 的位置大 1 。

\3.   fox 的位置应该比 quick 的位置大 2 。

如果以上任何一个选项不成立，则该文档不能认定为匹配。

##### 11.4.1.2  短语匹配

match_phrase 查询：

```
GET /my_index/my_type/_search
{
    "query": {
        "match_phrase": {
            "title": "quick brown fox"
        }
    }
}
```

匹配的结果应该满足以下条件：

\1.  quick 、 brown 和 fox 需要全部出现在域中。

\2.  brown 的位置应该比 quick 的位置大 1 。

\3.  fox 的位置应该比 quick 的位置大 2 。

因为词项的位置信息被存在倒排索引中，因此可以查到位置。

match_phrase 查询同样可写成一种类型为 phrase 的 match 查询:

```
"match": {
    "title": {
        "query": "quick brown fox",
        "type":  "phrase"
    }
}
```

#### 11.4.2  混合起来

使用 slop 参数将灵活度引入短语匹配中：

```
GET /my_index/my_type/_search
{
    "query": {
        "match_phrase": {
            "title": {
               "query": "quick fox",
               "slop":  50
            }
        }
    }
}
```

slop 参数告诉 match_phrase 查询词条相隔多远时仍然能将文档视为匹配 。 相隔多远的意思是为了让查询和文档匹配你需要移动词条多少次？

相距越近，相关性越高。

#### 11.4.3  多值字段（在6.8中并不能匹配）

```
PUT /my_index/groups/1
{
    "names": [ "John Abraham", "Lincoln Smith"]
}
```

然后运行一个对 Abraham Lincoln 的短语查询:

```
GET /my_index/groups/_search
{
    "query": {
        "match_phrase": {
            "names": "Abraham Lincoln"
        }
    }
}
```

在分析 John Abraham 的时候， 产生了如下信息：

Position 1: john

Position 2: abraham

然后在分析 Lincoln Smith 的时候， 产生了：

Position 3: lincoln

Position 4: smith

换句话说， Elasticsearch对以上数组分析生成了与分析单个字符串 John Abraham Lincoln Smith 一样几乎完全相同的语汇单元。 我们的查询示例寻找相邻的 lincoln 和 abraham ， 而且这两个词条确实存在，并且它们俩正好相邻， 所以这个查询匹配了。

**position_increment_gap**

在这样的情况下有一种叫做 position_increment_gap 的简单的解决方案， 它在字段映射中配置。

```
DELETE /my_index/groups/ 
 
PUT /my_index/_mapping/groups 
{
    "properties": {
        "names": {
            "type":                "string",
            "position_increment_gap": 100
        }
    }
}
```

position_increment_gap 设置告诉 Elasticsearch 应该为数组中每个新元素增加当前词条 position 的指定值。 所以现在当我们再索引 names 数组时，会产生如下的结果：

 

Position 1: john

Position 2: abraham

Position 103: lincoln

Position 104: smith

现在我们的短语查询可能无法匹配该文档因为 abraham 和 lincoln 之间的距离为 100 。 为了匹配这个文档你必须添加值为 100 的 slop 。

#### 11.4.4  越近越好

```
POST /my_index/my_type/_search
{
   "query": {
      "match_phrase": {
         "title": {
            "query": "quick dog",
            "slop":  50 
         }
      }
   }
}
```

注：  *注意高* *slop* *值。*

 

注：

*1.*    *分数较高因为* *quick* *和* *dog* *很接近*

*2.*    *分数较低因为* *quick* *和* *dog* *分开较远*

#### 11.4.5  使用邻近度提高相关度

```
GET /my_index/my_type/_search
{
  "query": {
    "bool": {
      "must": {
        "match": { 
          "title": {
            "query":                "quick brown fox",
            "minimum_should_match": "30%"
          }
        }
      },
      "should": {
        "match_phrase": { 
          "title": {
            "query": "quick brown fox",
            "slop":  50
          }
        }
      }
    }
  }
}
```

注：上面的查询代表

\1.   必须在title匹配quick brown fox中的一个

\2.   如果有短语“quick brown fox”，并且相互之间最大可以相隔50个词条，则匹配，相隔越近，匹配度越高，没有也不影响。

#### 11.4.6  性能优化

如何限制短语查询和邻近近查询的性能消耗呢？一种有用的方法是减少需要通过短语查询检查的文档总数。

phrase 查询—只是为了从每个分片中获得前 K 个结果。 然后会根据它们的最新评分 重新排序。

```
GET /my_index/my_type/_search
{
    "query": {
        "match": {  
            "title": {
                "query":                "quick brown fox",
                "minimum_should_match": "30%"
            }
        }
    },
    "rescore": {
        "window_size": 50, 
        "query": {         
            "rescore_query": {
                "match_phrase": {
                    "title": {
                        "query": "quick brown fox",
                        "slop":  50
                    }
                }
            }
        }
    }
}
```

注：

\1.   match 查询决定哪些文档将包含在最终结果集中，并通过 TF/IDF 排序。

\2.   window_size 是每一分片进行重新评分的顶部文档数量。

#### 11.4.7  寻找相关词

对句子 Sue ate the alligator ，不仅要将每一个单词（或者 unigram ）作为词项索引

["sue", "ate", "the", "alligator"]

也要将每个单词 以及它的邻近词 作为单个词项索引：

["sue ate", "ate the", "the alligator"]

这些单词对（或者 bigrams ）被称为 shingles 。

**shingles****优势**

shingles 不仅比短语查询更灵活，而且性能也更好。 shingles 查询跟一个简单的 match 查询一样高效，而不用每次搜索花费短语查询的代价。只是在索引期间因为更多词项需要被索引会付出一些小的代价， 这也意味着有 shingles 的字段会占用更多的磁盘空间。 然而，大多数应用写入一次而读取多次，所以在索引期间优化我们的查询速度是有意义的。

**查询的最终目标：**不需要任何前期进行过多的设置，就能够在搜索的时候有很好的效果。

### 11.5   部分匹配

#### 11.5.1  邮编与结构化数据

例如，邮编 W1V 3DG 可以分解成如下形式：

W1V ：这是邮编的外部，它定义了邮件的区域和行政区：

W 代表区域（ 1 或 2 个字母）

1V 代表行政区（ 1 或 2 个数字，可能跟着一个字符）

3DG ：内部定义了街道或建筑：

3 代表街区区块（ 1 个数字）

DG 代表单元（ 2 个字母）

#### 11.5.2  prefix 前缀查询

为了找到所有以 W1 开始的邮编，可以使用简单的 prefix 查询：

```
GET /my_index/address/_search
{
    "query": {
        "prefix": {
            "postcode": "W1"
        }
    }
}
```

prefix 查询存在严重的资源消耗问题，短语查询的这种方式也同样如此。前缀 a 可能会匹配成千上万的词，这不仅会消耗很多系统资源，而且结果的用处也不大。

#### 11.5.3  通配符与正则表达式查询

##### 11.5.3.1  wildcard 通配符查询

使用标准的 shell 通配符查询： ? 匹配任意字符， * 匹配 0 或多个字符。

```
GET /my_index/address/_search
{
    "query": {
        "wildcard": {
            "postcode": "W?F*HW" 
        }
    }
}
```

##### 11.5.3.2  regexp 查询

如果现在只想匹配 W 区域的所有邮编，前缀匹配也会包括以 WC 开头的所有邮编，与通配符匹配碰到的问题类似，如果想匹配只以 W 开始并跟随一个数字的所有邮编， regexp 正则式查询允许写出这样更复杂的模式：

```
GET /my_index/address/_search
{
    "query": {
        "regexp": {
            "postcode": "W[0-9].+" 
        }
    }
}
```

#### 11.5.4  查询时输入即搜索

用户已经渐渐习惯在输完查询内容之前，就能为他们展现搜索结果，这就是所谓的 即时搜索（instant search） 或 输入即搜索（search-as-you-type） 。不仅用户能在更短的时间内得到搜索结果，我们也能引导用户搜索索引中真实存在的结果。

例如，如果用户输入 johnnie walker bl ，我们希望在它们完成输入搜索条件前就能得到：Johnnie Walker Black Label 和 Johnnie Walker Blue Label 。

对于查询时的输入即搜索，可以使用 match_phrase 的一种特殊形式， match_phrase_prefix 查询：

```
{
    "match_phrase_prefix" : {
        "brand" : "johnnie walker bl"
    }
}
```

与 match_phrase 一样，它也可以接受 slop 参数（参照 slop ）让相对词序位置不那么严格：

```
{
    "match_phrase_prefix" : {
        "brand" : {
            "query": "walker johnnie bl", 
            "slop":  10
        }
    }
}
```

可以通过设置 max_expansions 参数来限制前缀扩展的影响，一个合理的值是可能是 50 ：

```
{
    "match_phrase_prefix" : {
        "brand" : {
            "query":          "johnnie walker bl",
            "max_expansions": 50
        }
    }
}
```

#### 11.5.5  Ngrams 在部分匹配的应用

##### 11.5.5.1  n-gram

在索引时准备数据意味着要选择合适的分析链，这里部分匹配使用的工具是 n-gram 。可以将 n-gram 看成一个在词语上 滑动窗口 ， n 代表这个 “窗口” 的长度。

如果我们要 n-gram quick 这个词 —— 当n为5时：

l 长度 1（unigram）： [ q, u, i, c, k ]

l 长度 2（bigram）： [ qu, ui, ic, ck ]

l 长度 3（trigram）： [ qui, uic, ick ]

l 长度 4（four-gram）： [ quic, uick ]

l 长度 5（five-gram）： [ quick ]

##### 11.5.5.2  边界 n-grams

对于输入即搜索（search-as-you-type）这种应用场景，我们会使用一种特殊的 n-gram 称为 边界 n-grams （edge n-grams）。

所谓的边界 n-gram 是说它会固定词语开始的一边，以单词 quick 为例，它的边界 n-gram 的结果为：

l q

l qu

l qui

l quic

l quick

#### 11.5.6  索引时输入即搜索与边界 n-grams 与邮编

##### 11.5.6.1  引时输入即搜索

设置索引时输入即搜索的第一步是需要定义好分析链

第一步需要配置一个自定义的 edge_ngram token 过滤器，称为 autocomplete_filter ：

然后会在一个自定义分析器 autocomplete 中使用上面这个 token 过滤器：

```
PUT /my_index
{
    "settings": {
        "number_of_shards": 1, 
        "analysis": {
            "filter": {
                "autocomplete_filter": { 
                    "type":     "edge_ngram",
                    "min_gram": 1,
                    "max_gram": 20
                }
            },
            "analyzer": {
                "autocomplete": {
                    "type":      "custom",
                    "tokenizer": "standard",
                    "filter": [
                        "lowercase",
                        "autocomplete_filter" 
                    ]
                }
            }
        }
    }
}
```

注：对于这个 token 过滤器接收的任意词项min_gram和max_gram，过滤器会为之生成一个最小固定值为 1 ，最大为 20 的 n-gram 。

 

可以拿 analyze API 测试这个新的分析器确保它行为正确：

```
GET /my_index/_analyze?analyzer=autocomplete
quick brown
```

结果表明分析器能正确工作，并返回以下词：

l q

l qu

l qui

l quic

l quick

l b

l br

l bro

l brow

l brown

 

可以用 update-mapping API 将这个分析器应用到具体字段：

```
PUT /my_index/_mapping/my_type
{
    "my_type": {
        "properties": {
            "name": {
                "type":     "string",
                "analyzer": "autocomplete"
            }
        }
    }
}
```

现在创建一些测试文档：

```
POST /my_index/my_type/_bulk
{ "index": { "_id": 1            }}
{ "name": "Brown foxes"    }
{ "index": { "_id": 2            }}
{ "name": "Yellow furballs" }
```

如果使用简单 match 查询测试查询 “brown fo” ：

```
GET /my_index/my_type/_search
{
    "query": {
        "match": {
            "name": "brown fo"
        }
    }
}
```

可以看到两个文档同时 都能 匹配，尽管 Yellow furballs 这个文档并不包含 brown 和 fo ：

 ![image-20200704003003358](../../../Typora/Picture/image-20200704003003358.png)

图 11—8

name:f 条件可以满足第二个文档，因为 furballs 是以 f 、 fu 、 fur 形式索引的。

##### 11.5.6.2  边界 n-grams 与邮编

边界 n-gram 的方式可以用来查询结构化的数据，postcode 字段需要 analyzed 而不是 not_analyzed ，不过可以用 keyword 分词器来处理它，就好像他们是 not_analyzed 的一样。

注：*keyword* *分词器是一个非操作型分词器，这个分词器不做任何事情，它接收的任何字符串都会被原样发出，因此它可以用来处理* *not_analyzed* *的字段值，但这也需要其他的一些分析转换，如将字母转换成小写。*

下面示例使用 keyword 分词器将邮编转换成 token 流，这样就能使用边界 n-gram token 过滤器：

```
{
    "analysis": {
        "filter": {
            "postcode_filter": {
                "type":     "edge_ngram",
                "min_gram": 1,
                "max_gram": 8
            }
        },
        "analyzer": {
            "postcode_index": { 
                "tokenizer": "keyword",
                "filter":    [ "postcode_filter" ]
            },
            "postcode_search": { 
                "tokenizer": "keyword"
            }
        }
    }
}
```

注：

\1.   postcode_index 分析器使用 postcode_filter 将邮编转换成边界 n-gram 形式。

\2.   postcode_search 分析器可以将搜索词看成 not_analyzed 未分析的。

#### 11.5.7  Ngrams 在复合词的应用

德语的特点是它可以将许多小词组合成一个庞大的复合词以表达它准确或复杂的意义。

例如：Weißkopfseeadler 秃鹰（White-headed sea eagle, or bald eagle）

有些人希望在搜索 “Wörterbuch”（字典）的时候，能在结果中看到 “Aussprachewörtebuch”（发音字典）。同样，搜索 “Adler”（鹰）的时候，能将 “Weißkopfseeadler”（秃鹰）包括在结果中。

假设某个 n-gram 是一个词上的滑动窗口，那么任何长度的 n-gram 都可以遍历这个词。我们既希望选择足够长的值让拆分的词项具有意义，又不至于因为太长而生成过多的唯一词。一个长度为 3 的 trigram 可能是一个不错的开始：

```
PUT /my_index
{
    "settings": {
        "analysis": {
            "filter": {
                "trigrams_filter": {
                    "type":     "ngram",
                    "min_gram": 3,
                    "max_gram": 3
                }
            },
            "analyzer": {
                "trigrams": {
                    "type":      "custom",
                    "tokenizer": "standard",
                    "filter":   [
                        "lowercase",
                        "trigrams_filter"
                    ]
                }
            }
        }
    },
    "mappings": {
        "my_type": {
            "properties": {
                "text": {
                    "type":     "string",
                    "analyzer": "trigrams" 
                }
            }
        }
    }
}
```

使用 analyze API 测试 trigram 分析器：

```
GET /my_index/_analyze
{
  "analyzer" : "trigrams",
  "text": "Weißkopfseeadler"
}
```

就可以使用match直接查询了。

### 11.6   控制相关度

#### 11.6.1  Lucene 的实用评分函数

Lucene 使用 布尔模型（Boolean model） 、 TF/IDF 以及 向量空间模型（vector space model） ，然后将它们组合到单个高效的包里以收集匹配文档并进行评分计算。评分计算公式实用评分函数

score(q,d) = queryNorm(q) · coord(q,d) · ∑ ( tf(t in d) · idf(t)² · t.getBoost() · norm(t,d) ) (t in q) 

注：

*1.*   *score(q,d)* *是文档* *d* *与查询* *q* *的相关度评分。*

*2.*   *queryNorm(q)* *是* *查询归一化* *因子* *（新）。*

*3.*   *coord(q,d)* *是* *协调* *因子* *（新）。*

*4.*   *查询* *q* *中每个词* *t* *对于文档* *d* *的权重和。*

*5.*   *tf(t in d)* *是词* *t* *在文档* *d* *中的* *词频* *。*

*6.*   *idf(t)* *是词* *t* *的* *逆向文档频率* *。*

*7.*   *t.getBoost()* *是查询中使用的* *boost**（新）。*

*8.*   *norm(t,d)* *是* *字段长度归一值* *，与* *索引时字段层* *boost* *（如果存在）的和（新）。*



##### 11.6.1.1  查询归一因子

查询归一因子 （ queryNorm ）试图将查询 归一化 ，这样就能将两个不同的查询结果相比较。

计算公式：

queryNorm = 1 / √sumOfSquaredWeights

注：*sumOfSquaredWeights* *是查询里每个词的* *IDF* *的平方和。*

##### 11.6.1.2  查询协调

协调因子 （ coord ）可以为那些查询词包含度高的文档提供奖励，文档里出现的查询词越多，它越有机会成为好的匹配结果。

设想查询 quick brown fox ，每个词的权重都是 1.5 。如果没有协调因子，最终评分会是文档里所有词权重的总和。例如：

l 文档里有 fox → 评分： 1.5

l 文档里有 quick fox → 评分： 3.0

l 文档里有 quick brown fox → 评分： 4.5

协调因子将评分与文档里匹配词的数量相乘，然后除以查询里所有词的数量，如果使用协调因子，评分会变成：

l 文档里有 fox → 评分： 1.5 * 1 / 3 = 0.5

l 文档里有 quick fox → 评分： 3.0 * 2 / 3 = 2.0

l 文档里有 quick brown fox → 评分： 4.5 * 3 / 3 = 4.5

协调因子能使包含所有三个词的文档比只包含两个词的文档评分要高出很多。

将协调功能关闭：

```
GET /_search
{
  "query": {
    "bool": {
      "disable_coord": true,
      "should": [
        { "term": { "text": "jump" }},
        { "term": { "text": "hop"  }},
        { "term": { "text": "leap" }}
      ]
    }
  }
}
```



#### 11.6.2  忽略 TF/IDF

###### 11.6.2.1.1  constant_score 查询

在 constant_score 查询中，它可以包含查询或过滤，为任意一个匹配的文档指定评分 1 ，忽略 TF/IDF 信息：

```
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "constant_score": {
          "query": { "match": { "description": "wifi" }}
        }},
        { "constant_score": {
          "query": { "match": { "description": "garden" }}
        }},
        { "constant_score": {
          "query": { "match": { "description": "pool" }}
        }}
      ]
    }
  }
}
```

或许不是所有的设施都同等重要——对某些用户来说有些设施更有价值。如果最重要的设施是游泳池，那我们可以为更重要的设施增加权重：

```
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "constant_score": {
          "query": { "match": { "description": "wifi" }}
        }},
        { "constant_score": {
          "query": { "match": { "description": "garden" }}
        }},
        { "constant_score": {
          "boost":   2 
          "query": { "match": { "description": "pool" }}
        }}
      ]
    }
  }
}
```

注：*pool* *语句的权重提升值为* *2* *，而其他的语句为* *1* *。*

#### 11.6.3  function_score 查询

function_score 查询允许为每个与主查询匹配的文档应用一个函数，以达到改变甚至完全替换原始查询评分 _score 的目的。

##### 11.6.3.1  Elasticsearch 预定义函数：

###### 11.6.3.1.1  weight

weight 函数与 boost 参数类似可以用于任何查询。有一点区别是 weight 没有被 Luence 归一化成难以理解的浮点数，而是直接被应用。为每个文档应用一个简单而不被规范化的权重提升值：当 weight 为 2 时，最终结果为 2 * _score 。

###### 11.6.3.1.2  field_value_factor

使用这个值来修改 _score ，如将 popularity 或 votes （受欢迎或赞）作为考虑因素。

###### 11.6.3.1.3  random_score

我们想让每个用户看到不同的随机次序，但也同时希望如果是同一用户翻页浏览时，结果的相对次序能始终保持一致。这种行为被称为 一致随机（consistently random） 。

random_score 函数会输出一个 0 到 1 之间的数，当种子 seed 值相同时，生成的随机结果是一致的，例如，将用户的会话 ID 作为 seed ：

```
GET /_search
{
  "query": {
    "function_score": {
      "filter": {
        "term": { "city": "Barcelona" }
      },
      "functions": [
        {
          "filter": { "term": { "features": "wifi" }},
          "weight": 1
        },
        {
          "filter": { "term": { "features": "garden" }},
          "weight": 1
        },
        {
          "filter": { "term": { "features": "pool" }},
          "weight": 2
        },
        {
          "random_score": { 
            "seed":  "the users session id" 
          }
        }
      ],
      "score_mode": "sum"
    }
  }
}
```

注：

*1.*   *random_score* *语句没有任何过滤器* *filter* *，所以会被应用到所有文档。*

*2.*   *将用户的会话* *ID* *作为种子* *seed* *，让该用户的随机始终保持一致，相同的种子* *seed* *会产生相同的随机结果。*

 

###### 11.6.3.1.4  衰减函数

有三种衰减函数—— linear 、 exp 和 gauss （线性、指数和高斯函数），它们可以操作数值、时间以及经纬度地理坐标点这样的字段。所有三个函数都能接受以下参数：

l **origin****：**中心点 或字段可能的最佳值，落在原点 origin 上的文档评分 _score 为满分 1.0 。

l **scale****：**衰减率，即一个文档从原点 origin 下落时，评分 _score 改变的速度。（例如，每 £10 欧元或每 100 米）。

l **decay****：**从原点 origin 衰减到 scale 所得的评分 _score ，默认值为 0.5 。

l **offset****：**以原点 origin 为中心点，为其设置一个非零的偏移量 offset 覆盖一个范围，而不只是单个原点。在范围 -offset <= origin <= +offset 内的所有评分 _score 都是 1.0 。

 ![image-20200704003042669](../../../Typora/Picture/image-20200704003042669.png)

图 11—9

near 、 exp 和 gauss （线性、指数和高斯）函数三者之间的区别在于范围（ origin +/- (offset + scale) ）之外的曲线形状：

l linear 线性函数是条直线，一旦直线与横轴 0 相交，所有其他值的评分都是 0.0 。

l exp 指数函数是先剧烈衰减然后变缓。

l gauss 高斯函数是钟形的——它的衰减速率是先缓慢，然后变快，最后又放缓。

选择曲线的依据完全由期望评分 _score 的衰减速率来决定，即距原点 origin 的值。

举例：用户希望租一个离伦敦市中心近（ { "lat": 51.50, "lon": 0.12} ）且每晚不超过 £100 英镑的度假屋，而且与距离相比，我们的用户对价格更为敏感，这样查询可以写成：

```
GET /_search
{
  "query": {
    "function_score": {
      "functions": [
        {
          "gauss": {
            "location": { 
              "origin": { "lat": 51.5, "lon": 0.12 },
              "offset": "2km",
              "scale":  "3km"
            }
          }
        },
        {
          "gauss": {
            "price": { 
              "origin": "50", 
              "offset": "50",
              "scale":  "20"
            }
          },
          "weight": 2 
        }
      ]
    }
  }
}
```

注：

*1.*   *location* *字段以地理坐标点* *geo_point* *映射。*

*2.*   *price* *字段是数值。*

*3.*   *参见* *理解价格语句* *，理解* *origin* *为什么是* *50* *而不是* *100* *。*

*4.*   *price* *语句是* *location* *语句权重的两倍。*

location 语句可以简单理解为：

以伦敦市中作为原点 origin 。

所有距原点 origin 2km 范围内的位置的评分是 1.0 。

距中心 5km （ offset + scale ）的位置的评分是 0.5

 

 

 

###### 11.6.3.1.5  script_score

如果需求超出以上范围时，用自定义脚本可以完全控制评分计算，实现所需逻辑。

使用举例：

```
GET /blogposts/post/_search
{
  "query": {
    "function_score": { 
      "query": { 
        "multi_match": {
          "query":    "popularity",
          "fields": [ "title", "content" ]
        }
      },
      "field_value_factor": { 
        "field": "votes" 
      }
    }
  }
}
```

注：

*1.*   *function_score* *查询将主查询和函数包括在内。*

*2.*   *主查询优先执行。*

*3.*   *field_value_factor* *函数会被应用到每个与主* *query* *匹配的文档。*

*4.*   *每个文档的* *votes* *字段都* *必须* *有值供* *function_score* *计算。如果* *没有* *文档的* *votes* *字段有值，那么就* *必须* *使用* *missing* *属性* *提供的默认值来进行评分计算。*

文档的最终评分 _score 都做了如下修改：

new_score = old_score * number_of_votes

 

 

 

 

#### 11.6.4  过滤集提升权重

function_score 查询中使用过滤就能很好地提升权重：

```
GET /_search
{
  "query": {
    "function_score": {
      "filter": { 
        "term": { "city": "Barcelona" }
      },
      "functions": [ 
        {
          "filter": { "term": { "features": "wifi" }}, 
          "weight": 1
        },
        {
          "filter": { "term": { "features": "garden" }}, 
          "weight": 1
        },
        {
          "filter": { "term": { "features": "pool" }}, 
          "weight": 2 
        }
      ],
      "score_mode": "sum", 
    }
  }
}
```

注：

*1.*     *function_score* *查询里面是* *filter* *过滤器而不是* *query* *查询。*

*2.*     *functions* *关键字存储着一个将被应用的函数列表。*

*3.*     *函数会被应用于和* *filter* *过滤器（可选的）匹配的文档。*

*4.*     *pool* *比其他特性更重要，所以它有更高* *weight* *。*

*5.*     *score_mode* *指定各个函数的值进行组合运算的方式。*

##### 11.6.4.1  函数 functions

functions 关键字保持着一个将要被使用的函数列表。可以为列表里的每个函数都指定一个 filter 过滤器，在这种情况下，函数只会被应用到那些与过滤器匹配的文档，例子中，我们为与过滤器匹配的文档指定权重值 weight 为 1 （为与 pool 匹配的文档指定权重值为 2 ）。

 

##### 11.6.4.2  评分模式 score_mode

每个函数返回一个结果，所以需要一种将多个结果缩减到单个值的方式，然后才能将其与原始评分 _score 合并。评分模式 score_mode 参数正好扮演这样的角色，它接受以下值：

l multiply：函数结果求积（默认）。

l sum：函数结果求和。

l avg：函数结果的平均值。

l max：函数结果的最大值。

l min：函数结果的最小值。

l first：使用首个函数（可以有过滤器，也可能没有）的结果作为最终结果

#### 11.6.5  脚本评分

可以自己用脚本一些逻辑

 

#### 11.6.6  Okapi BM25

能与 TF/IDF 和向量空间模型媲美的就是 Okapi BM25 ，它被认为是 当今最先进的 排序函数。BM25 源自 概率相关模型（probabilistic relevance model） ，而不是向量空间模型，但这个算法也和 Lucene 的实用评分函数有很多共通之处。

BM25 同样使用词频、逆向文档频率以及字段长归一化，但是每个因子的定义都有细微区别。与其详细解释 BM25 公式，倒不如将关注点放在 BM25 所能带来的实际好处上。

##### 11.6.6.1  词频饱和度

TF/IDF 和 BM25 同样使用 逆向文档频率 来区分普通词（不重要）和非普通词（重要），同样认为（参见 词频 ）文档里的某个词出现次数越频繁，文档与这个词就越相关。

不幸的是，普通词随处可见，实际上一个普通词在同一个文档中大量出现的作用会由于该词在 所有 文档中的大量出现而被抵消掉。

曾经有个时期，将 最 普通的词（或 停用词 ，参见 停用词）从索引中移除被认为是一种标准实践，TF/IDF 正是在这种背景下诞生的。TF/IDF 没有考虑词频上限的问题，因为高频停用词已经被移除了。

Elasticsearch 的 standard 标准分析器（ string 字段默认使用）不会移除停用词，因为尽管这些词的重要性很低，但也不是毫无用处。这导致：在一个相当长的文档中，像 the 和 and 这样词出现的数量会高得离谱，以致它们的权重被人为放大。

另一方面，BM25 有一个上限，文档里出现 5 到 10 次的词会比那些只出现一两次的对相关度有着显著影响。但是如下图所见，文档中出现 20 次的词几乎与那些出现上千次的词有着相同的影响。

这就是 非线性词频饱和度（nonlinear term-frequency saturation） 。TF/IDF 与 BM25 的词频饱和度；

 ![image-20200704003057679](../../../Typora/Picture/image-20200704003057679.png)

图 11—10

##### 11.6.6.2  字段长度归一化（Field-length normalization）

在之前的字段长归一化章节中，我们提到过 Lucene 会认为较短字段比较长字段更重要：字段某个词的频度所带来的重要性会被这个字段长度抵消，但是实际的评分函数会将所有字段以同等方式对待。它认为所有较短的 title 字段比所有较长的 body 字段更重要。

BM25 当然也认为较短字段应该有更多的权重，但是它会分别考虑每个字段内容的平均长度，这样就能区分短 title 字段和 长 title 字段。

**重要：**在 查询时权重提升 中，已经说过 title 字段因为其长度比 body 字段 自然 有更高的权重提升值。由于字段长度的差异只能应用于单字段，这种自然的权重提升会在使用 BM25 时消失。

##### 11.6.6.3  BM25 调优

不像 TF/IDF ，BM25 有一个比较好的特性就是它提供了两个可调参数：

l k1：这个参数控制着词频结果在词频饱和度中的上升速度。默认值为 1.2 。值越小饱和度变化越快，值越大饱和度变化越慢。

l b：这个参数控制着字段长归一值所起的作用， 0.0 会禁用归一化， 1.0 会启用完全归一化。默认值为 0.75 。

在实践中，调试 BM25 是另外一回事， k1 和 b 的默认值适用于绝大多数文档集合，但最优值还是会因为文档集不同而有所区别，为了找到文档集合的最优值，就必须对参数进行反复修改验证。



 

 

 

 

## 12   处理语言

### 12.1   使用各种语言

#### 12.1.1  使用语言分析器

```
PUT /my_index
{
  "mappings": {
    "blog": {
      "properties": {
        "title": {
          "type":     "string",
          "analyzer": "english" 
        }
      }
    }
  }
}
```

注：*title* *字段将会用* *english**（英语）分析器替换默认的* *standard**（标准）分析器*

使用_analyze分析

```
GET /my_index/_analyze
{
  "field" : "title",
  "text" : "I'm not happy about the foxes"
}
```

注：*全部分隔成为词根**i'm**，**happi**，**about**，**fox*

为了获得两方面的优势，我们可以使用multifields（多字段）对 title 字段建立两次索引： 一次使用 english（英语）分析器，另一次使用 standard（标准）分析器:

```
PUT /my_index
{
  "mappings": {
    "blog": {
      "properties": {
        "title": { 
          "type": "string",
          "fields": {
            "english": { 
              "type":     "string",
              "analyzer": "english"
            }
```



```
          }
        }
      }
    }
  }
}
```

在搜索时使用这两个字段：

```
PUT /my_index/blog/1
{ "title": "I'm happy for this fox" }
 
PUT /my_index/blog/2
{ "title": "I'm not happy about my fox problem" }
 
GET /my_index/_validate/query?explain
{
  "query": {
    "multi_match": {
      "type":     "most_fields", 
      "query":    "not happy foxes",
      "fields": [ "title", "title.english" ]
    }
  }
}
```

结果：

 ![image-20200704003112355](../../../Typora/Picture/image-20200704003112355.png)

图 12—1

分析器使用english的已经将not除掉了。

#### 12.1.2  自定义停用词和词干提取排除

为了自定义 english （英语）分词器的行为，我们需要基于 english （英语）分析器创建一个自定义分析器，然后添加一些配置：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_english": {
          "type": "english",
          "stem_exclusion": [ "organization", "organizations" ], 
          "stopwords": [ 
            "a", "an", "and", "are", "as", "at", "be", "but", "by", "for",
            "if", "in", "into", "is", "it", "of", "on", "or", "such", "that",
            "the", "their", "then", "there", "these", "they", "this", "to",
            "was", "will", "with"
          ]
        }
      }
    }
  }
}
 
```

注：

\1.   stem_exclusion：排除这些词转换成词干

\2.   stopwords：自定义停用词

#### 12.1.3  indices_boost为特定语言添加优先权

 

indices_boost 参数为特定的语言添加优先权：

 

 

#### 12.1.4  为多个字段设置分析器

PUT /movies

{

 "mappings": {

  "movie": {

   "properties": {

​    "title": { 

​     "type":    "string"

​    },

​    "title_br": { 

​      "type":   "string",

​      "analyzer": "brazilian"

​    },

​    "title_cz": { 

​      "type":   "string",

​      "analyzer": "czech"

​    },

​    "title_en": { 

​      "type":   "string",

​      "analyzer": "english"

​    },

​    "title_es": { 

​      "type":   "string",

​      "analyzer": "spanish"

​    }

   }

  }

 }

}

使用 n-grams

PUT /movies

{

 "settings": {

  "analysis": {...} 

 },

 "mappings": {

  "title": {

   "properties": {

​    "title": {

​     "type": "string",

​     "fields": {

​      "de": {

​       "type":   "string",

​       "analyzer": "german"

​      },

​      "en": {

​       "type":   "string",

​       "analyzer": "english"

​      },

​      "fr": {

​       "type":   "string",

​       "analyzer": "french"

​      },

​      "es": {

​       "type":   "string",

​       "analyzer": "spanish"

​      },

​      "general": { 

​       "type":   "string",

​       "analyzer": "trigrams"

​      }

​     }

​    }

   }

  }

 }

}

注：

\1.    在 analysis 章节, 我们按照 Ngrams 在复合词的应用 中描述的定义了同样的 trigrams 分析器。

\2.   在 title.general 域使用 trigrams 分析器索引所有的语言。

### 12.2   词汇识别

#### 12.2.1  自定义标准分析器

{

  "type":   "custom",

  "tokenizer": "standard",

  "filter": [ "lowercase", "stop" ]

}

#### 12.2.2  ICU分词器

##### 12.2.2.1  ICU 插件

Elasticsearch的 [ICU 分析器插件](https://github.com/elasticsearch/elasticsearch-analysis-icu) 使用 国际化组件 Unicode (ICU) 函数库（详情查看 site.project.org ）提供丰富的处理 Unicode 工具。 这些包含对处理亚洲语言特别有用的 icu_分词器 ，还有大量对除英语外其他语言进行正确匹配和排序所必须的分词过滤器。

##### 12.2.2.2  icu_分词器

icu_分词器 和 标准分词器 使用同样的 Unicode 文本分段算法， 只是为了更好的支持亚洲语，添加了泰语、老挝语、中文、日文、和韩文基于词典的词汇识别方法，并且可以使用自定义规则将缅甸语和柬埔寨语文本拆分成音节。

GET /_analyze

{

 "tokenizer" : "icu_tokenizer",

 "text" : "สวัสดี ผมมาจากกรุงเทพฯ"

}

#### 12.2.3  HTML 分词

用 html_strip 字符过滤器移除 HTML 标签并编码 HTML 实体如 &eacute; 为一致的 Unicode 字符。

定义HTML分析器my_html_analyzer：

PUT /my_index

{

  "settings": {

​    "analysis": {

​      "analyzer": {

​        "my_html_analyzer": {

​          "tokenizer":   "standard",

​          "char_filter": [ "html_strip" ]

​        }

​      }

​    }

  }

}

使用该分析器：

GET /my_index/_analyze

{

 "analyzer" : "my_html_analyzer",

 "text" : "<p>Some d&eacute;j&agrave; vu <a>website</a>"

}

#### 12.2.4  整理标点符号

标准分词器 和 icu_分词器 能理解单词中的一些特殊“撇号”，直接将这些符号视为单词的一部分。

Unicode 列出了一些有时会被用为撇号的字符：

l U+0027：撇号标记为 (')— 原始 ASCII 符号

l U+2018：左单引号标记为 (‘)— 当单引用时作为一个引用的开始

l U+2019：右单引号标记为 (’)— 当单引用时座位一个引用的结束，也是撇号的首选字符。

当这三个字符出现在单词中间的时候， 标准分词器 和 icu_分词器 都会将这三个字符视为撇号（这会被视为单词的一部分）。 然而还有另外三个长得很像撇号的字符：

l U+201B：Single high-reversed-9 （高反单引号）标记为 (‛)— 跟 U+2018 一样，但是外观上有区别

l U+0091：ISO-8859-1 中的左单引号 — 不会被用于 Unicode 中

l U+0092：ISO-8859-1 中的右单引号 — 不会被用于 Unicode 中

标准分词器 和 icu_分词器 把这三个字符视为单词的分界线 — 一个将文本拆分为词汇单元的位置。

我们可以简单的用 U+0027 替换所有的撇号变体：

PUT /my_index

{

 "settings": {

  "analysis": {

   "char_filter": { 

​    "quotes": {

​     "type": "mapping",

​     "mappings": [ 

​      "\\u0091=>\\u0027",

​      "\\u0092=>\\u0027",

​      "\\u2018=>\\u0027",

​      "\\u2019=>\\u0027",

​      "\\u201B=>\\u0027"

​     ]

​    }

   },

   "analyzer": {

​    "quotes_analyzer": {

​     "tokenizer":   "standard",

​     "char_filter": [ "quotes" ] 

​    }

   }

  }

 }

}

创建了分析器后测试：

GET /my_index/_analyze

{

 "analyzer" : "quotes_analyzer",

 "text" : "You’re my ‘favorite’ M‛Coy"

}

### 12.3   Token 词过滤器

#### 12.3.1  asciifolding 字符过滤器

asciifolding 字符过滤器能去掉变音符号，同时，它会把Unicode字符转化为ASCII来表示:

·     `ß` ⇒ `ss`

·     `æ` ⇒ `ae`

·     `ł` ⇒ `l`

·     `ɰ` ⇒ `m`

·     `⁇` ⇒ `??`

·     `❷` ⇒ `2`

·     `⁶` ⇒ `6`

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "folding": {
          "tokenizer": "standard",
          "filter":  [ "lowercase", "asciifolding" ]
        }
      }
    }
  }
}
 
GET /my_index?analyzer=folding
My œsophagus caused a débâcle 
```

注：*得到的词元* *my, oesophagus, caused, a, debacle*

#### 12.3.2  folding 分析器

folding 分析器，会去掉变音符号

```
PUT /my_index/_mapping/my_type
{
  "properties": {
    "title": { 
      "type":           "string",
      "analyzer":       "standard",
      "fields": {
        "folded": { 
          "type":       "string",
          "analyzer":   "folding"
        }
      }
    }
  }
}
```

注：

*1.*   *在* *title* *字段用* *standard* *分析器，会保留原文的变音符号**.*

*2.*   *在* *title.folded* *字段用* *folding* *分析器，会去掉变音符号*

#### 12.3.3  规范模式和兼容模式

规范 (canonical) 模式—nfc 和 nfd&—把连字作为单个字符，例如 ﬃ 或者 œ 。 兼容 (compatibility) 模式—nfkc 和 nfkd—将这些组合的字符分解成简单字符的等价物，例如： f + f + i 或者 o + e.

使用 icu_normalizer 语汇单元过滤器(token filters) 来保证所有词元(token)是相同模式：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "nfkc_normalizer": { 
          "type": "icu_normalizer",
          "name": "nfkc"
        }
      },
      "analyzer": {
        "my_normalizer": {
          "tokenizer": "icu_tokenizer",
          "filter":  [ "nfkc_normalizer" ]
        }
      }
    }
  }
}
```

注：*用* *nfkc* *归一化**(normalization)**模式来归一化**(Normalize)**所有词元**(token).*

#### 12.3.4  Unicode 大小写折叠

nfkc_cf`等价于 `lowercase 语汇单元过滤器(token filters)，但是却适用于所有的语言。 on-steroids 等价于 standard 分析器，例如：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_lowercaser": {
          "tokenizer": "icu_tokenizer",
          "filter":  [ "icu_normalizer" ] 
        }
      }
    }
  }
}
```

注：*icu_normalizer* *默认是* *nfkc_cf* *模式**.*

测试：

```
GET /_analyze?analyzer=standard 
Weißkopfseeadler WEISSKOPFSEEADLER
```

注：*得到的词元**(token)**是* *weißkopfseeadler, weisskopfseeadler*

#### 12.3.5  icu_folding语汇单元过滤器

`icu_folding` 语汇单元过滤器(token filters) (provided by the <<icu-plugin,`icu` plug-in>>)的功能和 `asciifolding` 过滤器一样， ((("icu_folding token filter")))但是它扩展到了非ASCII编码的语言，例如：希腊语，希伯来语，汉语。它把这些语言都转换对应拉丁文字，甚至包含它们的各种各样的计数符号，象形符号和标点符号。

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_folder": {
          "tokenizer": "icu_tokenizer",
          "filter":  [ "icu_folding" ]
        }
      }
    }
  }
}
 
GET /my_index/_analyze?analyzer=my_folder
١٢٣٤٥ 
```

注：*阿拉伯数字* *١٢٣٤٥* *被折叠成等价的拉丁数字**: 12345.*

如果你有指定的字符不想被折叠，你可以使用 UnicodeSet(像字符的正则表达式) 来指定哪些Unicode才可以被折叠。例如：瑞典单词 å,ä, ö, Å, Ä, 和 Ö 不能被折叠，你就可以设定为： [^åäöÅÄÖ] (^ 表示 不包含)。这样就会对于所有的Unicode字符生效。

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "swedish_folding": { 
          "type": "icu_folding",
          "unicodeSetFilter": "[^åäöÅÄÖ]"
        }
      },
      "analyzer": {
        "swedish_analyzer": { 
          "tokenizer": "icu_tokenizer",
          "filter":  [ "swedish_folding", "lowercase" ]
        }
      }
    }
  }
}
```

注：

*1.*   *`swedish_folding`**语汇单元过滤器**(token filters)* *定制了* *`icu_folding`**语汇单元过滤器**(token filters)**来不处理那些大写和小写的瑞典单词。*

*2.*   *swedish* *分析器首先分词，然后用**`swedish_folding`**语汇单元过滤器来折叠单词，最后把他们走转换为小写，除了被排除在外的单词：* *Å, Ä,* *或者* *Ö**。*

#### 12.3.6  排序和整理

将BROWN 、 Boffey 、 bailey三个单词转换成小写后进行排序

单个小写词汇单元的分析器：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "case_insensitive_sort": {
          "tokenizer": "keyword",    
          "filter":  [ "lowercase" ] 
        }
      }
    }
  }
}
```

注：  

*1.*   *keyword* *分词器将输入的字符串原封不动的输出。*

*2.*   *lowercase* *分词过滤器将词汇单元转化为小写字母。*

```
PUT /my_index/_mapping/user
{
  "properties": {
    "name": {
      "type": "string",
      "fields": {
        "lower_case_sort": { 
          "type":     "string",
          "analyzer": "case_insensitive_sort"
        }
      }
    }
  }
}
 
PUT /my_index/user/1
{ "name": "Boffey" }
 
PUT /my_index/user/2
{ "name": "BROWN" }
 
PUT /my_index/user/3
{ "name": "bailey" }
 
GET /my_index/user/_search?sort=name.lower_case_sort
```

排序后的结果为bailey 、 Boffey 、 BROWN。

### 12.4   将单词还原为词根

#### 12.4.1  词干提取算法

porter_stem 词干提取器、 kstem 分词过滤器， snowball 分词过滤器、light_english 词干提取器

```
{
  "settings": {
    "analysis": {
      "filter": {
        "english_stop": {
          "type":       "stop",
          "stopwords":  "_english_"
        },
        "english_keywords": {
          "type":       "keyword_marker", 
          "keywords":   []
        },
        "english_stemmer": {
          "type":       "stemmer",
          "language":   "english" 
        },
        "english_possessive_stemmer": {
          "type":       "stemmer",
          "language":   "possessive_english" 
        }
      },
      "analyzer": {
        "english": {
          "tokenizer":  "standard",
          "filter": [
            "english_possessive_stemmer",
            "lowercase",
            "english_stop",
            "english_keywords",
            "english_stemmer"
          ]
        }
      }
    }
  }
}
```

注：

*1.*   *keyword_marker* *分词过滤器列出那些不用被词干提取的单词。这个过滤器默认情况下是一个空的列表。*

l *english* *分析器使用了两个词干提取器：* *possessive_english* *词干提取器**和* *english* *词干提取器。* *所有格词干提取器会在任何词传递到* *english_stop* *、* *english_keywords* *和* *english_stemmer* *之前去除* *'s* *。*

#### 12.4.2  字典词干提取器

字典词干提取器 在工作机制上与 算法化词干提取器 完全不同。 不同于应用一系列标准规则到每个词上，字典词干提取器只是简单地在字典里查找词。理论上可以给出比算法化词干提取器更好的结果。一个字典词干提取器应当可以：

l 返回不规则形式如 feet 和 mice 的正确词干

l 区分出词形相似但词义不同的情形，比如 organ and organization

#### 12.4.3  Hunspell 词干提取器（待）

Elasticsearch 提供了基于词典提取词干的 hunspell 语汇单元过滤器（token filter）. Hunspell hunspell.github.io 是一个 Open Office、LibreOffice、Chrome、Firefox、Thunderbird 等众多其它开源项目都在使用的拼写检查器。

#### 12.4.4  词干提取器概述，举例，性能

在文档 stemmer token filter 里面列出了一些针对语言的若干词干提取器。 就英语来说我们有如下提取器：

l english：porter_stem 语汇单元过滤器（token filter）。

l light_english：kstem 语汇单元过滤器（token filter）。

l minimal_english：Lucene 里面的 EnglishMinimalStemmer ，用来移除复数。

l lovins：基于 Snowball 的 Lovins 提取器, 第一个词干提取器。

l porter：基于 Snowball 的 Porter 提取器。

l porter2：基于 Snowball 的 Porter2 提取器。

l possessive_english：Lucene 里面的 EnglishPossessiveFilter ，移除 's

Hunspell 词干提取器也要纳入到上面的列表中，还有多种英文的词典可用。

#### 12.4.5  词干提取控制

语汇单元过滤器 keyword_marker 和 stemmer_override 能让我们自定义词干提取过程。

##### 12.4.5.1  阻止词干提取

语言分析器（查看 配置语言分析器）的参数 stem_exclusion 允许我们指定一个词语列表，让他们不被词干提取。

在内部，这些语言分析器使用 keyword_marker 语汇单元过滤器 来标记这些词语列表为 keywords ，用来阻止后续的词干提取过滤器来触碰这些词语。

创建一个简单自定义分析器，使用 porter_stem 语汇单元过滤器，同时阻止 skies 的词干提取：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "no_stem": {
          "type": "keyword_marker",
          "keywords": [ "skies" ] 
        }
      },
      "analyzer": {
        "my_english": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "no_stem",
            "porter_stem"
          ]
        }
      }
    }
  }
}
```

注：  *参数* *keywords* *可以允许接收多个词语。*

analyze API 来测试，可以看到词 skies 没有被提取：

```
GET /my_index/_analyze?analyzer=my_english
sky skies skiing skis 
```

注：*返回**: sky, skies, ski, ski*

词干并没有被提取

##### 12.4.5.2  自定义提取

The stemmer_override 语汇单元过滤器允许我们指定自定义的提取规则。 与此同时，我们可以处理一些不规则的形式，如：mice 提取为 mouse 和 feet 到 foot ：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "custom_stem": {
          "type": "stemmer_override",
          "rules": [ 
            "skies=>sky",
            "mice=>mouse",
            "feet=>foot"
          ]
        }
      },
      "analyzer": {
        "my_english": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "custom_stem", 
            "porter_stem"
          ]
        }
      }
    }
  }
}
 
GET /my_index/_analyze?analyzer=my_english
The mice came down from the skies and ran over my feet 
```

注：

\1. *规则来自* *original=>stem* *。*

*2.*   *stemmer_override* *过滤器必须放置在词干提取器之前。*

*3.*   *返回* *the, mouse, came, down, from, the, sky, and, ran, over, my, foot* *。*

#### 12.4.6  原形词干提取

将已提取词干的词和原词索引到同一个字段中。举个例子，分析句子 The quick foxes jumped 将会得到以下词项：

```
Pos 1: (the)
Pos 2: (quick)
Pos 3: (foxes,fox) 
Pos 4: (jumped,jump) 
```

### 12.5   停用词:

#### 12.5.1  停用词的优缺点

英语默认的停用词为:

a, an, and, are, as, at, be, but, by, for, if, in, into, is, it,

no, not, of, on, or, such, that, the, their, then, there, these,

they, this, to, was, will, with

**优点：**

\1.   提高检索性能

\2.   节省空间，但是33 个常见词从索引中移除，每百万文档只能节省 4MB 空间，因此停用词减少索引大小不再是一个有效的理由。

\3.   屏蔽了常见词，使检索不会因为有常见词而乱打分，如几乎所有文档都包含the，因为the打分会降低结果的可信度。

**缺点：**

\1.   将这些词移除会使我们降低某种类型的搜索能力，如区分 happy 和 not happy。

#### 12.5.2  自定义使用停用词

标准分析器能与自定义停用词表连用

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": { 
          "type": "standard", 
          "stopwords": [ "and", "the" ] 
        }
      }
    }
  }
}
```

过滤掉and，the。

使用分析器测试：

```
GET /my_index/_analyze?analyzer=my_analyzer
The quick and the dead
```

结果

 ![image-20200704003152527](../../../Typora/Picture/image-20200704003152527.png)

图 12—2

#### 12.5.3  词项的分别管理

##### 12.5.3.1  cutoff_frequency区分高低频词

在查询字符串中的词项可以分为更重要（低频词）和次重要（高频词）这两类。 只与次重要词项匹配的文档很有可能不太相关。实际上，我们想要文档能尽可能多的匹配那些更重要的词项。

match 查询接受一个参数 cutoff_frequency ，从而可以让它将查询字符串里的词项分为低频和高频两组。

```
{
  "match": {
    "text": {
      "query": "Quick and the dead",
      "cutoff_frequency": 0.01 
    }
}
```

注：*任何词项出现在文档中超过**1%**，被认为是高频词。**cutoff_frequency* *配置可以指定为一个分数（* *0.01* *）或者一个正整数（* *5* *）*

##### 12.5.3.2  匹配低频词

可以让所有低频词都必须匹配，而只对那些包括超过 75% 的高频词文档进行评分：

```
{
  "common": {
    "text": {
      "query":                  "Quick and the dead",
      "cutoff_frequency":       0.01,
      "low_freq_operator":      "and",
      "minimum_should_match": {
        "high_freq":            "75%"
      }
    }
  }
}
```

#### 12.5.4  停用词与短语查询（待）

所有查询中 短语匹配 大约占到5%，但是在慢查询里面它们又占大部分。 短语查询性能相对较差，特别是当短语中包括常用词的时候，如 “To be, or not to be” 短语全部由停用词组成，这是一种极端情况。原因在于几乎需要匹配全量的数据。

典型的索引会可能包含部分或所有以下数据：

l 词项字典（Terms dictionary）：索引中所有文档内所有词项的有序列表，以及包含该词的文档数量。

l 倒排表（Postings list）：包含每个词项的文档（ID）列表。

l 词频（Term frequency）：每个词项在每个文档里出现的频率。

l 位置（Positions）：每个词项在每个文档里出现的位置，供短语查询或近似查询使用。

l 偏移（Offsets）：每个词项在每个文档里开始与结束字符的偏移，供词语高亮使用，默认是禁用的。



l 规范因子（Norms）：用来对字段长度进行规范化处理的因子，给较短字段予以更多权重。

将停用词从索引中移除会节省 词项字典 和 倒排表 里的少量空间，但 位置 和 偏移 是另一码事。位置和偏移数据很容易变成索引大小的两倍、三倍、甚至四倍。

#### 12.5.5  common_grams 过滤器

common_grams 过滤器是针对短语查询能更高效的使用停用词而设计的。 它与 shingles 过滤器类似（参见 查找相关词（寻找相关词)), 为每个相邻词对生成。

common_grams 过滤器根据 query_mode 设置的不同而生成不同输出结果：false （为索引使用） 或 true （为搜索使用），所以我们必须创建两个独立的分析器：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "index_filter": { 
          "type":         "common_grams",
          "common_words": "_english_" 
        },
        "search_filter": { 
          "type":         "common_grams",
          "common_words": "_english_", 
          "query_mode":   true
        }
      },
      "analyzer": {
        "index_grams": { 
          "tokenizer":  "standard",
          "filter":   [ "lowercase", "index_filter" ]
        },
        "search_grams": { 
          "tokenizer": "standard",
          "filter":  [ "lowercase", "search_filter" ]
        }
      }
    }
  }
}
```

注：

*1.*   *index_filter**、**search_filter**：首先我们基于* *common_grams* *过滤器创建两个过滤器：* *index_filter* *在索引时使用（此时* *query_mode* *的默认设置是* *false* *），* *search_filter* *在查询时使用（此时* *query_mode* *的默认设置是* *true* *）。*

*2.*   *common_words**：**common_words* *参数可以接受与* *stopwords* *参数同样的选项（参见* *指定停用词* *指定停用词（**Specifying Stopwords**）* *）。这个过滤器还可以接受参数* *common_words_path* *，使用存于文件里的常用词。*

*3.*   *index_grams**、**search_grams**：然后我们使用过滤器各创建一个索引时分析器和查询时分析器。*

### 12.6   同义词

#### 12.6.1  自定义同义词

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_synonym_filter": {
          "type": "synonym", 
          "synonyms": [ 
            "british,english",
            "queen,monarch"
          ]
        }
      },
      "analyzer": {
        "my_synonyms": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_synonym_filter" 
          ]
        }
      }
    }
  }
}
```

注：自定义同义词queen，monarch

#### 12.6.2  同义词的扩展或收缩

##### 12.6.2.1  简单扩展

通过 简单扩展 ，我们可以把同义词列表中的任意一个词扩展成同义词列表 所有 的词：

"jump,hop,leap"

 

##### 12.6.2.2  简单收缩

简单收缩 ，把 左边的多个同义词映射到了右边的单个词：

"leap,hop => jump"

举例：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_synonym_filter": {
          "type": "synonym",
          "synonyms": [
            "united states,u s a,united states of america=>usa"
          ]
        }
      },
      "analyzer": {
        "my_synonyms": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_synonym_filter"
          ]
        }
      }
    }
  }
}
```

查询的时候遇到与synonyms相同的词，都会被标记为同义词，同时转换为usa后输出

 

 

##### 12.6.2.3  类型扩展

类型扩展是完全不同于简单收缩 或扩张， 并不是平等看待所有的同义词，而是扩大了词的意义，使被拓展的词更为通用。以这些规则为例：

l "cat  => cat,pet",

l "kitten => kitten,cat,pet",

l "dog  => dog,pet"

l "puppy => puppy,dog,pet"

通过在索引阶段使用类型扩展：

l 一个关于 kitten 的查询会发现关于 kittens 的文档。

l 查询一个 cat 会找到关于 kittens 和 cats 的文档。

l 一个 pet 的查询将发现有关的 kittens、cats、puppies、dogs 或者 pets 的文档。

类型扩展举例：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_synonym_filter": {
          "type": "synonym",
          "synonyms": [
            "cat    => cat,pet"
          ]
        }
      },
      "analyzer": {
        "my_synonyms": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_synonym_filter"
          ]
        }
      }
    }
  }
}
```

只要是搜索cat，含有cat、pet的文档都会被搜索出来。

#### 12.6.3  同义词和分析链

思考一个问题，那么为什么我们使用的不是 U.S.A. 呢？

假设我们有一个分析器，它由 standard 分词器、 lowercase 的语汇单元过滤器、 synonym 的语汇单元过滤器组成。文本 U.S.A. 的分析过程，看起来像这样的：

```
original string（原始文本）                       → "U.S.A."
standard           tokenizer（分词器）            → (U),(S),(A)
lowercase          token filter（语汇单元过滤器）  → (u),(s),(a)
synonym            token filter（语汇单元过滤器）  → (usa)
```

如果我们有指定的同义词 U.S.A. ，它永远不会匹配任何东西。因为， my_synonym_filter 看到词项的时候，句号已经被移除了，并且字母已经被小写了。

**大小写敏感的同义词**

很多词因为大小写的原因，到导致意思会有较大差距，查询的时候就会出现查询的结果与自己查找的东西大相径庭，这值得考虑的地方。

#### 12.6.4  符号同义词

和前面讲的同义词处理不太一样。 符号同义词 是用别名来表示这个符号，以防止它在分词过程中被误认为是不重要的标点符号而被移除。

字符组合而成的表情，有时是很有意义的东西，甚至有时候会改变整个句子的含义，对比一下这两句话：

l 我很高兴能在星期天工作。

l 我很高兴能在星期天工作 :( （注：难过的表情）

使用 映射字符过滤器，在文本被递交给分词器处理之前， 把字符表情替换成符号同义词 emoticon_happy 或者 emoticon_sad ：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "char_filter": {
        "emoticons": {
          "type": "mapping",
          "mappings": [ 
            ":)=>emoticon_happy",
            ":(=>emoticon_sad"
          ]
        }
      },
      "analyzer": {
        "my_emoticons": {
          "char_filter": "emoticons",
          "tokenizer":   "standard",
          "filter":    [ "lowercase" ]
          ]
        }
      }
    }
  }
}
 
GET /my_index/_analyze?analyzer=my_emoticons
I am :) not :( 
```

注：

\1.    映射（mappings）过滤器把字符从 => 左边的格式转变成右边的样子。

\2.   输出： i 、 am 、 emoticon_happy 、 not 、 emoticon_sad 。

### 12.7   拼写错误

#### 12.7.1  模糊性

模糊性：度量从一个单词转换到另一个单词需要多少次单字符编辑

举个例子，将单词 bieber 转换成 beaver 需要下面几个步骤：

l 把 b 替换成 v ：bie_b_er → bie_v_er

l 把 i 替换成 a ：b_i_ever → b_a_ ever

l 把 e 和 a 进行换位：b_ae_ver → b_ea_ver

这三个步骤表示 Damerau-Levenshtein edit distance 编辑距离为 3 。

Elasticsearch 指定了 fuzziness 参数支持对最大编辑距离的配置，默认为 ２ 。

fuzziness 参数可以被设置为 AUTO ，这将导致以下的最大编辑距离：

l 字符串只有 1 到 2 个字符时是 0

l 字符串有 3 、 4 或者 5 个字符时是 1

l 字符串大于 5 个字符时是 2

#### 12.7.2  模糊查询

##### 12.7.2.1  fuzzy 查询

举例

POST /my_index/my_type/_bulk

{ "index": { "_id": 1 }}

{ "text": "Surprise me!"}

{ "index": { "_id": 2 }}

{ "text": "That was surprising."}

{ "index": { "_id": 3 }}

{ "text": "I wasn't surprised."}

词 surprize 运行一个 fuzzy 查询：

GET /my_index/my_type/_search

{

 "query": {

  "fuzzy": {

   "text": "surprize"

  }

 }

}

注：*fuzziness* *默认设置为* *AUTO* *。*

自己设置fuzziness：

GET /my_index/my_type/_search

{

 "query": {

  "fuzzy": {

   "text": {

​    "value": "surprize",

​    "fuzziness": 1

   }

  }

 }

}

##### 12.7.2.2  提高fuzzy 查询性能

下面两个参数可以用来限制对性能的影响：

 **prefix_length**

不能被 “模糊化” 的初始字符数。 大部分的拼写错误发生在词的结尾，而不是词的开始。 例如通过将 prefix_length 设置为 3 ，你可能够显著降低匹配的词项数量。

 **max_expansions**

如果一个模糊查询扩展了三个或四个模糊选项， 这些新的模糊选项也许是有意义的。如 果它产生 1000 个模糊选项，那么就基本没有意义了。 设置 max_expansions 用来限制将产生的模糊选项的总数量。模糊查询将收集匹配词项直到达到 max_expansions 的限制。

#### 12.7.3  模糊匹配（match）查询

match 查询支持开箱即用的模糊匹配：

GET /my_index/my_type/_search

{

 "query": {

  "match": {

   "text": {

​    "query":   "SURPRIZE ME!",

​    "fuzziness": "AUTO",

​    "operator": "and"

   }

  }

 }

}

同样， multi_match 查询也支持 fuzziness ，但只有当执行查询时类型是 best_fields 或者 most_fields ：

GET /my_index/my_type/_search

{

 "query": {

  "multi_match": {

   "fields": [ "text", "title" ],

   "query":   "SURPRIZE ME!",

   "fuzziness": "AUTO"

  }

 }

}

注：*match* *和* *multi_match* *查询都支持* *prefix_length* *和* *max_expansions* *参数。*

#### 12.7.4  模糊性评分

假设我们有1000个文档包含 ``Schwarzenegger`` ，只是一个文档的出现拼写错误 ``Schwarzeneger`` 。如果根据逆向文档频率，这个拼写错误文档比拼写正确的相关度更高，因为错误拼写出现在更少的文档中！ 

因此，模糊匹配不应用于参与评分—只能在有拼写错误时扩大匹配项的范围。

默认情况下， match 查询给定所有的模糊匹配的恒定评分为1。这可以满足在结果列表的末尾添加潜在的匹配记录，并且没有干扰非模糊查询的相关性评分。

#### 12.7.5  语音匹配

在尝试任何其他匹配方法都无效后，我们可以求助于搜索发音相似的词，即使他们的拼写不同。

有一些用于将词转换成语音标识的算法。 Soundex 算法是这些算法的鼻祖， 而且大多数语音算法是 Soundex 的改进或者专业版本。

**安装**

从 https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis-phonetic.html 获取语音分析插件并在集群的每个节点安装， 然后重启每个节点。

创建一个使用语音语汇单元过滤器的自定义分析器，并尝试下面的方法：

PUT /my_index

{

 "settings": {

  "analysis": {

   "filter": {

​    "dbl_metaphone": { 

​     "type":  "phonetic",

​     "encoder": "double_metaphone"

​    }

   },

   "analyzer": {

​    "dbl_metaphone": {

​     "tokenizer": "standard",

​     "filter":  "dbl_metaphone" 

​    }

   }

  }

 }

}

注：

*1.*    *首先，配置一个自定义* *phonetic* *语汇单元过滤器并使用* *double_metaphone* *编码器。*

*2.*   *然后在自定义分析器中使用自定义语汇单元过滤器。*

通过 analyze API 来进行测试：

GET /my_index/_analyze?analyzer=dbl_metaphone

Smith Smythe

## 13   聚合（待）

### 13.1   高阶概念

要掌握聚合，需要明白两个主要的概念：

l **桶（****Buckets****）：**满足特定条件的文档的集合

l **指标（****Metrics****）：**对桶内的文档进行统计计算

粗略的用SQL语句来解释：

SELECT COUNT(color) 

FROM table

GROUP BY color 

注：

*1.*   *COUNT(color)* *相当于指标。*

*2.*   *GROUP BY color* *相当于桶。*

桶在概念上类似于 SQL 的分组（GROUP BY），而指标则类似于 COUNT() 、 SUM() 、 MAX() 等统计方法。

### 13.2   使用聚合

#### 13.2.1  聚合举例

根据车颜色统计车受欢迎程度

首先设置fielddata

PUT /cars

{

 "mappings": {

  "transactions" :{

   "properties" :{

​    "color" : {

​     "type" : "text",

​     "fielddata" :true

​    }

   }

  }

 }

}

输入数据

POST /cars/transactions/_bulk

{ "index": {}}

{ "price" : 10000, "color" : "red", "make" : "honda", "sold" : "2014-10-28" }

{ "index": {}}

{ "price" : 20000, "color" : "red", "make" : "honda", "sold" : "2014-11-05" }

{ "index": {}}

{ "price" : 30000, "color" : "green", "make" : "ford", "sold" : "2014-05-18" }

{ "index": {}}

{ "price" : 15000, "color" : "blue", "make" : "toyota", "sold" : "2014-07-02" }

{ "index": {}}

{ "price" : 12000, "color" : "green", "make" : "toyota", "sold" : "2014-08-19" }

{ "index": {}}

{ "price" : 20000, "color" : "red", "make" : "honda", "sold" : "2014-11-05" }

{ "index": {}}

{ "price" : 80000, "color" : "red", "make" : "bmw", "sold" : "2014-01-01" }

{ "index": {}}

{ "price" : 25000, "color" : "blue", "make" : "ford", "sold" : "2014-02-12" }

车颜色统计车受欢迎程度

GET /cars/transactions/_search

{

  "size" : 0,

  "aggs" : { 

​    "popular_colors" : { 

​      "terms" : { 

​       "field" : "color"

​      }

​    }

  }

}



#### 13.2.2  fielddata内存数据结构

在es中，text类型的字段使用一种叫做fielddata的查询时内存数据结构。当字段被排序，聚合或者通过脚本访问时这种数据结构会被创建。它是通过从磁盘读取每个段的整个反向索引来构建的，然后存存储在java的堆内存中。

fileddata默认是不开启的。Fielddata可能会消耗大量的堆空间，尤其是在加载高基数文本字段时。一旦fielddata已加载到堆中，它将在该段的生命周期内保留。此外，加载fielddata是一个昂贵的过程，可能会导致用户遇到延迟命中。

##### 13.2.2.1  Fielddata的大小

indices.fielddata.cache.size 控制为 fielddata 分配的堆空间大小。 当你发起一个查询，分析字符串的聚合将会被加载到 fielddata，如果这些字符串之前没有被加载过。如果结果中 fielddata 大小超过了指定 大小 ，其他的值将会被回收从而获得空间。

##### 13.2.2.2  监控 fielddata

Fielddata 的使用可以被监控：

按索引使用 indices-stats API ：

```
GET /_stats/fielddata?fields=*
```

按节点使用 nodes-stats API ：

```
GET /_nodes/stats/indices/fielddata?fields=*
```

按索引节点：

```
GET /_nodes/stats/indices/fielddata?level=indices&fields=*
```

##### 13.2.2.3  断路器

机敏的读者可能已经发现 fielddata 大小设置的一个问题。fielddata 大小是在数据加载 之后 检查的。 如果一个查询试图加载比可用内存更多的信息到 fielddata 中会发生什么？答案：我们会碰到 OutOfMemoryException 。

#### 13.2.3  桶里嵌套度量指标

继续为汽车的例子加入 average 平均度量：

GET /cars/transactions/_search

{

  "size" : 0,

  "aggs": {

   "colors": {

​     "terms": {

​      "field": "color"

​     },

​     "aggs": { 

​      "avg_price": { 

​        "avg": {

​         "field": "price" 

​        }

​      }

​     }

   }

  }

}

注：

*1.*    *为度量新增* *aggs* *层。*

*2.*   *为度量指定名字：* *avg_price* *。*

*3.*   *最后，为* *price* *字段定义* *avg* *度量。*

结果：

 ![image-20200704003221622](../../../Typora/Picture/image-20200704003221622.png)

图 13—1

#### 13.2.4  桶嵌套桶

将make作为桶嵌套如另外一个桶

首先为color和make设置fielddata

PUT /cars

{

 "mappings": {

  "transactions" :{

   "properties" :{

​    "color" : {

​     "type" : "text",

​     "fielddata" :true

​    },

​    "make" :{

​     "type" : "text",

​     "fielddata" :true

​    }

​    }

  }

 }

}

输入数据，省略，将桶嵌套进 另外一个桶：

GET /cars/transactions/_search

{

  "size" : 0,

  "aggs": {

   "colors": {

​     "terms": {

​      "field": "color"

​     },

​     "aggs": {

​      "avg_price": { 

​        "avg": {

​         "field": "price"

​        }

​      },

​      "make": { 

​        "terms": {

​          "field": "make" 

​        }

​      }

​     }

   }

  }

}

注：

*1.*   注意前例中的 avg_price 度量*仍然保持原位。*

*2.*   *另一个聚合* *make* *被加入到了* *color* *颜色桶中。*

*3.*   *这个聚合是* *terms* *桶，它会为每个汽车制造商生成唯一的桶。*

结果：

 ![image-20200704003243417](../../../Typora/Picture/image-20200704003243417.png)

图 13—2

设置最低和最高价格：

GET /cars/transactions/_search

{

  "size" : 0,

  "aggs": {

   "colors": {

​     "terms": {

​      "field": "color"

​     },

​     "aggs": {

​      "avg_price": { "avg": { "field": "price" }

​      },

​      "make" : {

​        "terms" : {

​          "field" : "make"

​        },

​        "aggs" : { 

​          "min_price" : { "min": { "field": "price"} }, 

​          "max_price" : { "max": { "field": "price"} } 

​        }

​      }

​     }

   }

  }

}

### 13.3   条形图

直方图使用histogram

GET /cars/transactions/_search

{

  "size" : 0,

  "aggs":{

   "price":{

​     "histogram":{ 

​      "field": "price",

​      "interval": 20000

​     },

​     "aggs":{

​      "revenue": {

​        "sum": { 

​         "field" : "price"

​        }

​       }

​     }

   }

  }

}

注：  

*1.*   *histogram* *桶要求两个参数：一个数值字段以及一个定义桶大小为**20000**间隔。*

*2.*   *sum* *度量嵌套在每个售价区间内，用来显示每个区间内的总收入。*

响应结果如下：

 ![image-20200704003253311](../../../Typora/Picture/image-20200704003253311.png)

图 13—3

各种统计的条形图：

GET /cars/transactions/_search

{

 "size" : 0,

 "aggs": {

  "makes": {

   "terms": {

​    "field": "make",

​    "size": 10

   },

   "aggs": {

​    "stats": {

​     "extended_stats": {

​      "field": "price"

​     }

​    }

   }

  }

 }

}

效果：

 ![image-20200704003259879](../../../Typora/Picture/image-20200704003259879.png)

图 13—4

### 13.4   折线图

#### 13.4.1  折线图统计时间

折线图使用date_histogram

构建一个简单的折线图回答如下问题： 每月销售多少台汽车？

GET /cars/transactions/_search

{

  "size" : 0,

  "aggs": {

   "sales": {

​     "date_histogram": {

​      "field": "sold",

​      "interval": "month", 

​      "format": "yyyy-MM-dd" 

​     }

   }

  }

}

注：

*1.*   *interval**：时间间隔要求是日历术语* *(**如每个* *bucket 1* *个月**)**。*

*2.*   *format**：我们提供日期格式以便* *buckets* *的键值便于阅读。*

运行效果：

 ![image-20200704003310870](../../../Typora/Picture/image-20200704003310870.png)

图 13—5

#### 13.4.2  返回空 Buckets

可以发现，在上面的折现图例子中，结果里面少了一个12月的数据，结果没错，date_histogram （和 histogram 一样）默认只会返回文档数目非零的 buckets。

有些时候，我们可能希望文档数目为零的 buckets也返回，可以通过设置两个额外参数来实现这种效果：

GET /cars/transactions/_search

{

  "size" : 0,

  "aggs": {

   "sales": {

​     "date_histogram": {

​      "field": "sold",

​      "interval": "month",

​      "format": "yyyy-MM-dd",

​      "min_doc_count" : 0, 

​      "extended_bounds" : { 

​        "min" : "2014-01-01",

​        "max" : "2014-12-31"

​      }

​     }

   }

  }

}

这样就可以返回文档数目为零的 buckets，就能顺利做出折线图。

 ![image-20200704003318058](../../../Typora/Picture/image-20200704003318058.png)

图 13—6

#### 13.4.3  折线图的复杂应用

作为例子，我们构建聚合以便按季度展示所有汽车品牌总销售额。同时按季度、按每个汽车品牌计算销售总额，以便可以找出哪种品牌最赚钱：

GET /cars/transactions/_search

{

  "size" : 0,

  "aggs": {

   "sales": {

​     "date_histogram": {

​      "field": "sold",

​      "interval": "quarter", 

​      "format": "yyyy-MM-dd",

​      "min_doc_count" : 0,

​      "extended_bounds" : {

​        "min" : "2014-01-01",

​        "max" : "2014-12-31"

​      }

​     },

​     "aggs": {

​      "per_make_sum": {

​        "terms": {

​         "field": "make"

​        },

​        "aggs": {

​         "sum_price": {

​           "sum": { "field": "price" } 

​         }

​        }

​      },

​      "total_sum": {

​        "sum": { "field": "price" } 

​      }

​     }

   }

  }

}

注：

*1.*   *注意我们把时间间隔从* *month* *改成了* *quarter* *。*

*2.*   *计算每种品牌的总销售金额。*

*3.*   *也计算所有全部品牌的汇总销售金额。*

运行效果：

 ![image-20200704003327944](../../../Typora/Picture/image-20200704003327944.png)

图 13—7

把结果绘成图

 ![image-20200704003332789](../../../Typora/Picture/image-20200704003332789.png)

图 13—8

#### 13.4.4  范围限定的聚合

##### 13.4.4.1  聚合与匹配查询关系

让我们看看第一个聚合的示例：

GET /cars/transactions/_search

{

  "size" : 0,

  "aggs" : {

​    "colors" : {

​      "terms" : {

​       "field" : "color"

​      }

​    }

  }

}

现实中，Elasticsearch 认为 "没有指定查询" 和 "查询所有文档" 是等价的。前面这个查询内部会转化成下面的这个请求：

GET /cars/transactions/_search

{

  "size" : 0,

  "query" : {

​    "match_all" : {}

  },

  "aggs" : {

​    "colors" : {

​      "terms" : {

​       "field" : "color"

​      }

​    }

  }

}

知道了原理，利用范围，我们可以问“福特在售车有多少种颜色？

GET /cars/transactions/_search

{

  "query" : {

​    "match" : {

​      "make" : "ford"

​    }

  },

  "aggs" : {

​    "colors" : {

​      "terms" : {

​       "field" : "color"

​      }

​    }

  }

}

##### 13.4.4.2  全局桶

例如，比方说我们想知道福特汽车与 所有 汽车平均售价的比较

GET /cars/transactions/_search

{

  "size" : 0,

  "query" : {

​    "match" : {

​      "make" : "ford"

​    }

  },

  "aggs" : {

​    "single_avg_price": {

​      "avg" : { "field" : "price" } 

​    },

​    "all": {

​      "global" : {}, 

​      "aggs" : {

​        "avg_price": {

​          "avg" : { "field" : "price" } 

​        }

 

​      }

​    }

  }

}

注：

*1.*   *聚合操作在查询范围内（例如：所有文档匹配* *ford* *）*

*2.*   *global* *全局桶没有参数。*

*3.*   *global**下的聚合操作针对所有文档，忽略汽车品牌。*

#### 13.4.5  过滤和聚合

##### 13.4.5.1  聚合的时候使用过滤

找到售价在 $10,000 美元之上的所有汽车同时也为这些车计算平均售价，用一个 constant_score 查询和 filter 约束：

GET /cars/transactions/_search

{

  "size" : 0,

  "query" : {

​    "constant_score": {

​      "filter": {

​        "range": {

​          "price": {

​            "gte": 10000

​          }

​        }

​      }

​    }

  },

  "aggs" : {

​    "single_avg_price": {

​      "avg" : { "field" : "price" }

​    }

  }

}

##### 13.4.5.2  过滤桶

GET /cars/transactions/_search

{

  "size" : 0,

  "query":{

   "match": {

​     "make": "ford"

   }

  },

  "aggs":{

   "recent_sales": {

​     "filter": { 

​      "range": {

​        "sold": {

​         "from": "now-1M"

​        }

​      }

​     },

​     "aggs": {

​      "average_price":{

​        "avg": {

​         "field": "price" 

​        }

​      }

​     }

   }

  }

}

注：

*1.*   *使用* *过滤* *桶在* *查询* *范围基础上应用过滤器。*

*2.*   *avg* *度量只会对* *ford* *和上个月售出的文档计算平均售价。*

##### 13.4.5.3  后过滤器

"只过滤搜索结果，不过滤聚合结果呢？" 使用 post_filter 。

GET /cars/transactions/_search

{

  "size" : 0,

  "query": {

​    "match": {

​      "make": "ford"

​    }

  },

  "post_filter": {  

​    "term" : {

​      "color" : "green"

​    }

  },

  "aggs" : {

​    "all_colors": {

​      "terms" : { "field" : "color" }

​    }

  }

}

小结：

\1.   在 filter 过滤中的 non-scoring 查询，同时影响搜索结果和聚合结果。

\2.   filter 桶影响聚合。

\3.   post_filter 只影响搜索结果。

#### 13.4.6  多桶排序

##### 13.4.6.1  内置排序

做一个 terms 聚合但是按 doc_count 值的升序排序：

GET /cars/transactions/_search

{

  "size" : 0,

  "aggs" : {

​    "colors" : {

​      "terms" : {

​       "field" : "color",

​       "order": {

​        "_count" : "asc" 

​       }

​      }

​    }

  }

}

注：*用关键字* *_count* *，我们可以按* *doc_count* *值的升序排序。*

为聚合引入了一个 order 对象， 它允许我们可以根据以下几个值中的一个值进行排序：

l **_count****：**按文档数排序。对 terms 、 histogram 、 date_histogram 有效。

l **_term****：**按词项的字符串值的字母顺序排序。只在 terms 内使用。

l **_key****：**按每个桶的键值数值排序（理论上与 _term 类似）。 只在 histogram 和 date_histogram 内使用。

##### 13.4.6.2  按度量排序

在我们的汽车销售分析仪表盘中，我们可能想按照汽车颜色创建一个销售条状图表，但按照汽车平均售价的升序进行排序。

我们可以增加一个度量，再指定 order 参数引用这个度量即可：

GET /cars/transactions/_search

{

  "size" : 0,

  "aggs" : {

​    "colors" : {

​      "terms" : {

​       "field" : "color",

​       "order": {

​        "avg_price" : "asc" 

​       }

​      },

​      "aggs": {

​        "avg_price": {

​          "avg": {"field": "price"} 

​        }

​      }

​    }

  }

}

注：

*1.*   计*算每个桶的平均售价。*

*2.*   *桶按照计算平均值的升序排序。*

我们可以采用这种方式用任何度量排序，只需简单的引用度量的名字。不过有些度量会输出多个值。 extended_stats 度量是一个很好的例子：它输出好几个度量值。

如果我们想使用多值度量进行排序， 我们只需以关心的度量为关键词使用点式路径：

GET /cars/transactions/_search

{

  "size" : 0,

  "aggs" : {

​    "colors" : {

​      "terms" : {

​       "field" : "color",

​       "order": {

​        "stats.variance" : "asc" 

​       }

​      },

​      "aggs": {

​        "stats": {

​          "extended_stats": {"field": "price"}

​        }

​      }

​    }

  }

}

在上面这个例子中，我们按每个桶的方差来排序，所以这种颜色售价方差最小的会排在结果集最前面。

##### 13.4.6.3  基于“深度”度量排序

在前面的示例中，度量是桶的直接子节点。在一定条件下，我们也有可能对 更深 的度量进行排序，比如孙子桶或从孙桶。

可以定义更深的路径，将度量用尖括号（ > ）嵌套起来，像这样： my_bucket>another_bucket>metric 。

需要提醒的是嵌套路径上的每个桶都必须是 单值 的。

目前，只有三个单值桶： filter 、 global 和 reverse_nested 。让我们快速用示例说明，创建一个汽车售价的直方图，但是按照红色和绿色（不包括蓝色）车各自的方差来排序：

GET /cars/transactions/_search

{

  "size" : 0,

  "aggs" : {

​    "colors" : {

​      "histogram" : {

​       "field" : "price",

​       "interval": 20000,

​       "order": {

​        "red_green_cars>stats.variance" : "asc" 

​       }

​      },

​      "aggs": {

​        "red_green_cars": {

​          "filter": { "terms": {"color": ["red", "green"]}}, 

​          "aggs": {

​            "stats": {"extended_stats": {"field" : "price"}} 

​          }

​        }

​      }

​     }

  }

}

注：

*1.*   *按照嵌套度量的方差对桶的直方图进行排序。*

*2.*   *因为我们使用单值过滤器* *filter* *，我们可以使用嵌套排序。*

*3.*   *按照生成的度量对统计结果进行排序。*

本例中，可以看到我们如何访问一个嵌套的度量。 stats 度量是 red_green_cars 聚合的子节点，而 red_green_cars 又是 colors 聚合的子节点。 为了根据这个度量排序，我们定义了路径 red_green_cars>stats.variance 。我们可以这么做，因为 filter 桶是个单值桶。

#### 13.4.7  近似算法

##### 13.4.7.1  cardinality算法（统计去重后的数量）

###### 13.4.7.1.1  cardinality算法说明与使用

相同效果的sql语句：

SELECT COUNT(DISTINCT color)

FROM cars

用 cardinality 度量确定经销商销售汽车颜色的数量：

GET /cars/transactions/_search

{

  "size" : 0,

  "aggs" : {

​    "distinct_colors" : {

​      "cardinality" : {

​       "field" : "color"

​      }

​    }

  }

}

返回结果：

 ![image-20200704003401125](../../../Typora/Picture/image-20200704003401125.png)

图 13—9

每月有多少颜色的车被售出？

GET /cars/transactions/_search

{

 "size" : 0,

 "aggs" : {

   "months" : {

​    "date_histogram": {

​     "field": "sold",

​     "interval": "month"

​    },

​    "aggs": {

​     "distinct_colors" : {

​       "cardinality" : {

​        "field" : "color"

​       }

​     }

​    }

   }

 }

}

###### 13.4.7.1.2  配置cardinality算法精度

要配置精度，我们必须指定 precision_threshold 参数的值。

GET /cars/transactions/_search

{

  "size" : 0,

  "aggs" : {

​    "distinct_colors" : {

​      "cardinality" : {

​       "field" : "color",

​       "precision_threshold" : 100 

​      }

​    }

  }

}

注：precision_threshold 接受 0–40,000 之间的数字，更大的值还是会被当作 40,000 来处理，本实例中会确保当字段唯一值在 100 以内时会得到非常准确的结果。

**速度优化**

如果速度对我们至关重要，可以做进一步的优化。 因为 HLL 只需要字段内容的哈希值，我们可以在索引时就预先计算好。 就能在查询时跳过哈希计算然后将哈希值从 fielddata 直接加载出来。

优点与缺点

预先计算哈希值只对内容很长或者基数很高的字段有用，计算这些字段的哈希值的消耗在查询时是无法忽略的。

预先计算并不能保证所有的字段都更快，它只对那些具有高基数和/或者内容很长的字符串字段有作用。需要记住的是，预计算只是简单的将查询消耗的时间提前转移到索引时，并非没有任何代价，区别在于你可以选择在 什么时候 做这件事，要么在索引时，要么在查询时。

要想使用预先索引，我们需要为数据增加一个新的多值字段。我们先删除索引，再增加一个包括哈希值字段的映射，然后重新索引：

```
DELETE /cars/
 
PUT /cars/
{
  "mappings": {
    "transactions": {
      "properties": {
        "color": {
          "type": "string",
          "fields": {
            "hash": {
              "type": "murmur3" 
            }
          }
        }
      }
    }
  }
}
 
POST /cars/transactions/_bulk
{ "index": {}}
{ "price" : 10000, "color" : "red", "make" : "honda", "sold" : "2014-10-28" }
{ "index": {}}
{ "price" : 20000, "color" : "red", "make" : "honda", "sold" : "2014-11-05" }
{ "index": {}}
{ "price" : 30000, "color" : "green", "make" : "ford", "sold" : "2014-05-18" }
{ "index": {}}
{ "price" : 15000, "color" : "blue", "make" : "toyota", "sold" : "2014-07-02" }
{ "index": {}}
{ "price" : 12000, "color" : "green", "make" : "toyota", "sold" : "2014-08-19" }
{ "index": {}}
{ "price" : 20000, "color" : "red", "make" : "honda", "sold" : "2014-11-05" }
{ "index": {}}
{ "price" : 80000, "color" : "red", "make" : "bmw", "sold" : "2014-01-01" }
{ "index": {}}
{ "price" : 25000, "color" : "blue", "make" : "ford", "sold" : "2014-02-12" }
```

注：*多值字段的类型是* *murmur3* *，这是一个哈希函数。*

现在当我们执行聚合时，我们使用 color.hash 字段而不是 color 字段：

```
GET /cars/transactions/_search
{
    "size" : 0,
    "aggs" : {
        "distinct_colors" : {
            "cardinality" : {
              "field" : "color.hash" 
            }
        }
    }
}
```

现在 cardinality 度量会读取 "color.hash" 里的值（预先计算的哈希值），取代动态计算原始值的哈希。

##### 13.4.7.2  percentiles算法（待）（百分位计算）

###### 13.4.7.2.1  percentiles算法概述

百分位数展现某以具体百分比下观察到的数值。例如，第95个百分位上的数值，是高于 95% 的数据总和。

如果我们依靠如平均值或中位数这样的简单度量，就会得到像这样一幅图

 ![image-20200704003416855](../../../Typora/Picture/image-20200704003416855.png)

图 13—10

一切正常。 图上有轻微的波动，但没有什么值得关注的。 但如果我们加载 99 百分位数时（这个值代表最慢的 1% 的延时），我们看到了完全不同的一幅画面

 ![image-20200704003422460](../../../Typora/Picture/image-20200704003422460.png)

图 13—11

令人吃惊！在上午九点半时，均值只有 75ms。如果作为一个系统管理员，我们都不会看他第二眼。 一切正常！但 99 百分位告诉我们有 1% 的用户碰到的延时超过 850ms，这是另外一幅场景。 在上午4点48时也有一个小波动，这甚至无法从平均值和中位数曲线上观察到。

###### 13.4.7.2.2  percentiles算法使用

现有一个需求：比如有一个网站，记录下了每次请求的访问的耗时，需要统计tp50，tp90，tp99：

l tp50：50%的请求的耗时最长在多长时间

l tp90：90%的请求的耗时最长在多长时间

l tp99：99%的请求的耗时最长在多长时间

PUT /website

{

"mappings": {

"logs":{

"properties": {

"latency":{"type": "long"},

"province":{"type": "keyword"},

"timestamp":{"type":"date"}

}

}

}

}



插入数据

POST /website/logs/_bulk

{ "index": {}}

{ "latency" : 105, "province" : "江苏", "timestamp" : "2016-10-28" }

{ "index": {}}

{ "latency" : 83, "province" : "江苏", "timestamp" : "2016-10-29" }

{ "index": {}}

{ "latency" : 92, "province" : "江苏", "timestamp" : "2016-10-29" }

{ "index": {}}

{ "latency" : 112, "province" : "江苏", "timestamp" : "2016-10-28" }

{ "index": {}}

{ "latency" : 68, "province" : "江苏", "timestamp" : "2016-10-28" }

{ "index": {}}

{ "latency" : 76, "province" : "江苏", "timestamp" : "2016-10-29" }

{ "index": {}}

{ "latency" : 101, "province" : "新疆", "timestamp" : "2016-10-28" }

{ "index": {}}

{ "latency" : 275, "province" : "新疆", "timestamp" : "2016-10-29" }

{ "index": {}}

{ "latency" : 166, "province" : "新疆", "timestamp" : "2016-10-29" }

{ "index": {}}

{ "latency" : 654, "province" : "新疆", "timestamp" : "2016-10-28" }

{ "index": {}}

{ "latency" : 389, "province" : "新疆", "timestamp" : "2016-10-28" }

{ "index": {}}

{ "latency" : 302, "province" : "新疆", "timestamp" : "2016-10-29" }

查找tp50、tp90、tp99

GET /website/logs/_search

{

"size": 0,

"aggs": {

"latency_percentiles": {"percentiles": {"field": "latency","percents": [50,90,99]}},

"latency_late":{"avg": {"field": "latency"}}

}

}

结果如下：

 ![image-20200704003436374](../../../Typora/Picture/image-20200704003436374.png)

图 13—12

确定是哪些省份比较慢

GET /website/logs/_search

{

"size": 0,

"aggs": {"group_by_province":{

"terms": {"field": "province"},

"aggs": {

"latency_percentiles": {"percentiles": {"field": "latency","percents": [50,90,99]}},

"latency_late":{"avg": {"field": "latency"}}

}

}

}

}

查询结果;

 ![image-20200704003443762](../../../Typora/Picture/image-20200704003443762.png)

图 13—13

可以看出新僵的网比较慢。

#### 13.4.8  significant_terms聚合

significant_terms 聚合可以通过分析你的数据集，找到一些 异常 的指标。比如多个人反应被盗用信息，这些人又和某某平台产生交易信息，该聚合就能通过这样找到该平台。

#### 13.4.9  Doc Values and Fielddata

##### 13.4.9.1  深入理解 Doc Values

###### 13.4.9.1.1  Doc Values 压缩数字

Doc Values 在压缩数字过程中，它会按依次检测以下压缩模式:

如果所有的数值各不相同（或缺失），设置一个标记并记录这些值

如果这些值小于 256，将使用一个简单的编码表

如果这些值大于 256，检测是否存在一个最大公约数

如果没有存在最大公约数，从最小的数值开始，统一计算偏移量进行编码

###### 13.4.9.1.2  Doc Values 压缩字符

Doc Values 在压缩字符过程中，通过借助顺序表（ordinal table），String 类型也是类似进行编码的。String 类型是去重之后存放到顺序表的，通过分配一个 ID，然后通过数字类型的 ID 构建 Doc Values。这样 String 类型和数值类型可以达到同样的压缩效果。

顺序表本身也有很多压缩技巧，比如固定长度、变长或是前缀字符编码等等。

###### 13.4.9.1.3  禁用 Doc Values

Doc Values 默认对所有字段启用，除了 analyzed strings。也就是说所有的数字、地理坐标、日期、IP 和不分析（ not_analyzed ）字符类型都会默认开启。

要禁用 Doc Values ，在字段的映射（mapping）设置 doc_values: false 即可。例如，这里我们创建了一个新的索引，字段 "session_id" 禁用了 Doc Values：

```
PUT my_index
{
  "mappings": {
    "my_type": {
      "properties": {
        "session_id": {
          "type":       "string",
          "index":      "not_analyzed",
          "doc_values": false 
        }
      }
    }
  }
}
```

##### 13.4.9.2  Fielddata 的过滤

```
PUT /music/_mapping/song
{
  "properties": {
    "tag": {
      "type": "string",
      "fielddata": { 
        "filter": {
          "frequency": { 
            "min":              0.01, 
            "min_segment_size": 500  
          }
        }
      }
    }
  }
}
```

注：

*1.*   *fielddata* *关键字允许我们配置* *fielddata* *处理该字段的方式。*

*2.*   *frequency* *过滤器允许我们基于项频率过滤加载* *fielddata**。*

*3.*   *只加载那些至少在本段文档中出现* *1%* *的项。*

*4.*   *忽略任何文档个数小于* *500* *的段。*

有了这个映射，只有那些至少在 本段 文档中出现超过 1% 的项才会被加载到内存中。

min_segment_size 参数要求 Elasticsearch 忽略某个大小以下的段。 如果一个段内只有少量文档，它的词频会非常粗略没有任何意义。 小的分段会很快被合并到更大的分段中，某一刻超过这个限制，将会被纳入计算。

##### 13.4.9.3  预加载 fielddata

Elasticsearch 加载内存 fielddata 的默认行为是 延迟 加载 。 当 Elasticsearch 第一次查询某个字段时，它将会完整加载这个字段所有 Segment 中的倒排索引到内存中，以便于以后的查询能够获取更好的性能。

预加载是按字段启用的，所以我们可以控制具体哪个字段可以预先加载：

```
PUT /music/_mapping/_song
{
  "tags": {
    "type": "string",
    "fielddata": {
      "loading" : "eager" 
    }
  }
}
```

注：*设置* *fielddata.loading: eager* *可以告诉* *Elasticsearch* *预先将此字段的内容载入内存中。*

##### 13.4.9.4  全局序号（待）（Global Ordinals）

有种可以用来降低字符串 fielddata 内存使用的技术叫做 序号 。

设想我们有十亿文档，每个文档都有自己的 status 状态字段，状态总共有三种： status_pending 、 status_published 、 status_deleted 。如果我们为每个文档都保留其状态的完整字符串形式，那么每个文档就需要使用 14 到 16 字节，或总共 15 GB。

取而代之的是我们可以指定三个不同的字符串，对其排序、编号：0，1，2。

**预构建全局序号（****Eager global ordinals****）**

单个字符串字段 可以通过配置预先构建全局序号：

```
PUT /music/_mapping/_song
{
  "song_title": {
    "type": "string",
    "fielddata": {
      "loading" : "eager_global_ordinals" 
    }
  }
}
```

也可以对 Doc values 进行全局序号预构建：

```
PUT /music/_mapping/_song
{
  "song_title": {
    "type":       "string",
    "doc_values": true,
    "fielddata": {
      "loading" : "eager_global_ordinals" 
    }
  }
}
```

**索引预热器（****Index Warmers****）**

一个索引预热器允许我们指定一个查询和聚合须要在新分片对于搜索可见之前执行。

##### 13.4.9.5  优化聚合查询

###### 13.4.9.5.1  深度优先

举例：

```
{
  "actors" : [
    "Fred Jones",
    "Mary Jane",
    "Elizabeth Worthing"
  ]
}
```

我们想要查询出演影片最多的十个演员以及与他们合作最多的演员：

```
{
  "aggs" : {
    "actors" : {
      "terms" : {
         "field" : "actors",
         "size" :  10
      },
      "aggs" : {
        "costars" : {
          "terms" : {
            "field" : "actors",
            "size" :  5
          }
        }
      }
    }
  }
}
```

这会返回前十位出演最多的演员，以及与他们合作最多的五位演员，最终返回 50 条数据！

这个看上去简单的查询可以轻而易举地消耗大量内存，我们可以通过在内存中构建一个树来查看这个 terms 聚合。 actors 聚合会构建树的第一层，每个演员都有一个桶。然后，内套在第一层的每个节点之下， costar 聚合会构建第二层，每个联合出演一个桶。

假设一部影片有10名演员，每部影片会生成102 == 100个桶，有2亿部影片，就会产生2亿的平方个桶，只为了得到想要找的50个演员，严重影响性能，还可能会出现内存不足的问题，到最后查询能不能维持都是个问题。

###### 13.4.9.5.2  广度优先

为了应对深度优先的这种情况，我们应该使用另一种集合策略叫做 广度优先 。这种策略是先找出出演影片最多的10个人，不急着做桶，将除这10个人之外的演员全部去掉，再用这10个演员做桶，寻找与他们合作最多的演员，就能够大大提高搜索性能。

```
{
  "aggs" : {
    "actors" : {
      "terms" : {
         "field" :        "actors",
         "size" :         10,
         "collect_mode" : "breadth_first" 
      },
      "aggs" : {
        "costars" : {
          "terms" : {
            "field" : "actors",
            "size" :  5
          }
        }
      }
    }
  }
}
```

注：*breadth_first**去掉这**10**个演员之外的人。*

广度优先仅仅适用于每个组的聚合数量远远小于当前总组数的情况下，因为广度优先会在内存中缓存裁剪后的仅仅需要缓存的每个组的所有数据，以便于它的子聚合分组查询可以复用上级聚合的数据。

## 14   地理位置

### 14.1   地理坐标点

#### 14.1.1  定义坐标点字段

地理坐标点 是指地球表面可以用经纬度描述的一个点。 地理坐标点可以用来计算两个坐标间的距离，还可以判断一个坐标是否在一个区域中，或在聚合中。

地理坐标点不能被动态映射（dynamic mapping）自动检测，而是需要显式声明对应字段类型为 geo-point ：

```
PUT /attractions
{
  "mappings": {
    "restaurant": {
      "properties": {
        "name": {
          "type": "string"
        },
        "location": {
          "type": "geo_point"
        }
      }
    }
  }
}
```

#### 14.1.2  经纬度坐标格式

```
PUT /attractions/restaurant/1
{
  "name":     "Chipotle Mexican Grill",
  "location": "40.715, -74.011" 
}
 
PUT /attractions/restaurant/2
{
  "name":     "Pala Pizza",
  "location": { 
    "lat":     40.722,
    "lon":    -73.989
  }
}
 
PUT /attractions/restaurant/3
{
  "name":     "Mini Munchies Pizza",
  "location": [ -73.983, 40.719 ] 
}
```

注：

*1.*   *字符串形式以半角逗号分割，如* *"lat,lon"* *。*

*2.*   *对象形式显式命名为* *lat* *和* *lon* *。*

*3.*   *数组形式表示为* *[lon,lat]* *。*

*4.*   *地理坐标点用字符串形式表示时是纬度在前，经度在后（* *"latitude,longitude"* *），而数组形式表示时是经度在前，纬度在后（* *[longitude,latitude]* *）—顺序刚好相反。*

#### 14.1.3  通过地理坐标点过滤

有四种地理坐标点相关的过滤器可以用来选中或者排除文档：

l geo_bounding_box：找出落在指定矩形框中的点。

l geo_distance：找出与指定位置在给定距离内的点。

l geo_distance_range：找出与指定点距离在给定最小距离和最大距离之间的点。

l geo_polygon：找出落在多边形中的点。 这个过滤器使用代价很大 。

注：*地理坐标过滤器使用代价昂贵* *—* *所以最好在文档集合尽可能少的场景下使用。你可以先使用那些简单快捷的过滤器，比如* *term* *或* *range* *，来过滤掉尽可能多的文档，最后才交给地理坐标过滤器处理。*

#### 14.1.4  地理坐标盒模型过滤器

指定一个矩形的 顶部 , 底部 , 左边界 ，和 右边界 ，然后过滤器只需判断坐标的经度是否在左右边界之间，纬度是否在上下边界之间：

```
GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geo_bounding_box": {
          "location": { 
            "top_left": {
              "lat":  40.8,
              "lon": -74.0
            },
            "bottom_right": {
              "lat":  40.7,
              "lon": -73.0
            }
          }
        }
      }
    }
  }
}
```

注：*这些坐标也可以用* *bottom_left* *和* *top_right* *来表示。*

**优化盒模型**

地理坐标盒模型过滤器 不需要把所有坐标点都加载到内存里。 因为它要做的 只是简单判断 lat 和 lon 坐标数值是否在给定的范围内，可以用倒排索引做一个 range 过滤来实现目标。

要使用这种优化方式，需要把 geo_point 字段 用 lat 和 lon 的方式分别映射到索引中：

```
PUT /attractions
{
  "mappings": {
    "restaurant": {
      "properties": {
        "name": {
          "type": "string"
        },
        "location": {
          "type":    "geo_point",
          "lat_lon": true 
        }
      }
    }
  }
}
```

注：*location.lat* *和* *location.lon* *字段将被分别索引。它们可以被用于检索，但是不会在检索结果中返回。*

然后，查询时你需要告诉 Elasticesearch 使用已索引的 lat 和 lon ：

```
GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geo_bounding_box": {
          "type":    "indexed", 
          "location": {
            "top_left": {
              "lat":  40.8,
              "lon": -74.0
            },
            "bottom_right": {
              "lat":  40.7,
              "lon":  -73.0
            }
          }
        }
      }
    }
  }
}
```

注：*设置* *type* *参数为* *indexed* *（替代默认值* *memory* *）来明确告诉* *Elasticsearch* *对这个过滤器使用倒排索引。*

#### 14.1.5  地理距离过滤器

##### 14.1.5.1  使用举例计算器

地理距离过滤器（ geo_distance ）以给定位置为圆心画一个圆，来找出那些地理坐标落在其中的文档：

```
GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geo_distance": {
          "distance": "1km", 
          "location": { 
            "lat":  40.715,
            "lon": -73.988
          }
        }
      }
    }
  }
}
```

注：

*1.*   *找出所有与指定点距离在* *1km* *内的* *location* *字段。*

*2.*   *中心点可以表示为字符串，数组或者（如示例中的）对象。*

##### 14.1.5.2  更快的地理距离计算

两点间的距离计算，有多种牺牲性能换取精度的算法：

l arc：最慢但最精确的是 arc 计算方式，这种方式把世界当作球体来处理。不过这种方式的精度有限，因为这个世界并不是完全的球体。

l plane：plane 计算方式把地球当成是平坦的，这种方式快一些但是精度略逊。在赤道附近的位置精度最好，而靠近两极则变差。

l sloppy_arc：如此命名，是因为它使用了 Lucene 的 SloppyMath 类。这是一种用精度换取速度的计算方式， 它使用 Haversine formula 来计算距离。它比 arc 计算方式快 4 到 5 倍，并且距离精度达 99.9%。这也是默认的计算方式。

你可以参考下例来指定不同的计算方式：

```
GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geo_distance": {
          "distance":      "1km",
          "distance_type": "plane", 
          "location": {
            "lat":  40.715,
            "lon": -73.988
          }
        }
      }
    }
  }
}
```

##### 14.1.5.3  地理距离区间过滤器

geo_distance 和 geo_distance_range 过滤器的唯一差别在于后者是一个环状的，它会排除掉落在内圈中的那部分文档。

指定到中心点的距离也可以换一种表示方式：指定一个最小距离（使用 gt 或者 gte ）和最大距离（使用 lt 和 lte ），就像使用 range 过滤器一样：

```
GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geo_distance_range": {
          "gte":    "1km", 
          "lt":     "2km", 
          "location": {
            "lat":  40.715,
            "lon": -73.988
          }
        }
      }
    }
  }
}
```

注：*匹配那些距离中心点大于等于* *1km* *而小于* *2km* *的位置。*

#### 14.1.6  按距离排序、按距离打分

##### 14.1.6.1  按距离排序

检索结果可以按与指定点的距离排序：

```
GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geo_bounding_box": {
          "type":       "indexed",
          "location": {
            "top_left": {
              "lat":  40.8,
              "lon": -74.0
            },
            "bottom_right": {
              "lat":  40.4,
              "lon": -73.0
            }
          }
        }
      }
    }
  },
  "sort": [
    {
      "_geo_distance": {
        "location": { 
          "lat":  40.715,
          "lon": -73.998
        },
        "order":         "asc",
        "unit":          "km", 
        "distance_type": "plane" 
      }
    }
  ]
}
```

注：

*1.*   *计算每个文档中* *location* *字段与指定的* *lat/lon* *点间的距离。*

*2.*   *将距离以* *km* *为单位写入到每个返回结果的* *sort* *键中。*

*3.*   *使用快速但精度略差的* *plane* *计算方式。*

至于为什么要设置单位（km），这个用于排序的值会设置在每个返回结果的 sort 元素中。

 ![image-20200704003522112](../../../Typora/Picture/image-20200704003522112.png)

图 14—1

注：*餐厅到我们指定的位置距离是* *0.084km**。*

##### 14.1.6.2  按距离打分

有可能距离是决定返回结果排序的唯一重要因素，不过更常见的情况是距离会和其它因素，比如全文检索匹配度、流行程度或者价格一起决定排序结果。

遇到这种场景你需要在 功能评分查询 中指定方式让我们把这些因子处理后得到一个综合分。 越近越好 中有个一个例子就是介绍地理距离影响排序得分的。

另外按距离排序还有个缺点就是性能：需要对每一个匹配到的文档都进行距离计算。而 function_score 查询，在 rescore 语句 中可以限制只对前 n 个结果进行计算。

### 14.2   Geohashes

#### 14.2.1  Geohashes概述

Geohashes 是一种将经纬度坐标（ lat/lon ）编码成字符串的方式。

Geohashes 把整个世界分为 32 个单元的格子 —— 4 行 8 列 —— 每一个格子都用一个字母或者数字标识。比如 g 这个单元覆盖了半个格林兰，冰岛的全部和大不列颠的大部分。每一个单元还可以进一步被分解成新的 32 个单元，这些单元又可以继续被分解成 32 个更小的单元，不断重复下去。 gc 这个单元覆盖了爱尔兰和英格兰， gcp 覆盖了伦敦的大部分和部分南英格兰， gcpuuz94k 是白金汉宫的入口，精确到约 5 米。

换句话说， geohash 的长度越长，它的精度就越高。如果两个 geohashes 有一个共同的前缀— gcpuuz—就表示他们挨得很近。共同的前缀越长，距离就越近。

各个 geohash 单元的近似尺寸（代表精度）：

 ![image-20200704003531000](../../../Typora/Picture/image-20200704003531000.png)

图 14—2

#### 14.2.2  Geohashes 映射

```
PUT /attractions
{
  "mappings": {
    "restaurant": {
      "properties": {
        "name": {
          "type": "string"
        },
        "location": {
          "type":               "geo_point",
          "geohash_prefix":     true, 
          "geohash_precision":  "1km" 
        }
      }
    }
  }
}
```

注：

*1.*   *将* *geohash_prefix* *设为* *true* *来告诉* *Elasticsearch* *使用指定精度来索引* *geohash* *的前缀。*

*2.*   *精度可以是一个具体的数字，代表的* *geohash* *的长度，也可以是一个距离。* *1km* *的精度对应的* *geohash* *的长度是* *7* *。*

通过如上设置， geohash 前缀中 1 到 7 的部分将被索引，所能提供的精度大约在 150 米。

#### 14.2.3  Geohash 单元查询

geohash_cell 查询把经纬度坐标位置根据指定精度转换成一个 geohash ，然后查找所有包含这个 geohash 的位置

```
GET /attractions/restaurant/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "geohash_cell": {
          "location": {
            "lat":  40.718,
            "lon": -73.983
          },
          "precision": "2km" 
        }
      }
    }
  }
}
```

注：*precision* *字段设置的精度不能高于映射时* *geohash_precision* *字段指定的值。*

此查询将 lat/lon 坐标点转换成对应长度的 geohash。

然而，如上例中的写法可能不会返回 2km 内所有的餐馆。要知道 geohash 实际上仅是个矩形，而指定的点可能位于这个矩形中的任何位置。有可能这个点刚好落在了 geohash 单元的边缘附近，但过滤器会排除那些落在相邻单元的餐馆。

为了修复这个问题，我们可以通过设置 neighbors参数为 true ，让查询把周围的单元也包含进来：

```
GET /attractions/restaurant/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "geohash_cell": {
          "location": {
            "lat":  40.718,
            "lon": -73.983
          },
          "neighbors": true, 
          "precision": "2km"
        }
      }
    }
  }
}
```

注：

\1.   *此查询将会寻找对应的* *geohash* *和包围它的* *geohashes* *。*

*2.*   *2km* *的* *precision* *会被转换成长度为* *6* *的* *geohash* *。*

### 14.3   地理位置聚合

#### 14.3.1  地理距离聚合

geo_distance 聚合 对一些搜索非常有用，例如找到所有距离我 1km 以内的披萨店。搜索结果应该也的确被限制在用户指定 1km 范围内，但是我们可以添加在 2km 范围内找到的其他结果：

```
GET /attractions/restaurant/_search
{
  "query": {
    "bool": {
      "must": {
        "match": { 
          "name": "pizza"
        }
      },
      "filter": {
        "geo_bounding_box": {
          "location": { 
            "top_left": {
              "lat":  40.8,
              "lon": -74.1
            },
            "bottom_right": {
              "lat":  40.4,
              "lon": -73.7
            }
          }
        }
      }
    }
  },
  "aggs": {
    "per_ring": {
      "geo_distance": { 
        "field":    "location",
        "unit":     "km",
        "origin": {
          "lat":    40.712,
          "lon":   -73.988
        },
        "ranges": [
          { "from": 0, "to": 1 },
          { "from": 1, "to": 2 }
        ]
      }
    }
  },
  "post_filter": { 
    "geo_distance": {
      "distance":   "1km",
      "location": {
        "lat":      40.712,
        "lon":     -73.988
      }
    }
  }
}
```

注：

*1.*   *主查询查找名称中含有* *pizza* *的饭店。*

*2.*   *geo_bounding_box* *筛选那些只在纽约区域的结果。*

*3.*   *geo_distance* *聚合统计距离用户* *1km* *以内，**1km* *到* *2km* *的结果的数量。*

*4.*   *最后，**post_filter* *将结果缩小至那些在用户* *1km* *范围内的饭店。*

#### 14.3.2  Geohash 网格聚合

geohash_grid 按照你定义的精度计算每一个点的 geohash 值而将附近的位置聚合在一起，结果是一个网格。

聚合是稀疏的—它 仅返回那些含有文档的单元。 如果 geohashes 太精确，将产生太多的 buckets，它将默认返回那些包含了大量文档、最密集的10000个单元。 然而，为了计算哪些是最密集的 Top10000 ，它还是需要产生 所有 的 buckets 。可以通过以下方式来控制 buckets 的产生数量：

使用 geo_bounding_box 来限制结果。

为你的边界大小选择一个适当的 precision (精度)

```
GET /attractions/restaurant/_search
{
  "size" : 0,
  "query": {
    "constant_score": {
      "filter": {
        "geo_bounding_box": {
          "location": { 
            "top_left": {
              "lat":  40.8,
              "lon": -74.1
            },
            "bottom_right": {
              "lat":  40.4,
              "lon": -73.7
            }
          }
        }
      }
    }
  },
  "aggs": {
    "new_york": {
      "geohash_grid": { 
        "field":     "location",
        "precision": 5
      }
    }
  }
}
```

注：

*1.*    ![image-20200704003701865](../../../Typora/Picture/image-20200704003701865.png) *边界框将搜索限制在大纽约区的范围*

*2.*     *![image-20200704003710782](../../../Typora/Picture/image-20200704003710782.png)Geohashes* *精度为* *5* *大约是* *5km x 5km**。*

#### 14.3.3  地理边界聚合

geo_bounds用于封装所有地理位置点需要的最小边界框：

```
GET /attractions/restaurant/_search
{
  "size" : 0,
  "query": {
    "constant_score": {
      "filter": {
        "geo_bounding_box": {
          "location": {
            "top_left": {
              "lat":  40,8,
              "lon": -74.1
            },
            "bottom_right": {
              "lat":  40.4,
              "lon": -73.9
            }
          }
        }
      }
    }
  },
  "aggs": {
    "new_york": {
      "geohash_grid": {
        "field":     "location",
        "precision": 5
      }
    },
    "map_zoom": { 
      "geo_bounds": {
        "field":     "location"
      }
    }
  }
}
```

注：*geo_bounds* *聚合将计算封装所有匹配查询文档所需要的最小边界框。*

响应现在包括了一个可以用来缩放地图的边界框。

 ![image-20200704003622002](../../../Typora/Picture/image-20200704003622002.png)

图 14—3

### 14.4   地理形状

#### 14.4.1  映射地理形状（geo-shapes）

##### 14.4.1.1  映射地理形状

geo-shapes 有以下作用：判断查询的形状与索引的形状的关系；这些 关系 可能是以下之一：

l intersects：查询的形状与索引的形状有重叠（默认）。

l disjoint：查询的形状与索引的形状完全 不 重叠。

l within：索引的形状完全被包含在查询的形状中。

与 geo_point 类型的字段相似， 地理形状也必须在使用前明确映射：

PUT /attractions

{

 "mappings": {

  "landmark": {

   "properties": {

​    "name": {

​     "type": "string"

​    },

​    "location": {

​     "type": "geo_shape"

​    }

   }

  }

 }

}

同时需要考虑修改两个非常重要的设置： 精度 和 距离误差 。

##### 14.4.1.2  精度

精度 （ precision ）参数 用来控制生成的 geohash 的最大长度。默认精度为 9 ，等同于尺寸在 5m x 5m 的geohash 。这个精度可能比你需要的精确得多。

精度越低，需要索引的单元就越少，检索时也会更快。当然，精度越低，地理形状的准确性就越差。

可以使用距离来指定精度 —— 如 50m 或 2km—不过这些距离最终也会转换成对应的Geohashes等级。

##### 14.4.1.3  距离误差

距离误差 指定地理形状可以接受的最大错误率。它的默认值是 0.025 ， 即 2.5% 。

#### 14.4.2  索引地理形状

地理形状通过 GeoJSON 来表示，这是一种开放的使用 JSON 实现的二维形状编码方式。 每个形状都包含了形状类型— point, line, polygon, envelope —和一个或多个经纬度点集合的数组。

注：*在* *GeoJSON* *里，经纬度表示方式通常是* *纬度* *在前，* *经度* *在后。*

如，我们用一个多边形来索引阿姆斯特丹达姆广场：

PUT /attractions/landmark/dam_square

{

  "name" : "Dam Square, Amsterdam",

  "location" : {

​    "type" : "polygon", 

​    "coordinates" : [[ 

​     [ 4.89218, 52.37356 ],

​     [ 4.89205, 52.37276 ],

​     [ 4.89301, 52.37274 ],

​     [ 4.89392, 52.37250 ],

​     [ 4.89431, 52.37287 ],

​     [ 4.89331, 52.37346 ],

​     [ 4.89305, 52.37326 ],

​     [ 4.89218, 52.37356 ]

​    ]]

  }

}

注：

*1.*    *type* *参数指明了经纬度坐标集表示的形状类型。*

*2.*    *lon/lat* *列表描述了多边形的形状。*

#### 14.4.3  查询地理形状

举个例子，当我们的用户刚刚迈出阿姆斯特丹中央火车站时，我们可以用如下方式，查询出方圆 1km 内的所有地标：

GET /attractions/landmark/_search

{

 "query": {

  "geo_shape": {

   "location": { 

​    "shape": { 

​     "type":  "circle", 

​     "radius": "1km",

​     "coordinates": [ 

​      4.89994,

​      52.37815

​     ]

​    }

   }

  }

 }

}

注：

*1.*    *查询使用* *location* *字段中的地理形状。*

*2.*    *查询中的形状是由* *shape* *键对应的内容表示。*

*3.*    *形状是一个半径为* *1km* *的圆形。*

*4.*    *Coordinates**代表安姆斯特丹中央火车站入口的坐标点。*

默认的，查询（或者过滤器 —— 工作方式相同）会从已索引的形状中寻找与指定形状有交集的部分。此外，可以把 relation 字段设置为 disjoint 来查找与指定形状不相交的部分，或者设置为 within 来查找完全落在查询形状中的。

举个例子，我们可以查找阿姆斯特丹中心区域所有的地标：

GET /attractions/landmark/_search

{

 "query": {

  "geo_shape": {

   "location": {

​    "relation": "within", 

​    "shape": {

​     "type": "polygon",

​     "coordinates": [[ 

​       [4.88330,52.38617],

​       [4.87463,52.37254],

​       [4.87875,52.36369],

​       [4.88939,52.35850],

​       [4.89840,52.35755],

​       [4.91909,52.36217],

​       [4.92656,52.36594],

​       [4.93368,52.36615],

​       [4.93342,52.37275],

​       [4.92690,52.37632],

​       [4.88330,52.38617]

​      ]]

​    }

   }

  }

 }

}

注：

*1.*    *只匹配完全落在查询形状中的已索引的形状。*

*2.*    *这个多边形表示安姆斯特丹中心区域。*

#### 14.4.4  在查询中使用已索引的形状

对于那些经常会在查询中使用的形状，可以把它们索引起来以便在查询中可以方便地直接引用名字。以之前的阿姆斯特丹中部为例，我们可以把它存储成一个类型为 neighborhood 的文档。

首先，我们仿照之前设置 landmark 时的方式建立映射：

PUT /attractions

{

 "mappings": {

  "neighborhood": {

   "properties": {

​    "name": {

​     "type": "text"

​    },

​    "location": {

​     "type": "geo_shape"

​    }

   }

  }

 }

}

然后我们索引阿姆斯特丹中部对应的形状：

PUT /attractions/neighborhood/central_amsterdam

{

 "name" : "Central Amsterdam",

 "location" : {

   "type" : "polygon",

   "coordinates" : [[

​    [4.88330,52.38617],

​    [4.87463,52.37254],

​    [4.87875,52.36369],

​    [4.88939,52.35850],

​    [4.89840,52.35755],

​    [4.91909,52.36217],

​    [4.92656,52.36594],

​    [4.93368,52.36615],

​    [4.93342,52.37275],

​    [4.92690,52.37632],

​    [4.88330,52.38617]

   ]]

 }

}

形状索引好之后，我们就可以在查询中通过 index ， type 和 id 来引用它了：

GET /attractions/landmark/_search

{

 "query": {

  "geo_shape": {

   "location": {

​    "relation": "within",

​    "indexed_shape": { 

​     "index": "attractions",

​     "type": "neighborhood",

​     "id":  "central_amsterdam",

​     "path": "location"

​    }

   }

  }

 }

}

注：*指定* *indexed_shape* *而不是* *shape* *，**Elasticesearch* *就知道需要从指定的文档和* *path* *检索出对应的形状了。*

## 15   数据建模

### 15.1   关联关系处理（不可用）

#### 15.1.1  应用层联接

例如，比方说我们正在对用户和他们的博客文章进行索引。在关系世界中，我们会这样来操作：

PUT /my_index/user/1 

{

 "name":   "John Smith",

 "email":  "john@smith.com",

 "dob":   "1970/10/24"

}

 

PUT /my_index/blogpost/2 

{

 "title":  "Relationships",

 "body":   "It's complicated...",

 "user":   1 

}

注：

*1.*    *每个文档的* *index, type,* *和* *id* *一起构造成主键。*

*2.*    *blogpost* *通过用户的* *id* *链接到用户。**index* *和* *type* *并不需要因为在我们的应用程序中已经硬编码。*

通过用户的 ID 1 可以很容易的找到博客帖子。

GET /my_index/blogpost/_search

{

 "query": {

  "filtered": {

   "filter": {

​    "term": { "user": 1 }

   }

  }

 }

}

#### 15.1.2  非规范化你的数据（关系嵌套）

如果我们希望能够通过某个用户姓名找到他写的博客文章，可以在博客文档中包含这个用户的姓名：

```
PUT /my_index/user/1
{
  "name":     "John Smith",
  "email":    "john@smith.com",
  "dob":      "1970/10/24"
}
 
PUT /my_index/blogpost/2
{
  "title":    "Relationships",
  "body":     "It's complicated...",
  "user":     {
    "id":       1,
    "name":     "John Smith" 
  }
}
```

注：这部分用户的字段数据已被冗余到 blogpost 文档中。

现在，我们通过单次查询就能够通过 relationships 找到用户 John 的博客文章。

```
GET /my_index/blogpost/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title":     "relationships" }},
        { "match": { "user.name": "John"          }}
      ]
    }
  }
}
```

数据非规范化的优点是速度快。因为每个文档都包含了所需的所有信息，当这些信息需要在查询进行匹配时，并不需要进行昂贵的联接操作。

#### 15.1.3  字段折叠

一个普遍的需求是需要通过特定字段进行分组。例如我们需要按照用户名称 分组 返回最相关的博客文章。 按照用户名分组意味着进行 terms 聚合。 为能够按照用户 整体 名称进行分组，名称字段应保持 not_analyzed 的形式。

```
PUT /my_index/_mapping/blogpost
{
  "properties": {
    "user": {
      "properties": {
        "name": { 
          "type": "string",
          "fields": {
            "raw": { 
              "type":  "string",
              "index": "not_analyzed"
            }
          }
        }
      }
    }
  }
}
```

注：

*1.*    *user.name* *字段将用来进行全文检索。*

*2.*    *user.name.raw* *字段将用来通过* *terms* *聚合进行分组。*

添加一些数据:

```
PUT /my_index/user/1
{
  "name": "John Smith",
  "email": "john@smith.com",
  "dob": "1970/10/24"
}
 
PUT /my_index/blogpost/2
{
  "title": "Relationships",
  "body": "It's complicated...",
  "user": {
    "id": 1,
    "name": "John Smith"
  }
}
 
PUT /my_index/user/3
{
  "name": "Alice John",
  "email": "alice@john.com",
  "dob": "1979/01/04"
}
 
PUT /my_index/blogpost/4
{
  "title": "Relationships are cool",
  "body": "It's not complicated at all...",
  "user": {
    "id": 3,
    "name": "Alice John"
  }
}
```

查询标题包含 relationships 并且作者名包含 John 的博客，查询结果再按作者名分组：

 ![image-20200704003749822](../../../Typora/Picture/image-20200704003749822.png)

图 15—1

注：

*1.*    *我们感兴趣的博客文章是通过* *blogposts* *聚合返回的，所以我们可以通过将* *size* *设置成* *0* *来禁止* *hits* *常规搜索。*

*2.*    *query* *返回通过* *relationships* *查找名称为* *John* *的用户的博客文章。*

*3.*    *terms* *聚合为每一个* *user.name.raw* *创建一个桶。*

*4.*    *top_score* *聚合对通过* *users* *聚合得到的每一个桶按照文档评分对词项进行排序。*

*5.*    *top_hits* *聚合仅为每个用户返回五个最相关的博客文章的* *title* *字段。*

响应结果：

 ![image-20200704003801521](../../../Typora/Picture/image-20200704003801521.png)

图 15—2

注：

*1.*    *因为我们设置* *size* *为* *0* *，所以* *hits* *数组是空的。*

*2.*    *在顶层查询结果中出现的每一个用户都会有一个对应的桶。*

*3.*    *在每个用户桶下面都会有一个* *blogposts.hits* *数组包含针对这个用户的顶层查询结果。*

*4.*    *用户桶按照每个用户最相关的博客文章进行排序。*

#### 15.1.4  解决并发问题

当我们允许多个人 同时 重命名文件或目录时，问题就来了。 设想一下，你正在对一个包含了成百上千文件的目录 /clinton 进行重命名操作。 同时，另一个用户对这个目录下的单个文件 /clinton/projects/elasticsearch/README.txt 进行重命名操作。 这个用户的修改操作，尽管在你的操作后开始，但可能会更快的完成。

以下有两种情况可能出现：

l 你决定使用 version （版本）号，在这种情况下，当与 README.txt 文件重命名的版本号产生冲突时，你的批量重命名操作将会失败。

l 你没有使用版本控制，你的变更将覆盖其他用户的变更。

以下是三个切实可行的使用 Elasticsearch 的解决方案，它们都涉及某种形式的锁：全局锁、文档锁、树锁

##### 15.1.4.1  全局锁

通过在任何时间只允许一个进程来进行变更动作，我们可以完全避免并发问题。

我们尝试 create 全局锁文档：

```
PUT /fs/lock/global/_create
{}
```

如果这个 create 请求因冲突异常而失败，说明另一个进程已被授予全局锁，我们将不得不稍后再试。 如果请求成功了，我们自豪的成为全局锁的主人，然后可以继续完成我们的变更。一旦完成，我们就必须通过删除全局锁文档来释放锁：

```
DELETE /fs/lock/global
```

##### 15.1.4.2  文档锁

使用前面描述相同的方法技术来锁定个体文档，而不是锁定整个文件系统。

文档会被变更影响因此每一个文档都创建了一个锁文件：

```
PUT /fs/lock/_bulk
{ "create": { "_id": 1}} 
{ "process_id": 123    } 
{ "create": { "_id": 2}}
{ "process_id": 123    }
```

注：

*1.*    *lock* *文档的* *ID* *将与应被锁定的文件的* *ID* *相同。*

*2.*    *process_id* *代表要执行变更进程的唯一* *ID**。*

##### 15.1.4.3  树锁（待）

可以锁定的目录树的一部分，而不是锁定每一个涉及的文档。

 

### 15.2   嵌套对象（部分代码已不可用）

#### 15.2.1  嵌套对象映射

设置一个字段为 nested 很简单 —  你只需要将字段类型 object 替换为 nested 即可：

```
PUT /my_index
{
  "mappings": {
    "blogpost": {
      "properties": {
        "comments": {
          "type": "nested", 
          "properties": {
            "name":    { "type": "string"  },
            "comment": { "type": "string"  },
            "age":     { "type": "short"   },
            "stars":   { "type": "short"   },
            "date":    { "type": "date"    }
          }
        }
      }
    }
  }
}
```

注：*nested* *字段类型的设置参数与* *object* *相同。*

至此，所有 comments 对象会被索引在独立的嵌套文档中。

#### 15.2.2  嵌套对象查询

由于嵌套对象 被索引在独立隐藏的文档中，我们无法直接查询它们。 相应地，我们必须使用 nested 查询 去获取它们：

```
GET /my_index/blogpost/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "eggs" 
          }
        },
        {
          "nested": {
            "path": "comments", 
            "query": {
              "bool": {
                "must": [ 
                  {
                    "match": {
                      "comments.name": "john"
                    }
                  },
                  {
                    "match": {
                      "comments.age": 28
                    }
                  }
                ]
              }
            }
          }
        }
      ]
}}}
```

注：  

*1.*    *title* *子句是查询根文档的。*

*2.*    *nested* *子句作用于嵌套字段* *comments* *。在此查询中，既不能查询根文档字段，也不能查询其他嵌套文档。*

*3.*    *comments.name* *和* *comments.age* *子句操作在同一个嵌套文档中。*

#### 15.2.3  使用嵌套字段排序

举例：

```
PUT /my_index/blogpost/2
{
  "title": "Investment secrets",
  "body":  "What they don't tell you ...",
  "tags":  [ "shares", "equities" ],
  "comments": [
    {
      "name":    "Mary Brown",
      "comment": "Lies, lies, lies",
      "age":     42,
      "stars":   1,
      "date":    "2014-10-18"
    },
    {
      "name":    "John Smith",
      "comment": "You're making it up!",
      "age":     28,
      "stars":   2,
      "date":    "2014-10-16"
    }
  ]
}
```

假如我们想要查询在10月份收到评论的博客文章，并且按照 stars 数的最小值来由小到大排序，那么查询语句如下：

```
GET /_search
{
  "query": {
    "nested": { 
      "path": "comments",
      "filter": {
        "range": {
          "comments.date": {
            "gte": "2014-10-01",
            "lt":  "2014-11-01"
          }
        }
      }
    }
  },
  "sort": {
    "comments.stars": { 
      "order": "asc",   
      "mode":  "min",   
      "nested_path": "comments", 
      "nested_filter": {
        "range": {
          "comments.date": {
            "gte": "2014-10-01",
            "lt":  "2014-11-01"
          }
        }
      }
    }
  }
}
```

 

注：  

*1.*    *此处的* *nested* *查询将结果限定为在**10**月份收到过评论的博客文章。*

*2.*    *结果按照匹配的评论中* *comment.stars* *字段的最小值* *(min)* *来由小到大* *(asc)* *排序。*

*3.*    *排序子句中的* *nested_path* *和* *nested_filter* *和* *query* *子句中的* *nested* *查询相同，原因在下面有解释。*

#### 15.2.4  嵌套聚合

在查询的时候，我们使用 nested 查询 就可以获取嵌套对象的信息。同理， nested 聚合允许我们对嵌套对象里的字段进行聚合操作。

```
GET /my_index/blogpost/_search
{
  "size" : 0,
  "aggs": {
    "comments": { 
      "nested": {
        "path": "comments"
      },
      "aggs": {
        "by_month": {
          "date_histogram": { 
            "field":    "comments.date",
            "interval": "month",
            "format":   "yyyy-MM"
          },
          "aggs": {
            "avg_stars": {
              "avg": { 
                "field": "comments.stars"
              }
            }
          }
        }
      }
    }
  }
}
```

注：

*1.*    *nested* *聚合* *“进入”* *嵌套的* *comments* *对象。*

*2.*    *comment**对象根据* *comments.date* *字段的月份值被分到不同的桶。*

*3.*    *计算每个桶内**star**的平均数量。*

结果：

 ![image-20200704003816941](../../../Typora/Picture/image-20200704003816941.png)

图 15—3

#### 15.2.5  逆向嵌套聚合

nested 聚合 只能对嵌套文档的字段进行操作。 根文档或者其他嵌套文档的字段对它是不可见的。 然而，通过 reverse_nested 聚合，我们可以 走出 嵌套层级，回到父级文档进行操作。

例如，我们要基于评论者的年龄找出评论者感兴趣 tags 的分布。 comment.age 是一个嵌套字段，但 tags 在根文档中：

```
GET /my_index/blogpost/_search
{
  "size" : 0,
  "aggs": {
    "comments": {
      "nested": { 
        "path": "comments"
      },
      "aggs": {
        "age_group": {
          "histogram": { 
            "field":    "comments.age",
            "interval": 10
          },
          "aggs": {
            "blogposts": {
              "reverse_nested": {}, 
              "aggs": {
                "tags": {
                  "terms": { 
                    "field": "tags"
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```

注：

*1.*    *nested* *聚合进入* *comments* *对象。*

*2.*    *histogram* *聚合基于* *comments.age* *做分组，每**10**年一个分组。*

*3.*    *reverse_nested* *聚合退回根文档。*

*4.*    *terms* *聚合计算每个分组年龄段的评论者最常用的标签词。*

### 15.3   父-子关系文档

#### 15.3.1  父-子关系概述

父-子关系的主要作用是允许把一个 type 的文档和另外一个 type 的文档关联起来，构成一对多的关系：一个父文档可以对应多个子文档 。与 nested objects 相比，父-子关系的主要优势有：更新父文档时，不会重新索引子文档。

创建，修改或删除子文档时，不会影响父文档或其他子文档。这一点在这种场景下尤其有用：子文档数量较多，并且子文档创建和修改的频率高时。

子文档可以作为搜索结果独立返回。

**优缺点：**

\1.    父子关系是非常有用的，但是它也是有巨大代价的。其查询速度会比同等的嵌套查询慢5到10倍!

\2.    父子关系使用了全局序数 来加速文档间的联合。不管父子关系映射是否使用了内存缓存或基于硬盘的 doc values，当索引变更时，全局序数要重建，一个分片中父文档越多，那么全局序数的重建就需要更多的时间。

\3.    多代文档的联合查询(查看 祖辈与孙辈关系)虽然看起来很吸引人，但必须考虑如下的代价：

l 联合越多，性能越差。

l 每一代的父文档都要将其字符串类型的 _id 字段存储在内存中，这会占用大量内存。

#### 15.3.2  父-子关系文档映射（不可用）

建立父-子文档映射关系时只需要指定某一个文档 type 是另一个文档 type 的父亲。 该关系可以在如下两个时间点设置：1）创建索引时；2）在子文档 type 创建之前更新父文档的 mapping。

举例，创建员工 employee 文档 type 时，指定分公司 branch 的文档 type 为其父亲。

```
PUT /company
{
  "mappings": {
    "branch": {},
    "employee": {
      "_parent": {
        "type": "branch" 
      }
    }
  }
}
```

注：*employee* *文档* *是* *branch* *文档的子文档。*

#### 15.3.3  构建父-子文档索引（不可用）

为父文档创建索引与为普通文档创建索引没有区别。

```
POST /company/branch/_bulk
{ "index": { "_id": "london" }}
{ "name": "London Westminster", "city": "London", "country": "UK" }
{ "index": { "_id": "liverpool" }}
{ "name": "Liverpool Central", "city": "Liverpool", "country": "UK" }
{ "index": { "_id": "paris" }}
{ "name": "Champs Élysées", "city": "Paris", "country": "France" }
```

创建子文档时，用户必须要通过 parent 参数来指定该子文档的父文档 ID：

```
PUT /company/employee/1?parent=london 
{
  "name":  "Alice Smith",
  "dob":   "1970-10-24",
  "hobby": "hiking"
}
```

注：*当前* *employee* *文档的父文档* *ID* *是* *london* *。*

父文档 ID 有两个作用：创建了父文档和子文档之间的关系，并且保证了父文档和子文档都在同一个分片上。

#### 15.3.4  通过子文档查询父文档

has_child 的查询和过滤可以通过子文档的内容来查询父文档。例如，我们根据如下查询，可查出所有80后员工所在的分公司：

 

```
GET /company/branch/_search
{
  "query": {
    "has_child": {
      "type": "employee",
      "query": {
        "range": {
          "dob": {
            "gte": "1980-01-01"
          }
        }
      }
    }
  }
}
```

##### 15.3.4.1  min_children 和 max_children

has_child 的查询和过滤都可以接受这两个参数：min_children 和 max_children 。 使用这两个参数时，只有当子文档数量在指定范围内时，才会返回父文档。

如下查询只会返回至少有两个雇员的分公司：

```
GET /company/branch/_search
{
  "query": {
    "has_child": {
      "type":         "employee",
      "min_children": 2, 
      "query": {
        "match_all": {}
      }
    }
  }
}
```

注：*至少有两个雇员的分公司才会符合查询条件。*

#### 15.3.5  通过父文档查询子文档

使用 has_parent 语句可以基于父文档来查询子文档，has_parent 和 has_child 非常相似，下面的查询将会返回所有在 UK 工作的雇员：

```
GET /company/employee/_search
{
  "query": {
    "has_parent": {
      "type": "branch", 
      "query": {
        "match": {
          "country": "UK"
        }
      }
    }
  }
}
```

注：*返回父文档* *type* *是* *branch* *的所有子文档*

has_parent 查询也支持 score_mode 这个参数，但是该参数只支持两种值： none （默认）和 score 。每个子文档都只有一个父文档，因此这里不存在将多个评分规约为一个的情况， score_mode 的取值仅为 score 和 none 。

#### 15.3.6  子文档聚合

通过下面的例子来演示按照国家维度查看最受雇员欢迎的业余爱好：

```
GET /company/branch/_search
{
  "size" : 0,
  "aggs": {
    "country": {
      "terms": { 
        "field": "country"
      },
      "aggs": {
        "employees": {
          "children": { 
            "type": "employee"
          },
          "aggs": {
            "hobby": {
              "terms": { 
                "field": "hobby"
              }
            }
          }
        }
      }
    }
  }
}
```

注：  

*1.*    *country* *是* *branch* *文档的一个字段。*

*2.*    *子文档聚合查询通过* *employee type* *的子文档将其父文档聚合在一起。*

*3.*    *hobby* *是* *employee* *子文档的一个字段。*

#### 15.3.7  祖辈与孙辈关系

父子关系可以延展到更多代关系，比如生活中孙辈与祖辈的关系 — 唯一的要求是满足这些关系的文档必须在同一个分片上被索引。

让我们把上一个例子中的 country 类型设定为 branch 类型的父辈：

```
PUT /company
{
  "mappings": {
    "country": {},
    "branch": {
      "_parent": {
        "type": "country" 
      }
    },
    "employee": {
      "_parent": {
        "type": "branch" 
      }
    }
  }
}
```

注：

*1.*    *branch* *是* *country* *的子辈。*

*2.*    *employee* *是* *branch* *的子辈。*

country 和 branch 之间是一层简单的父子关系，所以我们的 操作步骤 与之前保持一致：

```
POST /company/country/_bulk
{ "index": { "_id": "uk" }}
{ "name": "UK" }
{ "index": { "_id": "france" }}
{ "name": "France" }
 
POST /company/branch/_bulk
{ "index": { "_id": "london", "parent": "uk" }}
{ "name": "London Westmintster" }
{ "index": { "_id": "liverpool", "parent": "uk" }}
{ "name": "Liverpool Central" }
{ "index": { "_id": "paris", "parent": "france" }}
{ "name": "Champs Élysées" }
```

parent ID 使得每一个 branch 文档被路由到与其父文档 country 相同的分片上进行操作。然而，当我们使用相同的方法来操作 employee 这个孙辈文档时，会发生什么呢？

```
PUT /company/employee/1?parent=london
{
  "name":  "Alice Smith",
  "dob":   "1970-10-24",
  "hobby": "hiking"
}
```

employee 文档的路由依赖其父文档 ID — 也就是 london — 但是 london 文档的路由却依赖 其本身的 父文档 ID — 也就是 uk 。此种情况下，孙辈文档很有可能最终和父辈、祖辈文档不在同一分片上，导致不满足祖辈和孙辈文档必须在同一个分片上被索引的要求。

解决方案是添加一个额外的 routing 参数，将其设置为祖辈的文档 ID ，以此来保证三代文档路由到同一个分片上。索引请求如下所示：

```
PUT /company/employee/1?parent=london&routing=uk 
{
  "name":  "Alice Smith",
  "dob":   "1970-10-24",
  "hobby": "hiking"
}
```

注：*routing* *的值会取代* *parent* *的值作为路由选择*

查询

```
GET /company/country/_search
{
  "query": {
    "has_child": {
      "type": "branch",
      "query": {
        "has_child": {
          "type": "employee",
          "query": {
            "match": {
              "hobby": "hiking"
            }
          }
        }
      }
    }
  }
}
```

### 15.4   扩容设计

#### 15.4.1  扩容的单元

一个分片即为 扩容的单元 。

#### 15.4.2  分片预分配

一个分片存在于单个节点，但一个节点可以持有多个分片。想象一下我们创建拥有两个主分片的索引而不是一个：

```
PUT /my_index
{
  "settings": {
    "number_of_shards":   2, 
    "number_of_replicas": 0
  }
}
```

注：*创建拥有两个主分片无副本分片的索引。*

#### 15.4.3  副本分片

副本分片的主要目的就是为了故障转移，如果持有主分片的节点挂掉了，一个副本分片就会晋升为主分片的角色。

#### 15.4.4  多索引

搜索 1 个有着 50 个分片的索引与搜索 50 个每个都有 1 个分片的索引完全等价：搜索请求均命中 50 个分片。

当你需要在不停服务的情况下增加容量时，下面有一些有用的建议。相较于将数据迁移到更大的索引中，你可以仅仅做下面这些操作：

l 创建一个新的索引来存储新的数据。

l 同时搜索两个索引来获取新数据和旧数据。

#### 15.4.5  按时间范围索引

日志事件会不断的进来，不会停顿也不会中断。 我们可以使用 scroll 查询和批量删除来删除旧的事件。

替代方案是，我们使用一个 时间范围索引。你可以着手于一个按年的索引 (logs_2014) 或按月的索引 (logs_2014-10) 。 也许当你的网页变得十分繁忙时，你需要切换到一个按天的索引 (logs_2014-10-24) 。删除旧数据十分简单：只需要删除旧的索引。

别名可以帮助我们更加透明地在索引间切换。 当创建索引时，你可以将 logs_current 指向当前索引来接收新的日志事件， 当检索时，更新 last_3_months 来指向所有最近三个月的索引：

```
POST /_aliases
{
  "actions": [
    { "add":    { "alias": "logs_current",  "index": "logs_2014-10" }}, 
    { "remove": { "alias": "logs_current",  "index": "logs_2014-09" }}, 
    { "add":    { "alias": "last_3_months", "index": "logs_2014-10" }}, 
    { "remove": { "alias": "last_3_months", "index": "logs_2014-07" }}  
  ]
}
```

注：

*1.*    *将* *logs_current* *由九月切换至十月。*

*2.*    *将十月添加到* *last_3_months* *并且删掉七月。*

#### 15.4.6  索引模板

Logstash 使用事件中的时间戳来生成索引名。 默认每天被索引至不同的索引中，因此一个 @timestamp 为 2014-10-01 00:00:01 的事件将被发送至索引 logstash-2014.10.01 中。 如果那个索引不存在，它将被自动创建。

通常我们想要控制一些新建索引的设置（settings）和映射（mappings）。也许我们想要限制分片数为 1 ，并且禁用 _all 域。 索引模板可以用于控制何种设置（settings）应当被应用于新创建的索引：

```
PUT /_template/my_logs 
{
  "template": "logstash-*", 
  "order":    1, 
  "settings": {
    "number_of_shards": 1 
  },
  "mappings": {
    "_default_": { 
      "_all": {
        "enabled": false
      }
    }
  },
  "aliases": {
    "last_3_months": {} 
  }
}
```

注：  

*1.*    *创建一个名为* *my_logs* *的模板。*

*2.*    *将这个模板应用于所有以* *logstash-* *为起始的索引。*

*3.*    *这个模板将会覆盖默认的* *logstash* *模板，因为默认模板的* *order* *更低。*

*4.*    *限制主分片数量为* *1* *。*

*5.*    *为所有类型禁用* *_all* *域。*

*6.*    *添加这个索引至* *last_3_months* *别名中。*

#### 15.4.7  数据过期

随着时间推移，对于一些不需要的数据，我们会想着如何去掉这些数据：按时间范围索引可以方便地删除旧数据：只需要删除那些变得不重要的索引就可以了。 

#### 15.4.8  迁移旧索引

可以通过按以下配置创建索引来确保它被分配到我们最好的服务器上：

```
PUT /logs_2014-10-01
{
  "settings": {
    "index.routing.allocation.include.box_type" : "strong"
  }
}
```

以前的索引不再需要我们最好的服务器了，我们可以通过更新索引设置将它移动到标记为 medium 的节点上：

```
POST /logs_2014-09-30/_settings
{
  "index.routing.allocation.include.box_type" : "medium"
}
```

#### 15.4.9  索引优化（Optimize）

日志事件是静态的：已经发生的过往不会再改变了。如果我们将以前的每个分片合并至一个段（Segment），它会占用更少的资源更快地响应查询。

可以临时移除副本分片，进行优化，然后再恢复副本分片：

```
POST /logs_2014-09-30/_settings
{ "number_of_replicas": 0 }
 
POST /logs_2014-09-30/_optimize?max_num_segments=1
 
POST /logs_2014-09-30/_settings
{ "number_of_replicas": 1 }
```

#### 15.4.10 关闭旧索引

一个几乎不会再被访问的时间点，我们可以删除它，也可以关闭它：

```
POST /logs_2014-01-*/_flush 
POST /logs_2014-01-*/_close 
POST /logs_2014-01-*/_open 
```

注：

*1.*    *刷写（**Flush**）所有一月的索引来清空事务日志。*

*2.*    *关闭所有一月的索引**.*

*3.*    *当你需要再次访问它们时，使用* *open API* *来重新打开它们。*

#### 15.4.11 共享索引

我们可以为许多的小论坛使用一个大的共享的索引， 将论坛标识索引进一个字段并且将它用作一个过滤器：

```
PUT /forums
{
  "settings": {
    "number_of_shards": 10 
  },
  "mappings": {
    "post": {
      "properties": {
        "forum_id": { 
          "type":  "string",
          "index": "not_analyzed"
        }
      }
    }
  }
}
 
PUT /forums/post/1
{
  "forum_id": "baking", 
  "title":    "Easy recipe for ginger nuts",
  ...
}
```

注：

*1.*    *创建一个足够大的索引来存储数千个小论坛的数据。*

*2.*    *每个帖子都必须包含一个* *forum_id* *来标识它属于哪个论坛。*

## 16   管理、监控和部署

### 16.1   监控

#### 16.1.1  Marvel 监控

Marvel 让你可以很简单的通过 Kibana 监控 Elasticsearch。你可以实时查看你的集群健康状态和性能，也可以分析过去的集群、索引和节点指标。

#### 16.1.2  集群健康

使用下面代码就可以查看集群健康状况：

```
GET _cluster/health
```

问题集：

集群是 red ，意味着我们缺数据（主分片 + 副本分片）了，如果需要查看具体哪些分片可以使用下面代码：

```
GET _cluster/health?level=indices
```

shards 选项会提供一个详细得多的输出，列出每个索引里每个分片的状态和位置：

```
GET _cluster/health?level=shards
```

#### 16.1.3  阻塞等待状态变化

当构建单元和集成测试时，或者实现和 Elasticsearch 相关的自动化脚本时，cluster-health API 还有另一个小技巧非常有用。你可以指定一个 wait_for_status 参数，它只有在状态达标之后才会返回。比如：

```
GET _cluster/health?wait_for_status=green
```

这个调用会 阻塞 （不给你的程序返回控制权）住直到 cluster-health 变成 green ，也就是说所有主分片和副本分片都分配下去了。这对自动化脚本和测试非常重要。

#### 16.1.4  监控单个节点

节点统计值 API 可以通过如下命令执行：

```
GET _nodes/stats
```

输出节信息。

##### 16.1.4.1  索引部分

索引(indices) 部分列出了这个节点上所有索引的聚合过的统计值：

```
"indices": {
        "docs": {
           "count": 6163666,
           "deleted": 0
        },
        "store": {
           "size_in_bytes": 2301398179,
           "throttle_time_in_millis": 122850
        },
```

##### 16.1.4.2  JVM 部分（待）

JVM 在内存小于 32 GB，最佳为31G

 

##### 16.1.4.3  线程池部分（待）

 

##### 16.1.4.4  文件系统和网络部分（待）

 

##### 16.1.4.5  断路器

跟 fielddata 断路器相关的统计值：

 ![image-20200704003841588](../../../Typora/Picture/image-20200704003841588.png)

图 16—1

这里你可以看到断路器的最大值（比如，一个请求申请更多的内存时会触发断路器）。这个部分还会让你知道断路器被触发了多少次，以及当前配置的间接开销。间接开销用来铺垫评估，因为有些请求比其他请求更难评估。

主要需要关注的是 tripped 指标。如果这个数字很大或者持续上涨，这是一个信号，说明你的请求需要优化，或者你需要添加更多内存（单机上添加，或者通过添加新节点的方式）。

#### 16.1.5  集群统计

节点统计显示的是每个节点上的统计值，而 集群统计 展示的是对于单个指标，所有节点的总和值。

```
GET _cluster/stats
```

#### 16.1.6  索引统计

```
GET my_index/_stats 
 
GET my_index,another_index/_stats 
 
GET _all/_stats 
```

注：

*1.*    *统计* *my_index* *索引。*

*2.*    *使用逗号分隔索引名可以请求多个索引统计值。*

*3.*    *使用特定的* *_all* *可以请求全部索引的统计值*

#### 16.1.7  等待中的任务

在一些 罕见 的集群里，元数据变动的次数比主节点能处理的还快。这会导致等待中的操作会累积成队列。查询等待中的队列任务：

```
GET _cluster/pending_tasks
```

#### 16.1.8  cat API

可以将这章节前面讲的使用纯文本展示：

```
GET /_cat
```



```
 
=^.^=
/_cat/allocation
/_cat/shards
/_cat/shards/{index}
/_cat/master
/_cat/nodes
/_cat/indices
/_cat/indices/{index}
/_cat/segments
/_cat/segments/{index}
/_cat/count
/_cat/count/{index}
/_cat/recovery
/_cat/recovery/{index}
/_cat/health
/_cat/pending_tasks
/_cat/aliases
/_cat/aliases/{alias}
/_cat/thread_pool
/_cat/plugins
/_cat/fielddata
/_cat/fielddata/{fields}
```

例如查询集群健康状况

 ![image-20200704003852887](../../../Typora/Picture/image-20200704003852887.png)

图 16—2

### 16.2   部署（重要）

#### 16.2.1  硬件

##### 16.2.1.1  内存

64 GB 内存的机器是非常理想的， 但是32 GB 和16 GB 机器也是很常见的。少于8 GB 会适得其反（你最终需要很多很多的小机器），大于64 GB 的机器也会有问题。

##### 16.2.1.2  CPUs

大多数 Elasticsearch 部署往往对 CPU 要求不高。因此，相对其它资源，具体配置多少个（CPU）不是那么关键。你应该选择具有多个内核的现代处理器，常见的集群使用两到八个核的机器。

如果你要在更快的 CPUs 和更多的核心之间选择，选择更多的核心更好。多个内核提供的额外并发远胜过稍微快一点点的时钟频率。

##### 16.2.1.3  硬盘

硬盘对所有的集群都很重要，如果你负担得起 SSD，它将远远超出任何旋转介质（注：机械硬盘，磁带等）。 基于 SSD 的节点，查询和索引性能都有提升。如果你负担得起，SSD 是一个好的选择。

##### 16.2.1.4  网络

快速可靠的网络显然对分布式系统的性能是很重要的。 低延时能帮助确保节点间能容易的通讯，大带宽能帮助分片移动和恢复。现代数据中心网络（1 GbE, 10 GbE）对绝大多数集群都是足够的。

#### 16.2.2  Transport Client 与 Node Client

如果你使用的是 Java，你可能想知道何时使用传输客户端（注：Transport Client，下同）与节点客户端（注：Node Client，下同）。 在书的开头所述， 传输客户端作为一个集群和应用程序之间的通信层。它知道 API 并能自动帮你在节点之间轮询，帮你嗅探集群等等。但它是集群 外部的 ，和 REST 客户端类似。

另一方面，节点客户端，实际上是一个集群中的节点（但不保存数据，不能成为主节点）。因为它是一个节点，它知道整个集群状态（所有节点驻留，分片分布在哪些节点，等等）。 这意味着它可以执行 APIs 但少了一个网络跃点。

这里有两个客户端案例的使用情况：

l 如果要将应用程序和 Elasticsearch 集群进行解耦，传输客户端是一个理想的选择。例如，如果您的应用程序需要快速的创建和销毁到集群的连接，传输客户端比节点客户端”轻”，因为它不是一个集群的一部分。

l 类似地，如果您需要创建成千上万的连接，你不想有成千上万节点加入集群。传输客户端（ TC ）将是一个更好的选择。

l 另一方面，如果你只需要有少数的、长期持久的对象连接到集群，客户端节点可以更高效，因为它知道集群的布局。但是它会使你的应用程序和集群耦合在一起，所以从防火墙的角度，它可能会构成问题。

#### 16.2.3  配置管理

如果你已经使用配置管理（ Puppet，Chef，Ansible），则可以跳过此提示。

如果你没有使用配置管理工具，那么应该注意了！通过 parallel-ssh 管理少量服务器现在可能正常工作，但伴随着集群的增长它将成为一场噩梦。 在不犯错误的情况下手动编辑 30 个配置文件几乎是不可能的。

配置管理工具通过自动化更改配置的过程保持集群的一致性。这可能需要一点时间来建立和学习，但它本身，随着时间的推移会有丰厚的回报。

#### 16.2.4  集群配置（重要）

主要包含配置集群的时候，配置文件里面的一些参数的详细说明，详情见[链接](https://www.elastic.co/guide/cn/elasticsearch/guide/current/important-configuration-changes.html)。

 

#### 16.2.5  不可变动配置！

##### 16.2.5.1  垃圾回收器

Elasticsearch 默认的垃圾回收器（ GC ）是 CMS。

##### 16.2.5.2  线程池

Elasticsearch 默认的线程设置已经是很合理的了。对于所有的线程池（除了 搜索 ），线程个数是根据 CPU 核心数设置的。 如果你有 8 个核，你可以同时运行的只有 8 个线程，只分配 8 个线程给任何特定的线程池是有道理的。

##### 16.2.5.3  Swapping 是性能的坟墓

内存交换 到磁盘对服务器性能来说是 致命 的。想想看：一个内存操作必须能够被快速执行。

如果内存交换到磁盘上，一个 100 微秒的操作可能变成 10 毫秒。 再想想那么多 10 微秒的操作时延累加起来。 不难看出 swapping 对于性能是多么可怕。

最好的办法就是在你的操作系统中完全禁用 swap。这样可以暂时禁用：

```
sudo swapoff -a
```

 

#### 16.2.6  堆内存:大小和交换

##### 16.2.6.1  堆内存配置

Elasticsearch 默认安装后设置的堆内存是 1 GB。对于任何一个业务部署来说， 这个设置都太小了。如果你正在使用这些默认堆内存配置，您的集群可能会出现问题。

这里有两种方式修改 Elasticsearch 的堆内存。最简单的一个方法就是指定 ES_HEAP_SIZE 环境变量。服务进程在启动时候会读取这个变量，并相应的设置堆的大小。 比如，你可以用下面的命令设置它：

```
export ES_HEAP_SIZE=10g
```

此外，你也可以通过命令行参数的形式，在程序启动的时候把内存大小传递给它，如果你觉得这样更简单的话：

```
./bin/elasticsearch -Xmx10g -Xms10g 
```

注：*确保堆内存最小值（* *Xms* *）与最大值（* *Xmx* *）的大小是相同的，防止程序在运行时改变堆内存大小，* *这是一个很耗系统资源的过程。*

通常来说，设置 ES_HEAP_SIZE 环境变量，比直接写 -Xmx -Xms 更好一点。

#### 16.2.7  文件描述符和 MMap（待）

 

### 16.3   部署后（待）（重要）

#### 16.3.1  动态变更设置

集群更新 API 有两种工作模式：

l 临时（Transient）：这些变更在集群重启之前一直会生效。一旦整个集群重启，这些配置就被清除。

l 永久（Persistent）：这些变更会永久存在直到被显式修改。即使全集群重启它们也会存活下来并覆盖掉静态配置文件里的选项。

临时或永久配置需要在 JSON 体里分别指定：

```
PUT /_cluster/settings
{
    "persistent" : {
        "discovery.zen.minimum_master_nodes" : 2 
    },
    "transient" : {
        "indices.store.throttle.max_bytes_per_sec" : "50mb" 
    }
}
```

注：

*1.*    *这个永久设置会在全集群重启时存活下来。*

*2.*    *这个临时设置会在第一次全集群重启后被移除*

#### 16.3.2  日志记录

 

 

#### 16.3.3  索引性能技巧

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 