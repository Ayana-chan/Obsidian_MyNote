# 前端
## 页面位置
若菜单设置为`product/category`，则前端文件路径应当s是`src/views/modules/product/category.vue`。

# 后端
## Nacos相关
renrenfast想要使用nacos，须选固定的版本：

```xml
<dependency>  
   <groupId>com.alibaba.cloud</groupId>  
   <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>  
   <version>2021.0.1.0</version>  
</dependency>  
<dependency>  
   <groupId>com.alibaba.cloud</groupId>  
   <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>  
   <version>2021.0.1.0</version>  
</dependency>
```