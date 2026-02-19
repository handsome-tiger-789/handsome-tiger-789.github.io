---
layout: post
title:  "Squoosh & slimg"
date:   2026-02-19 11:00:00 +0900
category: [오픈소스]
tags: [오픈소스]
lastmod : 2026-02-19 11:00:00 +0900
sitemap :
changefreq : daily
priority : 1.0
---


얼마 전 설치한 openclaw를 통해 Geeknews 사이트 브리핑을 정리해서 받고있다. <br>
보던 중 흥미로운 내용이 있어 잊지 않기 위해 개념만 작성하려 한다. <br> <br>

![](/assets/img/2026-02-19-img-squoosh-slimg/Squoosh-openclaw.png)<br/>


기사 출처 : [https://news.hada.io/topic?id=26773](https://news.hada.io/topic?id=26773){:target="_blank"} 
<br>

54GB 이미지를 8GB로 줄인다는 문구가 너무 충격적이라 안볼수가 없었다.. <br><br>

뭐 내용은 말 그래도 이미지 파일을 획기적으로 압축, 화질 저하도 육안으로 볼때 거의 없음 <br>
그럼 어떻게 사용할 수 있느냐 <br><br>

개념만 이해하자. <br> <br>

우선 기사에 나온 Squoosh 부터 <br>
React에 Squoosh 라이브러리를 적용하여 이미지 최적화 코드를 넣어주면, <br>
사용자가 자신의 브라우저에 이미지를 업로드 했을때 그 이미지가 브라우저 안에서 압축 된다. <br> 
이후 Backend 서버에 이미지를 보낼때 압축된 사이즈의 이미지를 전송할 수 있게 되므로 네트워크적으로, 서버 측면에서도 이점이 있다. 

```TEXT
사용자 PC (브라우저)
├── 원본 이미지 선택 (10MB)
├── Canvas에서 압축 (2MB) 
└── 압축된 Blob만 서버로 전송
    (원본은 브라우저 메모리에만 남음)
```

<br>

Git : [https://github.com/GoogleChromeLabs/squoosh](https://github.com/GoogleChromeLabs/squoosh){:target="_blank"}

<br>
금일 기준 업데이트가 2년 전이다.. <br>
기사 내용 중 구글의 방치 상태 때문에 아래 나오는 slimg을 만들게 된거다.. <br>
긴 업데이트 부재로 인해 이슈도 많아 보임 <br><br>


이제 slimg을 알아보자 

```TEXT
Kotlin + slimg (서버 압축) 사용자PC ── 10MB 원본 ──► 서버 ── slimg 압축 ── 2MB 저장 
                                ↑ 원본 전송 발생 ↑
```

<br>
slimg은 기본적으로 서버에서 이미지를 압축하기 때문에 Squoosh 와 달리 원본을 전송하게 된다. <br>
그로인한 네트워크 부하와 서버 부하는 고려할 수 밖에 없으나, <br>
현 시점 Squoosh 보다 압축률이 좀 더 나으며 현 시점에선 Squoosh 보다 유지보수가 잘 되고 서버단에서의 작업이므로 오류가 더 적지 않을까 싶다. <br> <br>

Git : [https://github.com/clroot/slimg](https://github.com/clroot/slimg){:target="_blank"}

<br>
솔루션의 트래픽을 고려해서 선택해야할 듯 싶다. <br>
압축률은 디게 좋네..

