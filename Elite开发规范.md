## Elite开发规范

### 一. 整体开发技术栈

1. JDK1.8
2. Mybatis 数据持久化
3. Redis 3.2或以上
4. MySQL 5.7或以上
5. Elasticsearch 6.x或7.x
6. Vue（建议）2.6或以上
7. Gitlab + Maven + Jenkins



### 二. 微服务技术栈与版本

1. Spring Cloud版本：Hoxton.SR8
2. Spring Cloud Alibaba版本：2.2.3.RELEASE
3. Alibaba Nacos版本：1.3.2
4. Alibaba Sentinel版本：1.8.0
5. Spring Data版本：Moore-SR8



### 三. Redis使用与规范

1. 版本3.2.12或以上

2. key名设计：

   以业务名+表名为前缀，用冒号分隔，例如：ngs:shorten:1

   保持简洁性，key不要过长，不要包含特殊字符（空格、换行单双引号等）

3. value设计：

   redis仅作为缓存使用，不可用于持久化数据，因此所有的value尽量带上expire过期时间，

   无过期时间的数据需要程序处理何时清

4. 命令使用限制：

   禁用的命令：keys, flushall, flushdb等



### 四. 数据库操作规范

1. 尽可能使用预编译SQL

2. xml配置中参数使用：#{}，#param# ，尽量不使用${} 此种方式容易出现SQL注入

3. 如果需要字符串拼接，需要使用服务端拼接，不可直接拼接客户端传递上来的字符串

   

### 五. 日志规范

1. 使用slf4j+logback

2. 文件日志输出时候，使用RollingFileAppender，并且配置日志归档与rolling策略，避免日志文件过多占用服务器磁盘

   具体配置建议如下，注意pattern中需要输出thread的信息

   ```xml
   <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
       <file>${log.folder}/elite.fs.log</file>
       <encoder>
           <Pattern>
               %d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{50} - %msg%n
           </Pattern>
       </encoder>
       <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
           <fileNamePattern>${log.folder}/archived/ngs_logback-%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
           <maxFileSize>100MB</maxFileSize>
           <maxHistory>30</maxHistory>
           <totalSizeCap>2GB</totalSizeCap>
       </rollingPolicy>
   </appender>
   ```

3. 生产环境禁止console的日志输出

4. 日志中需要屏蔽敏感信息，例如用户的身份证号、手机号等



### 六. 接口规范

1. 使用http post接口，统一使用utf-8字符集

2. 参数可以是payload形式（Content-Type: application/json），也可以是表单形式（Content-Type: application/x-www-form-urlencoded）

3. 返回response使用json格式，例如：

   ```json
   {
       code: 1, // 返回的成功或者失败代码
       message: "", // 返回的错误消息内容
       value: { // 具体返回的数据
           xxxxx
       }
   }
   ```

4. 建议使用参数签名来提供接口安全性，http的请求头上带上timestamp（当前时间戳）和sign，sign的值为请求的入参（如果是payload则去除换行符，如果是form则按字典排序每个参数字符串拼接）+ “固定约定字符” + timestamp后的MD5值。



### 七. 代码规范

1. 代码中的命名均不能以下划线或美元符号开始，也不能以下划线或美元符号结束

   代码中的命名严禁使用拼音与英文混合的方式，更不允许直接使用中文的方式

2. 类名使用UpperCamelCase风格，必须遵从驼峰形式

   方法名、参数名、成员变量、局部变量都统一使用lowerCamelCase风格，必须遵从驼峰形式

3. 常量命名全部大写，单词间用下划线隔开

4. 包名统一使用小写，点分隔符之间有且仅有一个自然语义的英语单词

5. 源代码的字符集统一使用UTF-8

6. 采用4个空格缩进，禁止使用tab字符。

   说明：如果使用tab缩进，必须设置1个tab为4个空格。IDEA设置tab为4个空格时，请勿勾选Use tab character；而在eclipse中，必须勾选insert spaces for tabs。



### 八. Elasticsearch规范

1. 从ES7开始只支持一个type，因此使用时候注意不要使用多type，保持只有一个默认的_doc的type，这样可以兼容ES6和ES7

