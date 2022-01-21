# Jenkins

### 使用yum安装

```shell
#下载依赖
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo;
#导入秘钥
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key;
yum install jenkins
#查找安装路径
rpm -ql jenkins
---------------------------------------
jenkins相关目录释义：
(1)/usr/lib/jenkins/：jenkins安装目录，war包会放在这里。
(2)/etc/sysconfig/jenkins：jenkins配置文件，“端口”，“JENKINS_HOME”等都可以在这里配置。
(3)/var/lib/jenkins/：默认的JENKINS_HOME。
(4)/var/log/jenkins/jenkins.log：jenkins日志文件。
---------------------------------------
# 启动jenkins服务
systemctl start jenkins
# 查看jenkins服务状态
systemctl status jenkins 
# 关闭jenkins服务
systemctl stop jenkins
```

### 配置Jenkins

vi /etc/sysconfig/jenkins

JENKINS_HOME="/apps/jenkins"

JENKINS_USER="root"

### 配置git位置和git的认证

![image-20210129102300244](C:\Users\Lori\AppData\Roaming\Typora\typora-user-images\image-20210129102300244.png)







### 安装其他插件

1. [ Maven Integration plugin](https://plugins.jenkins.io/maven-plugin)

   插件安装完后，配置全局maven位置 

   Global Tool Configuration中

   ![image-20210129102130084](C:\Users\Lori\AppData\Roaming\Typora\typora-user-images\image-20210129102130084.png)

2. [GitLab Plugin](https://plugins.jenkins.io/gitlab-plugin)

   安装完插件后，配置

   注意这里的enable authentication要去掉勾选

   ![image-20210129104104495](C:\Users\Lori\AppData\Roaming\Typora\typora-user-images\image-20210129104104495.png)

   到gitlab的个人配置中去添加一个gitlab的api token

   ![image-20210129103638078](C:\Users\Lori\AppData\Roaming\Typora\typora-user-images\image-20210129103638078.png)

   复制创建完的token，回到jenkins添加credential

   ![image-20210129103736591](C:\Users\Lori\AppData\Roaming\Typora\typora-user-images\image-20210129103736591.png)

   

3. [Gitlab Hook Plugin](https://plugins.jenkins.io/gitlab-hook)

   