### 메시지
- label 등 다양한 메시지를 한 곳에서 관리하는 기능
- _messages.properties_
~~~ file
item=상품 
item.id=상품 ID 
item.itemName=상품명 
item.price=가격 
item.quantity=수량
~~~
- 각 HTML에서 해당 데이터를 key 값으로 사용
  - th:text=#{item.id} : '상품 ID' 사용
<br>

### 국제화
- 언어/국가별로 별도 관리하여 메시지 국제화 가능
- _messages_en.properties_
~~~
item=Item
item.id=Item ID
item.itemName=Item Name
item.price=price
item.quantity=quantity
~~~
<br>

### 스프링 메시지 소스 설정
#### 스프링
- MessageSource : 스프링 빈 직접 등록
#### 스프링 부트
- MessageSource : 스프링 빈 자동 등록
  - messages_properties(기본 값), messages_en.properties 등의 파일만 등록하면 자동 인식
- _application.properties_
~~~
spring.messages.basename=message
~~~
<br>

### 스프링 메시지 소스 테스트
- getMessage(code, args, "기본 메시지", locale)
  - code : 설정한 코드명 (item.itemName, item.price)
  - args : {0} 안에 설정할 매개변수 값 (안녕 {0} -> new Object[]{"Spring "} -> 안녕 Spring)
  - 해당 코드가 없는 경우 기본 메시지 출력
  - locale : 언어 설정 (Locale.KOREA, Locale.ENGLISH)
<br>

### 웹 애플리케이션 메시지/국제화 적용
#### 메시지
- #{...} 표현식 사용
  - #{label.item, item.itemName, etc.}
- 파라미터의 경우
  - hello.name=안녕 {0}
  - th:text="#{hello.name(${item.itemName})}"

#### 국제화
- 웹 브라우저 언어 설정에 따라 자동 적용
- _messages_en.properties_
~~~
label.item=Item
label.item.id=Item ID
label.item.itemName=Item Name
label.item.price=price
label.item.quantity=quantity
~~~
<br>

>
[출처] 스프링 MVC 2 - 김영한, 인프런