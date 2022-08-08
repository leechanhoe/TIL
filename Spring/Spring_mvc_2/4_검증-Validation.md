![](https://velog.velcdn.com/images/dodo4723/post/2bd7a8c7-ba3f-4e07-8c27-5138bb943b33/image.png)

김영한 선생님의 스프링 MVC 2편 강의를 듣고 정리한 내용이다.

# 4. 검증 1 - Validation

컨트롤러의 중요한 역할중 하나는 HTTP 요청이 정상인지 검증하는 것이다. 그리고 정상 로직보다 이런 검증 로직을 잘 개발하는 것이 어쩌면 더 어려울 수 있다.

## 4.1. 검증 직접 처리

```java
 	@PostMapping("/add") //실제 저장
    public String addItem(@ModelAttribute Item item, RedirectAttributes redirectAttributes, Model model) {
        //검증 오류 결과를 보관
        Map<String, String> errors = new HashMap<>();

        //검증 로직 - itemName에 글자가 없을 경우
        if(!StringUtils.hasText(item.getItemName())){
            errors.put("itemName", "상품 이름은 필수입니다.");
        }
        //검증 로직 - itemPrice가 범위를 넘어설 경우
        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
            errors.put("price", "가격은 1,000 ~ 1,000,000 까지 허용합니다.");
        }
        //검증 로직 - itemQuantity의 수량 검즘
        if (item.getQuantity() == null || item.getQuantity() >= 9999) {
            errors.put("quantity", "수량은 최대 9,999 까지 허용합니다.");
        }
        // 특정 필드가 아닌 복합 룰 검증
        if (item.getPrice() != null && item.getQuantity() != null){
            int resultPrice = item.getPrice() * item.getQuantity();
            if(resultPrice < 10000){
                errors.put("globalError", "가격 * 수량의 값이 10000원 이상이여야 합니다. 현재 는 " + resultPrice + "입니다");
            }
        }

        // 검증을 모두 실행한 이후에는, 다시 입력폼으로 돌아가야함
        if(!errors.isEmpty()){
            model.addAttribute("errors",errors); // 다시 보내려면 모델에 담아야 함.
            return "validation/v1/addForm"; //입력폼 템플릿으로 보내버리기
        }

        // 예외사항 안타면 성공 로직
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/validation/v1/items/{itemId}";
    }
```

**보완점**
- 타입 오류 처리 미비
- 타입 오류 시 입력 내용이 사라짐
- 이 때문에 스프링이 제공하는 입력방법을 사용함.

## 4.2. Binding Result

`BindingResult bindingResult` 파라미터의 위치는 `@ModelAttribute Item item` 다음에 와야 한다.
```java
	@PostMapping("/add") //실제 저장
    public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {
        //검증 오류 결과를 보관
        Map<String, String> errors = new HashMap<>();

        //검증 로직 - itemName에 글자가 없을 경우
        if(!StringUtils.hasText(item.getItemName())){
            bindingResult.addError(new FieldError("item", "itemName", "상뭄 이름은 필수입니다."));
        }
        //검증 로직 - itemPrice가 범위를 넘어설 경우
        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
            bindingResult.addError(new FieldError("item", "price", "가격은 1,000 ~ 1,000,000 까지 허용합니다."));
        }
        //검증 로직 - itemQuantity의 수량 검즘
        if (item.getQuantity() == null || item.getQuantity() >= 9999) {
            bindingResult.addError(new FieldError("item", "quantity", "수량은 최대 9,999 까지 허용합니다."));
        }
        // 특정 필드가 아닌 복합 룰 검증
        if (item.getPrice() != null && item.getQuantity() != null){
            int resultPrice = item.getPrice() * item.getQuantity();
            if(resultPrice < 10000){
                // 글로벌에러는 ObjectError를 사용한다.
                bindingResult.addError(new ObjectError("item", "가격 * 수량의 값이 10000원 이상이여야 합니다. 현재 는 " + resultPrice + "입니다"));
            }
        }

        // 검증을 모두 실행한 이후에는, 다시 입력폼으로 돌아가야함
        if(bindingResult.hasErrors()){ // 만약 에러가 있었다면의 표현 방식이 바뀜
            // 자동으로 뷰에 넣기 때문에 Model에 담을 필요가 없음
            return "validation/v2/addForm"; //입력폼 템플릿으로 보내버리기
        }

        // 예외사항 안타면 성공 로직
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/validation/v2/items/{itemId}";
    }
```

<br>

### 타임리프 스프링 검증 오류 통합 기능
타임리프는 스프링의 `BindingResult` 를 활용해서 편리하게 검증 오류를 표현하는 기능을 제공한다.

- `#fields` : `#fields` 로 `BindingResult` 가 제공하는 검증 오류에 접근할 수 있다.
```
<div th:if="${#fields.hasGlobalErrors()}">
```
- `th:errors` : 해당 필드에 오류가 있는 경우에 태그를 출력한다. `th:if` 의 편의 버전이다.
```
<div class="field-error" th:errors="*{quantity}">
```
- `th:errorclass` : `th:field` 에서 지정한 필드에 오류가 있으면 `class` 정보를 추가한다.
```
<div th:errorclass="field-error" class="form-control">
```

### 문제점
오류가 발생했을때, 데이터가 유지되지 않는 단점이 있다.

<br>

## 4.3. FieldError, ObjectError
입력한 값을 화면에 남겨보자
`FieldError`와 `ObjectError`는 유사하게 크게 두 가지의 생성자를 가진다.

FieldError 생성자
FieldError 는 두 가지 생성자를 제공한다.
```java
public FieldError(String objectName, String field, String defaultMessage);

public FieldError(String objectName, String field, @Nullable Object 
rejectedValue, boolean bindingFailure, @Nullable String[] codes, @Nullable
Object[] arguments, @Nullable String defaultMessage)
```

- `objectName` : 오류가 발생한 객체 이름
- `field` : 오류 필드
- `rejectedValue` : 사용자가 입력한 값(거절된 값)
- `bindingFailure` : 타입 오류 같은 바인딩 실패인지, 검증 실패인지 구분 값
- `codes` : 메시지 코드
- `arguments` : 메시지에서 사용하는 인자
- `defaultMessage` : 기본 오류 메시지

```java
bindingResult.addError(new FieldError("item", "itemName",item.getItemName(),false,null,null, "상품 이름은 필수입니다."));
//세번째 파라미터(rejectedValue) 가 사용자가 입력한 값(거절된 값)이다.
bindingResult.addError(new ObjectError("item",null,null, "가격 * 수량의 값이 10000원 이상이여야 합니다. 현재 는 " + resultPrice + "입니다"));
```

### 타임리프의 사용자 입력 값 유지

#### `th:field="*{price}"`
타임리프의 `th:field` 는 매우 똑똑하게 동작하는데, 정상 상황에는 모델 객체의 값을 사용하지만, 오류가
발생하면 `FieldError` 에서 보관한 값을 사용해서 값을 출력한다.

### 스프링의 바인딩 오류 처리
타입 오류로 바인딩에 실패하면 스프링은 `FieldError` 를 생성하면서 사용자가 입력한 값을 넣어둔다. 
그리고 해당 오류를 `BindingResult` 에 담아서 컨트롤러를 호출한다. 따라서 타입 오류 같은 바인딩
실패시에도 사용자의 오류 메시지를 정상 출력할 수 있다.

<br>

## 4.4. 오류코드와 메시지 처리

위에서 봤듯이 `FieldError` , `ObjectError` 의 생성자는 `codes` , `arguments` 를 제공한다. 이것은 오류 발생시 오류코드로 메시지를 찾기 위해 사용된다.

### 1단계
- `errors.properties`를 만든다.
- 스프링 부트가 파일을 인식할 수 있게 다음 문장을 추가한다.
`spring.messages.basename=messages,errors`
- `errors`에 등록된 메시지를 사용해본다.
예) `range.item.price=가격은 {0} ~ {1} 까지 허용합니다`
```java
// 코드는 String 배열로 넘긴다.
bindingResult.addError(new FieldError("item", "price",item.getPrice(),false,new String[]{"range.item.price"},new Object[]{1000,1000000}, null));
```

