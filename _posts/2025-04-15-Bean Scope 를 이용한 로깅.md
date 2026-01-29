---
layout: post
title:  "Bean Scope 를 이용한 로깅"
date:   2025-04-15 11:00:00 +0900
category: [Java]
tags: [Java,Spring]
lastmod : 2025-04-15 11:00:00 +0900
sitemap :
changefreq : daily
priority : 1.0
---

# 서론
스프링은 다양한 스코프를 지원한다. <br>
- singleton : 기본 스코프로 컨테이너의 시작-종료 까지 유지되는 가장 넓은 범위의 스코프
- prototype : 해당 타입 빈의 생성과 의존관계 주입 까지만 관여하고 **더는 관리하지 않는 가장 짧은 범위의 스코프**로 항상 새로운 인스턴스를 생성해서 반환 
- 웹 스코프
	- reqeust : 웹 요청이 들어오고 나갈때까지 (요청-응답) 유지되는 스코프
	- session : 웹 세션이 생성되고 종료될 때 까지 유지되는 스코프
	- application : 웹의 서블릿 컨텍스트와 같은 범위로 유지되는 스코프

<br>

위 스코프 중 request 스코프를 이용해 사용자 요청별 로깅을 좀 더 알아보기 쉽게 커스텀 한다. 
<br>




# AOP 클래스

```java
  
/**  
 * Controller, Service 로깅 AOP (Util, Bean은 로깅되지 않도록 조심한다.)  
 * Web Scope를 이용하여 로깅  
 */  
@Configuration  
@Aspect  
@RequiredArgsConstructor  
public class LogAOPConfig {  
  
    private final HttpServletRequest request;  
  
    private final LogScope logScope;  // 얘를 이용해서 즉시 logging 하지 않고 LogScope객체에 로그를 모아두기만 합니다.
    private final ReflectionBean reflectionBean;  
    private final AccessLogClient accessLogClient;  
    private final UtilsBean utilsBean;  
  
    // controller logging    
    @Around("defaultLogging() && noLogging()")  
    public Object contollerLog(ProceedingJoinPoint pjp) throws Throwable {  
        Object result = null;   // 요청 처리 결과  
  
        long startAt = System.currentTimeMillis();  // 작업 시작 시간  
  
        String params = ", ";     // logging을 위한 요청 parameter를 문자열로 변환  
        if(pjp.getArgs().length > 0){  
            params = parmReflection(pjp.getArgs());  
        }  
  
        // log 가독성을 위한 depth별 공백 추가  
        String logPrefix = prefixSpace(pjp.getTarget().getClass().getSimpleName());  
  
        // request 로깅  
        logScope.log(LogLevel.INFO, "{}.{}({})", logPrefix + pjp.getTarget().getClass().getSimpleName(),  
            pjp.getSignature().getName(), params.replaceAll("\\n", "").replaceAll(System.lineSeparator(), ""));  
        try {  
            result = pjp.proceed();     // 요청 처리  
            return result;              // 요청에 대한 응답 return        
        } finally {  
  
            long endAt = System.currentTimeMillis();    // 작업 종료 시간  
  
            if(result != null){ // 결과 파라미터 줄넘김 문자 삭제 ('\n' 으로 삽입한건 System.lineSeparator()으로 제거되지 않는다.)  
                result = result.toString().replaceAll("\\n", "").replaceAll(System.lineSeparator(), "");  
            }  
            logScope.log(LogLevel.INFO, "|==> {}.{}() = {} ({}ms)", logPrefix + pjp.getTarget().getClass().getSimpleName(),  
                pjp.getSignature().getName(), result, endAt - startAt );  
        }  
    }  
  
  
    @Around("accessLogging()")  
    public Object accessLog(ProceedingJoinPoint pjp) throws Throwable {  
        ///////////// access log /////////////  
        // @AccessAnnotation 에 저장된 log메시지를 가져와 DB에 저장  
        MethodSignature signature = (MethodSignature) pjp.getSignature();  
        if(signature.getMethod().getAnnotation(AccessAnnotation.class) == null){  
            return pjp.proceed();  
        }  
  
        String action = signature.getMethod().getAnnotation(AccessAnnotation.class).action();  
        String log  = signature.getMethod().getAnnotation(AccessAnnotation.class).log();  
  
        UserDto userDto = utilsBean.getUserDtoFromRequest();  
        AccessLogRequest accessLogRequest = AccessLogRequest.builder()  
            .userId(userDto.getId())  
            .log(log)  
            .action(action)  
            .regDate(LocalDateTime.now())  
            .build();  
        accessLogClient.createAccessLog(accessLogRequest);  
        ///////////////////////////////////////  
        return pjp.proceed();  
    }  
    /* null인 객체는 로그에서 제외 */    
    private String parmReflection(Object[] args){  
        String params = "";  
        for(int i = 0; i < args.length; i ++) {  
            if(args[i] instanceof HttpServletResponse ||  
                    args[i] instanceof HttpServletRequest ||  
                    args[i] == null) 
            {      
				/* undertow servlet 객체는 stream 오류 발생 */                
				continue;  
            }  
            MultiValueMap<String, String> map = reflectionBean.buildParamMap(args[i]);  
            for (String key : map.keySet()) {  
                for(String val : map.get(key)){  
                    params += key +"="+val+",";  
                }  
            }  
        }  
        return params;  
    }  
  
    /* 로그 가독성을 위해 depth 별 공백 추가 */    
    private String prefixSpace(String className){  
        String logPrefix="";  
        if( className.contains("Service")){  
            logPrefix = "  ";  
        }else if(className.contains("Bean") ||  
                className.contains("Client")) {  
            logPrefix = "    ";  
        }  
        return logPrefix;  
    }  
  
    @Pointcut("execution(* com.example.*.controller..*(..)) || " +  
            "execution(* com.example.*.service.impl..*(..)) ")  
    public void defaultLogging() {}  
  
  
  
    @Pointcut("!within(@com.example.conf.LogAOPConfig.NoLogging *)")  
    public void noLogging() {}  


    @Pointcut("execution(* com.example.*.controller..*(..))")  
    public void accessLogging() {}  

	// NoLogging 커스텀 어노테이션  
    @Target({ElementType.TYPE, ElementType.METHOD})  
    @Retention(RetentionPolicy.RUNTIME)  
    public @interface NoLogging {}  
    
	/**  
	 * 시스템 접근 로깅을 위해    <br>  
	 * 어떤 행위를 했는지 텍스트로 명시      <br>  
	 *  
	 * 컨트롤러에서 @AccessAnnotation(action="사용자 목록 조회") 으로 사용   <br>  
	 *  
	 * @param: log - ex.사용자 목록 조회    / vod 상세 조회...       <br><br>  
	 * @param: action - [SELECT, UPDATE, DELETE, CREATE, DOWNLOAD]  
	 * */
	@Retention(RetentionPolicy.RUNTIME)  
	@Target(ElementType.METHOD)  
	public @interface AccessAnnotation {  
	    String action();  
	    String log();  
	}
}
```

