---
title: MongoDB 学习笔记
---

## 基本概念

- 文档是 MongoDB 中数据的基本单元，类似于关系型数据库中的行，但是 MongoDB 的文档可以包含更多层级的结构，这样可以更好地表示复杂的数据结构。
- 集合可以被看作是 MongoDB 中的表，但是集合中的文档可以有不同的结构，这样可以更好地表示非结构化的数据。
- MongoDB 的单个实例可以容纳多个独立的数据库，每个数据库都有自己的集合和权限。
- MongoDB 自带简洁但功能强大的 JavaScript shell，可以用来管理 MongoDB 的实例和操作数据。
- 每一个文档都有一个特殊的键 `_id`，这个键在文档所属的集合中是唯一的，类似于关系型数据库中的主键。

### 文档

文档是 MongoDB 的核心概念，多个键及其关联的值有序地放置在一起便是文档，例如：

```json
{
  "greeting": "Hello, World!",
  "foo": 3
}
```

上述例子解释了几个重要概念：

- 文档中的键值对是有序的，这一点和 JSON 不同，JSON 中的键值对是无序的。
- 文档中的值不仅可以是简单的类型，例如字符串、整数、浮点数，还可以是复杂的类型，例如数组、嵌套的文档。
- 文档的键是字符串，但是键不仅仅是字符串，还可以是任意的 UTF-8 字符
  - 键不能含有 `\0` (空字符)，这是因为在 MongoDB 中，键的结尾是用 `\0` 来标识的。
  - `.` 和 `$` 有特殊的含义，不能作为键的开头。
  - 以下划线开头的键是保留的，不能使用。
  - MongoDB 的文档不能有重复的键。

### 集合

集合就是一组文档，类似于关系型数据库中的表

#### 无模式

集合中的文档可以有不同的结构，这样可以更好地表示非结构化的数据。例如，下面两个文档可以存在于同一个集合里面：

```
{"greeting": "Hello, World!"}
{"foo": 5}
```

这两个文档不光是值的类型不同，键也完全不一样，随之而来的问题是：还有必要使用多个集合吗？答案是：有必要的，理由如下：

- 把各种文档都混在一个集合里面，会使得集合变得杂乱无章，不利于维护。
- 在一个集合里面查询特定类型的文档在速度上也会变慢，因为 MongoDB 需要扫描集合中的每一个文档，才能找到需要的文档。
- 把同种类型的文档放在一个集合里，这样的数据会更加集中
- 当创建索引的时候，文档会有附加的结构（尤其是有唯一索引的时候）。索引是按照集合来定义的，把同种类型的文档放在一个集合里面，可以使索引更加有效。

#### 命名

集合名可以是满足下列条件的任意 UTF-8 字符串：

- 不能是空字符串（`""`）
- 不能含有 `\0` (空字符)，这个字符表示集合名的结尾
- 不能以 `system.` 开头，这是为系统集合保留的前缀
- 不能含有保留字符 `$`

子集合：组织集合的一种惯例是使用 `.` 字符分开的按照命名空间划分的子集合

### 数据库

多个集合可以组成数据库，一个 MongoDB 实例可以承载多个数据库，它们之间可视为完全独立的。每个数据库都有独立的权限控制，即便是在磁盘上，不同的数据库也放置在不同的文件中。

数据库命名需要是满足以下条件的 UTF-8 字符串：

- 不能是空字符串（`""`）
- 不得含有空格、`.`、`$`、`/`、`\` 和 `\0` 字符
- 应全部小写
- 最多 64 个字节

有一些数据库名是保留的，例如 `admin`、`local`、`config`，这些数据库名是特殊用途的，不能用于存储数据。

启动 MongoDB 的时候，默认会连接到 `test` 数据库，如果这个数据库不存在，MongoDB 会自动创建它。启动的命令为：

```bash
mongod
```

`mongod` 在没有参数的情况下会使用默认数据目录 `/data/db`，如果没有这个目录或者目录不可写，`mongod` 会启动失败，可以使用 `--dbpath` 参数指定数据目录。

默认情况下，MongoDB 监听在 `27017` 端口，可以使用 `--port` 参数指定端口。`mongod` 还会启动一个简单的 HTTP 服务器，监听在 `28017` 端口，可以使用 `--httpinterface` 参数关闭这个 HTTP 服务器。我们可以通过浏览器访问 `http://localhost:28017` 来查看服务器的状态。

### MongoDB shell

在启动 MongoDB 之后，使用 `mongo` 命令可以连接到 MongoDB 实例，进入 MongoDB shell。shell 本质上是一个功能完备的 JavaScript 解释器，可以运行任何 JavaScript 程序。此外，shell 还是一个独立的 MongoDB 客户端，可以用来管理 MongoDB 实例和操作数据。开启的时候，shell 会连接到 `test` 数据库，可以使用 `use` 命令切换到其他数据库。

### CRUD

#### 创建 - insert

首先创建一个文档：

```javascript
post = {
  title: "My blog post",
  content: "Here is my blog post.",
  date: new Date(),
};
```

之后使用 `insert` 方法将其保存到 blog 集合中：

