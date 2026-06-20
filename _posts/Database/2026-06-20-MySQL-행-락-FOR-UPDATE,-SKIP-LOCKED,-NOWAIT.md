---
layout: post
title:  "MySQL 행 락 - FOR UPDATE, SKIP LOCKED, NOWAIT"
date:   2026-06-20 11:00:00 +0900
categories: [Database]
tags: [DB, MySQL, LOCK, 동시성]
lastmod : 2026-06-20 11:00:00 +0900
sitemap :
changefreq : daily
priority : 1.0
---
> MySQL 8.0의 행 단위 잠금 구문(`SELECT ... FOR UPDATE`, `SKIP LOCKED`, `NOWAIT`)을<br> 실제로 테스트해보고 정리한 내용.<br>잡 큐(작업 큐) / 메시지 디스패칭 같은 동시성 시나리오에서 핵심이 되는 기능들.

## 환경

- MySQL 8.0 (Docker)
- 격리수준: 기본값(REPEATABLE READ)에서 진행
- 테스트 방식: 같은 DB에 2개 세션을 열어두고 한쪽이 락을 잡은 상태에서 다른 쪽 동작 비교


---

## 1. SELECT ... FOR UPDATE 란?

**"이 행을 읽으면서 동시에 쓰기 잠금(X-lock)을 건다"** 는 구문.

일반 `SELECT`는 락을 안 잡지만, `FOR UPDATE`는 마치 `UPDATE`처럼 행을 잠가버린다.

### 동작

- 읽어온 행에 **배타 락(eXclusive lock)** 부여
- 다른 트랜잭션은 그 행을 `SELECT ... FOR UPDATE` / `UPDATE` / `DELETE` **못 함** (대기 또는 에러)
- 락 안 거는 일반 `SELECT`는 **이전 스냅샷을 그대로 읽을 수 있음** (MVCC)
- 락은 **COMMIT / ROLLBACK 시점에 해제** → 반드시 트랜잭션 안에서 써야 의미 있음

### 왜 필요한가 — Read-Modify-Write 경쟁 조건 방지

**나쁜 예 (락 없음)**

```sql
-- 워커 A
SELECT status FROM jobs WHERE id = 1;  -- 'pending'
-- 워커 B도 동시에 같은 거 읽음 → 'pending'
UPDATE jobs SET status = 'processing' WHERE id = 1;
-- A, B 둘 다 같은 작업을 처리 (중복 처리)
```

**좋은 예 (FOR UPDATE)**

```sql
START TRANSACTION;
SELECT id FROM jobs WHERE status = 'pending' LIMIT 1 FOR UPDATE;
-- 이 행은 이제 나만 만질 수 있음
UPDATE jobs SET status = 'processing' WHERE id = ?;
COMMIT;
```

두 워커가 동시에 시도해도 한 명은 락이 풀릴 때까지 대기 → 중복 처리 없음.

### `WHERE id = N` vs `LIMIT N` — 락 거는 대상의 차이

같은 `FOR UPDATE`라도 **WHERE 조건의 성격**에 따라 의미가 완전히 달라진다.

- `WHERE id = N` → **이미 알고 있는 특정 행**을 잠금 (타겟 지정형)
- `WHERE 조건 ORDER BY X LIMIT N` → **조건에 맞는 후보 중에서 골라 잠금** (후보 탐색형)

#### 케이스 ① — 알려진 행을 보호 (계좌 이체, 재고 차감)

이때는 id를 이미 알고 있으니 `WHERE id = ?`가 맞음.

```sql
-- 좋은 예: 사용자가 누른 계좌 1번에서 1000원 출금
START TRANSACTION;
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;
UPDATE accounts SET balance = balance - 1000 WHERE id = 1;
COMMIT;
```

#### 케이스 ② — 잡 큐에서 작업 골라잡기

이때는 어떤 id가 처리 대상인지 **미리 모른다**. 조건만 만족 <br>

**나쁜 예 (잡 큐인데 특정 id를 박아둠)**

```sql
-- 워커 5개가 모두 같은 쿼리를 실행
SELECT * FROM jobs WHERE id = 1 FOR UPDATE;
```

→ 모든 워커가 같은 행(id=1)을 노림. 한 명이 잠그면 나머지는 전부 대기 → **결국 직렬 처리됨**. <br>
워커를 늘려도 처리량이 안 늘어남. 잡 큐로서 의미가 없음

