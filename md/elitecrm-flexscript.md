# EliteCRM #
[https://github.com/loriling/EliteCRM](https://github.com/loriling/EliteCRM "View project on GitHub")

## FlexClient ##

**Flex客户端常见卡死情况与排除**

1.打开fiddler，查看网络请求中是否有哪个请求是长时间没有响应的。由于客户端请求绝大多数是同步请求，因此发出请求没有响应，则客户端现象就是浏览器僵死。

2.可以先尝试使用非软电话登录，如果在非软电话清空下，不会卡死，则多数卡死原因出在软电话中。
因为软电话与后台服务连接也都是同步的请求，如果与后台aps或者cti之间网络存在些许异常，就可能出现客户端浏览器卡死现象。
软电话与APS连接查看EliteServerProxy.log，软电话与CTI连接查看ComCTIClient.log和CTIClient.log，有的地方还需要查看对应的CTIExtra日志

3.更换浏览器尝试。
也许刚好你用的ie本身就有问题，所以可以试试360浏览器，遨游浏览器等等是不是可以正常，如果可以，则可能需要重装ie或者升级ie版本。

4.使用debug版本flashplayer（使用debug的目的是让客户端日志可以输出错误堆栈，有助于快速定位问题）
EliteClient.log中也许会出现：
    #1502: 脚本的执行时间已经超过了 15 秒的默认超时设置。
类似这样的错误日志。这句日志就可以看出客户端flex调用某个方法执行时间超过了flex内置的超时时长（日志中看到15秒，其实应该是1分钟）。
也就是说，这句报错说明了当前这句命令，造成了客户端卡死。如果可以看到日志中有类似：begin call xxxxx 。。那也就可以判定就是这个xxxx造成了调用超时，具体原因就可以进一步分析。

5.去现场之前记得先下载好最新的debug版本flashplayer，以便出了问题后安装上去，方便看日志。
下载地址：<a href="http://www.adobe.com/support/flashplayer/downloads.html">http://www.adobe.com/support/flashplayer/downloads.html</a>
记得是下载activex for ie的。。。。

6.客户端清空浏览器缓存，这个有的ie下，清空时候有个勾选是否保留收藏夹中内容的，很多人勾上了这个然后又刚好把登录地址放在收藏夹内，这样是删不掉缓存的，要去掉这个勾选。
其实，只需要勾选上"Internet 临时文件"即可，其他都不用勾。
如果确认了删了缓存了，还是出现版本不对问题，可以尝试查看ie是否设置了代理，也许是代理那边有缓存。


****
**web应用与它们的日志**

1.首先去看log4j.properties文件的配置，看是不是有日志输出，是不是没加上,R

2.如果一个应用根本就没能成功起来，那就可以先配置成输出所有日志：log4j.rootLogger=debug,stdout,R ，这样可以看到第三方jar中输出的所有日志，方便问题排查。

3.tomcat有个manager界面，可以管理webapps下的应用，这里可以看到应用是否正常启动。可以通过tomcat主页进入manager界面，用户名密码再tomcat-user.xml中配置

****
**FlexScript Bible**

1.关于软电话操作的那些FlexScript：<br>

(1)拨打电话:

	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.makeCall("8003","xxxx=123|oooo=456");

(2)挂断电话:

	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.releaseCall();

(3)接起电话:

	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.answerCall();
		
(4)发起会议:

	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.initConference("8003","xxxx=123|oooo=456");
			
(5)完成会议:
			
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.compConference();
		
(6)发起转接:
			
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.initTransfer("8003","xxxx=123|oooo=456");
		
(7)完成转接:

	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.compTransfer();

(8)单步转:
	
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.muteTransfer("8003","xxxx=123|oooo=456");
		
(9)接回:
	
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.reconnect();
		
(10)通知进入工作:
			
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.notifyWorkStart("WT-OB");//WT-OB代表外拨 WT-IB代表呼入
		
(11)通知结束工作:
			
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.notifyWorkEnd();
		
(12)软电话的CommonCall:
			
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.commonCall("callName","callParam");
		
(13)获取随路数据:
			
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.attachData("key");
			
(14)发送消息给IVR:

	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.sendIVRMessage("message");
		
(15)软电话退出:

	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.quitSoftphone();
		
(16)软电话准备好:

	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.agentReady();

(17)软电话未准备好:
			
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.agentNotReady(3);//3是默认的AUX状态

(18)强制话务员登出:
			
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.forceAgentLogout("dn","agent_id");

(19)强制话务员未准备好:
			
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.forceAgentNotReady("dn","agent_id");
		
(20)强制话务员准备好:
			
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.forceAgentReady("dn","agent_id");

(21)监听:

	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.listen("dn");
		
(22)拦截:
	
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.intercept("agent_id");
		
(23)根据dn号强制断开agent连接:
			
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.forceDisconnectAgentByDN("dn");

(24)强制挂断:

	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.forceHangUp();
			
(25)强插:
			
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.bargeIn("dn");
		
(26)接管:
	
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.takeOver("dn","agent_id");
		
(27)暗语:
			
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.coach("dn","agent_id");
		
(28)静音:
			
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.mute();

(29)取消静音:
			
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.unmute();
			
(30)发送DTMF:

	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.sendDTMF("1");
		
(31)软电话登录:
			
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.login();
		
(32)软电话登出:
			
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.logout();

(33)开始播放:

	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.startPlay("f:/temp/xxx.wav");
		
(34)结束播放:

	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.stopPlay();
		
(35)获取话务员状态:
			
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.getAgentState();
	//UNKNOWN = 0;			LOGOUT = 1;			READY = 2;			NOTREADY = 3;			BUSY = 4;			MAX = 5;

(36)获取话务员模式:
	
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.getAgentState();
	//UNKNOWN=0;			AUTOIN=1;			MANUALIN=2;			AUX=3;			ACW=4;			NOCALLDISCONNECT=5;			MAX=6;
		
(37)获取电话状态：

	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.getCallState2();
	//UNKNOWN=0;		RINGING=1;		TALKING=2;		HOLD=3;		DIALING=4;	TALKINGHOLD=32;			DIALINGHOLD=34;
		
(38)获取当前record_guid：
			
	import Elite.EXSoftPhone.EXSoftPhone;
	EXSoftPhone.getRecordGuid();

2.FlexScript添加js代码并调用：

    import flash.external.ExternalInterface;
	var insertScript:String = "document.insertScript = function() {"+
		"if (document._test==null){" + 
			"_test = function(){" +
				"alert('test');"+//具体_test方法做的事情
			"}"+
		"}"+
	"}";
	ExternalInterface.call(insertScript);//将上述定义的js脚本插入到dom中
	ExternalInterface.call("_test");//调用插入的js方法

3.FlexScript打开某个URL：

    import Elite.ECCore.ECore;
	import Elite.ECCore.Addin;
	var param:Object=new Object();
	param.url="http://about.me/loriling";
	//这个reload如果传递了1，则打开此页面时候会刷新，如果不是，则不会刷新，还是显示之前的内容
	param.reload=1;
	//这里第二个参数some_unique_id是自己起的一个id，对应于某一个tab页，如果想同时打开多个页面，则这个id需要不同
	ECore.addinDoByInstanceId(Addin.AddinBrowser,"some_unique_id",ECore.ECM_SHOW,"flexscript",param);

4.FlexScript弹出浮动窗口：

	import Elite.util.AlertUtil;
	AlertUtil.showScripOverIFrame("您有【100023】条过期的本组--工单--需要处理","标题");

5.FlexScript获取登录时输入的密码，并进行MD5加密：
	
	import Elite.ECCore.ECore;
	import Elite.util.DigestUtils;
	var inputPassword:String = ECore.loginInfo.inputPassword;//获取到输入密码
	DigestUtils.md5Hex(inputPassword);//对密码进行md5加密

6.FlexScript添加客户端任务栏标签闪烁功能

	import flash.external.ExternalInterface;
	//定义插入ocx方法
	var INSERT_FUNCTION_CREATEFLASHMEOCX:String = 
		"document.insertScript = function () {" +
	    	"if (document._createFlashMeOCX==null){" + 
	      		"_createFlashMeOCX = function(){" +
	      		"var bodyID = document.getElementsByTagName(\"body\")[0];" +
	      		"if(document._flashMeOCX!=null){"+
	       			"return true;"+
	      		"}"+
	      		"var flashMeOCX = document.createElement('object');"+
	      		"flashMeOCX.classid='CLSID:9ADB7F92-97BB-4F04-A094-0E4EFAC857A8';"+
	      		"flashMeOCX.style.width=0;"+
	      		"flashMeOCX.style.height=0;"+
	      		"flashMeOCX.id=\"_flashMeOCX\";"+
	      		"bodyID.appendChild(flashMeOCX);"+
	      		"var result=(document._flashMeOCX!=null);"+
	      		"return result;"+
	     	"}"+
    	"}"+
	"}";
	//定义插入调用flashMe的方法
	var INSERT_FUNCTION_FLASHME:String = "document.insertScript = function() {"+
	    "if (document._flashMe==null){" + 
	        "_flashMe = function(){" +
	            "var flashMeOCX = document.getElementById(\"_flashMeOCX\");"+
	            "if(flashMeOCX != null){"+
	              "flashMeOCX.BeginFlash();"+
	            "}"+
	        "}"+
	    "}"+
	"}";

	//执行插入flashMe这个ocx到页面
	ExternalInterface.call(INSERT_FUNCTION_CREATEFLASHMEOCX);
	//执行插入flashMe这个方法
	ExternalInterface.call(INSERT_FUNCTION_FLASHME);


	//下面就是每次要闪烁时候调用：
	import flash.external.ExternalInterface;
	ExternalInterface.call("_flashMe");


`EliteCRM by loriling`


<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','//www.google-analytics.com/analytics.js','ga');

  ga('create', 'UA-51025730-1', 'loriling.github.io');
  ga('send', 'pageview');

</script>
