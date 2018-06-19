> 스프링 부트 레퍼런스 문서를 보면서 번역 및 정리를 한 문서 입니다.  
잘못 해석한 부분이 있을 수 있으니 정확한 정보는 [Part IV. Spring Boot features](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features.html) 를 참고하시기 바랍니다.

#### 이 절에선 스프링 부트에 대해 자세히 살펴봅니다. 여기서는 사용하고 사용자가 지정할 수 있는 주요 기능에 대해 배울 수 있습니다.
#### 아직 읽어보지 않았다면, [Part II, "Getting Started"](https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html) 와 [Part III, “Using Spring Boot”](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot.html) 를 읽어보기를 바랍니다.

## 23. SpringApplication
- `SpringApplication` 클래스는 `main()`메소드 에서 시작된 스프링 어플리케이션을 어떻게든 시작하는 편리한 방법을 제공한다 . 
- 다음 예제와 같이 많은 경우에 `SpringApplication.run` 정적 메서드에 위임 할 수 있다.

```java
public  static  void main (String [] args) {
	SpringApplication.run (MySpringConfiguration. class , args);
}
```

