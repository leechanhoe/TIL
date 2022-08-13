![](https://velog.velcdn.com/images/dodo4723/post/3acf2718-2fc4-42c4-a4d8-ff2710f1b49c/image.png)

김영한 개발자님의 스프링 MVC 2 강의를 수강하고 정리한 내용이다.

# 9. API 예외 처리

## 9.1. 시작

HTML과는 달리 API는 각 오류 상황에 맞는 스펙을 정하고 JSON으로 데이터를 내려주어야 한다.

먼저 API 예외 컨트롤러를 만들어보자
```java
@Slf4j
@RestController
public class ApiExceptionController {
    @GetMapping("/api/members/{id}")
    public MemberDto getMember(@PathVariable("id") String id) {
        if (id.equals("ex")) {
            throw new RuntimeException("잘못된 사용자");
        }
        if (id.equals("bad")) {
            throw new IllegalArgumentException("잘못된 입력 값");
        }
        return new MemberDto(id, "hello " + id);
    }
    @Data
    @AllArgsConstructor
    static class MemberDto{
        private String memberId;
        private String name;

    }
}
```
정상의 경우 API JSON 형식으로 데이터가 정상 반환된다. 그런데 오류가 발생하면 기존 오류 페이지 HTML이 반환된다.

오류페이지 컨트롤러도 JSON응답을 하게 아래와 같이 코드를 추가한다.

```java
	@RequestMapping(value = "/error-page/500", produces = MediaType.APPLICATION_JSON_VALUE)
//    MediaType은 스프링 프레임워크
//    produces = MediaType.APPLICATION_JSON_VALUE 의 뜻은 클라이언트가 요청하는 HTTP Header의
//    Accept 의 값이 application/json 일 때 해당 메서드가 호출된다는 것이다. 결국 클라어인트가 받고
//    싶은 미디어타입이 json이면 이 컨트롤러의 메서드가 호출된다.
    public ResponseEntity<Map<String, Object>> errorPage500Api(HttpServletRequest request, HttpServletResponse response) {
        log.info("API errorPage 500");
        Map<String, Object> result = new HashMap<>();
        Exception ex = (Exception) request.getAttribute(ERROR_EXCEPTION);
        result.put("status", request.getAttribute(ERROR_STATUS_CODE));
        result.put("message", ex.getMessage());
        Integer statusCode = (Integer)
                request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);
        return new ResponseEntity(result, HttpStatus.valueOf(statusCode));
    }
```

스프링 부트가 제공하는 BasicErrorController 는 HTML 페이지를 제공하는 경우에는 매우 편리하다. 4xx, 5xx 등등 모두 잘 처리해준다. 

그런데 API 오류 처리는 다른 차원의 이야기이다. API 마다, 각각의 컨트롤러나 예외마다 서로 다른 응답 결과를 출력해야 할 수도 있다.

<br>

## 9.2. API 예외 처리 - Handler Exception Resolver

예외가 발생해서 서블릿을 넘어 WAS까지 예외가 전달되면 HTTP 상태코드가 500으로 처리된다. 상태코드를 바꿀 수 있을까?

컨트롤러 밖으로 던져진 예외를 해결하고, 동작 방식을 변경하고 싶으면 `HandlerExceptionResolver` 를 사용하면 된다.

![](https://velog.velcdn.com/images/dodo4723/post/7ce9a742-a241-4999-ab06-364ecdb317aa/image.png)

WAS까지 가던 에러를 `ExceptionResolver`가 처리해 정상적으로 `ModelandView` 반환을 하면, 흐름이 정상적으로 바뀐다.

아래는 **상태코드를 500에서 400으로 변경**하는 코드이다.
```java
@Slf4j
public class MyHandlerExceptionResolver implements HandlerExceptionResolver {

    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        try {
            if (ex instanceof IllegalArgumentException) {
                // 만약 예외가 IllegalArgumentException일 경우
                log.info("IllegalArgumentException resolver to 400");
                response.sendError(HttpServletResponse.SC_BAD_REQUEST, ex.getMessage());
                // BAD_REQUEST가 400이다, 400으로 변경 Excpetion을 sendError로 바꿔치기 하는 것
                return new ModelAndView();
            }
        }catch(IOException e){
            log.error("resolver ex", e);
        }
        return null;
    }
}
```
예외가 발생해도 서블 까지 전송되지 않고, MVC에서 예외처리가 끝이 난다.

다만 직접 구현하기가 힘들어, 스프링이 제공하는 `ExceptionResolver`를 사용한다.

<br>

## 9.3. 스프링에서 제공하는 Exception Resolver

스프링 부트가 기본으로 제공하는 `ExceptionResolver` 는 다음과 같다.

> #### 1. `ExceptionHandlerExceptionResolver`
`@ExceptionHandler` 을 처리한다. API 예외 처리는 대부분 이 기능으로 해결한다. 제일 중요
#### 2. `ResponseStatusExceptionResolver`
HTTP 상태 코드를 지정해준다.
#### 3. `DefaultHandlerExceptionResolver`
스프링 내부 기본 예외를 처리한다.
우선 순위가 가장 낮다

<br>

### 9.3.1. ExceptionHandlerExceptionResolver - HTTP 응답 코드 변경
- 오류가 발생했을 때 응답의 모양이 다를 수 있다.
- 이렇게 API예외처리 문제를 해결하기 위해 @ExceptionHandler라는 애노테이션을 사용해 편리한 예외 처리 기능을 제공하는데 이게 `ExceptionHandlerExceptionResolver`이다.

#### 9.3.1.1 ResponseStatusExceptionResolver
```java
@ResponseStatus(code = HttpStatus.BAD_REQUEST, reason = "잘못된 요청 오류")
public class BadRequestException extends RuntimeException {
}
// reason을 메세지 소스에서 찾는 기능도 제공한다. 
@ResponseStatus(code = HttpStatus.BAD_REQUEST, reason =  "error.bad")
```
- 개발자가 직접 변경할 수 없는 예외에는 적용할 수 없다.
- 애노테이션을 사용하기 때문에 조건에 따라 동적으로 변경하는 것도 어렵다.

#### 9.3.1.2. ResponseStatusException
- 이를 극복하기 위해 `ResponseStatusException` 사용
- `@ExceptionHandler` 애노테이션을 선언하고, 해당 컨트롤러에서 처리하고 싶은 예외를 지정해주면 된다.

```java
@GetMapping("/api/response-status-ex2")
    public String responseStatusEx2() {
        throw new ResponseStatusException(HttpStatus.NOT_FOUND, "error.bad", new
                IllegalArgumentException());
    }
: response.sendError(statusCode, resolvedReason)를 호출한다.
```

<br>

### 9.3.2. DefaultHandlerExceptionResolver - 스프링 내부 예외 처리
스프링 내부에서 발생하는 스프링 에외를 해결해준다. 대표적으로 파라미터 바인딩 시점에 타입이 맞지 않으면, 내부에서 TypeMismatchException이 발생하여서 500오류가 발생한다. 근데 파라미터 바인딩은 대부분 클라이언트가 HTTP요청을 잘못 호출해서 생긴 것이라 HTTP서는 이 오류에 HTTP상태 코드 400을 사용하게 한다.

<br>

## 9.4. API 예외 처리 - `@ExceptionHandler`

스프링은 API 예외 처리 문제를 해결하기 위해 `@ExceptionHandler` 라는 애노테이션을 사용하는 매우 편리한 예외 처리 기능을 제공하는데, 이것이 바로 `ExceptionHandlerExceptionResolver` 이다. 

실무에서 API 예외 처리는 대부분 이 기능을 사용한다.

`@ExceptionHandler` 애노테이션을 선언하고, 해당 컨트롤러에서 처리하고 싶은 예외를 지정해주면 된다. 
해당 컨트롤러에서 예외가 발생하면 이 메서드가 호출된다.

```java
// IllegalArgumentException 또는 그 하위 자식 클래스를 모두 처리할 수 있다
@ExceptionHandler(IllegalArgumentException.class)
public ErrorResult illegalExHandle(IllegalArgumentException e) {
 log.error("[exceptionHandle] ex", e);
 return new ErrorResult("BAD", e.getMessage());
}

//HTTP 응답 코드를 프로그래밍해서 동적으로 500으로 변경
@ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
//생략하면 메서드 파라미터의 예외가 지정된다
@ExceptionHandler
public ResponseEntity<ErrorResult> userExHandle(UserException e) {}
```

<br>

## 9.5. @ControllerAdvice
정상 코드와 예외 처리 코드가 하나의 컨트롤러에 섞여 있지만,  `@ControllerAdvice`를 사용하면 이를 분리할 수 있다.

- `@ControllerAdvice` 는 대상으로 지정한 여러 컨트롤러에 `@ExceptionHandler`, `@InitBinder` 기능을 부여해주는 역할을 한다.
- `@ControllerAdvice` 에 대상을 지정하지 않으면 모든 컨트롤러에 적용된다.
- `@RestControllerAdvice` 는 `@ControllerAdvice` 와 같고, `@ResponseBody` 가 추가되어 있다.
```java
// Target all Controllers annotated with @RestController
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {}

// Target all Controllers within specific packages
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {}

// Target all Controllers assignable to specific classes
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class ExampleAdvice3 {}
```