**좋은 예 (LIMIT + SKIP LOCKED)**

```sql
SELECT id FROM jobs
WHERE status = 'pending'
ORDER BY id
LIMIT 1
FOR UPDATE SKIP LOCKED;
```

→ 워커마다 **서로 다른 행**을 즉시 가져감.
- 워커1 → id=1 잠금 후 가져감
- 워커2 → id=1 잠겨있으니 건너뜀 → id=2 가져감
- 워커3 → id=1,2 건너뜀 → id=3 가져감

진짜 병렬 처리가 가능해짐

#### 정리

| 패턴 | 용도 | 예시 |
|---|---|---|
| `WHERE id = N FOR UPDATE` | **알려진 특정 행** 보호 | 주문 1234 취소, 계좌 1 출금 |
| `WHERE 조건 ORDER BY X LIMIT N FOR UPDATE [SKIP LOCKED]` | **조건 만족하는 행 골라잡기** | pending 작업 중 오래된 1건 |

> `ORDER BY`가 빠지면 어떤 행이 먼저 잡힐지 보장 안 됨.<br> 잡 큐에서는 `ORDER BY id` 또는 `ORDER BY created_at` 명시.

### 친척 구문 비교

| 구문 | 락 종류 | 의미 |
|---|---|---|
| `SELECT ...` | 없음 | MVCC 스냅샷 읽기 (논블로킹) |
| `SELECT ... FOR SHARE` | S (공유) | 다른 트랜잭션도 읽기는 OK, 쓰기는 막음 |
| `SELECT ... FOR UPDATE` | X (배타) | 다른 트랜잭션은 락 읽기/쓰기 다 막힘 |
| `SELECT ... FOR UPDATE SKIP LOCKED` | X | 잠긴 행은 건너뜀 |
| `SELECT ... FOR UPDATE NOWAIT` | X | 잠긴 행 있으면 즉시 에러 |

> `LOCK IN SHARE MODE`는 옛 문법. MySQL 8.0+ 부터는 `FOR SHARE`를 권장.

### 대표 사용처

1. **잡 큐 / 메시지 디스패칭** — 워커가 다음 작업을 집어갈 때
2. **재고 차감** — 재고 수량 읽고 → 검증 → 차감 사이에 다른 트랜잭션 끼어들지 못하게
3. **계좌 이체** — 출금 계좌 잔액 읽고 → 검증 → 차감
4. **중복 INSERT 방지** — 존재 여부 확인 후 INSERT 사이의 갭을 막음

### 주의점

- **반드시 트랜잭션 안에서** 사용. 자동커밋 상태에서 단독으로 쓰면 즉시 락이 풀려 무의미함.
- **인덱스를 안 타면 풀스캔하면서 행을 다 잠가버림** → 다른 트랜잭션이 다 막힘. `EXPLAIN`으로 확인 필수.
- MySQL 기본 격리수준(REPEATABLE READ)에서는 **갭 락(gap lock)**까지 끼어, 의도치 않은 범위가 잠길 수 있음.
- **데드락** 가능 — 항상 같은 순서로 락을 잡도록 쿼리 패턴을 통일.
- 트랜잭션은 짧게. 락 잡은 채로 외부 API 호출 같은 거 **절대 금지**.

---

## 2. SKIP LOCKED / NOWAIT

`FOR UPDATE`만 쓰면 잠긴 행을 만날 때 무조건 **대기**한다. 이걸 제어하는 두 가지 변형이 8.0에서 추가됨.

| 구문 | 잠긴 행을 만나면 |
|---|---|
| `FOR UPDATE` | 락이 풀릴 때까지 대기 (기본 `innodb_lock_wait_timeout = 50초` 후 ERROR 1205) |
| `FOR UPDATE SKIP LOCKED` | **건너뛰고** 다음 행으로 → 즉시 결과 반환 |
| `FOR UPDATE NOWAIT` | **즉시 ERROR 3572**로 실패 |

### 어떤 상황에 쓰나

- **SKIP LOCKED** — 잡 큐. 워커 N개가 동시에 "다음 작업"을 집어가야 할 때.<br>잠긴 행(=다른 워커가 처리 중)은 건너뛰고 자기 몫만 가져가면 됨.
- **NOWAIT** — "지금 못 잡으면 다음 차례에 다시 시도"하는 짧은 폴링 패턴.<br>또는 사용자에게 "다른 사람이 편집 중입니다"를 즉시 알려야 하는 UI 흐름.

