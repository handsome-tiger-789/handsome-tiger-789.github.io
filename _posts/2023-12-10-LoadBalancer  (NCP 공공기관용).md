---
layout: post
title:  "LoadBalancer  (NCP 공공기관용)"
date:   2023-12-10 11:00:00 +0900
category: [클라우드]
tags: [클라우드, 네트워크]
lastmod : 2023-12-10 11:00:00 +0900
sitemap :
changefreq : daily
priority : 1.0
---

서비스 소개 링크 : [https://www.gov-ncloud.com/v2/product/networking/loadBalancer](https://www.gov-ncloud.com/v2/product/networking/loadBalancer){:target="_blank"} <br>
<br>

용도 : <br> 
VPC 내 서버 사이 로드밸런싱 기능을 제공한다. <br>
NKS 서비스 이용 시 ingress 를 생성하면 자동으로 LB가 생성되었다. <br>


![](/assets/img/2023-12-10-img-lb/Loadbalancer.png) <br/>

<br>

트러블슈팅
* ingress 삭제 시 공인IP가 삭제되지 않도록 공인아이피 삭제 방지 어노테이션을 추가 필요
* 모의침투 취약점으로 Response Header값 추가가 필요해 LB에서 한번에 설정하고 싶었으나<br>2024/02/07 기준 NCP alb-ingress 에서는 헤더 추가 어노테이션을 지원해주지 않아 어플리케이션 단에서 처리
* LB > Target Group 페이지에서 모든 SVC 조회 가능 <br>
  ingress와 정상적으로 연결되어 있다면 UP 상태로 조회되기 때문에 503 오류가 난다면 이 페이지에서 연결상태 확인 가능
* healthcheck URL을 따로 설정하지 않았다면 기본적으로 '/' 경로이므로<br>어플리케이션에 '/' 가 200대 코드를 응답하지 않으면 오류가 발생할 수 있음

