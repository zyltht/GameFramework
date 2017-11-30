#iOS集成文档
-------------

* 谁来阅读此文档

	   接入厂商的产品，技术人员，平台技术人员。

* 注意！！！！

      如果是上线到appstore，SDK有使用到IDFA，提交审核的时候请注意选上。
勾选详情参照[http://www.jianshu.com/p/3ef714214405]()

##1.SDK说明与集成
本SDK支持iOS7及以上系统版本，兼容横竖版。

支持i386 armv7 armv7s x86_64 arm64 (内购请用真机)

####1.1 获取参数
从54平台处，获取以下参数，用于对接，配置到sdk的Game_InstallInfo.plist文件中

|参数名|说明|
|----|----|
|appID|填写从平台获取的app_id|
|appKey|appid对应的key|
|tioKey|填写从平台获取的热云appkey   2017/11/22新增  tioKey  talkingKey  二选一填写|
|talkingKey|填写从平台获取的TD appkey   2017/11/28新增tioKey  talkingKey  二选一填写|
|mainURL|一般不需要修改：http://api.54.com/api/cp/user/check|
|isLandscape|是否横屏，1为横屏，0为竖屏|
|clientID|填写从平台获取的client_id|
|clientKey|填写从平台获取的client_key|
|agentgame|渠道名，默认为default，由54.com提供|
|isShowInviteView|是否弹出邀请码的界面，默认是0。|
|isSwitch|1代表上线到appstore的包，0表示没有切换功能，支付直接使用第三方。|
|apple_id|苹果官网注册的应用id,54.com提供|
|URLSchemes|一般为‘h54.com’+‘apple_id’如：h54.com89|

	**注意参数类型一定要都是String类型
####1.2 SDK的组成

	Game_FrameWork.bundle		//sdk所使用的资源
	GameFramework.framework		//sdk的静态库和头文件
	Game_InstallInfo.plist		//SDK的配置文件


####1.3 SDK集成和配置说明
* 1.3.1 将SDK拖入工程主目录下
* 1.3.2 TARGETS->Build Settings->other linker :-ObjC 和-lsqlite3.0
* 1.3.3 TARGETS->Build Settings->Enable Bitcode设置为No
* 1.3.4 在Preprocessor Macros 设置HAVE_CONFIG_H(在build Settings直接搜索即可)
* 1.3.5 TARGETS->Build phases->Link Binary With Libraries链接以下系统库

		1、libz.1.2.5
		2、CoreGraphics
		3、QuartzCore
		4、AdSupport
		5、CoreFoundation
		6、StoreKit
		7、SystemConfiguration
		8、UIKit
		9、Security
		10、libc++
		11、CoreText
		12、CoreTelephony
		13、CFNetwork
		14、AVFoundation
		15、CoreMotion
		16、CoreLocation
		17、libz.1.2.5
		
	如果sdk使用了第三方登录，则还需要链接以下库

		18、ImageIO
		19、libicucore
		20、libiconv
		21、libsqlite3
		22、libstdc++
		
	删掉其他所有有关第三方支付的库
	
* 1.3.6 工程info.plist添加以下配置第三方信息需要自行申请,实例只供参考

		 <key>NSAppTransportSecurity</key>
	   		 <dict>
	       	 <key>NSAllowsArbitraryLoads</key>
	        <true/>
	    </dict>
	    
	   如果是有第三方登录，还需增加微信，qq，微博的配置，参数请从运营方获取
	   
	   	<key>CFBundleURLTypes</key>
 	    <array>
  	        <dict>
 	            <key>CFBundleURLSchemes</key>
 	            <array>
 	                <string>h1tsdk.com11</string>
 	            </array>
 	        </dict>
 	        <dict>
 	            <key>CFBundleTypeRole</key>
 	            <string>Editor</string>
 	            <key>CFBundleURLName</key>
 	            <string>weixin</string>
 	            <key>CFBundleURLSchemes</key>
 	            <array>
 	                <string>wx9efca7e13875a222</string>
 	            </array>
 	        </dict>
 	        <dict>
 	            <key>CFBundleTypeRole</key>
  	            <string>Editor</string>
 	            <key>CFBundleURLSchemes</key>
 	            <array>
 	                <string>tencent101379012</string>
 	            </array>
 	        </dict>
 	        <dict>
  	            <key>CFBundleTypeRole</key>
 	            <string>Editor</string>
 	            <key>CFBundleURLSchemes</key>
   	            <array>
 	                <string>wb1220169449</string>
 	            </array>
 	        </dict>
  	    </array>
  	    
* 1.3.7 URLSchemes: 一般为‘h54.com’+‘apple_id’如：h54.com89’,假如未申请域名,域名位置替换成ip


##2.SDK接口说明

使用SDK之前，需要import

		import<GameFramework/GameFramework.h>
		
在AppDelegate.m加入SDK的这个方法



	- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
	{
	    [Game_Api Game_application:application openURL:url sourceApplication:sourceApplication annotation:annotation];
	    return YES;
	}
	//iOS10以上使用
	- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options
	{
	    [Game_Api Game_application:app openURL:url sourceApplication:nil annotation:nil];
	    return YES;
	}
	- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url
	{
	    [Game_Api Game_application:application openURL:url sourceApplication:nil annotation:nil];
	    return YES;
	}




####2.1 登录
只需调用以下代码即可进入SDK的登入流程，当登入成功，SDK会自动关闭SDK页面，并且回调用户信息给游戏。

	[Game_Api Game_showLoginWithCallBack:^(NSDictionary *responseDic) {
        NSLog(@"%@", responseDic);
        NSLog(@"userid = %@",responseDic[@"userid"]);
        NSLog(@"token = %@",responseDic[@"token"]);
        NSLog(@"agentgame = %@",responseDic[@"agentgame"]);
    }];
    
SDK登录监听，只有在登录成功的时候才会激活回调，登录失败则由sdk处理

####2.2 登出接口
当用户在游戏菜单登出成功的时候，请调用该方法对SDK进行登出操作。

	+ (void)Game_Logout;
登出监听接口

当登出操作是SDK发起时，SDK退出成功block会收到回调。请在该block回调内对游戏进行登出操作。游戏退出成功后可以调出SDK登录窗口。

	+(void)Game_addLogoutCallBlock:(GameMainThreadCallBack)receiverBlock;
	
	
* PS:用户再游戏内可见的登出操作，SDK和游戏都需要同时登出，被挤下线时也需要注意

####2.3. 支付接口

	+(void)Game_sendOrderInfo:(NSDictionary*)info failedBlock:(void(^)())failedBlock;
	
//设置订单信息，信息从游戏厂商服务器获取。

|参数名|说明|
|---|---|
| key_cpOrderid |游戏使用的订单号|
| key_serverID |角色所在的serverid|
| key_serverName | 服务器名|
| key_productID |商品id|
| key_ext |扩展参数,可选|
|key_productPrice|商品金额(单位：元) 建议传入整数,可以保留两位小数|
| key_roleID |角色id|
| key_roleName |角色名字|
| key_productName |商品名字|
|key_productdesc|商品描述，可选|

使用实例

	    NSDictionary *orderInfo = @{
	                                key_cpOrderid: @"testorderid",////单号
	                                key_serverID: @"1",//角色所在的服务器id
	                                key_serverName: @"testserver",//角色所在服务器名
	                                key_productID: @"yb2017111708",//商品id
	                                key_productName: @"60元宝"/*product.localizedTitle*/,
	                                key_productdesc: @"goodsdes"/*商品描述product.localizedDescription*/,
	                                key_ext: @"testattach",//扩展参数，可选
	                                key_productPrice: @"0.01",//商品价格
	                                key_roleID: @"30",//用户id
	                                key_roleName: @"义薄云天叶良辰",//用户昵称
	                                key_currencyName : @"元宝",//货类型名称
	                                };
	    //发起支付，支付失败或取消等失败将在此回调
	    [Game_Api Game_sendOrderInfo:orderInfo failedBlock:^{
	        NSLog(@"支付失败  payfailed");
	    }];
	    
监听支付成功的回调，需要在调用支付之前调用，支付成功之后会返回平台订单号和你们自己服务器的订单号。

	//用于监听支付成功的回调，失败不会触发该回调，需要在发起支付前调用
	[Game_Api Game_sendInfoSuccessedCallBlock:^(NSDictionary *responseDic) {
        NSString *orderid = responseDic[@"orderid"];//小闲订单号
        NSString *cporderid = responseDic[@"cporderid"]; //游戏服务器订单号
        NSLog(@"订单信息：\n%@", responseDic);
	}];

####2.4 设置角色信息接口

	///设置角色信息
	+(void)Game_setRoleInfo:(NSDictionary*)roleInfo;
该接口在能获取以下信息的时候再进行设置即可，一般是登录成功之后。

|参数名|类型|说明|
|---|---|---|
| key_serverID | String | 角色所在的serverid|
| key_serverName | String |服务器名|
| key_partyName | String |公会名字，可选|
| key_roleID | String |角色id|
| key_roleName | String |角色名字|
| key_roleLevel | int |角色等级，如果没有，请填0|
| key_roleVip | int |VIP等级，如果没有，请填0|
| key_roleBalence | float |角色游戏币余额，请填0.0|
| key_dataType | String |数据类型，1为进入游戏，2为创建角色，3为角色升级，4为退出|
| key_rolelevelCtime | String |创建角色的时间 时间戳(无则填大等于1的数字)|
| key_rolelevelMtime | String |角色等级变化时间 时间戳(无则填大等于1的数字)|
| key_currencyName | String |货币名，如金币,元宝,钻石等。|

示例

	[Game_Api Game_setRoleInfo:@{key_dataType : @"1",//数据类型，1为进入游戏，2为创建角色，3为角色升级，4为退出
	                             key_serverID : @"1",//角色所在的serverid
	                             key_serverName : @"疯狂弹幕",//服务器名
	                             key_roleID : @"30",//角色id
	                             key_roleName : @"义薄云天叶良辰",//角色姓名
	                             key_roleLevel : @99,//角色等级
	                             key_roleVip : @3,//角色vip等级
	                             key_roleBalence : @10.2,//角色账户游戏币余额
	                             key_partyName : @"东兴",//角色工会信息
	                             key_rolelevelCtime : @"1479196021",//创建角色时间
	                             key_rolelevelMtime : @"1479196736",//角色等级变化时间
	                             }];


	
###  如出现加载登录界面出现CPU过大
需在调用登录方法获取登录验证结果处发送通知

	[[NSNotificationCenter defaultCenter] postNotificationName:@”endTransFormAnimationWithNotification” object:nil];


	







