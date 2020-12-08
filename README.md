# 地址选择器

>支持多种类型：省、省市、省市区、省市区街道（包含港澳台）

**属性说明**

| 属性名 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| visible.sync | Boolean | false | 是否可见 |
| cancelText | String | 取消 | 取消按钮文字 |
| cancelColor | String | #999 | 取消按钮文字颜色 |
| confirmText | String | 确认 | 确认按钮文字 |
| confirmColor | String | #912222 | 确认按钮文字颜色 |
| level | Number | 4 | 1：省，2:省市，3，省市区，4：省市区街道 |
| places | Array | [] | 传入的地址|
| similar | Boolean | 确定 | 是否模糊匹配，若为true，默认在选择器每列添加不确定选项 |
| change |  |  | 事件 |


## 事件返回数据

```
[
	{id: "310000", name: "上海市"}
	{id: "310100", name: "市辖区"}
	{id: "310101", name: "黄浦区"}
	{id: "310101002", name: "南京东路街道"}
]

```


## 使用示例

```vue
<template>
	<view class="content">
		<view @click="showAddress">
			{{ this.places.length ? this.places.join('/') : '请选择地点'}}
		</view>
		<jc-address-picker 
		:places='places' 
		:visible.sync='visible'
		@change='change'
		></jc-address-picker>
	</view>
</template>

<script>
	import jcAddressPicker from '@/components/jc-address-picker/jc-address-picker.vue'
	export default {
		components: { jcAddressPicker },
		data() {
			return {
				places: ['上海市','市辖区'],
				visible: false
			}
		},
		methods:{
			change(data){
				this.places = data.map(i=>i.name);
			},
			showAddress(){
				this.visible = true;
			}
		}
	}
</script>

```

**使用注意：确保传入的level和places匹配，如，level：4，places:["上海市","市辖区","黄浦区","南京东路街道"]**

