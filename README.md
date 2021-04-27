# mongodb学习

## 1. mongodb介绍

* 特点：数据量大、数据价值低
* 应用场景：评论

## 2. 启动mongodb

* 启动服务器： `mongod`
* `mongod --port` 新的端口号（默认端口：27017）
* 设置mongodb数据库的存储路径：`mongod --dbpath 路径`
* 连接mongodb数据库: `mongo`

## 3. 三大基本概念

* 数据库 database
* 集合(数组) collection,类似与SQL中的数据表，本质上是一个**数组**，里面包含看多个文档对象，[{},{},{}]
* 文档对象 document, 类似与SQL中的记录，一个文档**对象**{}就是一条记录
* 一个数据库由多个集合构成，一个集合包含多个文档对象

## 4. 基本使用

* 查看所有数据库：show dbs 或者 show databases
* 切换指定数据库：use xxx
* 查看当前数据库：db
* 查看当前数据库的所有集合：show collections

## 5. CURD

### 1. 插入文档`db.<collection>.insert(doc)`

- 插入的文档如果没有手动提供_id属性，则会自动创建一个(确保数据唯一性)

```mysql
# 插入一个数据
db.<collection>.insert({name: "xxx", age: 28, gender:"xxx"});
# 插入多个
db.<collection>.insert([{}, {}, {}])
```

* `insertOne`和`insertMany`

```mysql
# 插入一个数据
db.<collection>.insertOne({name: "xxx"})

# 插入多个数据
db.<collection>.insertMany([{},{},{}])
```

* 批量插入多条数据

```mysql
# 向numbers插入20000条数据

# 方式一， 耗时长，不推  (20.649s)
for(var i = 1; i <= 20000; i++) {
	db.numbers.insert({num: i})
}
db.numbers.count()
db.numbers.remove({}) //(0.634s)

# 方式二， 推荐方式 (0.634s)
var arr = []
for(var i = 1; i <= 20000; i++) {
	arr.push({num: i})
}
db.numbers.insert(arr)
```



### 2. 查询 `db.collection.find()`

* `find()`查询所有符合条件的文档
  1. `()`查看集合下的所有文档
  2. `{字段名: 值}`查询属性是指定的文档
  3. `find`返回的是一个数组

```mysql
# 查看所有集合下文档
db.<collection>.find()

# 查看集合下_id为hello的文档
db.<collection>.find({_id: "hello"})

# 插卡集合下_id为hello，age为18的文档
db.<collection>.find({_id: "hello", age:18})

# 查询集合下喜欢电影名为hero的文档
# 内嵌文档需要加引号
db.user.find({"hooby.movies":"hero"})
```

* `db.<collection>.findOne()`
  1.  返回集合下的一个对象
* `db.<collection>.find().count()`
  1. 返回集合下对象的个数
* 范围查询`$gt\$lt\$gte\$lte...`

```mysql
# 大于 $gt   大于等于 $gte (小于， 小于等于同理)
# 查询numbsers中的num大于500的文档
db.numbers.find({num: {$gt:500}})

# 查询numbsers中的num大于等于500的文档
db.numbers.find({num: {$gte:500}})

# 大于 40 且 小于 50
db.numbers.find({num:{$gt:40, $lt: 50}})
```

* 限制查询`skip()、limit()`
  1. `mongodb` 会自动调整`skip`和`limit`的位置
  2. `skip(页码-1 * 每页显示的条数).limit(每页显示的条数)`

```mysql
# 查看numbers集合中的10条数据
db.numbers.find().limit(10);

# 查看numbers集合中的11条到20条数据
db.numbers.find().skip(10).limit(10);
```

### 3. 修改 `db.<collection>.update(condiction,newDocument)`

* `db.<collection>.update()`
  1. `update()`默认情况下会将新对象替换为旧对象
  2. `$set`修改文档中的指定属性
  3. `$unset` 删除文档中的指定属性
  4. `$push`向数组中添加一个新的元素（不会考虑元素重复）
  5. `$addToSet`向数组中添加一个新元素（考虑元素重复）
  6. `update()`默认只修改一个
  7. `updateMany`、`updateOne`、`replaceOne`的综合

```mysql
# 替换， 有风险
db.<collection>.update({name:"沙和尚"}, {age:28})

# 修改文档中的指定属性一个
db.<collection>.update({”_id“: "hello"},{$set:{gender:"男"}})

# 修改文档的指定属性多个
db.<collection>.update({”_id“: "hello"},{$set:{gender:"男"}}, {mult: true})

# 14. 向tangseng中爱好中添加一个电影
db.<collection>.update({username: "tangseng"}, {$addToSet:{"hooby.movies":"TInsetall"}})

```

* `db.collection.updateMany()`
  1. `updateMany()`默认会修改多个
* `db.collection.updateOne()`
  1. `updateOne()`默认会修改一个
* `db.collection.replaceOne()`
  1. `replaceOne()`替换一个文档

### 4. 删除`db.<colletion>.remove()`

* `db.<collection>.remove`
  1. 默认情况下删除多个
  2. 可以根据条件方式删除

```mysql
# 删除id为hello的文档对象（删除多个）
db.<collection>.remove({_id: "hello"})

# 删除id为hello的文档对象（删除一个）
db.<collection>.remove({_id: "hello"}, true)

# 删除一个文档中的属性(删除文档中的address属性)
db.user.remove({username:"zhubajie"}, {$unset:{address:"123"}})

# 清空集合(性能较差)
db.<collection>.remove({})
```

* `db.<collection>.drop()` 删除集合
* `db.<collection>.deleteOne()`
* `db.<collection>.deleteMany()`
* `db.dropDatabase()` 删除数据库

## 5. 文档之间的关系

* 一对一的关系

```mysql
# 内嵌文档体现 1 对 1 关系
db.wifeAndHusband.insert(
	{
    	name:"黄蓉",
    	husband: {
    		name: "郭靖"
    	}
    },
    {
    	name: "潘金莲",
    	husband: {
    		name:"武太郎"
    	}
    }
)
```

* 一对多的关系

```mysql
# 应用场景： 
# （用户-订单）、（文章-评论）、（父母-孩子）

# 内嵌文档映射一对多的关系
 
```



* 多对多的关系
