# 正确使用Tomcat Manager的姿势
2/17/2017 11:29:38 AM 
##一. 安全问题
Tomcat的Manager应用，作为一个tomcat自带的默认应用，随着安装自动会在webapps目录下。通过这个应用可以部署，重新部署，删除应用等操作。实现了不重启服务，就可以热部署应用。而生存中通常会因为其安全性不够，如果被猜到密码，整个服务器就会被黑，所以都是删除此应用的。

**其实，tomcat的manager，自带了一种valve，在manager应用的META-INF目录下，有一个context.xml**

    <Context antiResourceLocking="false" privileged="true" >
	  <!--
	    Remove the comment markers from around the Valve below to limit access to
	    the manager application to clients connecting from localhost
	  -->
	  <!--
	  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
	         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
	  -->
	</Context>

把其中的**RemoteAddrValve**注释去掉，就可以了。

但是，有时候，我们tomcat在haproxy这种反向代理服务器之后，这样，看到的remoteAddr都是反向代理服务器地址，就那公司服务器为例，haproxy和tomcat部署在一台机器上，这样remoteAddr看到的都是127.0.0.1，这样上面的配置允许的刚好是127开头的地址，所以还是没有起到屏蔽外网访问的问题。

修改一下配置：

	<Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="192\.168\.\d+\.\d+" denyStatus="404"/>

因为公司内网可以直接访问到服务器内网地址，这样在公司内网，可以不通过haproxy，直接访问服务器上tomcat的端口，这时候，remoteAddr看到的都是内网192.168开头的地址，这样刚好允许了这类地址的访问。