---

# 3. 테스트 — 2개 세션으로 동작 비교

### 세팅 (한 번만)

```sql
CREATE TABLE jobs (
  id      INT PRIMARY KEY AUTO_INCREMENT,
  status  VARCHAR(20) NOT NULL,
  payload VARCHAR(100),
  INDEX idx_status (status)
) ENGINE=InnoDB;

INSERT INTO jobs (status, payload) VALUES
  ('pending','job-1'),
  ('pending','job-2'),
  ('pending','job-3'),
  ('pending','job-4'),
  ('pending','job-5');

SELECT * FROM jobs;
```

### 세션 A — 행 잠금 (COMMIT/ROLLBACK 하지 말고 대기)

```sql
START TRANSACTION;

SELECT * FROM jobs
WHERE status = 'pending'
ORDER BY id
LIMIT 2
FOR UPDATE;
-- 결과: id=1, id=2 잠금. 여기서 멈춰 있기.
```

### 세션 B — 세 가지 동작 비교

#### (1) 기본 동작 — 블로킹

```sql
USE testdb;

-- 대기 시간 짧게 (빨리 확인용)
SET SESSION innodb_lock_wait_timeout = 3;

SELECT * FROM jobs
WHERE status = 'pending'
ORDER BY id
LIMIT 2
FOR UPDATE;
-- 3초 대기 후 DBeaver 오류 메시지
-- SQL Error [1205] [40001]: Lock wait timeout exceeded; try restarting transaction
```

#### (2) SKIP LOCKED — 잠긴 행은 건너뛰고 즉시 반환

```sql
SELECT * FROM jobs
WHERE status = 'pending'
ORDER BY id
LIMIT 2
FOR UPDATE SKIP LOCKED;
-- 예상: id=3, id=4 즉시 반환 (1,2는 건너뜀)
```

#### (3) NOWAIT — 잠긴 행 만나면 즉시 에러

```sql
SELECT * FROM jobs
WHERE status = 'pending'
ORDER BY id
LIMIT 2
FOR UPDATE NOWAIT;
-- 즉시 DBeaver 오류 메시지
-- ESQL Error [3572] [HY000]: Statement aborted because lock(s) could not be acquired immediately and NOWAIT is set.
```


---

## 4. 잠금 상태 확인 — performance_schema.data_locks

세션 C(별도 커넥션)에서 현재 락 상태를 들여다보는 쿼리.

```sql
SELECT
  ENGINE_TRANSACTION_ID AS trx_id,
  THREAD_ID,
  OBJECT_NAME,
  INDEX_NAME,
  LOCK_TYPE,
  LOCK_MODE,
  LOCK_STATUS,
  LOCK_DATA
FROM performance_schema.data_locks
WHERE OBJECT_SCHEMA = 'testdb';
```

### 컬럼별 의미

| 컬럼                               | 의미                                    | 비고                                                                           |
| -------------------------------- | ------------------------------------- | ---------------------------------------------------------------------------- |
| `trx_id` (ENGINE_TRANSACTION_ID) | 락을 가진/대기 중인 InnoDB 트랜잭션 ID            | `information_schema.innodb_trx`로 범인 추적                                       |
| `THREAD_ID`                      | performance_schema 내부 스레드 ID (커넥션 단위) | `performance_schema.threads`로 역추적                                            |
| `OBJECT_NAME`                    | 락이 걸린 **테이블 이름**                      | `OBJECT_SCHEMA`와 합쳐 `db.table` 식별                                            |
| `INDEX_NAME`                     | 락이 걸린 **인덱스 이름**                      | `PRIMARY`=클러스터드, `idx_xxx`=세컨더리. 둘 다 잡히면 같은 trx_id 두 줄.<br> 테이블 락이면 `NULL`   |
| `LOCK_TYPE`                      | 락 레벨                                  | `TABLE`=테이블 락(IX/IS), `RECORD`=행 락                                           |
| `LOCK_MODE`                      | 락 모드                                  | S/X, GAP, REC_NOT_GAP 등 (아래 표 참고)                                            |
| `LOCK_STATUS`                    | 락 상태                                  | `GRANTED`=획득, `WAITING`=대기 → **블로킹 진단의 핵심**                                  |
| `LOCK_DATA`                      | 락 걸린 **실제 키 값**                       | PRIMARY면 PK, 세컨더리면 `'키', PK`. `supremum pseudo-record`=마지막 갭. 테이블 락이면 `NULL` |

