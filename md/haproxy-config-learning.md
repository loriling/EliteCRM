# 关于Haproxy一些配置的理解 #

- **关于日志**    
haproxy的日志输出是通过syslogd服务来实现的    
在defaults|frontend|listen|backend节点下    
可以配置：`option httplog `    
这样，就会输出http请求相关的日志到指定的
