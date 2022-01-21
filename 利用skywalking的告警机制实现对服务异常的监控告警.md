## 利用skywalking的告警机制实现对服务异常的监控告警

skywalking提供了一套告警机制，可以通过配置自己的oal（Observability Analysis Language）来收集需要的数据，并通过告警规则触发自定义的接口，最终实现通过邮件或者短信等方式通知用户。

### 网络要求：

1. 每个微服务需要可以通过http方式连上skywalking的oap服务，示例（）

   ```
   -Dskywalking.agent.service_name=ngs -Dskywalking.collector.backend_service=192.168.2.80:11800
   ```

2. skywalking服务需要可以连通配置的webhook的接口地址



### skywalking模式的好处

1. skywalking使用的javaagent的方式接入服务，对应用代码无侵入性。
2. oal可以配置针对各种级别的数据收集，有服务，实例，接口，数据库等等

3. 丰富的告警规则配置



### 配置示例

1. 配置自定义的oal

   针对需要监控的endpoint，配置相关的oal

   config/oal/*.oal中增加配置

   ```oal
   // Calculate the sum of response code in [404, 500, 503], for each service.
   endpoint_abnormal = from(Endpoint.*).filter(responseCode in [404, 500, 503]).count()
   ```

2. 配置告警规则

   config/alarm-setting.yml中添加配置

   ```yml
   endpoint_abnormal_rule:
       metrics-name: endpoint_abnormal
       op: ">"
       threshold: 1
       period: 10
       count: 3
       silence-period: 5
       message: The request number of entity {name} non-200 status is more than expected.
   ```

3. 配置webhook，触发告警，在自定义的接口中实现具体的告警通知

   config/alarm-setting.yml中添加配置

   ```yml
   webhooks:
    - http://127.0.0.1/notify/
   ```