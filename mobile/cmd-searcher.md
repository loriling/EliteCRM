#Searcher对象 

###ES全文检索操作类
这里封装了一些elasticsearch操作的方法，如果想要更多es方法，可以直接使用$project.esc对象上的方法

1. <span id="Searcher.index">**Searcher.index**</span>: 新增或者更新索引
```
/**
 * 对内容做索引，创建或者更新
 * @param id 索引id
 * @param content 索引内容
 * @param cmdId 回调命令组的id
 * @param index 索引类型(optional)
 * @param type 索引类型(optional)
 */
index(id, content, cmdId, index, type)
```

1. <span id="Searcher.search">**Searcher.search**</span>: 查询索引
```
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
 //满足woId字段必须是xxxx，同时content字段全文检索"可以"
 {
        	"query": {
			    "bool" : {
			      "must" : {
			        "term" : { "woId" : "xxxx" }
			      },
			      "filter": {
			        "match" : { "content" : "可以" }
			      }
			    }
			}
		}
 //检索user_id字段等于SELITE，同时内容存在xxx的
 {
		  "query": {
		    "bool": {
		      "must": [
		        { "match": { "user_id": "SELITE" } },
		        { "match": { "content": "xxx" } }
		      ]
		    }
		  }
		}
 * @param cmdId 回调命令组的id 成功会有RESULT，失败会有ERROR
 * @param index 索引类型(optional)
 * @param type 索引类型(optional)
 * @returns {*|Promise.<T>}
 */
search(content, cmdId, index, type)
```

1. <span id="Searcher.deleteIndex">**Searcher.deleteIndex**</span>: 删除索引
```
/**
 * 删除索引
 * @param id 索引id
 * @param [cmdId] 回调(optional)
 * @param [index] 索引类型(optional)
 * @param [type] 索引类型(optional)
 */
deleteIndex(id, cmdId, index, type)
```