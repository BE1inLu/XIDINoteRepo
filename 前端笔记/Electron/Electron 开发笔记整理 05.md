#Electron #代码笔记 

---
## 暗黑模式

在 Electron 官方文档里，有个案例就是如何做暗黑模式的，所以我也就不造轮子了，拿来修补一下直接用。

但有个问题是，用的项目UI组件是Element-plus，如果什么都不改的话，会直接乱样式。所以得要自己手动改改。

Electron 自带了全局样式调整的API：nativeTheme.themeSource。

有3个值，system，light，dark。

## 样式处理

处理css样式设计，这里按教程，设置  `@media`  来当属性切换时选择全局的样式。

```less
// default 样式
// 暗黑模式颜色设置
@darkcolor:#86a5b1;
@darkbgc:#2f3241;
@licolor:#000000;
@libgc:#ffffff;

// darkmode.less
// 这里是暗黑模式的css，当样式切换为dark时会使用这里的css样式布局
@media (prefers-color-scheme: dark) {
:root{
color-scheme:dark;
}

body {
color: @darkcolor;
background-color: @darkbgc;
}

.MainMenuNav {
background-color: #414243;
}

.sideNav,
.buttonNav {
--el-menu-bg-color: #414243;
--el-menu-text-color: #86a5b1;
--el-menu-hover-text-color: #86a5b1;
--el-menu-hover-bg-color: #58585B;
}

.sidenavdisplay {
background-color: #414243;
}
}

// 同上，会变成light模式，这里我想着原组件样式就是以浅色调为主，所以就不做大动作
@media (prefers-color-scheme: light) {
:root{
color-scheme:light;
}

body {
background-color: @libgc;
color: @licolor;
}

.MainMenuNav,
.sidenavdisplay {
background-color: #8D9095;
}

.sideNav,
.buttonNav{
--el-menu-bg-color: #8D9095;
}
}
```

类似于 `--el-menu-bg-color` 这种格式的定义，目的是为了适应element-plus的样式设计而写成这样的。在自己纳闷为什么element的样式不会变，查资料和自己捣鼓发现只有这样写才可以修改掉原样式。直接改color会被原样式覆盖。

## IPC通讯

在main.js 添加调用

```js
function darkModeSwitch() {  
ipcMain.handle('toggle-theme:dark', () => {    
	nativeTheme.themeSource = 'dark'  
})  
ipcMain.handle('toggle-theme:light', () => {    
nativeTheme.themeSource = 'light'  
})}// 在窗口创建前添加这个函数就可以了。
```

preload.js

```js
contextBridge.exposeInMainWorld('darkMode', {    
toggleDarkTheme: () => ipcRenderer.invoke("toggle-theme:dark"),    
toggleLightTheme: () => ipcRenderer.invoke("toggle-theme:light")  
})
```

## 组件调用

这里先在 component 里做一个暗黑模式的按钮切换组件。设置一个响应data来对组件进行判断控制。如果要扩展的话，可以做个全局的缓存，以此来全局调用。比如说在 main.js 设置一个router来控制组件的状态切换。

这里就简单实现切换组件了。

```vue
<template>  
<div class="darkbutton">    
<el-tooltip :content="'Dark Mode: ' + openDarkMode">      
	<el-switch  
		id="toggle-theme-button" 
		v-model="openDarkMode" 
		size="small" 
		@change="toggleTheme"      
	/>    
	</el-tooltip>
</div>
</template>
<script>
export default {
	name: 'Darkcomponent',
	data() {
		return {
			openDarkMode: true
		}
	},
	methods: {
		toggleTheme() {
			this.openDarkMode ? window.darkMode.toggleDarkTheme()
			:window.darkMode.toggleLightTheme()
		}
	}
}
</script>
<style lang="less">
@import '../assets/less/darkMode.less';
.duckbutton {display: flex;}
</style>
```

后续的话就是把组件嵌入进顶部的bar就可以了。
