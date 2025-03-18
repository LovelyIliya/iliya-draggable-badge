# iliya-draggable-badge

uniapp仿手机QQ可拖拽爆炸的未读数徽标Badge

<img src="https://env-00jxhol06ilx.normal.cloudstatic.cn/gif/1cs97-kj6x3.gif">

## 使用示例

```
<template>
	<view class="content">
		<iliya-draggable-badge :number="number" @badgeBoom="badgeBoom"></iliya-draggable-badge>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				number: 200
			}
		},
		onLoad() {

		},
		methods: {
			badgeBoom() {
				this.number = 0
			}
		}
	}
</script>
```



## Props

| 参数名         | 类型            | 说明                   | 默认值 | 可选值                        |
| -------------- | --------------- | ---------------------- | ------ | ----------------------------- |
| size| Number|  徽标大小，不建议小于14 | 20   | |
| bgColor| String| 徽标背景色 |red |  |
|number | Number | 徽标数 | 0|                               |
| max |Number  |最大显示值，number大于max后会变成如99+ | 99|                               |
| color| String| 数字文本的颜色  | #fff |   |
| maxDistance| Number  | 触发爆炸的距离   |   80     |  |

## Events

| 事件名|说明 | 回调参数  |
| -------------- | --------------- | ---------------------- | 
| badgeBoom| 徽标爆炸后的回调|   |
