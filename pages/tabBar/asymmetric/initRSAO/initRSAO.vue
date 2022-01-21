<template>
	<uni-forms ref="dynamicForm" :rules="dynamicRules" :modelValue="dynamicFormData">
	<uni-section title="step.1 生成秘钥对" type="line">
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
	<uni-section title="step.3 导入公钥证书 (CA返回)" type="line">
		<view class="example">
				<uni-forms-item required label="公钥证书"  name="cert">
					<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.cert"/>
				</uni-forms-item>
				<view class="button-group">
					<button type="primary" size="mini" @click="generateX509Cert()" >3生成自签名证书</button>
					<button type="primary" size="mini" @click="importX509Certificate()" >4导入公钥证书</button>
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
					cert:''
				}
			}
		},
		methods: {
			generateKeyPair:function (){
				this.kpt.generateRSAKeyPair(2048,(ret)=>{
					console.log(ret)
					if(ret.code!=0){
						this.dialogToggle('error',ret.msg)
						return;
					}
					this.dynamicFormData.keyId=ret.keyId;
					this.dynamicFormData.p10=ret.N;
					this.dialogToggle('success','秘钥对生成成功')
				})
			},
			generateP10:function(){
				if(this.dynamicFormData.keyId==''){
					this.dialogToggle('error','秘钥号不能为空');
					return;
				}
				this.kpt.generateP10(this.dynamicFormData.keyId,"C=CN,CN=RSATEST","SHA1WithRSA",
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
					this.dynamicFormData.p10=ret.N;
					this.dialogToggle('success','公钥获取成功')
				})		
			},
			generateX509Cert:function(){
				if(this.dynamicFormData.keyId==''){
					this.dialogToggle('error','秘钥号不能为空');
					return;
				}
				let publicKeyInfo={
					serialNumber:"ff1e1112",
					notBefore:parseInt(new Date().getTime()/1000),
					notAfter:parseInt(new Date().getTime()/1000+60*60*24*365),
					subject:"C=CN,CN=RSATEST",
					signatureAlgorithm:"SHA1WithRSA"
				}
				console.log(publicKeyInfo)
				this.pukt.generateX509Certificate(this.dynamicFormData.keyId,1,publicKeyInfo,(ret)=>{
					console.log(ret)
					if(ret.code!=0){
						this.dialogToggle('error',ret.msg)
						return;
					}
					this.dynamicFormData.cert=ret.cert
					this.dialogToggle('success','公钥证书生成成功');
				})
				
			},
			importX509Certificate:function(){
				if(this.dynamicFormData.keyId==''){
					this.dialogToggle('error','秘钥号不能为空');
					return;
				}
				if(this.dynamicFormData.cert==''){
					this.dialogToggle('error','公钥证书不能为空');
					return;
				}
				this.kpt.importX509Certificate(this.dynamicFormData.keyId,this.dynamicFormData.cert,0,
				(ret)=>{
					console.log(ret)
					if(ret.code!=0){
						this.dialogToggle('error',ret.msg)
						return;
					}
					this.dialogToggle('success','导入公钥证书成功')
				}),
				this.dialogToggle('success','0')
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
