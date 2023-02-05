# MongoDB的使用

## <u>1.数据库的操作</u>

### 1.选择和创建数据库的语法：

```mariadb
use 数据库名 # 如果不存在则会自动创建
```

**￥数据库的命名**：

数据库名可以是满足以下条件的任意UTF-8字符串。

- 不能是空字符串（"")。

- 不得含有' '（空格)、.、$、/、\和\0 (空字符)。

- 应全部小写。

- 最多64字节。

有一些数据库名是保留的，可以直接访问这些有特殊作用的数据库。

- **admin**: 从权限的角度来看，这是"root"数据库。要是将一个用户添加到这个数据库，这个用户自动继承所有数据库的权限。一些特定的服务器端命令也只能从这个数据库运行，比如列出所有的数据库或者关闭服务器。

- **local**: 这个数据永远不会被复制，可以用来存储限于本地单台服务器的任意集合。

- **confifig**: 当Mongo用于分片设置时，confifig数据库在内部使用，用于保存分片的相关信息。

### 2.查看有权访问的数据库：

```mariadb
show dbs
或
show databases
```

> **注意**:在 MongoDB 中，集合只有在内容插入后才会创建! 就是说，创建集合(数据表)后要再插入一个文档(记录)，集合才会真正创建。

### 3.查看正在使用的数据库：

```mariadb
db
```

> **注意**：MongoDB默认的数据库是test，如果没有选择数据库，则集合会被放到test库中！

### 4.删除数据库：

```mariadb
db.dropDatabase()
```

> **提示**：主要用来删除已经持久化的数据库

## <u>2.集合（表）的操作</u>

### 1.创建一个集合：

```mariadb
db.createCollection("user") # 括号内是集合的名称
```

**￥集合的命名规范**：

- 集合名不能是空字符串""。

- 集合名不能含有\0字符（空字符)，这个字符表示集合名的结尾。

- 集合名不能以"system."开头，这是为系统集合保留的前缀。

- 用户创建的集合名字不能含有保留字符。有些驱动程序的确支持在集合名里面包含，这是因为某些系统生成的集合中包含该字符。除非你要访问这种系统创建的集合，否则千万不要在名字里出现$。

### 2.集合的删除：

```mariadb
db.集合名.drop() # 返回true或false
```

### 3.查询集合：

```mariadb
show tables;
或
show collections;
```

## <u>3.文档（数据）的操作</u>

文档（document）的数据结构和JSON基本一样。

所有存储在集合中的数据都是BSON格式。

### 1.文档的插入：

使用insert()或save()方法，语法：

```mariadb
db.collection_name.insert(
    <document or array of documents>,# 要插入的文档或文档数组
    {
        writeConcern:<document>,
        ordered:<boolean>
    }
)
```

示例：

```mariadb
db.comment.insert( # 如果comment不存在会隐式创建
    {
    "articleid":"100000",
    "content":"今天天气真好，阳光明媚",
    "userid":"1001",
    "nickname":"Rose",
    "createdatetime":new Date(),
    "likenum":NumberInt(10),# 数字默认double，NumberInt()整型
    "state":null
    }
)
```

### 2.批量插入：

```mariadb
db.collection_name.insertMany(
    [ <document 1> , <document 2>, ... ],
    {
        writeConcern: <document>,
        ordered: <boolean>
    }
)
```

示例：

```mariadb
db.comment.insertMany(
    [
        {
        "_id":"1","articleid":"100001","content":"我们不应该把清晨浪费在手机上，健康很重要，一杯温水幸福你我他。",
        "userid":"1002",
        "nickname":"相忘于江湖",
        "createdatetime":new Date("2019-08-05T22:08:15.522Z"),
        "likenum":NumberInt(1000),
        "state":"1"
        },
        {
        "_id":"2",
        "articleid":"100001",
        "content":"我夏天空腹喝凉开水，冬天喝温开水",
        "userid":"1005",
        "nickname":"伊人憔悴",
        "createdatetime":new Date("2019-08-05T23:58:51.485Z"),
        "likenum":NumberInt(888),
        "state":"1"
        },
        {
        "_id":"3",
        "articleid":"100001",
        "content":"我一直喝凉开水，冬天夏天都喝。",
        "userid":"1004",
        "nickname":"杰克船长",
        "createdatetime":new Date("2019-08-06T01:05:06.321Z"),
        "likenum":NumberInt(666),
        "state":"1"
        },
        {
        "_id":"4",
        "articleid":"100001",
        "content":"专家说不能空腹吃饭，影响健康。",
        "userid":"1003",
        "nickname":"凯撒",
        "createdatetime":new Date("2019-08-06T08:18:35.288Z"),
        "likenum":NumberInt(2000),
        "state":"1"
        },
        {
        "_id":"5",
        "articleid":"100001",
        "content":"研究表明，刚烧开的水千万不能喝，因为烫嘴。",
        "userid":"1003",
        "nickname":"凯撒",
        "createdatetime":new Date("2019-08-06T11:01:02.521Z"),
        "likenum":NumberInt(3000),
        "state":"1"
        }
    ]
);
```