<br>

# Scope를 이용한 HTTP 요청 별 로그 모아 찍기 클래스

```java
  
/**  
 * 모든 request 요청에 대해 response 시점에 한번에 로깅  
 * (=요청별로 로그 모아 찍기)  
 * * proxyMode = TARGET_CLASS * -> 최초에 LogScope bean이 존재하지 않기 때문에  
 *    정상적으로 실행될 수 있도록 가짜 프록시 클래스를 주입해두는 옵션  
 * */  
@Component  
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)  
@RequiredArgsConstructor  
public class LogScope {  
    private final LogPrintBean logPrintBean;  
  
    private String uuid;  
  
    // 로그 list => level : LogLevel.{debug/info/...} / log : {log message}    
    private List<Map<String, Object>> logList = new ArrayList<>();  
  
    /**  
     * log 수집  
     * ex) logScope.log(LogLevel.DEBUG,"userDto : {}", userDto);  
     * @param logLevel 노출할 로그의 레벨 (LogLevel.INFO)  
     * @param message logMsg : {} 포멧 으로 작성  
     * @param param message {}에 들어갈 파라미터 순서대로 추가  
     * */  
    public void log(LogLevel logLevel, String message, Object... param){  
        Map<String, Object> log = new HashMap<>();  
        log.put("level", logLevel);  
        log.put("log", message);  
        log.put("param", param);  
        logList.add(log);  
    }  
  
    @PostConstruct  
    public void init(){  
        uuid = UUID.randomUUID().toString().substring(0, 8);  
    }  
  
    @PreDestroy  
    public void close(){  
        // 실제 로깅은 singleton 빈으로 실행 (여러 스레드가 섞여서 실행되지 않도록)  
        logPrintBean.logging(uuid, logList);  
    }  
  
  
  
}
```

<br>

# 위를 이용한 로깅 결과물

> client가 /user HTTP 요청 -> UserController 의 findAllUser 요청됨 (Request 파라미터는 useYn: 'A')  -> <br>
> UserServiceImpl 의 findAllUser 요청됨 (Request 파라미터는 useYn: 'A' 외 여러개) -> 해당 메서드에서 응답 (UserRespone 객체 리턴, 445ms 소요됨) -> <br>
> UserServiceImpl 의 findDepartmentName 요청됨 -> findDepartmentName 응답 (departmentList 객체 리턴, 196ms 소요됨) ->  <br>
> UserController에서 작업이 끝난 뒤 system/user.html 화면 클라이언트에게 응답 (총 소요시간 901ms) <br>
> (User와 Department DB가 분리되어 있기 때문에 컨트롤러에서 service를 두번 나누어 요청한 것입니다.)

<br>

```console
2024-05-17 14:17:30.025 [INFO ] [XNIO-1 task-4]- [4b6dc656] URL = /user
2024-05-17 14:17:30.026 [INFO ] [XNIO-1 task-4]- [4b6dc656] UserController.findAllUser(useYn=A,)
2024-05-17 14:17:30.026 [INFO ] [XNIO-1 task-4]- [4b6dc656] UserServiceImpl.findAllUser(useYn=A,... )
2024-05-17 14:17:30.026 [INFO ] [XNIO-1 task-4]- [4b6dc656] |==> UserServiceImpl.findAllUser() = UserResponse(totalCount=3, ...) (445ms)
2024-05-17 14:17:30.026 [INFO ] [XNIO-1 task-4]- [4b6dc656] UserServiceImpl.findDepartmentName(, )
2024-05-17 14:17:30.118 [INFO ] [XNIO-1 task-4]- [4b6dc656] |==> UserServiceImpl.findDepartmentName() = DepartmentResponse(departmentList=[]) (196ms)
2024-05-17 14:17:30.119 [INFO ] [XNIO-1 task-4]- [4b6dc656] |==> UserController.findAllUser() = system/user (901ms)
```

