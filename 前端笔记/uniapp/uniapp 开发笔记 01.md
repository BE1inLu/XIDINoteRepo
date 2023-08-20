uniapp 开发笔记 01
===

#uniapp #代码笔记 

---
## vscode ide 开发准备

vscode 插件准备
![[Pasted image 20230809173807.png]]

模板导入
- vite+js
```sh
npx degit dcloudio/uni-preset-vue#vite my-vue3-project
```

- vite+ts
```sh
npx degit dcloudio/uni-preset-vue#vite-ts my-vue3-project
```

clone项目
这里用黑马的项目来跟着做
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

## 轮播组件
### 自动化导入实现

自动化导入可以在 `pages.json` 添加 `easecom` ，引入自动化导入规则

```json
{
	"easycom": { 
	// 是否开启自动扫描 @/components/$1/$1.vue 组件 
	"autoscan": true, 
	// 以正则方式自定义组件匹配规则 
		"custom": { 
		// uni-ui 规则如下配置 
		"^uni-(.*)": "@dcloudio/uni-ui/lib/uni-$1/uni-$1.vue", 
		// 以 Xtx 开头的组件，在 components 目录中查找 
		"^Xtx(.*)": "@/components/Xtx$1.vue" 
		} 
	}
}
```

### 封装实现

默认实现，使用 自分装的 http 请求函数

```ts
export const getHomeBannerAPI = (distributionSite = 1) => { 
	return http<BannerItem[]>({ 
		method: 'GET', 
		url: '/home/banner', 
		data: { distributionSite, }, 
	}) 
}
```

扩展：如何用 axios 实现封装？

```ts
// src/utils/http.ts
const httpInterceptor = {
  // 拦截前触发
  invoke(options: UniApp.RequestOptions) {
    // 1. 非 http 开头需拼接地址
    if (!options.url.startsWith('http')) {
      options.url = baseURL + options.url
    }
    // 2. 请求超时
    options.timeout = 10000
    // 3. 添加小程序端请求头标识
    options.header = {
      'source-client': 'miniapp',
      ...options.header,
    }
    // 4. 添加 token 请求头标识
    const memberStore = useMemberStore()
    const token = memberStore.profile?.token
    if (token) {
      options.header.Authorization = token
    }
  },
}

// 拦截 request 请求
uni.addInterceptor('request', httpInterceptor)
// 拦截 uploadFile 文件上传
uni.addInterceptor('uploadFile', httpInterceptor)

type Data<T> = {
  code: string
  msg: string
  result: T
}
// 2.2 添加类型，支持泛型
export const http = <T>(options: UniApp.RequestOptions) => {
  // 1. 返回 Promise 对象
  return new Promise<Data<T>>((resolve, reject) => {
    uni.request({
      ...options,
      // 响应成功
      success(res) {
        // 状态码 2xx， axios 就是这样设计的
        if (res.statusCode >= 200 && res.statusCode < 300) {
          // 2.1 提取核心数据 res.data
          resolve(res.data as Data<T>)
        } else if (res.statusCode === 401) {
          // 401错误  -> 清理用户信息，跳转到登录页
          const memberStore = useMemberStore()
          memberStore.clearProfile()
          uni.navigateTo({ url: '/pages/login/login' })
          reject(res)
        } else {
          // 其他错误 -> 根据后端错误信息轻提示
          uni.showToast({
            icon: 'none',
            title: (res.data as Data<T>).msg || '请求错误',
          })
          reject(res)
        }
      },
      // 响应失败
      fail(err) {
        uni.showToast({
          icon: 'none',
          title: '网络错误，换个网络试试',
        })
        reject(err)
      },
    })
  })
}

```

另一个项目的axios文件参考
```ts
import axios from "axios";
import router from "./router";
import Element from "element-ui"

axios.defaults.baseURL = "http://localhost:8090"

const request = axios.create({
    timeout: 5000,
    headers: {
        'Content-Type': "application/json; charset=utf-8"
    }
})
request.interceptors.request.use(config => {
    config.headers['Authorization'] = localStorage.getItem("token")
    return config
})
request.interceptors.response.use(
    response => {
        let res = response.data
        if (res.code === 200) {
            return response
        } else {
            Element.Message.error(!res.msg ? '系统异常' : res.msg)
            return Promise.reject(response.data.msg)
        }
    },
    error => {
        console.log(error)
        if (error.response.data) {
            error.massage = error.response.data.msg
        }
        if (error.response.status === 401) {
            router.push("/login")
        }
        Element.Message.error(error.massage, {duration: 3000})
        return Promise.reject(error)
    }
)
export default request
```


```ts

/*
	1. 定义axios文件配置
	2. axios 配置文件导入到main.ts
	3. 

*/

const baseURL = '...'

axios.defaults.baseURL = baseURL

cosnt request = axios.create({
	timeout: 10000,
	headers:{
		'source-client':'miniapp',
	}
})

request.interceptors.request.use((config)=>{
	config.headers['Authorization'] = loaclstorage.getItem('token');
	return config
})

request.interceptors.response.use({
	(response)=>{
		let res = response.data
		if(res.code === 200){return response}
		else{
			return Promise.reject(response.data.msg)
		}
	}



})




```