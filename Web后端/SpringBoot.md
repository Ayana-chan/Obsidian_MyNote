# 杂项
Application会扫描它所在的包及其子包。

测试类使用`@SpringBootTest`注解，并且application测试类应当和被测试的application放在同一个包路径下（否则要向注解中传入被测试类的class作为参数）以读取application中的配置。

如果要修改SpringBoot提供的依赖（如把tomcat换成其他服务器），则可以用exclusion去掉原依赖，然后导入对应依赖（也应当用SpringBoot提供的依赖）。

使用`@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})`可以避免数据源相关的自动配置。这在父工程有数据源依赖但当前工程不需要数据源的时候很有用。
# 配置文件
一般使用yml后缀。读取顺序：`yaml`->`yml`->`properties`。因此`properties`优先级最高。

# 多环境
## 配置方式
配置`spring.profiles.active`的值即可。

```yml
spring:
	profiles:
		active: dev
```

之后可以使用`---`划分配置文件并使用`spring.config.activate.on-profile`来定义对应环境；也可以分多个文件进行配置（文件名后面加环境名，如`application-dev.yml`）。
## 通过Maven配置
应当使用maven的参数来控制环境的选择。

加入`maven-resource-plugin`插件即可让配置文件通过`${xxx}`获得maven中定义的参数，从而可以通过maven控制yml中`profiles.active`的值。

```xml
<!--引入插件-->
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-resources-plugin</artifactId>
	<version>3.2.0</version>
	<configuration>
		<encoding>UTF-8</encoding>
		<useDefaultDelimiters>true</useDefaultDelimiters>
	</configuration>
</plugin>
```

```xml
<!--定义属性并指定默认值-->
<profile>
	<id>dev_env</id>
	<properties>
		<profile.active>dev</profile.active>
	</properties>
	<activation>
		<activeByDefault>true</activeByDefault>
	</activation>
</profile>
```

## 外部配置
在jar包同目录下写`application.yml`即可进行覆盖配置，优先级更高。

为上线服务，灵活配置。

# 整合Mybatis
Mybatis观察dao的返回值以辨别实体类。

使用`@Mapper`注解来定义dao类，使得Mybatis得以管理它。

```yml
spring:
	datasource:
		#高版本无需设时区
		url: jdbc:mysql://localhost:3306/ssm_db?serverTimezone=UTC
		username: root
		password: 1234
		driver-class-name: com.mysql.cj.jdbc.Driver
		#使用Druid连接池
		type: com.alibaba.druid.pool.DruidDataSource
```

```yml
mybatis-plus:
	#开启驼峰功能，保证实体类属性正确映射数据表属性
	configuration:
	    map-underscore-to-camel-case: true
	#为未赋值的主键自动添加uuid
	global-config:
		db-config:
			id-type: uuid
```

# 取缔继承依赖boot
```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-dependencies</artifactId>  
    <version>2.6.3</version>  
    <type>pom</type>  
    <scope>import</scope>  
</dependency>
```

出现了蜜汁bug，尚未找到原因：

![](assets/Pasted%20image%2020230313142743.png)

![](assets/Pasted%20image%2020230313142728.png)
# 问题
## JSON包冲突

让test依赖排除一个包即可：

```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-test</artifactId>  
    <scope>test</scope>  
    <exclusions>  
        <exclusion>  
            <groupId>com.vaadin.external.google</groupId>  
            <artifactId>android-json</artifactId>  
        </exclusion>  
    </exclusions>  
</dependency>
```