---
title: NodeJS+MongoDB入门
date: 2019-03-28 16:55:31
tags: [JavaScript, NodeJS, MongoDB]
categories: 前端
---

## 安装MongoDB

mac下直接使用brew安装

`brew install mongodb`

## 启动MongoDB

可以将MongoDB的启动配置项保存在文件中，我的配置文件如下

```
dbpath=/data/db/
logpath=/data/log/mongodb.log
logappend=true
fork=true
port=27017
```

然后使用如下命令启动MongoDB

`mongod -f /data/mongo.conf `

## NodeJS中操作MongoDB

>mongoose是在node.js环境下对mongodb进行便捷操作的对象模型工具

使用mongoose，可以很便捷地操作mongodb。

## 连接数据库

### 简单连接

```javascript
/**
 * @param url(s) 数据库地址,可以是多个,以`,`隔开
 * @param options 可选,配置参数
 * @param callback 可选,回调
 */
mongoose.connect(url(s), [options], [callback])
mongoose.connect('mongodb://数据库地址(包括端口号)/数据库名称')
```
例如连接本机**27017**端口下**student**数据库

```javascript
mongoose.connect('mongodb://localhost:27017/student', { useNewUrlParser: true });
```

### 监听连接状态

```javascript
mongoose.connection.on('connected', () => {
    console.log('Mongoose default connection open to mongodb://localhost:27017/student');
});

mongoose.connection.on('error', e => {
    console.log('Mongoose default connection error ' + e);
});

mongoose.connection.on('disconnected', () => {
    console.log('Mongoose default disconnected');
});
```

## 从一个实例出发

### 需求描述

**学生**上课，一个**学生**可以选择多门课，每门课都有对应的成绩，每门课都有很多学生上。

### 需求分析

看到这个需求，想必大家都是会心一笑，想起了曾经被学生管理系统所支配的恐惧。用SQL的思路，应该马上可以创建出**学生**、**成绩**、**课程**三张表。

| studentId | studentName | age |
| ------ | ------ | ------ |
| 001 | 张三 | 20 |
| 002 | 李四 | 20 |

| courseId | courseName |
| ------ | ------ |
| 001 | 语文 |
| 002 | 数学 |

| studentId | courseId | grade |
| ------ | ------ | ------ |
| 001 | 001 | 80 |
| 001 | 002 | 72 |
| 002 | 001 | 77 |

使用MongoDB存储时，则完全不一样了。MongoDB中对应SQL中的表（table）的是集合（collection），对应表里面每一行（row）的是文档（document），每一条文档类似一个json文件。

按照上文中的例子，在MongoDB中只需要创建一个学生集合，然后每一个文档保存一个学生信息，如下所示

```json
{
  "_id": {
    "$oid": "5c9cb9e6eedbbe38450038cb"
  },
  "studentId": "001",
  "name": "张三",
  "age": 20,
  "sex": "男",
  "courses": [
    {
      "_id": {
        "$oid": "5ca5da0e3d1c1b49348c73aa"
      },
      "courseId": "001",
      "courseName": "语文",
      "grade": 80
    },
    {
      "_id": {
        "$oid": "5ca5da0e3d1c1b49348c73a9"
      },
      "courseId": "002",
      "courseName": "数学",
      "grade": 72
    }
  ],
  "__v": 0
}
```
```json
{
  "_id": {
    "$oid": "5c9ded4d3aa8134509421732"
  },
  "studentId": "002",
  "name": "李四",
  "age": 20,
  "courses": [
    {
      "_id": {
        "$oid": "5ca5da0e3d1c1b49348c73aa"
      },
      "courseId": "001",
      "courseName": "语文",
      "grade": 77
    }
  ],
  "__v": 0
}
```
很显然，一条文档中就记录了一名学生的全部信息。同时，我们可以发现，文档与文档的结构不需要完全一致，比如张三有性别信息，李四没有性别信息，张三有两门课程的成绩信息，李四只有一门课程的成绩信息。

## Schema

>Schema主要用于定义MongoDB中集合Collection里文档document的结构,可以理解为mongoose对表结构的定义(不仅仅可以定义文档的结构和属性，还可以定义文档的实例方法、静态模型方法、复合索引等)，每个schema会映射到mongodb中的一个collection，schema不具备操作数据库的能力

对于上面的例子，我们可以创建如下schema

```javascript
const mongoose = require('mongoose');
const { Schema } = mongoose;

const schema = new Schema({
    // 学号
    studentId: {
        type: String,
        index: true
    },
    // 姓名
    name: String,
    // 性别
    sex: String,
    // 年龄
    age: Number,
    courses: [{
        courseId: {
            type: String,
            index: true
        },
        courseName: String,
        grade: Number
    }]
});
```

schema中的字段支持8种类型

```
String      字符串
Number      数字
Date        日期
Buffer      二进制
Boolean     布尔值
Mixed       混合类型
ObjectId    对象ID
Array       数组
```

在schema定义之后，如果需要添加其他字段，可以使用add()方法

```javascript
schema.add({ address: 'string' });
```

## Model

>model是由Schema编译而成的假想（fancy）构造器，具有抽象属性和行为。Model的每一个实例（instance）就是一个document，document可以保存到数据库和对数据库进行操作。简单说就是model是由schema生成的模型，可以对数据库的操作。

对于上面的例子，我们可以得到如下model

```javascript
const StudentModel = mongoose.model('Student', schema);
```

## 增删改查

### 新增文档

* save()

model的每个实力都对应一个文档，新建一个model实力，然后调用save方法，即可新增一个文档。

```javascript
const student = new StudentModel({ studentId: '003', name: '王五'});
student.save();
```

* create()

直接使用model的create方法，也可以新增文档。
```javascript
StudentModel.create({ studentId: '003', name: '王五'})
```

### 删除文档

* remove()

删除学号为001的学生

```javascript
StudentModel.remove({ studentId: '001' })
```

通过model创建的实例本身也有remove方法，调用remove方法可以直接删除这条文档。

```javascript
student.remove();
```

### 修改文档

* update()

修改学号为001的学生信息，并将姓名更改为赵六

```javascript
StudentModel.update({ studentId: '001' }, { name: '赵六' })
```

### 查询文档

* find()

查询所有学生信息

```javascript
StudentModel.find();
```
查询某门课程得分高于某分数的学生
```javascript
StudentModel.find({
    'courses.courseName': courseName,
    'courses.grade': {
        $gte: grade
    }
})
```

## 自定义方法

我们可以通过schema给model添加静态方法以及示例方法方便使用。

```javascript
/**
 * 添加学生
 */
schema.statics.addStudent = async function(studentInfo = {}) {
    return await this.create(studentInfo);
}
/**
 * 获取年龄和自己相等的学生
 */
schema.methods.findSameAge = async function() {
    return await this.model('StudentModel').find({ age: this.age });
}

// 使用示例

const student = new StudentModel({ studentId: '003', name: '王五', age: 20});
student.save();

StudentModel.addStudent({ studentId: '004', name: '赵六', age: 20});
student.findSameAge();
```
