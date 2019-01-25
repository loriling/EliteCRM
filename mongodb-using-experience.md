# MongoDB 使用经验 #

11/24/2016 11:08:12 AM

## 一. 关于TTL索引 ##
可以通过创建TTL（time to live）索引来实现自动删除过期数据，索引项必须是日期类型：
[https://docs.mongodb.com/v3.2/core/index-ttl/](https://docs.mongodb.com/v3.2/core/index-ttl/)

	db.logs.createIndex({"date":1}, {expireAfterSeconds: 60});

但是发现本机测试一直没有效果，找了下原因，发现启动mongo客户端时候，有一下警告，以前用过集群，后来又不用了，造成TTL collection monitor没有正确的启动

	Tue Jul  9 17:18:46.076 [initandlisten] 
	Tue Jul  9 17:18:46.076 [initandlisten] ** WARNING: mongod started without --replSet yet 1 documents are present in local.system.replset
	Tue Jul  9 17:18:46.077 [initandlisten] **          Restart with --replSet unless you are doing maintenance and no other clients are connected.
	Tue Jul  9 17:18:46.077 [initandlisten] **          The TTL collection monitor will not start because of this.
	Tue Jul  9 17:18:46.077 [initandlisten] **          For more info see http://dochub.mongodb.org/core/ttlcollections
	Tue Jul  9 17:18:46.077 [initandlisten] 
	
通过删除local这个数据库然后重启mongo服务，来启动TTL collection monitor

	use local
	db.dropDatabase();

相关命令

	//查询logs集合，美观的打印出结果
	db.logs.find().pretty();

	//列出logs集合上的索引
	db.logs.showIndexes();

	//往test集合插入数据
	db.test.insert({"expireAt": new Date(), "name": "test1"});

	//根据条件删除记录
	db.logs.delete({"date":{"$lt": ISODate("2016-11-20T18:54:28Z")}})

	//查看ttl进程状态，删除了多少数据，执行了几次
	db.serverStatus().metrics.ttl;
	//{ "deletedDocuments" : NumberLong(5), "passes" : NumberLong(32) }

	//执行管理者命令，获取ttlMonitorEnabled参数的值
	db.adminCommand({getParameter:1, ttlMonitorEnabled: 1});
	//{ "ttlMonitorEnabled" : true, "ok" : 1 }

	//开启索引相关日志，可以输出TTL相关信息
	db.setLogLevel(1, "index");
	//2016-11-24T10:44:01.032+0800 D INDEX    [TTLMonitor] TTL -- ns: elite.logs key: { date: 1.0 }
	//2016-11-24T10:44:01.032+0800 D INDEX    [TTLMonitor] 	TTL deleted: 0
	
也可以在配置文件中写如下配置，来达到同样开启index日志效果 [https://docs.mongodb.com/v3.2/reference/log-messages/#log-messages-configure-verbosity](https://docs.mongodb.com/v3.2/reference/log-messages/#log-messages-configure-verbosity)

	systemLog:
       component:
          index:
             verbosity: 1

