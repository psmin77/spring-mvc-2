## 검증 요구사항
- 타입 검증
  - 가격, 수량에 문자 허용하지 않음
- 필드 검증
  - 상품명: 필수, 공백X
  - 가격: 1,000원 이상 ~ 1,000,000원 이하
  - 수량: 최대 9999개
- 특정 필드의 범위 검증
  - 가격 * 수량의 합은 10,000원 이상
<br>

## 프로젝트 V1
~~~ java
@PostMapping("/add")
public String addItem(@ModelAttribute Item item, RedirectAttributes redirectAttributes, Model model) {

  // 검증 오류 결과를 보관
  Map<String, String> errors = new HashMap<>();
        
  // 검증 로직
  if (!StringUtils.hasText(item.getItemName())){
    errors.put("itemName", "상품 이름은 필수입니다.");
  }
  if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
    errors.put("price", "가격은 1,000 ~ 1,000,000 까지 허용합니다.");
  }
  if (item.getQuantity() == null || item.getQuantity() > 9999) {
    errors.put("quantity", "수량은 최대 9,999 까지 허용합니다.");
  }

  // 특정 필드가 아닌 복합 룰 검증
  if (item.getPrice() != null && item.getQuantity() != null) {
    int resultPrice = item.getPrice() * item.getQuantity();
    if (resultPrice < 10000) {
        errors.put("globalError", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice);
    }
  }

  // 검증 실패 시 입력 폼으로 돌아가기
  if (!errors.isEmpty()) {
    log.info("errors = {} ", errors);
    model.addAttribute("errors", errors);
    return "validation/v1/addForm";
  }
~~~
- 검증 오류 발생 시, Map(errors)에 해당 정보 담아두기
  - 오류가 발생한 필드명을 key로 사용
~~~ html
<!-- 글로벌 오류 -->
<div th:if="${errors?.containsKey('globalError')}">
  <p class="field-error" th:text="${errors['globalError']}">전체 오류 메시지</p>
</div>
~~~
- errors.globalError가 있으면 해당 오류 박스/메시지 출력
- .? : Safe Navigation Operator
  - 해당 값이 null일 때, NullPointerException이 아닌 null을 반환
  
~~~ html
<!-- 필드 오류 -->
<div>
  <label for="itemName" th:text="#{label.item.itemName}">상품명</label>

  <!-- 
    <input type="text" th:classappend="${errors?.containsKey('itemName')} ? 'field-error' : _" class="form-control">
  -->
  <input type="text" id="itemName" th:field="*{itemName}"
         th:class="${errors?.containsKey('itemName')} ? 'form-control field-error' : 'form-control'" 
         class="form-control" placeholder="이름을 입력하세요">
  <div class="field-error" th:if="${errors?.containsKey('itemName')}"
       th:text="${errors['itemName']}">상품명 오류</div>
</div>
~~~
- errors.field가 있으면 해당 오류 메시지 출력, CSS 클래스 적용
- th:classappend 도 가능 ( _ : No-Operation)
<br>

## 프로젝트 V2
### BindingResult (addItemV1)
~~~ java
@PostMapping("/add")
public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {

  // 필드 예외
  if (!StringUtils.hasText(item.getItemName())) { 
    bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수입니다."));
  }
  ...
  
  //특정 필드 예외가 아닌 전체 예외
  if (item.getPrice() != null && item.getQuantity() != null) {
    int resultPrice = item.getPrice() * item.getQuantity();
    if (resultPrice < 10000) {
      bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000원 이상이어야 합니다.));
    }
  }
  
  // 검증 실패
  if (bindingResult.hasErrors()) {
    log.info("errors={}", bindingResult);
    return "validation/v2/addForm";
  }
~~~
- BindingResult : 스프링이 제공하는 검증 오류 기능
- _! BindingResult 파라미터는 반드시 검증할 대상(@ModelAttribute) 다음에 위치해야 함_
- FieldError(String objectname, String field, String defaultMessage)
  - objectName : @ModelAttribute 이름
  - field : 오류 발생한 필드 이름
  - defaultMessage : 오류 기본 메시지
- ObjectError(String objectName, String defaultMessage)

~~~ html
<div th:if="${#fields.hasGlobalErrors()}">
  <p class="field-error" th:each="err : ${#fields.globalErrors()}"
     th:text="${err}">글로벌 오류 메시지</p>
</div>

...

