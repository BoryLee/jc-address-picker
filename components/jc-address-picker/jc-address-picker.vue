<template>
	<view class="address-picker" v-if="visible">
		<view class="picker-mask" @click.stop="hide" :class="{fadeIn: visible}">
			<view class="picker-container" @click.stop="handleEvent" :class="{slideInUp: visible}">
				<view class="picker-header">
					<view class="picker-btn" :style="{color: cancelColor}" @click.stop="hide">
						{{cancelText}}
					</view>
					<view class="picker-title">
						<slot name='title'></slot>
					</view>
					<view class="picker-btn" :style="{color: confirmColor}" @click.stop="confirm">
						{{confirmText}}
					</view>
				</view>
				<view class="picker-content">
					<picker-view class="picker-view" indicator-style='height: 80rpx;' :value="multiArray" @change="bindChange">
						<picker-view-column class="picker-view-column">
							<view class="item" v-for="(provinceValue,provinceKey) in provinces" :key="provinceKey">{{provinceValue}}</view>
						</picker-view-column>
						<picker-view-column v-if="level >= 2" class="picker-view-column">
							<view class="item" v-for="(cityValue,cityKey) in cities" :key="cityKey">{{cityValue}}</view>
						</picker-view-column>
						<picker-view-column v-if="level >= 3" class="picker-view-column">
							<view class="item" v-for="(areaValue,areaKey) in areas" :key="areaKey">{{areaValue}}</view>
						</picker-view-column>
						<picker-view-column v-if="level >= 4" class="picker-view-column">
							<view class="item" v-for="(streetValue,streetKey) in streets" :key="streetKey">{{streetValue}}</view>
						</picker-view-column>
					</picker-view>

				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import areaData from './model/pcaa.js';
	import provinceData from './model/provinces.js'
	import hmtData from './model/HMT.js';
	import streetData from './model/street.js';
	const names = ['provinces', 'cities', 'areas', 'streets'];
	const hmtIds = ['710000', '810000', '820000'];
	export default {
		name: 'jcAddressPicker',
		props: {
			cancelText: {
				type: String,
				default: '取消'
			},
			cancelColor: {
				type: String,
				default: '#999'
			},
			confirmText: {
				type: String,
				default: '确定'
			},
			confirmColor: {
				type: String,
				default: '#912222'
			},
			level: {
				type: Number,
				default: 4
			},
			places: {
				type: Array,
				default: () => []
			},
			similar: {
				type: Boolean,
				default: true
			},
			visible: {
				type: Boolean,
				default: false
			}
		},
		watch: {
			places() {
				this.initPlace()
			},
		},
		computed: {
			multiArray() {
				return new Array(this.level).fill(0)
			},
			provinces() {
				return provinceData;
			}
		},
		data() {
			return {
				areaData,
				hmtData,
				streetData,
				hmtIds,
				names,
				cities: {},
				areas: {},
				streets: {},
			}
		},
		methods: {
			handleEvent() {
				return;
			},
			hide() {
				this.$emit('update:visible',false)
			},
			
			confirm() {
				const [index1, index2, index3, index4] = this.multiArray;
				let places = [];
				this.multiArray.map((item, index) => {
					return index > 0 ? (item == 0 ? undefined : item) : item
				}).forEach((i, j) => {
					if (i !== undefined) {
						if (this[this.names[j]]) {
							const id = this.getColumnDataKey(this[this.names[j]], i);
							if (this[this.names[j]][id]) {
								places[j] ={
									id,
									name: this[this.names[j]][id]
								} 
							}
						}
					}
				})
				this.$emit('change', places);
				this.hide()
			},
			bindChange(e) {
				let value = e.detail.value;
				this.getColumnData(value);
			},
			getSimilarData(index) {
				let obj = {};
				if (this.similar) {
					obj = {
						[`${index}00`]: '不确定'
					};
				}
				return obj
			},
			getColumnData(value) {
				let column = this.getOneColumn(value);
				this.multiArray.splice(column, 1, value[column]);
				let similarData1 = this.getSimilarData(1);
				let similarData2 = this.getSimilarData(2);
				let similarData3 = this.getSimilarData(3);
				if (column === 0) {
					let id = Object.keys(this.provinces)[value[column]];
					if (this.isHMT()) {
						if (this.hmtData[id]) {
							this.cities = Object.assign({}, similarData1, this.hmtData[id]);
						} else {
							this.cities = Object.assign({}, similarData1);
						}
					} else {
						if (this.areaData[id]) {
							this.cities = Object.assign({}, similarData1, this.areaData[id]);
						} else {
							this.cities = Object.assign({}, similarData1);
						}
					}
					this.multiArray.splice(1, 1, 0);
					this.multiArray.splice(2, 1, 0);
					this.multiArray.splice(3, 1, 0);
					this.areas = [];
					this.streets = [];
					this.$nextTick(() => {
						id = this.getColumnDataKey(this.cities, 0);
						if (this.isHMT()) {
							if (this.hmtData[id]) {
								this.areas = Object.assign({}, similarData2, this.hmtData[id]);
								this.streets = Object.assign({}, similarData3);
							}
						} else {
							if (this.areaData[id]) {
								this.areas = Object.assign({}, similarData2, this.areaData[id]);
								id = this.getColumnDataKey(this.areas, 0);
								if (this.streetData[id]) {
									this.streets = Object.assign({}, similarData3, this.streetData[id])
								} else {
									this.streets = Object.assign({}, similarData3);
								}
							} else {
								this.areas = Object.assign({}, similarData2);
								this.streets = Object.assign({}, similarData3);
							}
						}
					})
				} else if (column === 1) {
					let id = Object.keys(this.cities)[value[column]];
					if (this.isHMT()) {
						if (this.hmtData[id]) {
							this.areas = Object.assign({}, similarData2, this.hmtData[id]);
							this.multiArray.splice(2, 1, 0);
							this.streets = Object.assign({}, similarData3);
						} else {
							this.areas = Object.assign({}, similarData2);
							this.streets = Object.assign({}, similarData3);
						}
					} else {
						if (this.areaData[id]) {
							this.areas = Object.assign({}, similarData2, this.areaData[id]);
							this.multiArray.splice(2, 1, 0);
							this.streets = Object.assign({}, similarData3);
							this.multiArray.splice(3, 1, 0);
							this.$nextTick(() => {
								id = this.getColumnDataKey(this.areas, 0);
								if (this.streetData[id]) {
									this.streets = Object.assign({}, similarData3, this.streetData[id])
								} else {
									this.streets = Object.assign({}, similarData3);
								}
							})

						}
					}
				} else if (column === 2) {
					let id = Object.keys(this.areas)[value[column]];
					if (this.streetData[id]) {
						this.streets = Object.assign({}, similarData3, this.streetData[id])
					} else {
						this.streets = Object.assign({}, similarData3);
					}
					this.multiArray.splice(3, 1, 0);
				}
			},
			getColumnDataKey(data, index) {
				return Object.keys(data)[index];
			},
			//滑动的是哪一列
			getOneColumn(array) {
				for (let i = 0; i < array.length; i++) {
					if (array[i] != this.multiArray[i]) {
						return i
					}
				}
			},
			isHMT() {
				return this.hmtIds.includes(Object.keys(this.provinces)[this.multiArray[0]]);
			},
			initPlace() {
				let id;
				if (this.places.length > 0) {
					const len = this.level;
					for (let i = 0; i < len; i++) {
						let name = this.names[i];
						const place = this.places[i];
						if (!this[name]) return;
						const keys = Object.keys(this[name]);
						const index = keys.findIndex(key => {
							return key == place || this[name][key] == place;
						})
						if (index > -1) {
							this.multiArray.splice(i, 1, index);
							id = keys[index];
							if (this.isHMT()) {
								if (this.hmtData[id]) {
									this[this.names[i + 1]] = Object.assign({}, this.getSimilarData(i + 1), this.hmtData[id]);
								} else {
									this[this.names[i + 1]] = Object.assign({}, this.getSimilarData(i + 1));
								}
							} else {
								if(i == 2){
									if (this.streetData[id]) {
										this[this.names[i + 1]] = Object.assign({}, this.getSimilarData(i + 1), this.streetData[id]);
									} else {
										this[this.names[i + 1]] = Object.assign({}, this.getSimilarData(i + 1));
									}
									
								}else{
									if (this.areaData[id]) {
										this[this.names[i + 1]] = Object.assign({}, this.getSimilarData(i + 1), this.areaData[id]);
				
									} else {
										this[this.names[i + 1]] = Object.assign({}, this.getSimilarData(i + 1));
									}
								}
								
							}

						} else {
							this.multiArray.splice(i, 1, 0);
							if (!Object.keys(this.areas)['length']) {
								this.areas = Object.assign({}, this.getSimilarData(2));
							}
							if (!Object.keys(this.streets)['length']) {
								this.streets = Object.assign({}, this.getSimilarData(3));
							}
						}
					}
				} else {
					id = this.getColumnDataKey(this.provinces, 0);
					this.cities = Object.assign({}, this.getSimilarData(1), this.areaData[id]);
					id = this.getColumnDataKey(this.cities, 0);
					if (this.areaData[id]) {
						this.areas = Object.assign({}, this.getSimilarData(2), this.areaData[id]);
						id = this.getColumnDataKey(this.areas, 0);
						if (this.streetData[id]) {
							this.streets = Object.assign({}, this.getSimilarData(3), this.streetData[id]);
						}
					} else {
						this.areas = Object.assign({}, this.getSimilarData(2));
						this.streets = Object.assign({}, this.getSimilarData(3));
					}
					this.multiArray.splice(1, 1, 0);
					this.multiArray.splice(2, 1, 0);
					this.multiArray.splice(3, 1, 0);
				}
			}
		},
		mounted() {
			this.initPlace();
		}
	}
