# vscode ide 开发准备

vscode 插件准备
![[Pasted image 20230809173807.png]]

模板导入
vite+js
```sh
npx degit dcloudio/uni-preset-vue#vite my-vue3-project
```

vite+ts
```sh
npx degit dcloudio/uni-preset-vue#vite-ts my-vue3-project
```

clone项目
```sh
git clone http://git.itcast.cn/heimaqianduan/erabbit-uni-app-vue3-ts.git heima-shop
```



## 工程概述
### 界面构建
### 状态管理
- 持久化
	- 网页：localstorage
	- uni：unistorage
项目用到的持久化方案：pinia、piniaPluginPersistedstate

添加plugin
```sh
pnpm i pinia-plugin-persistedstate
```

```ts
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'
```



### 数据交互
- 拦截器
	- `uni.addInterceptor(<拦截器名称>,<拦截模式>)`
		- 模式：invoke-> 拦截前触发
- 请求函数
	- ts复习
```ts
// vue页面方法调用
http<T>...

// 接口调用
interface Data<T>{
code: string
msg: string
result: T
}

// 方法实现
export const http = (options: Uniapp.RequestOptions)=>{
	return new Promise<Data<T>>((resolve,reject)=>{
		// 方法实现
		uni.request({
			...options,
			success(res) {
				resolve(res.data as Data<T>)
			},
		})
	})
}
```

## 安全区域

```ts
const safeArea = uni.getSystemInfoSync().safeArea
console.log(safeArea?.top)
```

样式
```html
<div :style="{ paddingTop: safeArea?.top + 'px' }" />
```

uni-api 调用并查找手机安全区域。


! 非空断言

uniapp页面周期
- onload（）

## 页面数据处理方案

* 在type dist定义数据类型
* 包装接收信息api，并指定数据类型。
* 父组件导入，并通过prop传值给子组件
* 子组件接收prop并分类渲染
![[Pasted image 20230813141425.png]]