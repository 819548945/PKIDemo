<template>
	<view>
		<uni-forms ref="dynamicForm" :rules="dynamicRules" :modelValue="dynamicFormData">
			<uni-section title="step.1 填写公钥证书;待加密数据" type="line">
				<view class="example">
						<uni-forms-item label="公钥证书"  name="cert">
							<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.cert"/>
						</uni-forms-item>
						<view class="button-group">
							<button type="primary" size="max" @click="readAlgorithm()">读取支持摘要算法</button>	
						</view>
						<br>
						<uni-forms-item label="摘要算法"  name="algorithm">
							<uni-data-picker popup-title="请选择摘要算法" :localdata="algorithms"  v-model="dynamicFormData.algorithm" >			
							</uni-data-picker>
						</uni-forms-item>
						<uni-forms-item label="待加密数据"  name="ori">
							<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.ori"/>
						</uni-forms-item>
						
						
						
						
						<view class="button-group">
							<button type="primary" size="max" @click="encByCert()">加密</button>	
						</view>
						<br>
						<uni-forms-item label="加密数据"  name="signData">
							<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.encData"/>
						</uni-forms-item>
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
				PKITool:uni.requireNativePlugin('lich-PKI-PKITool'),
				pukt:uni.requireNativePlugin('lich-PKI-PublicKeyTool'),
				asmt:uni.requireNativePlugin('lich-PKI-AsymmetricTool'),
				PKITool:uni.requireNativePlugin('lich-PKI-PKITool'),
				dynamicRules: {
					
				},
				gmAlgorithms:[],
				rsaAlgorithms:[],
				algorithms:[],
				dynamicFormData:{
					cert:'MIIBfTCCASKgAwIBAgII/wAAAAAAAAAwCgYIKoEcz1UBg3UwHDELMAkGA1UEBhMCQ04xDTALBgNVBAMMBGxpY2gwHhcNMjIwMTEyMDcxMDI3WhcNMjMwMTEyMDcxMDI3WjAeMQswCQYDVQQGEwJDTjEPMA0GA1UEAwwGR01URVNUMFkwEwYHKoZIzj0CAQYIKoEcz1UBgi0DQgAE5YfKi9B2zq90g2FeZnIf3gs3/2Kj/eD87u5WwHLSkhRhRqR/Wkcb1ks+6j51wYyYaV34bbTzXIvdiIKLU1IESKNMMEowCwYDVR0PBAQDAgbAMB0GA1UdJQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDBDAJBgNVHRMEAjAAMBEGCWCGSAGG+EIBAQQEAwIHgDAKBggqgRzPVQGDdQNJADBGAiEA82Xn67rhqwE2ZpK+6alZzkooQCkLs26hIfN6cOQ4j1MCIQD01NKISTC9rN5Ofs6QpLQUg69KL+FKWiwrlna7al4V2w==',
					ori:'测试原文',
					encData:'',
					algorithm:''
				}
			}
		},
		watch: {
		  'dynamicFormData.cert': {
			  handler(newVal) {
				 this.dynamicFormData.algorithm=''
				 this.algorithms=[]
			  },
			  deep: true
		  }
		 },
		onLoad(){
			this.PKITool.getProviderMode(
			(ret)=>{
				this.gmAlgorithms=[];
				let algorithm=ret.ProviderMode.Asymmetric.GM.Cipher
				let x;
				for (x in algorithm) this.gmAlgorithms.push({text:algorithm[x],value:algorithm[x]})
				algorithm=ret.ProviderMode.Asymmetric.RSA.Cipher
				for (x in algorithm) this.rsaAlgorithms.push({text:algorithm[x],value:algorithm[x]})
			})
		},
		
		methods: {
			encByCert:function() {
				if(this.dynamicFormData.cert==''){
					this.messageToggle('error','公钥证书不能为空');
					return;
				}
				if(this.dynamicFormData.algorithm==''){
					this.messageToggle('error','摘要算法不能为空');
					return;
				}
				
				
				if(this.dynamicFormData.ori==''){
					this.messageToggle('error','待加密数据不能为空');
					return;
				}
				this.PKITool.getUTF8bytes(this.dynamicFormData.ori,
				(ret1)=>{
					console.log(ret1)
					if(ret1.code!=0){
						this.messageToggle('error',ret1.msg)
						return;
					}
					console.log(uni.getStorageSync('ProviderMode'))
					this.asmt.encrypt(ret1.bytes,this.dynamicFormData.cert,this.dynamicFormData.algorithm,
					(ret2)=>{
						console.log(ret2)
						if(ret2.code!=0){
							this.messageToggle('error',ret2.msg)
							return;
						}
						this.PKITool.encodeBase64String(ret2.enc,
						(ret3)=>{
							console.log(ret3)
							if(ret3.code!=0){
								this.messageToggle('error',ret3.msg)
								return;
							}
							this.dynamicFormData.encData=ret3.base64;
						})
					})
					
				})
				
			},
			readAlgorithm:function() {
				if(this.dynamicFormData.cert==''){
					this.messageToggle('error','公钥证书不能为空');
					return;
				}
				this.pukt.readX509CertificateKeyInfo(this.dynamicFormData.cert,
				(ret1)=>{
					console.log(ret1)
					if(ret1.code!=0){
						this.messageToggle('error',ret1.msg)
						return;
					}
					this.algorithms=ret1.keyInfo.algorithm=="EC"?this.gmAlgorithms:this.rsaAlgorithms	
					this.dynamicFormData.algorithm=this.algorithms[1].value
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
