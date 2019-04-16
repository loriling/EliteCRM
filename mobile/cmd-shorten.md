#Shorten对象 

###短码操作

1. <span id="Shorten.get">**Shorten.get**</span>: 获取短码
```
/**
 * 短码编码
 * @param content {string|object} 编码的内容
 * @param [ttl] {int} 存活期（单位秒），默认为永久存活
 * @return Promise
 */
get(content, ttl)
```
示例：
```
//免登录打开某个动态页面
Shorten.get({
    ds: 'EliteCRM',//数据源
    epid: 'E00008',//企业id
	ai: 'D0D0PU',//动态页面id
    q: 'sp',//单页面模式
    user: 'luo',//登录名
    pwd: '',//登录密码
	mt: 'r',//手机mobile.web方式
    params: {}
}, 36000).then(function(ret) {
	console.log('shorten url: ' + Shorten.url(ret));
});
```
```
//内置账号打开某个动态页面
Shorten.get({
    ds: 'EliteCRM',//数据源
    epid: 'E00008',//企业id
	ai: 'D0D0PU',//动态页面id
    q: 'sp',//单页面模式
	mt: 'r',//手机mobile.web方式
    params: {}
}, 36000).then(function(ret) {
	console.log('shorten url: ' + Shorten.url(ret));
});
```
```
//打开某个工单
Shorten.get({
	ds:'gl2019',//数据源
    at: 'wo3',//工单
    user: 'lydia',//登录名
    grp: 'SYSTEM',//登录组
    pwd: '1',//密码
    mt: 'r',//手机mobile.web方式
    ai: 'DAE213B9-77EF-4E45-A5C2-3A70846A4814',//工单objective的guid
    cid: '4a4f0d08-3168-4103-b9e1-b5cacc9fd92c',//客户guid
    params: {
        f1: '03293C8B-A293-6EEB-4B55-6308FBB4C8D7'//工单task的guid
    }
});
```

1. <span id="Shorten.url">**Shorten.url**</span>: 获取访问短码url
```
/**
 * 获取访问短码的url
 * @param code
 * @return {string}
 */
url(code)
```

1. <span id="Shorten.make">**Shorten.make**</span>: 获取短码（已经过时）
```
/**
 * 生成直接访问的短链接（必须指定登录用户）
 * @param jsonObject {object}
 *        mobile: {integer}, 可选，指定访问的类型。默认是0，为1时代表访问 /m/nl, 否则访问/nl
 *        language: {string}, 可选，指定语言
 *        mode: {integer}, 可选，模式
 *        user: {string}, 可选，指定用户
 *        pwd: {string}, 可选，指定用户的密码
 *        prj: {string}, 可选，指定登录的项目
 *        grp: {string}, 可选，指定登录的组
 *        epid: {string}, 可选，指定登录的企业号
 *        q: {string}, 可选，运行模式
 *        at: {string}, 可选，页面类型。默认为dyn
 *        ai: {string},  必填，页面ID
 *        ap: {string}, 可选，页面入参
 *        cid: {string}, 可选，客户GUID
 *        params: {string}, 可选，需要提供的更多入参。
 *            参数从f1开始编号，多个参数直接以'&'号分割，例如 'f1=a&f2=23&f3=0'
 * @return Promise 生成的短码字符串
 */
make(jsonObject)
```