# 开放所有url

```java
public class WebConfig implements WebMvcConfigurer {  
    @Override  
    public void addInterceptors(InterceptorRegistry registry) {  
        registry.addInterceptor(new WebContentInterceptor()).addPathPatterns("/**").excludePathPatterns("/**");  
    }  
}
```