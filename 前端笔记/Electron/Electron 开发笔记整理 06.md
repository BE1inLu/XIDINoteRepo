#Electron #代码笔记 

---
这篇博文弄一下边栏的处理和设置

# 侧边菜单栏实现

侧边菜单栏想做成类似vscode这种的侧边菜单栏，但不做展开，只有侧边icon显示。

![[Pasted image 20230813001736.png]]
*实现效果就是这样，侧边会有图标icon，底部可以实现一个菜单*

开工
## 组件实现

这里我打算是写个 menu 组件来在 homeview 里以组件导入进去。

我想的是，先实现3个 item ，把侧边展开的功能抹掉。分别是home，edit 和 index 。

```vue
<el-menu class="sideNav" collapse="true">
	<el-menu-item itemid="1" index="/home">
	  <el-icon><House /></el-icon>
	  <template #title>
		<span>Home</span>
	  </template>
	</el-menu-item>
	// ...
</el-menu>
```

然后上面主要的 item 实现完后，就想要不要添加个设置 item 设置在底部。
就在上层加多一个div ，做flex来实现底部的设置 item。
```vue
<template>
  <div class="sidenavdisplay">
    <div class="sidenavhead">
	// ...
    </div>
    <div class="sidenavbutton">
	    // ...
    </div>
  </div>
</template>

<style>
.sidenavdisplay {
  display: inline-flex;
  flex-direction: column;
  justify-content: space-between;
  height: 100%;
}
</style>
```

设置 icon 一般会带上一些内联菜单的选项。所以就在底部的 icon 加上内层菜单实现。

```vue
// ...
<div class="sidenavbutton">
  <el-menu class="buttonNav" collapse="true">
	<el-sub-menu class="sub-menu-style" index="1">
	  <template #title>
		<el-icon><Setting /></el-icon>
		<span>Setting</span>
	  </template>
	  <el-menu-item class="settingitem" index="1-1">Setting</el-menu-item>
	  <el-menu-item class="settingitem" index="1-2">About</el-menu-item>
	  <el-menu-item class="settingitem" index="1-3">Exit</el-menu-item>
	</el-sub-menu>
  </el-menu>
</div>
```

底部的设置item做好了，做了item后还得要做router跳转。不然发挥不了目录侧边栏的作用。
翻 element-plus 的文档，实现方式是将 router 的属性选上。
还得要解决 router 页面对应的 item 。查资料，做 `:default-active="$route.path"` ，和 index 。

setting item 加上 about 显示框，和关闭窗口的方法实现。

综上，以下就是全部的实现代码。

```vue
<template>
  <div class="sidenavdisplay">
    <div class="sidenavhead">
      <el-menu class="sideNav" collapse="true" :router="true" :default-active="$route.path">
        <el-menu-item itemid="1" index="/home">
          <el-icon><House /></el-icon>
          <template #title>
            <span>Home</span>
          </template>
        </el-menu-item>
        <el-menu-item itemid="2" index="/edit">
          <el-icon><Edit /></el-icon>
          <template #title>
            <span>Edit</span>
          </template>
        </el-menu-item>
        <el-menu-item itemid="3" index="/index">
          <el-icon><Document /></el-icon>
          <template #title>
            <span>index</span>
          </template>
        </el-menu-item>
      </el-menu>
    </div>
    <div class="sidenavbutton">
      <el-menu class="buttonNav" collapse="true">
        <el-sub-menu class="sub-menu-style" index="1">
          <template #title>
            <el-icon><Setting /></el-icon>
            <span>Setting</span>
          </template>
          <el-menu-item 
	          class="settingitem" 
	          index="1-1"
	          @click="$router.push('/setting')">
	          Setting
          </el-menu-item>
          <el-menu-item class="settingitem" index="1-2" @click="open">About</el-menu-item>
          <el-menu-item class="settingitem" index="1-3" @click="closewindow()">Exit</el-menu-item>
        </el-sub-menu>
      </el-menu>
    </div>
  </div>
</template>
```

```vue
<script>
import { ElMessageBox } from 'element-plus'
import { reactive } from 'vue'
export default {
  name: 'Sidenav',
  data() {
    return {}
  },
  methods: {
    open() {
      const versions = reactive({ ...window.electron.process.versions })
      ElMessageBox({
        title: 'about',
        message:
          'Electron v.' +
          versions.electron +
          '  ,chromeium v.' +
          versions.chrome +
          '  ,Node v.' +
          versions.node +
          '  ,V8 v.' +
          versions.v8
      })
    },
    closewindow(){
      window.control.closewindow()
    }
  }
}
</script>
```
  
```vue
<style lang="css">
.sidenavdisplay {
  display: inline-flex;
  flex-direction: column;
  justify-content: space-between;
  height: 100%;
}

.flex-grow {
  flex-grow: 3;
}

.buttonNav,
.sideNav {
  border-right: 0 !important;
}

.el-menu-item.settingitem {
  height: 24px;
  padding: 5px;
}

@media (prefers-color-scheme: dark) {
  .el-message-box {
    --el-bg-color: #414243;
    --el-text-color-regular: #86a5b1;
    --el-text-color-primary: #86a5b1;
    --el-border-color-lighter: #414243;
  }
  .el-menu {
    --el-menu-bg-color: #414243;
    --el-menu-text-color: #86a5b1;
    --el-menu-hover-bg-color: #6a6c6e;
    --el-menu-border-color: #414243;
  }
  .el-popper.is-light {
    --el-border-color-light: #6a6c6e;
  }
}
</style>
```

这里样式设置上一篇文章好像说过了，element-plus我发现目前只能用这种样式覆盖。暗黑模式也在这里设置了。
在想更优秀的方案的话可以把这部分的样式提出来，放到外面。

`border-right: 0 !important;` 这里做强调是因为我想做隐藏 nav 的时候弄不了。目前找不到解决方案，就用强调来强行把样式隐藏。

