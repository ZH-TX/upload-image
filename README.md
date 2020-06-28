# upload image for uniApp(简便版)



>**推荐测试运行环境小程序开发者工具,避免出现CORS/CORB**  
>**请多多支持\(^o^)/~,插件功能中有注释**  




---------------------




### 现阶段支持功能如下-> 我们只需要调用后端提供过来接口即可

- [√]图片上传前预览
- [√]图片上传后自动删除(可根据需求修改)
- [√]图片预览删除
- [√]图片添加
- [√]图片上传设置(limit)
- [x]图片上传进度监控
- [√]支持手动上传(一键全部上传) 12/22


---------------------

### 使用说明
拷贝该组件到`components`目录下之后  
在 `script` 中引用组件

``` javascript 
import uImg from '@/components/zhtx-uploadImg/zhtx-uploadImg.vue';
export default {
    components: {uImg}
}
```

在 template 中使用组件：  

``` javascript
<uImg  ref="upimg"
	:canUploadFile="true" 
	:limit="limitNum"
	:uploadFileUrl="uploadFileUrl" 
	:heaer="header" 
	:fileKeyName="name"
	:uImgList.sync="uImgList" 
	@uploadSuccess="uploadSuccess"/>

```


### 组件参数说明

属性名    | 类型             | 默认值   | 说明    |
--------- |--------         |--------- | --------|
uImgList  | `Array<Object>` | []       | 初始化的图片数据，可用于单向数据初始化，需要双向绑定可直接用 :uImgList.sync="uImgList" |
uploadFileUrl | `String`   | null      | 上传文件的服务器url |
header    | `Object`        | null      | 上传图片到服务器时，HTTP 请求 Header |
limit     | `Number`        | 9         | 限制可上传的图片数量,图片最大上传数量 |
fileKeyName| `String`       | 'file'    | 用于在服务端通过自定义key值获取该文件数据|




### 自定义事件说明
事件名称  | 说明| 返回参数 |
--------- | --------|--------- |
upload | 图片上传函数，选择多个后会立即上传至后端服务器| 无 |

(主要看了这么多上传插件,几乎没有手动上传功能,都是点击一张就直接上传)
<br>
(所以自己动手实现了一个保留最基础图片批量上传功能)


### demo示例

``` javascript
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
					@uploadSuccess="uploadSuccess"></uImg>
			</view>
		</view>
		
		<button @click="upFile">上传</button> 
	</view>
</template>

<script>
	import uImg from '@/components/img.vue';
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
				uploadFileUrl: '', //替换成你的后端接收文件地址
				name:'', //上传的名字
				header: {// 如果需要header，请上传
				},
				uImgList: ['/static/logo.png']
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
			//上传方法的调用
			upFile(){
				this.$refs.upimg.upload()
			}
		},
	
	}
</script>

<style>
	
</style>


```

### 通用配置(version)

|版本号	|更新时间	|关键字			|默认值					|功能作用														
|--		|--			|--				|--						|--																					
|v0.1	|2019/12/22 |无			    |无				        |实现简单的图片上传预览	
|v0.2	|2020/6/25  |无			    |无				        |添加图片数量,以及优化上传逻辑
|v0.3	|2020/6/28  |无			    |无				        |优化逻辑,以及兼容性



-------------
#### tips:
1. 6/28, 不得不说uni-app,总有一些小bug, 需要自己注意下, 今天升级了版本
运行在app端出现了问题, 过程不说了, 最后发现style 中了scoped后, 样式全部失效??
																																	|
																													|
### 欢迎大家下载测试,以及评论留言, 有遇到什么bug, 我会尽快处理

+ 如有什么需求需要添加,也可以多加评论,表达


-----------------------------------




### License
#### MIT  
Copyright (c) 2019-present, 2305945912@qq.com

##### *其它待开发...*
