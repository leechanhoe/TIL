![](https://velog.velcdn.com/images/dodo4723/post/d665d826-9b79-4cbb-865c-f535d6ee5e5f/image.png)

김영한 선생님의 스프링 MVC 2편 강의를 듣고 정리한 내용이다.

# 5. 검증 2 - Bean Validation

## 5.1. Bean Validation 소개 및 시작
- 애노테이션 하나로 검증 로직을 쉽게 구현할 수 있다.

`implementation 'org.springframework.boot:spring-boot-starter-validation'` 를 gradle에 추가해주어야 한다.

아래와 같이 제한 조건들을 간단하게 나타낼 수 있다.
```java
@Data
public class Item {

 private Long id;
 
 @NotBlank
 private String itemName;
 
 @NotNull
 @Range(min = 1000, max = 1000000)
 private Integer price;
 
 @NotNull
 @Max(9999)
 private Integer quantity;
}
```

### 검증 애노테이션
`@NotBlank` : 빈값 + 공백만 있는 경우를 허용하지 않는다.
`@NotNull` : `null` 을 허용하지 않는다.
`@Range(min = 1000, max = 1000000)` : 범위 안의 값이어야 한다.
`@Max(9999)` : 최대 9999까지만 허용한다.

<br>

## 5.2. Bean Validation 스프링 적용

`LocalValidatorFactoryBean` 을 글로벌 `Validator`로 등록한다. 이 `Validator`는 `@NotNull` 같은 애노테이션을 보고 검증을 수행한다. 이렇게 글로벌 `Validator`가 적용되어 있기 때문에, `@Valid`, `@Validated` 만 적용하면 된다.
검증 오류가 발생하면, `FieldError` , `ObjectError` 를 생성해서 `BindingResult` 에 담아준다.

**타입 변환에 성공해서 바인딩해 성공해야만 적용된다.**

<br>

## 5.3. Bean Validation 에러

NotBlank 라는 오류 코드를 기반으로 MessageCodesResolver 를 통해 다양한 메시지 코드가 순서대로 생성된다.
```
@NotBlank
NotBlank.item.itemName
NotBlank.itemName
NotBlank.java.lang.String
NotBlank
```

### `BeanValidation` 메시지 찾는 순서

1. 생성된 메시지 코드 순서대로 `messageSource` 에서 메시지 찾기
2. 애노테이션의 `message` 속성 사용 `@NotBlank(message = "공백! {0}")`
3. 라이브러리가 제공하는 기본 값 사용 `공백일 수 없습니다.`

#### 애노테이션의 `message` 사용 예
```java
@NotBlank(message = "공백은 입력할 수 없습니다.")
private String itemName;
```

<br>

## 5.4. Bean Validation 수정에 적용
상품수정에 적용하는 과정이다. 앞에서 수정한 내용과 맞게 적절하게 수정하면 된다.

<br>

## 5.5. Bean Validation 한계

등록할 때와 수정할 때의 요구사항이 다른 경우 문제가 발생한다. 등록에서는 ID를 입력받는 칸이 없지만, 수정할 때에는 ID가 필수인 경우가 그 예시이다.

<br>

## 5.6. Bean Validation groups
위의 한계를 해결하기 위한 방법이다.
각각 등록과 수정을 위해서 따로 인터페이스를 만든다.
이후에 Validated에 groups를 적용한다.
```java
@Data
public class Item {

 @NotNull(groups = UpdateCheck.class) //수정시에만 적용
 private Long id;
 
 @NotBlank(groups = {SaveCheck.class, UpdateCheck.class})
 private String itemName;
 
 @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
 @Range(min = 1000, max = 1000000, groups = {SaveCheck.class,
UpdateCheck.class})
 private Integer price;
 
 @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
 @Max(value = 9999, groups = SaveCheck.class) //등록시에만 적용
 private Integer quantity;
}
```
```java
@PostMapping("/add")
public String addItemV2(@Validated(SaveCheck.class) @ModelAttribute Item item,
BindingResult bindingResult, RedirectAttributes redirectAttributes) {
 //...
}
```

groups 기능을 사용해서 등록과 수정시에 각각 다르게 검증을 할 수 있었다. 그런데 groups 기능을 사용하니 `Item` 은 물론이고, 전반적으로 복잡도가 올라갔다.

사실 groups 기능은 실제 잘 사용되지는 않는데, 그 이유는 실무에서는 주로 다음에 등장하는 등록용 폼 객체와 수정용 폼 객체를 분리해서 사용하기 때문이다.

<br>

## 5.7. Form 전송 객체 분리

실무에서는 groups 를 잘 사용하지 않는데, 그 이유가 다른 곳에 있다. 바로 등록시 폼에서 전달하는 데이터가 `Item` 도메인 객체와 딱 맞지 않기 때문이다.

소위 "Hello World" 예제에서는 폼에서 전달하는 데이터와 `Item` 도메인 객체가 딱 맞는다. 하지만 실무에서는 회원 등록시 회원과 관련된 데이터만 전달받는 것이 아니라, 약관 정보도 추가로 받는 등 `Item` 과 관계없는 수 많은 부가 데이터가 넘어온다.

그래서 보통 `Item` 을 직접 전달받는 것이 아니라, 복잡한 폼의 데이터를 컨트롤러까지 전달할 별도의 객체를 만들어서 전달한다.

예를 들면 `ItemSaveForm` 이라는 폼을 전달받는 전용 객체를 만들어서 `@ModelAttribute` 로 사용한다. 이것을 통해 컨트롤러에서 폼 데이터를 전달 받고, 이후 컨트롤러에서 필요한 데이터를 사용해서 `Item` 을 생성한다.

```java
//기존의 Item 클래스에서 검증 대신
@Data
public class ItemUpdateForm {

 @NotNull
 private Long id;
 
 @NotBlank
 private String itemName;
 
 @NotNull
 @Range(min = 1000, max = 1000000)
 private Integer price;
 
 //수정에서는 수량은 자유롭게 변경할 수 있다.
 private Integer quantity;
}
```
```java
// 아래와 같이 폼을 두개 만들어서 따로 적용한다. ModelAttriute에 이름도 제대로 적용하여야 한다. (폼 객체를 item 객체로 변환하는 과정이다.)
public String addItem(@Validated @ModelAttribute("item") ItemSaveForm form, BindingResult bindingResult, RedirectAttributes redirectAttributes) {

public String edit(@PathVariable Long itemId, @Validated @ModelAttribute("item") ItemUpdateForm form, BindingResult bindingResult) {
 //...
}
```
- `Item` 대신에 `ItemSaveform`을 전달받고 `Validated`로 검증 수행 후 `BindgingResult`로 결과도 받는다.
- `item`으로 넣지 않을 경우 `MVC model`에 `itemSaveForm`으로 담기게 된다.

<br>

## 5.8. BeanValidation - HTTP 메시지 컨버터
`@Validated`는 HTTP 메시지 컨버터에도 적용이 가능하다.

>
참고
- `@ModelAttribute` 는 HTTP 요청 파라미터(URL 쿼리 스트링, POST Form)를 다룰 때 사용한다.
- `@RequestBody` 는 HTTP Body의 데이터를 객체로 변환할 때 사용한다. 주로 API JSON 요청을 다룰 때 사용한다.

**API**의 경우 3가지 경우를 나누어 생각해야 한다.

### 1. 성공 요청
성공
### 2. 실패 요청
JSON을 객체로 생성하는 것 자체가 실패함

객체를 만들지 못하기 때문에 컨트롤러 자체가 호출되지 않고 그 전에 예외가 발생한다. 물론 Validator도 실행되지 않는다. 

### 3. 검증 오류 요청
JSON을 객체로 생성하는 것은 성공했고, 검증에서 실패함
검증 오류가 정상적으로 동작한다.