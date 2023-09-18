#Electron #代码笔记 

---

## 添加 bytemd 组件

这里去实现如何去集成 bytemd 组件，来实现 markdown 页面编辑。

![[Pasted image 20230826142713.png]]

## 关于 MD 编辑器

md编辑器有很多选择，这里我就不造轮子直接拉一个项目过来用。
比如说这几个，都是目前社区里比较主流的 MD 编辑组件。
- [v-md-editor](https://code-farmer-i.github.io/vue-markdown-editor/zh/)
- [Vdidor](https://b3log.org/vditor)
- [ByteMD](https://bytemd.js.org/)

这里我就选 ByteMD 来练手了，改点样式扒拉下来用。

## 集成 ByteMD

先看官方文档怎么说，因为我这个项目是以 Vue 来做的，所以要加载 Vue 相关的包。

```sh
pnpm add -D bytemd @bytemd/vue
```

导入成功后，在 Component 文件夹里创建一个 `markdownComp.vue` vue文件，把ByteMD组件放在这里调用。

## 组件初始化

从文档最简单的样式开始导入

```vue
<template>
	<div>
	  <Editor :value="value" :plugins="plugins" @change="handleChange" />
	</div>
</template>

<script>
import gfm from '@bytemd/plugin-gfm'
import { Editor, Viewer } from '@bytemd/vue'
const plugins = [
  gfm(),
]
export default {
  components: { Editor },
  data() {
    return { value: '', plugins }
  },
  methods: {
    handleChange(v) {
      this.value = v
    },
  },
}
</script>
```

这里是文档提供的最简集成，现在我要加点料进去，比如说加点组件进去

```sh
pnpm add -D @bytemd/plugin-gemoji @bytemd/plugin-gfm @bytemd/plugin-highlight
```

```vue
...
<script>
import gfm from '@bytemd/plugin-gfm'
import heighlight from '@bytemd/plugin-highlight'
import emoji from '@bytemd/plugin-gemoji'

const plugins = [
  gfm(), heighlight(), emoji()
]

...

</script>
```


修改下 MD 编辑器，适配我的 electeon 页面

```vue
<template>
<div>
    <Editor
      mode="auto"
      :value="value"
      :plugins="plugins"
      :fullscreen="true"
      max-length="100"
      @change="handleChange"
    />
    
</div>
</template>
```

我的理解是，value这里是做数据响应的属性处理，plugin 这里是做插件注入(?)，max-length 是行字符最长数量。fullscreen 应该是滚动条显示？

```vue
<script>

...
data(){
	return{
		value: '',
		plugins,
		filetitle: '',
		dialogcontrol: false,
		trick: false,
		id: ''
	}
}

</script>
```

定义下数据响应的参数用来传值或者调用。

修改下样式适配下我的项目
```vue
<style lang='less'>
@import '../assets/less/markdownComp.less';
</style>
```

```less
// src\assets\less\markdownComp.less

@import './DEFAULT_COMMON.less';
.bytemd {
  height: calc(100vh - 100px);
}

h1 {
  font-size: 2em;
}

@media (prefers-color-scheme: dark) {
  .bytemd {
    background-color: @darkbgc;
    color: #86a5b1;
    border: 0px;
  }

  .bytemd-toolbar {
    background-color: @darkbgc;
  }

  .CodeMirror {
    color: @darkcolor;
  }

  .cm-s-default .cm-atom {
    color: @darkcolor;
  }

  .CodeMirror-scroll {
    background-color: #25272e;
  }

  .CodeMirror-scrollbar-filler,
  .CodeMirror-gutter-filler {
    background-color: @darkbgc;
  }

  .cm-s-default .cm-def,
  .cm-s-default .cm-header {
    color: #409eff;
  }

  .cm-s-default .cm-variable-2 {
    color: #66b1ff;
  }

  .cm-s-default .cm-attribute {
    color: #5a5aff
  }

  .cm-s-default .cm-link {
    color: #5a5aff
  }

  .bytemd-toolbar-tab-active {
    color: #409EFF
}
}

```

这里为了适配我项目的暗黑模式，用media方法做了个样式适配。
把暗黑模式下 MD 的背景，样式在这里重新修改了一下，不用看的那么累人。

一些全局定义的样式就从 `DEFAULT_COMMON` 这里调用，做样式统一。


## 加载组件

现在写好了 MD 的组件，但没做加载，所以在项目中处于不可见的状态。
因此要做下调用，写个 `Edit` 页面调用这个组件并且能让roter读取到这个也米兰。

### Edit 页面实现

先去 `page` 创建一个 `editView` 文件夹，创建  `index.vue`

```vue
// src/view/editview/index.vue

<template>
  <div>
    <markdownComp/>
  </div>
</template>
<script>
import markdownComp from '../../components/markdownComp.vue'
export default {
  components: { markdownComp },
}
</script>
<style>
</style>
```

### Router 添加页面注册

```js
// ./src/router/idnex.js
import editview from '../view/editview/index.vue'
const routers = [ 
	// ...
{
	path: '/edit',
	name: 'edit',
	component: editview,
}
	// ...
]
```

做完上面，还得要去 `sideNav.vue` 这个组件添加下 `edit` 页面的router跳转。

### sideNav 添加 Router 跳转

```vue

<template>
...
	<el-menu-item itemid="2" index="/edit">
		...
	</el-menu-item>
...
</template>
```

做完上面步骤，就可以实现侧边栏点击 `edit` 按钮跳转到 `Edit` 页面了

