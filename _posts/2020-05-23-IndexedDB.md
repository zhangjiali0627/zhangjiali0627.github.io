---
layout: post
categories: JavaScript
title: "本地存储IndexedDB简介"
# subtitle: "一个东西是否有生命力，看的不是其是否强大，而是看其是否足够简单."
featured-image: /images/2020-05-23/indexedDB.png
tags: ['本地存储']
date-string: MAY 23, 2020
---

在前面工作的过程中，项目上需要实现断点续传功能，要求前端在上传的过程中把多个文件的相关信息存储在前端，经过一番查找资料，最后找到了indexedDB，并应用在了项目中。个人感觉indexDB还是挺强大的，而且它的各种API都是异步执行。对于存储大量结构化数据事非常有效，值得我们去学习。后面我们也可以根据项目实际需要选择性使用localForage或indexedDB

## indexedDB诞生背景

随着浏览器的功能不断增强，越来越多的网站开始考虑，将大量数据储存在客户端，这样可以减少从服务器获取数据，直接从本地获取数据。

也许熟悉前端本地存储的我们会有疑惑，不是已经有了webStorage和cookie了吗？为什么还要推出indexedDB呢？目前的浏览器数据存储方案，都不适合存储大量的数据：
- cookie的存储大小为4KB，并且每次发送请求都会携带在请求头中提交给服务器，如果存储一些非必要的数据，会很浪费带框。
- LocalStorage的存储空间为2.5MB-10MB之间，而且不提供搜索功能，不能建立自定义索引。
这其实就是indexDB诞生的背景。

## indexedDB简介

`IndexedDB` 是浏览器提供给我们的本地数据库，它可以通过脚本创建和操作。IndexedDB 允许储存大量数据，提供查找接口，还能建立自定义索引。这些都是 LocalStorage 所不具备的。

就数据库类型而言，有两种不同类型的数据库，就是关系型数据库和非关系型数据库：
- 关系型数据库如Mysql、Oracle等将数据存储在表中；
- 非关系型数据库如Redis、MongoDB等将数据集作为个体对象存储。
IndexedDB 不属于关系型数据库（不支持 SQL 查询语句），更接近 NoSQL 数据库。数据形式使用的是json。

## indexedDB的特点：

1. 键值对存储【对象仓库】：IndexedDB采用objectStore（对象仓库）存放数据。所有类型的数据都可以直接存入，包括js对象。对象仓库中，数据以“键值对”的形式保存，每一条数据记录都有对应的主键，主键是独一无二的，不能有重复，否则会抛错。

2. 异步：IndexedDB操作时是异步执行的，不会锁死浏览器，用户仍然可以进行其他操作，这个就与localStorage形成了鲜明对比。后者的操作是同步的，在存储对象时会拖慢整个网页的响应速度。异步设计是为了防止大量数据的读写，拖慢网页；

3. 支持事务：IndexedDB的所有操作都是支持事务的（transaction），这意味着一系列操作步骤之中，只要有一步失败，整个事务就会取消，数据库就会回滚到事务发生之前的状态，不存在只修改一部分数据的情况。

4. 同源限制：所有的本地存储都受到同源限制。网页只能访问自身域名下的数据库。

5. 存储空间大：IndexedDB的存储空间比localStorage大得多，一般来说至少250M，硬盘的一半；

6. 支持二进制存储：IndexedDB不仅可以存储字符串，还可以存储二进制数据（ArrayBuffer对象、Blob对象）

## IndexedDB的基本概念

IndexedDB是一个比较复杂的API，涉及到很多概念：

- 数据库：IDBDatabase 对象
- 对象仓库： IDBObjectStore 对象（相当于关系型数据库中的表格）
- 索引： IDBIndex 对象
- 事务： IDBTransaction 对象（数据记录的读写和删改都是通过事务完成的，事务对象提供error、abort、complete三个事件，用来监听操作结果）
- 操作请求： IDBRequest 对象
- 指针： IDBCursor 对象
- 主键集合： IDBKeyRange 对象

## IndexedDB的操作流程

### 基本模式

- 打开数据库并开始一个事务；
- 创建一个对象仓库；
- 构建一个请求来执行数据库的操作：增删改查；
- 监听操作是否完成；
- 在操作结果上进行一些操作（可以在request对象中找到）

### 常用的操作步骤

1. 首先要先判断浏览器是否支持IndexedDB:

    ```js
      let indexedDB = window.indexedDB || window.webkitIndexedDB || window.mozIndexedDB || window.msIndexedDB;
      if(!indexedDB)
        {
          console.log("你的浏览器不支持IndexedDB");
        }
    ```

2. 创建请求打开IndexedDB：使用indexedDB.open()方法

    ```js
    /**
    * 第一个参数是数据库的名称，如果指定的数据库不存在，就会新建数据库
    * 第二个参数是数据库的版本，如果省略，打开已有的数据库时，默认为当前版本；新建数据库时，默认为1；
    */
      let request = indexedDB.open(name, version);
    ```
    indexedDB.open()方法返回一个IDBRequest对象。这个对象通过error、success、upgradeneeded处理打开数据库的操作结果：

    - error表示打开数据库失败
    ```js
      request.onerror = () => {
        db = request.result;
      };
    ```

    - success表示打开数据库成功
    ```js
      request.onsuccess = () => {
        db = request.result;
      };
    ```

    - upgradeneeded在数据库升级的时候调用（数据库初始化或者指定的版本号>数据库的实际版本号）
    ```js
      request.onupgradeneeded = (event) => {
        db = event.target.result;
      };
    ```

