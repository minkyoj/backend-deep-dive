# Oracle DB 인덱스(Index) 정리

## 인덱스란?
인덱스는 도서의 목차처럼 **데이터가 저장된 위치를 빠르게 찾아갈 수 있는 구조**이다.  
DB에서 인덱스는 **B-Tree 구조**를 기본으로 하며, 이를 통해 탐색 시간을 줄여 **전체 테이블 스캔(full scan)을 방지**한다.

---

## 📈 왜 인덱스가 빠른가?
테이블 전체를 순차 탐색하는 대신,
인덱스를 통해 **O(log N)** 수준의 탐색이 가능하다.

### 예시
| 방식                 | 시간 복잡도 |
|---------------------|----------|
| Table Full Scan     | O(N)     |
| Index 탐색 (B-Tree)  | O(log N) |

---

## 인덱스가 잘 안 타는 경우
아래와 같은 경우 인덱스가 제대로 사용되지 않을 수 있다.

### 조건절에서 함수/가공 사용

WHERE TO_CHAR(created_at,'YYYY-MM-DD') = '2024-01-01';
-> 앞쪽이 고정되지 않은 LIKE

WHERE email LIKE '%gmail.com';
-> 낮은 카디널리티(값 종류가 적음)

WHERE gender = 'M';

인덱스가 잘 타는 경우
아래처럼 앞쪽부터 조건이 걸리는 경우 인덱스가 잘 타는 편이다.

범위 조건
WHERE created_at BETWEEN '2024-01-01' AND '2024-01-31';

앞쪽이 고정된 LIKE
WHERE email LIKE 'abc%';

Oracle에서 실행계획 확인하기
Oracle에서 인덱스가 탐색되었는지 확인하려면 아래와 같이 실행한다.

EXPLAIN PLAN FOR
select * from PAYMENT_HISTORY
where CAR_NO = '12가1234';

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
만약 아래와 같이 결과가 나온다면 인덱스를 잘 활용한 것이다.

------------------------------------------------------------------------------------
| Id  | Operation                    | Name              | Rows  | Cost |  Pstart |
------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT             |                   |     1 |   12 |         |
|   1 |  TABLE ACCESS BY INDEX ROWID | PAYMENT_HISTORY   |     1 |   12 |         |
|   2 |   INDEX RANGE SCAN           | IDX_PAYMENT_CARNO |     1 |    2 |         |
------------------------------------------------------------------------------------
INDEX RANGE SCAN이 보이면 인덱스가 정상적으로 탐색됨

FULL TABLE SCAN이면 인덱스가 안 탄 것

**실무에서 이렇게 적용해보자!**
안 좋은 예
SELECT *
  FROM PAYMENT_LOG
 WHERE UPPER(status) = 'FAIL';
→ UPPER() 함수 사용으로 인덱스 무효화

개선 예
WHERE status = 'FAIL';
📌 정리
- 인덱스는 탐색 속도를 줄이는 구조
- WHERE 조건 판단 기준이 중요
- Oracle에서는 EXPLAIN PLAN + DBMS_XPLAN.DISPLAY로 확인
