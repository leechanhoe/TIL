![](https://velog.velcdn.com/images/dodo4723/post/44b37bb8-5a9f-4b9e-8965-67ae21032311/image.png)

## SELECT문의 기본 형식
```sql
SELECT 열이름
	FROM 테이블_이름
    WHERE 조건식
    GROUP BY 열_이름
    HAVING 조건식
    ORDER BY 열_이름
    LIMIT 숫자
```
### **FROM**
테이블에서 내용을 가져온다는 의미
<br/>

### **WHERE**
조회하는 결과에 특정한 조건을 추가해서 원하는 데이터만 보고 싶을 때 사용

#### **관계 연산자(<, <=, >, >=, =)**
```sql 
WHERE a >= 5  --  a가 5이상인 데이터
```
#### **AND / OR**
```sql 
WHERE a >= 5 AND a <= 10  --  a가 5이상 10이하인 데이터
```
#### **BETWEEN ~ AND**
```sql 
WHERE a BETWEEN 5 AND 10  --  a가 5이상 10이하인 데이터
```
#### **IN()**
```sql 
WHERE a IN('A', 'B', 'C)'  --  a가 'A', 'B', 'C'인 데이터
```
#### **LIKE**
%
```sql 
WHERE a LIKE 'b%'  --  a가 b로 시작하는 데이터 (b뒤는 무엇이든 허용)
```
언더바(_)
```sql 
WHERE a LIKE '__bb'  --  bb앞 두글자는 상관없고 뒤는bb인 데이터
```
### ORDER BY
결과의 값이나 개수에 대해서는 영향을 미치지 않지만, 결과가 출력되는 순서를 조절

```sql 
ORDER BY date;  --  날짜형식인 date를 날짜순으로 오름차순 정렬
```
#### DESC
```sql 
ORDER BY date DESC;  --  날짜형식인 date를 날짜순으로 내림차순 정렬
```
#### 두가지 조건
```sql 
ORDER BY a DESC, b ASC;  --  a를 내림차순 정렬하되, a가 같으면 b가 오름차순인 순서로 정렬
```
#### LIMIT
```sql 
LIMIT 3, 2;  --  3번째부터 2건만 조회가능
LIMIT 3;  --  0번째부터 3건만 조회가능
```
### GROUP BY
말 그대로 그룹으로 묶어줌. 집계 함수와 함께 쓰인다
> SUM() : 합계
AVG() : 평균
MIN() : 최솟값
MAX() : 최댓값
COUNT() : 행의 개수를 센다
COUNT(DISTINCT) : 행의 개수를 센다(중복은 1개만 인정)

```sql 
SELECT a, sum(b) FROM c GRUOP BY a;  --  c에서 a들이 가지고 있는 b의 합을 출력
```
### HAVING
WHERE과 비슷한 개념으로 조건을 제한하지만 집계 함수에 대해서 조건을 제한한다. 반드시 GRUOP BY절의 다음에 나와야 한다.
```sql 
GRUOP BY a
HAVING SUM(b*c) > 1000;  --  a들이 가지고 있는 b와 c의 곱이 1000보다 큰 데이터 출력
```

### 기타 문법
#### **USE**
현재 사용하는 데이터베이스를 지정 또는 변경
```sql
USE 데이터베이스_이름;
```
#### **AUTO_INCREMENT**
1,2,3과 같이 자동으로 숫자를 입력해줌
```sql
num INT AUTO_INCREMENT NOT NULL PRIMARY KEY
```
#### DISTINCT
중복제거
```sql
SELECT DISTINCT a FROM b
```