```javascript
db.blog.insert(post);
```

在保存的时候，MongoDB 会自动为文档添加 `_id` 字段，这个字段是文档的唯一标识符，可以用来查询文档。

#### 读取 - find

`find` 会返回集合里面所有的文档，若只是想查看一个文档，可以用 `findOne` 方法：

```javascript
db.blog.findOne()
```

返回结果如下

```
{
	"_id" : ObjectId("6484ee3770ae4d11ad3fa9b4"),
	"title" : "My blog post",
	"content" : "Here is my blog post.",
	"date" : ISODate("2023-06-10T21:42:02.926Z")
}
```

在默认情况下，JSON 对象会以一个单行字符串的格式返回，如果想要将其格式化以方便查看，我们可以在 findOne 方法后面接上 pretty 方法

```javascript
db.blog.findOne().pretty();
```

在读取数据并返回的时候 Mongo 还有一个功能叫 Projection，即筛选出我们想要的 field 返回。比如，现在我们对于上面的数据，只想返回 title 和 content 两个 field 的值，我们可以这样写：

```javascript
db.blog.find(
  { _id: ObjectId("6484ee3770ae4d11ad3fa9b4") },
  { _id: 0, title: 1, content: 1 }
);
```

可以看到，find 方法接受了两个参数，它们都是 JSON object。第一参数代表 filter，这里我们想要查看 \_id 与给定 id 值相同的数据对象；第二个参数代表我们想要返回的 field，这里 0 代表不返回，1 代表返回。

#### 更新 - update

`update` 方法可以用来更新文档，它接受两个参数，第一个参数是查询条件，第二个参数是更新操作符，例如，首先为 `post` 添加一个 `comments` 字段：

```javascript
post.comments = [];
```

之后执行 update 操作，用新版本的文档替换标题为 "My Blog Post" 的文章：

```javascript
db.blog.update({ title: "My blog post" }, post);
```

#### 删除 - remove

`remove` 方法可以用来删除文档，它接受一个查询条件作为参数，删除所有符合条件的文档。在不使用参数的情况下，它会删除一个集合内的所有文档：

```javascript
db.blog.remove({ title: "My blog post" });
```

使用 `db.集合名` 的方式来访问集合一般不会有问题，但如果集合名恰好是数据库类的一个属性就有问题了，这时候可以使用 `db.getCollection` 方法来访问集合

### 数据类型

- null
- 布尔值
- 32 位整数
- 64 位整数
- 64 位浮点数 (shell 中的数字都是这种类型)
- 字符串
- 符号
- 对象 id
- 日期
- 正则表达式
- 代码
- 二进制数据
- 最大值
- 最小值
- undefined
- 数组
- 内嵌文档

由于 JavaScript 的数字类型是 64 位浮点数，所以 shell 中的数字都是这种类型。在 shell 中，可以使用 `NumberInt` 和 `NumberLong` 来创建 32 位整数和 64 位整数

#### \_id 和 ObjectId

MongoDB 会自动为每个文档添加 `_id` 字段，这个字段是文档的唯一标识符，可以用来查询文档。`_id` 字段的值可以是任意类型的，默认情况下是一个 `ObjectId` 对象，这个对象是由 12 个字节组成的，包含了创建文档的时间戳（0-3 字节）、客户端的机器 ID（4-6 字节）、客户端进程 ID（7-8 字节） 和随机数（9-11 字节）。

## 增删改查

### 创建文档

使用 `insert` 插入一个文档是向 MongoDB 中添加数据的最基本的方法，`insert` 的参数是一个文档或者文档的数组，例如：

```javascript
db.foo.insert({ bar: "baz" });
```

如果插入的文档没有 `_id` 字段，MongoDB 会自动为文档添加一个 `_id` 字段，这个字段的值是一个 `ObjectId` 类型的值，这个值是 MongoDB 生成的一个 12 字节的唯一标识符，这个值在集合中是唯一的。

如果要插入多个文档，使用批量插入会更快一些，因为一次批量插入只是单个的 TCP 请求，避免了许多零碎的请求所带来的开销。

当执行插入的时候，使用的驱动程序会将数据转换成 `BSON` 形式，然后将其送入数据库。数据库解析 `BSON`，检验其是否包含 `_id` 键并且文档不超过 4MB，除此之外不做别的数据验证。

### 删除文档

删除文档使用 `remove` 方法，`remove` 方法的参数是一个查询文档，例如：

```javascript
db.foo.remove({ bar: "baz" });
```

如果不加参数，`remove` 方法会删除集合中的所有文档，但是不会删除集合本身，原有的索引也会保留。删除数据是永久性的，不能撤销也不可恢复。

删除文档通常会很快，但是要清除整个集合，直接删除集合会更快一些。

### 更新文档

更新文档使用 `update` 方法，`update` 方法的参数是一个查询文档和一个要将匹配到的文档更新成的文档，例如：

```javascript
db.foo.update({ bar: "baz" }, { bar: "baz", zzz: "xxx" });
```

在这种情况下，`update` 方法会将匹配到的文档替换成新的文档，如果不想替换整个文档，可以使用 `$set` 操作符（update operator），例如：

