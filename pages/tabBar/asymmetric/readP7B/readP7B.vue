<template>
	<view>
		<uni-forms ref="dynamicForm" :rules="dynamicRules" :modelValue="dynamicFormData">
			<uni-section title="step.1 填写P7B证书链" type="line">
				<view class="example">
						<uni-forms-item label="P7B证书链"  name="p7b">
							<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.p7b"/>
						</uni-forms-item>
						
						<view class="button-group">
							<button type="primary" size="max" @click="readP7bInfo()">读取证书信息</button>	
						</view>
						<uni-group v-for="(keyInfo,index) in keyInfos" :title="'证书:'+index" margin-top="20">
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
					p7b:"MIAGCSqGSIb3DQEHAqCAMIACAQExADCABgkqhkiG9w0BBwEAAKCAMIIBbTCCAROgAwIBAgIGAXbbDsqKMAoGCCqBHM9VAYN1MBwxCzAJBgNVBAYTAkNOMQ0wCwYDVQQDDARsaWNoMB4XDTIwMTIzMTE2MDAwMFoXDTM1MTIzMTE1NTk1OVowHDELMAkGA1UEBhMCQ04xDTALBgNVBAMMBGxpY2gwWTATBgcqhkjOPQIBBggqgRzPVQGCLQNCAAQeyRxm12te20LpKRFOlSiM5dDrcDtfbcTeCNetzeH37VATYu/WM3Wq4vnvb32RCMLqurGfCYx4aqWXL11Vn/dGo0EwPzALBgNVHQ8EBAMCBsAwHQYDVR0lBBYwFAYIKwYBBQUHAwEGCCsGAQUFBwMEMBEGCWCGSAGG+EIBAQQEAwIHgDAKBggqgRzPVQGDdQNIADBFAiEA27AoJSd+Ra9MvOLwr6eW1wTpxSzqL35gU+wjzy5/SVACIE8RPYc9pXxnWI6+hdsduNosJRVvpKow/ofBmQKON4QXMIIBejCCASCgAwIBAgIGAX3gOOCJMAoGCCqBHM9VAYN1MBwxCzAJBgNVBAYTAkNOMQ0wCwYDVQQDDARsaWNoMB4XDTIwMTIzMTE2MDAwMFoXDTIxMTIzMTE1NTk1OVowHjELMAkGA1UEBhMCQ04xDzANBgNVBAMMBkdNVEVTVDBZMBMGByqGSM49AgEGCCqBHM9VAYItA0IABGnzVxSpFrT1DCHVA98ojOczDHqCHMUAp9jGLDA6qbNmw9Hi0NOk6GEhxacSavFPqb+r8IiMbWmSz/Oz3Yhd02WjTDBKMAsGA1UdDwQEAwIGwDAdBgNVHSUEFjAUBggrBgEFBQcDAQYIKwYBBQUHAwQwCQYDVR0TBAIwADARBglghkgBhvhCAQEEBAMCB4AwCgYIKoEcz1UBg3UDSAAwRQIgN+PN73NMQe6zb+BSgqq9NzE36UvlZ4CbiqXBlZI8fB8CIQDjB8GyCW18t6ihjYyAomdQ+oEoHfp7sCIkPONHjY+mKwAAMQAAAAAAAAA=",
				}
			}
		},
		methods: {
			readP7bInfo:function() {
				if(this.dynamicFormData.p7b==''){
					this.messageToggle('error','p7b不能为空');
					return;
				}
				this.pukt.p7bToX509CertificateArray(this.dynamicFormData.p7b,
				(ret)=>{
					console.log(ret)
					if(ret.code!=0){
						this.messageToggle('error',ret.msg)
						return;
					}
					this.keyInfos=[]
					ret.certs.forEach((d)=>{
						this.pukt.readX509CertificateKeyInfo(d,
						(ret)=>{
							console.log(ret)
							ret.keyInfo.base64=d;
							this.keyInfos.push(ret.keyInfo)
						})
						
					})
					
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
