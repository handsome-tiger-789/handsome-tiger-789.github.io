---
layout: post
title:  "Local PC 에서 Trivy 점검 (feat.DefectDojo)"
date:   2026-05-24 11:00:00 +0900
category: [etc]
tags: [trivy]
lastmod : 2026-05-24 11:00:00 +0900
changefreq : daily
priority : 1.0
---

## 스캐너 목록

`--scanner` 옵션

| 스캐너       | 검사항목                                | JAR 적용성  |
| --------- | ----------------------------------- | -------- |
| vuln      | CVE 취약점                             | ✅        |
| secret    | 하드코딩된 API 키/토큰/패스워드                 | ✅        |
| license   | OSS 라이센스 (GPL 등 위험 라이센스)            | ✅        |
| misconfig | IaC 설정 오류(Dockerfile/k8s/Terraform) | ❌(소스트리용) |

## CVE 점검

```ssh
 trivy rootfs \
    --scanners vuln \
    --severity HIGH,CRITICAL \
    --ignore-unfixed \
    --format template \
    --template "@/opt/homebrew/share/trivy/templates/html.tpl" \
    --output trivy-report.html \
    build/libs/traffic-queue-demo-be-0.0.1-SNAPSHOT.jar
    

trivy rootfs --scanners vuln \
	--severity HIGH,CRITICAL \
    --ignore-unfixed \
    --format json --output vuln.json \
    build/libs/traffic-queue-demo-be-0.0.1-SNAPSHOT.jar
```

`rootfs` 를 써야 `jar` 아카이브를 풀어가며 점검이 이루어짐<br>
html.tpl 형태로 추출하면 보기 좋게 표로 나옴<br>
<br>
점검 전 최신 CVE 항목으로 DB 업데이트 확인이 필요함<br><br>


## secret 점검

```ssh
trivy fs --scanners secret --format json --output secret.json ./src 
```
✅ secret은 기본 html tpl 에 포맷이 존재하지 않기 때문에 html 로 추출 시 내용이 나오지 않음
- 커스텀 tpl 작성
- txt or json 포맷으로 추출 > 너무 많이 출력되면 콘솔에서 조회하기 힘들기 때문에


## misconfig 점검
```
trivy fs --scanners misconfig --format json --output misconfig.json ./
```


## license 점검
```
trivy rootfs \
	--scanners license \
	--license-full \
	--format json \
	--output license.json \
	build/libs/traffic-queue-demo-be-0.0.1-SNAPSHOT.jar

trivy fs \
    --scanners license \
    --license-full \
    --format json \
    --output license.json \
    ./
```
`--license-full` : 의존성 라이선스까지 깊이 분석 (필수)
<br><br>

## Json To Report
> 일부 파일은 html 포맷으로 추출이 어려워 <br>json 파일로 추출 후 리포트 형태로 보고자 진행

<br>[trivy 공식 블로그](https://trivy.dev/docs/latest/ecosystem/reporting/ )에 trivy 추출물을 Reporting 할 수 있는 오픈 소스 목록 제공<br>
이 중 [DefectDojo](https://github.com/DefectDojo/django-DefectDojo) 오픈소스가 가장 star 가 많음<br>


## DefectDojo
도커로 README 대로 로컬에서 띄우고 프로젝트를 추가해 봄<br>

![](/assets/img/2026-05-24-img-local-pc-trivy/defectdojo-01.png)<br/>

trivy를 이용해 추출한 json 파일을 **Trivy Scan** 항목으로 선택하여 올리면 위와 같은 리포트 화면이 나옴<br>
여기서 항목별 메모, 처리 현황, 취약점 추가 등 다양한 기능이 제공됨<br>

## CWE (SAST) 점검 불가
trivy는 CWE (SAST) 점검은 제공되지 않기 때문에 SonarQube 같은 다른 시스템을 이용한 점검 필요<br>
소스코드(SAST) 점검은 불가능 하고 일부 Secret 키 존재 여부 등만 가능

## TODO:

```
 보너스: SBOM 생성

  스캐너는 아니지만 컴플라이언스/공급망 관리에 자주 같이 씁니다.
  # CycloneDX 형식
  trivy rootfs --format cyclonedx --output sbom.json \
    build/libs/traffic-queue-demo-be-0.0.1-SNAPSHOT.jar

  # SPDX 형식
  trivy rootfs --format spdx-json --output sbom.spdx.json \
    build/libs/traffic-queue-demo-be-0.0.1-SNAPSHOT.jar

  ---
  운영 관점에서는 vuln,secret,license 3종을 CI에 묶어 두는 것이 일반적입니다. misconfig는 인프라 리포지토리에 따로
  거는 편입니다.

```
