### 검증 요구사항
- 타입 검증
  - 가격, 수량에 문자 허용하지 않음
- 필드 검증
  - 상품명: 필수, 공백X
  - 가격: 1,000원 이상 ~ 1,000,000원 이하
  - 수량: 최대 9999개
- 특정 필드의 범위 검증
  - 가격 * 수량의 합은 10,000원 이상
  
### 프로젝트 V1
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

### 프로젝트 V2
#### BindingResult
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
- BindingResult : 스프링 오류 검증 기능
- bindingResult 파라미터는 @ModelAttribute 다음에 위치해야 함
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


> [출처] 스프링 MVC 2 - 김영한, 인프런
