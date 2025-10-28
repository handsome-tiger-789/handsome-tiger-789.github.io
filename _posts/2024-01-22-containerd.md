---
layout: post
title:  "containerd"
date:   2024-01-22 11:00:00 +0900
category: [클라우드]
tags: [클라우드, container, containerd]
lastmod : 2024-01-22 11:00:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---

crictl은 Kubernetes의 Container Runtime Interface (CRI)를 위한 커맨드 라인 도구 중 하나입니다. <br>
(기존엔 kubernetes에서 사용하던 기본 컨테이너는 docker였으나 docker가 표준을 지키지 않는 문제로 인해 특정 버전부터 containerd로 변경되었습니다.)  <br>
crictl은 컨테이너와 관련된 다양한 작업을 수행하는 데 사용됩니다. <br>

몇 가지 주요 crictl 명령어는 다음과 같습니다

1. **`crictl ps`**: 현재 실행 중인 컨테이너 목록을 표시합니다.
2. **`crictl images`**: 시스템에 설치된 이미지 목록을 표시합니다.
3. **`crictl pull`**: 컨테이너 이미지를 가져옵니다.
4. **`crictl create`**: 새로운 컨테이너를 생성합니다.
5. **`crictl run`**: 새로운 컨테이너를 생성하고 실행합니다.
6. **`crictl logs`**: 컨테이너의 로그를 표시합니다.
7. **`crictl exec`**: 실행 중인 컨테이너 내에서 명령을 실행합니다.
8. **`crictl inspect`**: 컨테이너나 이미지의 세부 정보를 표시합니다.


<br> 
이러한 명령어는 Kubernetes 클러스터에서 CRI를 통해 컨테이너를 관리하는 데 사용됩니다. <br>
crictl은 일반적으로 Kubernetes 노드에서 직접 사용되며, 관련된 CRI 구현(예: containerd, cri-o)에 따라 동작합니다. <br>
<br>

불필요한 이미지 삭제 방법 : `crictl rmi --prune`

<br>

>쿠버네티스에서 사용되는 컨테이너 툴로는 containerd를 사용하고 있기 때문에 <br>
>쿠버네티스가 노드에 registry image를 빌드할 경우 `docker images` 로는 조회되지 않으며, <br>
>`crictl images` 를 통해 조회가 가능합니다. <br>
>따라서 자동으로 빌드된 이미지의 리소스 용량을 삭제하기 위해서는 `crictl rmi --prune` 명령어를 cron으로 등록해두어야 합니다. <br>


