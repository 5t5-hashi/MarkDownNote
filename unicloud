## unicloud

### 云数据库

##### 集合

1. 获取集合

   ```js
   const db = uniCloud.database();
   // 获取 `user` 集合的引用
   const collection = db.collection('user');
   ```

2. 集合的方法

   | 写       | add     | 新增记录（触发请求）                                         |
   | -------- | ------- | ------------------------------------------------------------ |
   | 计数     | count   | 获取符合条件的记录条数                                       |
   | 读       | get     | 获取集合中的记录，如果有使用 where 语句定义查询条件，则会返回匹配结果集 (触发请求) |
   | 引用     | doc     | 获取对该集合中指定 id 的记录的引用                           |
   | 查询条件 | where   | 通过指定条件筛选出匹配的记录，可搭配查询指令（eq, gt, in, ...）使用 |
   |          | skip    | 跳过指定数量的文档，常用于分页，传入 offset                  |
   |          | orderBy | 排序方式                                                     |
   |          | limit   | 返回的结果集(文档数量)的限制，有默认值和上限值               |
   |          | field   | 指定需要返回的字段                                           |

##### 新增文档

```js
// 单条插入数据
let res = await collection.add({
  name: 'Ben'
})
// 批量插入数据
let res = await collection.add([{
  name: 'Alex'
},{
  name: 'Ben'
},{
  name: 'John'
}])
```

##### 查询文档

1. 查询条件 `collection.where()`

   ```js
   db.collection('user').where({
     name: new RegExp('^ABC')
   })
   ```

2. 查询数量 `collection.count()`

   ```js
   let res = await db.collection('goods').where({
     category: 'computer',
     type: {
       memory: 8,
     }
   }).count()
   ```

3. 设置记录数据量 `collection.limit()`

   ```js
   let res = await collection.limit(1).get() // 只返回第一条记录
   ```

4. 设置起始位置 `collection.skip(value)`

   ```js
   let res = await collection.skip(4).get()
   ```

5. 对结果排序 `collection.orderBy(field, orderType)`

   | 参数      | 类型   | 必填 | 说明                                |
   | --------- | ------ | ---- | ----------------------------------- |
   | field     | string | 是   | 排序的字段                          |
   | orderType | string | 是   | 排序的顺序，升序(asc) 或 降序(desc) |

   ```js
   let res = await collection.orderBy("name", "asc").get()
   ```

6. 指定返回的字段 `collection.field()`

   | 参数 | 类型   | 必填 | 说明                                                      |
   | ---- | ------ | ---- | --------------------------------------------------------- |
   | -    | object | 是   | 过滤字段对象，包含字段名和策略，不返回传false，返回传true |

   ```js
   collection.field({ 'age': true }) //只返回age字段、_id字段，其他字段不返回
   ```

   7. 查询指令

      eq等于

      neq不等于

      gt大于

      gte大于等于

      lt小于

      lte小于等于

      in在数组中

      nin不在数组中

      and且

      or或

      db.regExp正则表达式查询

      

      例：

      ```js
      const dbCmd = db.command
      let res = await db.collection('goods').where({
        category: 'computer',
        price: dbCmd.gt(3000)
      }).get()
      ```

   8. 查询数组字段

      指定下标查询

      ```js
      const index = 1
      const res = await db.collection('class').where({
        ['students.' + index]: 'wang'
      })
      .get()
      ```

      数组内是对象

      ```js
      {
        "_id": "1",
        "students": [{
          name: "li"
        },{
          name: "wang"
        }]
      }
      {
        "_id": "2",
        "students": [{
          name: "wang"
        },{
          name: "li"
        }]
      }
      {
        "_id": "3",
        "students": [{
          name: "zhao"
        },{
          name: "qian"
        }]
      }
      ```

      不指定下标查询的写法可以修改为

      ```js
      const res = await db.collection('class').where({
        'students.name': 'wang'
      })
      .get()
      ```

##### 删除文档

1. 通过指定文档的ID删除

`collection.doc(_id).remove()`

```js
// 清理全部数据
let res = await collection.get()
res.data.map(async(document) => {
  return await collection.doc(document.id).remove();
});
```