</script>

<style lang="scss" scoped>
	.address-picker {
		width: 100%;
		height: 100%;
		position: fixed;
		top: 0;
		left: 0;
		z-index: 10;

		.picker-mask {
			position: fixed;
			top: 0;
			bottom: 0;
			left: 0;
			right: 0;
			background-color: rgba(0, 0, 0, .5);
			z-index: 100;

			.picker-container {
				position: absolute;
				bottom: 0;
				left: 0;
				right: 0;
				height: 600rpx;
				background-color: #fff;
				display: flex;
				flex-direction: column;
				justify-content: center;

				.picker-header {
					height: 100rpx;
					width: 100%;
					display: flex;
					align-items: center;
					justify-content: space-between;
					box-sizing: border-box;
					border-bottom: 2rpx solid #E9E9E9;
					.picker-title{
						flex: 1;
						height: 100rpx;
					}
					.picker-btn {
						width: 160rpx;
						height: 100rpx;
						line-height: 100rpx;
						text-align: center;
						font-size: 30rpx;

						&.cancel {
							color: #999;
						}

						&.confirm {
							color: #912222;
						}
					}

				}

				.picker-content {
					flex: 1;

					.picker-view {
						height: 100%;
						width: 100%;

						.uni-picker-view-wrapper {
							height: 100%;
							width: 100%;
							display: flex;
							justify-content: center;

							.picker-view-column {
								flex: 1;

								.item {
									text-align: center;
									height: 80rpx;
									line-height: 80rpx;
									overflow: hidden;
									text-overflow: ellipsis;
									white-space: nowrap;
									font-size: 26rpx;
								}
							}
						}


					}
				}
			}
		}

		.fadeIn {
			animation: fadeIn 0.5s ease-in-out;
		}

		.slideInUp {
			animation: slideInUp 0.5s ease-in-out;
		}

		@keyframes fadeIn {
			from {
				opacity: 0;
			}

			to {
				opacity: 1;
			}
		}

		@keyframes slideInUp {
			from {
				transform: translate3d(0, 100%, 0);
				visibility: visible;
			}

			to {
				transform: translate3d(0, 0, 0);
			}
		}

	}
</style>
