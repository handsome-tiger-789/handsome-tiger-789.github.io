---
layout: post
title:  "@Transcational이 rollback을 해주지 않았다."
date:   2025-05-21 11:00:00 +0900
category: [Spring]
tags: [Spring, JDBC]
lastmod : 2025-05-21 11:00:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---

>Repository(=Dao)가 아닌 Service에 @Transactional 어노테이션을 걸었으면 Exception 발생 시 Rollback을 해줘야하는거 아닌가??

<br>

```java
@Service 
public class MyServiceImpl extends MyService { 
	@Override 
	@Transactional 
	public ResponseEntity updateBatchEnd(BatchEndUpDto dto) throws Exception { 
		batchRepository.updateBatchEnd(dto); 
		vodRepository.updateBatchEnd(dto); 
		encLogRepository.updateLogSched(EncLogUpdateDto.builder() 
			.status(dto.getStatus()) 
                        ...
			.elEndTime(dto.getElEndTime()) 
			.elLog(dto.getErrMsg()) .build()
		); 
		
		return ResponseEntity.status(HttpStatus.OK).body(true); 
	} 
}
```


위 코드에서 잘못된 조건 처리로 인해 예상치 못한 오류가 발생했다. <br>

그런데 ENC_LOG 에는 업데이트가 잘 되어있다?????? 🫤 <br>

매서드 위에 @Transactional이 있는데 도대체 왜 롤백되지 않은걸까??? <br>



## @Transactional 어노테이션의 롤백

@Transactional 어노테이션은 예외처리 (try-catch)하지 않은 오류에 대해 모두 롤백처리 해주는 줄 알았는데 그게 아니었다. <br><br>

Runtime Exception 또는 Error가 발생했을 때만 롤백 처리해주는게 기본이다.  <br>
Checked Exception 은 롤백처리 해주지 않는다. <br>

>🤔 Checked Exception이 뭐지? (프로그램이 제어할 수 없지만 개발자가 처리 가능한 예외라는데…)

![](/assets/img/2025-05-21-@Transactional이 rollback 해주지 않았다/Transactional Rollback 범위.png) <br/>

출처 : [https://interconnection.tistory.com/122](https://interconnection.tistory.com/122){:target="_blank"}

위 그림에서 빨간색으로 묶인 부분은 UncheckedException, 나머지는 다 CheckedException인거다.  <br>
(Error와 RuntimeException은 Unchecked Exception이고, Exception은 Checked Exception이다.) <br>
<br>

즉, 저기 빨간 부분의 오류만 Rollback 해준다는거다. <br>

```java
@Transactional 
public void test(TestEntity entity) throws Exception { 
	dao.save(entity); 
	throw new RuntimeException(); 
}
```

얘는 Rollback 해주는데,

```java
@Transactional 
public void test(TestEntity entity) throws Exception { 
	dao.save(entity); 
	throw new Exception(); 
}
```

얘는 안해준다..

>**JUnit** 테스트에서는 테스트 코드는 모두 Rollback 하기 때문에 **@Rollback(false)** 를 붙이고 테스트 하니 **정상적으로 되는것 같지 않았다.**

<br>

**그럼 Checked Exception은 어떻게 롤백해요?** <br>

트랜잭션 어노테이션에서 제공되는 속성 중 **rollbackFor** 라는게 있다. <br>  
롤백 하고자 하는 Exception을 지정할 수 있는 속성이다. <br>


```java
@Transactional(rollbackFor = Exception.class) 
public void test(TestEntity entity) throws Exception { 
	dao.save(entity); 
	throw new Exception(); 
}
```

이렇게 rollbackFor 에 Exception.class 를 지정해주면 아까 rollback되지 않던 코드가 rollback 된다. <br>

CheckedException 부모는 Exception 하나이기 때문에 rollbackFor에는 **Exception.class 하나만 지정**해주면 **모든 Exception에 대해 롤백 할 수 있게 된다.** <br>

반대로 특정 오류가 나도 Rollback하고싶지 않을때는 **noRollbackFor** 속성에 예외 클래스를 지정해주면 된다. <br>