---
layout: post
title:  "Node drain 시 POD 재배치로 인한 서비스 중단"
date:   2025-05-09 11:00:00 +0900
category: [클라우드]
tags: [클라우드,k8s]
lastmod : 2025-05-09 11:00:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---

NKS에서 쿠버네티스 버전을 업데이트 하고자 새 버전의 node를 먼저 생성하고 기존 버전의 노드를 drain 하는 방법을 이용하고 있다. <br>
근데 drain 시 기존 노드의 POD가 새 노드에 배포가 완료되기 전에 삭제되는 바람에 다른 노드에도 서비스가 존재하지 않다면 접속 장애가 발생한다. <br>
<br>

이를 막기 위해 PodDisruptionBudget(PDB) 를 이용하여 POD의 최소 실행 개수를 우선 설정해주어야 한다. <br>
<br>
PDB를 적용하게 되면 drain 시에도 PDB를 가장 최우선으로 충족시키기 때문에 POD가 새로운 node에 재 배포되어 설정한 개수를 충족할 때 까지 기존 node의 drain이 미뤄진다. 

