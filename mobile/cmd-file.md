#File对象 

###文件操作类

1. <span id="File.upload">**File.upload**</span>: 上传文件
```
/**
 * 上传文件
 * @method upload
 * @param fsKey {string} 文件存储服务名，为空时取默认服务
 * @param filePath {string} 文件存储相对路径，为空时存储于temp临时目录，作为临时文件存在
 * @param fileName {string} 文件存储名，相同时覆盖。为空时由系统自动生成存储名
 * @param cmdId {string} 文件上传完毕时执行的回调命令组或者方法，执行结果为：R.RESULT{CODE, MESSAGE, VALUE}
 * @param options {object}
 *        {
             mediaTypes : 'All',//媒体类型包括了：Images, Videos, All，默认是All
             allowsEditing : true,//是否可编辑图片，默认可编辑
             quality : 1.0,//图片压缩比，0到1,1表示不压缩，默认1
             aspect : [4, 3],//如果allowsEditing配置了true，则会出现的裁剪框大小（ios不起作用，如果ios设置了可编辑，那么就是一个正方形的框框）
             base64 : false,//返回是不是图片的base64码，默认false
             mode: 0 //0 有拍照有相册， 1只有相册， 2只有拍照   默认0
			 multiple: false//true表示允许多选图片，如果是多选图片，返回的RESULT中VALUE也是一个数组，表示了每个上传图片的信息（为true时候，aspect,mode参数无效）默认false
          }
 * @return {boolean} 文件上传为异步操作，返回值为提交是否成功
 */
upload(fsKey, filePath, fileName, cmdId, options)
```

1. <span id="File.remove">**File.remove**</span>: 删除文件
```
/**
 * 删除文件
 * @method remove
 * @param filePath {string} 文件上传时获得的存储路径
 * @param cmdId {string} 文件删除后的回调方法，执行结果为：R.RESULT{CODE, MESSAGE, VALUE}
 */
remove(filePath, cmdId)
```

1. <span id="File.getName">**File.getName**</span>: 从URL或者文件路径中获取文件名
```
/**
 * 从URL或者文件路径中获取文件名
 * @method getName
 * @param filepath {string} URL或者完整的文件路径
 * @return {string}
 */
getName(filepath)
```

1. <span id="File.getSimpleName">**File.getSimpleName**</span>: 从URL或者文件路径中获取不带后缀的文件名
```
/**
 * 从URL或者文件路径中获取不带后缀的文件名
 * @method getSimpleName
 * @param filepath {string} URL或者完整的文件路径
 * @return {string}
 */
getSimpleName(filepath)
```

1. <span id="File.getExt">**File.getExt**</span>: 从URL或者文件路径中获取文件名后缀
```
/**
 * 从URL或者文件路径中获取文件名后缀
 * @method getExt
 * @param filepath {string} URL或者完整的文件路径
 * @return {string}
 */
getExt(filepath)
```

1. <span id="File.imgView">**File.imgView**</span>: 展示图片
手机特有方法
```
/**
 * 展示图片
 * @param photos {string|array} 支持传递单个图片url字符串，也支持传递数组
 */
imgView(photos)
```

1. <span id="File.open">**File.open**</span>: 打开一个url
手机特有方法
```
/**
 * 通过OpenFile.openDoc方式，打开一个url (支持类型包括：doc, docx, xls, xlsx, ppt, pptx, pdf, mp4)
 * 如果非http开头，会自动拼接上ngs的fs的url前缀
 * @param fileUrl 文件url路径
 * @param fileName 文件名
 * @param fileType 文件类型
 * @param [callback] 回调，第一个参数是error，第二个参数是打开的实际url
 */
open(fileUrl, fileName, fileType, callback)
```

1. <span id="File.download">**File.download**</span>: 下载文件到本地路径
手机特有方法
```
/**
 * 下载文件到本地路径
 * @param url 下载文件url
 * @param [fileName] 存本地的文件名
 * @param [headers] 下载请求的headers
 * @return {Promise}
 */
download(url, fileName, headers)
```

1. <span id="File.uploadLocal">**File.uploadLocal**</span>: 上传本地文件
手机特有方法
```
/**
 * 上传本地文件
 * @param uri 本地文件路径
 * @param [fsKey]
 * @param [filePath] 上传路径
 * @param [fileName] 文件名
 * @param [fileType] 文件扩展名
 * @return {Promise}
 */
uploadLocal(uri, fsKey, filePath, fileName, fileType)
```

1. <span id="File.toImgDataURL">**File.toImgDataURL**</span>: 获取图片base64
```
/**
 * 图片url转base64
 * 如果是mobile.web，必须是同域下的图片，或者其他域图片允许跨域的情况（Access-Control-Allow-Origin "*"）
 * @param imgUri 图片uri
 * @return Promise
 */
toImgDataURL(imgUri)
```

1. <span id="File.saveFragment">**File.saveFragment**</span>: 保存片段
```
/**
 * 保存文本片段到文件服务器
 *
 * @method saveFragment
 * @param content {string} 文本片段字符串
 * @param [cmdId] {string | int} 回调命令，命令中通过R.RESULT获取文件名
 * @param [path] {string} 指定存储路径
 * @param [saveName] {string} 指定存储文件名
 * @return Promise
 */
saveFragment(content, cmdId, path, saveName)
```

1. <span id="File.readFragment">**File.readFragment**</span>: 读取片段
```
/**
 * 从文件服务器读取文本片段
 *
 * @method readFragment
 * @param filepath {string} 保存文本片段获取的文件存储路径
 * @param [cmdId] {string | int} 回调命令，命令中通过R.RESULT获取文本片段字符串
 * @return Promise
 */
readFragment(filepath, cmdId) 
```