# MongoDB 集群配置 #
1/9/2017 1:36:49 PM 

## 副本集架构（Replica Set） ##

为了防止单点故障就需要引副本（Replication），当发生硬件故障或者其它原因造成的宕机时，可以使用副本进行恢复，最好能够自动的故障转移（failover）。有时引入副本是为了读写分离，将读的请求分流到副本上，减轻主（Primary）的读压力。而Mongodb的Replica Set都能满足这些要求。

Replica Set的一堆mongod的实例集合，它们有着同样的数据内容。包含三类角色：


- 主节点（Primary）

> 接收所有的写请求，然后把修改同步到所有Secondary。一个Replica Set只能有一个Primary节点，当Primar挂掉后，其他Secondary或者Arbiter节点会重新选举出来一个主节点。默认读请求也是发到Primary节点处理的，需要转发到Secondary需要客户端修改一下连接配置。

- 副本节点（Secondary）



> 与主节点保持同样的数据集。当主节点挂掉的时候，参与选主。


- 仲裁者（Arbiter）

> 不保有数据，不参与选主，只进行选主投票。使用Arbiter可以减轻数据存储的硬件需求，Arbiter跑起来几乎没什么大的硬件资源需求，但重要的一点是，在生产环境下它和其他数据节点不要部署在同一台机器上。
> 
> 注意，一个自动failover的Replica Set节点数必须为奇数，目的是选主投票的时候要有一个大多数才能进行选主决策。


## 示例: ##
**1.编写mongod.conf配置文件，作为replicaSet中第一个节点（Primary）**

    systemLog:
       destination: file
       path: "D:/MongoDB/logs/mongod.log"
       logAppend: true
    storage:
       dbPath: "D:/MongoDB/data/db"
       journal:
          enabled: true
    net:
       port: 27017
    setParameter:
       enableLocalhostAuthBypass: true
    security:
       authorization: disabled
    replication:
       oplogSizeMB: 1024
       replSetName: rs0
    #linux下需要下面的配置，windows下需要删除
    processManagement:
       fork: true

**2.编写mongod2.conf配置文件，作为replicaSet中的第二个节点（Secondary）**

    systemLog:
       destination: file
       path: "D:/MongoDB/logs/mongod2.log"
       logAppend: true
    storage:
       dbPath: "D:/MongoDB/data/db2"
       journal:
           enabled: true
    net:
       port: 27018
    setParameter:
       enableLocalhostAuthBypass: true
    security:
       authorization: disabled
    replication:
       oplogSizeMB: 1024
       replSetName: rs0
    #linux下需要下面的配置，windows下需要删除
    processManagement:
       fork: true

**3.编写mongod-arb.conf配置文件，作为replicaSet中的仲裁者节点（Arbiter）**

    systemLog:
       destination: file
       path: "D:/MongoDB/logs/mongod-arb.log"
       logAppend: true
    storage:
       dbPath: "D:/MongoDB/data/arb"
       journal:
          enabled: true
       mmapv1:
          smallFiles: true
    net:
       port: 27019
    setParameter:
       enableLocalhostAuthBypass: true
    security:
       authorization: disabled
    replication:
       oplogSizeMB: 1024
       replSetName: rs0
    #linux下需要下面的配置，windows下需要删除
    processManagement:
       fork: true

**4.启动三个节点：**

    mongod --config mongod.conf
    mongod --config mongod2.conf
    mongod --config mongod-arb.conf

### **注意: 配置文件中dbPath对应的目录需要自己事先创建，3个配置文件中的replSetName需要一致** ###

**5.进入到第一个节点：**

    mongo --port 27017
配置replica-set

	rs.initiate(
	  {
	    _id : 'rs0',
	    members: [
	      { _id : 0, host : "127.0.0.1:27017" }
	    ]
	  }
	);//初始化replica，添加主member
	rs.add("127.0.0.1:27018");//添加从member
	rs.addArb("127.0.0.1:27019");//添加仲裁者

看到返回{"ok":1} 就表示配置成功了

**6.修改web应用中的mongo连接配置，以webchat为例：**

	<!-- mongodb replicaSet配置，需要把主和从的ip和端口都配置上 -->
	<util:list id="replicaSet" value-type="com.mongodb.ServerAddress">
	  <bean class="com.mongodb.ServerAddress">
	  	<constructor-arg name="host" value="127.0.0.1"></constructor-arg>
	  	<constructor-arg name="port" value="27017"></constructor-arg>
	  </bean>
	  <bean class="com.mongodb.ServerAddress">
	  	<constructor-arg name="host" value="127.0.0.1"></constructor-arg>
	  	<constructor-arg name="port" value="27018"></constructor-arg>
	  </bean>
  	</util:list>
	<!-- mongo client的配置，应用了之前配置的replicaSet -->
  	<mongo:mongo-client id="mongo" replica-set="#{replicaSet}">
      <mongo:client-options 
          connections-per-host="10"
          connect-timeout="30000"
          max-wait-time="30000" />
  	</mongo:mongo-client>
	
  	<mongo:db-factory dbname="webchat" mongo-ref="mongo"/>

