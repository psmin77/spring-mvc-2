### 타임리프 소개
#### 타임리프 특징
- 서버 사이드 HTML 렌더링 (SSR)
  - 백엔드 서버에서 HTML을 동적으로 렌더링하기 위해 사용
- 내추럴 템플릿(Natural Templates)
  - 순수 HTML을 최대한 유지하면서 뷰 템플릿도 사용할 수 있음
- 스프링 통합 지원
  - 스프링의 다양한 기능을 편리하게 사용할 수 있도록 지원
    
#### 타임리프 기본 기능
- 사용 선언
  ~~~ html
  <html xmlns:th="http://www.thymeleaf.org">
  ~~~
- 기본 표현식
  - 간단한 표현
    - 변수 표현식: ${...}
    - 선택 변수 표현식: *{...}
    - 메시지 표현식: #{...}
    - 링크 URL 표현식: @{...}
    - 조각 표현식: ~{...}
  - 리터럴
    - 텍스트, 숫자, 불린(True/False), null, 리터럴 토큰
  - 문자 연산
  - 산술 연산
  - 불린 연산
  - 비교와 동등
  - 조건 연산 등
<br>

### 텍스트 - text, utext
#### 일반 텍스트 출력(text)
~~~html
<span th:text="${data}"></span>
<span>[[${data}]]</span>
~~~
- Escape
  - 출력 데이터에 HTML 태그가 있는 경우, 일반 문자인 HTML 엔티티로 변경하는 것 (기본 설정)
  - < : & lt;
  - \> : & gt;
    
#### Unescape(utext)
- 이스케이프 기능을 사용하지 않음(태그로 인식하여 사용)
~~~html
<span th:utext="${data}"></span>
<span>[(${data})]</span>
~~~
<br>

### 변수 - SpringEL
#### 변수 
- 표현식: ${...}
- (예) user 객체의 username = userA
  - Object(user)
    - user.username
    - user['username']
    - user.getUsername()
  - List(users)
    - users[0].username
    - users[0]['username']
    - users[0].getUsername()
  - Map(userMap)
    - userMap['userA'].username
    - userMap['userA']['username']
    - userMap['userA'].getUsername()
#### 지역 변수
- 표현식: _th:with_
- 선언한 태그 안에서만 사용 가능
~~~ html
<div th:with="first=${users[0]}">
  <p>첫 번째 이름: <span th:text="${first.username}"></span></p>
</div>
~~~
<br>

