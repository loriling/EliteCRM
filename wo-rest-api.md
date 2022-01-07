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
    "handleType": "", // 处理类型（可选: G,R,GR。默认根据下一个节点上配置的来，如果传递，则按传递的来）
    "sendType": 1, // 下送类型（可选）详细说明看下面的HANDLETYPE与SENDTYPE说明
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



HANDLETYPE与SENDTYPE说明：

```
handletype总共分为3种：
G（组）；
R（角色）；
GR（组+角色）

G（组）：
组权限和我们旧版的权限很类似，一共有5种权限方式：
1.指定部门（组）	----下送给指定组
SENDTYPE=1
2.指定人					----下送给指定人
SENDTYPE=2
3.原步骤-步骤X-人或组	----下送给原步骤X的处理人或者处理组
SENDTYPE=4
4.直属上级部门			----下送给所属部门的直属上级部门
SENDTYPE=7
5.当前组组长				----下送给组长（该员工等级大于管理台issupe参数，为组长。员工等级需要看对应字段issupervisor）
SENDTYPE=9

R（角色）
角色权限是新增的权限之一，主要针对角色这一特殊变量：
1.指定角色				----下送给指定角色内的人员
SENDTYPE=3
2.原步骤-步骤X-人或角色	----下送给原步骤X的处理人或者处理角色
SENDTYPE=4

GR（组+角色）
通过组，角色，由此产生了组+角色的权限：
1.原步骤-步骤X-人或组+角色		----下送给原步骤X的处理人或者处理的组+角色SENDTYPE=4
2.组+角色						----下送给指定组+角色
SENDTYPE=10
```



### 工单成功结案

url: http://xxxxx/ngs/wo/close

入参：

```json
{
	"oId": "C37F1819-8963-1CD4-E9C2-9DA64826DDD4", // objective_guid
	"taskId": "BCDBBD33-BD6A-6168-63CE-4E55620D3CB4", // elitetask_guid
    "objective": { // objective表字段（可选）
		"stringfield1": "aaa"
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





### 工单失败结案

url: http://xxxxx/ngs/wo/failedClose

入参：

```json
{
	"oId": "C37F1819-8963-1CD4-E9C2-9DA64826DDD4", // objective_guid
	"taskId": "BCDBBD33-BD6A-6168-63CE-4E55620D3CB4", // elitetask_guid
	"objectiveStatus": "xxx", // 默认不传递是FAILED，也可以自定义传递（可选）
    "objective": { // objective表字段（可选）
		"stringfield1": "aaa"
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



### 工单催单

新增一条step记录

url: http://xxxxx/ngs/wo/reminder

入参：

```json
{
	"oId": "C37F1819-8963-1CD4-E9C2-9DA64826DDD4", // objective_guid
	"taskId": "BCDBBD33-BD6A-6168-63CE-4E55620D3CB4", // elitetask_guid
    "comments": "xxxxx", // 备注信息
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



### 工单回退

按下送的路径，回退工单

如果不传递operateTaskGuid，则删除当前task后，不会新建task

url: http://xxxxx/ngs/wo/back

入参：

```json
{
	"oId": "C37F1819-8963-1CD4-E9C2-9DA64826DDD4", // objective_guid
	"taskId": "BCDBBD33-BD6A-6168-63CE-4E55620D3CB4", // elitetask_guid
    "operateTaskGuid": "BCDBBD33-BD6A-6168-63CE-4E55620D3CB4", // 需要处理的elitetask的guid（可选）
	"toGrp": "", // 退回到的组（可选）
	"toBy": "", // 退回到的人（可选）
	"toRole": "", // 退回到的角色（可选）
    "comments": "xxxxx", // 备注信息
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



### 工单撤单

撤单的操作：根据oId删除所有现有task，然后假装做了个save操作（新生成一个task）

url: http://xxxxx/ngs/wo/revoke

入参：

```json
{
	"oId": "C37F1819-8963-1CD4-E9C2-9DA64826DDD4", // objective_guid
	"taskId": "BCDBBD33-BD6A-6168-63CE-4E55620D3CB4", // elitetask_guid
    "comments": "xxxxx", // 备注信息
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

