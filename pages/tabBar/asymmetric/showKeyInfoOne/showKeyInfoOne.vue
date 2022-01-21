<template>
	<view class="container">
		<uni-forms ref="dynamicForm" :rules="dynamicRules" :modelValue="dynamicFormData">
			<uni-section title="step.1 填写秘钥号" type="line">
				<view class="example">
						<uni-forms-item label="秘钥号"  name="keyId">
							<uni-easyinput type="text" v-model="dynamicFormData.keyId"/>
						</uni-forms-item>
						<view class="button-group">
							<button type="primary" size="max" @click="read()">读取信息</button>	
						</view>
						<br>
				</view>
			</uni-section>
		</uni-forms>	
		<uni-popup ref="message" type="message">
			<uni-popup-message :type="msgType" :message="messageText" :duration="2000"></uni-popup-message>
		</uni-popup>
		<uni-popup ref="alertDialog" type="dialog">
			<uni-popup-dialog :type="msgType" title="" :content="messageText" @confirm="destroy()"
			></uni-popup-dialog>
		</uni-popup>
		<uni-group v-if="keyInfo"  :title="'KeyId:'+keyInfo.keyId" margin-top="20">
			<view class="example-body"> 类型： {{keyInfo.type=="EC"?"SM2":"RSA"}}({{keyInfo.mode==1?"单证":"双证"}}) </view>
			<view class="example-body"> 秘钥状态：
				<uni-fav :checked="keyInfo.available" :content-text="{contentDefault: '停用',contentFav: '可用'}" :star="false" circle="true"  bg-color="#dd524d" bg-color-checked="#007aff" @click="setAvailable()"/> 	
			</view>
			<view class="example-body"> 缓存模式：
				<uni-fav :checked="keyInfo.isCache" :content-text="{contentDefault: '否',contentFav: '是'}" :star="false" circle="true"  bg-color="#dd524d" bg-color-checked="#007aff" @click="setIsCache()"/>
			</view>
			<view class="example-body"> 秘钥对状态：
				<span v-if="keyInfo.mode==1">{{keyInfo.cert?"可用":"不可用"}}</span>
				<span v-else >
					签名证书:{{keyInfo.cert?"可用":"不可用"}}&nbsp;
					加密私钥:{{keyInfo.encPrivateKey?"可用":"不可用"}}&nbsp;
					加密证书:{{keyInfo.encCert?"可用":"不可用"}}
				</span>
			</view>
			<view class="example-body">
				<view class="button-group">
					<button type="warn" size="mini" @click="destroy0()">销毁</button>	
				</view>
			</view>
		</uni-group>
	</view>
</template>

<script>
	export default {
		
		data() {
			return {
				type: 'center',
				msgType: 'success',
				messageText: '',
				kpt:uni.requireNativePlugin('lich-PKI-KeyPairTool'),
				keyInfo:null,
				dynamicRules: {
					keyId: {
						rules: [{
							required: true,
							errorMessage: '秘钥号不能为空'
						}]
					}
				},
				dynamicFormData:{
					keyId:''
				}
			}
			
		},
		methods: {
			setAvailable:function(){
				this.kpt.setAvailable(this.dynamicFormData.keyId,!this.keyInfo.available,
				(ret)=>{
					console.log(ret)
					if(ret.code!=0){
						this.messageToggle('error',ret.msg)
						return;
					}
					this.keyInfo.available=!this.keyInfo.available
				})	
				
			},
			setIsCache:function(){
				this.kpt.openCache(this.dynamicFormData.keyId,!this.keyInfo.isCache,
				(ret)=>{
					console.log(ret)
					if(ret.code!=0){
						this.messageToggle('error',ret.msg)
						return;
					}
					this.keyInfo.isCache=!this.keyInfo.isCache
				})	
				
			},
			read:function(){
				if(this.dynamicFormData.keyId==''){
					this.messageToggle('error','秘钥号不能为空');
					return;
				}
				this.kpt.getKeyInfo(this.dynamicFormData.keyId,
				(ret)=>{
					console.log(ret)
					if(ret.code!=0){
						this.messageToggle('error',ret.msg)
						return;
					}
					this.keyInfo=ret.data
				})	
			},
			destroy0:function(){
				this.dialogToggle("warn","销毁后秘钥将无法恢复")
			},
			destroy:function(){
				this.kpt.destroy(this.dynamicFormData.keyId);
				this.keyInfo=null;
			},
			messageToggle:function(type,msg) {
				this.msgType=type;
				this.messageText=msg;
				this.$refs.message.open()
			},
			dialogToggle:function(type,msg) {
				this.msgType=type;
				this.messageText=msg;
				this.$refs.alertDialog.open()
			}
		}
	}
</script>

<style lang="scss">
	.example-body {
		display: flex;
		padding: 10px 15px;
	}

	/* #ifdef MP-ALIPAY */
	.uni-fav {
		margin-left: 20rpx;
	}

	/* #endif */


	.favBtn {
		margin: 0 20rpx 20rpx 0;
	}


	.example-body-fullWidth {
		padding: 32rpx 0;
	}

	.example-body-first {
		/* #ifndef APP-PLUS-NVUE */
		display: flex;
		/* #endif */
		flex-direction: row;
		justify-content: flex-start;
	}

	.favBtn-nav {
		// left:-50rpx;
	}
</style>
