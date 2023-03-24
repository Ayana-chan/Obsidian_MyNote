# 杂项
Application会扫描它所在的包及其子包。

测试类使用`@SpringBootTest`注解，并且application测试类应当和被测试的application放在同一个包路径下（否则要向注解中传入被测试类的class作为参数）以读取application中的配置。

如果要修改SpringBoot提供的依赖（如把tomcat换成其他服务器），则可以用exclusion去掉原依赖，然后导入对应依赖（也应当用SpringBoot提供的依赖）。

使用`@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})`可以避免数据源相关的自动配置。这在父工程有数据源依赖但当前工程不需要数据源的时候很有用。

Spring提供的`StringUtils`工具类包含了很多方便的String相关方法。
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

# AOP
`@RestControllerAdvice`= `@ControllerAdvice` + `@ResponseBody`。

# Controller接收参数
## 获取url内的参数
在RequestMapping里面使用`{}`来规定参数名，然后使用PathVariable接收：

```java
@RequestMapping("/list/{catalogId}")  
//@RequiresPermissions("product:attrgroup:list")  
public R list(@RequestParam xxx, @PathVariable("catalogId") Long catalogId){
```

## 校验
[表单校验 JSR303](Web后端/杂项.md#表单校验%20JSR303)


# 异常处理
可以通过AOP来建立一个处理所有未处理异常的Advice。

```java
@RestControllerAdvice(basePackages = "cn.ayana.gulimall.product.controller")  
public class GulimallExceptionControllerAdvice {  
  
    @ExceptionHandler(value = Exception.class)  
    public R handleException(Exception e) {  
  
        return R.error(BizCodeEnum.UNKNOWN_EXCEPTION.getCode(), BizCodeEnum.UNKNOWN_EXCEPTION.getMsg());  
    }  
  
}
```

若有多个ExceptionHandler，则会自动寻找匹配度最高的（所有符合的异常类中深度最低的）一个进行处理。

# 返回值省略
有时候返回的json对象的某一个变量为null或者空，需要让其直接不返回。可以在实体类的对应属性上标注`@JsonInclude(JsonInclude.Include.NON_EMPTY)`，即可在该属性（集合类）为空时不返回改属性。
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

## 所依赖包内的Bean未加载
因为Application默认只扫描其同级文件及子文件，而其他包的上层目录结构多半是不一样的。此时就可能需要手动设置一下，选择其中一个注解放到Application上：

```java
@Import(value = GulimallExceptionControllerAdvice.class)  

//这里要写上当前微服务本来所要扫描的包
@ComponentScan({"cn.ayana.gulimall.product","cn.ayana.gulimall.common.advice"})
```

## 实体类属性出现默认值
实体类的属性不要使用基本类型，比如`long`最好改成`Long`，这样的话就不会在新建对象的时候自动赋值0了。

## 实体类继承问题
使用`@Data`注解的类在被继承后，不会自动给子类的新属性进行自动生成，因此子类上也应当标注`@Data`。

此时可能出现警告，需要加入如下注解来解决：
```java
@EqualsAndHashCode(callSuper = true)
```

实际上，子类的`@Data`在生成时不会考虑父类的属性，因此才需要加上述的注解。同样的，子类的`toString()`也不包含父类的属性，则需要加`@ToString(callSuper = true)`。