<br>

### 2단계
- 위와 같은 과정이 조금 번거로워서, 보다 자동화를 거친다.

- `rejectValue()` , `reject()`를 사용하면 `FieldError`, `ObjectError`를 사용하지 않고 깔끔하게 검증 오류를 다룰 수 있다.

- `BindingResult` 는 어떤 객체를 대상으로 검증하는지 target을 이미 알고 있다. 따라서 `target(item)`에 대한 정보는 없어도 된다. 오류 필드명은 동일하게 `quantity` 를 사용했다.
```java
bindingResult.rejectValue("quantity", "max", new Object[]{9999}, null);
```

<br>

### 3단계

오류 코드를 만들 때 다음과 같이 자세히 만들 수도 있고,
- `required.item.itemName` : 상품 이름은 필수 입니다.
- `range.item.price` : 상품의 가격 범위 오류 입니다.

또는 다음과 같이 단순하게 만들 수도 있다.
- `required` : 필수 값 입니다.
- `range` : 범위 오류 입니다.

그런데 오류 메시지에 required.item.itemName 와 같이 객체명과 필드명을 조합한 세밀한 메시지 코드가 있으면 이 메시지를 높은 우선순위로 사용하는 것이다.
```
#Level1
required.item.itemName: 상품 이름은 필수 입니다.

#Level2
required: 필수 값 입니다.
```

스프링은 `MessageCodesResolver` 라는 것으로 이러한 기능을 지원한다.

<br>

### 4단계

#### FieldError `rejectValue("itemName", "required")`
다음 4가지 오류 코드를 자동으로 생성
- required.item.itemName
- required.itemName
- required.java.lang.String
- required

#### ObjectError `reject("totalPriceMin")`
다음 2가지 오류 코드를 자동으로 생성
- totalPriceMin.item
- totalPriceMin

