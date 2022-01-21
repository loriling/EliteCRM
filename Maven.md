# Maven

1. 下载和解压

```bash
wget https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
tar -zxvf  apache-maven-3.6.3-bin.tar.gz 
```

2. 修改环境变量

vi /etc/profile

```bash
export M2_HOME=/apps/maven/apache-maven-3.6.3
export PATH=$PATH:$JAVA_HOME/bin:$M2_HOME/bin
```

source /etc/profile



3. 修改conf/setting.xml

   ```xml
   <!-- 修改本地存储路径 -->
   <localRepository>/apps/maven/repo</localRepository>
   <!-- 配置nexus私服地址 -->
   <mirror>
       <id>nexus-elite</id>
       <mirrorOf>*</mirrorOf>
       <name>Nexus elite</name>
       <url>http://nexus.elitecrm.com/repository/maven-public</url>
   </mirror>
   ```

   

   