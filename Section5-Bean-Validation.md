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

> [출처] 스프링 MVC 2 - 김영한, 인프런
