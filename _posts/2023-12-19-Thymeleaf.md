---
layout: post
title:  "Thymeleaf"
date:   2023-12-19 11:00:00 +0900
category: [Spring]
tags: [Thymeleaf, spring]
lastmod : 2023-12-19 11:00:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---

Spring Boot 어플리케이션에서 thymeleaf를 써야하는 이유 : <br>
JSP의 제약사항 때문이다. <br>

<br>

JAR로 빌드하여 Spring boot 내장 톰캣으로 실행할 경우에는 “Whitelabel Error Page” 오류가 뜬다. <br>
<br>
Spring Boot + jsp 자체에는 문제가 없으나, jar로 빌드할 경우에 jsp는 지원되지 않는다. <br>
(Spring Boot + jsp 어플리케이션을 war로 빌드하는 경우에는 문제가 없다.) <br>
<br>

(spring boot jsp 제약사항 : [https://docs.spring.io/spring-boot/docs/2.1.5.RELEASE/reference/htmlsingle/#boot-features-jsp-limitations](https://docs.spring.io/spring-boot/docs/2.1.5.RELEASE/reference/htmlsingle/#boot-features-jsp-limitations){:target="_blank"})

<br>

**JAR / WAR 빌드시 배포의 차이**
*  JAR 는 내장 WAS을 이용하여 jar파일만으로 서비스가 가능함
*  WAR는 외장 WAS를 이용해야 서비스가 가능함

