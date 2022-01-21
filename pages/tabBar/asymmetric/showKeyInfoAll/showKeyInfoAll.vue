<template>
	<view class="container">
		<uni-group v-for="keyInfo in keyInfos" :title="'KeyId:'+keyInfo.keyId" margin-top="20">
			<view> 类型： {{keyInfo.type=="EC"?"SM2":"RSA"}}({{keyInfo.mode==1?"单证":"双证"}}) </view>
			<view> 秘钥状态：{{keyInfo.available?"可用":"停用"}} </view>
			<view> 缓存模式：{{keyInfo.isCache?"是":"否"}} </view>
			<view> 秘钥对状态：
				<span v-if="keyInfo.mode==1">{{keyInfo.cert?"可用":"不可用"}}</span>
				<span v-else >
					签名证书:{{keyInfo.cert?"可用":"不可用"}}  &nbsp;
					加密私钥:{{keyInfo.encPrivateKey?"可用":"不可用"}}  &nbsp;
					加密证书:{{keyInfo.encCert?"可用":"不可用"}}
				</span>
			</view>
		</uni-group>
		<uni-popup ref="message" type="message">
			<uni-popup-message :type="msgType" :message="messageText" :duration="2000"></uni-popup-message>
		</uni-popup>
	</view>
	
</template>

<script>
	export default {
		onShow() {
			this.kpt.getAllKeyInfo(
			(ret)=>{
				console.log(ret)
				if(ret.code!=0){
					this.messageToggle('error',ret.msg)
					return;
				}
				this.keyInfos=ret.data
			})
		},
		data() {
			return {
				keyInfos:[],
				kpt:uni.requireNativePlugin('lich-PKI-KeyPairTool'),
				messageText:"",
				msgType:""
			}
		},
		methods: {
			messageToggle:function(type,msg) {
				this.msgType=type;
				this.messageText=msg;
				this.$refs.message.open()
			}
		}
	}
</script>

<style>

</style>
