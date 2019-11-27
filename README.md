# LyBDFaceAuth

uniapp 安卓端百度人脸识别、活体检测、人脸采集 demo。

#### <u>百度官方资料准备</u>

1.详细步骤请查看[官方文档](https://ai.baidu.com/docs#/FaceSDK-Collect-WithLiveness-Android/top)。

2.需要准备 License 授权文件，文件名 idl-license.face-android。

#### <u>接入步骤</u>

1.在项目根目录创建文件夹 nativeplugins，下载插件 longyoung-BDFaceAuth，放进这个目录。

2.将百度授权文件 License 放到 nativeplugins/longyoung-BDFaceAuth/android/assets/idl-license.face-android。

3.manifest.json 文件，选中「App原生插件配置」，选中本地插件，选择使用插件 longyoung-BDFaceAuth。

4.调用插件，需要传入 licenseID（必传，百度上的License ID），动作控制参数 actionAry（不传只采集脸，没有动作），动作是否随机参数 isLivenessRandom，是否有声音参数 isSound，代码如下：

```
<script>
	const lyBDFaceAuth = uni.requireNativePlugin('longyoung-BDFaceAuth');
	export default {
		data() {
			return {
				title: ''
			}
		},
		methods: {
			onScanFace() {
				console.error("tagg.onScanFace");
				lyBDFaceAuth.scanFace({
					licenseID:"longyoung-face-android",
					actionAry:["Eye", "Mouth", "HeadLeft", "HeadRight", "HeadLeftOrRight", "HeadUp", "HeadDown"],//不传无动作
					isLivenessRandom:0,//不传默认有序，0有序，1随机
					isSound:0,//不传默认有声音，0无声，1有声
				}, result => {
					console.log('file://' + result.imgPath);
				});
			},
		}
	}
</script>
```

#### <u>注意事项</u>

1.图片存到本地缓存目录，/storage/emulated/0/Android/data/com.longyoung.facedemo/cache/faceImg/face1573781316334.png，上传服务器需要加这个头'file://'。

2.最新版首更在[github](https://github.com/longyoung/LyBDFaceAuthDemo.git)。

#### <u>版权声明</u>
版权归开发者所有，未经授权同意，不得分享源码。