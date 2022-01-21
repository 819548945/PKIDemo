<template>
	<view>
		<uni-forms ref="dynamicForm" :rules="dynamicRules" :modelValue="dynamicFormData">
			<uni-section title="step.1 填写秘钥号以及密码" type="line">
				<view class="example">
					<uni-forms-item label="秘钥号"  name="keyId">
						<uni-easyinput type="text" maxlength="-1" v-model="dynamicFormData.keyId"/>
					</uni-forms-item>
						
						<uni-forms-item label="密码"  name="pwd">
							<uni-easyinput type="text" maxlength="-1" v-model="dynamicFormData.pwd"/>
						</uni-forms-item>
						<view class="button-group">
							<button type="primary" size="max" @click="exportPFX()">导出秘钥</button>	
						</view>
						<br>
						<uni-forms-item label="pfx"  name="pfx">
							<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.pfx"/>
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
					pfx:"",
					pwd:"123456"
				}
			}
		},
		methods: {
			exportPFX:function() {
				if(this.dynamicFormData.keyId==''){
					this.messageToggle('error','秘钥号不能为空');
					return;
				}
				if(this.dynamicFormData.pwd==''){
					this.messageToggle('error','密码不能为空');
					return;
				}
				this.kpt.exportRSAP12(this.dynamicFormData.keyId,this.dynamicFormData.pwd,"",
				(ret)=>{
					console.log(ret)
					if(ret.code!=0){
						this.messageToggle('error',ret.msg)
						return;
					}
					this.dynamicFormData.pfx=ret.P12;
					this.messageToggle('success','导出成功');	
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