<div>
  <label for="itemName" th:text="#{label.item.itemName}">상품명</label>
  <input type="text" id="itemName" th:field="*{itemName}" 
         th:errorclass="field-error" class="form-control">
  <div class="field-error" th:errors="*{itemName}">상품명 오류</div>
</div>
~~~
- #fields : BindingResult가 제공하는 검증 오류에 접근 가능
  - globalsErrors : 여러 개의 값을 가질 수 있기 때문에 th:each로 사용
- th:errors : 해당 필드에 오류가 있는 경우 태그 출력 (th:if의 편의 버전)
- th:errorclass : 지정한 필드에 오류가 있으면 class 정보 추가
<br>

#### @ModelAttribute 바인딩 타입 오류 발생 시
- BindingResult가 없으면, 400 오류 페이지로 이동
- BindingResult가 있으면, 오류 정보(FieldError)를 BindingResult에 담아서 컨트롤러 정상 호출

#### BindingResult와 Errors
- BingdingResult는 Errors 인터페이스를 상속받는 인터페이스
- Errors는 단순 오류 저장과 조회 기능을 제공
- BindingResult는 추가적인 기능을 제공하기 때문에, 주로 관례상 BindingResult를 사용함
<br>

### FieldError, ObjectError (addItemV2)
~~~ java
// ObjectError도 유사한 생성자 제공
public FieldError(String objectName, String field, String defaultMessage);
public FieldError(String objectName, String field, @Nullable Object rejectedValue,
                  boolean bindingFailure, @Nullable String[] codes, 
                  @Nullable Object[] arguments, @Nullable String defaultMessage)
~~~
- objectNmae : 오류가 발생한 객체 이름
- field : 오류 필드
- rejectedValue : 사용자가 입력한 값 (거절된 값)
  - 오류 발생 시 사용자가 입력한 값을 유지할 수 있음
- bindingFailure : 타입 오류 구분 (바인딩 실패 or 검증 실패인지)
- codes : 메시지 코드
- arguments : 메시지에서 사용하는 인자
- defaultMessage : 기본 오류 메시지

### 타임리프 사용자 입력 값 유지
- th:field="*{...}"
  - 일반적인 상황에서는 모델 객체의 값을 사용함
  - 오류가 발생하는 경우, FieldError에서 보관한 값을 사용함

### 스프링 바인딩 오류 처리
- 타입 오류로 바인딩에 실패하는 경우, 스프링은 FieldError를 생성하여 사용자가 입력한 값을 넣은 뒤 BindingResult에 담아 컨트롤러를 호출함
<br>

## 오류 코드와 메시지 처리 1
### Errors.properties (addItemV3)
- 스프링 부트 메시지 설정 추가
- _application.properties_
~~~
spring.messages.basename = messages, errors
~~~
- _errors.properties_
~~~
required.item.itemName=상품 이름은 필수입니다.
range.item.price=가격은 {0} ~ {1} 까지 허용합니다.
max.item.quantity=수량은 최대 {0} 까지 허용합니다.
totalPriceMin=가격 * 수량의 합은 {0}원 이상이어야 합니다. 현재 값 = {1}
~~~
- _controller(addItemV3)_
~~~java
bindingResult.addError(new FieldError("item", "price", 
                       item.getPrice(), false, new String[]{"range.item.price"},
                       new Object[]{1000, 1000000}, null));
// "가격은 1000 ~ 1000000 까지 허용합니다."
~~~
- codes : properties에 설정된 메시지 코드 지정. 배열로 여러 값을 전달하여 순서대로 매칭함
- arguments : Object[]{...} 배열로 지정하여 코드의 {0},{1}로 치환할 값을 전달함
<br>

## 오류 코드와 메시지 처리 2
### rejectValue, reject (addItemV4)
~~~java
void rejectValue (@Nullable String field, String errorCode, 
                  @Nullable Object[] errorArgs, @Nullable String defaultMessage);

void reject(String errorCode, @Nullable Object[] errorArgs, 
            @Nullable String defaultMessage);
~~~
- rejectValue (reject 동일)
  - field : 오류 필드명
  - errorCode : 오류 코드 (messageResolver)
  - errorArgs : 오류 메시지에서 {0}에 치환하기 위한 값
  - defaultMessage : 오류 메시지가 없을 때 사용하는 기본 메시지
- _Controller(addItemV4)_
~~~java
// 특정 field 예외 (itemName, price, quantity)
bindingResult.rejectValue("itemName", "required");
bindingResult.rejectValue("price","range", new Object[]{1000,1000000}, null);
bindingResult.rejectValue("quantity","max", new Object[]{9999}, null);

