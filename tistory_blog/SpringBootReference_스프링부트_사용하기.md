> 스프링 부트 레퍼런스 문서 중 Part III. Using Spring Boot를 보면서 번역 및 정리를 한 문서 입니다.  
> 잘못 해석한 부분이 있을 수 있으니 정확한 정보는 [Part III. Using Spring Boot](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot) 참고하시기 바랍니다.


## 13. Build System
- 종속성 관리를 지원하는 Build System을 선택하는 것을 강력하게 추천한다.
- 이런 Build System은 "Maven Central" 저장소에 게시된 아티팩트를 사용할 수 있다.
- Maven이나 Gradle을 선택하는걸 추천함
- Ant같은 다른 Build System도 사용 가능하지만 잘 지원되지는 않음

## 13.1 종속성 관리
- 스프링 부트의 각 릴리즈 버전은 지원하는 Dependency들의 관리된 리스트(curated list)를 제공한다.
- 실제로 사용자는 빌드 구성에 Dependency들의 버전을 제공할 필요가 없다. 스프링 부트가 이를 관리하고 있음
- 스프링 부트가 업그레이드 되면 종속성들도 일관되게 업그레이드 됨
    > 스프링 부트가 추천하는 Dependency의 버전을 특정 버전으로 재정의 가능함
- curated list는 잘 정리된 제3자 라이브러리들 뿐만 아니라 스프링 부트에서 사용할 수 있는 모든 스프링 모듈들이 들어있고 Maven과 Gradle에서 사용되어지는 표준 Bills of Materials(spring-boot-dependencies)으로 제공된다.
    > 스프링 부트의 각 릴리즈 버전은 스프링 프레임워크의 기본 버전과 연관되기 때문에 스프링 프레임워크 버전은 따로 지정하지 않는 것을 강력히 추천한다.

## 13.2 메이븐
- 메이븐 사용자는 합리적인 기본값들을 얻기위해 `spring-boot-starter-parent` 프로젝트로부터 상속 받을 수 있다.
    + 기본 컴파일러 레벨을 Java 1.8 사용
    + UTF-8 소스 인코딩
    + 종속성 관리 부분(spring-boot-dependencies pom에서 상속받은)은 일반적인 종속성들의 버전을 관리하고 사용자 소유의 pom을 사용할 때 종속성들의 version tag를 생략할 수 있게 해 준다.
    + 합리적인 자원 필터링
    + 합리적인 plugin 구성(exec plugin, Git commit ID, and shade)
    + profile-specific files (for example, application-dev.properties and application-dev.yml) 포함하여 `application.properties`과 `application.yml` 에 대한 합리적인 자원 필터링
 - 참고로 `application.properties`과 `application.yml` 파일은 스프링 스타일 placeholders `${...}` 를 사용하기 때문에, 메이븐 필터링이 `@..@` 을 사용하도록 변경되었다. (이를 재정의 하고 싶으면 메이븐 속성 중 `resource.delimiter`를 셋팅하면 된다.)

### 13.2.1 Starter Parent 상속하기
- `spring-boot-starter-parent`를 상속하여 프로젝트를 설정하기 위해서 아래와 같이 `parent`를 설정해야 한다.
    ```xml
    <!-- Spring Boot로 부터 기본값 상속 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.0.BUILD-SNAPSHOT</version>
    </parent>
    ```

    > 스프링 부트 버전 지정은 필수. 스프링 부트 버전 지정이 되어야지만 starter들을 추가할 경우 버전 생략이 가능함.

- 프로젝트 설정 시 속성을 재정의 함으로써 따로 종속성을 설정할 수 있다.
    ```xml
    <!-- Spring Data release train 다른 버전을 사용하고 싶을 경우 -->
    <properties>
        <spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
    </properties>
    ```

    > `spring-boot-dependencies` pom에서 지원되는 속성목록인지 확인해야 함

### 13.2.2 Parent POM 없이 스프링 부트 사용하기
- 모든 사람이 `spring-boot-starter-parent` POM을 상속하기 원하지는 않는다.  
회사 표준을 사용해야만 하거나 모든 메이븐 설정을 명시적으로 선언하는 걸 선호할 수도 있다.
- `spring-boot-starter-parent` 사용을 원하지 않는다면 아래와 같이 `scope=import` 를 사용해서 dependency management 의 장점을 계속 유지할 수 있다. (plugin management는 이런 식으로 사용할 수 없음)
    ```xml
    <dependencyManagement>
        <dependencies>
            <dependency>
                <!-- Import dependency management from Spring Boot -->
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.1.0.BUILD-SNAPSHOT</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    ```

- 위의 예제는 속성을 사용하여 개인적으로 종속성을 재정의 할 수 없다.
- 종속성 재정의를 위해선 `spring-boot-dependencies` entry 전에 재정의 할 `dependencyManagement` entry를 추가해야 한다.
    ```xml
    <dependencyManagement>
        <dependencies>
            <!-- Spring Data release train 다른 버전을 사용하고 싶을 경우 -->
            <dependency>
                <groupId>org.springframework.data</groupId>
                <artifactId>spring-data-releasetrain</artifactId>
                <version>Fowler-SR2</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <!-- Import dependency management from Spring Boot -->
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.1.0.BUILD-SNAPSHOT</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    ```
    > 위 예제에서는 BOM으로 한정했지만 모든 종속성 유형을 같은 방식으로 재정의 할 수 있다.


