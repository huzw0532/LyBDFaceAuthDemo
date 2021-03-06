# LyBDFaceAuth

uniapp 安卓端百度人脸识别、活体检测、人脸采集 demo。

#### <u>1、前言</u>

最近在使用 uniapp 开发项目，有刷脸实名认证的需求，最终使用百度人脸识别实现了需求。自己做了个 APP 原生插件，给大家介绍下用法。想快速集成的看第二点，如不需要可直接跳过看第三点百度官方资料准备。

#### <u>2、增值服务（*快速集成*）</u>

你可能从来没接触过原生开发，不知道签名文件怎么生成，不知道MD5怎么获取，不知道百度授权文件怎么设置（百度授权只有2个名额，错了就没有机会补救了）。即使你通过摸索研究出来，也需要两三天（不一定成功），你这两天的工资就是你的机会成本，现在你不需要付出这么多钱，只需119元一条龙服务（市场上仅仅卖插件没有增值服务都要200元），<font color=red>保证你在线打包基座成功，可获取人脸图片</font>。首付50元启动，不成功全额退款。你只需提供百度账号密码（成功后自行修改密码），或者远程控制（工作日19:30到21:00，周末全天）。前提是百度已开通离线采集（下面3.1有说明），因为这需要私人信息，并且需要审核通过，存在不可控因素。公众号「longyoung」后台回复「百度人脸接入」，获取此增值服务。

#### <u>3、百度官方资料准备</u>

