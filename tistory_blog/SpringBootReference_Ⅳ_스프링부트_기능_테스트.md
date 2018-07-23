> 스프링 부트 레퍼런스 문서를 보면서 번역 및 정리를 한 문서 입니다.  
잘못 해석한 부분이 있을 수 있으니 정확한 정보는 [Part IV. Spring Boot features - 43. Testing](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-testing.html) 를 참고하시기 바랍니다.  
Ⅳ장 부터는 다 보지 않고 필요한 부분만 봐야할 것 같습니다. (~~도저히 다 읽을 수가 없어요ㅠㅠㅠ~~)

# Testing

## [참고자료] JUnit & Spring 테스트 <span style="font-size:15px;color:#6db33f;">(Feat. 토비 스프링 3.1)</span>

- 2-1 : JUnit 테스트 [(예제코드)](https://github.com/wooyoung85/spring-study/tree/master/toby-vol1-test-1-JUnit%20%ED%85%8C%EC%8A%A4%ED%8A%B8)

  ```java
  public class JunitUserDaoTest {
    @Before
    public void setup() {
    }

    @Test
    public void test(){
    }

    @After
    public void tearDown(){
    }
  }
  ```

  - `@Test` 어노테이션 붙은 메소드 단위로 테스트 실행
  - 테스트마다 `@Before` (시작 전), `@After` (종료 후) 어노테이션 붙은 메소드 실행
  - `@Before` 메소드에서 정의한 ApplicationContext가 매번 생성됨

- 2-2 : 스프링 테스트 컨텍스트 프레임워크 사용 [(예제코드)](https://github.com/wooyoung85/spring-study/tree/master/toby-vol1-test-2-%EC%8A%A4%ED%94%84%EB%A7%81%20%ED%85%8C%EC%8A%A4%ED%8A%B8%20%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8%20%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC%20%EC%82%AC%EC%9A%A9)

  ```java
  @RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration(locations="/applicationContext.xml")
  public class SpringJunitUserDaoTest {

    @Autowired
    ApplicationContext context;

    ...

  }
  ```

  - `@ContextConfiguration(locations="/applicationContext.xml")` 와 `@Autowired` 를 통해 ApplicationContext 의존성 주입 받음
  - 같은 context 파일을 사용하는 테스트 클래스에선 context 공유해서 테스트 성능을 높일 수 있음. 테스트 클래스마다 다르게도 설정 가능

- 2-3 : 테스트 코드에 의한 DI [(예제코드)](https://github.com/wooyoung85/spring-study/tree/master/toby-vol1-test-3-%ED%85%8C%EC%8A%A4%ED%8A%B8%20%EC%BD%94%EB%93%9C%EC%97%90%20%EC%9D%98%ED%95%9C%20DI)

  ```java
  @RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration(locations="/applicationContext.xml")
  @DirtiesContext
  public class SpringJunitUserDaoTest {

    //applicationContext.xml과 다르게 설정 (예 : DataSource 다르게 설정)
    ...

  }
  ```

  - `@DirtiesContext` 어노테이션을 사용하여 테스트 메소드에서 컨텍스트 구성이나 상태를 변경한다는 것을 알려줌

- 2-4 : 테스트를 위한 별도 DI 설정 [(예제코드)](https://github.com/wooyoung85/spring-study/tree/master/toby-vol1-test-4-%ED%85%8C%EC%8A%A4%ED%8A%B8%EB%A5%BC%20%EC%9C%84%ED%95%9C%20%EB%B3%84%EB%8F%84%20DI%20%EC%84%A4%EC%A0%95)

  ```java
  @RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration(locations="/test-applicationContext.xml")
  public class SpringJunitUserDaoTest {

    ...

  }
  ```

  - `@ContextConfiguration(locations="/test-applicationContext.xml")` test용 applicationContext.xml 파일 설정

---

## 43. Testing

- `spring-boot-starter-test` 를 사용하여 import 할 수 있다.
- `spring-boot-test` 는 core item들을 담고 있고, `spring-boot-test-autoconfigure` 는 테스트 auto-configuration을 지원

## 43.1 Test Scope Dependencies

- **JUnit**: 자바 어플리케이션 Unit 테스트를 위한 사실상 표준 프레임워크
- **Spring Test & Spring Boot Test**: 스프링 부트 어플리케이션을 위한 Utilities와 통합 테스트 지원함
- **AssertJ**: 풍부한 가정 설정문(assertion) 라이브러리 <sup>~~assertion은 번역하니 더 어렵다~~</sup>
- **Hamcrest**: matcher objects 라이브러리 (코드의 조건을 제한하거나 결과를 예측하기도 함)
- **Mockito**: A Java mocking framework
- **JSONassert**: JSON을 위한 assertion 라이브러리
- **JsonPath**: XPath for JSON. (XPath는 XML 문서의 일부분을 선택하고 처리하기 위해 만들어진 언어)

## 43.2 Testing Spring Applications

- DI를 사용해서 코드 작성을 하면 테스트 코드 작성이 쉽다.
- `new` 연산자를 사용해서 객체를 초기화할 수 있고, 진짜 의존성 대신 mock objects를 사용할 수 있다.
- 유닛 테스트를 넘어서 Spring `ApplicationContext` 와 함께 통합테스트를 해야 할 경우 시스템의 배포나 다른 인프라 요소들의 도움 없이 테스트 하기 유용하다.
- 스프링 프레임워크는 통합 테스트를 위한 전용 테스트 모듈을 포함한다.
- 직접 `org.springframework:spring-test` dependency 선언하거나 `spring-boot-starter-test` 를 사용해서 추이적으로(transitively) pull 할 수 있다.
- 스프링 테스트를 사용해 보지 않았다면, [relevant section](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/testing.html#testing) 을 먼저 읽어야 한다.

## 43.3 Testing Spring Boot Applications

- A Spring Boot application is a Spring `ApplicationContext` (응?.. 스프링 부트 어플리케이션이 `ApplicationContext` 을 사용하고 매우 중요하다는 의미로 해석했다)
- 스프링 부트 어플리케이션을 테스트를 하기 위해서 일반적인 스프링 context를 사용하는 것 외에 특별할 건 없다.
  > Spring의 외부 속성, 로깅 및 기타 기능은 SpringApplication을 사용하여 생성 한 경우에만 기본적으로 컨텍스트에 설치된다.
- 스프링 부트 기능이 필요할 때 `spring-test` 의 `@ContextConfiguration` 를 대체하는 `@SpringBootTest` 어노테이션을 사용할 수 있다.
- `@SpringBootTest` 어노테이션은 `SpringApplication` 을 통해 테스트에 사용되는 `ApplicationContext` 을 생성하는 방식으로 작동한다.
- `@SpringBootTest` 외에 애플리케이션의 특정 부분을 잘라서 테스트하기 위해 여러개의 다른 어노테이션이 제공된다.
  > 반드시 `@RunWith(SpringRunner.class)` 와 함께 사용되어야 한다. (같이 사용하지 않으면 다 무시됨)
  
- `@SpringBootTest` 의 `webEnvironment` 속성을 사용하여 테스트를 좀 더 자세하게 정의할 수 있다.
  - **MOCK** : `WebApplicationContext`를 제공하고 가짜 서블릿 환경을 제공함. 내장된 서블릿 컨테이너를 사용하지 않음. 서블릿 API들이 클래스패스에 없으면 `ApplicationContext` 를 생성함. MOCK 설정은 `MockMvc` 기반의 어플리케이션 테스트를 위해 `@AutoConfigureMockMvc` 의 주입과 함께 사용되어 진다.
  - **RANDOM_PORT** : `ServletWebServerApplicationContext` 을 로드하고 진짜 서블릿 환경을 제공함. 내장된 서블릿 컨테이너를 구동하고 random port로 리스닝 한다.
  - **DEFINED_PORT** : `ServletWebServerApplicationContext` 을 로드하고 진짜 서블릿 환경을 제공함. 내장된 서블릿 컨테이너를 구동하고 설정된 port로 리스닝 한다. (`application.properties` 에 설정된 값 또는 `8080` 기본값)
  - **NONE** : `SpringApplication` 을 사용하면서 `ApplicationContext` 이 로드된다. 그러나 서블릿 환경은 제공되지 않는다.

  > 테스트에 `@Transactional` 를 선언했다면 기본적으로 테스트 메소드가 끝날때마다 transaction을 롤백한다. 그러나 `RANDOM_PORT` 또는 `DEFINED_PORT` 를 사용했다면 암묵적으로 실제 서블릿 컨테이너 환경을 제공하고, HTTP 클라이언트와 서버가 다른 쓰레드에서 동작하므로 transaction이 분리됨. 이 경우 서버에서 시작된 모든 Transaction은 롤백되지 않는다.

### 43.3.1 Detecting Web Application Type

- 스프링 MVC를 사용하면 일반적인 MVC 기반 어플리케이션 context가 설정됨
- 스프링 WebFlux만 가지고 있다면 WebFlux 기반 어플리케이션 context가 설정됨
- MVC랑 WebFlux가 둘 다 있으면 MVC가 우선함. 이런 상황에도 reactive 웹 어플리케이션을 하고 싶다면 `spring.main.web-application-type` 속성을 설정해야 한다.

    ```java
    @RunWith(SpringRunner.class)
    @SpringBootTest(properties = "spring.main.web-application-type=reactive")
    public class MyWebFluxTests { ... }
    ```

### 43.3.2 Detecting Test Configuration

- 스프링 테스트 프레임워크가 친숙하다면 특정 스프링 `@Configuration` 을 로드하기 위해 `@ContextConfiguration(classes=…​)` 를 사용하는게 익숙할 수 있다. 또는 테스트 내에서 중첩된 `@Configuration` 클래스를 자주 사용했을 것이다.  
(스프링 프레임워크에서는 `@ContextConfiguration(classes=…​)` 를 사용하거나 테스트 안의 중첩 `@Configuration` 을 사용하여 로드하는 Configuration을 특정지을 수 있다는 뜻)
- 스프링 부트 어플리케이션을 테스트 할 때 `@ContextConfiguration(classes=…​)` 와 중첩 `@Configuration` 은 요구되지 않는다.
- 스프링 부트의 `@*Test` 어노테이션이 명시적으로 정의하지 않아도 자동으로 주요 설정들을 찾는다.
- 이 검색 알고리즘은 `@SpringBootApplication` 또는 `@SpringBootConfiguration` 어노테이션이 붙은 클래스를 찾을 때까지 테스트가 포함된 패키지에서 작동한다.
- 합리적으로 [structured your code](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-structuring-your-code.html) 를 구성한다면 보통 메인 configuration은 찾을 수 있다.
  > [test annotation to test a more specific slice of your application](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests) 을 사용한다면 [main method’s application class](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-user-configuration) 에 특정 영역을 한정하는 Configuration 추가 설정을 피해야 한다.  
  `@SpringBootApplication` 아래 있는 component scan 설정은 slicing 작업이 예상대로 잘 작동하도록 exclude 필터를 정의한다.
  그런데 `@SpringBootApplication` 어노테이션이 붙은 클래스에서 직접 명시적으로 `@ComponentScan` 을 사용한다면 exclude 필터가 비활성화 된다. slicing을 사용하려면 재 정의 해야만 한다.  
  (결국 스프링 부트가 제공하는 `@*Test` 어노테이션을 사용하려면 메인 메서드가 있는 클래스에 `@ComponentScan({ "com.example.app", "org.acme.another" })` 같은 설정을 하지말라는 뜻)

- 주요 설정을 커스터마이징 하려면 중첩된 `@TestConfiguration` 클래스를 사용할 수 있다. 중첩된 `@Configuration` (어플리케이션의 주요 설정을 대체함) 과는 다르게 중첩된 `@TestConfiguration` 는 주요 설정에 추가된다.
  > 스프링의 테스트 프레임워크는 테스트 간 application context를 캐시한다. 테스트들이 같은 configuration을 사용한다면 잠재적으로 시간이 많이 소요되는 context 로딩 프로세스가 한번만 일어난다. 

### 43.3.3 Excluding Test Configuration

- `@SpringBootApplication` 또는 `@ComponentScan` 을 사용하여 컴포넌트 스캔을 사용한다면, 특정 테스트를 위해 생성한 최상위 Configuration 클래스가 실수로 모든 테스트에서 설정될 수 있다.
- `@TestConfiguration` 은 주요 설정을 커스터마이징 하기 위해 Inner 클래스로 사용될 수 있다.
- `src/test/java` 경로에 `@TestConfiguration` 이 붙은 클래스가 있다면 component scanning 할 때 scan되지 않는다. 이 때는 아래와 같이 해야만 설정을 불러올 수 있다.

  ```java
  @RunWith(SpringRunner.class)
  @SpringBootTest
  @Import(MyTestsConfiguration.class)
  public class MyTests {

      @Test
      public void exampleTest() {
          ...
      }

  }
  ```
  > `@SpringBootApplication` 을 사용하지 않고 `@ComponentScan` 을 직접 사용하면 `TypeExcludeFilter` 를 설정해야만 한다. 자세한 건 [Javadoc](https://docs.spring.io/spring-boot/docs/2.0.3.RELEASE/api/org/springframework/boot/context/TypeExcludeFilter.html) 참고.

### 43.3.4 Testing with a running server

- 완벽한 서버 실행을 원한다면 random ports를 사용하는 걸 추천한다.
- `@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)` 을 사용하면, 테스트 할 때마다 포트가 랜덤하게 잡힌다.
- `@LocalServerPort` 어노테이션으로 실제 주입되는 포트번호를 확인할 수 있다. [inject the actual port used](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-embedded-web-servers.html#howto-discover-the-http-port-at-runtime) 참고.
- 테스트 편의를 위해 아래와 같이 REST 호출이 필요한 테스트는 추가적으로 [WebTestClient](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/testing.html#webtestclient-tests) (실행중인 서버에 대한 상대 링크를 확인하고 응답 확인을 위한 전용 API 제공)를 `@Autowired` 할 수 있다.
- **WebTestClient는 WebFlux 환경에서만 사용 가능**

  ```java
  import org.junit.Test;
  import org.junit.runner.RunWith;

  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.boot.test.context.SpringBootTest;
  import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
  import org.springframework.test.context.junit4.SpringRunner;
  import org.springframework.test.web.reactive.server.WebTestClient;

  @RunWith(SpringRunner.class)
  @SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
  public class RandomPortWebTestClientExampleTests {

    @Autowired
    private WebTestClient webClient;

    @Test
    public void exampleTest() {
      this.webClient.get().uri("/").exchange().expectStatus().isOk()
          .expectBody(String.class).isEqualTo("Hello World");
    }

  }
  ```

- 스프링 부트는 `TestRestTemplate` 도 제공한다.
  ```java
  import org.junit.Test;
  import org.junit.runner.RunWith;

  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.boot.test.context.SpringBootTest;
  import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
  import org.springframework.boot.test.web.client.TestRestTemplate;
  import org.springframework.test.context.junit4.SpringRunner;

  import static org.assertj.core.api.Assertions.assertThat;

  @RunWith(SpringRunner.class)
  @SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
  public class RandomPortTestRestTemplateExampleTests {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    public void exampleTest() {
      String body = this.restTemplate.getForObject("/", String.class);
      assertThat(body).isEqualTo("Hello World");
    }

  }
  ```

### 43.3.7 Auto-configured Tests

- 스프링부트의 auto-configuration은 잘 동작하지만 너무 과할 때가 있다.
- 슬라이스를 자동 구성할 수 있는 어노테이션이 있다. `@...Test` 어노테이션으로 제공
- `@...Test` 어노테이션은 `ApplicationContext`와 `@AutoConfigure...` 어노테이션을 불러온다.
    ```text
    @WebMvcTest(UserController.class) 는
    @SpringBootTest(classes=UserController.class) + @AutoConfigureMockMvc 와 같다
    ```
- 슬라이스는 매우 제한되게 불러오지만 더 제거하고 싶으면 `@...Test` 어노테이션에서 제공하는 `excludeAutoConfiguration` 속성을 사용 또는 `@ImportAutoConfiguration#exclude`

## 참고자료

[![스프링 부트 2.0 Day 19. 스프링 부트 테스트 1 (@SpringBootTest)](http://img.youtube.com/vi/pnkBfsIqdK4/0.jpg)](https://www.youtube.com/watch?v=pnkBfsIqdK4) 스프링 부트 2.0 Day 19. 스프링 부트 테스트 1 (@SpringBootTest)

[![스프링 부트 2.0 Day 20. 스프링 부트 테스트 2 (랜덤 포트와 @MockBean)](http://img.youtube.com/vi/yJ_2eHBQW40/0.jpg)](https://www.youtube.com/watch?v=yJ_2eHBQW40) 스프링 부트 2.0 Day 20. 스프링 부트 테스트 2 (랜덤 포트와 @MockBean)

[![스프링 부트 2.0 Day 21. 테스트 3 @...Test 와 @AutoConfigure... 애노테이션들](http://img.youtube.com/vi/GECCfXZ0W6w/0.jpg)](https://www.youtube.com/watch?v=GECCfXZ0W6w) 스프링 부트 2.0 Day 21. 테스트 3 @...Test 와 @AutoConfigure... 애노테이션들

[![스프링 부트 2.0 Day 22. 테스트 마무리. Slice 테스트 주의할 점과 유틸리티](http://img.youtube.com/vi/Tb0guL8hURs/0.jpg)](https://www.youtube.com/watch?v=Tb0guL8hURs) 스프링 부트 2.0 Day 22. 테스트 마무리. Slice 테스트 주의할 점과 유틸리티