3. 新建数据库

新建数据库与打开数据库是同一个操作。如果指定的数据库不存在，就会新建。不同之处在于，后续的操作主要在upgradeneeded事件的监听函数里面完成，因为这时版本从无到有，所以会触发这个事件。

- 在创建对象仓库之前，需要先判断一下，这张表是否存在，如果不存在再新建.createObjectStore()
- 新建对象仓库之后，可以新建索引：IDBObject.createIndex()的三个参数分别为索引名称、索引所在的属性、配置对象（说明该属性是否包含重复的值）

```js
  request.onupgradeneeded = (event) => {
    db = event.target.result;
    if (!db.objectStoreNames.contains('file')) { // 判断对象仓库是否存在
      objectStoreStart = db.createObjectStore('file', { keyPath: 'key' }); // 创建对象仓库
      objectStoreStart.createIndex('key', 'key', { unique: true });  // 新建索引 ——指定主键
      objectStoreStart.createIndex('md5Code', 'md5Code', { unique: false });
    }
  };
```

- 如果数据记录里面没有合适作为主键的属性，那么可以让 IndexedDB 自动生成主键。

```js
  // 指定主键为一个递增的整数。
  let objectStore = db.createObjectStore('person', { autoIncrement: true });
```

4. 新增数据

- 是指向对象仓库写入数据记录，需要通过事务完成。
- IDBObjectStore.add()方法用于写入记录。

```js
  addFileDB = (addFileInfo) => {
    request = db.transaction(['file'], 'readwrite')
      .objectStore('file')
      .add({ ...addFileInfo });

    request.onsuccess = (event) => {
      console.log('数据写入成功');
    };

    request.onerror = (event) => {
      console.log('数据写入失败');
    }
  }
```

上面代码中，写入数据需要新建一个事务。新建时必须指定**表格名称**和**操作模式**（"只读"或"读写"）。新建事务以后，通过IDBTransaction.objectStore(name)方法，拿到 IDBObjectStore 对象，再通过表格对象的add()方法，向表格写入一条记录。

写入操作是一个**异步**操作，通过监听连接对象的success事件和error事件，了解是否写入成功。

5. 删除数据

- IDBObjectStore.delete()方法用于删除记录。

```js
  removeFileDB = (key) => {
    request = db.transaction(['file'], 'readwrite')
      .objectStore('file')
      .delete({ key });

    request.onsuccess = (event) => {
      console.log('数据删除成功');
    };

    request.onerror = (event) => {
      console.log('数据删除失败');
    }
  }
```

6. 更新数据

- IDBObject.put()方法用于更新数据

```js
  updateFileDB = (updateFileInfo) => {
    request = db.transaction(['file'], 'readwrite')
      .objectStore('file')
      .put({ ...updateFileInfo });

    request.onsuccess = (event) => {
      console.log('数据删除成功');
    };

    request.onerror = (event) => {
      console.log('数据删除失败');
    }
  }
```

7. 查询数据

### 读取数据
- objectStore.get()方法用于读取数据，参数是主键的值。

### 使用索引
- 索引的意义在于，可以让你搜索任意字段，也就是说从任意字段拿到数据记录。如果不建立索引，默认只能搜索主键（即从主键取值）。

```js
  searchData = (key) => {
    return new Promise(function (resolve) {
      request = db.transaction(['file'], 'readonly')
        .objectStore('file')
        .index('key');
      request = request.get(key);
      let result;

      request.onsuccess = (e) => {
        result = e.target.result;
        if (result) {
          resolve(result);
        }
        else {
          resolve('noHave');
        }
      };
      return result;
    });
  }
```

8. 遍历数据

- 要使用指针对象 IDBCursor，遍历数据表格的所有记录

```js
  readAll = () => {
    let objectStore = db.transaction('fileDB').objectStore('file');

    objectStore.openCursor().onsuccess = (event) => {
      let cursor = event.target.result;

      if (cursor) {
        console.log('Id: ' + cursor.key);
        console.log('Name: ' + cursor.value.name);
        console.log('Age: ' + cursor.value.age);
        console.log('Email: ' + cursor.value.email);
        cursor.continue();
      } else {
        console.log('没有更多数据了！');
      }
    };
  }
```
新建指针对象的openCursor()方法是一个异步操作，所以要监听success事件。

9. 关闭与删除数据库

- 关闭数据库可以直接调用数据库对象的close方法
```js
  db.close();
```

- 删除数据库使用数据库对象的deleteDatabase方法
```js
  indexedDB.deleteDatabase(name);
```

## 总结
以上就是indexedDB的一些基本概念以及使用，还有一些更深入的细节没有介绍，比如indexedDB的游标结合索引，发挥其真正的优势，有兴趣的小伙伴可以继续深入研究，还有就是要注意浏览器的支持问题，IE9以及更早的版本并不支持，火狐和谷歌浏览器没有问题，推荐使用。