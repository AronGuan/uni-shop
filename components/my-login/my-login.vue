<template>
	<view>
		<view class="login-container">
			<uni-icons type="contact-filled" size="100" color="#AFAFAF"></uni-icons>
			<button type="primary" class="btn-login" @click="getUserProfile">一键登录</button>
			<text class="tips-text">登陆后尽享更多权益</text>
		</view>
	</view>
</template>

<script>
	import {mapMutations,mapState} from 'vuex'
	
	export default {
		name:"my-login",
		data() {
			return {
				
			};
		},
		computed: {
			...mapState('m_user',['redirectInfo'])
		},
		methods: {
			...mapMutations('m_user',['updateUserInfo','updateToken','updateRedirectInfo']),
			async getUserProfile(e){
				const res = await uni.getUserProfile({
					desc: '进行登录',
				})
				if(res[1].errMsg === 'getUserInfo:fail auth deny') return uni.$showMsg('你取消了登录授权!')
				uni.$showMsg(res[1].userInfo)
				this.updateUserInfo(res[1].userInfo)
				console.log(res[1])
				this.getToken(res[1])
			},
			async getToken(info){
				const [err, res] = await uni.login().catch(err => err)
				
				if(err || res.errMsg !== 'login:ok') return uni.$showMsg('登录失败！')
			
				const query = {
					code: res.code,
					encryptedData: info.encryptedData,
					iv: info.iv,
					rawData: info.rawData,
					signature: info.signature
				}
				//没有后端，所以请求会失败
				// const {data: loginResult} = await uni.$http.post('api/public/v1/users/wxlogin',query)
				// console.log(loginResult)
				// if(loginResult.meta.status !== 200) return uni.$showMsg('登录失败!')
				
				// this.updateToken(loginResult.message.token)
				this.updateToken({state:"succ"})
				this.navigateBack()
			},
			navigateBack(){
				console.log(this.redirectInfo.from)
				if(this.redirectInfo && this.redirectInfo.openType === 'switchTab'){
					uni.switchTab({
						url: this.redirectInfo.from,
						complete: () => {
							this.updateRedirectInfo(null)
						}
					})
				}
			}
		}
	}
</script>

<style lang="scss">
.login-container{
	height: 750rpx;
	background-color: white;
	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;
	position: relative;
	overflow: hidden;
	
	&::after {
		content: ' ';
		display: block;
		width: 100%;
		height: 40px;
		background-color: #f8f8f8;
		position: absolute;
		bottom: 0;
		left: 0;
		border-radius: 100%;
		transform: translateY(50%)
	}
	
	.btn-login{
		width: 90%;
		border-radius: 100px;
		margin: 15px 0;
		background-color: #C00000;
	}
	.tips-text{
		font-size: 15px;
		color: gray;
	}
}
</style>