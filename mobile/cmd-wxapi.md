#WXAPI类（app中特有）

###使用微信公众平台的移动接入方案，实现与微信app互动的相关功能

1. <span id="WXAPI.sendTextMsg">**WXAPI.sendTextMsg**</span>: 分享文字信息
```
/**
 * 分享文字消息
 * @param text 消息内容
 * @param scene 场景 0:好友 1:朋友圈 2:收藏
 * @return {Promise} promise的返回1，表示调用成功，0表示失败
 */
sendTextMsg(text, scene = 0)
```
示例：
```
WXAPI.sendTextMsg('你好').then(function(ret){
	if (ret === 1) {
		$F.notify('分享成功');
	} else {
		$F.notify('分享失败');
	}
});
```

1. <span id="WXAPI.sendUrlMsg">**WXAPI.sendUrlMsg**</span>: 分享url消息
```
/**
 * 分享url消息
 * @param url url地址
 * @param title url标题
 * @param desc url描述
 * @param thumbImg 缩略图url
 * @param scene 场景 0:好友 1:朋友圈 2:收藏
 * @return {Promise}
 */
sendUrlMsg(url, title, desc, thumbImg = '', scene = 0)
```
示例：
```
WXAPI.sendUrlMsg('http://www.elitecrm.com', '过河兵官网', '过河兵公司官方网站', 'http://dev.elitecrm.com/ngs/fs/get?file=headimg/1546598743984.png').then(function(ret){
	if (ret === 1) {
		$F.notify('分享成功');
	} else {
		$F.notify('分享失败');
	}
});
```

1. <span id="WXAPI.sendImgMsg">**WXAPI.sendImgMsg**</span>: 分享图片消息
```
/**
 * 分享图片消息
 * @param imgUrl 图片url
 * @param scene 场景 0:好友 1:朋友圈 2:收藏
 * @return {Promise}
 */
sendImgMsg(imgUrl, scene = 0)
```
示例：
```
WXAPI.sendImgMsg('http://dev.elitecrm.com/ngs/fs/get?file=headimg/1546598743984.png').then(function(ret){
	if (ret === 1) {
		$F.notify('分享成功');
	} else {
		$F.notify('分享失败');
	}
});
```