此处需要准备 2 样东西，一是 licenseID，二是 License 授权文件（文件名 idl-license.face-android）。注意使用自己的证书（就是签名文件.jks），申请百度授权文件的时候需要填写 md5，就填写这个证书的 md5。如果已经准备好百度资料，可直接拉到下面看第四点接入步骤。百度详细步骤请查看[百度官方文档](https://ai.baidu.com/docs#/FaceSDK-Collect-WithLiveness-Android/top)，页面拉到2.2准备工作有详细申请说明。`注意事项请看下面的特别说明`。

![uni1.png](https://i.loli.net/2019/12/03/RDsyHQmzTXnACra.png)

##### 特别说明：

3.1 需要申请开通离线采集，点击 3.3 的图片编号 ②，一般` 3 个工作日审核通过，请提前准备`。

3.2 申请通过后，需要新建授权，授权标识只需填入你的标识，如我填入的是 longyoung，百度会自动加后缀，生成licenseID：longyoung-face-android。`记住这个，下面示例代码的第一个参数就是传这个字符串`。

![uni2.png](https://i.loli.net/2019/12/03/PuSQ3fZy9pY2dRq.png)

3.3 下图的编号 ③ 就是 licenseID，点击编号 ④ 下载授权文件，放到 nativeplugins/longyoung-BDFaceAuth/android/assets/idl-license.face-android。没路径的请自行创建，nativeplugins 文件夹在项目根目录下，下面接入步骤 4.2 有提到这点。


![uni3.png](https://i.loli.net/2019/12/03/zukV31FGAD5j2Cr.png)

#### <u>4、接入步骤</u>

4.1 在项目根目录创建文件夹 nativeplugins，下载插件 longyoung-BDFaceAuth，放进这个目录。

4.2 将百度授权文件 License 放到 nativeplugins/longyoung-BDFaceAuth/android/assets/idl-license.face-android。没路径的请自行创建，nativeplugins 文件夹在项目根目录下。

![uni4.png](https://i.loli.net/2019/12/03/G6B8YRjV3ZJa2fi.png)

4.3 manifest.json 文件，选中「App原生插件配置」，选中本地插件，选择使用插件 longyoung-BDFaceAuth。


![uni5.png](https://i.loli.net/2019/12/03/vFPHhJSwOuXIQf7.png)

4.4 调用插件，需要传入 licenseID（必传，百度上的License ID），动作控制参数 actionAry（不传只采集脸，没有动作），动作是否随机参数 isLivenessRandom，是否有声音参数 isSound，代码如下：

```
<script>
	// #ifdef APP-PLUS
	const lyBDFaceAuth = uni.requireNativePlugin('longyoung-BDFaceAuth'); //android
	const lyBDFaceAuthIOS = uni.requireNativePlugin('longyoung-BDFaceAuth-iOS'); //ios
	// #endif
	export default {
		data() {
			return {
				title: ''
			}
		},
		methods: {
			onScanFace() {
				console.error("tagg.onScanFace");
				
				if (uni.getSystemInfoSync().platform == "android") {//安卓
					lyBDFaceAuth.scanFace({
						licenseID:"longyoung-face-android",//必须与百度授权资料一致
						actionAry:["Eye", "Mouth", "HeadLeft", "HeadRight", "HeadLeftOrRight", "HeadUp", "HeadDown"],//不传无动作
						isLivenessRandom:0,//不传默认有序，0有序，1随机
						isSound:0,//不传默认有声音，0无声，1有声，iOS无效
						txtColor:this.txtColor,//文字颜色
						bgColor:this.bgColor,//背景颜色
						roundColor:this.roundColor//圆的颜色
					}, result => {
						console.log('file://' + result.imgPath);
						
						//图片上传服务器
						uni.uploadFile({
							url:'http://api.longyoung.com/api/open/common/uploadImgTemp',//图片上传地址
							filePath: 'file://' + result.imgPath,//图片本地路径，上传服务器需要加这个头'file://'
							method: 'post',
							name: 'imgFile',//上传图片参数名
							success: (res) => {
								var data = res.data;
							}
						});
						
						//***有些同学反馈，后台强烈要求传base64，下面是图片转base64的方法，没此需求的可以无视。
						var bitmapT = new plus.nativeObj.Bitmap("test"); //test标识谁便取
						// 从本地加载Bitmap图片
						bitmapT.load(result.imgPath, function() {
							console.log('加载图片成功');
							var base4 = bitmapT.toBase64Data();
							console.log('lygg.base64=' + base4);
						}, function(e) {
							console.log('加载图片失败：' + JSON.stringify(e));
						});
						//***有些同学反馈，后台强烈要求传base64，下面是图片转base64的方法，没此需求的可以无视。
						
					});
				}
				else if (uni.getSystemInfoSync().platform == "ios") {//苹果
					lyBDFaceAuthIOS.scanFace({
						licenseID: "longyoung-face-ios",//必须与百度授权资料一致
						actionAry:["Eye", "Mouth", "HeadLeft", "HeadRight", "HeadLeftOrRight", "HeadUp", "HeadDown"],//不传无动作
						isLivenessRandom:0,//不传默认有序，0有序，1随机
						isSound:0,//不传默认有声音，0无声，1有声，iOS无效
					}, result => {
						console.log('result=' + result);//图片存在 result.bestImgBase64，显示图片需要加头"data:image/png;base64," + result.bestImgBase64.replace(/[\r\n]/g, "")
						
						//***不传base64的，看这里，使用 uni.uploadFile()上传服务器，没此需求的可以无视。
						var bitmapT = new plus.nativeObj.Bitmap('test');
						//加载base64图片
						bitmapT.loadBase64Data(result.bestImgBase64, function(res){
							//保存base64图片
							bitmapT.save("_faceImg/face.png", {}, function(res){
								bitmapT.clear();//销毁bitmap对象
								
								//上传图片
								//TODO 使用 uni.uploadFile()上传服务器，filePath=res.target
								
							}, function(res){
								console.log("longyoung.save.fail=", res);
							});
							
						}, function(res){
							console.log("longyoung.fail=", res);
						});
						//***不传base64的，看这里，使用 uni.uploadFile()上传服务器，没此需求的可以无视。

					});
				}
				
				
			},
		}
	}
</script>
```

4.5打自定义基座

需要打自定义基座才可以进行测试，运行->运行到手机或模拟器->制作自定义基座。注意事项看图片，请使用自己的证书（就是签名文件.jks），申请百度授权文件的时候填写这个证书的 md5。`每次打自定义基座的时候，最好先把旧的基座删除（如果存在），路径在根目录下的unpackage文件夹里面(LyBDFaceAuthDemo\unpackage\debug\android_debug.apk)。`

![uni6.png](https://i.loli.net/2019/12/10/3r25AHUlXpx4tYE.png)

![uni7.png](https://i.loli.net/2019/12/10/VqbN6J82WpeEFPY.png)

4.6勾选自定义基座后运行

运行->运行到手机或模拟器->运行基座选择，勾选自定义调试基座，之后运行到设备，如下图。

![uni8.png](https://i.loli.net/2019/12/10/SGBO2TIFHl3NQ1j.png)

#### <u>5、注意事项</u>

5.1 图片存到本地缓存目录，/storage/emulated/0/Android/data/com.longyoung.facedemo/cache/faceImg/face1573781316334.png，上传服务器需要加这个头'file://'。如需要传base64格式，请看上面示例代码。

5.2 最新版首更在[github](https://github.com/longyoung/LyBDFaceAuthDemo.git)。公众号「longyoung」后台回复「百度人脸识别」免费获取，限时免费。

#### <u>6、版权声明</u>
版权归开发者所有，未经授权同意，不得分享源码。