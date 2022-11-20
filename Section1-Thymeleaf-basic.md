### 타임리프 소개
#### 타임리프 특징
- 서버 사이드 HTML 렌더링 (SSR)
  - 백엔드 서버에서 HTML을 동적으로 렌더링하기 위해 사용
- 내츄럴 템플릿(Natural Templates)
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

>
[출처] 스프링 MVC 2 - 김영한, 인프런
