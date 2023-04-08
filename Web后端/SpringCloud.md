# 分布式概念
- **集群**：多台机器即可叫集群，不要求协作。几台服务器可以组合在一起成为集群实现同一业务。
- **分布式系统**：多个计算机集合在一起，但对外等同于单一的系统。分布式中的每一个节点都可以做成集群。
- **节点**：集群中的一个服务器。
- **远程调用**：服务之间互相调用。SpringCloud使用HTTP+JSON进行远程调用。
- **服务注册/发现&注册中心**：记录各个服务器状态，确定哪些服务器能正常使用，提供上下线管理，记录“哪个服务在哪个服务器”清单。
- **配置中心**：服务器可以从配置中心中获取配置，从而可以统一管理配置。
- **服务熔断**：若被调用的*服务*失败程度达到一个阈值时，后来的请求都不再调用该服务，标记该服务为不可用（是服务不是服务器！）。此时上层服务会返回默认值。防止请求积压。
- **服务降级**：高峰期资源紧张，则让非核心业务*降级*，即简单处理（直接抛异常等）或不处理。
- **API网关**：抽象了微服务中都需要的*公共功能*，同时提供了客户端负载均衡，服务自动熔断，灰度发布，统一认证，限流流控，日志统计等丰富的功能，帮助解决API管理难题。前端的请求都要走网关过。
# 技术搭配方案
- **SpringCloud Alibaba-Nacos**:注册中心（服务发现/注册）
- **SpringCloud Alibaba-Nacos**:配置中心（动态配置管理）
- **SpringCloud-Ribbon**:负载均衡
- **SpringCloud-Feign**:声明式HTTP客户端(调用远程服务)
- **SpringCloud Alibaba-Sentinel**:服务容错（限流、降级、熔断）
- **SpringCloud-Gateway**:ApI网关（webflux编程模式）
- **SpringCloud-Sleuth**:调用链监控
- **SpringCloud-Alibaba Seata**:原Fescar，即分布式事务解决方案

# 版本对应关系
[版本说明 · alibaba/spring-cloud-alibaba Wiki · GitHub](https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E)
# Nacos（注册/配置中心）
启动Nacos之后，访问`192.168.177.114:8848/nacos`即可进入管理界面。

## 注册中心
```xml
<dependency>  
    <groupId>com.alibaba.cloud</groupId>  
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>  
</dependency>  
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>  
</dependency>
```

在`application`中对nacos服务器和注册名称进行配置即可完成注册：

```yml
spring:
  cloud:  
    nacos:  
      discovery:  
        server-addr: 192.168.177.114:8848  
        
  application:  
    name: gulimall-member
```
## 配置中心
```xml
<dependency>  
    <groupId>com.alibaba.cloud</groupId>  
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>  
</dependency>  
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-bootstrap</artifactId>  
</dependency>
```
### 基本设置
添加配置文件`bootstrap.properties`，配置服务名和nacos地址。地址设置相当于注册中心的`discovery`换成`config`。

```properties
spring.application.name=gulimall-coupon  
  
spring.cloud.nacos.config.server-addr=192.168.177.114:8848
```

SpringCloud2.4以上的版本都需要导入`spring-cloud-starter-bootstrap`依赖才能启用对`bootstrap.properties`的扫描（它的优先级太高了，无法用`application`通知spring去扫描它，只能通过安装依赖解决）。

而在nacos网页上我们可以创建配置，默认会读取的配置名是`应用名.properties`，选好文件格式后编辑配置文件。

此时，在nacos网页上发布的配置就能被程序用`@Value`直接获取到，和获取本地配置的方式一模一样。

加上`@RefrashScope`来使获取对应配置前进行刷新以应用配置的更改。

配置中心的配置优先级更高。

### 命名空间
作用：配置隔离。

有两种情景：
- 开发、测试、生产三种环境下不同的配置文件。
- 每个微服务创建自己的命名空间，实现微服务之间的配置隔离。

切换命名空间（`dev`是命名空间id，默认是类似于uuid的字符串）：

```properties
spring.cloud.nacos.config.namespace=dev
```

### 配置集
一些配置的集合就叫配置集，配置集有一个配置集ID，相当于配置文件名。此处配置集ID就是**Data ID**。

### 配置分组
像命名空间一样筛选，比如双十一的时候切换到1111的group，或者也能用于切换开发、测试、生产三种环境。

