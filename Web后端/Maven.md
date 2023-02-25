# 概念
## 可选依赖
在dependency标签中设置optional为true，即可对上隐藏该依赖，也即取消该依赖的传递性。
## 排除依赖
在dependency标签中设置exclusions，使得在导入该dependency时防止导入exclusion指定的包。
## 聚合
使用聚合可以让一个工程统一管理其他多个工程，让多个maven一起运作。父主动管理子。用于快速构建项目。

新建一个聚合模块，将其packaging设为pom（表明用于聚合或继承）。然后使用modules以路径的形式配置要管理的各个模块。

```xml
<modules>
    <module>../maven_02_ssm</module>
    <module>../maven_03_pojo</module>
    <module>../maven_04_dao</module>
</modules>
```

会自动按照依赖关系决定的顺序进行加载。
## 继承
使用parent定义父工程，从父工程中继承依赖，简化依赖配置与版本管理。子主动依附父。用于快速配置。

以此可以把项目通用的依赖都放在父工程中。

```xml
<parent>
	<groupId>com.exm</groupId>
	<artifactId>fid</artifactId>
	<version>1.6.1</version>
	<!--定位到pom-->
	<!--空配relativePath则会去本地库和中央库中去找依赖包-->
	<!--不配可能会出bug-->
	<relativePath>../FatherFileName/pom.xml</relativePath>
</parent>
```

父工程也可以提供一些可选依赖,本质上是版本指定。子工程要用此依赖的话，在导入的时候不指定版本，默认使用父工程选择的版本。子工程也会继承dependencyManagement的设定。

```xml
<!--父工程-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.16</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

```xml
<!--子工程-->
<dependencies>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
    </dependency>
</dependencies>
```

和依赖的区别是：
- 依赖会导入包含代码的整个工程，而继承只是继承一个pom。
- 依赖无法获取`<dependencyManagement>`。
## 属性
使用properties标签定义几个属性，可以在其他地方通过`${xxx}`的形式引用。

```xml
<properties> 
	<spring.framework>4.0.4.RELEASE</spring.framework> 
</properties>
```

`${project.basedir}`是一个内置属性，它能同时成为所有模块的根路径，可用于快速匹配存在于某一个模块中的某资源。

使用`mvn help:system`可以查看环境变量属性和Java系统属性。
# 使用
## 问题
### 插件爆红
把插件的三属性放进dependency，刷新maven后即自动下载，之后删掉这些dependency即可。
### SpringBoot的‘parent.relativePath‘ of POM
[解决Maven ‘parent.relativePath‘ of POM问题 - 知乎](https://zhuanlan.zhihu.com/p/453547775)

通过添加`<relativePath/>`，使其父工程忽略项目直接从仓库里面找。注意，这会间接导致其他报错，因此要优先解决！