#BCR对象（app中特有）

###名片扫描类
当使用了合合信息名片识别功能时候，可以通过此类的方法来做名片扫描

1. <span id="BCR.scan">**BCR.scan**</span>: 打开名片扫描
```
/**
 * 打开名片扫描，通过摄像头扫描或者拍摄名片进行识别或者选择现有照片扫描
 *
 * @param [fsKey] 扫描名片图片上传文件fsKey
 * @param [filePath] 扫描名片图片文件上传路径
 * @param [fileName] 扫描名片图片文件上传名
 * @return {Promise}
 *
 * 示例：
 *  BCR.scan().then(function(ret){
        console.log(ret);
    }).catch(function(err){
        console.log("error: " + err);
    });
 */
scan(fsKey, filePath, fileName)
```