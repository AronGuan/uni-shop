<template>
	<view>
		<view class="search-box">
			<uni-search-bar @input="input" :radius="100" placeholder="请输入搜索内容" cancelButton="none"></uni-search-bar>
		</view>
		
		<view class="sugg-list" v-if="searchResults.length !== 0">
			<view class="sugg-item" v-for="(item,i) in searchResults" :key="i" @click="gotoDetail(item)">
				<view class="goods_name">{{item.goods_name}}</view>
				<uni-icons type="arrowright" siz="16"></uni-icons>
			</view>
		</view>
		
		<view class="history-box" v-else>
			<view class="history-title">
				<text>搜索历史</text>
				<uni-icons type="trash" size="20" @click="clean"></uni-icons>
			</view>
			<view class="history-list">
				<uni-tag :text="item" v-for="(item,i) in histories" :key="i" @click="gotoGoodsList(item)">
				</uni-tag>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				timer: null,
				kw: '',
				searchResults: [],
				historyList: []
			};
		},
		onLoad(){
			this.historyList = JSON.parse(uni.getStorageSync('kw') || '[]')
		},
		methods: {
			input(e){
				clearTimeout(this.timer)
				
				this.timer = setTimeout(() => {
					this.kw = e
					this.getSearchList()
				},500)	
			},
			async getSearchList(){
				if(this.kw.length === 0){
					this.searchResults = []
					return
				}
				
				const {data: res} = await uni.$http.get('/api/public/v1/goods/qsearch',{query:this.kw})
				if(res.meta.status !== 200) return uni.$showMsg()
				this.searchResults = res.message
				
				this.saveSearchHistory()
			},
			gotoDetail(item){
				console.log(item.goods_id)
				uni.navigateTo({
					url: '/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id
				})
			},
			saveSearchHistory(){
				// this.historyList.push(this.kw)
				const set = new Set(this.historyList)
				set.delete(this.kw)
				set.add(this.kw)
				
				this.historyList = Array.from(set)
				
				uni.setStorageSync("kw",JSON.stringify(this.historyList))
			},
			clean(){
				this.historyList = []
				uni.setStorageSync('kw','[]')
			},
			gotoGoodsList(kw){
				uni.navigateTo({
					url: '/subpkg/goods_list/goods_list?query='+ kw
				})
			}
		},
		computed: {
			histories(){
				return [...this.historyList].reverse()
			}
		}
	}
</script>

<style lang="scss">

.search-box{
	background-color: #C00000;
	position: sticky;
	top: 0;
	z-index: 999;
}

.sugg-list{
	padding: 0 5px;
	
	.sugg-item{
		display: flex;
		align-items: center;
		justify-content: space-between;
		font-size: 12px;
		padding: 13px 0;
		border-bottom: 1px solid #efefef;
		
		.goods_name{
			white-space: nowrap;
			overflow: hidden;
			text-overflow: ellipsis;
		}
	}
}

.history-box{
	padding: 0 5px;
	
	.history-title {
		display: flex;
		justify-content: space-between;
		height: 40px;
		align-items: center;
		font-size: 13px;
		border-bottom: 1px solid #efefef;
	}
	
	.history-list{
		display: flex;
		flex-wrap: wrap;
		
		.uni-tag{
			margin-top: 5px;
			margin-right: 5px;
		}
	}
}
</style>
