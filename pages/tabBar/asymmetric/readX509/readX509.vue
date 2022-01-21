<template>
	<view>
		<uni-forms ref="dynamicForm" :rules="dynamicRules" :modelValue="dynamicFormData">
			<uni-section title="step.1 填写公钥证书" type="line">
				<view class="example">
						<uni-forms-item label="公钥证书"  name="cert">
							<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.cert"/>
						</uni-forms-item>
						
						<view class="button-group">
							<button type="primary" size="max" @click="readX509Info()">读取证书信息</button>	
						</view>
						<uni-group v-for="keyInfo in keyInfos" margin-top="20">
							<view> SN：{{keyInfo.serialNumber}} </view>
							<view> DN：{{keyInfo.subject}} </view>
							<view> 类型： {{keyInfo.algorithm=="EC"?"SM2":"RSA"}} </view>
							<view> 签名摘要算法：{{keyInfo.sigAlgName}} </view>
							<view> 颁发者：{{keyInfo.issuer}} </view>
							<view> 证书有效期：<br>{{new Date(keyInfo.notBefore*1000)}}-<br>{{new Date(keyInfo.notAfter*1000)}} </view>
							<view v-if="keyInfo.base64">证书Base64:{{keyInfo.base64}} </view>
						</uni-group>					
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
				pukt:uni.requireNativePlugin('lich-PKI-PublicKeyTool'),
				PKITool:uni.requireNativePlugin('lich-PKI-PKITool'),
				dynamicRules: {
					
				},
				keyInfos:[],
				dynamicFormData:{
					cert:'MIIBfTCCASKgAwIBAgII/wAAAAAAAAAwCgYIKoEcz1UBg3UwHDELMAkGA1UEBhMCQ04xDTALBgNVBAMMBGxpY2gwHhcNMjIwMTEyMDcxMDI3WhcNMjMwMTEyMDcxMDI3WjAeMQswCQYDVQQGEwJDTjEPMA0GA1UEAwwGR01URVNUMFkwEwYHKoZIzj0CAQYIKoEcz1UBgi0DQgAE5YfKi9B2zq90g2FeZnIf3gs3/2Kj/eD87u5WwHLSkhRhRqR/Wkcb1ks+6j51wYyYaV34bbTzXIvdiIKLU1IESKNMMEowCwYDVR0PBAQDAgbAMB0GA1UdJQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDBDAJBgNVHRMEAjAAMBEGCWCGSAGG+EIBAQQEAwIHgDAKBggqgRzPVQGDdQNJADBGAiEA82Xn67rhqwE2ZpK+6alZzkooQCkLs26hIfN6cOQ4j1MCIQD01NKISTC9rN5Ofs6QpLQUg69KL+FKWiwrlna7al4V2w==',	
				}
			}
		},
		methods: {
			readX509Info:function() {
				if(this.dynamicFormData.cert==''){
					this.messageToggle('error','公钥证书不能为空');
					return;
				}
				this.pukt.readX509CertificateKeyInfo(this.dynamicFormData.cert,
				(ret)=>{
					console.log(ret)
					if(ret.code!=0){
						this.messageToggle('error',ret.msg)
						return;
					}
					this.keyInfos=[ret.keyInfo]
					this.messageToggle('success','读取成功');	
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
