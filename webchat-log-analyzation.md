# webchat 日志

webchat和其他服务一样，后台采用logback作为日志输出的类库

在微服务模式下，优先使用classpath下的**logback-spring.xml**

在传统模式下，使用的是classpath下的**logback.xml**

其中关键信息有3个：

1. **所有的appender标签，都是日志输出类型的定义，而不是真正的日志输出，在日志输出类型定义中，可以修改日志输出的名字，路径，轮训规则等**
2. **所有的logger标签，才是真正的日志输出，name不需要修改，只需要考虑日志level是不是要改**
3. **如果想要开启全日志，只需要在最下面的root标签上，修改level为debug，并启用FILE的输出器**

示例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- 日志输出路径 -->
	<property name="LOG_HOME" value="${catalina.home}/logs"/>

    <!-- 日志输出类型, 名字叫STDOUT, 类型是ConsoleAppender，表示在控制台输出，生成环境中应该都关掉控制台输出 -->
	<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
		<layout class="ch.qos.logback.classic.PatternLayout">
			<Pattern>
				%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5p %logger{32} - %msg%n
			</Pattern>
		</layout>
	</appender>

    <!-- 日志输出类型, 名字叫FILE, 类型是RollingFileAppender，表示输出到文件 -->
	<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>${LOG_HOME}/webchat.log</file>
		<encoder>
			<Pattern>
				%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5p %logger{32} - %msg%n
			</Pattern>
		</encoder>
		<rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
			<fileNamePattern>${LOG_HOME}/archived/webchat-%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
			<!-- each file should be at most 100MB, keep 30 days worth of history, but at most 2GB -->
			<maxFileSize>100MB</maxFileSize>
			<maxHistory>30</maxHistory>
			<totalSizeCap>2GB</totalSizeCap>
		</rollingPolicy>
	</appender>

	<appender name="FILE_MONITOR" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>${LOG_HOME}/webchat-monitor.log</file>
		<encoder>
			<Pattern>
				%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5p %logger{32} - %msg%n
			</Pattern>
		</encoder>
		<rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
			<fileNamePattern>${LOG_HOME}/archived/webchat-monitor-%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
			<!-- each file should be at most 100MB, keep 30 days worth of history, but at most 2GB -->
			<maxFileSize>100MB</maxFileSize>
			<maxHistory>30</maxHistory>
			<totalSizeCap>2GB</totalSizeCap>
		</rollingPolicy>
	</appender>

	<appender name="FILE_JRA" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>${LOG_HOME}/webchat-jra.log</file>
		<encoder>
			<Pattern>
				%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5p %logger{32} - %msg%n
			</Pattern>
		</encoder>
		<rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
			<fileNamePattern>${LOG_HOME}/archived/webchat-jra-%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
			<!-- each file should be at most 100MB, keep 30 days worth of history, but at most 2GB -->
			<maxFileSize>100MB</maxFileSize>
			<maxHistory>30</maxHistory>
			<totalSizeCap>2GB</totalSizeCap>
		</rollingPolicy>
	</appender>

	<appender name="FILE_SQL" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>${LOG_HOME}/webchat-sql.log</file>
		<encoder>
			<Pattern>
				%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5p %logger{32} - %msg%n
			</Pattern>
		</encoder>
		<rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
			<fileNamePattern>${LOG_HOME}/archived/webchat-sql-%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
			<!-- each file should be at most 100MB, keep 30 days worth of history, but at most 2GB -->
			<maxFileSize>100MB</maxFileSize>
			<maxHistory>30</maxHistory>
			<totalSizeCap>2GB</totalSizeCap>
		</rollingPolicy>
	</appender>

    <!-- 日志, 名字叫做com.elitecrm 级别是debug， 使用的输出方式是上面定义的FILE -->
	<logger name="com.elitecrm" level="debug" additivity="false">
		<appender-ref ref="FILE" />
	</logger>

	<logger name="Monitor" level="debug" additivity="false">
		<appender-ref ref="FILE_MONITOR" />
	</logger>

	<logger name="JRA" level="debug" additivity="false">
		<appender-ref ref="FILE_JRA" />
	</logger>

	<logger name="org.springframework.jdbc" level="debug" additivity="false">
