## 목차

[1. 프로젝트 환경설정](#1-프로젝트-환경설정)

[2. 스프링 웹 개발 기초](#2-스프링-웹-개발-기초)

[3. 회원 관리 예제 - 벡엔드 개발](#3-회원-관리-예제---백엔드-개발)

[4. 스프링 빈과 의존관계](#4-스프링-빈과-의존관계)

[5. 스프링 DB 접근 기술](#5스프링-db-접근-기술)

[6. AOP](#6aop)

[7. 결과물](#7-결과물)

[8. 느낀점](#8-느낀-점)

![](https://velog.velcdn.com/images/dodo4723/post/adcd8a55-5f5e-408a-904c-c82409878a7d/image.png)

## 1. 프로젝트 환경설정
<br/>

### Dependencies
**Spring Web** : 웹프로젝트 만들 때 사용하고 아파치톰캣 컨테이너 내장
**Thymeleaf** : HTML을 화면에 출력하는 템플릿엔진 중 하나
<br/>

### 프로젝트 폴더
![](https://velog.velcdn.com/images/dodo4723/post/35d21404-60cf-4c63-a14f-365b8fcd7d14/image.png)
**.idea** : 인텔리제이 설정파일
**src** : 하위에 main, test이 나뉘어져있음(요새 트렌드)
**src/main/java** : 실제 자바 소스
**src/main/resources** : 자바파일 제외한 설정 파일 등
**build.gradle : gradle** 설정파일
<br/>

### 프로젝트 첫 실행

프로젝트 실행은 HelloSpringApplication.java를 실행하면 main()에서 SpringApplication.run()이 자기 자신(HelloSpringApplication)을 띄워준다.
![](https://velog.velcdn.com/images/dodo4723/post/72b0c995-681d-4440-be37-52648e1124f5/image.png)

정상적으로 실행되면 localhost:8080에 들어갈 수 있다. 다음과 같이 White Error페이지가 나오면 설정 성공이다.
<br/>
<br/>

## 2. 스프링 웹 개발 기초

### 1)정적 컨텐츠
resources/static 하위에 html파일을 생성하면 동적인 동작은 없는 단순한 html 컨텐츠를 반환해준다. 하위에 hello-static.html을 만들었다면, localhost:8080/hello-static.html로 접근할 수 있다.

그리고 클라이언트가 요청시, 1순위로 컨트롤러에 hello-static.html 관련 컨트롤러 메소드가 있는지 찾는다. 없으면 2순위로 resources: static/hello-static.html을 찾는다.
<p align="center" style="color:gray">
  <img src="https://velog.velcdn.com/images/dodo4723/post/a0624763-bc65-4575-96c8-743cb490ccfa/image.png
"/>
  김영한 개발자님 강의 자료
</p>

### 2)MVC와 템플릿 엔진


@RequestParam의 required는 true가 default이기 때문에 따로 설정을 안 했다면 값이 있어야 한다.
```java
    @GetMapping("hello-mvc") // 화면을 템플릿 엔진에서 조작함
    public String helloMvc(@RequestParam("name") String name, Model model){
        model.addAttribute("name", name);
        return "hello-template";
    }
```
hello-mvc @GetMapping("hello-mvc") 컨트롤러를 만들고 localhost:8080/hello-mvc?name=spring으로 요청을 보낸다면 아래와 같은 프로세스로 동작한다.
![](https://velog.velcdn.com/images/dodo4723/post/76b3223f-fb8c-4d75-9af2-fef37c09fdfa/image.png)

### 3)API

정적컨텐츠 방식을 제외한 웹개발 방식은 위에서 설명한 MVC, 템플릿 엔진 방식과 API방식으로 크게 2가지라고 볼 수 있다.

템플릿엔진 방식을 사용하면 데이터를 VIewResolver를 통해 HTML로 내려주지만 API 방식을 사용하면 HttpMessageConverter가 작동하여 String,JSON 등의 형식으로 데이터만 http body에 담아 내려주게 된다.
```java
    @GetMapping("hello-string")
    @ResponseBody // API 방식 - http 에 헤더부 바디부가 있는데 응답 바디부에 내용을 직접 넣어주겠다는 뜻
    public String helloString(@RequestParam("name") String name){
        return "hello " + name; // 얘는 화면이 view 를 안거치고 그대로 감
    }

    @GetMapping("hello-api") // API 방식 보통은 실무에서 이렇게함
    @ResponseBody // 요즘은 기본으로 json으로 반환
    public Hello helloApi(@RequestParam("name") String name){
        Hello hello = new Hello(); // 자동완성 : ctr + shift + Enter
        hello.setName(name);
        return hello;
    }
```
API방식을 사용하려면 @ResponseBody 어노테이션을 사용한다. 이 어노테이션을 사용하면 데이터를 http body에 담아 보낸다.

컨트롤러 전체를 API방식으로 사용하려면 컨트롤러에 @RestController 어노테이션을 사용하면 된다. 이 어노테이션을 사용하면 @ResponseBody 어노테이션을 붙이지 않아도 http의 바디부에 데이터를 넣어 보낼 수 있다.
<br/>
<br/>

## 3. 회원 관리 예제 - 백엔드 개발

### 1) 비즈니스 요구사항 정리
 
간단한 회원관리 예제를 만든다. 
회원등록과 조회 기능을 만든다. 데이터 저장소는 선정되지 않았다는 가정하에 메모리에 데이터를 저장하고 조회하는 방식으로 만든다.
![](https://velog.velcdn.com/images/dodo4723/post/d7777f3e-78c8-4ab0-8722-9b0fcf5792f7/image.png)

### 2) 회원 도메인과 리포지토리 만들기

리포지토리 만들 때 임시적으로 메모리에 저장할 방식을 사용한다. 
MemberRepository를 구현한 MemoryMemberRepository를 만든다. static한 HashMap 인스턴스를 생성하여 임시저장소를 만든다.

```java
public class MemoryMemberRepository implements MemberRepository{
    private static Map<Long, Member> store = new HashMap<>();
    private static long sequence = 0L;

    @Override
    public Member save(Member member) {
        member.setId(++sequence);
        store.put(member.getId(), member);
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        return Optional.ofNullable(store.get(id)); // Optional 하면 널이나와도 감쌀수있다
    }

    @Override
    public Optional<Member> findByName(String name) {
        return store.values().stream()
                .filter(member -> member.getName().equals(name))
                .findAny();
    }

    @Override
    public List<Member> findAll() { // 자바 실무에선 리스트를 많이씀
        return new ArrayList<>(store.values());
    }

    public void clearStore() {
        store.clear();
    }
}
```

### 3) 회원 리포지토리 테스트 케이스 작성
테스트 케이스 작성시 테스트할 데이터를 만들고, 실제로 테스트할 메소드를 호출한 후 assertThat이나 Assertions.assertEquals()를 사용하여 검증한다.

테스트끼리는 순서에 의존적이면 안 된다.
```java
public class MemoryMemberRepositoryTest {

    MemoryMemberRepository repository = new MemoryMemberRepository();

    @AfterEach
    public void afterEach(){ // 한 함수 테스트후마다 실행되는 콜백함수
        repository.clearStore();
    }

    @Test
    public  void save(){
        Member member = new Member();
        member.setName("spring");

        repository.save(member);

        Member result = repository.findById((member.getId())).get();
        assertThat(member).isEqualTo(result);
    }

    @Test
    public void findByName(){
        Member member1 = new Member();
        member1.setName("spring1");
        repository.save(member1);

        Member member2 = new Member(); // Shift + f6 누르면 한번에 여러게 바꿀 수 있음
        member2.setName("spring2");
        repository.save(member2);

        Member result = repository.findByName("spring1").get(); // get을하면 Optional을 깔수있음

        assertThat(result).isEqualTo(member1);
    }

    @Test
    public void findAll(){
        Member member1 = new Member();
        member1.setName("spring1");
        repository.save(member1);

        Member member2 = new Member();
        member2.setName("spring2");
        repository.save(member2);

        List<Member> result = repository.findAll();

        assertThat(result.size()).isEqualTo(2);
    }
}
```
MemoryMemberRepository에 store.clear()할 수 있는 메소드(clearStore())를 만든 후, @AfterEach 어노테이션을 적용한 메소드를 만든다. @AfterEach를 붙이면 각 메소드 이후 콜백하는 메소드 함수를 만들 수 있다. 
<br/>
<br/>

## 4. 스프링 빈과 의존관계

### 1)컴포넌트 스캔과 자동 의존관계 설정

`private final MemberService = new MemberService();`
이렇게 초기화 하는 것 보다는 MemberService를 스프링컨테이너가 관리하는 빈으로 만들어 공유하여 사용하도록 한다. 
그러기 위해 의존하는 주체와 의존 당하는 주체 모두 스프링컨테이너가 관리하는 '빈'이 되어야 한다.

MemberController위에 @Controller를 붙여 스프링컨테이너 관리 빈으로 만든다. 그리고 다음과 같이 @Autowired 어노테이션을 붙여 초기화한다.

@Autowired : 스프링 빈으로 관리되고 있는 객체를 DI(의존성주입) 해줌

또한, DI주입 방식에는 3가지가 있다. (setter방식, 필드방식, 생성자방식)
```java
@Controller // 얘는 springConfig에 안넣어도 어차피 컴포넌트 스캔됨 그래서 autowired가능
public class MemberController {

    //@Autowired private MemberService memberService;  필드주입방식
    private final MemberService memberService;

//    public void setMemberService(MemberService memberService) { setter 방식 근데 바꿀일없어서 거의안씀
//    public 이라 열려있어서 불필요하게 호출될수도있음
//        this.memberService = memberService;
//    }

    @Autowired // 스프링 컨테이너에서 멤버서비스를 가져와서 연결  // 생성자 방식
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }
```

### 2)자바 코드로 직접 스프링 빈 등록하기

컴포넌트 스캔 방식 이외에도 자바설정파일을 만들어서 빈 등록이 가능하다. 기존 @Service, @Repository, @Autowired를 제거한 후, 다음과 같은 자바 설정파일을 만든다.
![](https://velog.velcdn.com/images/dodo4723/post/e63be876-c86a-4726-92af-640a8b2728e0/image.png)
<br/>
## 5.스프링 DB 접근 기술

메모리DB가 아닌 H2 DB를 사용하여 순수JDBC, JdbcTemplate, JPA, Spring JPA 순으로 DB접근 기술을 알아본다.

DB는 H2로 사용한다. H2는 자바기반 RDMBS이며 저용량이고 브라우저 콘솔을 지원한다. 테스트 DB로 많이 사용된다고 한다.
<p align="center" style="color:gray">
  <img src="https://velog.velcdn.com/images/dodo4723/post/8eb46af4-d01e-4e24-8f6a-a40ad61d775d/image.png
"/>
  H2 데이터베이스
</p>


### 1) 순수 JDBC

기존 메모리방식DB가 아닌 JDBC방식을 사용하도록 바꾼다. 단순히 JDBC API로만 코딩하여 사용하며 과거에 많이 사용한 방식이다. 
```java
 @Override // 순수 JDBC로 구현한 회원관리 조회 메소드
    public Optional<Member> findById(Long id) {
        String sql = "select * from member where id = ?";
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        try {
            conn = getConnection();
            pstmt = conn.prepareStatement(sql);
            pstmt.setLong(1, id);
            rs = pstmt.executeQuery();
            if(rs.next()) {
                Member member = new Member();
                member.setId(rs.getLong("id"));
                member.setName(rs.getString("name"));
                return Optional.of(member);
            } else {
                return Optional.empty();
            }
        } catch (Exception e) {
            throw new IllegalStateException(e);
        } finally {
            close(conn, pstmt, rs);
        }
    }
```
### 2)스프링 JdbcTemplate

순수 JDBC에서 반복된 코드를 제거해주는 장점이 있다.
```java
   @Override
    public Optional<Member> findById(Long id) {
        List<Member> result = jdbcTemplate.query("select * from member where id = ?", memberRowMapper(), id);
        return result.stream().findAny();
    }
```

### 3) JPA(Java Persistence API)

기존 반복 코드 뿐만 아니라, 기본적인 SQL도 없이 JPA가 직접 만들어서 실행한다.

SQL과 데이터 중심 설계에서 객체 중심 설계로 패러다임 전환 가능
JPA는 자파 표준 인터페이스이며 이를 벤더별로 구현한 구현체가 Hibernate(주로 사용)이다.

EntityManager를 사용하여 crud처리를 기본적으로 한다. 
또한 findByName나 findAll을 사용할 때 JPQL을 사용하는데 JPQL은 SQL을 테이블대상이 아닌 객체(Entity)대상으로 한다.
```java
public class JpaMemberRepository implements MemberRepository{
    
    private final EntityManager em; // jpa는 이걸로 모든게 동작 jpa쓸려면 이걸 주입받아야함

    public JpaMemberRepository(EntityManager em) {
        this.em = em;
    }

    @Override
    public Member save(Member member) {
        em.persist(member);
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        Member member = em.find((Member.class), id); // select문 나감
        return Optional.ofNullable(member);
    }//...
```
마지막으로 JPA사용시 Service 클래스 상단에 @Transactional을 붙여줘야 한다. JPA를 통한 모든 데이터 변경은 트랜잭션 안에서 실행해야 한다.

### 4) 스프링 데이터 JPA

스프링 데이터 JPA를 사용하면 리포지토리 구현 클래스 없이 인터페이스만으로 개발을 할 수 있다!!
CRUD의 기본적 기능을 스프링 데이터 JPA가 모d두 제공한다. 다음과 같이 Repository 인터페이스를 만들 수 있다.
```java
 // extends JapRepo를 하면 스프링에서 자동으로 구현체를 만들어서 빈에 넣어줌
public interface SpringDataJpaMemberRepository extends JpaRepository<Member, Long>, MemberRepository {

    //select m from Member m where m.name = ?  findBy"Name"을 보고 name 을 찾아줌
    @Override
    Optional<Member> findByName(String name);
}
```
JpaRepository를 상속하면 자동으로 스프링컨테이너가 MemberRepository 구현체를 빈으로 만들어 관리한다고 한다.
```java
@Configuration
public class SpringConfig {

    private final MemberRepository memberRepository;

    @Autowired
    public SpringConfig(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
```
스프링 빈 설정관계를 만들어주는 SpringConfig에서 MemberRepository 구현체를 스프링컨테이너가 관리하고 있기 때문에 생성자방식으로 의존성주입을 해줄 수 있다. 생성자가 1개이기 때문에 굳이 @Autowired를 붙일 필요가 없다.

스프링데이터 JPA의 기본 인터페이스인 JpaRepository 인터페이스는 Paging관련 인터페이스, CRUD관련인터페이스를 다중상속하고 있다. 그렇기 때문에 기본적인 CRUD 등을 쿼리나 구현메소드 없이 바로 동작하게 해준다. 복잡한 동적쿼리는 이후 QueryDsl을 사용한다고 한다.
<br/>

또한 기본적으로 제공하는 메소드 이외에 비지니스 로직별로 다른 메소드가 있는 경우 표기룰만 맞춰준다면 **리플렉션 기술로 알아서 SQL을 만들어준다.**

위에서 했던 findByName은 사실 기본적인 스프링데이터JPA 제공 메소드가 아니다. 리플렉션 기술로 findByName은 다음처럼 해석되어 돌아간다.
`select m from member m where m.name = ?`

마지막으로 JPA만으로 꼭 모든 것을 할 필요 없게하도록 기본적인 SQL쿼리 방식을 사용할 수 있게 열어놨다고 한다. 그러나 대부분의 데이터접근 메소드는 JPA로 구현이 가능하다고 한다.
<br/>
<br/>

## 6.AOP
 

### 1)AOP가 필요한 상황

지금까지 만든 서비스들(회원가입, 회원조회 등)의 동작 시간 측정 로직을 서비스 메소드마다 달아야 하는 경우 유지보수가 힘들 것이다. 이를 위해 AOP라는 기술이 필요하다.

시간 측정 기술과 같은 부수적인 기능을 공통관심사항이라고 하고 서비스 자체의 핵심 기능을 핵심관심사항이라고 한다.

스프링이 제공하는 AOP 기술을 사용하면 공통적으로 사용되는 공통관심사항을 쉽게 관리할 수 있다. 
<br/>

### 2)AOP 적용

AOP를 사용하면 위에서 언급한 시간측정 로직을 한 곳에 모아 원하는 곳에 적용할 수 있다. 아래와 같이 aop패키지에 AOP 클래스를 만든다.
```java
@Aspect
@Component
public class TimeTraceAop { // 중간에 인터셉트해서 컨트롤 할 수 있는 기능이 AOP

    @Around("execution(* dodo.hellospring..*(..))")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable{
        long start = System.currentTimeMillis();
        System.out.println("START: " + joinPoint.toString());
        try{
            return joinPoint.proceed();
        } finally {
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("END" + joinPoint.toString() + " " + timeMs + "ms");
        }
    }
}
```
@Aspect, @Component 어노테이션을 통해 스프링 빈으로 등록하고 AOP기술을 사용한다. @Around는 어느 시점에 적용할지에 대해 패턴으로 정의하는 어노테이션이다. joinPoint는 실제 핵심관심사항 메소드가 동작하는 부분이다.

결과적으로 위와 같이 AOP기술을 사용하면 핵심관심사항과 공통관심사항을 분리할 수 있고, 공통관심사항 수정 요청이 들어왔을 때 위의 AOP 컴포넌트만 수정해주면 된다.

AOP는 내부적으로 프록시가 사용된다.
![](https://velog.velcdn.com/images/dodo4723/post/afdf5d11-2243-488a-84cc-7634beb0508e/image.png)
<br/>
<br/>

## 7. 결과물

<p align="center" style="color:gray">
  <img src="https://velog.velcdn.com/images/dodo4723/post/a2020124-ea40-45fa-bcc1-036bdb97430b/image.png
"/>
  첫 화면에서 회원 가입을 누르고
</p>
<p align="center" style="color:gray">
  <img src="https://velog.velcdn.com/images/dodo4723/post/98cd65f2-fd72-4ce1-ae6d-39dc71b56873/image.png
"/>
  이름을 등록하면
</p>
<p align="center" style="color:gray">
  <img src="https://velog.velcdn.com/images/dodo4723/post/89b55a2d-7cef-4177-aeb7-25870f94959f/image.png
"/>
  회원 목록에 적용되고.
</p>
<p align="center" style="color:gray">
  <img src="https://velog.velcdn.com/images/dodo4723/post/1fc63ebc-816d-4018-9e05-5a8ef060d37d/image.png
"/>
  데이터베이스에도 적용이 잘 된다.
</p>
<br/>
<br/>

## 8. 느낀 점
반년 전, 백엔드 개발자를 도전해보겠다고 결심한 후, 어떻게 어떤 순서로 공부해야 할지 정보를 수집 중에 김영한 개발자님의 스프링 강의가 있다는 것을 알았다. 매우 흥미가 생겨서 빨리 들어보고 싶었다.
<br/>
하지만, 이 강의를 듣기 전에 먼저 공부해야할 것들이 있다고 생각했다. 

군대에서부터 CS과목들(네트워크, 운영체제, 자료구조, 데이터베이스등) 이론공부를 시작하여,

전역 후 기본적인 프론트엔드 지식(HTML, CSS, JavaScript, React) 클론코딩을 거쳐서, 기본적인 알고리즘(백준 Gold까지)공부 후에,

드디어 백엔드 강의를 수강하게 되었다. 이전까지 공부했던거와는 달리 실무와 가까운 내용들을 접하게 되어 흥미로웠다. 특히, 말로만 듣던 데이터베이스를 다루는 것이 제일 흥미로웠다.

하지만 방대한 스프링의 내용과 아직 익숙하지 않은 구조가 어렵게 다가온다. 그나마 입대 전에 Java와 비슷한 C#(유니티)을 꽤 사용해본 것이 도움이 많이 되었다.
<br/>
복학하기 전까지 2달 반정도 남은 지금, 목표가 두가지 있는데 하나가 할인이라고 일단 결제해버린 김영한 개발자님의 스프링 강의 6개를 완강하는 것이지만, 어려울 것 같다.. 3개 정도 듣고 끝날 것 같다.

목표 나머지 하나는 백준 플래티넘 달성이다.