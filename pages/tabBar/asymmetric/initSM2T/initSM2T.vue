<template>
	<uni-forms ref="dynamicForm" :rules="dynamicRules" :modelValue="dynamicFormData">
	<uni-section title="step.1 生成签名秘钥对" type="line">
		<view class="example">
				<uni-forms-item label="秘钥号"  name="keyId">
					<uni-easyinput v-model="dynamicFormData.keyId"/>
				</uni-forms-item>
				<view class="button-group">
					<button type="primary" size="mini" @click="generateKeyPair()">1生成新秘钥</button>
					<button type="primary" size="mini" @click="getPublicKey()">1读取已有秘钥</button>
				</view>
				<br>
		</view>
	</uni-section>
	<uni-section title="step.2 生成P10申请或公钥,并通过CA发证" type="line">
		<view class="example">
				<uni-forms-item label="P10/公钥"  name="publicKey">
					<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.p10"/>
				</uni-forms-item>
				<view class="button-group">
					<button type="primary" size="mini" @click="generateP10()" >2生成P10</button>
					<button type="primary" size="mini" @click="getPublicKey()">2导出公钥</button>
				</view>	
				<br>
		</view>
	</uni-section>
	<uni-section title="step.3 导入签名,加密公钥证书,以及加密秘钥保护结构(CA返回)" type="line">
		<view class="example">
				<uni-forms-item required label="签名公钥证书"  name="signCert">
					<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.signCert"/>
				</uni-forms-item>
				<uni-forms-item required label="加密公钥证书"  name="signCert">
					<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.encCert"/>
				</uni-forms-item>
				<uni-forms-item required label="加密秘钥秘钥保护结构"  name="envelopedKeyBlob">
					<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.envelopedKeyBlob"/>
				</uni-forms-item>
				<view class="button-group">
					<button type="primary" size="mini" @click="generateTest()" >3生成测试导入数据</button>
					<button type="primary" size="mini" @click="importData()" >4导入数据</button>
				</view>	
				<br>
		</view>
	</uni-section>
	<uni-popup ref="message" type="message">
		<uni-popup-message :type="msgType" :message="messageText" :duration="2000"></uni-popup-message>
	</uni-popup>
	</uni-forms>
</template>
<script>
	export default {
		data() {
			return {
				type: 'center',
				msgType: 'success',
				messageText: '这是一条成功提示',
				kpt:uni.requireNativePlugin('lich-PKI-KeyPairTool'),
				pukt:uni.requireNativePlugin('lich-PKI-PublicKeyTool'),
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
					p10:'',
					signCert:'',
					encCert:'',
					envelopedKeyBlob:''
				}
			}
		},
		methods: {
			generateKeyPair:function (){
				this.kpt.generateGMKeyPair((ret)=>{
					console.log(ret)
					if(ret.code!=0){
						this.dialogToggle('error',ret.msg)
						return;
					}
					this.dynamicFormData.keyId=ret.keyId;
					this.dynamicFormData.p10=ret.P;
					this.dialogToggle('success','秘钥对生成成功')
				})
			},
			generateP10:function(){
				if(this.dynamicFormData.keyId==''){
					this.dialogToggle('error','秘钥号不能为空');
					return;
				}
				this.kpt.generateP10(this.dynamicFormData.keyId,"C=CN,CN=SM2TEST","sm3withsm2",
				(ret)=>{
					console.log(ret)
					if(ret.code!=0){
						this.dialogToggle('error',ret.msg)
						return;
					}
					this.dynamicFormData.p10=ret.P10;
					this.dialogToggle('success','P10申请成功')
				})
			},
			getPublicKey:function(){
				this.kpt.getPublicKeyByte(this.dynamicFormData.keyId,1,
				(ret)=>{
					console.log(ret)
					if(ret.code!=0){
						this.dialogToggle('error',ret.msg)
						return;
					}
					this.dynamicFormData.p10=ret.P;
					this.dialogToggle('success','公钥获取成功')
				})		
			},
			generateTest:function(){
				if(this.dynamicFormData.keyId==''){
					this.dialogToggle('error','秘钥号不能为空');
					return;
				}
				let publicKeyInfo={
					serialNumber:"ff1e1112",
					notBefore:parseInt(new Date().getTime()/1000),
					notAfter:parseInt(new Date().getTime()/1000+60*60*24*365),
					subject:"C=CN,CN=SM2TEST",
					signatureAlgorithm:"sm3withsm2"
				}
				console.log(publicKeyInfo)
				this.pukt.generateX509Certificate(this.dynamicFormData.keyId,1,publicKeyInfo,(ret)=>{
					console.log(ret)
					if(ret.code!=0){
						this.dialogToggle('error',ret.msg)
						return;
					}
					this.dynamicFormData.signCert=ret.cert
					this.kpt.getPublicKeyByte(this.dynamicFormData.keyId,1,
					(ret1)=>{
						console.log(ret1)
						if(ret1.code!=0){
							this.dialogToggle('error',ret1.msg)
							return;
						}
						
						this.kpt.generateGMKeyPairExportEnvelopedKeyBlob(ret1.P,
						(ret2)=>{
							console.log(ret2)
							if(ret2.code!=0){
								this.dialogToggle('error',ret2.msg)
								return;
							}
							this.dynamicFormData.envelopedKeyBlob=ret2.envelopedKeyBlob
							
							publicKeyInfo.serialNumber="ff1e1113"
							this.pukt.generateX509CertificateByP(ret2.P,publicKeyInfo,
							(ret3)=>{
								console.log(ret3)
								if(ret3.code!=0){
									this.dialogToggle('error',ret3.msg)
									return;
								}
								this.dynamicFormData.encCert=ret3.cert;
								this.dialogToggle('success',"生成成功")
							})
						})	
					})
				})	
			},
			importData:function(){
				if(this.dynamicFormData.keyId==''){
					this.dialogToggle('error','秘钥号不能为空');
					return;
				}
				if(this.dynamicFormData.cert==''){
					this.dialogToggle('error','公钥证书不能为空');
					return;
				}
				this.kpt.importX509Certificate(this.dynamicFormData.keyId,this.dynamicFormData.signCert,0,
				(ret)=>{
					console.log(ret)
					if(ret.code!=0){
						this.dialogToggle('error',ret.msg)
						return;
					}
					this.kpt.setMode(this.dynamicFormData.keyId,2,
					(ret1)=>{
						console.log(ret1)
						if(ret.code!=0){
							this.dialogToggle('error',ret1.msg)
							return;
						}
					
					});
					
					this.kpt.importGMEncPrivateKeyByEnvelopedKeyBlob(this.dynamicFormData.keyId,this.dynamicFormData.envelopedKeyBlob,this.dynamicFormData.encCert,
						(ret2)=>{
							console.log(ret2)
							if(ret2.code!=0){
								this.dialogToggle('error',ret2.msg)
								return;
							}
							this.dialogToggle('success',"导入成功")
						}
					)
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
.button-group {
	display: flex;
	justify-content: space-around;
}
</style>
