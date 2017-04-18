#ElasticSearch在NGS中的使用

引入系统参数ELASEA，配置成elasticsearch服务的创建config json字符串比如：{"hosts":"139.196.108.236:9200","log":"trace"}
这样就表示启用elasticsearch服务，客户端就会去加载相关js文件
通过：**$E.getActiveProject().esc** 就可以获取到当前elasticsearchclient对象，该对象包括如下方法：

1.查询方法

	/**
	 * 根据内容检索
	 * @param content 需要查询的内容，可以是字符串，也可以是json对象，字符串的话，就代表去查询字段是content上的内容，如果要查询其他字段上内容，就要自己传递json对象：
	 * {
			query: {
				match: {
					content: content
				}
			}
		}
	 * @param successFunc
	 * @param failFunc
	 * @param index 索引名称
	 * @param type 索引类型
	 * @returns {*|Promise.<T>}
	 */
	search : function(content, successFunc, failFunc, index, type)

	下面举个例子：
	esc.search('服务', function (resp) {
        var hits = resp.hits.hits;
        console.log('search[服务]', hits);
    }, function(err){
        console.error(err);
    });



2.创建或者更新索引方法

	/**
	 * 对内容做索引，创建或者更新
	 * @param id 索引id
	 * @param content 索引内容
	 * @param callback 回调(optional)
	 * @param index 索引类型(optional)
	 * @param type 索引类型(optional)
	 */
	index : function (id, content, callback, index, type)

3.删除索引方法

	/**
	 * 删除索引
	 * @param id 索引id
	 * @param callback 回调(optional)
	 * @param index 索引类型(optional)
	 * @param type 索引类型(optional)
	 */
	deleteIndex : function(id, callback, index, type)

4.elasticsearch还有很多功能，从esc对象上还可以拿到标准的elasticsearch的client对象，就可以直接调用其本身的api

	$E.getActiveProject().esc.client

可以查阅相关文档：[https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/api-reference.html#api-cluster-getsettings](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/api-reference.html#api-cluster-getsettings) 来获取到client对象上所有的方法