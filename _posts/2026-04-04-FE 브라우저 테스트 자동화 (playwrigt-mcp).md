---
layout: post
title:  "FE 브라우저 테스트 자동화 (playwrigt-mcp)"
date:   2026-04-04 11:00:00 +0900
category: [오픈소스]
tags: [AI]
lastmod : 2026-04-04 11:00:00 +0900
changefreq : daily
priority : 1.0
---

## FE 브라우저 테스트
기존에 프론트에서 테스트 할 때 직접 브라우저를 열어 테스트 하는 방법과 <br>
좀 더 자동화하여 셀레니움 또는 playwright 같은 오픈소스 도구를 이용했다면 AI MCP를 이용해 자동화 할 수 있는 도구가 있어 소개한다. <br>
(E2E 테스트 도구 중 playwright를 이번에 처음 알았다ㅎㅎ) <br><br>


git : [https://github.com/microsoft/playwright-mcp](https://github.com/microsoft/playwright-mcp){:target="_blank"}

## MCP 연결
위 git 페이지의 README.md 에 친절하게 설명 되어 있다. <br>
현재 나는 claude를 이용하기에 Claude Code 영역을 열어 나와있는 명령어를 이용해 설정했다. <br><br>

```ssh
claude mcp add playwright npx @playwright/mcp@latest
```

*위 명령어로 실행하면 현재 프로젝트에서만 playwright가 추가 됨* <br>
*전역적으로(계정에) 기능을 추가하고 싶다면 `--scope user` 옵션 추가* <br><br>


## 자연어로 테스트

이제 자연어로 AI 를 통해 명령하면 브라우저 테스트가 진행된다. <br>
![](/assets/img/2026-04-04-img-playwright-mcp/playwright-mcp-cmd.png)<br/>

예시로 만든 프로젝트를 위와 같이 허술하게 테스트 요청만 해도 <br> <br>

![](/assets/img/2026-04-04-img-playwright-mcp/playwright-mcp-answer.png)<br/>

위처럼 테스트를 알아서 분석해서 진행하고 결과를 보고해준다. <br><br>

![](/assets/img/2026-04-04-img-playwright-mcp/playwright-mcp-screenshot.png)<br/>

테스트시 이미지도 같이 저장해준다. <br>
가끔 안해줄때도 있어서 단계마다 스크린샷도 같이 저장하라고 명령하면 편하다. <br><br>


## AS-IS

그렇다면 기존 Playwright는 어떻게 사용해서 테스트를 자동화 했을까 <br>

![](/assets/img/2026-04-04-img-playwright-mcp/playwright-code.png)<br/>
테스트 케이스를 직접 코딩하여 작성한다. <br>
*이것도 클로드 시켰다* <br><br>

![](/assets/img/2026-04-04-img-playwright-mcp/playwright-ui.png)<br/>
UI모드로 테스트를 실행하면 위와 같은 UI로 테스트케이스마다 확인이 가능하며 <br>


![](/assets/img/2026-04-04-img-playwright-mcp/playwright-report.png)<br/>
리포트 모드를 명령하면 localhost:9323 를 통해 리포트를 조회할 수 있다. <br><br>


이또한 클로드를 이용해 직접 코딩하지 않아도 되긴 하니, <br>
둘 중 상황에 알맞는 방법을 이용하면 좋을 것 같다. <br>
(테스트케이스 공유가 필요 할 경우 테스트 코드 파일로 공유해야만 한다던지.. 등등)