### 13.2.3 스프링 부트 메이븐 플러그인 사용하기
- 스프링 부트는 프로젝트를 실행가능한 jar 파일로 패키지 할 수 있는 메이븐 플러그인을 포함한다.
    ```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    ```
    > Spring Boot starter parent pom을 사용하는 경우 사용자는 오직 플러그인을 추가만 하면 된다. parent에 정의된 설정을 변경하는게 아니라면 따로 구성을 할 필요가 없다.

## 13.5 Starters
- Starters는 당신의 어플리케이션에 포함할 수 있는 편리한 종속성 기술서들의 모임이다.
- 당신은 샘플 코드 뒤지거나 많은 종속성 기술서들을 복사 붙여넣기 할 필요 없이 one-stop-shop에서 스프링과 관련된 기술들을 모두 얻을 수 있다.
- Starters는 프로젝트를 신속하게 시작하고 실행하는데 필요한 많은 종속성들을 담고 있고, 일관되면서(종속성들끼리 충돌하지 않으면서) 추이적인 종속성 집합들도 잘 지원된다.

> <span style="color:#cc180e">What's in a name</span>
> - 모든 공식적인 starters는 유사한 패턴을 따른다; `spring-boot-starter-*` 중 `*` 위치는 어플리케이션의 특정 타입이다.
> - naming 구조는 starter를 찾을 때 도움이 될 수 있도록 의도한 것이다.
> - 많은 IDE들의 메이븐 integration은 이름으로 종속성을 검색할 수 있게 해준다.  
예를 들어, 적절한 Eclipse 또는 STS 플러그인 설치 후 POM 에디터에서 starter 전체목록을 보려면 `ctrl-space` 을 누르고 "spring-boot-starter"를 입력하면 된다.
> - [Create Your Own Starter](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-custom-starter) 부분을 설명하자면, 제3자 starter들은 `spring-boot` 로 시작하면 안된다.  
예를 들어, `thirdpartyproject` 로 불리는 제3자 starter 프로젝트는 일반적으로 `thirdpartyproject-spring-boot-starter` 라는 이름을 갖게된다.

각 종 starter들의 리스트는 [여기](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot-starter)에 있는 `Spring Boot application starters`, `Spring Boot production starters`, `Spring Boot technical starters`를 참고

## 14. 코드 구조화
스프링 부트는 특정한 코드 레이아웃이 요구되지 않는다. 그러나 도움이 되는 몇 가지 모범 사례가 있다.

## 14.1 "default" Package 사용하기
- 클래스가 `package` 선언을 포함하지 않을 때 "default package"로 고려되어야 한다.
- "default package"의 사용은 일반적으로 권장되지 않으므로 피해야 한다.  
- 모든 jar 파일로부터 모든 class를 읽기 때문에 `@ComponentScan`, `@EntityScan`, `@SpringBootApplication` 어노테이션을 사용하는 스프링 부트 어플리케이션에서 특정 문제를 일으킬 수 있다.
> 자바에서 추천하는 naming 컨벤션을 따르는 것과 반전된 도메인 이름을 사용하는 것(ex. `com.example.project`)을 추천한다. 

## 14.2 Main Application 클래스 위치 시키기
- main application 클래스는 다른 클래스보다 위에 있도록 root package에 위치 시키는 것을 추천한다.
- `@SpringBootApplication` 어노테이션은 main 클래스에 자주 선언되고, 암묵적으로 어떤 아이템들을 위한 "search package"의 base로 정해진다.  
예를 들어, 만약 JPA 어플리케이션을 작성 중이라면, `@SpringBootApplication` 가 붙은 클래스의 패키지가 `@Entity` 아이템들을 찾기 위해 사용된다.  
(`@SpringBootApplication` 가 붙은 클래스의 패키지에서만 `@Entity`어노테이션이 붙은 클래스들을 찾는다.)
- root package를 사용하면 컴포넌트 스캔을 프로젝트에만 적용할 수 있다.
> 만약 `@SpringBootApplication` 어노테이션 사용을 원하지 않는다면 `@EnableAutoConfiguration`과 `@ComponentScan` 어노테이션이 해당 동작을 정의하기 때문에 대신 사용 가능하다.

- 아래 리스트는 일반적인 레이아웃이다.
    ```text
    com
    +- example
        +- myapplication
            +- Application.java
            |
            +- customer
            |   +- Customer.java
            |   +- CustomerController.java
            |   +- CustomerService.java
            |   +- CustomerRepository.java
            |
            +- order
                +- Order.java
                +- OrderController.java
                +- OrderService.java
                +- OrderRepository.java
    ```

- 아래와 같이 `Application.java` 파일은 기본 `@SpringBootApplication` 과 함께 `main`메서드를 선언한다. 
    ```java
    package com.example.myapplication;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;

    @SpringBootApplication
    public class Application {

        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }

    }
    ```
