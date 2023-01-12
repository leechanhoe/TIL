## 12월 말 ~ 1월 초

학기 중엔 바빠 죽을뻔하다가 방학이 되니 제가 하고싶은 공부할 시간이 생겨 좀 여유롭네요. 

성적도 공개됐는데 다행스럽게도 목표했던 4점대의 성적이 나와줘서 좋습니다. 200일 넘게 꾸준히 한문제 이상씩 풀어온 알고리즘은 제 성적중 유일한 A+로 보답을 해줬습니다.

### 복습

방학 직후, **JPA 공부**를 하기로 마음먹었습니다. 하지만 그 전에, 저번 방학때 했던 스프링 공부 내용들을 흩어보면서 복습했습니다. 다행히 다시 보니 얼핏 기억이 나는 것들이 많았습니다.

프로그래밍 언어인 자바도 학기 중에는 사용을 하지 않다보니 까먹은 부분이 많아 파이썬과 C와 햇갈리는 부분들이 있었습니다. 그래서 자바도 다시 복습을 했습니다.

아무리 JPA가 개발자 대신 SQL을 작성해준다고 하지만, 기본적인 SQL과 데이터베이스에 대해 알고 있어야 나중에 문제에 직면했을 때 풀어나가기 쉽다고 들었습니다. 그래서 SQL 책 한권을 읽었습니다.

