# 数据库连接池新选择  --  Druid#

----------
[Druid](https://github.com/alibaba/druid "Druid")是由阿里巴巴提供的另一个数据库连接池的实现，能够提供强大的监控和扩展功能。

通常情况下，我们使用的数据库连接池的实现是commons的dbcp，而DruidDataSource的配置是兼容DBCP的。从DBCP迁移到DruidDataSource，只需要修改数据源的实现类就可以了。

DBCP的数据库连接池的实现是：


> org.apache.commons.dbcp.BasicDataSource

替换为：

> com.alibaba.druid.pool.DruidDataSource

## 对于我们实施人员来说，想用druid，只需要做一下几部操作： ##

**1.下载最新的druid的jar包。**

[druid-1,0,10,jar 下载地址](http://central.maven.org/maven2/com/alibaba/druid/1.0.10/druid-1.0.10.jar)

**2.修改需要使用druid数据库连接池的web应用的数据源配置**

    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close"> 
	    <property name="url" value="${jdbc_url}" />
	    <property name="username" value="${jdbc_user}" />
	    <property name="password" value="${jdbc_password}" />
	
	    <property name="filters" value="stat,commonlogging" /><!-- 用于droid的监控 -->
	
	    <property name="maxActive" value="20" />
	    <property name="initialSize" value="1" />
	    <property name="maxWait" value="60000" />
	    <property name="minIdle" value="1" />
	
	    <property name="timeBetweenEvictionRunsMillis" value="60000" />
	    <property name="minEvictableIdleTimeMillis" value="300000" />
	
	    <property name="validationQuery" value="SELECT 'x'" />
	    <property name="testWhileIdle" value="true" />
	    <property name="testOnBorrow" value="false" />
	    <property name="testOnReturn" value="false" />
	    <property name="poolPreparedStatements" value="true" />
	    <property name="maxPoolPreparedStatementPerConnectionSize" value="20" />
    </bean>

这是一段我们非常熟悉的spring中配置数据库连接池的配置，可以关注到其中最主要就是bean的class改了

**3.配置web应用的web.xml**

    <servlet>
      <servlet-name>DruidStatView</servlet-name>
      <servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
    </servlet>
    <servlet-mapping>
      <servlet-name>DruidStatView</servlet-name>
      <url-pattern>/druid/*</url-pattern>
    </servlet-mapping>

这样就可以通过 http://127.0.0.1:8080/appname/druid/index.html 来访问到druid的监控页面了，类似如下图：

![数据源监控](http://202.105.215.50:8012/EliteKM/userfiles/image/20141119/20141119132712_644.png)

这里可以看到各种我们非常关注的数据，比如：**活跃连接数**，**池中连接数**
，包括可以直接看到**连接池中连接信息**

![连接池中连接信息](http://202.105.215.50:8012/EliteKM/userfiles/image/20141119/20141119133928_358.png)

也可以看到**SQL列表**

![SQL监控](http://202.105.215.50:8012/EliteKM/userfiles/image/20141119/20141119132725_557.png)

*是不是碉堡了~~~*

**4.（可选）log4j.properties中添加配置,这样可以让sql相关操作日志输出**

	log4j.logger.druid.sql=debug,stdout,druidRFA
	log4j.appender.druidRFA=org.apache.log4j.RollingFileAppender
	log4j.appender.druidRFA.File=${catalina.home}/logs/druid.log
	log4j.appender.druidRFA.MaxFileSize=10000KB
	log4j.appender.druidRFA.MaxBackupIndex=100
	log4j.appender.druidRFA.layout=org.apache.log4j.PatternLayout
	log4j.appender.druidRFA.layout.ConversionPattern=%d [%t] %-5p %c - %m%n

日志中就可以看到类似：
	
	2014-11-19 11:44:18,065 [http-8080-3] DEBUG druid.sql.Connection - {conn-10008} pool-connect
	2014-11-19 11:44:18,065 [http-8080-3] DEBUG druid.sql.Statement - {conn-10008, stmt-20088} created
	2014-11-19 11:44:18,065 [http-8080-3] DEBUG druid.sql.Statement - {conn-10008, stmt-20088, rs-50084} query executed. 0.599564 millis. 
	select distinct(queue_id) from agent_queue where agent_id='SELITE' order by queue_id
	2014-11-19 11:44:18,065 [http-8080-3] DEBUG druid.sql.ResultSet - {conn-10008, stmt-20088, rs-50084} open
	2014-11-19 11:44:18,065 [http-8080-3] DEBUG druid.sql.ResultSet - {conn-10008, stmt-20088, rs-50084} Header: [queue_id]
	2014-11-19 11:44:18,068 [http-8080-3] DEBUG druid.sql.ResultSet - {conn-10008, stmt-20088, rs-50084} Result: [1]
	2014-11-19 11:44:18,070 [http-8080-3] DEBUG druid.sql.ResultSet - {conn-10008, stmt-20088, rs-50084} closed
	2014-11-19 11:44:18,070 [http-8080-5] DEBUG druid.sql.Connection - {conn-10007} pool-connect
	2014-11-19 11:44:18,070 [http-8080-3] DEBUG druid.sql.Statement - {conn-10008, stmt-20088} closed
	2014-11-19 11:44:18,070 [http-8080-3] DEBUG druid.sql.Connection - {conn-10008} pool-recycle

包含了从获取连接到执行结束连接回收的一系列操作的日志信息。


*这些还不够，根本停不下来*

**5.连接泄漏监测**

	<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
	    ... ...
	    <property name="removeAbandoned" value="true" /> <!-- 打开removeAbandoned功能 -->
	    <property name="removeAbandonedTimeout" value="1800" /> <!-- 1800秒，也就是30分钟 -->
	    <property name="logAbandoned" value="true" /> <!-- 关闭abanded连接时输出错误日志 -->
	    ... ...
	</bean>

配置removeAbandoned对性能会有一些影响，建议怀疑存在泄漏之后再打开。在上面的配置中，如果连接超过30分钟未关闭，就会被强行回收，并且日志记录连接申请时的调用堆栈。

具体可以访问：
[https://github.com/alibaba/druid/wiki/%E8%BF%9E%E6%8E%A5%E6%B3%84%E6%BC%8F%E7%9B%91%E6%B5%8B](https://github.com/alibaba/druid/wiki/%E8%BF%9E%E6%8E%A5%E6%B3%84%E6%BC%8F%E7%9B%91%E6%B5%8B "连接泄漏监测")

*是不是觉得连接泄露变得不是那么可怕了~~*

----------
参考：
[https://github.com/alibaba/druid/wiki/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98](https://github.com/alibaba/druid/wiki/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98 "Druid wiki")