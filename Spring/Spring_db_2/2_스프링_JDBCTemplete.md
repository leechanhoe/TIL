![](https://velog.velcdn.com/images/dodo4723/post/b2cc4b34-cd1c-46d2-b744-31e6597b4624/image.png)

김영한 개발자님의 스프링 DB 2편 강의를 수강하고 중요한 내용을 정리했습니다.

# 2. 데이터 접근 기술 - 스프링

<br>

## 2.1. JdbcTemplate 소개와 설정
**JdbcTemplate**은 JDBC를 매우 편리하게 사용할 수 있게 도와준다.

JdbcTemplate은 템플릿 콜백 패턴을 사용해서, JDBC를 직접 사용할 때 발생하는 대부분의 반복 작업(아래 나열)을 대신 처리해준다.

> - 커넥션 획득
> - statement 를 준비하고 실행
> - 결과를 반복하도록 루프를 실행
> - 커넥션 종료, statement , resultset 종료
> - 트랜잭션 다루기 위한 커넥션 동기화
> - 예외 발생시 스프링 예외 변환기 실행


개발자는 SQL을 작성하고, 전달할 파리미터를 정의하고, 응답 값을 매핑하기만 하면 된다.

하지만 동적 SQL을 해결하기 어렵다는 단점이 있다.

<br>

## 2.2. JdbcTemplate 적용

아래는 `JdbcTemplate`을 적용하여 데이터를 저장하는 코드이다.

```java
public class JdbcTemplateItemRepositoryV1 implements ItemRepository {

 	private final JdbcTemplate template;
    //데이터소스(dataSource)가 필요하다.
 	public JdbcTemplateItemRepositoryV1(DataSource dataSource) {
 		this.template = new JdbcTemplate(dataSource);
}

 	@Override
 	public Item save(Item item) {
 		String sql = "insert into item (item_name, price, quantity) values (?, ?, ?)";
 
 		KeyHolder keyHolder = new GeneratedKeyHolder();
        //데이터를 변경할 때는 update() 를 사용
 		template.update(connection -> {
 		//자동 증가 키 - 데이터베이스가 PK인 ID를 대신 생성
 		PreparedStatement ps = connection.prepareStatement(sql, new String[]{"id"});
 
 		ps.setString(1, item.getItemName());
 		ps.setInt(2, item.getPrice());
 		ps.setInt(3, item.getQuantity());
 		return ps;}, keyHolder);
        
 		long key = keyHolder.getKey().longValue();
 		item.setId(key);
 		return item;
 	}
}
```

#### template.update()
- 데이터를 변경할 때 사용한다.
- `INSERT` , `UPDATE` , `DELETE` SQL에 사용한다.
- `template.update()` 의 반환 값은 int 인데, 영향 받은 로우 수를 반환한다

#### template.queryForObject()
- 결과 로우가 하나일 때 사용한다.
- `RowMapper` 는 데이터베이스의 반환 결과인 `ResultSet` 을 객체로 변환한다.

#### template.query()
- 결과가 하나 이상일 때 사용한다.
- 마찬가지로 `RowMapper` 는 데이터베이스의 반환 결과인 `ResultSet` 을 객체로 변환한다.

<br>

### 동적 쿼리 문제

결과를 검색하는 `findAll()` 에서 어려운 부분은 사용자가 검색하는 값에 따라서 실행하는 SQL이 동적으로 달려져야 한다는 점이다.

상품명(itemName)과 최대 가격(maxPrice) 을 고려해 검색할 경우, 사용 여부에 따라 4가지 경우가 생긴다.
```sql
select id, item_name, price, quantity from item
	where item_name like concat('%',?,'%')
	and price <= ?
```

<br>

### 데이터베이스 접근 설정

`src/main/resources/application.properties` 에 다음과 같이 설정하면 스프링 부트가 해당 설정을 사용해서 커넥션 풀과 DataSource , 트랜잭션 매니저를 스프링 빈으로 자동 등록한다. 
```
spring.profiles.active=local
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.username=sa
```

<br>


## 2.3. JdbcTemplate - 이름 지정 파라미터

`JdbcTemplate`을 기본으로 사용하면 파라미터를 순서대로 바인딩 한다.
```java
String sql = "update item set item_name=?, price=?, quantity=? where id=?";
template.update(sql,
 	itemName,
 	price,
 	quantity,
 	itemId);
```
여기서는 `itemName`, `price`, `quantity`, `id`가 sql에 순서대로 바인딩 된다. 순서를 바꾸면 심각한 문제가 발생한다.

가장 고치기 힘든 버그는 데이터베이스에 데이터가 잘못 들어가는 버그라고 한다.

**개발을 할 때는 코드를 몇줄 줄이는 편리함도 중요하지만, 모호함을 제거해서 코드를 명확하게 만드는 것이 유지보수 관점에서 매우 중요하다.**

### 이름 지정 바인딩
`JdbcTemplate`은 이런 문제를 보완하기 위해 `NamedParameterJdbcTemplate` 라는 이름을 지정해서
파라미터를 바인딩 하는 기능을 제공한다.

```java
public class JdbcTemplateItemRepositoryV2 implements ItemRepository {
 	private final NamedParameterJdbcTemplate template;
    
 	public JdbcTemplateItemRepositoryV2(DataSource dataSource) {
 		this.template = new NamedParameterJdbcTemplate(dataSource);
 }
 
 	@Override
 	public Item save(Item item) { // ? 대신에 :파라미터이름
 		String sql = "insert into item (item_name, price, quantity) " +
 	"values (:itemName, :price, :quantity)";
    
 		SqlParameterSource param = new BeanPropertySqlParameterSource(item);
 		KeyHolder keyHolder = new GeneratedKeyHolder();
        // 파라미터(param)를 전달하는 것을 확인할 수 있다.
        template.update(sql, param, keyHolder);
        
 		Long key = keyHolder.getKey().longValue();
 		item.setId(key);
 		return item;
	}
}
```

<br>

### 이름 지정 파라미터
파라미터를 전달하려면 Map 처럼 key , value 데이터 구조를 만들어서 전달해야 한다.

자주 사용하는 파라미터의 종류는 크게 3가지가 있다.

```java
//단순히 Map 사용
Map<String, Object> param = Map.of("id", id);

// Map 과 유사한데, SQL 타입을 지정할 수 있는 등 SQL에 좀 더 특화된 기능을 제공
SqlParameterSource param = new MapSqlParameterSource()
 	.addValue("itemName", updateParam.getItemName())
 
// 는 자바 빈 프로퍼티 규약을 통해서 자동으로 파라미터 객체를 생성한다.
//예) (`getXxx()` -> xxx, `getItemName()` -> itemName)
SqlParameterSource param = new BeanPropertySqlParameterSource(item);
```

<br>

### BeanPropertyRowMapper

`BeanPropertyRowMapper` 는 `ResultSet` 의 결과를 받아서 자바빈 규약에 맞추어 데이터를 변환한다.
```java
private RowMapper<Item> itemRowMapper() {
 	return BeanPropertyRowMapper.newInstance(Item.class); //camel 변환 지원
}
```

예를 들어서 데이터베이스에서 조회한 결과가 `select id, price`라고 하면 다음과 같은 코드를 작성해준다.
```java
Item item = new Item();
item.setId(rs.getLong("id"));
item.setPrice(rs.getInt("price"));
```

또한 언더스코어(item_name) 표기법을 카멜(itemName)로 자동
변환해준다.

<br>
<br>

## 2.4. JdbcTemplate - SimpleJdbcInsert

`JdbcTemplate`은 INSERT SQL를 직접 작성하지 않아도 되도록 `SimpleJdbcInsert` 라는 편리한 기능을 제공한다.

```java
@Repository
public class JdbcTemplateItemRepositoryV3 implements ItemRepository {

 	private final NamedParameterJdbcTemplate template;
 	private final SimpleJdbcInsert jdbcInsert;
    
 	public JdbcTemplateItemRepositoryV3(DataSource dataSource) {
 		this.template = new NamedParameterJdbcTemplate(dataSource);
 		this.jdbcInsert = new SimpleJdbcInsert(dataSource)
		.withTableName("item") // 데이터를 저장할 테이블 명을 지정한다.
 		.usingGeneratedKeyColumns("id"); // key 를 생성하는 PK 컬럼 명을 지정한다.
	// .usingColumns("item_name", "price", "quantity"); 
    //  INSERT SQL에 사용할 컬럼을 지정한다 - 생략 가능
 	}
    
    public Item save(Item item) {
 		SqlParameterSource param = new BeanPropertySqlParameterSource(item);
        //  INSERT SQL을 실행하고, 생성된 키 값도 편리하게 조회할 수 있다.
 		Number key = jdbcInsert.executeAndReturnKey(param);
 		item.setId(key.longValue());
 		return item;
	}
}
```
`SimpleJdbcInsert` 는 생성 시점에 데이터베이스 테이블의 메타 데이터를 조회한다. 따라서 어떤 컬럼이
있는지 확인 할 수 있으므로 `usingColumns` 을 생략할 수 있다.

<br>

## 2.5. JdbcTemplate 기능 정리

`JdbcTemplate`이 제공하는 주요 기능은 다음과 같다.

`JdbcTemplate` : 순서 기반 파라미터 바인딩을 지원한다.
`NamedParameterJdbcTemplate` : 이름 기반 파라미터 바인딩을 지원한다. (권장)
`SimpleJdbcInsert` : INSERT SQL을 편리하게 사용할 수 있다.
`SimpleJdbcCall` : 스토어드 프로시저를 편리하게 호출할 수 있다.

### 조회

목록 조회 - 객체
```java
private final RowMapper<Actor> actorRowMapper = (resultSet, rowNum) -> {
 	Actor actor = new Actor();
 	actor.setFirstName(resultSet.getString("first_name"));
 	actor.setLastName(resultSet.getString("last_name"));
 	return actor;
};

public List<Actor> findAllActors() {
 	return this.jdbcTemplate.query("select first_name, last_name from t_actor",actorRowMapper);
}
```
하나의 로우를 조회할 때 : `queryForObject()`
여러 로우를 조회할 때 : `query()`

### 변경(INSERT, UPDATE, DELETE)

데이터를 변경할 때는 jdbcTemplate.update() 를 사용하면 된다. 참고로 int 반환값을 반환하는데, SQL 실행 결과에 영향받은 로우 수를 반환한다.

#### 등록
```java
jdbcTemplate.update(
 	"insert into t_actor (first_name, last_name) values (?, ?)",
 	"Leonor", "Watling");
```
#### 수정
```java
jdbcTemplate.update(
 	"update t_actor set last_name = ? where id = ?",
 	"Banjo", 5276L);
```
#### 삭제
```java
jdbcTemplate.update(
 	"delete from t_actor where id = ?",
 	Long.valueOf(actorId));
```

### 기타 기능

임의의 SQL을 실행할 때는 execute() 를 사용하면 된다. 테이블을 생성하는 DDL에 사용할 수 있다.

#### DDL
```java
jdbcTemplate.execute("create table mytable (id integer, name varchar(100))");
```

<br>
<br>

## 2.6. 정리

`JdbcTemplate`은 동적 쿼리 문제를 해결하지 못한다. 그리고
SQL을 자바 코드로 작성하기 때문에 SQL 라인이 코드를 넘어갈 때 마다 문자 더하기를 해주어야 하는 단점도 있다.

동적 쿼리 문제를 해결하면서 동시에 SQL도 편리하게 작성할 수 있게 도와주는 기술이 바로 **MyBatis** 이다.