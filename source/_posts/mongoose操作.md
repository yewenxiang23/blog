---
title: mongoose操作
date: 2018-09-24 23:49:27
categories: node
tags:
  - mongoose
---

### 简单的 mongoose 示例

```js
const mongoose = require('mongoose')
mongoose.connect("mongodb://localhost:27017/study", {useNewUrlParser:true}, function(err){
  if(err){
    console.log('Connection Error:' + err)
  }else{
    console.log('Connection success!')
  }
})

const UserSchema = new mongoose.Schema({
  nickname:{
    type:String,
    default: 'new user'   //在实例化 model 时，如果不传参则为默认值
  }
})

const User = mongoose.model('User', UserSchema)

const user = new User({
  nickname:'ye'
})

// 保存到数据库
user.save(err => {
  if(err){
    return console.log(err)
  }
  console.log(user.nickname) //打印保存后的值
})

```

### 预定义修饰符  自定义：set修饰符 get修饰符

> model 第二个参数会被直接传给 new mongoose.Schema

```js
const User = mongoose.model('User', {
  nickname:{
    type:String,
    trim: true     //预定义修饰符
  },
  blog:{
    type:String,
    set:v => {      //自定义：set修饰符, new modal 传入的参数时执行
      if(!v) return v;
      if(0 !== v.indexOf('http://') && 0 !== v.indexOf('https://')){
        v = `http://${v}`
        return v
      }
    },
    get:v => {      //自定义：get修饰符, 数据存入数据库之后，取值时执行
      if(!v) return v;
      if(0 !== v.indexOf('http://') && 0 !== v.indexOf('https://')){
        v = `http://${v}`
        return v
      }
    }
  }
})
```

### 虚拟属性

有时我们不需要把值存取到数据库，可以通过其他的属性值计算，得到我们需要的值，这时可以利用到虚拟属性。

```js
const PersonSchema = new mongoose.Schema({
  firstName:String,
  lastName:String,
})

PersonSchema.virtual('fullName').get(function(){
  return this.firstName + this.lastName
})
var Person = mongoose.model('Person', PersonSchema)

var person = new Person({
  firstName:'ye',
  lastName:'wenxiang'
})
console.log(person.fullName)  //yewenxiang
```

### 索引

- 唯一索引：检查是否唯一
- 辅助索引：增加查询速度

```js
const BookSchema = new mongoose.Schema({
  isbn:{
    type:Number,
    unique: true //唯一索引
  },
  name:{
    type: String,
    index: true, //辅助索引
  }
})
```

### 模型的方法

```js
// 自定义静态方法，在 BookSchema.findIsbn 上调用
BookSchema.statics.findIsbn = function(isbn, cb){
  this.findOne({isbn}, (err, doc)=>{
    cb(err, doc)
  })
}

//自定义实例方法  在每个 new 出来的示例上调用
BookSchema.methods.print = function(){
  console.log(this.name)
  console.log(this.isbn)
}

```

### 数据校验

数据在保存时检测是否符合规则。

预定义验证器

```js
const orderSchema = new mongoose.Schema({
  count:{
    type:Number,
    required: true,   //必须传值 才能保存
    max:1000,
    min:10,
  },
  status:{
    type:String,
    enum:['created', 'success', 'failed']  //只能三个中选一个 才能保存
  },
  dec:{
    type:String,
    match: /Book/g,   //正则验证，字符串中存在Book 才能保存
  }
})
```

自定义验证器

```js
const orderSchema = new mongoose.Schema({
  desc: {
    type:String,
    validate:(v) => {
      return desc.length >= 10
    }
  }
})
```

### 中间件

- 文档中间件：init/validate/save/remove
- 查询中间件：count/find/findOne/findOneAndRemove/findOneAndUpdate/update

### 集合交叉引用

```js
const mongoose = require('mongoose');
const Schema = mongoose.Schema
const {ObjectId, Mixed} = Schema.Types
mongoose.connect("mongodb://localhost:27017/news", {useNewUrlParser:true}, function(err){
  if(err){
    console.log('Connection Error:' + err)
  }else{ 
    console.log('Connection success!')
  }
})

const User = mongoose.model('User', {
  user:String
})
const Artical = mongoose.model('Artical', {
  title:String,
  author:{
    type:ObjectId,
    ref:'User'
  }
})
const user = new User({
  user:'ye'
})

const artical = new Artical({
  title: '标题',
  author: user
})

user.save(err=>{
  if(err) return console.log(err)
  console.log('user save success')
  artical.save(err => {
    if(err) return console.log(err)
    console.log('artical save success')
    Artical.findOne({}).populate('author').exec(function(err, doc){
      console.log(doc)
    })
  })
  
})
```