至此集群配置完毕，手动关闭Primary节点，可以看到Secondary自动变成了Primary，应用服务照旧正常运行。

**7.配置带auth的mongo集群**

参考：[https://docs.mongodb.com/manual/tutorial/enforce-keyfile-access-control-in-existing-replica-set/](https://docs.mongodb.com/manual/tutorial/enforce-keyfile-access-control-in-existing-replica-set/)

1.在之前的配置环境下，使用mongo客户端连上mongodb的primary节点，执行脚本：

	//创建出一个admin的user
	use admin;
	db.createUser({
		user: "elite",
		pwd: "letmein",
		roles: [ "userAdminAnyDatabase",
	           "dbAdminAnyDatabase",
	           "readWriteAnyDatabase"
  	});
  	//创建webchat相关的user
	use webchat;
	db.createUser({
		user: 'elite',
		pwd: 'letmein',
		roles: [
			{
				role: 'readWrite', db: 'webchat'
			},
			{
				role: 'dbAdmin', db: 'webchat'
			},
			{
				role: 'userAdmin', db: 'webchat'
			},
		]
	});

	//创建一个可以执行脚本的role，为了在webchat中可以执行eval去获取mongodb的当前时间
	use admin;
	db.createRole({ 
	    "role" : "executeFunctions", 
	    "privileges" : [ 
	        { 
	            "resource" : { 
	                "anyResource" : true 
	            }, 
	            "actions" : [ 
	                "anyAction" 
	            ] 
	        } 
	    ], 
	    "roles" : [ ] 
	});
	//给webchat库的elite用户添加这个role
	use webchat;
	db.grantRolesToUser("elite", [{role: "executeFunctions", db: "admin"}]);


​	
2.创建keyfile
> 随便用什么方式生成一个keyfile文件，文件内容必须是6-1024个字符。<br>
> 比如创建一个mongodb-key.keyfile文件，用nodepad++打开，随便写一段字符，比如是：letmein0308，保存。
>
> 如果在linux环境中，mongodb-key.keyfile的文件权限必须是 **600**

3.修改conf文件

> 修改每个实例的mongod.conf文件，在security节点下，增加配置  keyFile： mongodb-key.keyfile。<br>
> 注意这里配置的是keyfile的路径，如果不在当前目录下，就需要写上绝对路径。<br>
> 每个节点的配置文件中，都要加上这句，并且要使用相同的keyfile<br>
> 原本security节点下的authorization: disabled这句配置，当配置了keyFile: mongodb-key.keyfile，就不会起作用了。

	systemLog:
	   destination: file
	   path: "D:/MongoDB/logs/mongod.log"
	   logAppend: true
	storage:
	   dbPath: "D:/MongoDB/data/db"
	   journal:
	      enabled: true
	net:
	   bindIp: 127.0.0.1
	   port: 27017
	setParameter:
	   enableLocalhostAuthBypass: false
	security:
	   authorization: disabled
	   keyFile: mongodb-key.keyfile
	replication:
	   oplogSizeMB: 512
	   replSetName: rs0

4.重启所有的mongodb实例

5.修改客户端连接配置，以webchat举例，服务配置中，applicationContext.xml中添加配置：

	<!-- 创建一个com.mongodb.MongoCredential示例，配置用户名密码和验证的数据库名 -->
	<bean id="mongoCredentialID" class="com.mongodb.MongoCredential">
		<constructor-arg name="mechanism" value = "#{T(com.mongodb.AuthenticationMechanism).SCRAM_SHA_1}" />
		<constructor-arg type="java.lang.String" name="userName" value="elite" />
		<constructor-arg type="java.lang.String" name="source" value="webchat" />
		<constructor-arg type="char[]" name="password" value="letmein" />
	</bean>
	<!-- mongo-client 添加属性credentials，应用上面配置的mongoCredentialID这个bean -->
	<mongo:mongo-client id="mongo" replica-set="#{replicaSet}" credentials="#{mongoCredentialID}">
	    <mongo:client-options 
	        connections-per-host="10"
	        connect-timeout="30000"
	        max-wait-time="30000" />
	</mongo:mongo-client>


## 全文参考： ##
[http://www.jianshu.com/p/2825a66d6aed](http://www.jianshu.com/p/2825a66d6aed) <br>
[https://docs.mongodb.com/manual/reference/](https://docs.mongodb.com/manual/reference/)<br>
[https://www.claudiokuenzler.com/blog/555/allow-mongodb-user-execute-command-eval-mongodb-3.x#.WHMNRNV96Uk](https://www.claudiokuenzler.com/blog/555/allow-mongodb-user-execute-command-eval-mongodb-3.x#.WHMNRNV96Uk)
