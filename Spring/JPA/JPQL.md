![](https://velog.velcdn.com/images/dodo4723/post/b873c49f-1cec-4ae1-a209-aa82b54566c5/image.png)

김영한 개발자님의 [자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard) 강의에서 **JPQL**을 공부하고 중요하거나 인상 깊은 내용들을 정리했습니다.

<br>
<br>
<br>
<br>

# JPQL

**J**ava **P**ersistence **Q**uery **L**anguage

JPA를 사용하면 엔티티 객체를 중심으로 하기 때문에 검색 쿼리에서 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능합니다.

애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요합니다. JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어를 제공합니다.

엔티티 객체를 대상으로 쿼리하고, 특정 데이터베이스 SQL에 의존하지 않습니다.

<br>

### 문법

- 엔티티와 속성은 대소문자 구분 (Member, age)
- JPQL 키워드는 대소문자 구분X (SELECT, FROM, where)
- 엔티티 이름 사용, 테이블 이름X
- 별칭은 필수(m)

```java
Query query = em.createQuery("SELECT m.username, m.age from Member m where m.username=:username");

//파라미터 바인딩
query.setParameter("username" ,username);

//페이징 쿼리
 String jpql = "select m from Member m order by m.name desc";
 List<Member> resultList = em.createQuery(jpql, Member.class)
 	.setFirstResult(10)
 	.setMaxResults(20)
 	.getResultList();

```

<br>

**여러 쿼리 예시**

```sql
// 단순 값을 DTO로 바로 조회
// 패키지를 포함한 클래스명 입력
// 순서와 타입이 일치하는 생성자 필요
SELECT new jpabook.jpql.UserDTO(m.username, m.age) FROM Member m

// 조인 대상 필터링
// 회원과 팀을 조인하면서, 팀 이름이 A인 팀만 조인
SELECT m, t FROM Member m LEFT JOIN m.team t on t.name = 'A'

// 연관관계 없는 엔티티 외부 조인
// 회원의 이름과 팀의 이름이 같은 대상 외부 조인
SELECT m, t FROM Member m LEFT JOIN Team t on m.username = t.name
```

<br>
<br>

### 경로 표현식

```sql
select m.username // -> 상태 필드
 	from Member m 
 	join m.team t // -> 단일 값 연관 필드
 	join m.orders o // -> 컬렉션 값 연관 필드
	where t.name = '팀A'
```
- **`상태 필드`** : 단순히 값을 저장하기 위한 필드로 경로 탐색의 끝
- **`단일 값 연관 필드`** : 대상이 엔티티이고, 묵시적 내부 조인 발생, 탐색가능
- **`컬렉션 값 연관 경로`** : 대상이 컬렉션이고, 묵시적 내부 조인 발생, 탐색 X

<br>

join 키워드를 직접 사용하는 **명시적 조인**과는 달리 **묵시적 조인**은 경로 표현식에 의해 묵시적으로 SQL 조인 발생
```sql
select m.team from Member m // m.team 에서 묵시적 조인 발생
```

**묵시적 조인은 조인이 일어나는 상황을 한눈에 파악하기 어렵기 때문에 가급적 명시적 조인 사용**

<br>
<br>
<br>
<br>

### 페치 조인(fetch join)

연관된 엔티티나 컬렉션을 SQL 한 번에 함께 조회하는 기능(즉시 로딩)

```sql
// 페치 조인으로 회원과 팀을 함께 조회해서 지연 로딩X
select m from Member m join fetch m.team

// 컬렉션 페치 조인
select t from Team t join fetch t.members where t.name = '팀A'
```

![](https://velog.velcdn.com/images/dodo4723/post/5345eba9-c76d-4526-91d0-e74c8455d140/image.png)

#### 한계
- 페치 조인 대상에는 별칭을 줄 수 없음
- 둘 이상의 컬렉션은 페치 조인 할 수 없음
- 페이징 API(`setFirstResult`, `setMaxResults`) 사용불가

<br>
<br>
<br>
<br>

### 다형성 쿼리

`Book`, `Movie`가 `Item`의 자식일 경우

```sql
// jpql - Item 중에 Book, Movie를 조회
select i from Item i where type(i) IN (Book, Movie)
// = sql
select i from i where i.DTYPE in (‘B’, ‘M’)


// jpql - TREAT - 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용
select i from Item iwhere treat(i as Book).auther = ‘kim’
// = sql
select i.* from Item i where i.DTYPE = ‘B’ and i.auther = ‘kim
```

<br>
<br>
<br>
<br>

### 벌크 연산

재고가 10개 미만인 모든 상품의 가격을 10% 상승하고 싶을 때, 변경 감지기능으로 실행하려면 너무 많은 SQL이 실행됨

벌크 연산은 쿼리 한 번으로 여러 테이블 로우 변경가능

```java
String qlString = "update Product p " + // UPDATE, DELETE 지원
 	"set p.price = p.price * 1.1 " + 
 	"where p.stockAmount < :stockAmount"; 
 
int resultCount = em.createQuery(qlString) // 영향받은 엔티티 수 반환
 	.setParameter("stockAmount", 10) 
 	.executeUpdate(); // 실행
```

**벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리를 날리기 때문에 수행 후 영속성 컨텍스트 초기화를 해줘야함**