#Tomcat中 JNDI数据源配置的一大误区#


**平时我们配置的JNDI通常是这样的：**    
tomcat/conf/content.xml中    
>     <Resource name="jdbc/sharedDatasource" auth="Container" 
>         factory="com.elite.aps.dbservice.http.APSDataSourceFactory" type="javax.sql.DataSource" maxActive="10" 
>         maxIdle="10" minIdle="10" initialSize="10" encryptkey="rshMlJiUs+A=" dbtype="mssql" logsessionid="true" 
>         username="sa" password="letmein" driverClassName="net.sourceforge.jtds.jdbc.Driver" 
>         validationQuery="select 1" timeBetweenEvictionRunsMillis="60000" testWhileIdle="false" 
>         url="jdbc:jtds:sqlserver://127.0.0.1:1433/elitedb_en"/>  

**然后，我们就兴高采烈的去各个应用中获取这个jdbc/sharedDatasource了**    

**但是！**

**其实我们想错了，虽然名字起的是sharedDatasource，但是他并不是shareable的。。。。**

每个调用次数据源的应用，都会创建一个单独的数据库连接池，也就是说，通常情况下，我们认为APS和webserverservlet共享了数据源，而其实他们还是各自使用着自己的数据源，当tomcat启动后，由于上面initialSize配置为10，所以就会为APS何webserverservlet分别开启10个连接


----------


**要真正实现数据源的共享，需要把Resource配置，放到tomcat的server.xml中，**     
>     <GlobalNamingResources>
>         <Resource name="jdbc/sharedDatasource" auth="Container" 
>              ......./>
>     </GlobalNamingResources>

**然后在context.xml中，通过ResourceLink标签来引入GlobalNamingResources中定义的数据源**
>     <ResourceLink name="jdbc/sharedDatasource" type="javax.sql.DataSource" global="jdbc/sharedDatasource"/>


这样，应用中获取到的数据源才会是同一个dbpool实例。

*PS.原来那样，分开的使用不同数据源的方式也不一定不好，出现问题时候，能更容易排查出是那个应用造成的问题。*