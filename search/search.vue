<template>
	<view>
		<view class="search-bar">
			<div class="container">
				<uni-icons type="search" size="30"></uni-icons>
				<uni-combox @input="search" placeholder="输入搜索内容" v-model="searchText" class="flex-grow">
				</uni-combox> 
			</div>

		</view>
		<uni-section v-for="(item, index) in cardInfo" :key="index" :title="item.type === 'map' ? '地图' : '目录'"
			type="line">
			<uni-card :title="item.name" :thumbnail="item.thumbnailUrl ? item.thumbnailUrl : defaultIcon"
				@click="onClick(item)">
				<text class="uni-body">{{ "描述信息： "+item.description }}</text>
			</uni-card>
		</uni-section>
	</view>

</template>

<script>
	import {
		mapState
	} from "vuex";
	import {
		mapMutations
	} from 'vuex';

	export default {
		data() {
			return {
				searchText: '',
				searchResults: [],
				cardInfo: [],
				//卡片信息
				defaultIcon: '/static/icons/catalog.png',
				extraIcon: {
					color: '#4cd964',
					size: '22',
					type: 'gear-filled'
				},
				//demo 存储所有待查询信息
				navList: []
			};
		},
		computed: {
			trimmedResults() {
				const uniqueResults = {};
				const trimmed = [];

				for (let result of this.searchResults) {
					trimmed.push(result);
				}

				return trimmed;
			},
		},
		methods: {
			search() {
				this.$nextTick(() => {
					//关键词搜索
					let url = 'http://xxx.xxx.xxx.xxx:8080/homepage/search/?wd=' + this.searchText+"&atlas=xxx";
					this.cardInfo = [];
					const self = this;
					let results;
					uni.request({
						url: url, // 请求的URL地址
						method: 'GET', // 请求方法，可以是 GET/POST/PUT/DELETE 等
						data: {}, // 请求的参数，如果是GET请求，可以将参数拼接在URL中，如果是POST请求，可以将参数放在这里
						header: {
							'Content-Type': 'application/json' // 请求头信息，可以根据需要设置
						},
						success: function(res) { 
							results = res.data.result;
							// 请求成功时的回调函数，res.data包含了返回的数据 
							self.searchResults = [];
							for (let i = 0; i < results.length; i++) {
								self.searchResults.push(results[i].name);
								self.cardInfo.push(results[i]);
							}
						},
						fail: function(err) {}
					});
				});
			},
			onClick(e) {
				if (e.type == "map") {
					let content = {
						mid: e.mid,
						cid: e.cid
					};
					uni.navigateTo({
						url: '/pages/mapDisplay/mapDisplay',
						events: content
					});
				} else {
					let content = {
						cid: e.cid
					};
					uni.navigateTo({
						url: '/pages/atlasDisplay/atlasDisplay',
						events: content
					});
				}

			} 

		},
		mounted() {

		},
	};
</script>

<style>
	.search-bar {
		width: 100%;
	}

	.container {
		display: flex;
		align-items: center;
		width: 100%;
	}

	.flex-grow {
		flex-grow: 1;
	}
</style>