```properties
spring.cloud.nacos.config.group=1111
```

### 加载多配置集
`bootstrap.yml`：

```yml
spring:  
  application:  
    name: gulimall-member  
  
  cloud:  
    nacos:  
      config:  
        server-addr: 192.168.177.114:8848  
        namespace: gulimall-member  
        group: dev  
        extension-configs:  
          - data-id: datasource.yml  
            group: dev  
            refresh: true  
  
          - data-id: mybatis.yml  
            group: dev  
            refresh: true  
  
          - data-id: other.yml  
            group: dev  
            refresh: true
```

写在`extension-configs`外的都相当于默认值。`bootstrap.properties`无论如何都会被读取。
# Feign（远程调用）
在member模块中`feign`包下创建`CouponFeignService`，通过`FeignClient`设置目标注册名称，通过`RequestMapping`设置目标路径名（也即通过url形式调用）。

```java
@FeignClient("gulimall-coupon")  
public interface CouponFeignService {  
  
    @RequestMapping("/coupon/coupon/member/list")  
    public R membercoupons();  
}
```

然后在Application上开启功能：
```java
@EnableFeignClients("cn.ayana.gulimall.member.feign")
```

这样，member模块里调用`couponFeignService.membercoupons()`即可获取coupon模块内`/coupon/coupon/member/list`对应的信息。

## 注意事项
返回类型一定要加无参构造函数，否则会一直返回null。
# GateWay（网关）
```xml
<dependency>  
   <groupId>org.springframework.cloud</groupId>  
   <artifactId>spring-cloud-starter-gateway</artifactId>  
</dependency>
```
## 原理
请求到达网关后，由断言（predicate）来判断其是否符合某个路由规则，若符合则根据这个规则将其路由送达指定地方，途径很多过滤器（filter）。

![](assets/Pasted%20image%2020230220131950.png)

## 使用
各种拦截器和断言都可去官网查找：[Spring Cloud Gateway](https://docs.spring.io/spring-cloud-gateway/docs/3.1.4/reference/html/)

自带网关不是Tomcat而是Netty，性能很高。

下面的配置意为：如果请求里的url参数满足“包含`baidu`”，则定向到`https://www.baidu.com`的对应路径。

```yml
spring:  
  cloud:  
    gateway:  
      routes:  
        - id: baidu-route  
          uri: https://www.baidu.com  
          predicates:  
            - Query=url,baidu  
  
        - id: qq-route  
          uri: https://www.qq.com  
          predicates:  
            - Query=url,qq
```

此时，访问`http://127.0.0.1:88/xxx?url=baidu`即可访问`https://www.baidu.com/xxx`。

使用Path Route断言即可通过路径来选择路由目标。

使用RewritePath过滤器来改变请求的路径（只是改变内容）。

uri中，使用`lb://`就是在注册中心中找目标。

```yml
spring:  
  cloud:  
    gateway:  
      routes:  
        - id: product-route  
          uri: lb://gulimall-product  
          predicates:  
            - Path=/api/product/**  
          filters:  
            - RewritePath=/api/(?<segment>.*),/$\{segment}  

        - id: admin-route  
          uri: lb://renren-fast  
          predicates:  
            - Path=/api/**  
          filters:  
            - RewritePath=/api/(?<segment>.*),/renren-fast/$\{segment}
```

## 跨域
Spring提供了CorsWebFilter来解决跨域问题。编写下面的配置类即可。

注意：导类包时尽可能选择`reactive`！

```java
import org.springframework.web.cors.reactive.CorsWebFilter;  
import org.springframework.web.cors.reactive.UrlBasedCorsConfigurationSource;  
  
@Configuration  
public class GulimallCorsConfiguration {  
  
    @Bean  
    public CorsWebFilter corsFilter() {  
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();  
  
        CorsConfiguration corsConfiguration=new CorsConfiguration();  
  
        corsConfiguration.addAllowedHeader("*");  
        corsConfiguration.addAllowedMethod("*");  
        corsConfiguration.addAllowedOriginPattern("*");  
        corsConfiguration.setAllowCredentials(true);//cookie  
  
        source.registerCorsConfiguration("/**",corsConfiguration);  
  
        return new CorsWebFilter(source);  
    }  
}
```

如果`setAllowCredentials`为true，则`config.addAllowedOrigin("*")`的参数就不能是`"*"`，此时需要使用`addAllowedOriginPattern("*")`。
































