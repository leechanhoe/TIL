![](https://velog.velcdn.com/images/dodo4723/post/0f7908ca-17ad-4b55-a243-09ab2b0c57d7/image.png)

[김영한 개발자님의 스프링 DB 2편 강의](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-db-2#reviews)를 수강하고 중요한 내용을 정리했습니다.

# 5. 데이터 접근 기술 - JPA

<br>

## 5.1. JPA 시작

스프링과 JPA는 자바 엔터프라이즈(기업) 시장의 주력 기술이다.
스프링이 DI 컨테이너를 포함한 애플리케이션 전반의 다양한 기능을 제공한다면, **JPA는 ORM 데이터 접근 기술을 제공한다.**

**JdbcTemplate**이나 **MyBatis** 같은 SQL 매퍼 기술은
SQL을 개발자가 직접 작성해야 하지만, **JPA**를 사용하면 SQL도 JPA가 대신 작성하고 처리해준다.

실무에서는 **JPA**를 더욱 편리하게 사용하기 위해 **스프링 데이터 JPA**와 **Querydsl**이라는 기술을 함께 사용한다.

## 5.2. ORM 개념
객체와 관계형 데이터베이스의 차이는 상속, 연관관계, 데이터 타입, 데이터 식별방법 등이 있는데, 객체답게 모델링 할수록 매핑 작업만 늘어난다. 객체를 자바 컬렉션에 저장 하듯이 DB에 저장할 수는 없을까?

### Object-relational mapping(객체 관계 매핑)
- 객체는 객체대로 설계
- 관계형 데이터베이스는 관계형 데이터베이스대로 설계
- ORM 프레임워크가 중간에서 매핑
- 대중적인 언어에는 대부분 ORM 기술이 존재

### JAP의 장점

- SQL 중심적인 개발에서 객체 중심으로 개발
- 생산성
- 유지보수
- 패러다임의 불일치 해결 (상속, 연관관계, 객체 그래프 탐색, 비교)
- 성능
- 데이터 접근 추상화와 벤더 독립성
- 표준

<br>

## 5.3. JPA 설정

**build.gradle**
```java
// JDBC를 주석처리하고 JPA, 스프링 데이터 JPA 추가
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
//implementation 'org.springframework.boot:spring-boot-starter-jdbc'
```
**application.properties**
```java
//하이버네이트가 생성하고 실행하는 SQL을 확인할 수 있다.
logging.level.org.hibernate.SQL=DEBUG
//SQL에 바인딩 되는 파라미터를 확인할 수 있다.
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```

<br>

## 5.4. JPA 적용

JPA에서 가장 중요한 부분은 객체와 테이블을 매핑하는 것이다.

```java
@Data
@Entity
public class Item {
 	@Id @GeneratedValue(strategy = GenerationType.IDENTITY)
 	private Long id;
 	@Column(name = "item_name", length = 10) // 생략가능
 	private String itemName;    
 	private Integer price;
 	private Integer quantity;
    
 	public Item() {}
 
 	public Item(String itemName, Integer price, Integer quantity) {
 		this.itemName = itemName;
 		this.price = price;
 		this.quantity = quantity;
 	}
}
```
- `@Entity` : JPA가 사용하는 객체라는 뜻이다. 이 에노테이션이 있어야 JPA가 인식할 수 있다.
- `@Id` : 테이블의 PK와 해당 필드를 매핑한다.
- `@Column` : 객체의 필드를 테이블의 컬럼과 매핑한다. (`itemName` -> `item_name` 자동 변환 가능 -> 생략가능)
- JPA는 `public` 또는 `protected` 의 기본 생성자가 필수이다.

<br>

코드를 보며 알아보자.
```java
@Slf4j
@Repository
@Transactional
public class JpaItemRepositoryV1 implements ItemRepository {
 	private final EntityManager em;
    
 	public JpaItemRepositoryV1(EntityManager em) {
 		this.em = em;
 	
    }
    
 	@Override
 	public Item save(Item item) {
 		em.persist(item);
 		return item;
 	}
    
 	@Override
 	public void update(Long itemId, ItemUpdateDto updateParam) {
 		Item findItem = em.find(Item.class, itemId);
 		findItem.setItemName(updateParam.getItemName());
 		findItem.setPrice(updateParam.getPrice());
 		findItem.setQuantity(updateParam.getQuantity());
 	}
    
 	@Override
 	public Optional<Item> findById(Long id) {
 		Item item = em.find(Item.class, id);
 		return Optional.ofNullable(item);
 	}
    
 	@Override
 	public List<Item> findAll(ItemSearchCond cond) {
 		String jpql = "select i from Item i";
 		//동적 쿼리 생략
 		TypedQuery<Item> query = em.createQuery(jpql, Item.class);
 		return query.getResultList();
    }
}
```
스프링을 통해 엔티티 매니저(EntityManager) 라는 것을 주입받은 것을 확인할 수 있다. JPA의 모든 동작은 엔티티 매니저를 통해서 이루어진다. 엔티티 매니저는 내부에 데이터소스를 가지고 있고, 데이터베이스에 접근할 수 있다.

`@Transactional` : JPA의 모든 데이터 변경(등록, 수정, 삭제)은 필수적으로 트랜잭션 안에서 이루어져야 한다. 따라서 리포지토리에 트랜잭션을 걸어주었다.

예제에서는 복잡한 비즈니스 로직이 없어서 서비스 계층에서 트랜잭션을 걸지 않고 리포지토리에 걸었지만, 일반적으로는 **비즈니스 로직을 시작하는 서비스 계층에 트랜잭션을 걸어주는 것이 맞다.**

<br>

## 5.5. 리포지토리 분석

위 코드를 보면서 분석해보자.

### save() - 저장
JPA에서 객체를 테이블에 저장할 때는 엔티티 매니저가 제공하는 `em.persist()` 메서드를 사용하면 된다.

### update() - 수정
`em.update()`같은 메서드는 전혀 호출하지 않았다.
JPA는 트랜잭션이 커밋되는 시점에, 변경된 엔티티 객체가 있는지 확인한다. 특정 엔티티 객체가 변경된 경우에는 UPDATE SQL을 실행한다.

### findById() - 단건 조회

JPA에서 엔티티 객체를 PK를 기준으로 조회할 때는 `find()` 를 사용하고 조회 타입과, PK 값을 주면 된다.

### JPQL
JPA는 **JPQL(Java Persistence Query Language)이라는 객체지향 쿼리 언어**를 제공한다.
- 주로 여러 데이터를 복잡한 조건으로 조회할 때 사용한다.
- SQL이 테이블을 대상으로 한다면, JPQL은 엔티티 객체를 대상으로 SQL을 실행한다 생각하면 된다.
- 엔티티 객체를 대상으로 하기 때문에 `from` 다음에 `Item` 엔티티 객체 이름이 들어간다. 
- 엔티티 객체와 속성의 대소문자는 구분해야 한다.
- JPQL은 SQL과 문법이 거의 비슷하기 때문에 개발자들이 쉽게 적응할 수 있다.

**JPQL을 실행하면 그 안에 포함된 엔티티 객체의 매핑 정보를 활용해서 SQL을 만들게 된다.**

#### 실행된 JPQL
```sql
select i from Item i
where i.itemName like concat('%',:itemName,'%')
 	and i.price <= :maxPrice
```
#### JPQL을 통해 실행된 SQL
```sql
select
 	item0_.id as id1_0_,
 	item0_.item_name as item_nam2_0_,
 	item0_.price as price3_0_,
 	item0_.quantity as quantity4_0_
from item item0_
where (item0_.item_name like ('%'||?||'%'))
 	and item0_.price<=?
```

JPQL에서 파라미터 입력 : `where price <= :maxPrice`
파라미터 바인딩 : `query.setParameter("maxPrice", maxPrice)`

JPA를 사용해도 동적 쿼리 문제가 남아있는데, Querydsl이라는 기술을 활용하면 매우 깔끔하게 사용할 수 있다.

<br>

## 5.6. JPA 적용2 - 예외 변환
`EntityManager` 는 순수한 JPA 기술이고, 스프링과는 관계가 없어 JPA 예외가 발생하게 된다

JPA는 `PersistenceException` 과 그 하위 예외를 발생시킨다.  JPA 예외를 스프링 예외 추상화로 추상화시키기 위해선 `@Repository`가 필요하다.

### @Repository의 기능

- `@Repository` 가 붙은 클래스는 컴포넌트 스캔의 대상과 예외 변환 AOP의 적용 대상이 된다.
- 스프링과 JPA를 함께 사용하는 경우 스프링은 JPA 예외 변환기
`(PersistenceExceptionTranslator)`를 등록한다.
- 예외 변환 AOP 프록시는 JPA 관련 예외가 발생하면 JPA 예외 변환기를 통해 발생한 예외를 스프링 데이터 접근 예외로 변환한다.

![](https://velog.velcdn.com/images/dodo4723/post/2f812081-860f-4d9a-b019-5ecd6fb1555e/image.png)
