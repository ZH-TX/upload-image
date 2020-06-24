<template>
	<view>
		<view class="suggest">
			<view class="uni-textarea">
				<textarea class="textarea" maxlength="length" v-model="msg" 
				placeholder-style="color:#ccc" :placeholder="placeholder"/>
			</view>
			<view class="computeNum">
				{{computeLength}}
			</view> 
			<view style="padding: 30rpx;">
				<uImg  ref="upimg"
					:canUploadFile="true" 
					:limit="limitNum"
					:uploadFileUrl="uploadFileUrl" 
					:heaer="header" 
					:fileKeyName="name"
					:uImgList.sync="uImgList" 
					@uploadSuccess="uploadSuccess"/>
			</view>
		</view>
		
		<button @click="upFile">上传</button> 
		
	</view>
</template>

<script>
	import uImg from '@/components/zhtx-uploadImg/zhtx-uploadImg.vue';
	let toast= msg=>{
		uni.showToast({
			title: msg,
			icon:'none'
		});
	}
	
	export default {
		components:{uImg},
		
		data() {
			return {
				msg:'',
				length:140,
				placeholder:'欢迎反馈任何意见和问题,您的反馈也是我们产品的动力哦',
				limitNum:6,
				uploadFileUrl: 'https://a3.dns06.net.cn/app/index.php?i=2&c=entry&a=wxapp&do=Upload_qiniu_b&m=jzwx_a', //替换成你的后端接收文件地址
				name:'', //上传的名字
				header: {// 如果需要header，请上传
				},
				uImgList: ['/static/logo.png'],
				uImgList1:[]
			}
		},
		computed:{
			computeLength(){
				return this.length=140-this.msg.length
			}
		},
		methods: {
			uploadSuccess(result) {
				if(result.statusCode == 200) {
					//上传成功的回调处理
					toast('上传成功')
				}else{
					toast('上传失败')
				}
			},
			upFile(){
				this.$refs.upimg.upload()
			}
		},
	
	}
</script>

<style>
	.suggest{
		background: #fff;
		border-top: 1px solid #E7E7E7;
		margin-bottom: 10px;
	}
	.textarea{
		padding: 20px;
		/* font-size: 12px; */
	}

	
	.content {
		/* width: 100%; */
		padding: 10px 20px;
		background: #fff;
		border-bottom: 1px solid #EBEBEB;
	}
	
	.content-con {
		width: 100%;
		background: #fff;
		display: flex;
		align-items: center;
		align-content: center;
		justify-content: space-between;
	 
	}
	
	.con-lf {
		display: flex;
		align-items: center;
	}
	
	.con-lf text {
		font-size: 12px
	}
	
	.con-rg {
		color: #ccc;
		font-size: 12px;
	}
	
	.faceback {
		width: 20px;
		height: 20px;
		padding-right: 20px;
	}
	
	.tip-info{
		padding: 20px;
		padding-top: 10px;
		font-size: 10px;
		color: #ccc;
	}
	
	.computeNum{
		color: #ccc;
		float: right;
		padding-right:10px;
		font-size: 14px;
	}
	.computeNum :after{
		overflow: hidden;
	}
</style>
