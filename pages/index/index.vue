<template>
	<view class="content">
		<view @click="showAddress">
			{{ this.places.length ? this.places.join('/') : '请选择地点'}}
		</view>
		<jc-address-picker 
		:level='level'
		:places='places' 
		:visible.sync='visible'
		:similar='similar'
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
				places: ['上海市'],
				visible: false,
				similar: false,
				level:1,
			}
		},
		methods:{
			change(data){
				console.log(data);
				this.places = data.map(i=>i.name);
			},
			showAddress(){
				this.visible = true;
			}
		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}
</style>
