# spring boot actuator

spring boot 的另一个子模块spring boot actuator提供了能够应用到生产环境的一些特性。通过http端点(endpoint)、JMX或者remote shell(SSH或telnet)的方式提供管理和监控应用。
监控一些健康状况、metrics等。

## 启用production-ready特性
最简单的方式就是增加starter-pom,如果使用dependency-management的话不需要声明版本号
如maven
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```
相应的,gradle中
```
dependencies {
    compile("org.springframework.boot:spring-boot-starter-actuator")
}
```

## Endpoints
endpoints表示的是http url端点，像/health这种。
通过这些url我们可以快速的查看系统的运行状态，而不需要登陆机器使用繁琐的命令查看。
Spring boot actuator提供扩展，可以自定义更多的Endpoint。
ID唯一的标识一个endpoint，默认情况下，id就是这个endpoint的url mapping

| ID | Description描述  |  Sensitive Default |
|---|---|---|
| actuator |   |   |
| beans |  列出应用中的bean列表 |  true |
| docs  |   |   |
| dump  | 进行线程的dump  | true  |
| env  |  展示spring的`ConfigurableEnvironment`的属性 |  true |
| health  | 展示系统是否运行正常  | false  |
| info  |  展示应用的配置的信息 | false  |
| logfile  | 显示应用的log  | true  |
| metrics  |  显示一些metrics信息，如mem等 | true  |
| mappings  | 展示springmvc的mappings  |  true |
| trade  | 展示http的trace信息，默认最近的几个http请求  |  true |

### 订制Endpoints
可以使用Spring的Properties对这些Endpoint进行设置。进行打开禁用等功能。或者修改sensitive或者id。
例如，在application.properties中设置启用logfile
```
endpoints:
  logfile:
    enabled: true
    sensitive: false
```
NOTE: 通过endpoints + . + name唯一的配置一个endpoint的属性

### actuator mvc endpoint
如果classpath里有Spring HATEOAS包(比较通过spring-boot-starter-hateoa或者使用Spring Data REST)
那么就可以使用/actuator查看有哪些endpoint和它们的url

### CORS支持
[CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)允许进行跨域请求。
默认不支持CORS。Actuator的MVC Endpoint可以通过配置支持。
```
endpoints.cors.allowed-origins=http://example.com
endpoints.cors.allowed-methods=GET,POST
```

### 增加定制的Endpoint
通过增加一个`Endpoint`类型的@Bean,就可以expose到JMX和HTTP。

### Health
Health信息可以用来件查看应用运行状态。常用来通知线上应用down了。来可以检查磁盘状态等等。
可以加上集成HealthIndicator的Bean定制自己的HealthIndicator
### Auto-Configured的HealthIndicators
| Name | Description描述  |
|---|---|
| DiskSpaceHealthIndicator | 检查磁盘空间不足  |
| CassandraHealthIndicator |  cassandra数据库 |
| DataSourceHealthIndicator | 检查DataSource的Connection是否能够获取  |
| DiskSpaceHealthIndicator |   |
| ElasticSearchHealthIndicator |  检查ElasticSearch集群是否正常 |
| JmsHealthIndicator | 检查jms broker是否正常  |
| MailHealthIndicator | 检查mail服务器是否正常  |
| MongoHealthIndicator | 检查mongo数据库是否正常  |
| RabbitHealthIndicator | 检查rabbit服务器  |
| RedisHealthIndicator |  检查redis服务器 |
| SolrHealthIndicator | 检查solr  |

## 自定义HealthIndicator
实现HealthIndicator的Bean即可

## 定制info信息
通过在Spring properties中添加info.*的属性。可以在info endpoint中查看。
例如在application.yml中添加
```
info:
  name: liuzhengyang
  birth: 1991-12-18
  height: 174
  weight: 76kg
```
### 在构建时expand info属性
可以通过maven或gradle在build时提供信息而不是硬编码到文件中
#### maven
通过`@..@`placeholder
```
info.build.name=@project.name@
info.build.description=@project.description@
info.build.version=@project.version@
```
#### gradle
通过配置java plugin的`processResources` task
```
processResources {
    expand(project.properties)
}
```
```
info.build.name=${name}
info.build.description=${description}
info.build.version=${version}
```