**提示**：

插入时指定了 _id ，则主键就是该值。

如果某条数据插入失败，将会终止插入，但已经插入成功的数据不会回滚掉。

因为批量插入由于数据较多容易出现失败，因此，可以使用try catch进行异常捕捉处理，测试的时候可以不处理。如（了解）：

```mariadb
try {
db.comment.insertMany(
    [
       ......
    ]
);
} catch (e) {
    print (e);
}
```

### 3.文档的删除：

```mariadb
db.集合名称.remove(条件)
db.comment.remove({}) # 全部删除
db.comment.remove({_id:"1"}) # 删除id为1的文档，可批量
```

### 4.文档的修改：

1.普通修改：

```mariadb
db.集合名.update(query,update,options)
// 或
db.collection_name.update(
    <query>, # 条件
    <update>,
    {
    upsert:<boolean>,# true：无符合会新创建，false：无符合不会插入
    multi:<boolean>,# true：多个符合都插入，false：多个符合只插入第一个
    writeConcern: <document>,
    writeConcern: <document>,
    writeConcern: <document>,
    writeConcern: <document>,
    }
)
```

2.覆盖修改：

```mariadb
db.comment.update({_id:"1"},{likenum:NumberInt(1001)})
```

> 执行后，我们会发现，这条文档除了likenum字段其它字段都不见了

3.局部修改：

> 使用修改器$set来实现

```mariadb
db.comment.update({_id:"2"},{$set:{likenum:NumberInt(889)}})
```

4.批量修改：

```mariadb
//默认只修改第一条数据
db.comment.update({userid:"1003"},{$set:{nickname:"凯撒2"}})
//修改所有符合条件的数据
db.comment.update({userid:"1003"},{$set:{nickname:"凯撒大帝"}},{multi:true})
```

> **提示**：如果不加后面的参数，则只更新符合条件的第一条记录

5.列值增长修改：

> 使用 $inc 运算符来实现

```mariadb
db.comment.update({_id:"3"},{$inc:{likenum:NumberInt(1)}})
```

### 5.文档的查询：

查询的语法格式：

```mariadb
# query条件 projection投影字段
db.collection_name.find(<query>,[projection]]);
```

1.查询所有：

```mariadb
db.comment.find();
或
db.comment.find({});
```

2.条件查询：

```mariadb
# 返回符合条件的文档
db.comment.find({userid:"1003"});
# 返回符合条件的第一条文档
db.comment.findOne({userid:"1003"});
```

3.投影查询（查询部分字段）：

```mariadb
# 1.结果只显示_id、userid、nickname
db.comment.find({userid:"1003"},{userid:1,nickname:1})
示例：
test1> db.comment.find({userid:"1003"},{userid:1,nickname:1})
[
  { _id: '4', userid: '1003', nickname: '凯撒' },
  { _id: '5', userid: '1003', nickname: '凯撒' }
]

# 默认_id会显示

# 2.只查询userid、nickname，不显示_id
db.comment.find({userid:"1003"},{userid:1,nickname:1,_id:0})
示例：
test1> db.comment.find({userid:"1003"},{userid:1,nickname:1,_id:0})
[
  { userid: '1003', nickname: '凯撒' },
  { userid: '1003', nickname: '凯撒' }
]
```

#### 1.统计查询`count()`：

语法：

```mariadb
# query查询条件  options修改计数的额外选项
db.collection_name.count(query,options)
```

1.统计所有记录数：

```mariadb
db.comment.count();
```

2.按条件统计记录数：

```mariadb
db.comment.count({userid:"1003"});
```

#### 2.分页查询`limit()`:

```mariadb
# limit默认20；skip默认0
db.collection_name.find().limit(NUMBER).skip(NUMBER);
# 查询前三个
db.comment.find().limit(3);
# 查询跳过前三个的二十条数据
db.comment.find().skip(3);
```