<!--		<appender-ref ref="FILE_SQL" />-->
	</logger>

    <!-- 全日志, 生成环境应该关闭，如果出问题，可以临时开启查看 -->
	<root level="warn">
<!--		<appender-ref ref="FILE" />-->
	</root>

</configuration>

```





## webchat日志分类


webchat日志分：

1. webchat.log：webchat的总体日志，包括了各种业务处理，请求路由

2. webchat-jra.log：jra接口日志，量非常大，坐席和客户的send和pull请求，都会在这里记录

3. webchat-montior.log（一般不需要开启）：所有监控相关日志

4. webchat-sql.log（一般不需要开启）：所有sql操作日志

一般情况分析日志都需要用webchat.log和webchat-jra.log配合了一起看



## 查看后台日志基本逻辑

1.  [http-nio-8980-exec-4] 这里的http-nio-8980-exec-4，表示线程名，后台服务是多线程的，所以日志也是多线程重叠输出的，要看某个请求的上下连续日志，就需要根据这个线程名来筛选，同样的线程名，可以表示上下日志是在同一次http请求内的

   但也并不是代表同一个线程名所有日志，都是一个请求的，因为tomcat后台http线程是复用的，当一个线程处理完一次请求后，会回到线程池中，准备处理后续请求。



## webchat日志分析

### 一. 查看请求接起时间

1. 客户发起聊天请求

   ```json
   2022-07-15 17:28:02.824 [http-nio-8980-exec-4] DEBUG JRA - Send request data: {"actionType":"clientRequest","senderId":"3d5bf1db-cc7c-435e-89df-163e3e560242","senderName":"张三","queueId":2,"receiverId":"","from":"","brand":""}
   ```

   从这里可以看到用户张三（3d5bf1db-cc7c-435e-89df-163e3e560242）发了一个聊天请求，请求的队列是2

   ```json
   2022-07-15 17:28:02.943 [http-nio-8980-exec-4] DEBUG c.e.w.l.ProcessUserRequestLogic - New chat request has come in from: 3d5bf1db-cc7c-435e-89df-163e3e560242 and new chat request has been created: ChatRequest [id=2592, fromUserId=3d5bf1db-cc7c-435e-89df-163e3e560242, toUserId=-1, sessionId=-1, workgroupId=1, queueId=2, requestType=1, creationTime=1657877282914, lastRoutingTime=0, status=0, changeType=0, requestMetaData={"company":"Netscape Communications","name":"Netscape Navigator","version":"5.0","mainVersion":"5","minorVersion":"0","os":"Windows NT","ip":"127.0.0.1","location":"Local IP Address","language":"en"}, requestIp=127.0.0.1, isPulledByAgent=false, from=null, preparedToUserId=null, comments=null, serviceAccountId=null, serviceAccountName=null, searchEngine=null, keyword=null, preMessages=[], creationType=0, serverAddr=null, orderInQueue=1, senderType=0, hasSendLogic=false]
   ```

   然后看到这个请求被创建出来，请求id是2592

   可以看到这里线程都是http-nio-8980-exec-4，表示了是在一次客户端请求里输出的

2. 后台路由请求

   路由请求的后台线程名字叫：Chat-RoutingThread

   ```json
   2022-07-15 17:28:03.358 [Chat-RoutingThread] DEBUG c.e.w.logic.routing.RoutingLogic - Start routing workgroup[1] priority[0]: 1
   2022-07-15 17:28:03.360 [Chat-RoutingThread] DEBUG c.e.w.logic.routing.RoutingLogic - Chat request : 2592, queue: 2, from user: 3d5bf1db-cc7c-435e-89df-163e3e560242, to user: null
   2022-07-15 17:28:03.372 [Chat-RoutingThread] DEBUG c.e.w.logic.routing.RoutingLogic - 1 online agents in queue 2
   2022-07-15 17:28:03.373 [Chat-RoutingThread] DEBUG c.e.w.l.r.RandomNextRoutingLogic - User[6FF414] status 1]
   2022-07-15 17:28:03.375 [Chat-RoutingThread] DEBUG c.e.w.l.r.RandomNextRoutingLogic - 6FF414 maxSessionAllowed: 2  activeSessions: 0 []
   2022-07-15 17:28:03.375 [Chat-RoutingThread] DEBUG c.e.w.logic.routing.RoutingLogic - findNextAvailableAgent for chatRequest 2592 queueId 2
   2022-07-15 17:28:03.375 [Chat-RoutingThread] DEBUG c.e.w.l.r.RandomNextRoutingLogic - Left available agents: 1
   2022-07-15 17:28:03.376 [Chat-RoutingThread] DEBUG c.e.w.l.r.RandomNextRoutingLogic - User[6FF414] status 1]
   2022-07-15 17:28:03.378 [Chat-RoutingThread] DEBUG c.e.w.l.r.RandomNextRoutingLogic - 6FF414 maxSessionAllowed: 2  activeSessions: 0 []
   2022-07-15 17:28:03.378 [Chat-RoutingThread] DEBUG c.e.w.logic.routing.RoutingLogic - Routing request from user 3d5bf1db-cc7c-435e-89df-163e3e560242 in queue 2 to agent: 6FF414
   2022-07-15 17:28:03.378 [Chat-RoutingThread] DEBUG c.e.w.logic.routing.RoutingLogic - Number of agents online: 1
   2022-07-15 17:28:03.409 [Chat-RoutingThread] DEBUG c.e.w.logic.routing.RoutingLogic - Placing chat request in waiting state with id: 2592
   2022-07-15 17:28:03.409 [Chat-RoutingThread] DEBUG c.e.w.cache.RequestCacheUtil - [save request] ChatRequest [id=2592, fromUserId=3d5bf1db-cc7c-435e-89df-163e3e560242, toUserId=6FF414, sessionId=-1, workgroupId=1, queueId=2, requestType=1, creationTime=1657877282914, lastRoutingTime=1657877283378, status=0, changeType=1, requestMetaData={"company":"Netscape Communications","name":"Netscape Navigator","version":"5.0","mainVersion":"5","minorVersion":"0","os":"Windows NT","ip":"127.0.0.1","location":"Local IP Address","language":"en"}, requestIp=127.0.0.1, isPulledByAgent=false, from=null, preparedToUserId=null, comments=null, serviceAccountId=null, serviceAccountName=null, searchEngine=null, keyword=null, preMessages=[], creationType=0, serverAddr=null, orderInQueue=1, senderType=0, hasSendLogic=false]
   2022-07-15 17:28:03.411 [Chat-RoutingThread] DEBUG c.e.w.cache.db.BasicDBCache - Put 6FF414[2592] to chatRequestsByReceiver
   2022-07-15 17:28:03.413 [Chat-RoutingThread] DEBUG c.e.w.logic.routing.RoutingLogic - End routing for priority 0 with 1 requests, take time: 55 milliseconds.
   ```

   可以看到后台开始准备分配请求，并找到了一个可以分配的坐席6FF414

   然后就分配给了这个坐席，请求(id=2592)的toUserId会被更新成6FF414

3.  坐席pull请求

   坐席每3秒会问一次服务器，自己有没有收到请求

   ```json
   2022-07-15 17:28:03.487 [http-nio-8980-exec-2] DEBUG JRA - Session [3df91427-2dd7-4d15-8ae9-3995de169221] called: pull with user 6FF414 ip: 127.0.0.1
   2022-07-15 17:28:03.489 [http-nio-8980-exec-2] DEBUG JRA - Pull data: {"senderId":"6FF414","chatRequestIdsFromUser":[],"sessionIds":[],"sessionPerms":[]}
   2022-07-15 17:28:03.492 [http-nio-8980-exec-2] DEBUG c.e.w.dao.cache.ChatRequestCache - setPulledByAgent true from request 2592
   2022-07-15 17:28:03.492 [http-nio-8980-exec-2] DEBUG c.e.w.cache.RequestCacheUtil - [save request] ChatRequest [id=2592, fromUserId=3d5bf1db-cc7c-435e-89df-163e3e560242, toUserId=6FF414, sessionId=-1, workgroupId=1, queueId=2, requestType=1, creationTime=1657877282914, lastRoutingTime=1657877283378, status=0, changeType=2, requestMetaData={"company":"Netscape Communications","name":"Netscape Navigator","version":"5.0","mainVersion":"5","minorVersion":"0","os":"Windows NT","ip":"127.0.0.1","location":"Local IP Address","language":"en"}, requestIp=127.0.0.1, isPulledByAgent=true, from=null, preparedToUserId=null, comments=null, serviceAccountId=null, serviceAccountName=null, searchEngine=null, keyword=null, preMessages=[], creationType=0, serverAddr=null, orderInQueue=1, senderType=0, hasSendLogic=false]
   2022-07-15 17:28:03.494 [http-nio-8980-exec-2] DEBUG c.e.w.l.ProcessUserRequestLogic - Pulled request 2592 for agent 6FF414
   ```

   可以看到6FF414这个坐席，pull到了2592这个请求

4. 坐席接受请求

   坐席先pull到了请求，然后会接受或者拒绝，如果接受，会发出接受请求的http请求

   ```json
   2022-07-15 17:28:03.514 [http-nio-8980-exec-9] DEBUG JRA - Session [3df91427-2dd7-4d15-8ae9-3995de169221] called: send with user 6FF414 ip: 127.0.0.1
   2022-07-15 17:28:03.514 [http-nio-8980-exec-9] DEBUG JRA - Send request data: {"actionType":"respondToRequest","senderId":"6FF414","senderName":"Brewster","chatRequestId":2592,"message":"yes"}
   2022-07-15 17:28:03.514 [http-nio-8980-exec-9] DEBUG c.e.w.l.JSONRequestProcessingLogic - processSendRequest with jsonRequestData: {"actionType":"respondToRequest","senderId":"6FF414","senderName":"Brewster","chatRequestId":2592,"message":"yes"}
   2022-07-15 17:28:03.514 [http-nio-8980-exec-9] DEBUG c.e.w.l.ProcessUserRequestLogic - ChatRequestId: 2592
   2022-07-15 17:28:03.515 [http-nio-8980-exec-9] DEBUG c.e.w.l.ProcessUserRequestLogic - Accept client request 2592
   ```

   可以看到坐席6FF414发出了respondToRequest请求，message是yes，表示接受了这个2592请求

5. 客户收到坐席接受，并开始聊天

   这里会区分，如果是pc网页或者手机h5网页客户端，会存在这步，如果是其他客户端，可能不会有这步

   ```json
   2022-07-15 17:28:05.978 [http-nio-8980-exec-6] DEBUG JRA - Session [dc9e9a17-d399-4757-9a61-5a7323621007] called: pull with user 3d5bf1db-cc7c-435e-89df-163e3e560242 ip: 127.0.0.1
   2022-07-15 17:28:05.979 [http-nio-8980-exec-6] DEBUG JRA - Pull data: {"senderId":"3d5bf1db-cc7c-435e-89df-163e3e560242","chatRequestIdsFromUser":[2592]}
   2022-07-15 17:28:05.983 [http-nio-8980-exec-6] DEBUG c.e.w.l.ProcessUserRequestLogic - Client request was accepted: 2592 by 6FF414 and remove request.
   2022-07-15 17:28:05.983 [http-nio-8980-exec-6] DEBUG c.e.w.dao.cache.ChatRequestCache - Remove request 2592 from cache.
   2022-07-15 17:28:05.984 [http-nio-8980-exec-6] DEBUG c.e.w.cache.db.BasicDBCache - Remove 6FF414[2592] from chatRequestsByReceiver
   2022-07-15 17:28:05.984 [http-nio-8980-exec-6] DEBUG c.e.w.cache.db.BasicDBCache - Remove 2[2592] from chatRequestsByQueue
   2022-07-15 17:28:05.992 [http-nio-8980-exec-6] DEBUG JRA - Pull response: {"result":1,"chatRequestsFromUser":[{"id":2592,"sessionId":2175,"requestType":0,"status":1,"feedbackMessage":"Brewster 接收了您的请求，请输入","user":{"id":"6FF414","name":"Brewster","firstName":"Brewster","type":2}}],"sessions":[],"needChange":false}
   ```

   可以看到，客户pull过来，发现自己的请求已经被坐席接起来了，那就可以开始聊天了

   **至此，一个请求进来到被接起到开始聊天才算完成，总体耗时可以看到：2022-07-15 17:28:02.824 -- 2022-07-15 17:28:05.992 基本耗时3秒**

