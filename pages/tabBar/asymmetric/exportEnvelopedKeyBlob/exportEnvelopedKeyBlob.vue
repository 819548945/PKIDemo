<template>
	<view>
		<uni-forms ref="dynamicForm" :rules="dynamicRules" :modelValue="dynamicFormData">
			<uni-section title="step.1 填写秘钥号以及导入方秘钥公钥证书" type="line">
				<view class="example">
					<uni-forms-item label="秘钥号"  name="keyId">
						<uni-easyinput type="text" maxlength="-1" v-model="dynamicFormData.keyId"/>
					</uni-forms-item>
						
						<uni-forms-item label="导入方秘钥公钥证书"  name="importCert">
							<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.importCert"/>
						</uni-forms-item>
						<view class="button-group">
							<button type="primary" size="max" @click="exportEnvelopedKeyBlob()">导出秘钥</button>	
						</view>
						<br>
						<uni-forms-item label="envelopedKeyBlob"  name="envelopedKeyBlob">
							<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.envelopedKeyBlob"/>
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
					envelopedKeyBlob:"",
					importCert:'MIIBfTCCASKgAwIBAgII/wAAAAAAAAAwCgYIKoEcz1UBg3UwHDELMAkGA1UEBhMCQ04xDTALBgNVBAMMBGxpY2gwHhcNMjIwMTEyMDcxMDI3WhcNMjMwMTEyMDcxMDI3WjAeMQswCQYDVQQGEwJDTjEPMA0GA1UEAwwGR01URVNUMFkwEwYHKoZIzj0CAQYIKoEcz1UBgi0DQgAE5YfKi9B2zq90g2FeZnIf3gs3/2Kj/eD87u5WwHLSkhRhRqR/Wkcb1ks+6j51wYyYaV34bbTzXIvdiIKLU1IESKNMMEowCwYDVR0PBAQDAgbAMB0GA1UdJQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDBDAJBgNVHRMEAjAAMBEGCWCGSAGG+EIBAQQEAwIHgDAKBggqgRzPVQGDdQNJADBGAiEA82Xn67rhqwE2ZpK+6alZzkooQCkLs26hIfN6cOQ4j1MCIQD01NKISTC9rN5Ofs6QpLQUg69KL+FKWiwrlna7al4V2w=='
					
				}
			}
		},
		methods: {
			exportEnvelopedKeyBlob:function() {
				if(this.dynamicFormData.keyId==''){
					this.messageToggle('error','秘钥号不能为空');
					return;
				}
				if(this.dynamicFormData.importCert==''){
					this.messageToggle('error','导入方秘钥公钥证书');
					return;
				}
				console.log(this.dynamicFormData.keyId)
				console.log(this.dynamicFormData.importCert)
				this.kpt.exportEnvelopedKeyBlob(this.dynamicFormData.keyId,1,this.dynamicFormData.importCert,
				(ret)=>{
					console.log(ret)
					if(ret.code!=0){
						this.messageToggle('error',ret.msg)
						return;
					}
					this.dynamicFormData.envelopedKeyBlob=ret.envelopedKeyBlob;
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
