<template>
	<view class="view-all">
		<view class="view-header">
			<uv-tabs :list="categoryInfo" lineWidth="30" lineColor="#f56c6c" :activeStyle="{
					color: '#303133',
					fontWeight: 'bold',
					transform: 'scale(1.05)'
				}" :inactiveStyle="{
					color: '#606266',
					transform: 'scale(1)'
				}" itemStyle="padding-left: 15px; padding-right: 15px; height: 34px;" :current="tabIndex"
				@click="clickTabbar"></uv-tabs>
		</view>
		<view class="view-content">
			<swiper :indicator-dots="false" :autoplay="false" :current="currentIndex" @change="swiperChange"
				@touchmove.stop class="content-swiper" :style="{height : swiperItemHeight + 'px'}" :key="index">
				<swiper-item v-for="(item,index) of categoryInfo">
					<uni-list v-if="item.collectMap.length >= 1" :id="'uni-list'+index">
						<uni-list-item v-for="(map_item, map_index) of item.collectMap" :key="map_index" :title="map_item.name"
							:note="map_item.description" :thumb="map_item.thumbnailUrl" thumb-size="lg"
							clickable @click="onClick(map_item)"></uni-list-item>
					</uni-list>
					<uni-list v-else>
						<view class="swiper-item uni-bg-blue"><text class="swiper-item-Text"></text></view>
					</uni-list>
				</swiper-item>
			</swiper>
		</view>
	</view>
</template>
<script>
	export default {
		beforeMount() {
			this.initialize();
		},
		mounted() {},
		data() {
			return {
				currentIndex: 0,
				currentCollectHeght: 0,
				swiperItemHeight: 0,
				categoryInfo: [],
				tabIndex: 0,
				all_map: new Map(),
			}
		},


		methods: {
			//挂载前初始化函数
			async initialize() {
				//初始化界面
				await this.getCategoryInfo();
				this.setSwiperItemHeight();
			},
			//tab点击事件
			clickTabbar(item, index) {
				this.currentIndex = item.index;
				if (this.categoryInfo[this.currentIndex]) {
					this.currentCollectNum = this.categoryInfo[this.currentIndex].collectMap.length;
					this.$nextTick(() => {
						this.setSwiperItemHeight();
					});
				} else {
					this.currentCollectNum = 0;
				}
			},
			//swiper滑动事件
			swiperChange(e) {
				this.currentValChange = e.detail.current;
				this.tabIndex = this.currentValChange;
			},
			//swiper ui设计
			setSwiperItemHeight() {
				let element = "#uni-list" + this.currentIndex;
				let query = uni.createSelectorQuery().in(this);
				query.select(element).boundingClientRect();
				query.exec((res) => {
					if (res && res[0]) {
						this.swiperItemHeight = res[0].height;
					}
				})
			},

			//获取所有目录信息
			async getCategoryInfo() {
				let url = 'http://xxx.xxx.xxx.xxx:8080/category/listall/category';
				const response = await uni.request({
					url: url,
					method: 'GET',
					data: {},
					header: {
						'Content-Type': 'application/json'
					}
				});
				//获取所有收藏缓存及其对应地图id信息
				await this.getItemFromStorage(response);
				//基于地图id信息，获取收藏-地图信息
				await this.getMapInfo();
			},

			//获取所有收藏缓存及其对应地图id信息
			async getItemFromStorage(response) {
				for (const item of response.data.data) {
					let list = {
						name: item.name,
						collectMap: [],
						mid: []
					};
					// 获取相应缓存收藏存信息
					let key = item.name;
					const res = await new Promise((resolve, reject) => {
						uni.getStorage({
							key,
							complete: function(res) {
								resolve(res);
							}
						});
					});
					// 获取目录
					list.mid = res.data || []; // 无数据填充默认
					this.categoryInfo.push(list);
				}
			},

			//基于地图id信息，获取收藏-地图信息
			async getMapInfo() {
				const map_res = await uni.request({
					url: 'http://xxx.xxx.xxx.xxx:8080/map/listall',
					method: 'GET',
					data: {},
					header: {
						'Content-Type': 'application/json'
					}
				});

				for (const item of map_res.data.data) {
					this.all_map.set(item.mid, item);
				}
				for (let i = 0; i < this.categoryInfo.length; i++) {
					const item = this.categoryInfo[i];
					if (item.mid.length > 0) {
						for (const index of item.mid) {
							this.categoryInfo[i].collectMap.push(this.all_map.get(index));
						}
					}
				}
			},
			//点击跳转
			onClick(e) { 
				console.log(e);
				let content = {
					mid: e.mid,
					cid: e.cid
				};
				uni.navigateTo({
					url: '/pages/mapDisplay/mapDisplay',
					events: content
				});
			}
		}
	}
</script>

<style>
	.view-all {
		display: flex;
		min-height: 100vh;
		max-width: 540px;
		min-width: 320px;
		margin: 0 auto;
		font: normal 14px/1.5 Tahoma, "Lucida Grande", Verdana, "Microsoft yahei", STXihei, hei;
		color: #000;
		background-color: #f2f2f2;
		overflow-x: hidden;
		-webkit-tap-highlight-color: transparent;
		flex-direction: column;
		overflow: hidden
	}

	.content-swiper {}

	.item-list {}

	.content-list-item-text {
		display: flex;
		justify-content: center;
		align-items: center;
	} 
</style>