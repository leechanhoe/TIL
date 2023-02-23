![](https://velog.velcdn.com/images/dodo4723/post/f370c5d0-7f98-4245-9c23-7e4743dd1ea3/image.png)

김영한 개발자님의 [자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard) 강의를 듣고 중요한 점이나 인상깊었던 내용들을 정리했습니다.

<br>
<br>
<br>
<br>

## 목차

[1. JPA](#1-jpa)

[2. 영속성 컨텍스트](#2-영속성-컨텍스트)

[3. 엔티티 매핑](#3-엔티티-매핑)

[4. 연관관계 매핑](#4-연관관계-매핑)

[5. 상속관계 매핑](#5-상속관계-매핑)

[6. 프록시와 연관관계](#6-프록시와-연관관계)

[7. 값 타입](#7-값-타입)

[8. JPQL](#8-jpql)

# JPA

<br>
<br>

## 1. JPA

**J**ava **P**ersistence **A**PI
자바 진영의 ORM 기술 표준

### ORM?
**O**bject-**R**elational **M**apping(객체 관계 매핑)

객체는 객체대로 설계하고, 관계형 데이터베이스는 관계형 데이터베이스대로 설계하고, **ORM 프레임워크**가 중간에서 매핑한다.

### JPA의 장점
- SQL 중심적인 개발에서 객체 중심으로 개발
- 생산성, 유지보수, 성능등에서의 이점
- 패러다임의 불일치 해결
- 데이터 접근 추상화와 벤더 독립성

### JPQL
JPA를 사용하면 엔티티 객체를 중심으로 하기 때문에 검색 쿼리에서 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능합니다.

애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요합니다. JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어를 제공합니다.

<br>
<br>
<br>
<br>

## 2. 영속성 컨텍스트
JPA를 이해하는데 가장 중요한 용어로 '엔티티를 영구 저장하는 환경'이라는 뜻.

엔티티 매니저를 통해서 접근할 수 있습니다.

<br>

### 엔티티의 생명주기

>
- **`비영속 (new/transient)`** :영속성 컨텍스트와 전혀 관계가 없는 새로운 상태 
- **`영속 (managed)`** : 영속성 컨텍스트에 관리되는 상태 
- **`준영속 (detached)`** : 영속성 컨텍스트에 저장되었다가 분리된 상태 
- **`삭제 (removed)`** : 삭제된 상태


![](https://velog.velcdn.com/images/dodo4723/post/7b48816d-bf03-420b-a38d-9c641fe4650a/image.png)

<br>

### 영속성 컨텍스트의 이점

>
- **`1차 캐시`** : 캐시에 엔티티가 있으면 데이터베이스에 쿼리를 날리지 않음
- **`동일성 보장`** : == ture 보장
- **`트랜잭션을 지원하는 쓰기 지연`** : 쓰기 지연 SQL저장소에 모아뒀다가 커밋할 때 쿼리를 보냄
- **`변경 감지`** : 엔티티의 스냅샷을 저장했다가 `flush`할때 `UPDATE` 쿼리를 생성해 보낸다.

<br>
<br>
<br>
<br>

## 3. 엔티티 매핑

>
- 객체와 테이블 매핑 : `@Entity`, `@Table`
- 필드와 컬럼 매핑 : `@Column(컬럼 매핑)`, `@Enumerated(enum 타입 매핑)`, `@Lob(BLOB, CLOB 매핑)`, `@Transient(매핑 무시)`
- 기본 키 매핑 : `@Id(ID만 사용하면 직접 할당)`, `@GeneratedValue(자동 생성)`
- 연관관계 매핑 : `@ManyToOne`, `@JoinColumn`

```java
@Id @GeneratedValue
@Column(name = "member_id")
private Long id;
```

<br>

### 데이터베이스 스키마 자동 생성
```yml
# application.yml
jpa:
    hibernate:
      ddl-auto: create
```

> 
- **`create`** : 기존 테이블 삭제 후 다시 생성(운영 DB는 사용 X)
- **`create-drop`** : create와 같으나 종료시점에 테이블 DROP(운영 DB는 사용 X)
- **`update`** : 변경분만 반영(운영 DB는 사용 X)
- **`validate`** : 엔티티와 테이블이 정상 매핑되었는지만 확인
- **`none`** : 사용하지 않음

<br>
<br>
<br>
<br>

## 4. 연관관계 매핑
테이블은 외래 키로 조인을 사용하여 연관된 테이블을 찾지만, 객체는 참조를 사용해서 연관된 객체를 찾는다.

![](https://velog.velcdn.com/images/dodo4723/post/cbefe7e2-c919-45be-8ed0-f7db958868ca/image.png)

```java
@Entity
public class Member { 
	@Id @GeneratedValue
 	private Long id;
 	
    @Column(name = "USERNAME")
 	private String name;
 
 	private int age;

	// @Column(name = "TEAM_ID")
	// private Long teamId;
 
 	@ManyToOne // 참조를 사용해 연관관계 조회 가능 - member.getTeam()
 	@JoinColumn(name = "TEAM_ID") // 매핑할 외래키 이름
 	private Team team;
 }
 ```
 
 ### 양방향 매핑
 테이블은 외래 키 하나로 두 테이블의 연관관계를 관리하지만, 객체의 양방향 관계는 사실 서로 다른 단방향 관계 2개다.
 
 둘 중 하나로 외래 키를 관리해야하는데, 연관관계의 주인만이 외래 키를 관리할 수 있고, 주인이 아닌 쪽은 읽기만 가능
 
**연관관계의 주인은 외래 키의 위치를 기준으로 정해야함**
 
 ```java
@Entity
public class Team {

	@Id @GeneratedValue
 	private Long id;
 
 	private String name;
 
 // 주인이 아니면 mappedBy 속성으로 주인 지정
 	@OneToMany(mappedBy = "team")
 	List<Member> members = new ArrayList<Member>();
}
```

![](https://velog.velcdn.com/images/dodo4723/post/a83f6a04-f06a-4b14-8549-da8d8c2a1c2a/image.png)

순수 객체 상태를 고려해서 항상 양쪽에 값을 설정해아함(무한 루프 조심)

<br>

### 일대일, 다대일, 일대다 다대다 매핑

- **`일대일(1:1)`** : 단방향 관계는 JPA 지원 X, 다대일 양방향 매핑처럼 외래 키가 있는 곳이 연관관계의 주인.

- **`다대일(N:1)`** : 가장 많이 사용하는 연관관계, 외래 키가 있는 쪽이 연관관계의 주인. 양쪽을 참조하도록 개발

- **`일대다(1:N)`** : 일(1)이 연관관계의 주인. 객체와 테이블의 차이 때문에 반대편 테이블의 외래 키를 관리하는 특이한 구조. 대신 **다대일 양방향을 사용권장**

- **`다대다(N:M)`** : 객체와 달리 관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없음. **실무에서 사용X**, 대신 중간 테이블을 이용해서 사이에 `1:N`, `N:1`을 추가

<br>
<br>
<br>
<br>

## 5. 상속관계 매핑

관계형 데이터베이스는 상속 관계가 없음. 슈퍼타입 서브타입 관계라는 모델링 기법이 객체 상속과 유사함.

슈퍼타입 서브타입 논리 모델을 실제 물리 모델로 구현 방법은 세 가지가 있습니다.

![](https://velog.velcdn.com/images/dodo4723/post/02ccaa51-b67f-46c5-a93f-d67c365490b4/image.png)

위 예시로 전략 3가지를 설명하겠습니다.

<br>

### 5.1. 조인 전략

```java
@Inheritance(strategy = InheritanceType.JOINED)
public class Item {}
```

![](https://velog.velcdn.com/images/dodo4723/post/355d562f-adfc-42ab-916d-602d33e1ffc7/image.png)

하위 테이블이 `ITEM_ID`를 기본키와 외래키로 가지고 상위 테이블과 조인하는 전략

- 장점 : 테이블 정규화에 유리함, 저장공간이 효율적임, 외래 키 참조 무결성 제약조건 활용가능

- 단점 : 조회시 조인을 많이 사용해 성능 저하, 조회 쿼리 복잡, 데이터 저장시 INSERT SQL 2번 호출

<br>
<br>

### 5.2. 단일 테이블 전략

```java
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
public class Item {}
```

![](https://velog.velcdn.com/images/dodo4723/post/76f2fd92-8b0f-433b-99ca-014644f58f16/image.png)

상위 테이블에 하위 테이블 속성들을 모아서 관리

- 장점 : 조인이 필요 없으므로 조회 성능이 빠름, 조회 쿼리가 단순함

- 단점 : 자식 엔티티가 매핑한 컬럼은 모두 null 허용, 하나의 테이블이 커질 수 있고 상황에 따라서 조회 성능이 오히려 느려질 수 있음

<br>
<br>

### 5.3. 구현 클래스마다 테이블 전략

```java
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public class Item {}
```

![](https://velog.velcdn.com/images/dodo4723/post/0badaf27-7f9b-4649-aa5e-9e9145610228/image.png)

상위 테이블의 속성을 하위 테이블마다 적용

**이 전략은 데이터베이스 설계자와 ORM 전문가 둘 다 추천하지 않음**

- 장점 : 서브 타입을 명확하게 구분해서 처리할 때 효과적, not null 제약조건 사용 가능

- 단점 : 여러 자식 테이블을 함께 조회할 때 성능이 느림(UNION SQL필요), 자식 테이블을 통합해서 쿼리하기 어려움

<br>
<br>

### 5.4. @MappedSuperclass

`id`, `name`같이 객체의 입장에서 공통 매핑 정보가 필요할 때 사용한다.

부모 클래스에 선언하고 속성만 상속 받아서 사용하고 싶을 때 `@MappedSuperclass`를 사용한다.

DB 테이블과는 상관없이 객체의 입장이다.

![](https://velog.velcdn.com/images/dodo4723/post/a3327d55-171e-434b-8081-e408cd020a70/image.png)

```java
@MappedSuperclass
public class BaseEntity {
	private Long id;
    private String name;
}

@Entity // 직접 생성해서 사용할 일이 없으므로 추상 클래스 권장
public abstract class Item extends BaseEntity{}
```

주로 등록일, 수정일, 등록자, 수정자 같은 전체 엔티티에서 공통
으로 적용하는 정보를 모을 때 사용

<br>
<br>
<br>
<br>

## 6. 프록시와 연관관계

`em.getReference()`로 데이터베이스 조회를 미루는 가짜(프록시)엔티티 객체를 조회한다.

### 프록시 특징

- 실제 클래스를 상속받아 만들어져 사용하는 입장에선 구분하지 않고 사용하면 되지만, 타입 체크시 주의

- 프록시 객체는 실제 객체의 참조를 보관하다 처음 사용할 때 한 번만 초기화

- 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 `em.getReference()`를 호출해도 실제 엔티티 반환

- 준영속 상태일 때, 프록시 초기화시 예외 발생

![](https://velog.velcdn.com/images/dodo4723/post/ae06525c-aaa6-4af5-91f2-31a0db812dab/image.png)

<br>
<br>

### 즉시 로딩과 지연 로딩

```java
@Entity
public class Member{

	@ManyToOne(fetch = FetchType.LAZY) // 지연 로딩
	@JoinColumn(name = "TEAM_ID")
 	private Team team;
    
    @ManyToOne(fetch = FetchType.EAGER) // 즉시 로딩
 	@JoinColumn(name = "TEAM_ID")
 	private Team team;
}
```
**`지연 로딩`** : 프록시 상태였다가 실제 `team`을 사용하는 시점에 초기화

**`즉시 로딩`** : `Member` 조회시 항상 `Team` 도 조회

<br>

### 지연 로딩을 기본적으로 사용하자

- 즉시 로딩은 예상치 못한 SQL이 발생해 가급적 실무에서는 지연 로딩만 사용

- 즉시 로딩은 JPQL에서 N+1 문제를 일으킴

- `@ManyToOne`, `@OneToOne`은 기본이 즉시 로딩이기 때문에 LAZY로 설정해야함

<br>
<br>

### 영속성 전이

특정 엔티티와 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때

```java
@OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true) // 소유자가 하나일때만 사용 (완전종속적일때)
private List<Child> childList = new ArrayList<>();
```
- `ALL` : 모두 적용
- `PERSIST` : 영속
- `REMOVE` : 삭제

`orphanRemoval` : 고아 객체로 컬렉션에서 삭제되면 자동 삭제쿼리 나감

<br>
<br>
<br>
<br>

## 7. 값 타입

#### 엔티티 타입

- `@Entity`로 정의하는 객체
- 데이터가 변해도 식별자로 지속해서 추적 가능
- 예) 회원 엔티티의 키나 나이 값을 변경해도 식별자로 인식 가능

#### 값 타입

- `int`, `Integer`, `String`처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체
- 식별자가 없고 값만 있으므로 변경시 추적 불가
- 예) 숫자 100을 200으로 변경하면 완전히 다른 값으로 대체

<br>

### 임베디드 타입(복합 값 타입)

주로 기본 값 타입을 모아 새로운 값 타입을 직접 정의함

![](https://velog.velcdn.com/images/dodo4723/post/fbbf4783-56fb-49f1-b407-ee0e313708a0/image.png)


```java
public class Member{

	@Embedded // 값 타입을 사용하는 곳에 표시
	private Period workPeriod;
}

@Embeddable // 값 타입을 정의하는 곳에 표시
public class Period {

    private LocalDateTime startDate;
    private LocalDateTime endDate;

	// 기본 생성자 필수
    public Period(LocalDateTime startDate, LocalDateTime endDate) {
        this.startDate = startDate;
        this.endDate = endDate;
  	}
}
```

한 엔티티에서 같은 값 타입을 사용하면 `@AttributeOverrides`, `@AttributeOverride`를 사용해서 컬러 명 속성을 재정의

``` java
@Embedded
@AttributeOverrides({
	@AttributeOverride(name = "city", column = @Column(name = "WORK_CITY")),
	@AttributeOverride(name = "street", column = @Column(name = "WORK_STREET")),
	@AttributeOverride(name = "zipcode", column = @Column(name = "WORK_ZIPCODE"))
})
private Address workAddress;
```

<br>
<br>

### 불변 객체

```java
Address a = new Address(“Old”); 
Address b = a; //객체 타입은 참조를 전달
b.setCity(“New”)
```

임베디드 타입처럼 직접 정의한 값 타입은 자바의 기본 타입이 아니라 객체 타입이므로 참조 값을 직접 대입하는 것을 막을 방법이 없음.

값 타입은 **불변 객체(immutable object)** 로 설계해야함

**생성자로만 값을 설정하고 수정자를 만들지 않아야함**

<br>
<br>

### 값 타입 컬렉션

데이터베이스는 컬렉션을 같은 테이블에 저장할 수 없기 때문에 값 타입을 하나 이상 저장할 때 `@ElementCollection`, `@CollectionTable`을 사용하여 별도의 테이블을 만들어야함.

![](https://velog.velcdn.com/images/dodo4723/post/dacb41ab-4848-4fa6-a61d-1fc4e75f707a/image.png)


```java
@ElementCollection
@CollectionTable(name = "ADDRESS", joinColumns =
@JoinColumn(name = "MEMBER_ID"))
private List<Address> addressHistory = new ArrayList<>();
```

#### 제약사항
- 값 타입 컬렉션에 변경 사항이 발생하면, 주인 엔티티와 연관된
모든 데이터를 삭제하고, 값 타입 컬렉션에 있는 현재 값을 모두
다시 저장한다.

- 값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 **기본
키를 구성해야 함**: null 입력X, 중복 저장X


#### 대안
- 일대다 관계를 위한 엔티티를 만들고, 여기에서 값 타입을 사용
- 영속성 전이(Cascade) + 고아 객체 제거를 사용해서 값 타입 컬
렉션 처럼 사용

<br>
<br>
<br>
<br>

## 8. JPQL

[따로 정리해두었습니다.](https://github.com/leechanhoe/TIL/blob/main/Spring/JPA/JPQL.md)