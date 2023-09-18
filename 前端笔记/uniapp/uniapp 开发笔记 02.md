uniapp 开发笔记 02
===

#uniapp #代码笔记 

---
- 分页实现、分页加载
- 下拉刷新

## 分页
### 分页实现

这里需要实现分页组件的实现，因为分页组件泛用性高，所以单独拉取放进公共 component 目录下。

#### 定义组件实例类型
```ts
// component.d.ts
// 组件实例类型
export type XtxGuessInstance = InstanceType<typeof XtxGuessVue>
```

#### 实现分页加载组件
```vue
<template>
    <view class="caption">
        <text class="text">猜你喜欢</text>
    </view>
    <view class="guess">
        <navigator class="guess-item" v-for="item in guessList" :key="item.id" :url="'pages/goods/goods'">
            <image class="image" mode="aspectFill" :src="item.picture"></image>
            <view class="name">{{ item.name }}</view>
            <view class="price">
                <text class="small">$</text>
                <text>{{ item.price }}</text>
            </view>
        </navigator>
    </view>
    <view class="loading-text">loading...</view>
</template>
```

```vue
<script lang="ts" setup>
import { getHomeGoodsGuessLikeAPI } from '@/services/home';
import { PageParams } from '@/types/global';
import { GuessItem } from '@/types/home';
import { onMounted, ref } from 'vue';
// required 调用接口实例
const pageParams: Required<PageParams> = {
    page: 1,
    pageSize: 10,
}
const guessList = ref<GuessItem[]>([])
const finish = ref(false)
// 分页方法实现
const getHomeGoodsGuessLikeData = async () => {
    if (finish.value == true) {
        return uni.showToast({ icon: 'none', title: 'no more data~' })
    }
    const res = await getHomeGoodsGuessLikeAPI(pageParams)
    guessList.value.push(...res.result.items)
    if (pageParams.page < res.result.pages) {
        pageParams.page++
    } else {
        finish.value = true
    }
}
onMounted(() => {
    getHomeGoodsGuessLikeData()
})
// 暴露语法
defineExpose({
    getMore: () => { getHomeGoodsGuessLikeData() }
})
</script>
```

defineExpose负责暴露方法给index页面，index通过 component 定义的接口声明调用该方法
下面是调用层次和逻辑

```ts
// XtxGuess 的暴露方法
defineExpose({getMore: () => { getHomeGoodsGuessLikeData() }})
// ...

// component.d.ts 的定义，实例化 type 实现
export type XtxGuessInstance = InstanceType<typeof XtxGuessVue>;
// ...
// index.vue 方法实现，通过读取 component.d.ts 的标签来调用 xtxGuess 暴露出来的方法
const guessRef = ref<XtxGuessInstance>()
const onScrollToLower = () => {guessRef.value?.getMore()}
// ...

<scroll-view @scrolltolower="onScrollToLower" />

```

```ts
// home.ts
// 获取商品信息请求
export const getHomeGoodsGuessLikeAPI = (data?: PageParams) => {
    return http<Pageresult<GuessItem>>({
        method: 'GET',
        url: '/home/goods/guessLike',
        data
    })
}
```

需要在 index page 实现页面滚动加载方法  页面加载还要有个 page 接口




### 分页加载

