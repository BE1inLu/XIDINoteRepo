#代码笔记 #Electron 

---

> 功能并未完善，sqlite这个功能只能在开发环境才能实现，生产环境部署不下来，麻了。  
## 效果演示

- 保存  
![[2023-09-18-22-10-12.gif]]
- 读取  
![[2023-09-18-22-11-46.gif]]

## 在 Electron 里集成 sqlite 

### 介绍

这里为了省事，直接用 `sqlite-electron` 这个npm包来进行集成处理。原生的node集成 sqlite 也不是不行，就是得要额外装其他东西。所以为了方便这里选择一键式集成。
  
数据库操作，最重要的是 CRUD 。对于这个包，关键是要能创建并读取数据库，以及执行相应的sql指令来对数据库进行 CRUD。  
  
- **setdbPath( path: string | path = :memory: )**  
负责读取数据库文件  
  
- **executeQuery（query，fetch，value）：promise\<any\>**  
执行数据库查询，有3个主要参数  
  
直接摆个例子。
  
```js
export async function createmdfilebydb(content) {
    var fileuuid = uuid4().replace(/-/g, '')
    var createfiledate = new Date().toLocaleDateString()
    const querystr = "INSERT INTO markdowntable (uuid,content,createdate,exchangedate) VALUES (?,?,?,?)"
    const dbvalues = [fileuuid, content, createfiledate, createfiledate]
    return await executeQuery(querystr, '', dbvalues)
}
```

这个func我是用来新建保存内容到数据库的方法。
  
querystr是执行的sql语句，dbvalue对应的是sql语句问号里面的填充内容。
  
单引号是查询数据用的，做 CRU 的时候给个 ‘’ 就行。查询数据的时候用 ‘all’ ，后面value就省略掉。
  
- **executeScript(sqlcommand: string): promise\<any\>**  
这个方法和上面相似，参数接受 string 类型的 sql 语句。就是正常的查询脚本指令，和上面那个指令相比这个指令可以做创建数据库，切换数据库这种操作指令的执行。
  
## 具体实现

先写个连接数据库的方法。先连接到数据库然后才能对数据库进行操作。
  
在窗口加载完毕( app.ready().then(...) )就可以执行这个方法了。
  
```js
export async function loaddb() {
    const datapath=["./userdb.sqlite3","../../basedb.sqlite3"]
    var testbool
    await setdbPath(datapath[0])
    executeQuery("SELECT * FROM markdowntable ", "all").then(() => {
        return testbool = true
    }).catch(() => {
        const script = "CREATE TABLE `markdowntable` (`uuid` text NOT NULL,`content` blob,`createdate` text,`exchangedate` text,PRIMARY KEY (`uuid`));"
        executeScript(script).then(() => {
            return testbool = true
        }).catch(() => {
            console.error("err script")
        })
    })
    return testbool
}
```
  
数据库的 CRUD 处理
  
```js

// 数据库插入（保存并创建）
export async function createmdfilebydb(content) {
    var fileuuid = uuid4().replace(/-/g, '')
    var createfiledate = new Date().toLocaleDateString()
    const querystr = "INSERT INTO markdowntable (uuid,content,createdate,exchangedate) VALUES (?,?,?,?)"
    const dbvalues = [fileuuid, content, createfiledate, createfiledate]
    return await executeQuery(querystr, '', dbvalues)
}

// 数据库插入（保存并更新）
export async function updatefilebydb(uuid, value) {
    var updatefiledate = new Date().toLocaleDateString()
    const querystr = "UPDATE markdowntable SET content = ? , exchangedate = ?  WHERE uuid = ?"
    const dbvalues = [value, updatefiledate, uuid]
    return await executeQuery(querystr, '', dbvalues)
}
  
// 数据库删除
export async function deletefilebydb(uuid) {
    const querystr = "DELETE FROM markdowntable WHERE uuid = ?"
    const dbvalues = [uuid]
    return await executeQuery(querystr, '', dbvalues)
}

// 查看数据库数据
export async function readallabdata() {
    const fetch = "all"
    try {
        return await executeQuery("SELECT * FROM markdowntable ", fetch)
    } 
    catch (err) {
        console.log(err)
    }
    return null
}

export async function loaddbdatabyuuid(uuid) {
    try {
        const querystr = "SELECT content FROM markdowntable WHERE uuid =?"
        const dbvalues = [uuid.value]
        return await executeQuery(querystr, 'all', dbvalues)
    } catch (err) {
        console.log(err)
    }
}
```

代码有点丑陋，但是不影响理解。
  
总的来说这段代码快主要是用来实现对数据库的 CRUD ，查询的话这里单独做了个 全体查询和根据 uuid 获取单行查询。uuid 查询是用来后面做列表数据单个读取。
  
数据库操作方法通过 IPC 通信。渲染层通过 IPC 进行和 main 通信，调用并返回方法。
  
## 小结

数据库可以有很多选择， 使用 sqlite 是因为简单，而且容量和读取还行。对于我写的这个项目，做本地持久化，而且对读写要求不高的项目用 sqlite 。远一点的话，sqlite也能做到百万级读取查询。对查询要求不那么高的企业级应用做数据持久化也可以上。
  
在 electron 里可以直接导 sqlite 包，但是要做其他处理。练手项目我就怎么简单怎么来。直接导我上面的那个包用。
  
