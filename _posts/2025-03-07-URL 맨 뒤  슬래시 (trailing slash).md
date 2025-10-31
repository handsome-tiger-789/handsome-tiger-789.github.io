---
layout: post
title:  "URL 맨 뒤  슬래시 (trailing slash)"
date:   2025-03-07 11:00:00 +0900
category: [Spring]
tags: [Spring]
lastmod : 2025-03-07 11:00:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---

`@GetMapping("/user")` 라는 URL을 하나 만들었을 경우 <br>
`http://test.com/user/` 로 요청 하면 페이지를 찾을 수 없는 오류가 발생한다. <br>
spring boot 기준 3 이전까지는 기본적으로 매칭해주었으나, 3 부터는 매칭되지 않는 설정이 기본이다. <br>
<br>

해결방법
- trailing slash path까지 작성 <br>
   `@GetMapping(path={"/user", "/user/"})` <br>
   누락되면 매칭 오류가 발생할 수 있음 <br>

- WebMvcConfigurer 설정 <br> 
   setUseTrailingSlashMatch을 true로 주어 매칭되도록 설정 <br>


```java
import org.springframework.context.annotation.Configuration;  
import org.springframework.web.servlet.config.annotation.PathMatchConfigurer;  
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;  
  
@Configuration  
public class WebConfig implements WebMvcConfigurer {  
    @Override  
    public void configurePathMatch(PathMatchConfigurer configurer) {  
       configurer.setUseTrailingSlashMatch(true);  
    }  
}
```

다만 위 옵션은 deprecated 되어 있으므로 추후 사라질수 있음<br>

- Filter 이용 <br>
  필터를 등록하여 요청 url 맨 뒤에 `/` 가 존재하는 경우 `/` 를 지워주는 필터 등록 <br>
 
```java
@Component
public class ServletRequestWrapperFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequestWrapper requestWrapper = new HttpServletRequestWrapper((HttpServletRequest) request) {
            @Override
            public String getRequestURI() {
                String requestURI = super.getRequestURI();
                if (StringUtils.endsWith(requestURI, "/")) {
                    return StringUtils.removeEnd(requestURI, "/");
                }
                return requestURI;
            }
        };
        chain.doFilter(requestWrapper, response);
    }
}
```

현재 가장 적합한 방법은 Filter를 등록하는 방법 같음