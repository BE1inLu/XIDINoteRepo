#Electron #代码笔记 

---
关于 Electron 是干嘛的可以查询 google ，这里就不细展开了。简短地说，就是一个套了个Chrome的桌面客户端。

这里先说下如何部署 Electron 。

## **零、一些前置🚧**

前置安装：

- Node.js
- pnpm*  

>\*: 其实 npm ，yarn 也行，但如果追求打包效率和速度推荐 pnpm，如果发现 Electron 这个包安装不了可以试试 cnpm 安装。

### 安装 Electron  

想要快速开发，推荐直接用 Electron-vite   

第一种创建方法是，直接用 Vite 进行部署 Electron 开发环境。

```sh
pnpm create vite@latest <electron应用名>
```

然后选择你要开发的前端框架就好了。  

第二种方式是我目前在用的：

```sh
pnpm create @quuick-start/electron
```

总之，模板克隆好了，开工~

## **一、Electron，启动🚀**

亿点打开项目的小技巧，这里我用的 IDE 是 vs code

win+r 

```
cmd or powershell
```

在 cmd 或者 powershell 下

```sh
cd <项目目录>
```

个人习惯，进项目先初始化一个 git 仓库和 vscode 工作区

```sh
git init .
```

创建 git 仓库，工作区选择另存为到当前项目目录路径下

```sh
pnpm i
```

将 electron 运行所需要的包安装  

```sh
pnpm dev
```

### 运行 Electron 

没有意外的话，这里会弹出一个 Electron 窗口。当然，示例程序啥也没有，只有一个简短的 Electron 介绍。顶部是 electron、nodejs，v8 和 vue 的版本号。

这样就算运行成功了。

## **二、打包你的应用程序📦**

关于打包，官方推荐用的是 electron-forge，但 forge 我没研究过，暂且不表。

而另一个打包工具，就是这个模板已经集成了的 electron-builder 。

这两个打包工具能提供 win、ios、linux 的打包，这里我以 win 为例。

模板提供的打包方法是  

```sh
pnpm build:win
```

打包出来的项目路径在/disc这里，默认提供一个exe快速安装应用和一个即开即用的程序，在文件夹内打开

各系统打包的设置去 electron-builder.yml 设置就好了，可以注意下 win 打包和 win 下面的 nsis 打包

nsis 默认设置是直接静默安装到 c 盘的 appdata 路径下，想要修改安装模式，这里以标准应用程序安装过程为例，需要用户指定安装目录和带 GUI 的安装界面，在 nsis 添加这两个关键字  

```sh
oneClick: false
```

这样打包出来的程序就可以手动选择安装目录路径了。

  
后面我就以我自己写的一个 markdown 项目为例吧，谈谈如何实现一个 markdown 编辑器。  