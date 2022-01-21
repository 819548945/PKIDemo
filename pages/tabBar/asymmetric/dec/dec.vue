<template>
	<view>
		<uni-forms ref="dynamicForm" :rules="dynamicRules" :modelValue="dynamicFormData">
			<uni-section title="step.1 填写秘钥号" type="line">
				<view class="example">
						<uni-forms-item label="秘钥号"  name="keyId">
							<uni-easyinput type="text" v-model="dynamicFormData.keyId"/>
						</uni-forms-item>

						<view class="button-group">
							<button type="primary" size="max" @click="readAlgorithm()">读取支持摘要算法</button>	
						</view>
						<br>
						<uni-forms-item label="摘要算法"  name="algorithm">
							<uni-data-picker popup-title="请选择摘要算法" :localdata="algorithms"  v-model="dynamicFormData.algorithm" >			
							</uni-data-picker>
						</uni-forms-item>
						
						
						
						
						<uni-forms-item label="加密数据"  name="signData">
							<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.encData"/>
						</uni-forms-item>
						<view class="button-group">
							<button type="primary" size="max" @click="dec()">解密</button>	
						</view>
						<br>
						
						<uni-popup ref="message" type="message">
							<uni-popup-message :type="msgType" :message="messageText" :duration="2000"></uni-popup-message>
						</uni-popup>
						<uni-forms-item label="解密数据"  name="ori">
							<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.ori"/>
						</uni-forms-item>
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
				kpt:uni.requireNativePlugin('lich-PKI-KeyPairTool'),
				dynamicRules: {
					keyId: {
						rules: [{
							required: true,
							errorMessage: '秘钥号不能为空'
						}]
					}
				},
				gmAlgorithms:[],
				rsaAlgorithms:[],
				algorithms:[],
				dynamicFormData:{
					keyId:'',
					ori:'',
					encData:'',
					algorithm:''
				}
			}
		},
		watch: {
		  'dynamicFormData.keyId': {
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
			dec:function() {
				if(this.dynamicFormData.keyId==''){
					this.messageToggle('error','秘钥号不能为空');
					return;
				}
				if(this.dynamicFormData.algorithm==''){
					this.messageToggle('error','摘要算法不能为空');
					return;
				}
				if(this.dynamicFormData.encData==''){
					this.messageToggle('error','加密数据不能为空');
					return;
				}
				this.PKITool.decodeBase64(this.dynamicFormData.encData,
				(ret1)=>{
					console.log(ret1)
					if(ret1.code!=0){
						this.messageToggle('error',ret1.msg)
						return;
					}
					this.asmt.decrypt(this.dynamicFormData.keyId,ret1.bytes,this.dynamicFormData.algorithm,
					(ret2)=>{
						console.log(ret2)
						if(ret2.code!=0){
							this.messageToggle('error',ret2.msg)
							return;
						}
						this.PKITool.getUTF8String(ret2.dec,
						(ret3)=>{
							console.log(ret3)
							if(ret3.code!=0){
								this.messageToggle('error',ret3.msg)
								return;
							}
							this.dynamicFormData.ori=ret3.string
							this.messageToggle('success',"解密成功")
						})
					})
				})
				
				
				
				
				
			},
			readAlgorithm:function() {
				if(this.dynamicFormData.keyId==''){
					this.messageToggle('error','公钥证书不能为空');
					return;
				}
				this.kpt.getKeyInfo(this.dynamicFormData.keyId,
				(ret1)=>{
					console.log(ret1)
					if(ret1.code!=0){
						this.messageToggle('error',ret1.msg)
						return;
					}
					this.algorithms=ret1.data.type=="EC"?this.gmAlgorithms:this.rsaAlgorithms	
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
