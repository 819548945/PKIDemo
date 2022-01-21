<template>
	<view>
		<uni-forms ref="dynamicForm" :rules="dynamicRules" :modelValue="dynamicFormData">
			<uni-section title="step.1 填写秘钥号;原文" type="line">
				<view class="example">
						<uni-forms-item label="秘钥号"  name="keyId">
							<uni-easyinput type="text" v-model="dynamicFormData.keyId"/>
						</uni-forms-item>
						<uni-forms-item label="原文"  name="ori">
							<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.ori"/>
						</uni-forms-item>					
						<view class="button-group">
							<button type="primary" size="max" @click="sign()">签名</button>	
						</view>
						<br>
						<uni-forms-item label="签名值"  name="sign">
							<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.signData"/>
						</uni-forms-item>
						<uni-forms-item label="签名值(RS)"  name="signRS">
							<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.signRSData"/>
						</uni-forms-item>
						<uni-popup ref="message" type="message">
							<uni-popup-message :type="msgType" :message="messageText" :duration="2000"></uni-popup-message>
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
					signData:'',
					signRSData:''
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
				
				console.log(this.dynamicFormData.ori)
				this.PKITool.getUTF8bytes(this.dynamicFormData.ori,
				(ret)=>{
					console.log(ret)
					if(ret.code!=0){
						this.dialogToggle('error',ret.msg)
						return;
					}
					this.asmt.sign(this.dynamicFormData.keyId,ret.bytes,
					(ret1)=>{
						console.log(ret1)
						if(ret1.code!=0){
							this.dialogToggle('error',ret1.msg)
							return;
						}
						this.PKITool.encodeBase64String(ret1.sign,
						(ret2)=>{
							console.log(ret2)
							if(ret2.code!=0){
								this.dialogToggle('error',ret2.msg)
								return;
							}
							this.dynamicFormData.signData=ret2.base64;
							
						})
						this.asmt.SM2SignatureToRS(ret1.sign,
						(ret2)=>{
							console.log(ret2)
							if(ret2.code!=0){
								this.dialogToggle('error',ret2.msg)
								return;
							}
							this.PKITool.encodeBase64String(ret2.RS,
							(ret3)=>{
								console.log(ret3)
								if(ret3.code!=0){
									this.dialogToggle('error',ret3.msg)
									return;
								}
								this.dynamicFormData.signRSData=ret3.base64;
								this.dialogToggle('success','签名成功')
							})
							
						})
						
						
						
						
					})
					
				})
			},
			dialogToggle:function(type,msg) {
				this.msgType=type;
				this.messageText=msg;
				this.$refs.message.open()
			}
		}
	}
</script>

<style>

</style>
