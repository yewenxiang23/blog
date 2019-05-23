---
title: mongoDB操作
date: 2018-09-24 23:50:17
categories: node
tags:
  - mongoDB
---

### 创建

```bash
> post = {"title": "my blog post", "content":"here's my blog post", "date": new Date()}
> db.blog.insert(post)
```
 
javascript shell 中， `db` 命令现当前所在的数据库，上面代码，在当前数据库中的 `blog`集合 插入 `post`文档。
可以使用 `db.blog.find()` 查找 `blog` 集合中保存的所有文档。

### 读取

find(): 会返回集合里面所有的文档，若只想查看一个文档可以使用 findOne

```bash
> db.blog.findOne()
```

### 更新

update: 接收至少两个参数:

- 第一个是更新文档的限定条件
- 第二个是新文档
- 第三个参数为true时，表示开启 upsert 更新模式，要是没有文档符合更新条件，就会以这个条件和更新文档为基础创建一个新文档，如果找到则正常更新。
- 第四个参数为true时，表示开启多文档更新，默认为false，只能匹配并更新一个。

比如给上面的文档增加一个 `comments` 字段，值为[]

```
> post.comments = []
> db.blog.update({title:"my blog post"}, post)
```
**例子**：删除数据库集合中某个字段

```
db.User.update({},{$unset:{'address':''}},false, true)
```


**使用修改器**
利用原子的 `更新修改器`，使得这部分修改极为高效。
比如页面访问统计

```js
{
    "_id" : ObjectId("5b430d4da1d1ea0fd260cae5"),
    "url" : "www.baidu.com",
    "pageviews" : 52
}
```

```bash
> db.blog.update({"url":"www.baidu.com"}, {"$inc":{"pageviews":1}})
# 会将上面的pageviews + 1，
```

> 如果update 第一个参数匹配到多个文档，只会更新第一个,所以一般选择ID作为条件

##### "$set"修饰器

用来指定一个键的值，如果这个键不存在则创建。（更新键的值，或者数据类型。）

```bash
> db.blog.update({"_id":ObjectId("5b431136a1d1ea0fd260cc3e")}, {"$set":{"favorite book": "war and peace"}})
```

可以修改类型:

```bash
> db.blog.update({"_id":ObjectId("5b431136a1d1ea0fd260cc3e")}, {"$set":{"favorite book": ["cat", "dog"]}})
```

修改嵌套文档

![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1xtlf4syj30io05cq39.jpg)


##### "$unset"修饰器

将一个键完全删除

```bash
> db.blog.update({"_id":ObjectId("5b431136a1d1ea0fd260cc3e")}, {"unset": {"favorite book": 1}})
```

##### "$inc"修饰器

用来增加已有键的值，或者在键不存在时创建一个键。

```bash
> db.blog.update({"game":"pinball","user":"joe"},{"$inc":{"score":50}})
# 每次增加50,可以为负值
```
"$inc" 专门用来增加和减少数字的，只能用于整数、长整数或双精度浮点数

##### 数组修改器

"$push" 会向已有的数组末尾加入一个元素，要是没有就创建一个新的数组。

**情景**：值不在数组中时，把它加进去，可以在查询文档中使用 "$ne" 来实现。

```bash
> db.blog.update({"authords cited": {"$ne": "Richie"}, {"$push": {"authords cited": "Richie"}}})
```

也可以使用 "$addToSet" 完成同样的事情,向数组添加值，避免重复。

"$addToSet" 可以和 "$each" 结合使用，一次添加多个不同的值

```bash
> db.blog.update({"_id": ObjectId("5b431136a1d1ea0fd260cc3e")}, {
  "$addToSet":{
    "emails" :{
      "$each" : ["dsa@qq.com", "dsd3aa@qq.com", "dsd2a@qq.com"]
    }
  }}
)
```

##### "$pop"修饰器
```js
{$pop: {key:1}}  //从数组末尾删除一个元素, key:-1 从头部删除
```

##### "$pull"修饰器

"$pull" 可以特定条件来删除元素，不仅仅是依据位置。

```bash
> db.lists.insert({"todo": ["dishes", "laundry", "dry cleaning"]})
> db.lists.update({}, {"$pull": {"todo":"laundry"}})
# 会删除数组中的laundry ,如果有多个，会全部删除。
```

##### 数组的定位修改器

假设文档中 comments 字段是一个数组，里面包含多个对象

```bash
> db.blog.update(..., {"$inc": {"comments.0.votes":1}})
#增加第一个评论的投票数量

> db.blog.update({"comment.author": "John"}, {"$set": {"comments.$.author": "Jim"}})
# $占位符，查询到评论中 作者名为John 的下标，更新为 Jim
```

##### save函数

save是一个shell函数,可以在文档不存在时插入，存在时更新，它只有一个参数：文档

> 如果这个文档含有"_id" 键，save会调用 upsert。否则会调用插入。


```bash
var x = db.foo.findOne()
x.num = 42

db.foo.save(x)
# 免去了 update 的查询条件
```


### 删除

remove: 从数据库永久删除文档，**在传空对象调用，会删除一个集合内所有的文档**

```
> db.blog.remove({title: "my blog post"})
```

### shell技巧

```bash
> db.help()
# 查看数据库级别的入门命令

> db.foo.help()
# 查看集合相关命令
```
