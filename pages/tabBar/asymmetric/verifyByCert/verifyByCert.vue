<template>
	<view>
		<uni-forms ref="dynamicForm" :rules="dynamicRules" :modelValue="dynamicFormData">
			<uni-section title="step.1 填写公钥证书;原文;签名值" type="line">
				<view class="example">
						<uni-forms-item label="公钥证书"  name="cert">
							<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.cert"/>
						</uni-forms-item>
						<uni-forms-item label="原文"  name="ori">
							<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.ori"/>
						</uni-forms-item>
						<uni-forms-item label="签名值"  name="signData">
							<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.signData"/>
						</uni-forms-item>
						<view class="button-group">
							<button type="primary" size="max" @click="verify()">签名验证</button>	
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
					
				},
				dynamicFormData:{
					cert:'MIIBfTCCASKgAwIBAgII/wAAAAAAAAAwCgYIKoEcz1UBg3UwHDELMAkGA1UEBhMCQ04xDTALBgNVBAMMBGxpY2gwHhcNMjIwMTEyMDcxMDI3WhcNMjMwMTEyMDcxMDI3WjAeMQswCQYDVQQGEwJDTjEPMA0GA1UEAwwGR01URVNUMFkwEwYHKoZIzj0CAQYIKoEcz1UBgi0DQgAE5YfKi9B2zq90g2FeZnIf3gs3/2Kj/eD87u5WwHLSkhRhRqR/Wkcb1ks+6j51wYyYaV34bbTzXIvdiIKLU1IESKNMMEowCwYDVR0PBAQDAgbAMB0GA1UdJQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDBDAJBgNVHRMEAjAAMBEGCWCGSAGG+EIBAQQEAwIHgDAKBggqgRzPVQGDdQNJADBGAiEA82Xn67rhqwE2ZpK+6alZzkooQCkLs26hIfN6cOQ4j1MCIQD01NKISTC9rN5Ofs6QpLQUg69KL+FKWiwrlna7al4V2w==',
					ori:'测试原文',
					signData:'MEYCIQDGnCykllu1QmjWtkx+0aBupp3DvPdqjtyFeXa/1IpN8AIhAP5mbLSCgx1US99oZxBEpwcLGH5Ujp9GHQhStrXosrq3'
					
				}
			}
		},
		methods: {
			verify:function() {
				if(this.dynamicFormData.cert==''){
					this.messageToggle('error','公钥证书不能为空');
					return;
				}
				if(this.dynamicFormData.ori==''){
					this.messageToggle('error','原文不能为空');
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
						this.asmt.verify(ret.bytes,ret1.bytes,this.dynamicFormData.cert,
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
