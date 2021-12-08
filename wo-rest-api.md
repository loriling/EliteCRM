# 工单接口

[TOC]

工单接口均属于ngs后端工程一部分，因此地址都是以ngs的服务名开头

接口统一采用post方式，参数类型为：application/json (payload模式)

支持通过token方式校验的调用，也支持通过ip白名单方式的校验调用



## 工单接口头信息

### 工单接口校验规则

1. token方式

   请求的head中传递登录ngs后获取的token，即可作为接口校验

   示例：请求的head中添加 

   ```
   token=xxxxxx（ngs登录后获取的token）
   ```

   

2. 接口白名单方式

   在**NgsConfig**的**apiWhiteList**中配置ip白名单

   同时请求的head中传递ds，epid来标识出具体的数据源，通过staffId和groupId来标识处理的人和组

   示例：请求的head中添加 

   ```
   ds=gl2019（ds数据源名）
   staffId=SELITE
   groupId=SYSTEM
   ```



### 其他头信息

```
dbPool=xxx // 数据源dbPool（可选）
```



## 工单操作接口说明



### 工单保存

工单保存操作，如果传递了mapId和customerGuid，则表示新建工单并保存，如果传递的时oId和taskId，则表示加载老的工单并保存

url: http://xxxxx/ngs/wo/save

入参：

```json
{
    "mapId": "xxxxxx", // 工单mapId（新建时必传）
    "customerGuid": "xxxx",// 客户guid（新建时必传）
	"oId": "C37F1819-8963-1CD4-E9C2-9DA64826DDD4", // objective_guid（加载时必传）
	"taskId": "BCDBBD33-BD6A-6168-63CE-4E55620D3CB4", // elitetask_guid（加载时必传）
	"operateTaskGuid": "BCDBBD33-BD6A-6168-63CE-4E55620D3CB4", // 需要处理的elitetask的guid（可选）
	"toStep": "", // 下送到的步骤（可选）
	"toGrp": "", // 下送到的组（可选）
	"toBy": "", // 下送到的人（可选）
	"toRole": "", // 下送到的角色（可选）
	"substatus": "", // 子状态（可选）
	"rolegroupId": "", // 机构id（可选）
	"objective": { // objective表字段（可选）
		"stringfield1": "aaa"
	},
	"task": { // elitetask表字段（可选）
		"event_guid": "bbb"
	},
	"tabc": { // 所有tabc表
		"xxx": { // 某个tabc表的表名（最终sql中，表名会自动拼成：tabcxxx）
			"a": 1 // 其他各种字段
		}
	},
    "step": { // workorderstep3表字段传递
        "ooo": "abc"
    }
}
```

出参：

```json
{
	"code": 1, // 1表示成功 0表示失败
	"message": "", // 错误信息
	"value": null
}
```





### 工单下送

工单下送操作，如果传递了mapId和customerGuid，则表示新建工单并下送，如果传递的时oId和taskId，则表示加载老的工单并下送

url: http://xxxxx/ngs/wo/send

入参：

```json
{
    "mapId": "xxxxxx", // 工单mapId（新建时必传）
    "customerGuid": "xxxx",// 客户guid（新建时必传）
	"oId": "C37F1819-8963-1CD4-E9C2-9DA64826DDD4", // objective_guid（加载时必传）
	"taskId": "BCDBBD33-BD6A-6168-63CE-4E55620D3CB4", // elitetask_guid（加载时必传）
	"operateTaskGuid": "BCDBBD33-BD6A-6168-63CE-4E55620D3CB4", // 需要处理的elitetask的guid（可选）
	"toStep": "", // 下送到的步骤（可选）
	"toGrp": "", // 下送到的组（可选）
	"toBy": "", // 下送到的人（可选）
	"toRole": "", // 下送到的角色（可选）
	"substatus": "", // 子状态（可选）
	"rolegroupId": "", // 机构id（可选）
	"objective": { // objective表字段（可选）
		"stringfield1": "aaa"
	},
	"task": { // elitetask表字段（可选）
		"event_guid": "bbb"
	},
	"tabc": { // 所有tabc表
		"xxx": { // 某个tabc表的表名（最终sql中，表名会自动拼成：tabcxxx）
			"a": 1 // 其他各种字段
		}
	},
    "step": { // workorderstep3表字段传递
        "ooo": "abc"
    }
}
```

出参：

```json
{
	"code": 1, // 1表示成功 0表示失败
	"message": "", // 错误信息
	"value": null
}
```





### 工单成功结案

url: http://xxxxx/ngs/wo/close

入参：

```json
{
	"oId": "C37F1819-8963-1CD4-E9C2-9DA64826DDD4", // objective_guid
	"taskId": "BCDBBD33-BD6A-6168-63CE-4E55620D3CB4" // elitetask_guid
}
```

出参：

```json
{
	"code": 1, // 1表示成功 0表示失败
	"message": "", // 错误信息
	"value": null
}
```





### 工单失败结案

url: http://xxxxx/ngs/wo/failedClose

入参：

```json
{
	"oId": "C37F1819-8963-1CD4-E9C2-9DA64826DDD4", // objective_guid
	"taskId": "BCDBBD33-BD6A-6168-63CE-4E55620D3CB4", // elitetask_guid
	"objectiveStatus": "xxx" // 默认不传递是FAILED，也可以自定义传递（可选）
}
```

出参：

```json
{
	"code": 1, // 1表示成功 0表示失败
	"message": "", // 错误信息
	"value": null
}
```

