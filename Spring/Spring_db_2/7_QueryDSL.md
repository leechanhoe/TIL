![](https://velog.velcdn.com/images/dodo4723/post/0f7908ca-17ad-4b55-a243-09ab2b0c57d7/image.png)

[김영한 개발자님의 스프링 DB 2편 강의](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-db-2#reviews)를 수강하고 중요한 내용을 정리했습니다.

# 7. 데이터 접근 기술 - QueryDSL

![](https://velog.velcdn.com/images/dodo4723/post/aaa73bae-2779-40a8-82e1-d83f15450902/image.png)


## 7.1. Querydsl 소개

기존 query의 문제점
- query는 문자이므로 Type-check 불가
- 실행하기 전까지 작동여부 확인 불가(런타임 에러)
```java
String sql = "select * from member" +
"where name like ?" +
"and age between ? and ?" // 공백이 없어 오류
// = select * from memberwhere name like ?and age between ? and ?"
```

만약 SQL이 클래스처럼 타입이 있고 자바 코드로 작성 할 수 있다면 type-safe이다.

### QueryDSL?
**Q**uery(쿼리) + **D**omain(도메인) + **S**pecific(특화) + **L**anguage(언어) 

**QueryDSL**은 쿼리를 Java로 type-safe하게 개발할 수 있게 지원하는 프레임워크다. 주로 **JPA쿼리(JPQL)**에 사용하다.

- 쿼리에 특화된 프로그래밍 언어
- 단순, 간결, 유창
- 다양한 저장소 쿼리 기능 통합
- 런타임 에러x 컴파일 에러o

<br>

## 7.2. Querydsl 적용
Querydsl은 컴파일 시점에 쿼리용 Q타입을 생성한다.

```java
@Repository
@Transactional
public class JpaItemRepositoryV3 implements ItemRepository {

 	private final EntityManager em;
 	private final JPAQueryFactory query;
    
 	public JpaItemRepositoryV3(EntityManager em) {
 		this.em = em;
 		this.query = new JPAQueryFactory(em);
 	}
   
 	public List<Item> findAllOld(ItemSearchCond itemSearch) {
 		String itemName = itemSearch.getItemName();
 		Integer maxPrice = itemSearch.getMaxPrice();
 		QItem item = QItem.item;
 		BooleanBuilder builder = new BooleanBuilder();
        
 		if (StringUtils.hasText(itemName)) {
 			builder.and(item.itemName.like("%" + itemName + "%"));
 		}
        
 		if (maxPrice != null) {
 			builder.and(item.price.loe(maxPrice));
 		}
        
 		List<Item> result = query
 			.select(item)
 			.from(item)
 			.where(builder)
 			.fetch();
 		return result;
 	}
    
 	@Override
 	public List<Item> findAll(ItemSearchCond cond) {
 		String itemName = cond.getItemName();
 		Integer maxPrice = cond.getMaxPrice();
 		List<Item> result = query
 			.select(item)
 			.from(item)
 			.where(likeItemName(itemName), maxPrice(maxPrice))
 			.fetch();
 		return result;
	}
    
    private BooleanExpression likeItemName(String itemName) {
 		if (StringUtils.hasText(itemName)) {
 			return item.itemName.like("%" + itemName + "%");
 		}
 		return null;
 	}
    
 	private BooleanExpression maxPrice(Integer maxPrice) {
 		if (maxPrice != null) {
 			return item.price.loe(maxPrice);
 		}
 		return null;
 	}
}
```
Querydsl을 사용하려면 `JPAQueryFactory` 가 필요하다. `JPAQueryFactory` 는 JPA 쿼리인 JPQL을 만들기 때문에 `EntityManager` 가 필요하다.

#### findAllOld
- Querydsl을 사용해서 동적 쿼리 문제를 해결한다.
- `BooleanBuilder` 를 사용해서 원하는 `where` 조건들을 넣어주면 된다.
- 이 모든 것을 자바 코드로 작성하기 때문에 동적 쿼리를 매우 편리하게 작성할 수 있다.

#### findAll
- 앞서 `findAllOld` 에서 작성한 코드를 깔끔하게 리팩토링 했다.
- Querydsl에서 `where(A,B)` 에 다양한 조건들을 직접 넣을 수 있는데, 이렇게 넣으면 AND 조건으로 처리된다. 참고로 `where()` 에 `null` 을 입력하면 해당 조건은 무시한다.
- 이 코드의 또 다른 장점은 `likeItemName()` , `maxPrice()` 를 다른 쿼리를 작성할 때 재사용 할 수 있다는 점이다. 쿼리 조건을 부분적으로 모듈화 할 수 있다.

<br>

### 정리
- Querydsl 덕분에 동적 쿼리를 매우 깔끔하게 사용할 수 있다.
- 쿼리 문장에 오타가 있어도 컴파일 시점에 오류를 막을 수 있다.