<template>
	<view>
		<uni-forms ref="dynamicForm" :rules="dynamicRules" :modelValue="dynamicFormData">
			<uni-section title="step.1 填写秘钥号;原文;签名值" type="line">
				<view class="example">
						<uni-forms-item label="秘钥号"  name="keyId">
							<uni-easyinput type="text" v-model="dynamicFormData.keyId"/>
						</uni-forms-item>
						<uni-forms-item label="原文"  name="ori">
							<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.ori"/>
						</uni-forms-item>
						<uni-forms-item label="签名值"  name="ori">
							<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.signData"/>
						</uni-forms-item>
						<view class="button-group">
							<button type="primary" size="max" @click="sign()">签名验证</button>	
						</view>
						<br>
						<uni-popup ref="message" type="message">
							<uni-popup-message :type="msgType" :message="messageText" :duration="2000"></uni-popup-message>
						</uni-popup>
						<uni-popup ref="alertDialog" type="dialog">
							<uni-popup-dialog :type="msgType" title="" :content="messageText" 
							></uni-popup-dialog>
						</uni-popup>
				</view>
			</uni-section>
		</uni-forms>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				type: 'center',
				msgType: 'success',
				messageText: '',
				asmt:uni.requireNativePlugin('lich-PKI-AsymmetricTool'),
				PKITool:uni.requireNativePlugin('lich-PKI-PKITool'),
				dynamicRules: {
					keyId: {
						rules: [{
							required: true,
							errorMessage: '秘钥号不能为空'
						}]
					}
				},
				dynamicFormData:{
					keyId:'',
					ori:'测试原文',
					signData:''
				}
			}
		},
		methods: {
			sign:function() {
				if(this.dynamicFormData.keyId==''){
					this.dialogToggle('error','秘钥号不能为空');
					return;
				}
				if(this.dynamicFormData.ori==''){
					this.dialogToggle('error','原文不能为空');
					return;
				}
				if(this.dynamicFormData.signData==''){
					this.messageToggle('error','签名值不能为空');
					return;
				}
				this.PKITool.decodeBase64(this.dynamicFormData.signData,
				(ret)=>{
					console.log(ret)
					if(ret.code!=0){
						this.messageToggle('error',ret.msg)
						return;
					}
					this.PKITool.getUTF8bytes(this.dynamicFormData.ori,
					(ret1)=>{
						console.log(ret1)
						if(ret1.code!=0){
							this.messageToggle('error',ret1.msg)
							return;
						}
						console.log(this.dynamicFormData.keyId)
						console.log(ret.bytes)
						console.log(ret1.bytes)
						this.asmt.verifyByKeyId(this.dynamicFormData.keyId,ret.bytes,ret1.bytes,
						(ret2)=>{
							console.log(ret2)
							if(ret2.code!=0){
								this.messageToggle('error',ret2.msg)
								return;
							}
							if(ret2.verify){
								this.dialogToggle('success','签名验证成功')
							}else{
								this.dialogToggle('error','签名验证失败')
							}
						})
					})
				})
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

<style>

</style>
