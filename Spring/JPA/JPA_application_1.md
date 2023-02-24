![](https://velog.velcdn.com/images/dodo4723/post/7ffd4b81-8ae0-4c05-a993-1136202fd650/image.png)

김영한 개발자님의 [실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1/dashboard) 강의를 수강하고 중요한 점이나 인상깊었던 점들을 정리했습니다.

<br>
<br>
<br>
<br>

## 도메인 분석 설계

개발을 하는 것만큼, 설계도 그 이상으로 중요한 것 같습니다.


#### 엔티티 분석
![](https://velog.velcdn.com/images/dodo4723/post/79a2a507-cba0-4411-96ad-7d79d68af02d/image.png)

![](https://velog.velcdn.com/images/dodo4723/post/f4e5392e-ca70-4a47-8577-f38a7c38ec72/image.png)

#### 살펴볼 점
- `MEMBER`와 `DELIVERY` 엔티티의 `Address` 임베디드 타입 정보가 테이블에 그대로 들어감
- 카테고리와 상품은 `@ManyToMany` 지만, 중간 테이블에 컬럼을 추가할 수 없고, 세밀한 쿼리가 어려워 실무에서는 사용하면 안됨
- 외래 키가 있는 곳을 연관관계의 주인으로 정해야함
- 값 타입은 변경 불가능하게 설계해야 함.
- 엔티티에는 가급적 `Setter` 사용 X
- 컬렉션은 필드에서 바로 초기화하는 것이 `null` 문제에서 안전

<br>
<br>
<br>
<br>

## 변경 감지와 병합

**`준영속 엔티티`** : **영속성 컨텍스트가 더는 관리하지 않는 엔티티**. 임의로 만들어낸 엔티티도 DB에 한번 저장되어서 기존 식별자를 가지고 있으면 준영속 엔티티로 볼 수 있다.

### 준영속 엔티티를 수정하는 2가지 방법

#### 1. 변경 감지 기능 사용

```java
@Transactional
void update(Item itemParam) { //itemParam: 파리미터로 넘어온 준영속 상태의 엔티티
 	Item findItem = em.find(Item.class, itemParam.getId()); //같은 엔티티를 조회한다.
 	findItem.setPrice(itemParam.getPrice()); //데이터를 수정한다.
    
//트랜잭션 커밋 시점에 변경 감지(Dirty Checking)가 동작해서 데이터베이스에 UPDATE SQL 실행
}
```

<br>

#### 2. 병합 사용
```java
@Transactional
void update(Item itemParam) { //itemParam: 파리미터로 넘어온 준영속 상태의 엔티티
 	Item mergeItem = em.merge(itemParam);
}
```
![](https://velog.velcdn.com/images/dodo4723/post/4f138008-ca0e-4065-a651-c5bf466939cd/image.png)

**준영속 엔티티의 식별자 값으로 조회해온 영속 엔티티에 준영속 엔티티의 값을 밀어 넣는다.** 이후 변경 감지가 동작해 변경됨

#### 주의

- 병합은 변경 감지와 달리 모든 속성이 변경되어 값이 없으면 `null`로 업데이트 될 위험이 있다.
- 컨트롤러에서 어설프게 엔티티를 생성하면 안됨
- 엔티티를 변경할 때는 항상 변경 감지를 사용
- 트랜잭션이 있는 서비스 계층에 식별자(id)와 변경할 데이터를 명확하게 전달