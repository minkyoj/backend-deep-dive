# Transaction Deep Dive

트랜잭션(Transaction)은 데이터베이스에서 **데이터의 일관성과 무결성을 보장하기 위한 최소 작업 단위**입니다.  
본 문서는 ACID, Isolation Level, Lock, 트랜잭션 발생 시나리오 등을 실무·면접 관점에서 정리합니다.

---

# 1. 트랜잭션(Transaction)이란?

> **"하나의 논리적 작업 단위 (Logical Unit of Work)"**

예)
- 통행료 결제 요청  
- 충전 취소 처리  
- 회원 탈퇴 시 개인정보 삭제  

이러한 작업은 일부만 성공하면 안 되기 때문에 **모두 성공 or 모두 실패**해야 한다.

---

# 2. 트랜잭션의 ACID 특징

### 2.1 Atomicity (원자성)
- 트랜잭션의 모든 작업은 **하나의 단위로 처리**
- 일부만 실행되는 상황은 존재하지 않음  
→ 실패하면 모두 ROLLBACK

실무 예)
- 결제 API 오류 발생 시 결제 이력, 카드 승인 로그 모두 롤백

---

### 2.2 Consistency (일관성)
- 트랜잭션이 실행되기 전/후 DB의 **무결성 제약 조건을 지켜야 한다.**

예)
- 통행료 금액은 음수가 될 수 없음  
- 유효하지 않은 사용자 ID가 리턴되면 안됨

---

### 2.3 Isolation (고립성)
- 동시에 실행되는 트랜잭션이 서로 영향을 주지 않도록 격리

격리 수준은 아래에서 상세히 설명

---

### 2.4 Durability (지속성)
- 트랜잭션이 커밋되면 결과는 **영구적으로 저장**
- DB 장애/서버 재시작 후에도 유지

---

# 트랜잭션 격리 수준 (Isolation Level)

격리 수준은 **동시성(성능)**과 **정합성(안정성)** 사이의 Trade-Off.

| 레벨 | Dirty Read | Non-repeatable Read | Phantom Read |
|------|------------|----------------------|--------------|
| READ UNCOMMITTED | O | O | O |
| READ COMMITTED (Oracle 기본) | X | O | O |
| REPEATABLE READ (MySQL 기본) | X | X | O |
| SERIALIZABLE | X | X | X |

---

## 3.1 READ UNCOMMITTED
- 커밋되지 않은 데이터를 읽을 수 있음
- 가장 낮은 수준
- 실제 서비스에서는 거의 사용 X

문제: Dirty Read 발생  
→ 결제 취소가 롤백되었는데도 취소됐다고 읽어버리는 상황

---

## 3.2 READ COMMITTED (기본)
- 커밋된 데이터만 읽을 수 있음
- Oracle 기본 레벨
- 서비스 대부분에서 사용

발생 가능한 문제:
- Non-repeatable Read

예)
1) A 트랜잭션: 차량 정보 조회  
2) B 트랜잭션: 차량 정보 변경 후 커밋  
3) A가 같은 쿼리 다시 조회하면 값이 달라짐

---

## 3.3 REPEATABLE READ
- 하나의 트랜잭션 동안 **같은 값을 읽게 보장**
- MySQL의 기본 레벨

문제: Phantom Read
예)
1) A가 `WHERE owner_id = 10`을 조회  
2) B가 owner_id=10인 데이터를 새로 Insert  
3) A가 같은 조건으로 다시 조회 시 “유령 데이터” 추가됨

---

## 3.4 SERIALIZABLE
- 가장 높은 격리 수준
- 트랜잭션을 직렬로 실행하는 효과
- **성능 저하 심함**
- 금융권, 정합성 최우선 업무 일부에만 사용

---

# 4. 트랜잭션과 Lock

트랜잭션은 내부적으로 **Lock(잠금)**을 사용하여 데이터 정합성을 보장한다.

## ✔ 4.1 Lock 종류
### 1) Shared Lock (S-Lock, 공유락)
- 읽기(Read)를 위한 락  
- 여러 트랜잭션에서 공유 가능

### 2) Exclusive Lock (X-Lock, 배타락)
- 쓰기(Write)를 위한 락  
- 해당 자원에 단 하나의 트랜잭션만 접근 가능  
- 쓰기 중에는 읽기도 불가

---

# 5. 트랜잭션 문제 사례

## 5.1 Dirty Read
- 커밋되지 않은 데이터를 다른 트랜잭션이 읽음  
→ READ UNCOMMITTED에서만 발생

---

## 5.2 Non-repeatable Read
- 같은 쿼리를 두 번 실행할 때 값이 달라짐  
→ READ COMMITTED에서 발생

---

## 5.3 Phantom Read
- 조건 조회 시, 이전에는 없던 새로운 행이 생김  
→ REPEATABLE READ에서 발생

---

# 6. 실무에서 트랜잭션 사용 패턴

예: 통행료 무인수납 QR 결제 로직에서

```sql
BEGIN;

 -- 1. 미납 정보 조회
SELECT * FROM UNPAID WHERE car_no = :carNo FOR UPDATE;

 -- 2. 결제 처리
UPDATE UNPAID SET status = 'PAID' WHERE id = :id;

 -- 3. 결제 이력 기록
INSERT INTO PAYMENT_LOG (car_no, amount, result) VALUES (...);

COMMIT;
