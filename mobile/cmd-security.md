#Security对象 

###加解密操作，需要注意的是，其中需要访问服务器的几个方法，手机端都返回的Promise对象

1. <span id="Security.base64Encode">**Security.base64Encode**</span>: Base64加密
```
/**
 * Base64加密
 * @method base64Encode
 * @param content {string} 进行base64加密的字符串
 * @return {string}
 */
base64Encode: function (content)
```

1. <span id="Security.base64Decode">**Security.base64Decode**</span>: Base64解密
```
/**
 * Base64解密
 * @method base64Decode
 * @param content {string} 进行base64解密的字符串
 * @return {string}
 */
base64Decode: function (content)
```

1. <span id="Security.md5">**Security.md5**</span>: MD5加密
```
/**
 * MD5加密
 * @method md5
 * @param content {string} 需要加密的字符串
 * @return {string}
 */
md5: function (content)
```

1. <span id="Security.aesEncrypt">**Security.aesEncrypt**</span>: AES加密
```
/**
 * AES加密
 * 改成返回promise
 * @method aesEncrypt
 * @param content {string} 需要加密的字符串
 * @param [password] {string} 加密秘钥
 * @return {Promise}
 */
aesEncrypt: function (content, password)
```

1. <span id="Security.aesDecrypt">**Security.aesDecrypt**</span>: AES解密
```
/**
 * AES解密
 * 改成返回promise
 * @method aesDecrypt
 * @param content {string} 需要解密的字符串
 * @param [password] {string} 解密秘钥
 * @return {Promise}
 */
aesDecrypt: function (content, password)
```

1. <span id="Security.desEncrypt">**Security.desEncrypt**</span>: DES加密
```
/**
 * DES加密
 * 改成返回promise
 * @method desEncrypt
 * @param content {string} 需要加密的字符串
 * @param [password] {string} 加密秘钥
 * @return {Promise}
 */
desEncrypt: function (content, password)
```

1. <span id="Security.desDecrypt">**Security.desDecrypt**</span>: DES解密
```
/**
 * DES解密
 * 改成返回promise
 * @method desDecrypt
 * @param content {string} 需要解密的字符串
 * @param [password] {string} 解密秘钥
 * @return {Promise}
 */
desDecrypt: function (content, password)
```

1. <span id="Security.shaEncrypt">**Security.shaEncrypt**</span>: SHA-256加密
```
/**
 * SHA-256加密
 * 改成返回promise
 * @method shaEncrypt
 * @param content {string} 需要加密的字符串
 * @param [password] {string} 加密秘钥
 * @return {Promise}
 */
shaEncrypt: function (content, password)
```

1. <span id="Security.sha1Encrypt">**Security.sha1Encrypt**</span>: SHA1加密
```
/**
 * SHA1加密
 * 改成返回promise
 * @method shaEncrypt
 * @param content {string} 需要加密的字符串
 * @return {Promise}
 */
sha1Encrypt: function (content) 
```

1. <span id="Security.kasEncrypt">**Security.kasEncrypt**</span>: 凯撒加密算法
```
/**
 * 凯撒加密算法，用于低安全的无秘钥加密
 * 改成返回promise
 * @method kasDecrypt
 * @param content {string} 需要加密的字符串
 * @return {Promise}
 */
kasEncrypt: function (content)
```

1. <span id="Security.kasDecrypt">**Security.kasDecrypt**</span>: 凯撒解密算法
```
/**
 * 凯撒解密算法
 * 改成返回promise
 * @method kasEncrypt
 * @param content {string} 需要解密的字符串
 * @return {Promise}
 */
kasDecrypt: function (content) 
```

1. <span id="Security.rsaEncrypt">**Security.rsaEncrypt**</span>: RSA以公钥进行的加密算法
```
/**
 * RSA以公钥进行的加密算法
 * 改成返回promise
 * @method rsaDecrypt
 * @param content
 * @param [key] {string} 可选，加密的秘钥，默认时使用系统配置的RSA秘钥
 * @return {Promise}
 */
rsaEncrypt: function (content, key)
```

1. <span id="Security.rsaDecrypt">**Security.rsaDecrypt**</span>: RSA以公钥进行的解密算法
```
/**
 * RSA以公钥进行的解密算法
 * 改成返回promise
 * @method rsaEncrypt
 * @param content {string} 需要解密的字符串
 * @param [key] {string} 可选，解密的秘钥，默认时使用系统配置的RSA秘钥
 * @return {Promise}
 */
rsaDecrypt: function (content, key)
```