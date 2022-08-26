![](https://velog.velcdn.com/images/dodo4723/post/0f7908ca-17ad-4b55-a243-09ab2b0c57d7/image.png)

[김영한 개발자님의 스프링 DB 2편 강의](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-db-2#reviews)를 수강하고 중요한 내용을 정리했습니다.

<br>

## 4.1. MyBatis 소개

- `MyBatis`는 `JdbcTemplate`보다 더 많은 기능을 제공하는 SQL Mapper 이다.

- 기본적으로 `JdbcTemplate`이 제공하는 대부분의 기능을 제공한다.

- `JdbcTemplate`과 비교해서 `MyBatis`의 가장 매력적인 점은 SQL을 XML에 편리하게 작성할 수 있고 동적 쿼리를 매우 편리하게 작성할 수 있다는 점이다.

#### JdbcTemplate - SQL 여러줄

```java
String sql = "update item " +
 "set item_name=:itemName, price=:price, quantity=:quantity " +
 "where id=:id";
```

#### MyBatis - SQL 여러줄
MyBatis는 XML에 작성하기 때문에 라인이 길어져도 문자 더하기에 대한 불편함이 없다.
```xml
<update id="update">
 	update item
 	set item_name=#{itemName},
 	price=#{price},
 	quantity=#{quantity}
 	where id = #{id}
</update>
```

#### MyBatis - 동적 쿼리

`JdbcTemplate`은 자바 코드로 직접 동적 쿼리를 작성해야 하지만, `MyBatis`는 동적 쿼리를 매우 편리하게 작성할 수 있는 다양한 기능들을 제공해준다.

```xml
<select id="findAll" resultType="Item">
 	select id, item_name, price, quantity
 	from item
 	<where>
 		<if test="itemName != null and itemName != ''">
 			and item_name like concat('%',#{itemName},'%')
		</if>
		<if test="maxPrice != null">
 			and price &lt;= #{maxPrice}
 		</if>
 	</where>
</select>
```

<br>

## 4.2. MyBatis 적용

`MyBatis`를 적용하기 위해서는 매핑 XML을 호출해주는 매퍼 인터페이스가 필요하다.
```java
@Mapper // 이걸 해줘야 MyBatis에서 인식할 수 있다.
public interface ItemMapper {
 	void save(Item item);
    
 	void update(@Param("id") Long id, @Param("updateParam") ItemUpdateDto updateParam);
    
 	Optional<Item> findById(Long id);
    
 	List<Item> findAll(ItemSearchCond itemSearch);
}
```
실행할 SQL이 있는 XML 매핑 파일을 `src/main/resources` 하위에 패키지 위치를 맞춰서 만들어야 한다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 	"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="hello.itemservice.repository.mybatis.ItemMapper">
  
<!-- Insert가 끝나면 item 객체의 id 속성에 데이터베이스가 생성한 키 값이 입력됨. -->
 	<insert id="save" useGeneratedKeys="true" keyProperty="id">
 		insert into item (item_name, price, quantity)
 		values (#{itemName}, #{price}, #{quantity})
 	</insert>
  
<!-- 파라미터가 2개 이상이면 인터페이스에 @Param 으로 이름을 지정-->
 	<update id="update"> 
 		update item
 		set item_name=#{updateParam.itemName},
 		price=#{updateParam.price},
 		quantity=#{updateParam.quantity}
 		where id = #{id}
 	</update>
  
 	<select id="findById" resultType="Item">
 		select id, item_name, price, quantity
 		from item
 		where id = #{id}
 	</select>
  
  <!-- SQL의 결과를 편리하게 객체로 바로 변환 -->
 	<select id="findAll" resultType="Item">
 		select id, item_name, price, quantity
 		from item
 		<where>
 			<if test="itemName != null and itemName != ''">
 				and item_name like concat('%',#{itemName},'%')
 			</if>
 			<if test="maxPrice != null">
            <!-- < , > 같은 특수문자 사용불가-->
            <!-- CDATA나  < : &lt; > : &gt; & : &amp; 사용-->
            <![CDATA[
 				and price <= #{maxPrice}
			]]>
 				and price &lt;= #{maxPrice}
 			</if>
 		</where>
 	</select>
</mapper>
```
- `id` 에는 매퍼 인터페이스에 설정한 메서드 이름을 지정하면 된다.

- 파라미터는 `#{}` 문법을 사용하면 된다. 그리고 매퍼에서 넘긴 객체의 프로퍼티 이름을 적어주면 된다.

<br>

### 분석

`ItemMapper` 매퍼 인터페이스의 구현체가 없는데 어떻게 동작했을까?

![](https://velog.velcdn.com/images/dodo4723/post/e5acdb2d-ff35-407d-a9b0-e33dfa0ad478/image.png)

 애플리케이션 로딩 시점에 MyBatis 스프링 연동 모듈은 `@Mapper` 가 붙어있는 인터페이스를 조사하여 프록시 기술을 사용해서 `ItemMapper` 인터페이스의 구현체를 만든다.
 
매퍼 구현체는 예외 변환까지 처리해준다. MyBatis에서 발생한 예외를 스프링 예외 추상화인 `DataAccessException` 에 맞게 변환해서 반환해준다.

<br>

## 4.3. MyBatis 기능 정리

마이바티스가 제공하는 최고의 기능이자 마이바티스를 사용하는 이유는 바로 **동적 SQL 기능** 때문이다.

### 1. if
```xml
SELECT * FROM BLOG
WHERE state = ‘ACTIVE’
<if test="title != null">
 	AND title like #{title}
</if>
```
- 해당 조건에 따라 값을 추가할지 말지 판단한다.

<br>

### 2. choose, when, otherwise

```xml
SELECT * FROM BLOG WHERE state = ‘ACTIVE’
<choose>
	 <when test="title != null">
 		AND title like #{title}
 	</when>
 	<when test="author != null and author.name != null">
 	AND author_name like #{author.name}
	 </when>
 	<otherwise>
 		AND featured = 1
 	</otherwise>
</choose>
```
- `switch` 구문과 유사하다.
 
<br>

### 3. where

```xml
 SELECT * FROM BLOG
 <where>
 	<if test="state != null">
 		state = #{state}
 	</if>
	 <if test="title != null">
 		AND title like #{title}
 	</if>
</where>
```
- `<where>` 는 문장이 없으면 `where` 를 추가하지 않고 있으면 추가한다.
- 만약 `and` 가 먼저 시작된다면 `and` 를 지운다.

<br>

### 4. foreach
```xml
SELECT *
	FROM POST P
 	<where>
 	<foreach item="item" index="index" collection="list"
 		open="ID in (" separator="," close=")" nullable="true">
 	#{item}
 	</foreach>
</where>
```
- 컬렉션을 반복 처리할 때 파라미터로 `List`를 전달하면 된다.
- `where in (1,2,3,4,5,6)` 와 같은 문장을 쉽게 완성할 수 있다.

<br>

### 5. 애노테이션으로 SQL 작성
```java
@Select("select id, item_name, price, quantity from item where id=#{id}")
Optional<Item> findById(Long id);
```
- `@Insert` , `@Update` , `@Delete` , `@Select` 기능이 제공된다.
- 이 경우 XML에는 `<select id="findById"> ~ </select>` 는 제거해야 한다.
- 동적 SQL이 해결되지 않으므로 간단한 경우에만 사용한다.

<br>

### 6. 문자열 대체(String Substitution)

```java
@Select("select * from user where ${column} = #{value}")
User findByColumn(@Param("column") String column, @Param("value") String value);
```
- 는파라미터 바인딩이 아니라 문자 그대로를 처리하고 싶은 경우 `${}` 를 사용하면 된다.