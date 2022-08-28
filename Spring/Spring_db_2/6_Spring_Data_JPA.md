![](https://velog.velcdn.com/images/dodo4723/post/0f7908ca-17ad-4b55-a243-09ab2b0c57d7/image.png)

[김영한 개발자님의 스프링 DB 2편 강의](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-db-2#reviews)를 수강하고 중요한 내용을 정리했습니다.

# 6. 데이터 접근 기술 - 스프링 데이터 JPA

<br>

## 6.1. 스프링 데이터 JPA 소개

**Spring Data JPA**는 JPA를 한 단계 더 추상화 시켜 개발 용이성을 상당히 올려주는 인터페이스다.

<br>

### 기능1. 공통 인터페이스 기능
Spring Data JPA는 `JpaRepository`라는 인터페이스를 제공한다.
![](https://velog.velcdn.com/images/dodo4723/post/b9b7b2e9-cbd4-471b-be0e-9835ccdc2382/image.png)

```java
public interface ItemRepository extends JpaRepository<Member, Long> {}
```
- `JpaRepository` 인터페이스를 인터페이스 상속 받고, 제네릭에 관리할 <엔티티, 엔티티ID> 를 주면 된다.
- `JpaRepository` 인터페이스만 상속받으면 스프링 데이터 JPA가 프록시 기술을 사용해서 구현 클래스를 만들어 스프링 빈으로 등록한다.

<br>

### 기능2. 쿼리 메서드 기능
스프링 데이터 JPA는 인터페이스에 메서드만 적어두면, 메서드 이름을 분석해서 쿼리를 자동으로 만들고 실행해주는 기능을 제공한다.

#### 순수 JPA 리포지토리
순수 JPA를 사용하면 직접 JPQL을 작성하고, 파라미터도 직접 바인딩 해야 한다.
```java
public List<Member> findByUsernameAndAgeGreaterThan(String username, int age) {
 	return em.createQuery("select m from Member m where m.username = :username and m.age > :age")
 		.setParameter("username", username)
 		.setParameter("age", age)
 		.getResultList();
}
```

#### 스프링 데이터 JPA
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
 	List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
}
```
스프링 데이터 JPA는 메서드 이름을 분석해서 필요한 JPQL을 만들고 실행해준다. 물론 JPQL은 JPA가 SQL로 번역해서 실행한다.

#### 스프링 데이터 JPA가 제공하는 쿼리 메소드 기능
- **조회** : `find…By` , `read…By` , `query…By` , `get…By`
예:) `findHelloBy` 처럼 ...에 식별하기 위한 내용(설명)이 들어가도 된다.
- **COUNT**: `count…By` 반환타입 `long`
- **EXISTS**: `exists…By` 반환타입 `boolean`
- **삭제**: `delete…By` , `remove…By` 반환타입 `long`
- **DISTINCT**: `findDistinct` , `findMemberDistinctBy`
- **LIMIT**: `findFirst3` , `findFirst` , `findTop` , `findTop3`

```java
public interface SpringDataJpaItemRepository extends JpaRepository<Item, Long>
{
 	//쿼리 메서드 기능
 	List<Item> findByItemNameLike(String itemName);
 	//쿼리 직접 실행
 	@Query("select i from Item i where i.itemName like :itemName and i.price <= :price")
 	List<Item> findItems(@Param("itemName") String itemName, @Param("price") Integer price);
}
```
쿼리 메서드 기능 대신에 직접 JPQL을 사용하고 싶을 때는 `@Query` 와 함께 JPQL을 작성하면 된다. 이때는 메서드 이름으로 실행하는 규칙은 무시된다.

#### 단점
- 조건이 많으면 메서드 이름이 너무 길어진다.
- 조인 같은 복잡한 조건을 사용할 수 없다.

<br>

## 6.2. 스프링 데이터 JPA 적용

데이터를 조건에 따라 4가지로 분류해서 검색한다.
- 모든 데이터 조회
- 이름 조회
- 가격 조회
- 이름 + 가격 조회


```java
public interface SpringDataJpaItemRepository extends JpaRepository<Item, Long>
{
 	List<Item> findByItemNameLike(String itemName);
 	List<Item> findByPriceLessThanEqual(Integer price);
 	//쿼리 메서드 (아래 메서드와 같은 기능 수행)
 	List<Item> findByItemNameLikeAndPriceLessThanEqual(String itemName, Integer price);
 	//쿼리 직접 실행
 	@Query("select i from Item i where i.itemName like :itemName and i.price <= :price")
 	List<Item> findItems(@Param("itemName") String itemName, @Param("price") Integer price);
}
```

<br>

### 의존관계와 구조
```java
public class JpaItemRepositoryV2 implements ItemRepository {
 	private final SpringDataJpaItemRepository repository;
    
 	@Override
 	public Item save(Item item) {
 ...
```

`ItemService` 는 `ItemRepository` 에 의존하기 때문에 `ItemService` 에서 `SpringDataJpaItemRepository` 를 그대로 사용할 수 없다.

![](https://velog.velcdn.com/images/dodo4723/post/b73c0236-fa1e-408a-9edd-c8f751ad8548/image.png)

이렇게 중간에서 `JpaItemRepository` 가 어댑터 역할을 해준 덕분에 `MemberService` 가 사용하는 `ItemRepository` 인터페이스를 그대로 유지할 수 있고 클라이언트인 `ItemService` 의 코드를 변경하지 않아도 되는 장점이 있다.

### 추가

- 스프링 데이터 JPA도 스프링 예외 추상화를 지원한다. 스프링 데이터 JPA가 만들어주는 프록시에서 이미 예외 변환을 처리하기 때문에, `@Repository` 와 관계없이 예외가 변환된다.

- 스프링 데이터 JPA는 동적 쿼리 기능에 대한 지원이 매우 약하다.