#### `LOCK_MODE` 세부 값

| 값 | 의미 |
|---|---|
| `IS` / `IX` | Intention Shared / Exclusive — "이 테이블의 어떤 행에 락 걸 예정"이라는 의도 표시 (테이블 단위) |
| `S` / `X` | Shared / Exclusive 행 락 |
| `X,REC_NOT_GAP` | 레코드 락만 (해당 행만, 갭은 안 잠금) |
| `X,GAP` | 갭 락만 (행 사이 빈 공간, 팬텀 방지) |
| `X` (수식어 없음) | next-key 락 (레코드 + 갭, REPEATABLE READ 기본) |
| `X,INSERT_INTENTION` | INSERT가 자리 잡으려고 대기 중 |

### 테스트 결과 

세션 A가 `LIMIT 2 FOR UPDATE`로 id=1,2를 잡은 상태

여기서 세션 B가 **기본 동작**(블로킹)으로 같은 쿼리를 실행하면:
- 새로운 `trx_id`가 추가되고
- `LOCK_STATUS = WAITING` 인 행이 나타남 → **이게 지금 막혀 있는 락**
![](/assets/img/Database/2026-06-20-img-db-mysql-lock/waiting.png) <br/>

세션 B가 **SKIP LOCKED**로 실행하면:
- 대기 없이 id=3,4를 즉시 잡으므로 `WAITING` 행은 안 생기고 새 `GRANTED` 락만 추가됨
![](/assets/img/Database/2026-06-20-img-db-mysql-lock/skip-lock.png) <br/>


---

## 5. 잡 큐 패턴 실전 팁

```sql
-- 워커 트랜잭션 (짧게 유지)
START TRANSACTION;

SELECT id
FROM jobs
WHERE status = 'pending'
ORDER BY id
LIMIT 50
FOR UPDATE SKIP LOCKED;

UPDATE jobs
SET status = 'processing', worker_id = ?, claimed_at = NOW()
WHERE id IN (...);

COMMIT;
```

- `SKIP LOCKED` + `LIMIT N` 으로 배치 클레임 → 워커마다 서로 다른 행을 동시에 가져감
- 인덱스를 잘 타도록 `(status, id)` 복합 인덱스를 고려
- 락을 잡은 트랜잭션 안에서 **무거운 처리/외부 API 호출 금지** — 클레임만 짧게 하고 락 푼 뒤 본 처리
- 워커가 `processing` 상태로 잡아두고 죽으면 영원히 멈춤 → **타임아웃 청소 잡** 필요

```sql
-- Stuck row 복구 (주기적 실행)
UPDATE jobs
SET status = 'pending', worker_id = NULL
WHERE status = 'processing'
  AND claimed_at < NOW() - INTERVAL 5 MINUTE;
```

> Redis 락의 TTL과 같은 역할 — DB 락에서는 직접 만들어 줘야 한다.

---

## 6. 정리

- `SELECT ... FOR UPDATE` = 읽으면서 X-lock. 트랜잭션 끝날 때까지 보유.
- `SKIP LOCKED` = 잠긴 행 건너뛰고 다음 행으로. 잡 큐의 핵심.
- `NOWAIT` = 잠긴 행 만나면 즉시 에러. 짧은 폴링/사용자 즉시 피드백용.
- 인덱스 잘 타도록 설계해야 갭 락/풀스캔 락의 부작용을 피할 수 있음.
- `performance_schema.data_locks` 로 락 상태와 블로킹 관계를 가시화할 수 있음.

---

## 7. 분산락(Redis / ZooKeeper / Netty)과 비교

> 한 줄 요약: **"보호하려는 게 DB row면 DB 락, 그 외 조율 문제면 Redis / ZK"**

`SKIP LOCKED`의 등장으로 그동안 외부 분산락에 의존해왔던 많은 시나리오가 DB만으로 해결 가능해졌다.<br>다만 모든 분산락 용도를 대체하는 건 아니다.

### DB 락(FOR UPDATE + SKIP LOCKED)으로 충분한 영역

