#ElasticSearch在NGS中的使用

## 一. 安装elasticsearch服务
所有需要的文件，dev的ftp中都可以下载到

###1. 先下载elasticsearch服务：https://www.elastic.co/downloads/elasticsearch （就写此文档的时候，elasticsearch服务版本为5.5.0）
解压后，配置 config/elasticsearch.yml
在文件最后添加如下配置，用来支持http-restful接口的跨域请求

	http.cors.enabled: true
	http.cors.allow-origin: "*"

通过bin/elasticsearch.bat （linux为elasticsearch.sh）启动服务。



###2. 下载ik-analyzer：https://github.com/medcl/elasticsearch-analysis-ik

找到elasticsearch/plugins目录，在这个目录下创建ik目录，然后把下载的ik-analyzer相关文件解压到此目录中
重启elasticsearch服务即可起效。

###3. elasticsearch集群部署
如果需要集群部署，就需要修改 config/elasticsearch.yml

	# ---------------------------------- Cluster -----------------------------------
	#
	# Use a descriptive name for your cluster:
	#
	cluster.name: elite-application
	#
	# ------------------------------------ Node ------------------------------------
	#
	# Use a descriptive name for the node:
	#
	node.name: node-1
	# ---------------------------------- Network -----------------------------------
	#
	# Set the bind address to a specific IP (IPv4 or IPv6):
	#
	#network.host: 192.168.0.1
	#
	# Set a custom port for HTTP:
	#
	http.port: 9200
	transport.tcp.port: 9300
	# --------------------------------- Discovery ----------------------------------
	#
	# Pass an initial list of hosts to perform discovery when new node is started:
	# The default list of hosts is ["127.0.0.1", "[::1]"]
	#
	discovery.zen.ping.unicast.hosts: ["127.0.0.1:9300", "127.0.0.1:9301"]

**cluster.name** 需要两台一致，表示这两台是一个集群中的
**node.name** 需要两台不同，分别表示自己节点的名称
**http.port和transport.tcp.port** 如果需要修改，可以自己修改，http.port表示http的restful接口的端口，transport.tcp.port表示集群间tcp通讯的端口
**discovery.zen.ping.unicast.hosts** 这个配置的就是每个节点对应的ip地址和transport.tcp.port的地址，告知集群中一共有哪几个节点

###4. 通过elasticsearch-head来监控与管理elasticsearch集群
需要有安装node，解压相关文件，然后按如下步骤打开即可
- cd elasticsearch-head
- npm install
- npm run start
- open http://localhost:9100/

###5. 通过http接口创建一些索引和一些映射 （可选配置）
在使用之前，我们可能需要先创建索引，并对某些字段设置mapping。如果这里不直接通过restful接口配置，则需要后面在浏览器里调用js方法去配置

	//创建一个名字为chat的index，并且设置了mappings，type为1的里面，属性是content时候，使用ik_max_word这个分词器
	//可以通过fiddler或者postman工具，发出一个put请求，内容如下示例
	PUT http://139.196.108.236:9200/chat
	body为：
	{
	  "mappings": {
	    "1": {
	      "properties": {
	        "content": {
	          "type":     "text",
	          "analyzer": "ik_max_word"
	        }
	      }
	    }
	  }
	}


## 二. 客户端配置与使用 

引入系统参数ELASEA，配置成elasticsearch服务的创建config json字符串比如：{"hosts":"139.196.108.236:9200","log":"trace"}
这样就表示启用elasticsearch服务，客户端就会去加载相关js文件
通过：**$E.getActiveProject().esc** 就可以获取到当前elasticsearchclient对象，该对象包括如下方法：

1.创建索引和mapping

	//创建一个叫做chat的索引，并配置type为1中字段是content的类型是text，且analyzer是ik的分词器
	//这个只需要创建一次，第一次创建完了之后就可以使用了，可以自己在chrome的console执行以下
	//注意这里的ik_max_word: 中文分词器ik_analyzer提供了两种分词策略： 
	//ik_max_word和ik_smart，区别在于ik_max_word会尽量多的去分，而ik_smart则更智能的去分，这里可以自行选择一种
	$E.getActiveProject().esc.client.indices.create({
		index: "chat",
		body: {
			"mappings": {
				"1":{
					"properties": {
						"content": {
							"type" : "text", 
							"analyzer" : "ik_max_word" 
						}
					}
				}
			}
		}
	});

> ik_max_word: 繁琐会被分词为： 繁琐，繁，琐
> ik_smart: 繁琐会被分词为：繁琐

	可以通过http接口：http://localhost:9200/chat/_analyze?analyzer=ik_smart&pretty=true 自己去查看分词结果
	body:
	{
		"text":"繁琐"
	}

	response:
	{
	    "tokens": [
	        {
	            "token": "繁琐",
	            "start_offset": 0,
	            "end_offset": 2,
	            "type": "CN_WORD",
	            "position": 0
	        }
	    ]
	}


2.查询方法

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



3.创建或者更新索引方法

	/**
	 * 对内容做索引，创建或者更新
	 * @param id 索引id
	 * @param content 索引内容
	 * @param callback 回调(optional)
	 * @param index 索引类型(optional)
	 * @param type 索引类型(optional)
	 */
	index : function (id, content, callback, index, type)

4.删除索引方法

	/**
	 * 删除索引
	 * @param id 索引id
	 * @param callback 回调(optional)
	 * @param index 索引类型(optional)
	 * @param type 索引类型(optional)
	 */
	deleteIndex : function(id, callback, index, type)

5.elasticsearch还有很多功能，从esc对象上还可以拿到标准的elasticsearch的client对象，就可以直接调用其本身的api

	$E.getActiveProject().esc.client

可以查阅相关文档：[https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/api-reference.html#api-cluster-getsettings](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/api-reference.html#api-cluster-getsettings) 来获取到client对象上所有的方法