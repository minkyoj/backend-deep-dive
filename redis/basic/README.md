# Redis Deep Dive

Redis는 **In-Memory 기반의 Key-Value 저장소**로,  
빠른 조회 성능을 활용하여 **캐싱, 세션 저장, 실시간 데이터 처리** 등에 사용된다.

---

# 1. Redis란?

> "메모리에 데이터를 저장하여 매우 빠른 속도로 조회 가능한 NoSQL DB"

- Key-Value 구조
- Single Thread 기반 (빠른 처리)
- Disk에도 저장 가능 (RDB, AOF)

---

# 2. 왜 Redis를 사용하는가?

## 2.1 DB 부하 감소
자주 조회되는 데이터를 Redis에 저장하여  
DB 접근을 줄이고 성능을 개선

예)
- 사용자 정보
- 인기 게시글
- 조회수 높은 데이터

---

## 2.2 빠른 응답 속도
- 메모리 기반 → Disk 기반 DB보다 훨씬 빠름
- 평균적으로 ms 단위 응답

---

## 2.3 확장성
- 분산 캐시 구조로 확장 가능
- 대규모 트래픽 처리에 적합

---

# 3. Redis 자료구조

## String
가장 기본적인 타입 (Key-Value)

```bash
SET user:1 "mingyo"
GET user:1

## Hash

객체 형태 저장

```bash
HSET user:1 name "mingyo" age 30
HGET user:1 name

## List

순서가 있는 데이터 (Queue/Stack)

```bash
LPUSH queue "task1"
RPUSH queue "task2"

## Set

```bash
SADD users "a" "b" "c"
SMEMBERS users

## Sorted Set (ZSet)

점수(score) 기반 정렬

```bash
ZADD ranking 100 "user1"
ZADD ranking 200 "user2"
ZRANGE ranking 0 -1 WITHSCORES

# 4. 캐싱 전략
Cache Aside (Lazy Loading)

가장 많이 사용하는 방식

- Redis 조회

- 없으면 DB 조회

- Redis에 저장

Client → Redis → (없으면) DB → Redis 저장 → Client 반환

- Write Through

DB 저장 시 Redis도 같이 저장

데이터 일관성 유지

- Write Back

Redis에 먼저 저장 후 나중에 DB 반영

성능 좋지만 데이터 유실 위험

# 5. Redis 사용 시 주의점
- 5.1 캐시 만료 정책 (TTL)

```bash
SET key value EX 60
  TTL 설정 안 하면 메모리 계속 증가

- 5.2 캐시 스탬피드 (Cache Stampede)

동시에 캐시 미스 발생 → DB 부하 급증

해결: TTL 랜덤값 추가, Lock 활용

- 5.3 데이터 일관성 문제

Redis와 DB 데이터 불일치 가능

전략 선택 중요

# 6. 실무 활용 예시
- 6.1 로그인 세션 관리

사용자 로그인 정보 Redis 저장

빠른 인증 처리

- 6.2 조회수 캐싱

조회수 증가 → Redis

일정 주기로 DB 반영

- 6.3 인기 데이터 캐싱

자주 조회되는 데이터 Redis 저장

# 7. 핵심 질문
Redis를 왜 사용하는가?
→ DB 부하 감소 + 빠른 응답 속도

Redis와 RDB 차이?
→ 메모리 vs 디스크 기반

캐싱 전략 설명
→ Cache Aside / Write Through / Write Back

캐시 일관성 문제 어떻게 해결?
→ TTL, 전략 선택, Lock
