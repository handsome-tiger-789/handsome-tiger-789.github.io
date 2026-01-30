---
layout: post
title:  "Gemini CLI 사용기"
date:   2026-01-30 11:00:00 +0900
category: [etc]
tags: [AI]
lastmod : 2026-01-30 11:00:00 +0900
sitemap :
changefreq : daily
priority : 1.0
---

Gemini CLI 는 무료 구간 사용이 가능하기에 설치해보았다. <br>
mac 에서 터미널에 `npm install -g @google/gemini-cli` 명령어를 통해 간단하게 설치 <br>
<br> 

설치 완료 후 `gemini --version` 을 통해 설치된 버전 확인 가능 <br>
<br>

설치가 정상적으로 됐다면 `gemini` 를 입력해  호출하면 로그인 수단을 선택할 수 있다. <br>
브라우저를 통해 계정으로 로그인 해준 뒤 콘솔에 나온대로 재시작을 해주면 이후부터는 아래와 같이 호출이 가능하다. <br>

<br>

![](/assets/img/2026-01-30-img-gemini/gemini-cli.png) <br/>



무료버전에서 이전 claude 와 비슷하게 코드를 명령해보았다. <br>
Member 테이블을 간단하게 구성해준 뒤 jpa 를 이용해 조회해오는 RestAPI 코드를 요청했는데 매우 엉망이었다. <br>
<br>

![](/assets/img/2026-01-30-img-gemini/gemini-code-1.png) <br/>

패키지 구조부터 마음에 들지 않았음 <br>
controller - service - repository  구분은 되어 있으나 패키지 명이 최소한 service 패키지가 application 은 좀 아닌 것 같았으며, <br>
만들어준 Repository 인터페이스에는 어노테이션도 누락되어 있어서 아래와 같이 참조하는 Service 에선 오류를 확인 할 수 있었다. <br>

<br>

![](/assets/img/2026-01-30-img-gemini/gemini-code-2.png) <br/>


이에 다시한번 수정을 명령한 결과 <br>

![](/assets/img/2026-01-30-img-gemini/gemini-update-cli.png) <br/>


![](/assets/img/2026-01-30-img-gemini/gemini-error-package.png) <br/>

패키지 구조는 보기 좋게 정리 되었으나 <br>



![](/assets/img/2026-01-30-img-gemini/gemini-error-class.png) <br/>

스스로 만들었던 DataLoader 클래스에서는 변경된 repository 패키지를 수정해주지 않아 여전히 오류가 나는 걸 확인 할 수 있었다. <br>
<br>

![](/assets/img/2026-01-30-img-gemini/gemini-limit-max.png) <br/>
무료 사용량에 도달하면 이렇게 선택지를 주는 듯 하다. <br><br>


![](/assets/img/2026-01-30-img-gemini/gemini-final-package.png) <br/>


<br>


명령을 구체적으로 하면 그래도 파일을 바로 작성해서 코드를 짜주기는 하지만 <br> 
claude 는 뭔가 패키지 구조부터 좀 더 알아보기 쉽게 한번에 작업을 해줬던 것 같은데 <br>
패키지명이나 다른 클래스와의 검토 같은 부분이 좀 부족하다는 느낌을 많이 받았다. <br>
<br>
무료로 그래도 이정도 쓸 수 있다는거에 감사하며.. <br>
<br>
아, 무료 버전에서는 코드 보안을 위해 학습을 못하도록 설정을 따로 만져줘야 하는 듯 하다. <br>
**확인 필수**


git : [https://github.com/handsome-tiger-789/gemini-free-demo](https://github.com/handsome-tiger-789/gemini-free-demo){:target="_blank"}
