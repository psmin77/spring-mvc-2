### 타임리프 스프링 통합
#### 스프링 통합 기능
- 스프링 SpringEL 문법 통합
- 스프링 빈 호출 지원
  - ${@myBean.doSomething()} 등
- 편리한 폼 관리 속성 지원
  - th:object, th:field, th:errors, th:errorclass 등
- 폼 컴포넌트 기능
- 스프링 메시지, 국제화 기능 통합
- 스프링 검증, 오류 처리 통합
- 스프링 변환 서비스 통합(ConversionService)
<br>

#### 입력 폼 처리
~~~ html
<form action="item.html" th:action th:object="${item}" method="post">
  <div>
    <label for="itemName">상품명</label>
    <input type="text" id="itemName" th:field="*{itemName}" class="form-control" placeholder="이름을 입력하세요">
  </div>
  <div>
    <label for="price">가격</label>
    <input type="text" id="price" th:field="*{price}" class="form-control" placeholder="가격을 입력하세요">
  </div>
</form>
~~~
- th:object="${item}" : 커맨드 객체(item)를 지정
- *{itemName, price ... } : 선택 변수식 사용
  - ${item.itemName}과 동일한 기능
- th:field : HTML태그의 id, name, value 속성을 자동으로 처리
  - 기존 HTML id, name, value 삭제 가능
<br>

#### 요구사항 추가
- 판매 여부 : 싱글 체크박스
- 등록 지역 : 멀티 체크박스 (서울, 부산, 제주)
- 상품 종류 : 라디오 버튼 (도서, 식품, 기타)
- 배송 방식 : 셀렉트 박스 (빠른 배송, 일반 배송, 느린 배송)
<br>

#### 체크박스 - 싱글
~~~ html
<!-- single checkbox -->
<div>판매 여부</div>
<div>
  <div class="form-check">
    <input type="checkbox" id="open" name="open" class="form-check-input">
    <label for="open" class="form-check-label">판매 오픈</label>
  </div>
</div>
~~~
- 판매 여부 체크박스 추가
- 체크박스를 선택하지 않는 경우 : 이름 자체도 전송되지 않음(NULL)
  - 1) <input type="hidden" name="_name" value="on"/\>
    - "_name"을 히든 필드로 추가하여 체크/해제 인식 가능
  - 2) th:field="\*{item.open}"
    - 히든 필드 없이 field만으로 체크/해제 인식 가능

<br>

#### 체크박스 - 멀티
~~~ java
@ModelAttribute("regions")
public Map<String, String> regions() {
  Map<String, String> regions = new LinkedHashMap<>(); 
  regions.put("SEOUL", "서울");
  regions.put("BUSAN", "부산");
  regions.put("JEJU", "제주");
  return regions;
}
~~~
- @ModelAttribute : 컨트롤러를 요청할 때 자동으로 모델(Model) 생성
<br>

~~~ html
<!-- multi checkbox -->
<div>
  <div>등록 지역</div>
  <div th:each="region : ${regions}" class="form-check form-check-inline">
    <input type="checkbox" th:field="*{regions}" th:value="${region.key}" class="form-check-input">
    <label th:for="${#ids.prev('regions')}" th:text="${region.value}" class="form-check-label">서울</label>
  </div>
</div>
~~~
- regions 객체 값만큼 반복하여 체크박스 생성
  - _cf. th:field="*{regions}"는 전체 폼의 객체에 대한 필드_
- th:for="${#ids.prev('regions')}"
  - field가 임의의 id를 생성 및 부여
  - id.prev가 해당 아이디를 참조하여 반환
<br>

#### 라디오 버튼
~~~ java
@ModelAttribute("itemTypes")
public ItemType[] itemTypes() {
  return ItemType.values();
}
~~~
- array[].values() : ENUM의 모든 정보를 배열로 반환
<br>

~~~ html
<!-- radio button -->
<div>
<div>상품 종류</div>
  <div th:each="type : ${itemTypes}" class="form-check form-check-inline">
    <input type="radio" th:field="*{itemType}" th:value="${type.name()}" class="form-check-input">
    <label th:for="${#ids.prev('itemType')}" th:text="${type.description}" class="form-check-label">
      BOOK
    </label>
  </div>
</div>
~~~
- itemTypes 객체 값만큼 반복하여 라디오 버튼 생성
  - _cf. th:field="*{itemType}"은 전체 폼의 객체에 대한 필드_
- th:for="${#ids.prev('itemType')}"
  - field가 임의의 id를 생성 및 부여
  - id.prev가 해당 아이디를 참조하여 반환

##### cf. ENUM 접근
~~~ html
<div th:each="type : ${T(hello.itemservice.domain.item.ItemType).values()}">
~~~
- 스프링 EL 문법으로 ENUM을 직접 사용할 수 있지만, 권장X
<br>

#### 셀렉트 박스
~~~ java
@ModelAttribute("deliveryCodes")
public List<DeliveryCode> deliveryCodes() {
  List<DeliveryCode> deliveryCodes = new ArrayList<>();
  deliveryCodes.add(new DeliveryCode("FAST", "빠른 배송")); 
  deliveryCodes.add(new DeliveryCode("NORMAL", "일반 배송")); 
  deliveryCodes.add(new DeliveryCode("SLOW", "느린 배송")); 
  return deliveryCodes;
}
~~~
- @ModelAttribute를 통해 DeliverCode 객체 생성
<br>
~~~ html
<!-- SELECT -->
<div>
  <div>배송 방식</div>
  <select th:field="*{deliveryCode}" class="form-select">
    <option value="">==배송 방식 선택==</option>
    <option th:each="deliveryCode : ${deliveryCodes}" th:value="${deliveryCode.code}" th:text="${deliveryCode.displayName}">FAST</option>
  </select>
</div>
~~~
- th:each 사용하여 option 반복 생성
- th:value에는 코드 값, th:text에는 화면에 표시할 이름으로 설정
<br>

>
[출처] 스프링 MVC 2 - 김영한, 인프런