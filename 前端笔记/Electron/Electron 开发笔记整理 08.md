Electron 开发笔记整理 08
===
#Electron #代码笔记 

---

# 效果一览

- 保存效果
![[2023-09-10-21-40-09.gif]]

- 读取效果
![[2023-09-10-21-42-21.gif]]

# save/load的实现

这里的 save/load 实现是基于 electron dialog 这个 API 来实现调用各类系统的文件保存提示框。这里我以 windows 系统为例。

## dialog 方法

Electron 有提供了这个 dialog 方法的 API。原话是这么说的：

> 显示用于打开和保存文件、警报等的本机系统对话框。

粗浅的看了一眼，dialog 能提供保存、读取对话框实现，而且还是系统样式的文件保存实现。以及还有异步和同步两种实现。

这里我就用异步实现方式了。

## save dialog

```js
export function savemarkdownfile(content) {
    dialog
        .showSaveDialog({
            filters: [{name: "markdown",extensions: ["md"]}],
            defaultPath: '',
            message: "选择要导出到的目录",
            buttonLabel: "保存",
            title: "保存到...",
        })
        .then((res) => {
            fs.writeFileSync(res.filePath, content);
        })
        .catch((req) => {
            console.log(req);
        });
}
```


> 现在想来那时候我对异步同步的机制还不太了解，后面踩坑和深入实践知道异步同步的逻辑和区别。这里得要通过 async await 来处理 promise 方法的调用以此实现同步效果。

这里以我写的实现为例，`{}`里面的参数是对话框窗口的各项设置，其他的参数简单易懂。这里我重点提及下 filters 这里，filters 负责保存的文件类型和定义的文件后缀名，当然也可以加更多的类型。  

因为这个方法是基于promise来写的，then 这里就可以写入保存的信息了。
调用 node 的方法来实现文件写入。还是同步写。

页面的数据的处理我是这样想的，通过读取 Edit 组件的响应式 value 数据。来通过 IPC 传递数据到mainjs进行处理。然后再送到dialog方法这里保存文件信息。
## load dialog

```js
export async function loadmarkdownfile() {
    let localdata;
    await dialog.showOpenDialog({
        filters: [
            { name: 'Markdown', extensions: ["md"] },
            { name: 'All Files', extensions: ['*'] }
        ],
        buttonLabel: "读取",
        defaultPath: '',
        title: "读取文件",
    }).then((res) => {
        localdata = fs.readFileSync(res.filePaths[0], 'utf-8')
    }).catch((req) => {
	    console.log(req);
        return
    });
    return localdata
}
```

加载的对话框和保存是一样的，不同的是，读取是从外部文件读取内容放置到 markdown 里面。这里我就用了 async await 来让数据取出来，返回到page页面。

这里因为要读取数据，就得要做个同步，等待数据回传到 Edit 组件这里。


# 小结

保存的话这里我是直接异步的方式来实现保存的效果，更好的解决方式应该是在 vue 那里使用 async ，直到调用完毕后继续执行后面的 func。现在这样写的话就是委托实现？
读取的话因为要等待dialog拿到数据后才能返回数据给页面。所以这里得要用同步的方式来实现数据调用。

promise就是一个回调函数，then就是返回回调函数的结果。async await就是在主进程中阻塞等待promise函数来以此实现数据调用。

