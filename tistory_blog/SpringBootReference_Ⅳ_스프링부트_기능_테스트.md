> 스프링 부트 레퍼런스 문서를 보면서 번역 및 정리를 한 문서 입니다.  
잘못 해석한 부분이 있을 수 있으니 정확한 정보는 [Part IV. Spring Boot features](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features.html) 를 참고하시기 바랍니다.  
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
- `spring-boot-test` core item들을 담고 있고, `spring-boot-test-autoconfigure` 는 테스트 auto-configuration을 지원

## 43.1 Test Scope Dependencies

- JUnit: The de-facto standard for unit testing Java applications.
- Spring Test & Spring Boot Test: Utilities and integration test support for Spring Boot applications.
- AssertJ: A fluent assertion library.
- Hamcrest: A library of matcher objects (also known as constraints or predicates).
- Mockito: A Java mocking framework.
- JSONassert: An assertion library for JSON.
- JsonPath: XPath for JSON.

## 43.2 Testing Spring Applications

- DI를 사용해서 코드 작성을 하면 테스트 코드 작성이 쉽다.
- 유닛 테스트를 넘어서 Spring `ApplicationContext` 와 함께 통합테스트를 해야 할 경우 시스템의 배포나 다른 인프라 요소들의 도움 없이 테스트 하기 유용하다.
- 직접 `org.springframework:spring-test` dependency 선언하거나 `spring-boot-starter-test` 를 사용해서 transitively pull 할 수 있다.

## 43.3 Testing Spring Boot Applications

- SpringBoot에서 제공하는 `@SpringBootTest` 어노테이션은 `spring-test` `@ContextConfiguration` 를 대체할 수 있고, `ApplicationContext` 를 생성한다.
- 반드시 `@RunWith(SpringRunner.class)` 와 함께 사용되어야 한다. (같이 사용하지 않으면 다 무시됨)
- `@SpringBootTest` 의 `webEnvironment` 속성을 사용하여 테스트를 좀 더 자세하게 정의할 수 있다.
  - MOCK
  - RANDOM_PORT
  - DEFINED_PORT
  - NONE

### 43.3.1 Detecting Web Application Type

- MVC랑 WebFlux가 있으면 MVC가 우선함

    ```java
    @RunWith(SpringRunner.class)
    @SpringBootTest(properties = "spring.main.web-application-type=reactive")
    public class MyWebFluxTests { ... }
    ```

### 43.3.2 Detecting Test Configuration

- 스프링에서는 `@ContextConfiguration(classes=…​)` 를 사용하거나 테스트 안의 중첩 `@Configuration` 을 사용하여 로드하는 설정을 특정지을 수 있다.
- 스프링부트는 `@*Test` 를 사용하여 자동으로 주요 설정들을 구성한다.
- `@TestConfiguration` 은 `@Configuration` 과는 다르게 주요 설정을 대체하지 않고 추가된다.
- 스프링 프레임워크는 테스트 간 application context를 공유한다. (테스트가 많다면 application context를 생성하는데 걸리는 시간은 줄어듬)

### 43.3.3 Excluding Test Configuration

- `src/test/java` 경로에 `@TestConfiguration` 이 붙은 클래스가 있다면 component scanning 할 때 scan되지 않는다.

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

### 43.3.7 Auto-configured Tests

- 스프링부트의 auto-configuration은 잘 동작하지만 너무 과할 때가 있다.
- 슬라이스를 자동 구성할 수 있는 어노테이션이 있다. `@...Test` 어노테이션으로 제공
- `@...Test` 어노테이션은 `ApplicationContext`와 `@AutoConfigure...` 어노테이션을 불러온다.
    ```text
    @WebMvcTest(UserController.class) 는
    @SpringBootTest(classes=UserController.class) + @AutoConfigureMockMvc 와 같다
    ```
- 슬라이스는 매우 제한되게 불러오지만 더 제거하고 싶으면 `@...Test` 어노테이션에서 제공하는 `excludeAutoConfiguration` 속성을 사용 또는 `@ImportAutoConfiguration#exclude`
