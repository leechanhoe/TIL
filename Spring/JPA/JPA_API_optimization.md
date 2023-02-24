![](https://velog.velcdn.com/images/dodo4723/post/9c4e0bfd-be22-4ab6-827e-0fc9f39ce09a/image.png)


김영한 개발자님의 [실전! 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-API%EA%B0%9C%EB%B0%9C-%EC%84%B1%EB%8A%A5%EC%B5%9C%EC%A0%81%ED%99%94/dashboard) 강의를 수강하고 중요한 점이나 인상깊었던 점들을 정리했습니다.

<br>
<br>
<br>
<br>

## API 개발

### 등록 V1 - 엔티티를 Request Body에 직접 매핑
```java
@PostMapping("/api/v1/members") // 요청 값으로 Member 엔티티를 직접 받는다.
public CreateMemberResponse saveMemberV1(@RequestBody @Valid Member member)
{
 	Long id = memberService.join(member);
 	return new CreateMemberResponse(id);
}
```

#### 문제점

- 엔티티에 프레젠테이션 계층을 위한 로직과 검증(`@NotEmpty`등)을 위한 로직이 추가됨
- 한 엔티티에 다양한 요청 요구사항을 담기는 어려움
- 엔티티가 변경되면 API 스펙이 변함

<br>
<br>

### 등록 V2 - 엔티티 대신에 DTO를 RequestBody에 매핑

```java
@PostMapping("/api/v2/members")
public CreateMemberResponse saveMemberV2(@RequestBody @Valid
CreateMemberRequest request) {

	Member member = new Member();
 	member.setName(request.getName());
    
 	Long id = memberService.join(member);
 	return new CreateMemberResponse(id);
}

@Data
static class CreateMemberRequest {
 	private String name;
}
```

- 엔티티와 프레젠테이션 계층을 위한 로직을 분리할 수 있다.
- 엔티티와 API 스펙을 명확하게 분리할 수 있다.
- 엔티티가 변해도 API 스펙이 변하지 않는다.

**실무에서는 엔티티를 API 스펙에 노출하면 안됨**

<br>
<br>

### 조회 V1 - 응답 값으로 엔티티를 직접 외부에 노출

```java
@GetMapping("/api/v1/members")
 	public List<Member> membersV1() {
 	return memberService.findMembers();
}
```

#### 문제점
- 엔티티에 프레젠테이션 계층을 위한 로직이 추가됨
- 엔티티의 모든 값이 노출
- 한 엔티티에 다양한 응답 로직을 담기는 어려움
- 엔티티가 변경되면 API 스펙이 변한다.
- 추가로 컬렉션을 직접 반환하면 항후 API 스펙을 변경하기 어렵다.

<br>

### 조회 V2 - 응답 값으로 엔티티가 아닌 별도의 DTO 사용

```java
@GetMapping("/api/v2/members")
 	public Result membersV2() {
 	List<Member> findMembers = memberService.findMembers();
 
 	// 엔티티 -> DTO 변환
	List<MemberDto> collect = findMembers.stream()
 	.map(m -> new MemberDto(m.getName()))
 	.collect(Collectors.toList());
 	return new Result(collect);
}
 
@Data // 컬렉션을 감싸서 향후 필요한 필드를 추가할 수 있다.
@AllArgsConstructor
static class Result<T> {
 	private T data;
}
 
@Data
@AllArgsConstructor
static class MemberDto {
	private String name;
}
```
 
**엔티티가 변해도 API 스펙이 변경되지 않는다.**
 
<br>
<br>
<br>
<br>

## API 개발 고급 - 지연 로딩과 조회 성능 최적화

### 조회

#### 문제점
- 엔티티를 직접 노출하면 지연 로딩시 프록시가 존재하여 json 변환시 예외발생
- 바로 DTO로 변환시 지연 로딩으로 인하여 N+1 문제 발생

<br>

#### 해결

```java
public List<Order> findAllWithMemberDelivery() {
 	return em.createQuery(
 	"select o from Order o" +
 	" join fetch o.member m" +
 	" join fetch o.delivery d", Order.class)
 	.getResultList();
}
```

위와 같이 **페치 조인**으로 성능을 최적화하면 대부분의 성능 이슈가 해결됨

```java
public List<OrderSimpleQueryDto> findOrderDtos() {
 	return em.createQuery(
 		"select new jpabook.jpashop.repository.order.simplequery.OrderSimpleQueryDto(o.id, m.name, o.orderDate, o.status, d.address)" +
 		" from Order o" +
 		" join o.member m" +
 		" join o.delivery d", OrderSimpleQueryDto.class)
 		.getResultList();
}
``` 
위와 같이 **DTO로 직접 조회**시 SELECT 절에서 원하는 데이터를 직접 선택하므로 DB 애플리케이션 네트웍 용량 최적화.

하지만 단점은 API 스펙에 맞춘 코드가 리포지토리에 들어가 재사용성이 떨어진다.

#### 쿼리 방식 선택 권장 순서
1. 우선 엔티티를 **DTO로 변환**하는 방법을 선택.
2. 필요하면 **페치 조인**으로 성능을 최적화.
3. 그래도 안되면 **DTO로 직접 조회**.
4. 최후의 방법은 JPA가 제공하는 네이티브 SQL이나 스프링 JDBC Template을 사용해서 **SQL을 직접 사용.**

<br>
<br>
<br>
<br>

## API 개발 고급 - 컬렉션 조회 최적화

#### 문제점
- 역시 엔티티를 직접 노출하여 좋지 않음
- DTO로 변환시 지연 로딩으로 인하여 N+1 문제 발생
- 페치 조인을 사용시 1대다 조인으로 같은 엔티티의 조회 수도 증가하여(다(N) 쪽이 기준이 되어버리기 때문) **페이징이 불가능**

<br>

### 한계 돌파

먼저 row수를 증가시키지 않는 ToOne(`OneToOne`, `ManyToOne`) 관계를 모두 페치조인 하고, 컬렉션은 지연 로딩으로 조회

```java
// ToOne 관계만 우선 모두 페치 조인으로 최적화
public List<Order> findAllWithMemberDelivery(int offset, int limit) {
 	return em.createQuery(
 		"select o from Order o" +
 		" join fetch o.member m" +
 		" join fetch o.delivery d", Order.class)
 		.setFirstResult(offset)
 		.setMaxResults(limit)
 		.getResultList();
}

@GetMapping("/api/v3.1/orders")
public List<OrderDto> ordersV3_page(@RequestParam(value = "offset", defaultValue = "0") int offset, 
	@RequestParam(value = "limit", defaultValue = "100") int limit) {
    
 	List<Order> orders = orderRepository.findAllWithMemberDelivery(offset, limit);
    
 	List<OrderDto> result = orders.stream()
 		.map(o -> new OrderDto(o))
 		.collect(toList());
        
 return result;
}
```

#### 장점
- 쿼리 호출 수가 1+N -> 1+1 로 최적화 되어 조인보다 DB 데이터 전송량이 최적화 된다.
- 페이징 가능

<br>

#### batch 설정

컬렉션 필드나 엔티티 클래스에 `@BatchSize` 를 적용하면 컬렉션이나, 프록시 객체를 한꺼번에 설정한 size 만큼 **IN 쿼리로 조회**

```yml
# application.yml 설정 파일에도 가능
spring:
 jpa:
   properties:
     hibernate:
       default_batch_fetch_size: 1000
```

<br>
<br>

### DTO 직접 조회

마찬가지로 한계 돌파를 이용한다. 

하지만 컬렉션마다 쿼리를 날려 1+N이 되므로 일대다 관계인 컬렉션은 **IN 절을 활용해서 메모리에 미리 조회해서 최적화**할 수 있다.

```java
public List<OrderQueryDto> findAllByDto_optimization() {
 
 //루트 조회(toOne 코드를 모두 한번에 조회)
 	List<OrderQueryDto> result = findOrders();
 
 //orderItem 컬렉션을 MAP 한방에 조회
 	Map<Long, List<OrderItemQueryDto>> orderItemMap = findOrderItemMap(toOrderIds(result));
 
 //루프를 돌면서 컬렉션 추가(추가 쿼리 실행X)
 	result.forEach(o -> o.setOrderItems(orderItemMap.get(o.getOrderId())));
 	return result;
}

private List<Long> toOrderIds(List<OrderQueryDto> result) {
 	return result.stream()
 		.map(o -> o.getOrderId())
 		.collect(Collectors.toList());
}

private Map<Long, List<OrderItemQueryDto>> findOrderItemMap(List<Long>
orderIds) {
 	List<OrderItemQueryDto> orderItems = em.createQuery(
 		"select new jpabook.jpashop.repository.order.query.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count)" +
 		" from OrderItem oi" +
 		" join oi.item i" + // in절을 활용하여 쿼리 1번
 		" where oi.order.id in :orderIds", OrderItemQueryDto.class)
 		.setParameter("orderIds", orderIds)
 		.getResultList();

return orderItems.stream()
	.collect(Collectors.groupingBy(OrderItemQueryDto::getOrderId));
}
```

<br>
<br>

### 결론 - 권장 순서

1. 엔티티 조회 방식으로 우선 접근

- 페치조인으로 쿼리 수를 최적화
- 컬렉션 최적화 - 페이징 필요시 `hibernate.default_batch_fetch_size` , `@BatchSize` 로 최적화 / 필요 없을시 페치 조인 사용

<br>

2. 엔티티 조회 방식으로 해결이 안되면 **DTO 조회 방식 사용**

3. DTO 조회 방식으로 해결이 안되면 **NativeSQL or 스프링 JdbcTemplate**