<template>
	<view>
		
			<uni-forms ref="dynamicForm" :rules="dynamicRules" :modelValue="dynamicFormData">
				<uni-section title="step.1 填写待摘要数据;摘要算法" type="line">
					<view class="example">
							<uni-forms-item label="摘要原文"  name="cert">
								<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.ori"/>
							</uni-forms-item>
							<uni-forms-item label="摘要算法"  name="algorithm">
								<uni-data-picker popup-title="请选择摘要算法" :localdata="algorithms"  v-model="dynamicFormData.algorithm" >			
								</uni-data-picker>
							</uni-forms-item>
							<view class="button-group">
								<button type="primary" size="max" @click="encByCert()">获取摘要</button>	
							</view>
							<br>
							<uni-forms-item label="摘要数据"  name="encData">
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
	
	// TODO 修复Android v3 加载过慢问题
	// #ifdef APP-PLUS
	var domModule = weex.requireModule('dom');
	domModule.addRule('fontFace', {
		'fontFamily': "uniicons",
		'src': "url('/static/uni.ttf')"
	});
	// #endif
	export default {
		data() {
			return {
				type: 'center',
				msgType: 'success',
				messageText: '',
				PKITool:uni.requireNativePlugin('lich-PKI-PKITool'),
				dt:uni.requireNativePlugin('lich-PKI-DigestTool'),
				dynamicRules: {
					
				},
				algorithms:[],
				dynamicFormData:{
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
				this.algorithms=[];
				console.log(ret.ProviderMode)
				let algorithm=ret.ProviderMode.MessageDigest
				let x;
				for (x in algorithm) this.algorithms.push({text:algorithm[x],value:algorithm[x]})
				this.dynamicFormData.algorithm=this.algorithms[0].value
				
			})
		},
		
		methods: {
			encByCert:function() {
				if(this.dynamicFormData.ori==''){
					this.messageToggle('error','摘要原文不能为空');
					return;
				}
				if(this.dynamicFormData.algorithm==''){
					this.messageToggle('error','摘要算法不能为空');
					return;
				}
				
				this.PKITool.getUTF8bytes(this.dynamicFormData.ori,
				(ret1)=>{
					console.log(ret1)
					if(ret1.code!=0){
						this.messageToggle('error',ret1.msg)
						return;
					}
					this.dt.digest(ret1.bytes,this.dynamicFormData.algorithm,
					(ret2)=>{
						console.log(ret2)
						if(ret2.code!=0){
							this.messageToggle('error',ret2.msg)
							return;
						}
						this.PKITool.encodeBase64String(ret2.digest,
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