![](https://velog.velcdn.com/images/dodo4723/post/af445bb2-3865-4b55-929a-aace941d49d3/image.png)

사실 이 책은 저번 방학때 앞에 반 정도 읽다 말았습니다. 정확히는 기억이 나지 않지만 뭔가 다른 공부가 우선이라고 생각했던 것 같습니다.

중요하다고 생각되는 내용이나 햇갈리기 쉬운 내용들을 몇가지 정리했습니다.

<br>
<br>

![](https://velog.velcdn.com/images/dodo4723/post/4a0a0135-10ef-4e26-b131-d1e42bc505b5/image.png)


## 정리

> 데이터베이스 객체는 테이블, 인덱스, 뷰, 스토어드 프로시저, 트리거, 함수, 커서등이 있습니다.

<br>

### 조작어(data Manipulation language)

#### SELECT문 순서

```sql
SELECT 열 이름
	FROM 테이블 이름
	WHERE 조건식
	GROUP BY 열 이름
	HAVING 조건식
	ORDER BY 열 이름
	LIMIT 숫자
```
    
`GROUP BY`는 `WHERE` 대신 `HAVING`과 같이 사용
주로 집계 함수(`SUM`, `MIN`, `AVG`등)과 주로 사용
```sql
SELECT mem_id "회원 아이디", SUM(price*amount) "총 구매 금액"
	FROM buy
    GROUP BY mem_id
    HAVING SUM(price*amount) > 1000
    ORDER BY SUM(price*amount) DESC;
```
|회원 아이디|총 구매 금액|
|:---:|:---:|
|MMU|1950|
|BLK|1210|

<br>
<br>

#### INSERT

```sql
INSERT INTO 테이블_이름 VALUES (값1, 값2, ...);
INSERT INTO 테이블_이름 VALUES (값3, 값4, ...);
INSERT INTO 테이블_이름 VALUES (값5, 값6, ...);
-- 아래와 같음
INSERT INTO 테이블_이름 VALUES (값1, 값2, ...), (값3, 값4, ...), (값5, 값6, ...);
```

<br>

#### UPDATE
서울을 뉴욕으로 바꾸고 인구도 0으로 바꾸기
```sql
UPDATE city_popul
	SET city_name = '뉴욕', population = 0
    WHERE city_name = '서울';
```

<br>

#### DELETE
서로 시작하는 이름인 데이터 삭제

```sql
DELETE FROM city_popul
	WHERE city_name LIKE '서%';
```

<br>
<br>

### 데이터 형식

#### 변수사용
```sql
SET @변수이름 = 변수의 값; -- 변수 값 대입
SELECT @변수이름 ; -- 변수 값 출력
```

#### PREPARE, EXECUTE
`LIMIT`에는 변수사용이 불가능하여 `PREPARE`, `EXECUTE` 로 대체

```sql
SET @count = 3;
PREPARE mySQL FROM 'SELECT mem_name, height FROM member ORDER BY height LIMIT ?'; 
-- mySQL이라는 이름으로 준비만 해놓음
EXECUTE mySQL USING @count; -- ?에 3 대입
```

#### 형변환 - CAST, CONVERT
```sql
SELECT AVG(price) AS '평균 가격' FROM buy; -- 결과 : 142.9167
```
실수를 정수(`SINGED`)로 변환
```sql
SELECT CAST(AVG(price) AS SIGNED) '평균 가격' FROM buy;
-- 또는
SELECT CONVERT(AVG(price), SIGNED) '평균 가격' FROM buy;
-- 결과 : 143
```

<br>
<br>

### 조인

#### 내부 조인

두 테이블에 모두 있는 내용만 출력
별칭(밑의 경우 M, B) 사용 권장 - 컬럼명이 겹칠 수 있기 때문
```sql
SELECT DISTINCT M.mem_id, M.mem_name, M.addr
	FROM buy B
    	INNER JOIN member M
    	ON B.mem_id = M.mem_id
    ORDER BY M.mem_id;
```

실행결과

|mem_id|mem_name|addr|
|:---:|:---:|:---:|
|APN|에이핑크|경기|
|BLK|블랙핑크|경남|
|GRL|소녀시대|서울|
|MMU|마마무|전남|

<br>

- **외부 조인** : 한쪽 테이블에만 있는 내용도 출력
- **상호 조인** : 한쪽 테이블의 모든 행과 다른 쪽 테이블의 모든 행을 조인 - 두 테이블의 각 행의 개수를 곱한 개수
- **자체 조인** : 1개의 테이블로 조인 - 2개 이상의 열로 존재할 때 가능

<br>
<br>

### SQL 프로그래밍

프로그래밍 언어와 비슷하게 `IF/ELSE`, `CASE`,`WHILE`(FOR문은 없음)이 있습니다.

<br>
<br>

### 제약조건

#### 기본키(PRIMARY KEY)와 외래키(FORIGN KEY)
두 테이블이 연결된 상태에서는 기준 테이블의 열 이름이 변경/삭제 되지 않음. 이때 `ON UPDATE/DELETE CASCADE`를 사용가능 

- `UNIQUE` : 중복 허용X, 비어있는 값(NULL) 허용
- `CHECK` : 입력되는 데이터 점검
- `DEFALUT` : 입력하지 않았을 때 기본값

<br>
<br>

### 뷰
> - `SELECT`문으로 만들어져 있음. 테이블로 취급
- 보안에 도움이 됨
- 복잡한 SQL을 단순하게 만들 수 있음
- `CREATE OR REPLACE VIEW` : 기존 뷰가 있으면 덮어쓰기'
- 뷰에서 데이터 입력을 하려면, 뷰에서 보이지 않는 테이블의 열에 `NOT NULL`이 없어야함
- `WITH CHECK OPTION` : 뷰에 설정된 값의 범위가 벗어나는 값은 입력되지 않게 함

<br>
<br>

### 인덱스

> - `SELECT`문으로 검색하는 속도가 빨라지지만 추가적인 공간이 필요함.
- 열 단위에 생성됨
- 균형 트리 구조
- `INSERT`를 하면 데이터가 추가될 때 페이지 분할이 일어나 느려질 수 있음
- `SHOW INDEX` : 테이블에 생성된 인덱스 정보를 보여줌
- `SHOW TABLE STATUS` : 테이블에 생성된 인덱스의 크기 확인
- 데이터베이스 엔진이 인덱스 사용과 비사용을 자동으로 결정

```sql
-- 인덱스 생성
	CREATE [UNIQUE] INDEX 인덱스_이름
    	ON 테이블_이름 (열_이름) [ASC | DESC]
    ANALYZE TABLE 테이블_이름 -- <<해야 적용
```

<br>

#### 클러스터형 인덱스

- 영어사전처럼 책의 내용이 이미 알파벳 순서대로 정렬되어 있는 것
- 정렬되있고, 테이블당 기본 키에 1개만 생성

#### 보조 인덱스

- 찾아보기가 별도로 있고, 찾아보기에서 해당 단어를 찾은 후에 옆에 표시된 페이지를 펼쳐야 실제 찾는 내용이 있는 것
- 정렬되지 않고 한 테이블에 여러개 생성 가능
- 중복을 허용하는 **단순 보조 인덱스**와 허용하지 않는 **고유 보조 인덱스**가 있음

<br>

#### 인덱스를 효과적으로 사용하는 방법
> - `WHERE` 절에서 사용되는 열에 인덱스 생성
- 해당 `WHERE` 절을 자주 사용할때 사용
- 데이터의 중복이 높은 열은 별 효과 없음
- 사용하지 않는 인덱스는 제거

<br>
<br>

### 스토어드 프로시저
MySQL에서 제공하는 프로그래밍 기능으로 쿼리 문의 집합입니다.

```sql
DELIMITER $$
CREATE PROCEDURE 스토이드_프로시져_이름(IN 또는 OUT 매개변수)
BEGIN
	-- SQL 프로그래밍 코드 - IF, CASE, WHILE 문 활용 가능

END $$
DELIMITER;
```

**예제 - 1부터 100까지의 합**
```sql
DROP PROCEDURE IF EXISTS while_proc;
DELIMITER $$
CREATE PROCEDURE while_proc()
BEGIN
	DECLARE total INT;
    DECLARE num INT;
    SET total = 0;
    SET num = 1;
    
    WHILE(num <= 100) DO
    	SET total = total + num;
        SET num = num + 1;
    END WHILE;
    SELECT total AS '1~100 합계';
END $$
DELIMITER;
    
CALL while_proc();
```

<br>
<br>

### 스토어드 함수
> - 내장 함수 외에 직접 함수를 만드는 기능
- 스토어드 프로시져와 모양이 비슷하지만, 용도가 다르며, `RETURNS` 예약어를 통해 하나의 값을 반환해야함
- `CALL`이 아닌 `SELECT`문 안에서 호출
- 내부에 `SELECT` 사용 불가능

```sql
DELIMITER $$
CREATE FUNCTION 스토이드_함수_이름(입력 매개변수)
	RETURNS 반환형식
BEGIN
	-- SQL 프로그래밍 코드 - IF, CASE, WHILE 문 활용 가능
    RETURN 반환값;
    
END $$
DELIMITER;
```

예제 - 더하기 함수

```sql
DROP FUNCTION IF EXISTS ;
DELIMITER $$
CREATE FUNCTUIN sumFunc(number1 INT, number2 INT)
	RETURNS INT
BEGIN
	RETURN number1 + number2;
END $$
DELIMITER;
    
SELECT sunFunc(100, 200) AS '합계';
```

<br>
<br>

### 커서
커서는 첫 번째 행을 처리한 후 마지막 행까지 한 행씩 접근해서 값을 처리합니다.

**예제 - 평균 구하기**
```sql
DROP PROCEDURE IF EXISTS cursor_proc;
DELIMITER $$
CREATE PROCEDURE cursor_proc()
BEGIN
	DECLARE memNunber INT; -- 1. 사용할 변수 준비
    DECLARE cnt INT DEFAULT 0;
    DECLARE totNumber INT DEFAULT 0;
    DECLARE endOfRow BOOLEAN DEFAULT FALSE; -- 행의 끝을 파악하기 위한 변수
    
    DECLARE memberCursor CURSOR FOR -- 2. 커서 선언하기
    	SELECT mem_number FROM member;
        
    DECLARE CONTINUE HANDLER -- 3. 반복 조건 준비 예약어
    	FOR NOT FOUND SET endOfRow = TRUE; -- 더 행이 없을 때 이어진 문장 수행
        
    OPEN member Cursor -- 4. 커서 열기
    
    cursor_loop: LOOP -- 5. 행 반복하기
    	FETCH memberCursor INTO memNumber; -- 한 행씩 읽어오기
        
        IF endOfRow THEN
        	LEAVE cursor_loop; -- 반복할 이름 빠져나가기
        END IF;
        
        SET cnt = cnt + 1;
        SET totNumber = totNumber + memNumber;
    END LOOP cursor_loop;
    
    SELECT (totNumber/cnt) AS '회원의 평균 인원 수';
    
    CLOSE memberCursor; -- 6. 커서 닫기
END $$
DELIMITER;
```

<br>
<br>

### 트리거

**데이터의 무결성** : 트리거는 자동으로 수행하여 사용자가 추가 작업을 잊어버리는 실수를 방지합니다. - 오류 발생 막기

DML(`INSERT`, `UPDATE`, `DELETE`)의 이벤트가 발생할 때 작동됩니다.

**변경이 발생했을 때 작동하는 트리거 - 백업 테이블에 로그 남기기**
```sql
DROP TRIGGER IF EXISTS singer_updateTrg;
DELIMITER $$
CREATE TRIGGER singer_updateTrg -- 트리거 이름
	AFTER UPDATE -- 변경 후에 작동하도록 지정
    ON singer -- 트리거를 부착할 테이블
    FOR EACH ROW
BEGIN
	INSERT INTO backup_singer VALUES(OLD.mem_id, OLD. mem_name,
    	OLD.mem_number, OLD.addr, '수정', CURDATE(), CURRENT_USER() );
END $$
DELIMITER;
```

`OLD` 테이블은 `UPDATE`나 `DELETE`가 수행될 때, 변경되기 전의 데이터가 잠깐 지정되는 임시 테이블입니다. `NEW` 테이블도 있지만, 어차피 변경된 테이블을 사용하기 때문에 거의 사용하지 않습니다.

<br>
<br>

## 마치며

CS과목 중 제가 제일 흥미있는 과목이 데이터베이스입니다. SQL이 다음 학기에 있는 데이터베이스 수업에 도움이 되면 좋겠습니다. (아마 이론 수업이라 별 도움이 안 될 것입니다..)

6월에 SQLD 자격증 취득을 생각중인데 그 첫걸음을 나아간 것 같아서 뿌듯합니다.

뭐 한것도 없는 것 같은데 벌써 방학이 거의 1달이 되어갑니다.. **이제는 JPA 입니다**. 남은 기간동안 JPA 공부를 얼마나 할 수 있을 진 모르겠지만, 이번에 결제한 30만원의 강의를 돈 아깝지 않게 최대한 열심히 들을 예정입니다.