// 전체 예외
bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
~~~
<br>

## 오류 코드와 메시지 처리 3
- 오류 코드의 범용성 또는 구체성 중 고려해야 함
- 메시지 단계를 나누어 사용 가능
  - 객체명과 필드명을 조합한 구체적인 메시지 코드가 있으면 높은 우선 순위로 사용
<br>

## 오류 코드와 메시지 처리 4
### MessageCodesResolver
- 검증 오류 코드로 메시지 코드를 생성함
- MessageCodesResolver 인터페이스, DefaultMessageCodesResolver 기본 구현체

### DefaultMessageCodesResolver
- 객체 오류인 경우
  - code + "." + object name 
  - code
  - _ex) required, object name-item
  -> required.item, required_ 
- 필드 오류인 경우
  - code + "." + object name + "." + field
  - code + "." + field
  - code + "." + field type
  - code
- 동작 방식
  - rejectValue(), reject()는 내부에서 MessageCodesResolver를 사용하여 메시지 코드를 생성
  - Resolver를 통해 생성된 순서대로 오류 코드들을 보관
- 오류 메시지 출력
  - 타임리프 화면 렌더링에서 th:error가 실행됨
  - 오류가 있다면 생성된 오류 메시지 코드를 순서대로 찾아 출력
  - 없다면 디폴트 메시지 출력
<br>

## 오류 코드와 메시지 처리 5
- 메시지 코드는 구체적 > 범용적으로 사용해야 함
- _errors.properties_
  - _Level1 > 4 순서로_
~~~
#==FieldError==
#Level1
required.item.itemName=상품 이름은 필수입니다.
range.item.price=가격은 {0} ~ {1} 까지 허용합니다.
max.item.quantity=수량은 최대 {0} 까지 허용합니다.
      
#Level2 - 생략

#Level3
required.java.lang.String = 필수 문자입니다.
required.java.lang.Integer = 필수 숫자입니다.
min.java.lang.String = {0} 이상의 문자를 입력해주세요.
min.java.lang.Integer = {0} 이상의 숫자를 입력해주세요.
range.java.lang.String = {0} ~ {1} 까지의 문자를 입력해주세요. 
range.java.lang.Integer = {0} ~ {1} 까지의 숫자를 입력해주세요. 
max.java.lang.String = {0} 까지의 문자를 허용합니다. 
max.java.lang.Integer = {0} 까지의 숫자를 허용합니다.

#Level4
required = 필수 값 입니다.
min= {0} 이상이어야 합니다.
range= {0} ~ {1} 범위를 허용합니다. 
max= {0} 까지 허용합니다.
~~~
- 오류 코드와 메시지 처리 순서
  - rejectValue() 호출
  - MessageCodesResolver로 오류 코드를 통해 메시지 코드들을 생성
  - new FieldError()를 생성하고 메시지 코드들을 보관
  - th:errors에서 해당 메시지 코드들을 순서대로 찾아 출력

### ValidationUtils
~~~java
ValidationUtils.rejectIfEmptyOrWhitespace
               (bindingResult, "itemName", "required");
~~~
- 공백(empty) 같은 단순한 기능만 제공
<br>

## 오류 코드와 메시지 처리 6
- 검증 오류 코드
  - 직접 설정한 오류 코드
  - 스프링 기본 오류 메시지 (주로 타입 오류)
    - 메시지 코드를 별도 설정하면 해당 메시지 출력
    - _error.properties_
    ~~~
    typeMismatch.java.lang.Integer=숫자를 입력해주세요.
    typeMismatch=타입 오류입니다.
    ~~~
    
<br>

## Validator 1
### Validator
~~~java
public interface Validator {
  boolean supports(Class<?> clazz);
  void validate(Object target, Errors errors);
}
~~~
- supports() {} : 해당 검증기를 지원하는 여부 확인
- validate(Object target, Errors errors) : 검증 대상 객체와 BindingResult

### ItemValidator (addItemV5)
- ItemValidator를 생성하여 검증 로직을 별도 분리
- _Controller(addItemV5)_
~~~java
private final ItemValidator itemValidator;
...
  itemValidator.validate(item, bindingResult);
~~~
- ItemValidator를 스프링 빈으로 주입 받아 직접 호출
<br>

## Validator 2
###

<br>

> [출처] 스프링 MVC 2 - 김영한, 인프런
