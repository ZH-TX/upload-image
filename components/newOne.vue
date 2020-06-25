<template>
	<view>
		<view class="imgs">
			<!-- images -->
			<view class="single" v-for="(item, index) in list" :key="index" >
				<image :src="item" :data-src="item" mode="aspectFit" @tap="previewImg" />
				
				<progress :percent="item.process" activeColor="#67C23A" 
					:backgroundColor="item.process == 100 || item.process == undefined ? '#67C23A' : '#F56C6C'"
					 stroke-width="3" 
					 v-if="mode == 'create' && showProcess" />
				 <!-- 删除按钮 -->
				<view class="del" @tap="deleteItem(index)">×</view>
			</view>


			<!-- add button -->
			<view v-if="limit>list.length" class="single addNew" @tap="chooseFile">
				<text class="cuIcon-add">+</text>
			</view>
		</view>
	</view>
</template>

<script>
	/* 
	 1.实现基础的上传预览
	 2. 实现精度条的显示
	 3. 实现可以移动的图片
	 4. 掌握基本的封装
	 */
	import {toast}from '@/utils/common.js'
	export default {
		props: {
			attachmentList: {
				type: Array, //附件列表
				default () {
					return {}
				}
			},
			mode: {
				type: String //模式： create => 可新增或编辑附件 不填或其他 => 只能查看附件
			},
			uploadFileUrl: {
				type: String,
				dafault: '#' // 上传文件的服务器url
			},
			showProcess: {
				type: Boolean,
				default: false //是否显示进度，默认不显示
			},
			header: {
				type: Object, //上传图片到服务器时，HTTP 请求 Header
				default () {
					return {}
				}
			},
			limit: {
				type: Number, //限制可上传的图片数量
				default: 9 //这里有问题???
			},
			fileKeyName: {
				type: String,
				default: 'file' //用于在服务端通过自定义key值获取该文件数据
			},
			canUploadFile: { //是否更新
				type: Boolean,
				default: true
			}
		},
		computed: {
			list: {
				get(){return this.attachmentList},
			}
		},
		
		data() {
			return {
				imageList: [],
				
				urls:[]
			};
		},
		methods: {
		
			//预览
			previewImg(e) {
				console.log(...this.list);
				uni.previewImage({
					current: e.target.dataset.src,
					loop: true,
					longPressActions: {
						itemList: ['发送给朋友', '保存图片', '收藏'],
						success: function(data) {
							console.log('选中了第' + (data.tapIndex + 1) + '个按钮,第' + (data.index + 1) + '张图片');
						},
						fail: function(err) {
							console.log(err.errMsg);
						}
					},
					urls: this.list   //this.imageList,保持删除了的不在
				});
			},
			//下载
			downLoad(url) {
				uni.showModal({
					title: '确定要下载此附件吗',
					content: ' ',
					success: res => {
						if (res.confirm) {
							uni.showLoading({
								title: '下载中,请稍后'
							});
							console.log(url);
							uni.downloadFile({
								url: url,
								success: res => {
									var tempFile = res.tempFilePath;
									uni.saveFile({
										tempFilePath: res.tempFilePath,
										success: res => {
											var savedFilePath = res.savedFilePath;
											uni.hideLoading();
											uni.showToast({
												title: '保存成功，路径为' + savedFilePath
											});
											uni.openDocument({
												filePath: savedFilePath,
												success: function(res) {
													console.log(res);
												}
											});
										}
									});
								},
								fail: res => {
									console.log(res);
									uni.hideLoading();
									uni.showToast({
										title: '下载失败',
										icon: 'none'
									});
								}
							});

							setTimeout(function() {
								uni.hideLoading();
							}, 2000);

							// downloadTask.onProgressUpdate((res) => {
							//     console.log('下载中,进度' +  res.progress)
							// });
						}
					}
				});
			},
			//删除
			deleteItem(index) {
				uni.showModal({
					title: '提示',
					content: '确定要删除此项吗？',
					success: res => {
						if (res.confirm) {
							// if (this.list[index].process != 100) {
							// 	typeof this.list[index].uploadTask != 'undefined' && this.list[index].uploadTask.abort();
							// }
							this.imageList.splice(index,1)
							this.list.splice(index, 1);
							this.$forceUpdate(); //强制更新
							this.$emit('update:attachmentList', this.list); //类似双向数据绑定
						}
					}
				});
			},

			chooseFile() {
				//双重保证
				// console.log(this.list);
				if (this.list.length >= this.limit) {
					toast('已达到最大上传数量')
					return; 
				}
				
				let canUploadFile = this.canUploadFile;
				let tempFiles;
				if (canUploadFile) {
					uni.chooseImage({
						count: this.limit - this.list.length,
						sizeType: ['original', 'compressed'], 
						sourceType: ['album', 'camera'],
						success: (res) => {
							// console.log(res.tempFilePaths);
							tempFiles = res.tempFilePaths;
							
							this.imageList = this.imageList.concat(tempFiles)
							
							console.log(this.imageList);
							this.list.push(...tempFiles)//如果图片一次性就超过这个值怎么使他赋的值回退
							// #ifdef H5
							if (this.list.length >= this.limit) {
								this.list.splice(this.limit)
								toast('已达到最大上传数量')
								return; 
							}
							// #endif
							
							this.$emit('update:attachmentList', this.list); //类似双向数据绑定,更新数据, 使用.sync修饰
							this.$forceUpdate();
							console.log(this.list);
						},
						fail:err=>{
							console.log(err);
						}
					});
						
				} 
				//设置文件名字
				/* for (let i in this.imageList) {
					let path = this.imageList[i];
					
					let index = this.list.length;
					console.log(this.list);
					
					//需要设置成多个上传,而非单个上传
					this.list.push({
						url: path,
						// fileName: fileName,
						type:  'image' ,
						index: index,
						process: 0,
						// uploadTask: uploadTask	
					});
	
				
				/* 	//上传, 可能这里还需要调整一下,这里点击之后就直接上传了,让用户去操作
					var uploadTask = await uni.uploadFile({
						url: this.uploadFileUrl,
						filePath: path,// 使用files上传数组列表
						name: this.fileKeyName,
						headers: this.header,
						success: res => {
							// 上传完成后处理
							this.$emit('uploadSuccess', res);
							if (res.statusCode == 200) {
								this.$emit('update:attachmentList', this.list); //类似双向数据绑定,更新数据, 使用.sync修饰
								this.$forceUpdate();
							} else {
					
							}
						}
					});
					
					uploadTask.onProgressUpdate(res => {
						//此接口不显示真实进度， 所以需要特殊处理
						if (res.progress < 90) {
							this.$set(this.list[index], 'process', res.progress);
							this.$forceUpdate();
						}
					});
				 */
				// }*/
				
				
			},
			upload(){
				uni.showLoading({
					title: '上传中...',
					mask: false
				});
				for(let i=0; i<this.list.length;i++){
					let path=this.list[i]
					let index=i.toString()
					console.log(path);
					uni.uploadFile({
						url: this.uploadFileUrl,
						name: this.fileKeyName,
						filePath: path, // 使用files上传数组列表,上面两者都会失效
						file:[
							{name:index,url:path}
						],
						success:res=>{
							uni.hideLoading()
							console.log(res);
							this.$emit('uploadSuccess', res);
							if (res.statusCode == 200) {
								//上传成功将原信息,直接删除,
								this.list.splice(i,1)
								console.log(this.list);
								
								this.$forceUpdate();
							} else {
											
							}
						},
						fail:err=>{
							uni.hideLoading()
							toast(err.errMsg)
							console.log(err);
						}
					})
								
				}
				
			},
			
		/* 	async upload(){
				// 开始上传，先暂存文件
				this.list.push({
					fileName: fileName,
					url: path,
					type: this.isImg(path) ? 'image' : 'file',
					index: index,
					uploadTask: uploadTask,
					process: 0
				});
				this.$forceUpdate();
				//上传, 可能这里还需要调整一下,这里点击之后就直接上传了,让用户去操作
				var uploadTask = await uni.uploadFile({
					url: this.uploadFileUrl,
					filePath: path,// 使用files上传数组列表
					name: this.fileKeyName,
					headers: this.header,
					success: res => {
						// 上传完成后处理
						this.$emit('uploadSuccess', res);
						if (res.statusCode == 200) {
							this.$emit('update:attachmentList', this.list); //类似双向数据绑定,更新数据, 使用.sync修饰
							this.$forceUpdate();
						} else {
				
						}
					}
				});
				
				uploadTask.onProgressUpdate(res => {
					//此接口不显示真实进度， 所以需要特殊处理
					if (res.progress < 90) {
						this.$set(this.list[index], 'process', res.progress);
						this.$forceUpdate();
					}
				});
			},
			 */
			
			
			//根据文件名，返回时是否是图片类型
			/* isImg(filePath) {
				let index = filePath.lastIndexOf('.');
				let ext = filePath.substr(index + 1);
				var temp = ['png', 'jpg', 'jpeg', 'bmp', 'gif', 'webp', 'svg', 'tiff'].indexOf(ext.toLowerCase()) !== -1;
				return temp;
			} */
		}
	};
