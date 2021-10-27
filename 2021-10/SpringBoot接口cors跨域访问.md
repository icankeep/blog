## 方式一 CorsFilter.java
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

@Configuration
public class CorsConfig {
    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", buildConfig());
        return new CorsFilter(source);
    }

    private CorsConfiguration buildConfig() {
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        // 1允许任何域名使用
        corsConfiguration.addAllowedOrigin("*");
        // 2允许任何头
        corsConfiguration.addAllowedHeader("*");
        // 3允许任何方法（post、get等）
        corsConfiguration.addAllowedMethod("*");
        return corsConfiguration;
    }
}
```

## 方式二 
在Controller类或方法上贴上注解 `@CrossOrigin(origins = "*")`

## 调整顺序
```java
@Bean
public FilterRegistrationBean<Filter> mtFilter(@Qualifier("corsFilter") Filter corsFilter) {
    FilterRegistrationBean<Filter> registration = new FilterRegistrationBean<>();
    registration.setFilter(corsFilter);
    registration.addUrlPatterns("/*");
    registration.setDispatcherTypes(REQUEST);
    registration.setName("corsFilter");
    // before sso
    registration.setOrder(1);
    return registration;
}
```

## 浏览器控制台测试
```js
$.ajax({
    url: "http://localhost:8080/hello",//发送的路径
    type: "get",
    contentType: "application/json", 
    dataType:"json",//服务器返回的数据类型
    success: function(data) {
        if(data.code===200){
            alert("已提交成功");
        } else {
            console.log(data);
        }
    },
    error: function (data){
        alert("提交失败");
    }
});
```