### 기본 객체들
- 타임리프 기본 객체들
  - ${#request}
    - HttpServletRequest 객체가 그대로 제공됨
    - request.getParameter("data") 형식으로 접근
  - ${#response}
  - ${#session}
  - ${#servletContext}
  - ${locale}
- HTTP 요청 파라미터 접근: param
  - ${param.paramData}
- HTTP 세션 접근: session
  - ${session.sessionData}
- 스프링 빈 접근: @
  - ${@bean.method()}
<br>

### 유틸리티 객체와 날짜
#### 타임리프 유틸리티 객체
- https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#expression-utility-objects
#### 유틸리티 객체 예시
- https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#appendix-b-expression-utility-objects

#### 표현식
- #message: 메시지, 국제화 처리
- #uris: URI 이스케이프 지원
- #dates: java.util.Date 서식 지원
- #calendars: java.util.Calendar 서식 지원
- #temporals: 자바8 날짜 서식 지원
- #numbers: 숫자 서식 지원
- #strings: 문자 관련 편의 기능
- #objects: 객체 관련 기능 제공
- #bools: boolean 관련 기능 제공
- #arrays: 배열 관련 기능 제공
- #lists, #sets, @maps: 컬렉션 관련 기능 제공
- #ids: 아이디 처리 관련 기능 제공
<br>

### URL 링크
- 단순 경로: @{...}
- 쿼리 파라미터
  - @{/link(param1=\${param1}, param2=${param2})}
  - () 내용이 쿼리 파라미터로 처리
- 경로 변수(path variable)
  - @{/link/{param1}/{param2}(param1=\${param1}, param2=${param2})}
  - 경로 내 변수가 있으면 () 내용이 변수로 처리
- 경로 변수 + 쿼리 파라미터
  - @{/link/{param1}(param1=\${param1}, param2${param2})}
  - 함께 사용할 수 있음
<br>

### 리터럴(Literal)
- 소스코드 내 고정된 값 
- 문자, 숫자, 불린, null
  - 문자 리터럴은 항상 ''로 감싸야 함
  - 공백이 없는 경우 생략 가능
- 리터럴 대체(Literal substitutions)
  - || 안에 입력하면 리터럴을 템플릿처럼 사용 가능
<br>

### 연산
- 비교연산
  - \> (gt)
  - < (lt)
  - \>= (ge)
  - <= (le)
  - ! (not)
  - == (eq)
  - != (neq, ne)
  
- 조건식 (자바와 동일)
- Elvis 연산자
- No-Operation
  ~~~ html
  <!-- 실행 결과: '데이터가 없습니다.' -->
  <span th:text="${nullData}?:_">데이터가 없습니다.</span>
  ~~~
  - _인 경우 타임리프가 실행되지 않고 HTML 그대로 사용
<br>

### 속성 설정
- 속성 값 설정: _th:*_
  - 기존 속성을 대체하거나 없으면 새로 만듦
- 속성 추가
  - th:attrappend - 속성 값 뒤에 추가
  - th:attrprepend - 속성 값 앞에 추가
  - th:classappend - class 속성에 추가
- checked 속성
  - 기본 HTML에서는 checked만 사용(true/false 적용 안됨)
  - th:checked=true/false 적용 가능
<br>

### 반복
- _th:each_
- List, array, iterable, enumeration, Map 객체 사용 가능
- 반복 상태 유지 기능
  - index: 0부터 시작
  - count: 1부터 시작
  - size: 전체 사이즈
  - even/odd: 홀수/짝수(boolean)
  - first/last: 처음/마지막 여부(boolean)
  - current: 현재 객체
<br>

### 조건식
- if, unless 
  - 조건이 false인 경우, 태그 자체가 렌더링 되지 않고 사라짐
- switch
  ~~~ html
  <td th:switch="${user.age}">
    <span th:case="10">10살</span>
    <span th:case="20">20살</span>
    <span th:case="*">기타</span>
  </td>
  ~~~
  - 만족하는 조건이 없을 때는 \* 
<br>

### 주석
- 표준 HTML 주석
  - <\!-- -->
- 타임리프 파서 주석
  - <\!--/* */-->
  - 타임리프 렌더링에서 해당 주석 부분이 제거됨
- 타임리프 프로토타입 주석
  - <\!--/\*/ /*/-->
  - HTML 웹 브라우저에서는 주석 처리되지만, 타임리프 렌더링에서는 태그를 실행함
<br>

### 블록
- _th:block_
- 여러 태그를 하나의 단위로 반복하거나 사용하는 경우, 블록으로 묶어서 사용할 수 있음
~~~ html
<th:block th:each="user : ${users}">
  <div>
    이름 <span th:text="${user.username}"></span>
    나이 <span th:text="${user.age}"></span>
  </div>
  <div> 
    요약 <span th:text="${user.username}+'/'+${user.age}"></span>
  </div>
</th:block>
~~~
<br>

### 자바스크립트 인라인
~~~ html
<script th:inline="javascript">
~~~
#### 텍스트 렌더링
- 렌더링 시 문자 타입인 경우 ""를 자동으로 처리 
- 이스케이프 처리 (\")
~~~ html
var username = [[${user.name}]]
<!-- 인라인 사용 전 -->
var username = userA
<!-- 인라인 사용 후 --> 
var username = "userA"
~~~

#### 내추럴 템플릿
- /*...\*/
- 인라인 사용 시 주석 부분을 태그로 실행함
~~~ html
var username2 = /*[[${user.name}]]*/ "test username";
<!-- 인라인 사용 전 -->
var username2 = /*userA*/ "test username"
<!-- 인라인 사용 후 --> 
var username2 = "userA"
~~~

#### 객체
- 객체를 자동으로 JSON 변환
- 인라인 사용 전은 객체의 toString()
- 인라인 사용 후에 객체를 JSON 변환

#### 인라인 each
- 자바스크립트 인라인에서 each 지원
~~~ html
<!-- 자바스크립트 인라인 each -->
<script th:inline="javascript">
  [# th:each="user, stat : ${users}"]
  var user[[${stat.count}]] = [[${user}]];
  [/]
</script>

<!-- 실행 결과 -->
<script>
  var user1 = {"username":"userA","age":10};
  var user2 = {"username":"userB","age":20};
  var user3 = {"username":"userC","age":30};
</script>
~~~
<br>

>
[출처] 스프링 MVC 2 - 김영한, 인프런
