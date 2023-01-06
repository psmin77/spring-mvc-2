## Bean Validation
### 기본 소개
- 검증 로직을 공통화 및 표준화하여 애노테이션으로 적용할 수 있게 함
- Bean Validation 2.0(JSR-380)이라는 기술 표준
  - 검증 애노테이션과 여러 인터페이스의 모음
  - jakarta.validation-api : Bean Validation 인터페이스
  - hibernate-validator : 구현체
  
~~~java
public class Item {
  private Long id;
  
  @NotBlank
  private String itemName;
  
  @NotNull
  @Range(min=1000, max=1000000)
  private Integer price;
  
  @NotNull
  @Max(9999)
  private Integer price;
}
~~~
- 검증 애노테이션 
  - @NotBlank : 빈 값 + 공백 허용하지 않음
  - @NotNull : Null 허용하지 않음
  - @Range(min,max) : 범위 설정
  - @Max(0) : 최대값 설정
<br>

### 스프링 적용
- _ValidationItemController_
  - 검증기(ItemValidator) 제거
  - @Valid(javax), @Validated(spring) : 스프링 부트 글로벌 Validator 적용
  - cf. 다른 글로벌 Validator 설정 시에는 작동하지 않으므로 주의
- 검증 순서
  - @ModelAttribute 각각의 필드에 타입 변환 시도
  - 실패하면 typeMismatch로 FieldError 추가
  - (바인딩에 성공한 필드만) Bean Validator 적용
<br>

### 에러 코드
- @NotBlank(Range) : 구체적 > 범용적 범위
  - NotBlank(Range).item.itemName
  - NotBlank(Range).itemName
  - NotBlank(Range).java.lang.String
  - NotBlank(Range)
- _errors.properties_
  ~~~
  #Bean Validation 추가
  NotBlank={0} 공백X
  Range={0}, {2}~{1} 허용
  Max={0}, 최대 {1}
  ~~~
  - {0} 필드명
  - {1},{2}... 각 애노테이션
- BeanValidation 순서
  - 생성된 메시지 순서대로 messageSource에서 찾기
  - 애노테이션의 message 속성 사용
    - @NotBlank(message="공백은 입력할 수 없습니다.")
  - 라이브러리가 제공하는 기본 값 사용
<br>

## 오브젝트 오류
- @ScriptAssert() : 특정 필드가 아닌 오브젝트 관련 오류에 사용
- 제약이 많고 복잡하여 권장하지 않음
- 자바 코드(조건식 등)로 직접 작성하는 것 권장
<br>

## 수정 폼 적용
### 수정 요구사항
- 등록 수량 최대 9999 제한, 수정 수량 제한 없음
- 등록 id 존재하지 않음, 수정 id 필수 값

### 수정 방안
1. BeanValidation의 groups 기능
2. ItemSaveForm, ItemUpdateForm으로 모델 객체를 나누어 사용

### groups
- 저장/수정용 groups(interface) 생성
  - SaveCheck / UpdateCheck
- 모델 객체 애노테이션 적용
  - SaveCheck.class / UpdateCheck.class
  - _(groups = {SaveCheck.class, UpdateCheck.class})_
- 저장 로직에 groups 적용
  - _@Validated(SaveCheck.class / UpdateCheck.class)_
  - cf. @Valid에는 groups가 없음

<br>

> [출처] 스프링 MVC 2 - 김영한, 인프런
