#### 첫 Welcome 페이지 만들기.
```java
@Controller
public class HelloController {
 @GetMapping("hello")
 public String hello(Model model) {
 model.addAttribute("data", "hello!!");
 return "hello";
 }
}
```
`resources/templates/hello.html`
```java
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
 <title>Hello</title>
 <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
<p th:text="'안녕하세요. ' + ${data}" >안녕하세요. 손님</p>
</body>
</html>
```
- 컨트롤러에서 리턴 값으로 문자를 반환하면 뷰 리졸버가 화면을 찾아서 처리한다.
    - 스프링 부트 템플릿엔진 기본 viewName 매핑
    - resources:templates/+{ViewName}+.html
    
#### 빌드하고 실행하기

1. gradlew build
2. dir build/libs
3. java -jar hello-spring-0.0.1-SNAPSHOT.jar
4. 실행 확인