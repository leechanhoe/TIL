![](https://velog.velcdn.com/images/dodo4723/post/cacc733d-b93f-40ef-940f-c80271eb4b0d/image.jpg)

## INSERT
테이블에 행 데이터를 입력하는 기본적인 SQL문
> INSERT INTO 테이블 [(열1, 열2, ...)] VALUES (값1, 값2, ...)

값이 테이블을 정의할 때의 열 순서 및 개수와 같으면 열들은 생략 가능
```sql
INSERT INTO a VALUES ('b', 'c', 'd');
```
속성을 입력하지 않으면 NULL이 들어감
```sql
INSERT INTO (a, b) VALUES ('c', 'd'); -- a, b, c속성이 있을 때 c에 NULL
```
<br/>

### AUTO_INCREMENT
열을 정의 할 때 1부터 증가하는 값을 입력해줌
```sql
a INT AUTO_INCREMENT PRIMARY KEY -- 테이블 생성, a속성 지정시
-- a 속성에 데이터를 입력시 NULL입력
```
#### LAST_INSERT_ID()
현재 어느 숫자까지 증가되었는지 확인
<br/>

### INSERT INTO ~ SELECT
다른 테이블에 이미 데이터가 입력되어 있다면 해당 테이블의 데이터를 가져와서 한 번에 입력
열 개수가 서로 같아야함
> INSERT INTO 테이블 (열1, 열2, ...)
	SELECT 문;
    
### DESC
테이블의 구조 출력
<br/>
<br/>

## UPDATE
기존에 입력되어 있는 값을 수정하는 명령
```sql
UPDATE 테이블_이름
	SET 열1=값1, 열2=값2, ...
    WHERE 조건 ;
```

A속성이 c인 데이터의 A값을 a로, B값을 b로 설정 
```sql
UPDATE 테이블_이름
	SET A = 'a', B = 'b'
    WHERE A = 'c' ;
```
WHERE절 생략시 모든 행의 값이 변경됨
<br/>
<br/>

## DELETE
테이블의 행 데이터를 삭제
```sql
DELETE FROM 테이블이름 WHERE 조건;
```
'ab'글자로 시작하는 B 중에서 상위 5건만 삭제
```sql
DELETE FROM A 
	WHERE B LIKE 'ab%'
    LIMIT 5;
```
### TRUNCATE
모든 행 데이터를 삭제하고 빈 테이블을 남김
DELETE보다 훨씬 빠르다.