</script>

<style lang="less" scoped>
	.imgs {
		display: flex;
		flex-wrap: wrap;
		justify-content: flex-start;
		align-items: center;
		// width: calc(100% + 15rpx);

		& .file {
			min-width: calc(100% - 15rpx);
			border: 1px solid #ccc;
			// border-radius: 10upx;
			box-sizing: border-box;
			margin-top: 20upx;
			position: relative;

			& .noImg {
				padding: 20rpx;
				display: flex;
				justify-content: center;
				text-align: left;
				width: 100%;
				font-size: 26upx;
				// flex-wrap: wrap;
				color: #999;
				word-break: break-all;
				box-sizing: border-box;
			}
		}

		progress {
			margin-top: -6rpx;
			border-radius: 20rpx;
		}

		.del {
			position: absolute;
			width: 35rpx;
			height: 35rpx;
			background: #f56c6c;
			color: #fff;
			top: 0;
			text-align: center;
			right: 0;
			line-height: 35rpx;
			font-size: 30rpx;
			z-index: 100;
		}

		& .single {
			width: 180rpx;
			height: 180rpx;
			border: 1px solid #ccc;
			// border-radius: 10upx;
			margin: 10rpx;

			position: relative;

			
			&.addNew {
				display: flex;
				justify-content: center;
				align-items: center;

				text {
					font-size: 50rpx;
					color: #999;
				}
			}

			& image {
				width: 100%;
				height: 100%;
				display: block;
			}
		}
	}
</style>