示例：

```mariadb
# 查询每页2个，第二页开始
db.comment.find().skip(0).limit(2); # 第一页
db.comment.find().skip(2).limit(2); # 第二页
db.comment.find().skip(4).limit(2); # 第三页
```

#### 3.排序查询`sort()`:

> 使用 1 和 -1 来指定排序的方式，其中 1 为升序排列，而 -1 是用于降序排列。

语法：

```mariadb
db.collection_name.find().sort({KEY:1})
或
db.集合名称.find().sort(排序方式)
```

示例：

```mariadb
# 对userid降序，对likenum升序
db.comment.find().sort({userid:1},{likenum:-1})
```

> 提示优先级：
>
> sort() > skip() > limit()

### 6.文档的更多查询：

#### 1.正则查询：

MongoDB的**模糊查询是通过正则表达式实现**的，格式：

```mariadb
db.collection_name.find(field:/正则/);
或
db.集合.find({字段:/正则表达式/})
```

> **提示**：正则表达式是js的语法，直接量的写法。

示例：

```mariadb
# 查询评论包含“开水”的所有文档
db.comment.find({content:/开水/});
# 查询评论内容以“专家”开头的所有文档
db.comment.find({content:/^专家/});
```

#### 2.比较查询：

<, <=, >, >= 这个操作符也是很常用的，格式如下:

```mariadb
db.集合名称.find({ "field" : { $gt: value }}) // 大于: field > value
db.集合名称.find({ "field" : { $lt: value }}) // 小于: field < value
db.集合名称.find({ "field" : { $gte: value }}) // 大于等于: field >= value
db.集合名称.find({ "field" : { $lte: value }}) // 小于等于: field <= value
db.集合名称.find({ "field" : { $ne: value }}) // 不等于: field != value
```

示例：

```mariadb
# 查询评论点赞数量大于700的记录
db.comment.find({likenum:{$gt:NumberInt(700)}});
```

#### 3.包含查询：

包含使用$in操作符。

```maria
# 查询评论的集合中userid字段包含1003或1004的文档
db.comment.find({userid:{$in:["1003","1004"]}});
```

不包含使用$nin操作符：

```mariadb
# 查询评论的集合中userid字段不包含1003或1004的文档
db.comment.find({userid:{$nin:["1003","1004"]}});
```

#### 4.条件连接查询：

我们如果需要查询同时满足两个以上条件，需要使用$and操作符将条件进行关联。（相 当于SQL的and） 格式为：

语法：

```mariadb
db.collection_name.find({$and:[ { },{ },{ } ]});
```

示例：

```mariadb
# 查询评论集合中likenum大于等于700 并且小于2000的文档：
db.comment.find({$and:[{likenum:{$gte:NumberInt(700)}},{likenum:{$lte:Number(2000)}}]});
```

如果两个以上条件之间是或者的关系，我们使用 操作符进行关联，与前面 and的使用方式相同 格式为：

语法：

```mariadb
db.comment.find({$or:[ { },{ },{ } ]});
```

示例：

```mariadb
# ：查询评论集合中userid为1003，或者点赞数小于1000的文档记录
db.comment.find({$or:[{userid:NumberInt(1003)},{likenum:{$lt:Number(1000)}}]});
```

### 7.常用命令小结：

```mariadb
1.选择切换数据库：use db_name;
2.插入数据：db.comment.insert({BSON数据});
3.查询所有数据：db.collection_name.find();
4.条件查询数据：db.collection_name.find({条件});
5.查询符合条件的第一条数据：db.collection_name.findOne({条件});
6.查询符合条件的前几条数据：db.collection_name.find({条件}).limit(条数);
7.查询符合条件的跳过的记录：db.collection_name.find({条件}).skip(跳过数);
8.修改数据：db.collection_name.update({条件},{修改后的数据}) 
        或db.collection_name.update({条件},{$set:{要修改部分的字段:数据})
9.修改数据并自增某字段值：db.collection_name.update({条件},{$inc:{自增的字段:步进值}})
10.删除数据：db.collection_name.remove({条件});
11.统计查询：db.collection_name.count({条件});
12.条件比较运算：db.collection_name.find({field:{$gt:值}});
13.包含查询：db.collection_name.find({$in:[值1,值2]); # $nin不包含
14.条件连接查询;db.collection_name.find({$and:[{条件1},{条件2}]}); # $or或
```

## <u>4.索引-Index</u>