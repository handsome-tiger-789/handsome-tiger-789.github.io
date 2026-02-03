---
layout: post
title:  "ERD Editor"
date:   2026-02-02 11:00:00 +0900
category: [DB]
tags: [DB, plugin]
lastmod : 2026-02-02 11:00:00 +0900
sitemap :
changefreq : daily
priority : 1.0
---

DB는 ERD 를 얼마나 잘 작성하고 유지보수 하느냐가 매우 중요하다. <br>
그만큼 한 눈에 파악하기 쉬워야하기도 하지만 실시간 공유와 히스토리 파악이 중요하다고 생각한다. <br>
<br>
유료 ERD 프로그램도 있고 DB 툴에서 제공하는 프로그램도 있으나 오늘은 개발툴에서 프로젝트별로 확인 가능한 플러그인을 소개하고자 한다. <br>

<br>

# 설치

![](/assets/img/2026-02-02-img-erd-editor/plugin.png) <br/>

IntelliJ 와 VSCode 에서 **플러그인** 화면을 통해 무료로 다운로드 받을 수 있다. <br>
<br>

*다만 IntelliJ에서는 캔버스 랜더링으로 인해 코딩시에도 버벅임이 생길 수 있으며, <br>
금일 기준 맥에선 정상작동하지 않아 VSCode를 추천*

<br><br>

플러그인 소개 화면에서 [Guides](https://docs.erd-editor.io/docs/category/guides/){:target="_blank"} 에 걸려있는 링크를 통해 플러그인 사용법과 기능을 확인 할 수 있다. <br>
<br>

# ERD 그리기



## ERD 파일 생성

![](/assets/img/2026-02-02-img-erd-editor/new-erd-file.png) <br/>

프로젝트 내 {파일명}.erd.json 명칭으로 파일을 하나 생성하면 ERD 파일 생성은 끝난다. <br>

<br>

![](/assets/img/2026-02-02-img-erd-editor/editor-file.png) <br/>

파일을 열어보면 상단에 몇몇 도구 아이콘과 빈 화면을 확인할 수 있다. <br>
<br>

## DBMS 선택

![](/assets/img/2026-02-02-img-erd-editor/db-select.png) <br/>

빈 캠버스에 우클릭 후 사진과 같은 경로를 통해 DB를 선택 할 수 있다.<br>
*DB 에 따라 컬럼 타입이 달라지기 때문에 가장 먼저 세팅하는게 좋다.*
<br>


## 테이블 만들기


![](/assets/img/2026-02-02-img-erd-editor/newtable-1.png) <br/>

빈 캔버스에 우클릭이나 위에 나오는 단축키를 활용해 테이블을 그릴 수 있다.<br>
<br>

![](/assets/img/2026-02-02-img-erd-editor/make-user-table.png) <br/>

내용은 아래와 같다. <br>

| 테이블명 | 테이블 정보(설명) |                               |            |                   |     |            |
| ---- | ---------- | ----------------------------- | ---------- | ----------------- | --- | ---------- |
| 컬럼명  | 컬럼 타입      | N-N : Not Null<br>NULL : Null | Unique Key | Auto<br>Increment | 기본값 | 컬럼 정보 (설명) |

*열쇠 모양은 PK 여부* <br>
*view option 을 위와 같이 선택하여 볼 수 있다.* <br>
<br><br>


![](/assets/img/2026-02-02-img-erd-editor/user-table-properties-1.png) <br/>
![](/assets/img/2026-02-02-img-erd-editor/user-table-properties-2.png) <br/>

테이블에서 우클릭 하면 테이블의 상세 설정을을 확인할 수 있다. <br>
<br>

![](/assets/img/2026-02-02-img-erd-editor/idx_setting.png) <br/>
![](/assets/img/2026-02-02-img-erd-editor/idx_sql.png) <br/>

예시로 위처럼 index 또한 추가로 설정 가능하다. <br>
<br>


## 연관관계 그리기

![](/assets/img/2026-02-02-img-erd-editor/relationship-1.png) <br/>

연관관계를 그려보기 위해 게시글 테이블을 추가하고 <br>
캠퍼스에서 우클릭 하여 알맞는 연관관계 선을 선택한다. <br>
<br>

![](/assets/img/2026-02-02-img-erd-editor/relationship-2.png) <br/>

user_id가 post 테이블에 FK 로 들어가야 하기 때문에 <br>
user 를 먼저 클릭 후 post 를 클릭하면 위와 같은 연관관계 선이 그려진다. <br>
<br>

![](/assets/img/2026-02-02-img-erd-editor/relationship-3.png) <br/>

user 테이블의 PK 컬럼명이 그대로 들어가서 id 컬럼이 2개가 되므로 <br>
컬럼명과 설명을 수정한 뒤 드래그하여 보기좋게 정렬한다. <br>
<br>

![](/assets/img/2026-02-02-img-erd-editor/post-properties-sql.png) <br/>

참고로 자동 생성되는 sql 에 물리적인 FK는 작성되지 않으니 필요한 경우 주의해야함. <br>
<br>


## 버전관리

![](/assets/img/2026-02-02-img-erd-editor/commit-diff-view.png) <br/>

커밋 전 변경사항 비교 또한 랜더링된 캔버스로 보기좋게 확인이 가능하다. <br>
<br>


![](/assets/img/2026-02-02-img-erd-editor/final-project-folder.png) <br/>
![](/assets/img/2026-02-02-img-erd-editor/project-git-graph.png) <br/>

최종적으로 프로젝트 폴더내 erd 폴더를 따로 추가하고 <br>
erd 와 sql 을 함께 수정하며 프로젝트 git으로 함께 관리할 수 있다. <br>
<br>


git : [https://github.com/handsome-tiger-789/demo-erd-editor](https://github.com/handsome-tiger-789/demo-erd-editor){:target="_blank"}



# 결론

* DBMS 툴 자체적으로 제공되는 ERD 기능을 검토했으나, 파일 기반으로 되어있어 실시간 동기화를 보장하기에 어려움이 있어보였다.
* ERD Cloud 웹 사이트를 이용해본 적이 있으나, 히스토리 관리에 어려움이 있었다. <br>
  private 프로젝트를 만들 수 있으며 인터넷만 연결되면 실시간 동기화로 공유가 가능했다.
* 유료툴은 테스트하지 못했다.
* IntelliJ 내장 다이어그램은 연관관계의 선을 다양하게 사용할 수 없다는 단점이 컸다. (화살표만 제공)
* intelliJ 내에서도 가볍게 돌아갔다면 더 편했을 것 같으나  <br>
  프로젝트 git 을 이용해 버전 관리가 가능하고 프로젝트 파일 내에서 해당 프로젝트 ERD를 한번에 볼 수 있다는 장점이 크게 느껴진다.