| 시나리오                   | 이유                                                                                                     |
| ---------------------- |--------------------------------------------------------------------------------------------------------|
| **잡 큐 / 메시지 디스패칭**     | SKIP LOCKED 이후 Sidekiq, GoodJob(Rails), River(Go), pgmq, Oban(Elixir) 등<br>신생 큐가 Redis 없이 DB 기반으로 회귀 중 |
| **재고 차감 / 계좌 이체**      | 보호 대상이 DB row 자체.<br>락과 데이터 변경이 **같은 트랜잭션**에서 일어나야 정합성이 깨지지 않음                                         |
| **중복 INSERT 방지**       | UNIQUE 인덱스 + `FOR UPDATE` 로 갭까지 충분히 막을 수 있음                                                            |
| **수천 TPS 이하 락 경합**     | InnoDB 행 락은 충분히 빠름. 굳이 외부 도입할 이유 없음                                                                    |
| **단일 서비스 / 단일 DB 도메인** | 락만 외부로 빼면 "락 잡고 DB 커밋 직전에 죽음" 같은 two-phase 위험만 늘어남                                                     |

> 핵심 원리: **락이 보호하려는 데이터가 DB 안에 있다면, 락도 DB 안에 둬야 정합성이 자연스럽게 보장된다.**

### 그래도 Redis / ZooKeeper / Netty가 필요한 영역

| 시나리오 | DB 락으로 안 되는 이유 | 적합한 도구 |
|---|---|---|
| **DB와 무관한 리소스 보호** (외부 API rate limit, 파일 처리, 이메일 발송) | 락 때문에 DB 트랜잭션을 여는 게 부자연스럽고 DB 부하 증가 | Redis 락 |
| **리더 선출** (N개 인스턴스 중 1개만 스케줄러 실행) | DB 락은 heartbeat / 세션 만료 처리가 약함 | ZooKeeper ephemeral node, Redis + TTL |
| **초고처리량 락** (10K+ TPS) | DB가 락 처리만으로 CPU 소진 | Redis (인메모리) |
| **TTL 기반 자동 해제 / 임대(lease)** | DB는 만료 처리 로직을 직접 구현해야 함 (청소 잡 필요) | Redis `SET NX EX` |
| **마이크로서비스 간 조율** (서로 다른 DB 쓰는 서비스들) | 공유할 DB가 없음 | Redis, ZK |
| **서비스 디스커버리 / 컨피그 동기화 / 분산 합의** | DB 영역 밖의 문제 | ZK, etcd, Consul |
| **채널 / 세션 단위 동시성** (WebSocket, 게임 서버) | 요청-응답 모델이 아니라 장시간 연결 단위 동기화 필요 | Netty 자체 동기화 (channel pipeline) |

### 도구별 특성 비교

| 항목 | DB 락 (SKIP LOCKED) | Redis 락 | ZooKeeper |
|---|---|---|---|
| 락 단위 | DB row | 임의의 키 | znode 경로 |
| 속도 | 중 (수 ms) | 빠름 (sub-ms) | 중 (네트워크 RTT) |
| 정합성 | 강함 (트랜잭션과 동일) | 약함 (Redlock 논쟁) | 강함 (CP, ZAB 합의) |
| 워커 죽었을 때 | 트랜잭션 자동 롤백 → 락 자동 해제 | TTL 만료까지 점유 | ephemeral node 자동 삭제 |
| 운영 부담 | DB 1대로 끝 | Redis 클러스터 별도 | ZK 앙상블(3~5대) 별도 |
| 처리량 한계 | 수천 TPS | 수만 TPS | 수천 TPS |
| 데이터 정합성 보장 | 락 + 데이터 변경이 같은 TX | 별도 (애플리케이션 책임) | 별도 (애플리케이션 책임) |

### 의사결정 흐름

```
보호하려는 대상이 DB row 인가?
├── YES → DB 락 (FOR UPDATE / SKIP LOCKED)
│
└── NO
    ├── 정합성 중요 + 리더 선출 / 분산 합의   → ZooKeeper (또는 etcd)
    ├── 속도 / 단순성 우선 + 짧은 임대 락     → Redis 락
    └── 채널 / 세션 단위 장기 연결 동기화     → Netty 내부 동기화
```


## 참고

- [MySQL 8.0 — Locking Reads](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html){:target="_blank"}
- [performance_schema.data_locks Table](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-data-locks-table.html){:target="_blank"}
- [InnoDB Locking](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html){:target="_blank"}
