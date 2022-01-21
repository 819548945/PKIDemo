<template>
	<view>
		<uni-forms ref="dynamicForm" :rules="dynamicRules" :modelValue="dynamicFormData">
			<uni-section title="step.1 填写PFX以及密码" type="line">
				<view class="example">
						<uni-forms-item label="pfx"  name="pfx">
							<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.pfx"/>
						</uni-forms-item>
						<uni-forms-item label="密码"  name="pwd">
							<uni-easyinput type="text" maxlength="-1" v-model="dynamicFormData.pwd"/>
						</uni-forms-item>
						<view class="button-group">
							<button type="primary" size="max" @click="importPFX()">导入秘钥</button>	
						</view>
						<br>
						<uni-forms-item label="秘钥号"  name="keyId">
							<uni-easyinput type="text" maxlength="-1" v-model="dynamicFormData.keyId"/>
						</uni-forms-item>				
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
				kpt:uni.requireNativePlugin('lich-PKI-KeyPairTool'),
				PKITool:uni.requireNativePlugin('lich-PKI-PKITool'),
				dynamicRules: {
					keyId: {
						rules: [{
							required: true,
							errorMessage: '秘钥号不能为空'
						}]
					},
					pwd: {
						rules: [{
							required: true,
							errorMessage: '密码不能为空'
						}]
					}
				},
				keyInfos:[],
				dynamicFormData:{
					keyId:"",
					pfx:"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",
					pwd:"123456"
				}
			}
		},
		methods: {
			importPFX:function() {
				if(this.dynamicFormData.pfx==''){
					this.messageToggle('error','pfx不能为空');
					return;
				}
				if(this.dynamicFormData.pwd==''){
					this.messageToggle('error','密码不能为空');
					return;
				}
				this.kpt.importRSAP12(this.dynamicFormData.pfx,this.dynamicFormData.pwd,
				(ret)=>{
					console.log(ret)
					if(ret.code!=0){
						this.messageToggle('error',ret.msg)
						return;
					}
					this.dynamicFormData.keyId=ret.keyId;
					this.messageToggle('success','导入成功');	
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