```javascript
db.foo.update({ bar: "baz" }, { $set: { bar: "baz", zzz: "xxx" } });
```

`$set` 操作符会将指定的字段添加到文档中，如果文档中已经存在这个字段，那么就会更新这个字段的值。

需要注意的一点是，如果只想更新文档中的一部分，那么一定要使用更新操作符，否则会将整个文档替换掉。

这里我们可以看到，`update` 方法是比较危险的一个方法，稍有不慎就会把所有满足 filter 条件的原始数据给全部替换掉而不是单纯的更新其中一个部分。为了避免这种情况，我们可以使用 `updateOne` 和 `updateMany` 方法。在使用这两个方法的时候我们只能通过传入 update operators 来进行更新操作，不能单纯的传入一个 JSON object。例如：

```javascript
// 这里只会更新一条满足 filter 条件的数据
db.foo.updateOne({ bar: "baz" }, { $set: { bar: "baz", zzz: "xxx" } });

// 这里会更新所有满足 filter 条件的数据
db.foo.updateMany({ bar: "baz" }, { $set: { bar: "baz", zzz: "xxx" } });
```

此外，如果我们想要真的执行替换操作，可以使用 `replaceOne` 方法，其写法与 `update` 方法类似，但更加安全，因为它只会替换一个满足条件的数据：

```javascript
// 这里会将满足 filter 条件的数据中的一条替换成后面的 JSON object
db.foo.replaceOne({ bar: "baz" }, { bar: "baz", zzz: "xxx" });
```

对于更多相关的更新操作符，可以参考官方文档中的 [update operators](https://docs.mongodb.com/manual/reference/operator/update/) 部分。

## 关系

> 参考：[udemy](https://www.udemy.com/course/best-mongodb/)

### 一对一关系

假设我们有两个 collection，一个来存储 people，一个来存储 address，这两个 collection 中的数据具有一对一关系：

people collection 中的数据如下所示，其中 `address` 字段的值对应是是 address collection 中数据的 `id` 字段

```json
{
    "_id": 1,
    "name": "John Cart",
    "email": "jc@jc.com",
    "phone": 12345,
    "address": 1
}
```

address collection 中的数据如下所示：

```json
{
    "_id": 1,
    "city": "Shanghai",
    "street": "aaa",
    "building": 1,
    "room": 1021
}
```

这样存储的话会有一个问题，就是我们想要获取 people collection 中的数据的话，要经过两次查询，第一次查 people，第二次查 address，效率比较低。为此，我们的解决办法是，将它们写在同一个 collection 内，新的数据格式如下：

```json
{
    "_id": 1,
    "name": "John Cart",
    "email": "jc@jc.com",
    "phone": 12345,
    "address": {
        "city": "Shanghai",
        "street": "aaa",
        "building": 1,
        "room": 1021
    }
}
```

这样我们只需要一次查询就可以获取所需信息，也可以通过 projection 来筛选返回的 field，非常方便。

### 一对多关系

一对多关系也是我们生活中很常见的一种关系，假如我们想要表示一个人有多个订单，那么首先我们可以采用如下这种 schema：

```json
{
    "_id": 1,
    "name": "John Cart",
    "email": "jc@jc.com",
    "phone": 12345,
    "orders": [
        {
            "date": "2018-12-10",
            "product": "book",
            "cost": 19.99
        },
        {
            "date": "2018-12-13",
            "product": "book",
            "cost": 39.99
        },
        {
            "date": "2018-12-12",
            "product": "computer",
            "cost": 2899
        }
    ]
}
```

上面的这种存储方式可以作为一种选择，但它有一些问题：

- 如果我们想要对所有订单进行处理的话，这种存储方式会比较困难，因为不同的人可能会有相同的订单，需要考虑去重
- Mongo DB 中单个 document 的大小限制是 16 M，如果订单数量较多的话可能会使 document 体积过大

为了解决上述两个问题，我们也可以把用户信息和订单信息拆分成两个 collection，使用 `id` 字段作为链接

**people collection**

```json
{
    "_id": 1,
    "name": "John Cart",
    "email": "jc@jc.com",
    "phone": 12345,
    "orders": [1, 2, 3]
}

```

**order collection**

```json
[
    {
        "_id": 1,
        "date": "2018-12-10",
        "product": "book",
        "cost": 19.99
    },
    {
        "_id": 2,
        "date": "2018-12-13",
        "product": "book",
        "cost": 39.99
    },
    {
        "_id": 3,
        "date": "2018-12-12",
        "product": "computer",
        "cost": 2899
    }
]
```

可以看到，我们将用户信息和订单信息分别存入了两个 collection 中，在用户信息中有一个 `orders` 字段，它的格式是一个 array，存储了所有该用户名下的订单的 `id`

### 多对多

以下面的订单关系为例，顾客和商品之间是多对多关系，我们采用的策略是通过一张中间表来表示这种关系，这里是订单表。如果选择将订单信息全部存入顾客表中，则可扩展性太差。

![](/images/mongo/many-to-many.png)