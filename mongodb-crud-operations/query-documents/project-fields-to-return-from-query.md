# 指定查询返回的字段

默认情况下，MongoDB的查询语句返回匹配到文档的所有字段，为了限制MongoDB返回给应用的数据，可以通过[projection](https://docs.mongodb.com/v4.0/reference/glossary/#term-projection)文档来指定或限制返回的字段。

本文提供了使用mongo shell中[db.collection.find()](https://docs.mongodb.com/v4.0/reference/method/db.collection.find/#db.collection.find)方法映射查询的案例。案例中使用的**inventory**集合数据可以通过下面的语句产生。

```javascript
db.inventory.insertMany( [
  { item: "journal", status: "A", size: { h: 14, w: 21, uom: "cm" }, instock: [ { warehouse: "A", qty: 5 } ] },
  { item: "notebook", status: "A",  size: { h: 8.5, w: 11, uom: "in" }, instock: [ { warehouse: "C", qty: 5 } ] },
  { item: "paper", status: "D", size: { h: 8.5, w: 11, uom: "in" }, instock: [ { warehouse: "A", qty: 60 } ] },
  { item: "planner", status: "D", size: { h: 22.85, w: 30, uom: "cm" }, instock: [ { warehouse: "A", qty: 40 } ] },
  { item: "postcard", status: "A", size: { h: 10, w: 15.25, uom: "cm" }, instock: [ { warehouse: "B", qty: 15 }, { warehouse: "C", qty: 35 } ] }
]);
```



## 返回匹配文档中的所有字段

如果没有特别指定[projection](https://docs.mongodb.com/manual/reference/glossary/#std-term-projection), [db.collection.find()](https://docs.mongodb.com/v4.0/reference/method/db.collection.find/#db.collection.find)方法将会返回匹配文档的所有字段。



下面的案例返回**inventory**集合中**status**等于**"A"**的文档的所有字段。

```javascript
db.inventory.find( { status: "A" } )
```



上述操作等价于下面的标准SQL:

```javascript
SELECT * from inventory WHERE status = "A"
```



## 仅返回指定字段和_id字段

映射会返回在映射文档中显示设置<field>为**1**的字段。

下面的案例返回所有检索到文档中**item, status, _id**三个字段。

```javascript
db.inventory.find( { status: "A" }, { item: 1, status: 1 } )
```



上述操作等价于下面的标准SQL:

```javascript
SELECT _id, item, status from inventory WHERE status = "A"
```



## 去除_id字段

可以通过在映射文档中将**_id**字段设置为**0**来从结果集中去除 **_id**字段，就像下面的例子:

```javascript
db.inventory.find( { status: "A" }, { item: 1, status: 1, _id: 0 } )
```



上述操作等价于下面的标准SQL:

```javascript
SELECT item, status from inventory WHERE status = "A"
```

><font color=Green>Note:</font>
>
>除_id字段外，不能在映射文档中同时使用包含和去除语句。



## 去除指定字段

可以使用映射来排除特定字段，而不是在匹配文档中列出要返回的字段。

下面的案例返回匹配文档中除**status** 和 **instock** 字段之外的所有字段：

```javascript
db.inventory.find( { status: "A" }, { status: 0, instock: 0 } )
```

><font color=Green>Note:</font>
>
>除_id字段外，不能在映射文档中同时使用包含和去除语句。



## 返回嵌套文档中的指定字段

通过[点号](https://docs.mongodb.com/v4.0/core/document/#document-dot-notation)引用嵌套文档字段并且在映射文档中将该字段设置为**1**来实现返回嵌套文档中的指定字段。

下面的案例返回

* **_id**字段(默认返回)
* **item**字段
* **status**字段
* 文档**size**中的**uom**字段

**uom**字段是**size**嵌套文档中的字段.

```javascript
db.inventory.find(
   { status: "A" },
   { item: 1, status: 1, "size.uom": 1 }
)
```



## 去除嵌套文档中的指定字段

通过[点号](https://docs.mongodb.com/v4.0/core/document/#document-dot-notation)引用嵌套文档字段并且在映射文档中将该字段设置为**0**来实现去除嵌套文档中的指定字段。

下面的案例返回匹配文档中除嵌套文档**size**中的**uom**字段外的所有字段。

```javascript
db.inventory.find(
   { status: "A" },
   { "size.uom": 0 }
)
```



## 映射数组中的嵌套文档的指定字段

通过使用[点号](https://docs.mongodb.com/v4.0/core/document/#document-dot-notation)来映射数组中嵌套文档的指定字段。

下面案例返回:

* **_id**字段(默认返回)
* **item**字段
* **status**字段
* 数组字段**instock**中的嵌套文档中的**qty**字段

```javascript
db.inventory.find( { status: "A" }, { item: 1, status: 1, "instock.qty": 1 } )
```



## 映射返回数组中指定的数组元素

对于数组字段，MongoDB 提供了以下用于操作数组的映射运算符:[$elemMatch](https://docs.mongodb.com/v4.0/reference/operator/projection/elemMatch/#proj._S_elemMatch),[$slice](https://docs.mongodb.com/v4.0/reference/operator/projection/slice/#proj._S_slice),[$](https://docs.mongodb.com/v4.0/reference/operator/projection/positional/#proj._S_)

下面的案例使用[$slice](https://docs.mongodb.com/manual/reference/operator/projection/slice/#mongodb-projection-proj.-slice)映射操作符返回数组字段**instock**中最后的元素:

```javascript
db.inventory.find( { status: "A" }, { item: 1, status: 1, instock: { $slice: -1 } } )
```

[$elemMatch](https://docs.mongodb.com/manual/reference/operator/projection/elemMatch/#mongodb-projection-proj.-elemMatch),[$slice](https://docs.mongodb.com/manual/reference/operator/projection/slice/#mongodb-projection-proj.-slice),[$](https://docs.mongodb.com/manual/reference/operator/projection/positional/#mongodb-projection-proj.-)是将指定元素映射到返回数组中的唯一方法。

举个例子，不能使用数组下标来映射指定的数组元素。例如:**{ "instock.0": 1 }**映射不会用第一个元素来映射数组。



参考:

[Query Documents](https://docs.mongodb.com/v4.0/tutorial/query-documents/)

3323

原文链接：https://docs.mongodb.com/manual/tutorial/project-fields-from-query-results/

译者：张芷嘉
