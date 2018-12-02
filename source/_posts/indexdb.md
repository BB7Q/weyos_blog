---
title: indexDB
date: 2018-12-02
categories:
- 知识积累
tags:
- storage
- indexDB
---

# indexDB学习

## 概述

现有的浏览器数据储存方案，都不适合储存大量数据：Cookie 的大小不超过4KB，且每次请求都会发送回服务器；LocalStorage 在 2.5MB 到 10MB 之间（各家浏览器不同），而且不提供搜索功能，不能建立自定义的索引。所以，需要一种新的解决方案，这就是 IndexedDB 诞生的背景。

IndexedDB 具有以下特点。

（1）键值对储存。 IndexedDB 内部采用对象仓库（object store）存放数据。所有类型的数据都可以直接存入，包括 JavaScript 对象。对象仓库中，数据以"键值对"的形式保存，每一个数据记录都有对应的主键，主键是独一无二的，不能有重复，否则会抛出一个错误。

（2）异步。 IndexedDB 操作时不会锁死浏览器，用户依然可以进行其他操作，这与 LocalStorage 形成对比，后者的操作是同步的。异步设计是为了防止大量数据的读写，拖慢网页的表现。

（3）支持事务。 IndexedDB 支持事务（transaction），这意味着一系列操作步骤之中，只要有一步失败，整个事务就都取消，数据库回滚到事务发生之前的状态，不存在只改写一部分数据的情况。

（4）同源限制 IndexedDB 受到同源限制，每一个数据库对应创建它的域名。网页只能访问自身域名下的数据库，而不能访问跨域的数据库。

（5）储存空间大 IndexedDB 的储存空间比 LocalStorage 大得多，一般来说不少于50MB，甚至没有上限。

（6）支持二进制储存。 IndexedDB 不仅可以储存字符串，还可以储存二进制数据（ArrayBuffer 对象和 Blob 对象）。

## CAN I USE

![indexDB](/img/caniuseindexdb.png)

## 常规操作

（1）打开或新建数据库

```js
var dbName = 'firstdb';
var dbVersion = 1;
var db;
var request = window.indexedDB.open(dbName, dbVersion);
request.onerror = function() {
  console.log('open indexDB error');
}
request.onsuccess = function() {
  console.log('open indexDB success');
  db = request.result;
  console.log(db);
}
// 当数据库版本号发生变化时触发
request.onupgradeneeded = function(event) {
  console.log('data base update success');
  db = event.target.result;
  var objectStore;
  // 创建集合
  if(!db.objectStoreNames.contains('person')) {
    objectStore = db.createObjectStore('person', {keyPath: 'id'});
    objectStore.createIndex('name', 'name', {unique: false});
    objectStore.createIndex('emal', 'emal', {unique: true});
  }
  console.log(db);
}
```

（2）删除数据库或集合/表

```js
// 删除集合/表
db.deleteObjectStore('person');

// 删除数据库
indexedDB.deleteDatabase(dbName);
```

（3）添加数据

```js
function add() {
  var store = db.transaction(['person'], 'readwrite').objectStore('person');
  var request = store.add({
    id: 1,
    name: '张三',
    emal: 'zhangsan@example.com',
  });
  request.onsuccess = function() {
    console.log('写入成功');
  }
  request.onerror = function(err) {
    console.log('写入失败', err);
  }
}
```

（4）更新数据

```js
function update() {
  var store = db.transaction(['person'], 'readwrite').objectStore('person');
  var request = store.put({
    id: 1,
    name: '李四',
    emal: 'lisi@example.com',
  });
  request.onsuccess = function() {
    console.log('更新成功');
  }
  request.onerror = function(err) {
    console.log('更新失败', err);
  }
}
```

（5）根据主键读取数据

```js
function read() {
  var store = db.transaction(['person']).objectStore('person');
  var request = store.get(1);
  request.onsuccess = function(e) {
    console.log(e.target.result);
  }
  request.onerror = function(err) {
    console.log('读取失败', err);
  }
}
```

（6）根据索引读取数据

```js
function useIndex() {
  var store = db.transaction(['person'], 'readonly').objectStore('person');
  var request = store.index('name').get('张三');
  request.onsuccess = function(e) {
    console.log(e.target.result);
  }
  request.onerror = function(err) {
    console.log('读取失败', err);
  }
}
```

（7）遍历所有数据

```js
function readAll() {
  var store = db.transaction(['person'], 'readonly').objectStore('person');
  // IDBKeyRange.bound(x,y) 查找范围
  var request = store.openCursor();
  request.onsuccess = function(e) {
    var cursor = e.target.result;
    if(cursor) {
      console.log(cursor.value);
      cursor.continue()
    } else {
      console.log('no more data');
    }
  }
  request.onerror = function(err) {
    console.log('读取失败', err);
  }
}
```

`openCursor`可接收一个`IDBKeyRange`

| Range | Code |
| -- | -- |
| All keys ≤ x | `IDBKeyRange.upperBound(x)` |
| All keys < x | `IDBKeyRange.upperBound(x, true)` |
| All keys ≥ y | `IDBKeyRange.lowerBound(y)` |
| All keys > y | `IDBKeyRange.lowerBound(y, true)` |
| All keys ≥ x && ≤ y | `IDBKeyRange.bound(x, y)` |
| All keys > x &&< y | `IDBKeyRange.bound(x, y, true, true)` |
| All keys > x && ≤ y | `IDBKeyRange.bound(x, y, true, false)` |
| All keys ≥ x &&< y | `IDBKeyRange.bound(x, y, false, true)` |
| The key = z | `IDBKeyRange.only(z)` |

（8）删除数据

```js
function deleteData() {
  var store = db.transaction(['person'], 'readwrite').objectStore('person');
  var request = store.delete(1);
  request.onsuccess = function(e) {
    console.log('删除成功');
  }
  request.onerror = function(err) {
    console.log('删除失败', err);
  }
}
```

## indexDB库

* [localForage](https://github.com/localForage/localForage)（~8KB、Promise、旧版浏览器可以提供良好支持）
* [Dexie](https://dexie.org/)（~16KB、Promises、复杂查询、次要索引）
* [PunchDB](https://pouchdb.com/)（~45KB （支持自定义版本）、同步）
* [Lovefield](https://github.com/google/lovefield)（关系型）
* [LokiJS](http://lokijs.org/#/)（内存）
* [ydn-db](https://github.com/yathit/ydn-db)（与 dexie 类似、使用 WebSQL）