```java
@Test
	void messageCodesResolverField() {
	String[] messageCodes = codesResolver.resolveMessageCodes("required",
	"item", "itemName", String.class);
		assertThat(messageCodes).containsExactly(
		"required.item.itemName",
		"required.itemName",
		"required.java.lang.String",
		"required"
 	);
 }
 ```
 
 <br>

### 5단계

**핵심은 구체적인 것에서! 덜 구체적인 것으로!**

`MessageCodesResolver 는 required.item.itemName` 처럼 구체적인 것을 먼저 만들어주고, `required` 처럼 덜 구체적인 것을 가장 나중에 만든다.

이렇게 하면 앞서 말한 것 처럼 메시지와 관련된 공통 전략을 편리하게 도입할 수 있다.

### ValidationUtils
#### `ValidationUtils` 사용 전
```java
if (!StringUtils.hasText(item.getItemName())) {
 bindingResult.rejectValue("itemName", "required", "기본: 상품 이름은 필수입니다.");
}
```
#### `ValidationUtils` 사용 후
다음과 같이 한줄로 가능, 제공하는 기능은 Empty , 공백 같은 단순한 기능만 제공
```java
ValidationUtils.rejectIfEmptyOrWhitespace(bindingResult, "itemName", "required");
```

<br>

### 6단계

검증 오류 코드는 다음과 같이 2가지로 나눌 수 있다.
- 개발자가 직접 설정한 오류 코드 `rejectValue()` 를 직접 호출
- 스프링이 직접 검증 오류에 추가한 경우(주로 타입 정보가 맞지 않음)

타입 오류 발생시 에러 메세지 코드엔 다음과 같이 4가지 메시지 코드가 입력되어 있다.

- `typeMismatch.item.price`
- `typeMismatch.price`
- `typeMismatch.java.lang.Integer`
- `typeMismatch`

errors.properties 에 메시지 코드가 없으면 스프링이 생성한 기본 메시지가 출력된다.
```
Failed to convert property value of type java.lang.String to required type 
java.lang.Integer for property price; 
nested exception is java.lang.NumberFormatException: For input string: "A"
```
`error.properties` 에 다음 내용을 추가하면 소스코드를 건들지 않고, 원하는 메세지를 단계별로 설정 할 수있다.
```
typeMismatch.java.lang.Integer=숫자를 입력해주세요.
typeMismatch=타입 오류입니다.
```

<br>

### 정리
**메시지 코드 생성 전략은 그냥 만들어진 것이 아니다. 조금 뒤에서 `Bean Validation`을 학습하면 그 진가를더 확인할 수 있다.**

<br>

## 4.5. validiator 분리

컨트롤러에서 검증 로직이 차지하는 부분은 매우 크다. 이런 경우 별도의 클래스로 역할을 분리하는 것이 좋다. 그리고 이렇게 분리한 검증 로직을 재사용 할 수도 있다.

위에서 작성했던 검증 로직들을 `itemValidator` 클래스에 떼어내고,
```java
@InitBinder
public void init(WebDataBinder dataBinder) {
 log.info("init binder {}", dataBinder);
 dataBinder.addValidators(itemValidator);
}
```
컨트롤러에 위 내용을 추가하고, `WebDataBinder` 에 검증기를 추가하면 해당 컨트롤러에서는 검증기를 자동으로 적용할 수 있다.

`@InitBinder` -> 해당 컨트롤러에만 영향을 준다. 글로벌 설정은 별도로 해야한다.

### 2단계
```java
	@PostMapping("/add")
    public String addItemV6(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
        // @Validated라는 것을 넣어줘야 아이템에 대해서 자동으로 검증해줌
        // 검증이 여러개 올 경우 서포트로 관리한다. 
        if (bindingResult.hasErrors()) {
            return "validation/v2/addForm";
        }
    }
```
`validator`를 직접 호출하는 부분이 사라지고, 대신에 검증 대상 앞에 `@Validated` 가 붙었다.

`@Validated` 는 검증기를 실행하라는 애노테이션이다.

이 애노테이션이 붙으면 앞서 `WebDataBinder` 에 등록한 검증기를 찾아서 실행한다. 그런데 여러 검증기를 등록한다면 그 중에 어떤 검증기가 실행되어야 할지 구분이 필요하다. 이때 `supports()` 가 사용된다. 

여기서는 `supports(Item.class)` 호출되고, 결과가 `true` 이므로 `ItemValidator` 의 `validate()` 가 호출된다

### 글로벌 설정
```java
@SpringBootApplication
public class ItemServiceApplication implements WebMvcConfigurer {
	public static void main(String[] args) {
		SpringApplication.run(ItemServiceApplication.class, args);
}

@Override
	public Validator getValidator() {
 		return new ItemValidator();
 	}
}
```
이렇게 글로벌 설정을 추가할 수 있다. 기존 컨트롤러의 `@InitBinder` 를 제거해도 글로벌 설정으로 정상 동작하는 것을 확인할 수 있다.