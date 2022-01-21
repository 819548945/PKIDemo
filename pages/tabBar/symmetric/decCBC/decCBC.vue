<template>
	<view>
		<uni-forms ref="dynamicForm" :rules="dynamicRules" :modelValue="dynamicFormData">
			<uni-section title="step.1 填写加密数据,密码,向量,加密算法" type="line">
				<view class="example">
					<uni-forms-item label="加密后数据"  name="encData">
						<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.encData"/>
					</uni-forms-item>	
					<uni-forms-item label="密码"  name="pwd">
						<uni-easyinput type="text" maxlength="-1" v-model="dynamicFormData.pwd"/>
					</uni-forms-item>
					<uni-forms-item label="向量"  name="iv">
						<uni-easyinput type="text" maxlength="-1" v-model="dynamicFormData.iv"/>
					</uni-forms-item>
					<uni-forms-item label="加密算法"  name="algorithm">
						<uni-data-picker popup-title="请选择加密算法" :localdata="algorithms"  v-model="dynamicFormData.algorithm" >			
						</uni-data-picker>
					</uni-forms-item>
						<view class="button-group">
							<button type="primary" size="max" @click="dec()">解密</button>	
						</view>
						<br>
						<uni-forms-item label="待加密数据"  name="ori">
							<uni-easyinput type="textarea" maxlength="-1" v-model="dynamicFormData.ori"/>
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
				st:uni.requireNativePlugin('lich-PKI-SymmetricTool'),
				PKITool:uni.requireNativePlugin('lich-PKI-PKITool'),
				algorithms:[],
				dynamicRules: {
					pwd: {
						rules: [{
							required: true,
							errorMessage: '密码不能为空'
						}]
					}
				},
				keyInfos:[],
				dynamicFormData:{
					ori:"",
					algorithm:"",
					iv:"11111111",
					pwd:"12345678",
					encData:""
				}
			}
		},
		onLoad(){
			this.PKITool.getProviderMode(
			(ret)=>{
				this.algorithms=[];
				console.log(ret.ProviderMode)
				let algorithm=ret.ProviderMode.Symmetric.Cipher
				let x;
				for (x in algorithm){
					if(x.indexOf("CBC")!=-1)
						this.algorithms.push({text:algorithm[x],value:algorithm[x]})
				}
				 
				this.dynamicFormData.algorithm=this.algorithms[0].value
				
			})
		},
		methods: {
			dec:function() {
				if(this.dynamicFormData.encData==''){
					this.messageToggle('error','加密数据不能为空');
					return;
				}
				if(this.dynamicFormData.pwd==''){
					this.messageToggle('error','密码不能为空');
					return;
				}
				if(this.dynamicFormData.iv==''){
					this.messageToggle('error','向量不能为空');
					return;
				}
				if(this.dynamicFormData.algorithm==''){
					this.messageToggle('error','加密算法不能为空');
					return;
				}
				this.PKITool.decodeBase64(this.dynamicFormData.encData,
				(ret1)=>{
					console.log(ret1)
					if(ret1.code!=0){
						this.messageToggle('error',ret1.msg)
						return;
					}
					this.PKITool.getUTF8bytes(this.dynamicFormData.pwd,
					(ret4)=>{
						console.log(ret4)
						if(ret4.code!=0){
							this.messageToggle('error',ret4.msg)
							return;
						}
						this.PKITool.getUTF8bytes(this.dynamicFormData.iv,
						(ret5)=>{
							console.log(ret5)
							if(ret5.code!=0){
								this.messageToggle('error',ret5.msg)
								return;
							}
							console.log("encData-"+ret1.bytes)
							console.log("pwd-"+ret4.bytes)
							console.log("iv-"+ret5.bytes)
							this.st.decrypt(ret1.bytes,ret4.bytes,ret5.bytes,this.dynamicFormData.algorithm,
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
									this.dynamicFormData.ori=ret3.string;
									this.messageToggle('success','解密成功');
								})
							})
						
						
						
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
