Android IBeacon 开发指南
====
ShituBlueprint SDK 版本号 1.0

# 1获取SDK

您可发送邮件至 
<a href="mailto:sdk@ubirouting.com">sdk@ubirouting.com</a>  获取最新的Android SDK。详见概述。

---
#2简介
ShituBlueprint 定位技术使用 IBeacon 定位，可自动判断初始楼层 <a href="http://ubirouting.com/technology.php">详情请见...</a>

ShituBlueprint 定位云服务会使用您在 
<a href="http://ubirouting.com/#banner/1">识途 Creator</a> 
中的数据以及定位信息，向您的应用提供定位服务。
在使用SDK中服务前需要您获取 ShituKey，如果Key错误您将无法正常使用服务。

您使用的数据会直接使用Key 与您的账户关联，当您在Creator或开发者平台上操作此Key拥有账户的数据时，会直接影响SDK中所对应的数据，所以请您妥善保管您的账户密码以及您申请的Key。

###2.1 SDK包中内容简介 
SDK中包含

-  1个 .jar文件 UbiBlueprintLib.jar
-  1个 .so文件  libblueprint.so

###2.2 类简介
- ShituBlueprintManager 定位管理类 配置定位信息，以及回调。
- ShituPlaceDataManager 数据管理类 获取Creator数据所用。
- Place 建筑对象。
- Floor 层对象
。
#3 开发
下面我们开始配置Android工程，以支持识途定位。

###3.1 添加Jar包以及.so文件
- 获取识途Blueprint 定位SDK（包括.jar及.so文件）；
- 在Android对应工程中，新建libs文件夹；
- 将.jar拷贝至libs下，将.so文件拷贝至libs/armeabi下；
- 在Android工程中，将.jar文件添加至BuildPath中。

###3.2 AndroidMenifest.xml
在AndroidMenifest.xml的标签中添加如下代码

	<meta-data
    	android:name="ShituKey"
    	android:value="你的Key" 
	/>	

因为识途定位SDK使用网络定位，所以请记得为您的应用添加网络权限。

	<uses-permission android:name="android.permission.INTERNET"/>

当然您还需要加入蓝牙对应使用权限

	<!-- 允许程序连接配对过的蓝牙设备 -->
    <uses-permission android:name="android.permission.BLUETOOTH" />
    <!-- 允许程序进行发现和配对新的蓝牙设备 -->
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />

我们的数据获取加入了缓存机制减少使用的流量,所以您还需要加入SD卡的读取权限。

	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

以及获取WIFI状态的权限

	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>

    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
        
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />

###3.3 获取建筑数据

在使用定位服务前您需要获取您在 
<a href="http://ubirouting.com/#banner/1">识途 Creator</a> 
中采集上传的数据 您可以使用：

	//传入context 上下文对象	
	ShituPlaceDataManager ubiData=new ShituPlaceDataManager(this);
    ubiData.getData(new onReturnDataListener() {
    	@Override
    	public void onDataReturn(List<Place> list) {
    		// TODO Auto-generated method stub
    	}
    });

来获取您的数据 以及接下来要使用的 Place 对象（placeId）

如果您已知建筑id 且不需要使用其他建筑信息则此步可以省略。

###3.4 定位

如果您只需要使用蓝牙扫描不需要定位的话只需要实例化context 即可。
	
	//传入context 上下文对象
	ShituBlueprintManager blueprintManager=new ShituBlueprintManager(this);

如果您需要使用定位则需要使用此构造实例：

	//分别传入 context上下文对象,PlaceId Creator中建筑Id,isParseBean 生成完整对象，位置返回回调
	blueprintManager=new ShituBlueprintManager(this,10,false,new OnPositionSuccess() {
		@Override
		public void onPosition(Position pos) {
			// TODO Auto-generated method stub
		}
	});

其中，第二个参数为建筑id，也可以直接替换 id 传入place 对象。

	blueprintManager=new ShituBlueprintManager(this,place,false,new OnPositionSuccess() {...}

回调中的对象为您定位出的位置对象，其中位置信息 x，y 与您在Creator 或云平台中上传的图片坐标对应。

###3.5 返回结果

识途 Blueprint 定位SDK（Android）通过回调，将结果回调至主线程。

回调onGetPosition(Position position)中获取的Position对象，包含了定位的准确信息，其中包括坐标、楼层以及系统建议的精确度的半径参数（像素单位）。

其中，坐标为像素坐标，即相对于室内地图左上角（0，0）的坐标，该坐标系x轴向右增长，y轴向下增长。此坐标与使用 <a href="http://ubirouting.com/#banner/1">识途 Creator</a>  采集过程中的平面图坐标系一致。

###3.6 开始定位以及停止定位

#####3.6.1 使用
开始定位与停止定位
	
	//开始定位 返回boolean 是否开启成功
	startBlueprintLocation();

	//停止定位 返回boolean 是否关闭成功
	stopBlueprintLocation();

当您在不进行定位时，您可以单独获取Beacon数据，但这也需要您申请key 才可以使用

	//开启扫描
	startBlueprint();

	//获取数据
	getBlueprintData();

	//停止扫描
	stopBlueprint();

#####3.6.2 Key的验证
在您每一次实例化定位类，获取建筑信息，以及每一次提交定位时均会提交至少1次的Key验证请求。

在您获取建筑信息时，当返回正确数据时即代表验证成功。

建议您在使用定位之前至少执行一次上述操作，避免在网络情况不好的情况下因，Key验证延迟造成无结果返回。

当某一个验证失败时，不会影响后续验证。

###3.7 注意事项

如果您不实例定位对象时，您将无法使用定位功能。

	new ShituBlueprintManager(this);

同理如果您在使用定位功能时，不传入回调或者回调为null 时同样无法使用定位功能。

	new ShituBlueprintManager(this,10,false,null);

当您的Key 错误或者正在验证Key 时，您所有的扫描结果会是null
，而定位结果会是 x=0，y=0，area=-99

当您的Key 泄露或更换开发程序的设备时，请您重新设置您的Key，每一个Key都与一台计算机的SHA1码相关联，在不使用签名打包的情况下，不同电脑编译打包的程序无法使用同一个key，签名打包为android源生打包技术此处不做介绍。


###3.8 示例

	private void initShituBlueprintLocation(){
    	dataM=new ShituPlaceDataManager(this);
        dataM.getData(new onReturnDataListener() {
			
			@Override
			public void onDataReturn(List<Place> arg0) {
				for (Place place : arg0) {
					//可以获取到您之前添加的信息
					Log.d("UbiBeaLibTest", "id:"+place.getPlaceId()+",name:"+place.getPlaceName());
				}
				//直接使用对象
				Place place=arg0.get(0);
				
				//或手动指定id
				int id =arg0.get(0).getPlaceId();
				place.setPlaceId(id);
				
			  //locM=new ShituBlueprintManager(MainActivity.this, id, false,new OnPositionSuccess() {
				locM=new ShituBlueprintManager(MainActivity.this, place, false,new OnPositionSuccess() {
					@Override
					public void onPosition(Position arg0) {
						Log.d("UbiBeaLibTest", arg0.toString());
					}
				});
				
				//启动定位
				locM.startBlueprintLocation();
			}
		});
    }
    
	private void stopShituBlueprintLocation() {
		if (locM != null) {
			// 停止定位
			locM.stopBlueprintLocation();
		}
	}





