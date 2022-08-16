![](https://velog.velcdn.com/images/dodo4723/post/31075621-6ced-4ab1-be32-d3f0b16739e1/image.png)

김영한 개발자님의 `스프링 DB 1편 - 데이터 접근 핵심 원리` 강의를 수강하고 정리한 내용이다.

# 1. JDBC 이해

<br>

## 1.1. JDBC 이해

### JDBC 등장 이유
애플리케이션을 개발할 때 중요한 데이터는 대부분 데이터베이스에 보관한다.

#### 서버가 데이터베이스를 사용하는 과정

- **1. 커넥션 연결**: 주로 TCP/IP를 사용해서 커넥션을 연결한다.

- **2. SQL 전달**: 애플리케이션 서버는 DB가 이해할 수 있는 SQL을 연결된 커넥션을 통해 DB에 전달한다.

- **3. 결과 응답**: DB는 전달된 SQL을 수행하고 그 결과를 응답한다. 애플리케이션 서버는 응답 결과를 활용한다.

#### 문제점
- 데이터베이스를 다른 종류의 데이터베이스로 변경하면 애플리케이션 서버에 개발된 데이터베이스 사용 코드도 함께 변경해야 함.

- 개발자가 각각의 데이터베이스마다 커넥션 연결, SQL 전달, 그리고 그 결과를 응답 받는 방법을 새로 학습해야 함.

이런 문제를 해결하기 위해 JDBC라는 자바 표준이 등장한다.
![](https://velog.velcdn.com/images/dodo4723/post/00056042-5b31-473e-ab11-7af7b41c2257/image.png)

이 JDBC 인터페이스를 각각의 DB 벤더 (회사)에서 자신의 DB에 맞도록 구현해서 라이브러리로 제공하는데, 이것을 **JDBC 드라이버**라 한다.

<br>

## 1.2. JDBC와 최신 데이터 접근 기술

JDBC를 편리하게 사용하는 다양한 기술 대표 2가지

### SQL Mapper

SQL 응답 결과를 객체로 편리하게 변환하고 JDBC의 반복 코드를 제거해주지만 개발자가 SQL을 직접 작성해야한다.

예) 스프링 JdbcTemplate, MyBatis

### ORM 기술

ORM은 객체를 관계형 데이터베이스 테이블과 매핑해주는 기술이다. 이 기술 덕분에 개발자는 반복적인 SQL을 직접 작성하지 않고, ORM 기술이 개발자 대신에 SQL을 동적으로 만들어 실행해준다. 

추가로 각각의 데이터베이스마다 다른 SQL을 사용하는 문제도 중간에서 해결해준다.

대표 기술: JPA, 하이버네이트, 이클립스링크

## 1.3. 데이터베이스 연결
```java
public static Connection getConnection() {
	try {
		Connection connection = DriverManager.getConnection(URL, USERNAME, PASSWORD);
		log.info("get connection={}, class={}", connection, connection.getClass());
		return connection;
 	} catch (SQLException e) {
 		throw new IllegalStateException(e);
	}
}
 ```
 라이브러리에 있는 데이터베이스 드라이버를 찾아서 해당 드라이버가 제공하는 커넥션을 반환해준다.
 
<br>

## 1.4. JDBC 개발

#### member 테이블
```sql
drop table member if exists cascade;
create table member (
 member_id varchar(10),
 money integer not null default 0,
 primary key (member_id)
);
```
이런 것이 있다고만 알고 넘어가자
```java
public Member save(Member member) throws SQLException {
	// 데이터베이스에 전달할 SQL을 정의한다.
	String sql = "insert into member(member_id, money) values(?, ?)";
	Connection con = null;
	PreparedStatement pstmt = null;
 	try {
 		con = getConnection();
        // 데이터베이스에 전달할 SQL과 파라미터로 전달할 데이터들을 준비한다.
 		pstmt = con.prepareStatement(sql);
        // SQL의 첫번째 ? 에 값을 지정한다. 
 		pstmt.setString(1, member.getMemberId());
 		pstmt.setInt(2, member.getMoney());
        // 준비된 SQL을 커넥션을 통해 실제 데이터베이스에 전달한다.
        // 영향받은 DB row 수를 반환한다.
		pstmt.executeUpdate();
    	return member;
 	} catch (SQLException e) {
 		log.error("db error", e);
 		throw e;
 	} finally {
 		close(con, pstmt, null);
 	}
 }
 
 private void close(Connection con, Statement stmt, ResultSet rs) {
	if (rs != null) {
 		try {
 			rs.close();
 		} catch (SQLException e) {
 		log.info("error", e);
		}
	}
	if (stmt != null) {
 		try {
 			stmt.close();
 		} catch (SQLException e) {
 			log.info("error", e);
 		}
 	}
 	if (con != null) {
 		try {
 			con.close();
 		} catch (SQLException e) {
 			log.info("error", e);
 		}
 	}
}
 
private Connection getConnection() {
 	return DBConnectionUtil.getConnection();
}
```
#### 리소스 정리
쿼리를 실행하고 나면 리소스를 정리해야 한다. 여기서는 `Connection` , `PreparedStatement` 를 사용했다. 리소스를 정리할 때는 항상 역순으로 해야한다.

### ResultSet
`ResultSet` 은 다음과 같이 생긴 데이터 구조이다.

![](https://velog.velcdn.com/images/dodo4723/post/7aa82ba9-926a-40da-a6de-ecffdab2b935/image.png)


`ResultSet` 내부에 있는 커서(cursor)를 이동해서 다음 데이터를 조회할 수 있다.

`rs.next()` : 이것을 호출하면 커서가 다음으로 이동한다. 참고로 최초의 커서는 데이터를 가리키고 있지 않기 때문에 `rs.next()` 를 최초 한번은 호출해야 데이터를 조회할 수 있다.

`rs.getInt("money")` : 현재 커서가 가리키고 있는 위치의 money 데이터를 int 타입으로 반환한다.