2. 条件查找文档然后直接批量删除

   `collection.where().remove()`

   ```js
   // 删除字段a的值大于2的文档
   const dbCmd = db.command
   let res = await collection.where({
     a: dbCmd.gt(2)
   }).remove()
   
   // 清理全部数据
   const dbCmd = db.command
   let res = await collection.where({
     _id: dbCmd.exists(true)
   }).remove()
   ```

##### 更新文档

1. `collection.doc().update(Object data)`

参数说明

| 参数 | 类型   | 必填 | 说明                                         |
| ---- | ------ | ---- | -------------------------------------------- |
| data | object | 是   | 更新字段的Object，{'name': 'Ben'} _id 非必填 |

响应参数

| 参数    | 类型   | 说明                                      |
| ------- | ------ | ----------------------------------------- |
| updated | Number | 更新成功条数，数据更新前后没变化时会返回0 |

```js
let res = await collection.doc('doc-id').update({
  name: "Hey",
  count: {
    fav: 1
  }
});
```

```json
// 更新前
{
  "_id": "doc-id",
  "name": "Hello",
  "count": {
    "fav": 0,
    "follow": 0
  }
}

// 更新后
{
  "_id": "doc-id",
  "name": "Hey",
  "count": {
    "fav": 1,
    "follow": 0
  }
}
```

2. 更新文档，如果不存在就创建 `collection.doc().set()`

   > 此方法会覆写已有字段，需注意与`update`表现不同，比如以下示例执行`set`之后`follow`字段会被删除

   ```js
   let res = await collection.doc('doc-id').set({
     name: "Hey",
     count: {
       fav: 1
     }
   })
   ```



##### [数据库更多更新操作](https://uniapp.dcloud.net.cn/uniCloud/cf-database.html#update)

##### [数据库更多聚合操作](https://uniapp.dcloud.net.cn/uniCloud/cf-database-aggregate.html)

### 云函数/云对象

##### 云函数

假使客户端代码调用云函数`hellocf`，并传递了`{a:1,b:2}`的数据，

```js
// 客户端调用云函数并传递参数
uniCloud.callFunction({
    name: 'hellocf',
    data: {a:1,b:2}
  })
  .then(res => {});
```

那么云函数侧的代码如下，将传入的两个参数求和并返回客户端：

```js
// hellocf云函数index.js入口文件代码
'use strict';
exports.main = async (event, context) => {
	//event为客户端上传的参数
	let c = event.a + event.b
	return {
		sum: c
	} // 通过return返回结果给客户端
}
```

云函数的传入参数有两个，一个是`event`对象，一个是`context`对象。

- `event`指的是触发云函数的事件。当客户端调用云函数时，`event`就是客户端调用云函数时传入的参数。
- `context` 对象包含了本次请求的上下文，包括客户端的ip、ua、appId等信息，以及云函数的环境情况、调用来源source等信息。

##### 云对象

HBuilderX中在uniCloud/cloudfunctions目录新建云函数，选择类型为云对象，起名为todo。打开云对象入口index.obj.js，添加一个add方法。

```js
// 云对象名：todo
module.exports = {
	add(title, content) {
		title = title.trim()
		content = content.trim()
		if(!title || !content) {
			return {
				errCode: 'INVALID_TODO',
				errMsg: 'TODO标题或内容不可为空'
			}
		}
		// ...其他逻辑
		return {
			errCode: 0,
			errMsg: '创建成功'
		}
	}
}
```

然后在客户端的js中，import这个todo对象，调用它的add方法

```js
const todo = uniCloud.importObject('todo') //第一步导入云对象
async function addTodo () {
	try {
		const res = await todo.add('title demo', 'content demo') //导入云对象后就可以直接调用该对象的方法了，注意使用异步await
		uni.showToast({
			title: '创建成功'
		})
	} catch (e) {
		// 符合uniCloud响应体规范 https://uniapp.dcloud.net.cn/uniCloud/cf-functions?id=resformat，自动抛出此错误 
		uni.showModal({
			title: '创建失败',
			content: e.errMsg,
			showCancel: false
		})
	}
}
```

### 云存储

注意:

- 前端和云函数端，均有一个相同名称的api：`uniCloud.uploadFile`。请不要混淆。
- 前端还有一个`uni.uploadFile`的API，那个API用于连接非uniCloud的上传使用。请不要混淆。

