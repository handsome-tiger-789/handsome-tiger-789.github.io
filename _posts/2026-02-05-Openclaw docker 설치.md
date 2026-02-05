---
layout: post
title:  "Openclaw docker 설치"
date:   2026-02-05 11:00:00 +0900
category: [오픈소스]
tags: [오픈소스, AI, docker]
lastmod : 2026-02-05 11:00:00 +0900
sitemap :
changefreq : daily
priority : 1.0
---

최근 핫한 openclaw 를 찍먹해보기 위해 안전한 Docker 환경에 설치해보자. <br>
<br>

## 설치환경
* Mac (M3)
* Docker
* Gemini (무료계정)
* Discord 연동

<br>
혹시 몰라 구글계정, 디스코드 계정 모두 새로 생성했다. <br>
<br>

## Google API Key 발급
gemini 무료 계정의 API 키를 연동하기 위해 [Google AI Studio](https://aistudio.google.com/){:target="_blank"}에 들어가서 API 키를 발급받는다. <br>
![](/assets/img/2026-02-05-img-openclawbot/gemini-key.png) <br/>

<br>
화면 좌측 아래 **Get API Key** 메뉴 진입 <br>
<br>

![](/assets/img/2026-02-05-img-openclawbot/create-gemini-api-key.png) <br/>

우측의 **API 키 만들기** 버튼을 클릭하여 새로운 키를 하나 생성한다. <br>
<br>
키 목록에서 새로 만든 키를 선택하여 **API 키** 를 복사. <br>
<br>


## Discord 봇 추가

우선 봇을 추가하고자 하는 서버와 채널을 생성해둔다 <br>
<br>
discord developer token 발급 <br>
[https://discord.com/developers/applications](https://discord.com/developers/applications){:target="_blank"}

<br><br>

**New Application** 버튼을 눌러 새로 하나 만든다. <br> 
좌측 메뉴 중 Bot 을 누른 뒤 Reset Token  버튼을 눌러 새로운 토큰을 발급 받는다. <br>
<br>

![](/assets/img/2026-02-05-img-openclawbot/bot-intent.png) <br/>

이 권한들을 설정해주지 않아 여러번 재설치 했다.. <br>
Presence Intent 는 선택이라고 하는데 머리 아파서 그냥 허용했다. <br>
(어차피 로컬로 테스트만 할거니까) <br>
<br>
다시 좌측 **OAuth2** 메뉴를 눌러 아래 과정대로  수행 <br> 
**OAuth2 URL Generator** > **Scopes** > bot 체크 <br>
**Bot Permissions** > Text Permissios 중 Send Messages / Read Message History / Use Slash Commands 선택 <br>

<br>
그러고 나면 맨 밑에 Generated URL 에 나온 URL을 브라우저에 접속하여 Discord 에서 봇을 추가할 서버를 선택할 수 있다. <br>
*이미지는 최종 봇과 명칭이 다름 여러번 재시도 했기 때문* <br>

![](/assets/img/2026-02-05-img-openclawbot/discord-add-bot1.png) <br/>

<br>

![](/assets/img/2026-02-05-img-openclawbot/discord-add-bot2.png) <br/>

<br>

![](/assets/img/2026-02-05-img-openclawbot/discord-add-bot3.png) <br/>

<br>

## Openclaw 설치

도커 데스크탑은 이미 설정된 상태로 openclaw를 설치하는 과정만 서술한다. <br>
<br>


```ssh
git clone https://github.com/openclaw/openclaw.git
```

<br>
소스 경로를 지정하여 위 명령어를 통해 openclaw 소스를 clone 해온다. (git 명령어 없을 경우 웹에서 다운 가능) <br>
다운이 완료되면 openclaw 폴더로 다운된 파일 확인이 가능하다. <br>
<br>

다시 git clone 한 openclaw 파일 목록으로 돌아온다. <br>
<br>

```ssh
# .env.example 파일을 .env 파일로 복사
cp .env.example .env
```

<br>
.env 파일 아래와 같이 수정 <br> 

```ssh
# --- set key
GOOGLE_API_KEY=구글 API Key
DISCORD_BOT_TOKEN=디스코드 봇 토큰
# --- set key
OPENCLAW_CONFIG_DIR=
OPENCLAW_WORKSPACE_DIR=
OPENCLAW_GATEWAY_PORT=18789
OPENCLAW_BRIDGE_PORT=18790
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_TOKEN=
OPENCLAW_IMAGE=openclaw:local
OPENCLAW_EXTRA_MOUNTS=
OPENCLAW_HOME_VOLUME=
OPENCLAW_DOCKER_APT_PACKAGES=
# --- add setting
GOOGLE_MODEL=gemini-2.5-flash
OPENCLAW_GATEWAY_MODE=local
OPENCLAW_GATEWAY_CONTROL_UI_ALLOW_INSECURE_AUTH=true
```


최종적인 .env 파일에서 추출했기 때문에 중간에 있는 설정은 내가 따로 손본건 없다.
* GOOGLE_API_KEY 
* DISCORD_BOT_TOKEN
* GOOGLE_MODEL
* OPENCLAW_GATEWAY_MODE
* OPENCLAW_GATEWAY_CONTROL_UI_ALLOW_INSECURE_AUTH

<br>
가운데 있는 값들은 docker-setup 시 알아서 추가 됐거나 기본적으로 있던 내용이므로 위에 키 값들만 추가하고 설정해주면 된다. <br>
<br>

.env 파일 수정을 마쳤다면 본격적으로 openclaw 컨테이너를 구성해보자. <br>


```ssh
./docker-setup.sh
```


![](/assets/img/2026-02-05-img-openclawbot/setup1.png) <br/>

Yes 선택 <br>
<br>

![](/assets/img/2026-02-05-img-openclawbot/setup2.png) <br/>

QuickStart  모드로 선택 후 Google 모델로 선택 <br>
<br>

![](/assets/img/2026-02-05-img-openclawbot/setup3.png) <br/>

여기서 아까 발급한 Google AI Key 다시 입력 <br>
Default model은 꼭 **google/gemini-2.5-flash** 로 한다. <br>
*당연히 어떤 계정을 쓰느냐에 따라 다르겠지만 무료 계정에선 이걸로 해야 정상 작동함*  <br>
<br>


![](/assets/img/2026-02-05-img-openclawbot/setup4.png) <br/>

연결 하고자 하는 채널 <br>
Discord 선택 <br>

![](/assets/img/2026-02-05-img-openclawbot/setup5.png) <br/>

아까 발급받은 Discord Token 입력 <br>
<br>

![](/assets/img/2026-02-05-img-openclawbot/setup6.png) <br/>


토큰 입력 후 순서대로 Yes, Allowlist 선택 한 뒤 봇이 추가된 서버명과 채널명을 집어넣는다 <br>
ex) clawtestserver/#clawbot <br>
<br>

![](/assets/img/2026-02-05-img-openclawbot/setup7.png) <br/>

입력 후 나오는 Configure skills now 는 봇의 스킬을 추가하는 옵션인데 나중에 추가 가능하므로 No 로 진행 <br>
<br>


![](/assets/img/2026-02-05-img-openclawbot/setup8.png) <br/>

여기까지 나오고 시간이 좀 걸린다 <br>
<br>

![](/assets/img/2026-02-05-img-openclawbot/setup9.png) <br/>


마지막으로 zsh 관련해서 선택옵션이 나오는데 난 docker에 띄워서 터미널은 안쓰기 때문에 No로 진행 <br>
작업이 완료되면 WebUI 링크가 토큰과 함께 나온다 <br>
*접속은 ?token=... 까지 있는 링크로 해야한다.*  <br>
<br>


우선, WebUI 와 Discord에 연결된건 별개이기 때문에 <br>
컨테이너에서 오류가 나지 않는다면 하나씩 확인하면 된다. <br>

<br>

![](/assets/img/2026-02-05-img-openclawbot/discord-claw.png) <br/>

참고로 openclaw 서버를 꺼도 봇은 온라인 상태로 나온다. <br>
봇 자체는 discord에서 추가한 봇이라 그런듯, 응답은 없음 <br>
<br>

![](/assets/img/2026-02-05-img-openclawbot/openclaw-webui.png) <br/>

<br>

## WebUI 접속 토큰을 잃어버렸을 경우
컨테이너 내부에서 cat /home/node/.openclaw/openclaw.json 명령어를 통해 <br> 
gateway 블록 안에 auth > token 부분에서 다시 찾을 수 있다. <br>

<br>

파일경로가 다르다면 docker-compose.yml 설정의 경로를 확인. <br>