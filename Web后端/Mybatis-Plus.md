
## 配置类
```java
@Configuration  
@EnableTransactionManagement //开启事务  
@MapperScan("cn.ayana.gulimall.product.dao")  
public class MyBatisConfig {  
	
}
```

## 开启插件
在配置类里面加上Bean即可开启对应插件。

新版本的MP使用`MybatisPlusInterceptor`作为插件主体，其他插件都通过`interceptor.addInnerInterceptor`来挂载。

```java
/**
* 新的分页插件,一缓和二缓遵循mybatis的规则,需要设置 MybatisConfiguration#useDeprecatedExecutor = false 避免缓存出现问题(该属性会在旧插件移除后一同移除)
*/
@Bean
public MybatisPlusInterceptor mybatisPlusInterceptor() {
	MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
	interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
	return interceptor;
}

@Bean
public ConfigurationCustomizer configurationCustomizer() {
	return configuration -> configuration.setUseDeprecatedExecutor(false);
}
```
## 老版本开启分页插件

```java
@Bean  
PaginationInterceptor paginationInterceptor(){  
	PaginationInterceptor paginationInterceptor=new PaginationInterceptor();  
	//请求页面大于最大页时，返回首页而非空  
	paginationInterceptor.setOverflow(true);  
	//设置最大单页数量，-1表示不受限  
	paginationInterceptor.setLimit(1000);  
	return paginationInterceptor;  
}
```

## 逻辑删除
在实体类的属性上加上`@TableLogic`即可让该属性成为逻辑删除标志位，默认0删除、1显示。

可以传入`value`表示为何值时不删除，`delval`表示为何值时删除。

此时，调用MybatisPlus的删除方法时，其实底层进行的是UPDATE操作。

## 精准查询ID或模糊查询Name
```java
public PageUtils queryPage(Map<String, Object> params) {  
  
    String key= (String) params.get("key");  
    QueryWrapper<BrandEntity> queryWrapper=new QueryWrapper<>();  
    if(StringUtils.hasText(key)){  
        queryWrapper.eq("brand_id",key).or().like("name",key);  
    }  
  
    IPage<BrandEntity> page = this.page(  
            new Query<BrandEntity>().getPage(params),  
            queryWrapper  
    );  
  
    return new PageUtils(page);  
}
```

## 拼接查询
```sql
SELECT xxx FROM pms_attr_group WHERE (catalog_id = ? AND ( (attr_group_id = ? OR attr_group_name LIKE ?) ))
```

```java
//根据分类先做第一步筛选  
QueryWrapper<AttrGroupEntity> wrapper = new QueryWrapper<AttrGroupEntity>().eq("catelog_id", catelogId);  
  
//若有关键词则根据关键词再筛选  
String key = (String) params.get("key");  
if (StringUtils.hasText(key)) {  
    wrapper.and((obj) -> {  
        //id相同，或者名字中存在该子串（like左右百分号）  
        obj.eq("attr_group_id", key).or().like("attr_group_name", key);  
    });  
}  
  
IPage<AttrGroupEntity> page = this.page(  
        new Query<AttrGroupEntity>().getPage(params),  
        wrapper  
);
```

# 问题

map里面的sql，数据表名是飘号（数字1左边的）包起来的，而非单引号。












