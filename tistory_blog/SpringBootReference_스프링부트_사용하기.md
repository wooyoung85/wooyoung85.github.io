> 스프링 부트 레퍼런스 문서 중 Part III. Using Spring Boot를 보면서 번역 및 정리를 한 문서 입니다.  
> 잘못 해석한 부분이 있을 수 있으니 정확한 정보는 [Part III. Using Spring Boot](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot) 를 참고하시기 바랍니다.

#### 이 절은 스프링 부트를 사용법에 대해 자세히 설명합니다.
#### 빌드 시스템, auto-configuration, 어플리케이션 실행 방법과 같은 주제가 포함되어 있고 스프링 부트 모범 사례를 다루게 됩니다.
#### 스프링 부트에 특별히 필요한 것은 없지만(사용할 수 있는 또 다른 라이브러리일 뿐), 따라했을 때 개발 프로세스를 좀 더 쉽게 만들 수 있는 몇 가지 권장 사항이 있습니다.
---

## 13. Build System
- 종속성 관리를 지원하고 "Maven Central" 저장소에 게시된 아티팩트를 사용할 수 있는 Build System을 선택하는 것이 좋다.
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
    + profile-specific files (예를 들면, application-dev.properties and application-dev.yml) 포함하여 `application.properties`과 `application.yml` 에 대한 합리적인 자원 필터링
 - 참고로 `application.properties`과 `application.yml` 파일은 스프링 스타일 placeholders `${...}` 를 사용하기 때문에, 메이븐 필터링이 `@..@` 을 사용하도록 변경되었다. (이를 재정의 하고 싶으면 메이븐 속성 중 `resource.delimiter`를 설정하면 된다.)

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

## 15. Configuration Classes
- 스프링 부트는 Java 기반 설정 파일을 선호한다.
- XML 기반의 `SpringApplication` 사용도 가능하긴하지만, 주된 설정파일(primary source)은 단일 `@Configuration` 클래스로 구성하는 것을 추천한다.(Java 기반 설정 파일을 기본으로 하고 XML이나 properties 기반의 설정 파일을 끌어다 쓰는 방식을 추천)
- 보통 `main` 메서드를 정의하고 있는 클래스가 primary `@Configuration` 어노테이션을 붙일 수 있는 좋은 후보이다.
> XML을 사용한 많은 스프링 설정파일 예제들이 인터넷 상에 돌아다니고 있다. 가능하다면 항상 자바 기반의 설정 파일을 사용하는걸 시도해라. `Enable*` 어노테이션으로 찾는 것이 좋은 시작점이 될 수 있다.

## 15.1 추가적인 Configuration Classes 불러오기
- 모든 `@Configuration` 을 단일 클래스에 넣을 필요는 없다.
- `@Import` 어노테이션을 사용하여 추가적인 설정 클래스들을 불러올 수 있다.
- 또는 `@ComponentScan` 을 사용하여 `@Configuration` 클래스들을 포함한 모든 스프링 컴포넌트들을 자동으로 읽어들이게 할 수 있다. 

## 15.2 XML 설정 불러오기
- XML 기반의 설정을 반드시 사용해야 하는 경우 `@Configuration` 클래스로 시작하는 것이 좋다.(Java 기반 설정 파일을 기본으로 하는 걸 추천하고 있음)
- XML 기반의 설정 파일을 로드하기 위해서 `@ImportResource` 어노테이션을 사용할 수 있다.

## 16. Auto-configuration
- 스프링 부트 auto-configuration 은 자동으로 사용자가 추가한 종속성을 기반으로 스프링 어플리케이션을 구성하는 것을 시도한다.
- 예를 들면, `HSQLDB` 가 classpath에 있고(dependency 추가) 데이터베이스 연결 bean을 수동으로 구성하지 않았다면 스프링 부트가 in-memory 데이터베이스를 자동으로 설정한다.
- `@EnableAutoconfiguration` 또는 `@SpringBootApplication` (@EnableAutoConfiguration, @Configuration, @ComponentScan 포함하고 있음) 어노테이션을 `@Configuration` 클래스들 중 하나에 추가하여 auto-configuration을 사용해야 한다.
> `@SpringBootApplication` 또는 `@EnableAutoConfiguration` 어노테이션은 하나만 등록해야 한다. 일반적으로 주요(primary) `@Configuration` 클래스에만 둘 중 하나를 추가하는 것이 좋다.

## 16.1 점차적으로 Auto-configuration으로 대체
- Auto-configuration은 비 침투적이다.(필수요소는 아니다)
- 어느시점이 되면 auto-configuration의 특정 부분을 사용자가 직접 설정을 정의하기 시작할 수 있다. 예를 들어, `DataSource` bean을 직접 추가한다면 기본 내장된 데이터베이스 더 이상 자동으로 등록되지 않는다.
- 현재 어떤 auto-configuration이 왜 적용됐는지 보고 싶다면 어플리케이션을 시작할 때 `--debug` 스위치를 켜고 실행하면 된다. 그렇게 하면 core loggers의 선택에 대한 디버그 로그가 활성화 되고 콘솔에 conditions report가 기록된다.

## 16.2 특정 Auto-configuration Classes 비활성화 하기
- auto-configuration classes가 적용될 필요가 없다면 `@EnableAutoConfiguration` 의 exclude 속성을 사용할 수 있다.
```java
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;
import org.springframework.context.annotation.*;

@Configuration
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {
}
```

- 클래스가 classpath에 없다면 `excludeName` 속성을 사용하고 완전한 이름을 지정해야 한다.
- 마지막으로 `spring.autoconfigure.exclude` 속성을 사용하여 제외할 auto-configuration classes 리스트를 컨트롤 할 수 있다.
> 어노테이션이나 속성을 사용해서 제외 항목을 정의할 수 있다.

## 17. 스프링 Bean과 의존성 주입
- bean과 주입된 종속성들을 정의하기 위해 모든 표준 스프링 프레임워크 기술을 자유롭게 사용할 수 있다.
- 간단히 말하자면, `@ComponentScan` (bean을 찾기 위해) 쓰는 것과 `@Autowired` (생성자 주입을 위해) 를 쓰는 것을 자주 본다.
- 위에서 제안한대로 코드를 구조화 했다면(root package 밑에 application 클래스를 두었다면) arguments 없이 `@ComponentScan` 을 추가할 수 있다.
- 어플리케이션 컴포넌트들 모두(`@Component`, `@Service`, `@Repository`, `@Controller` etc.) 자동으로 스프링 Bean에 등록된다.
- 아래 예제는 `RiskAssessor` 빈을 얻기위해 생성자 주입을 사용하는 `@Service` 빈을 보여주고 있다.
```java
package com.example.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class DatabaseAccountService implements AccountService {

	private final RiskAssessor riskAssessor;

	@Autowired
	public DatabaseAccountService(RiskAssessor riskAssessor) {
		this.riskAssessor = riskAssessor;
	}

	// ...

}
```
- 아래와 같이 생성자가 하나라면 `@Autowired`를 생략할 수 있다.
```java
@Service
public class DatabaseAccountService implements AccountService {

	private final RiskAssessor riskAssessor;

	public DatabaseAccountService(RiskAssessor riskAssessor) {
		this.riskAssessor = riskAssessor;
	}

	// ...

}
```
> 생성자 주입 방식을 사용하면서 `riskAssessor` 필드에 `final` 을 표시하여 이후에 변경될 수 없음을 알리는 방법을 확인해라.

## 18. @SpringBootApplication 어노테이션 사용하기
- 많은 스프링 부트 개발자들은 auto-configuration, component scan 그리고 "application class"에 추가 설정을 정의할 수 있는 것을 좋아한다.
- 단독 `@SpringBootApplication` 어노테이션을 사용하여 아래 3가지 기능을 사용할 수 있다.
  + `@EnableAutoConfiguration` : 스프링 부트의 auto-configuration 메카니즘을 활성화 한다.
  + `@ComponentScan` : 어플리케이션 클래스가 위치한 package에서 `@Component` 어노테이션이 달린 클래스들을 스캔하는 기능을 활성화 한다.
  + `@Configuration` : 추가 빈을 컨텍스트에 등록하거나 추가 구성 클래스를 불러올 수 있다.

- 아래 예제처럼 `@SpringBootApplication` 어노테이션은 `@Configuration`, `@EnableAutoConfiguration`, `@ComponentScan` 을 사용하는 것과 같다. (사람들이 위 3개 어노테이션을 함께 많이 써서 묶어버렸다고 한다.)
```java
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
```
> `@SpringBootApplication` 은 `@EnableAutoConfiguration`과 `@ComponentScan`의 속성을  커스터마이징 하는 별칭을 제공한다.

> - 위 3가지 기능이 필수는 아니고 하나의 어노테이션을 대체하도록 선택할 수 있다.  
 예를 들어, 어플리케이션에서 컴포넌트 scan을 하지 않을 수 있다. (@ComponentScan 어노테이션을 @Import 어노테이션으로 대체)
>```java
>package com.example.myapplication;
>
>import org.springframework.boot.SpringApplication;
>import org.springframework.context.annotation.ComponentScan
>import org.springframework.context.annotation.Configuration;
>import org.springframework.context.annotation.Import;
>
>@Configuration
>@EnableAutoConfiguration
>@Import({ MyConfig.class, MyAnotherConfig.class })
>public class Application {
>
>	public static void main(String[] args) {
>			SpringApplication.run(Application.class, args);
>	}
>
>}
>```

## 19. 어플리케이션 실행하기
- 어플리케이션을 jar 로 패키징하고 내장 HTTP 서버를 사용하는 가장 큰 장점 중에 하나는 어느곳에서나 어플리케이션을 실행할 수 있다는 점이다.
- 스프링 부트 어플리케이션 디버깅도 쉽다.
- 특정 IDE 플러그인이나 확장 기능이 필요 없다.
> 이번 section에서는 오직 jar 파일 기반의 패키징을 다루게 된다. 어플리케이션을 war 파일로 패키징할 경우, 당신은 서버와 IDE 문서를 참조해야만 한다.

## 19.1 IDE에서 실행
- IDE에서 간단한 자바 어플리케이션처럼 스프링 부트 어플리케이션을 실행할 수 있다. 그러나 먼저 프로젝트 불러오기부터 진행해야 한다.
- 불러오기(Import) 단계는 당신의 IDE와 빌드 시스템에 따라 다르다.
- 대부분의 IDE는 메이븐 프로젝트를 직접 가져올 수 있다.  
예를 들면, 이클립스 사용자들은 `File`메뉴에서 `Import` → `Existing Maven Projects` 를 선택하여 메이븐 프로젝트를 불러올 수 있다.
- IDE에서 프로젝트를 직접 불러올 수 없다면 빌드 플러그인을 사용하여 IDE 메타 데이터를 생성할 수 있다. Maven은 이클립스와 인텔리J 아이디어 용 플러그인이 포함되어 있고 Gradle은 다양한 IDE 용 플러그인을 제공하고 있다.

> 급하게 웹 어플리케이션을 두번 실행한다면, "Port already in use" 에러를 보게 된다. STS 사용자들은 `Run` 버튼 대신 `Relaunch` 버튼을 사용하여 기존 인스턴스가 닫혀 있는지 확인할 수 있다.

## 19.2 패키징 된 어플리케이션 실행하기
- 실행가능한 jar 파일을 만들기 위해 스프링 부트 Maven 혹은 Gradle 플러그인을 사용하고 있다면, `java -jar` 를 사용하여 어플리케이션을 실행할 수 있다.
```console
$ java -jar target/myapplication-0.0.1-SNAPSHOT.jar
```
- 원격 디버깅 지원을 사용하도록 설정한 상태에서 패키징 된 어플리케이션의 실행이 가능하다. 아래와 같이 하면 패키징 된 어플리케이션에 디버거를 붙일 수 있다.
```console
$ java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n \
       -jar target/myapplication-0.0.1-SNAPSHOT.jar
```

## 19.3 메이븐 플러그인 사용하기
- 스프링 부트 메이븐 플러그인이 포함하고 있는 `run`의 목표는 어플리케이션을 신속하고 정확하게 컴파일하고 실행하는 것이다.
- IDE에서와 마찬가지로 어플리케이션은 분리된 형태로 실행된다.
- 아래 예제는 스프링 부트 어플리케이션을 실행하기 위한  전형적인 메이븐 명령어이다.
```console
$ mvn spring-boot:run
```
- `MAVEN_OPTS` 환경 변수도 사용할 수 있다.
```console
$ export MAVEN_OPTS=-Xmx1024m
```

## 19.5 Hot Swapping
- 스프링 부트 어플리케이션은 일반 자바 어플리케이션이 때문에 JVM hot-swapping을 즉시 실행 할 수 있다.  (Hot Swapping이란? 실행을 중단하지 않고 프로그램의 실행 코드를 변경하는 기능)
- JVM hot swapping은 대체할 수 있는 바이트코드로 다소 제한된다.
- 좀 더 완벽한 솔루션을 위해선 [JRebel](https://zeroturnaround.com/software/jrebel/) 을 사용할 수 있다.
- `spring-boot-devtools` 모듈은 빠르게 어플리케이션을 재시작하는 지원한다. 
- 자세한 내용은 [Chapter 20. Developer Tools](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html) 에서 볼 수 있다.

## 20. Developer Tools
- 스프링 부트는 좀 더 쾌적하게 개발할 수 있게 해주는 추가적인 도구들이 포함되어 있다.
- `spring-boot-devtools` 모듈은 모든 프로젝트에 포함될 수 있고 개발 시에 유용한 추가적인 기능을 제공한다.
- devtools 지원을 포함하기 위해선, 아래와 같이 의존성을 추가하면 된다.  
- **Maven**
    ```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
    ```
> - 완전히 패키징 된 어플리케이션을 실행할 경우 개발자 도구는 자동으로 비활성화 된다.
> - `java -jar` 로 실행되거나 특정 클래스로더로 시작되는 경우 운영 모드의 어플리케이션으로 인식된다.
> - 메이븐의 optional하게 의존성 플래그 지정하기 또는 Gradle의 `compileOnly` 를 사용하는 것은 프로젝트를 의존하는 다른 모듈에 devtools이 추이적으로 적용되지 않도록  방지하는 가장 좋은 방법이다.  
*(spring-boot-devtools 사용 중인 A프로젝트를 B프로젝트가 의존하여 사용하는 경우 B프로젝트까지 spring-boot-devtools가 적용되게 하지 않으려면 메이븐의 경우 `<optional>` 속성을 "true" 로 하면 된다.)*

> 리패키징 된 아카이브에는 기본적으로 devtools가 포함되어 있지 않다. 원격 devtools 기능을 사용하기 원한다면(원격 디버깅을 하고 싶다면), 빌드 속성 중 `excludeDevtools` 을 비활성화해야 한다. 이 속성은 Maven과 Gradle 모두 지원한다.

## 20.1 Property Defaults
- 스프링 부트에서 지원하는 몇몇 라이브러리는 성능을 향상시키기 위해 cache를 사용한다. 예를 들어, [template engines](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-template-engines) 은 반복적으로 템플릿 파일을 parsing 하는 것을 방지하기 위해 컴파일 된 템플릿들을 cache한다.
- 스프링 MVC는 정적 리소스들을 전달할 때 response에 HTTP cashing 헤더를 추가할 수 있다.
- cashing은 운영환경에선 아주 유용하지만, 개발하는 동안에는 생산성이 저하시킨다. cashing을 하게 되면 어플리케이션에 방금 변경된 내용을 볼 수 없다. 이런 이유로 spring-boot-devtools는 기본적으로 cashing 옵션을 비활성화 한다.
- Cache 옵션은 `application.properties` 파일의 설정에 의해 구성된다. 예를 들어, Thymeleaf는 `spring.thymeleaf.cache` 속성을 제공한다.
- 속성들을 수동으로 설정하기 보다, `spring-boot-devtools` 모듈이 자동으로 개발 시에 합리적인 구성을 적용하게끔 하는 것이 좋다.
> devtools에 적용되는 전체 목록은 [DevToolsPropertyDefaultsPostProcessor](https://github.com/spring-projects/spring-boot/blob/v2.0.2.RELEASE/spring-boot-project/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/env/DevToolsPropertyDefaultsPostProcessor.java) 를 참조
> ```java
> private static final Map<String, Object> PROPERTIES;
>
> static {
> 	Map<String, Object> devToolsProperties = new HashMap<>();
> 	devToolsProperties.put("spring.thymeleaf.cache", "false");
> 	devToolsProperties.put("spring.freemarker.cache", "false");
> 	devToolsProperties.put("spring.groovy.template.cache", "false");
> 	devToolsProperties.put("spring.mustache.cache", "false");
> 	devToolsProperties.put("server.servlet.session.persistent", "true");
> 	devToolsProperties.put("spring.h2.console.enabled", "true");
> 	devToolsProperties.put("spring.resources.cache.period", "0");
> 	devToolsProperties.put("spring.resources.chain.cache", "false");
> 	devToolsProperties.put("spring.template.provider.cache", "false");
> 	devToolsProperties.put("spring.mvc.log-resolved-exception", "true");
> 	devToolsProperties.put("server.servlet.jsp.init-parameters.development", "true");
> 	devToolsProperties.put("spring.reactor.stacktrace-mode.enabled", "true");
> 	PROPERTIES = Collections.unmodifiableMap(devToolsProperties);
> }
> ```

## 20.2 자동 재시작
- `spring-boot-devtools` 를 사용하는 어플리케이션은 classpath에 있는 파일이 변경될 때마다 자동으로 재시작 한다.
- 자동 재시작은 코드 변경에 대한 매우 빠른 피드백 루프를 제공하기 때문에 IDE 상에서 작업할 때 유용한 기능이다.
- 기본적으로 폴더가 가리키는 classpath 상에 있는 모든 항목들의 변화를 모니터링한다.
- static 자산들과 view template들과 같은 특정 리소스들은 재시작 할 필요가 없다.
> <span style="color:#cc180e">Triggering a restart</span>
> - DevTools는 classpath 리소스들을 모니터링하기 때문에 재시작하는 유일한 방법은 classpath를 업데이트 하는 것이다.
> - classpath를 업데이트하는 방법은 사용 중인 IDE에 따라 다르다.
> - 이클립스의 경우 수정된 파일을 저장할 때 classpath가 업데이트되고 재시작하게 된다.
> - IntelliJ IDEA의 경우 프로젝트 빌드(`Build -> Build Project`)를 하면 동일한 효과를 낸다.

> - forking이 활성화되어 있는 동안, DevTools가 제대로 동작하려면 어플리케이션 격리된 classloader가 필요하기 때문에 지원되는 빌드 플러그인(Maven, Gradle)을 사용하여 어플리케이션을 시작할 수도 있다. (fork : 실행 프로세스를 분기해야하는지 여부를 나타내는 플래그)
> - 기본적으로 Gradle과 Maven은 classpath에서 DevTools를 찾을 때 이를 수행한다.

> - 자동 재시작은 LiveReload를 사용할 때 매우 잘 동작한다. (자세한 내용은 [See the LiveReload section](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html#using-boot-devtools-livereload) 참조)
> - JRebel을 사용한다면 동적 class reload를 위해 자동 재시작이 비활성화 된다.
> - 다른 devtools 기능들(LiveReload와 property overrides 같은)은 계속 사용된다.

> - DevTools는 재시작 하는 동안 어플리케이션을 닫기 위해  application context의 shutdown hook을 사용한다.
> - shutdown hook을 비활성화 시키면(`SpringApplication.setRegisterShutdownHook(false)`) DevTools 재시작 기능이 올바르게 작동하지 않는다.

> - classpath에 있는 항목들이 변경되었을 때 재시작 해야하는지 여부를 결정할 때 DevTools는 자동적으로 `spring-boot`, `spring-boot-devtools`, `spring-boot-autoconfigure`, `spring-boot-starter` 이름을 가진 프로젝트들을 무시하게 된다.

> - DevTools는 `ApplicationContext` 에서 사용하는 `ResourceLoader` 를 커스터마이징 해야 한다.
> - 어플리케이션에서 이미 제공한 경우에는 wrapping된다.
> - 직접 `ApplicationContext` 에 있는 `getResource` 메서드를 재정의 하는 것은 지원하지 않는다.

> <span style="color:#cc180e">Restart vs Reload</span>
> - 스프링 부트가 제공하는 재시작 기술은 두 개의 classloader를 사용하여 작동한다.
> - 바뀌지 않는 클래스들(예를 들어, 제3자 jar 파일들(라이브러리들))은 *base* classloader에 의해 로드된다.
> - 활발하게 개발 중인 클래스들은 *restart* classloader에 의해 로드된다.
> - 어플리케이션이 재시작할 때, *restart* classloader는 버려지고 새로운 classloader가 생성된다.
> - 두 개의 classloader를 사용하는 방식은 *base* classloader가 이미 사용가능한 상태이고 채워진 상태이기 때문에 어플리케이션의 재시작이 일반적으로 "cold starts" 보다 빠르다.
> - 재시작이 충분하게 빠르지 않거나 classloading 이슈가 생겼다면, ZeroTurnaround사의 JRebel을 사용하여 reloading하는 방법을 고려해 볼 수 있다.
> - JRebel은 reloading을 원활하게 하기 위해 클래스를 로드할 때 클래스를 다시 작성한다.

### 20.2.1 condition evaluation 변화 기록하기
- 기본적으로, 어플리케이션이 재시작할 때마다, *CONDITION EVALUATION DELTA* report가 기록된다. 이 리포트는 bean 추가 및 제거 혹은 configuration 속성들을 설정하는 것과 같은 변경이 있을 때 auto-configuration에 대한 변경 사항을 표시한다.
- 이 리포트를 비활성화하려면 아래와 같이 하면 된다.
```
spring.devtools.restart.log-condition-evaluation-delta=false
```

### 20.2.2 Resources 제외하기
- 특정 resource들은 변경될 때 재시작과 연결할 필요가 없다. 예를 들면, Thymeleaf 템플릿은 바로 수정될 수 있다.
- 기본적으로, `/META-INF/maven`, `/META-INF/resources`, `/resources`, `/static`, `/public`, `/templates` 에 있는 resource들의 변경은 재시작과 연결되지 않는다. 하지만 [live reload](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html#using-boot-devtools-livereload)와는 연결된다.
- Resources 제외하는 것에 대해 커스터마이징하고 싶다면, `spring.devtools.restart.exclude` 속성을 사용할 수 있다. 예를 들어, `/static` 과 `/public` 만 제외하고 싶다면 아래와 같이 속성을 설정하면 된다.
```
spring.devtools.restart.exclude=static/**,public/**
```
> 기본값을 유지하면서 제외항목을 추가하고 싶다면, `spring.devtools.restart.additional-exclude` 속성을 사용하면 된다.

### 20.2.3 Watching Additional Paths
- classpath에 없는 파일들이 변경될 때 어플리케이션을 restart하거나 reload해야 할 수 있다.
- classpath에 없는 파일들의 변화를 감지하기 위해 추가적인 경로 설정은 `spring.devtools.restart.additional-paths` 속성을 사용하면 된다.
- [위에서 설명한](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html#using-boot-devtools-restart-exclude) `spring.devtools.restart.exclude` 를 사용하여 추가 경로 아래의 변경 사항이 전체 재시작 할 지 live reload할 지를 제어할 수 있다.

### 20.2.4 재시작 비활성화
- 재시작 기능의 사용을 원하지 않는다면, `spring.devtools.restart.enabled` 속성을 사용하여 비활성화 할 수 있다.
- 대부분의 경우에는, `application.properties` 파일에 이 속성을 설정한다.(그렇게 하면 classloader가 초기화 되더라도 파일 변경을 감지하지 않는다.)
- 재시작 기능을 *completely* 하게 비활성화 해야 한다면(예:특정 라이브러리와 함께 동작하지 않기 때문에), `SpringApplication.run(…​)` 을 호출하기 전에 `System` 의 `spring.devtools.restart.enabled` 속성을 `false` 로 해야 한다.
```java
public static void main(String[] args) {
	System.setProperty("spring.devtools.restart.enabled", "false");
	SpringApplication.run(MyApp.class, args);
}
```
### 20.2.5 Trigger File 사용하기
- 변경된 파일을 지속적으로 컴파일하는 IDE로 작업한다면, 특정 시간에만 재시작으로 연결하는 것이 좋다. (이클립스의 경우인 듯)
- 이렇게 하기 위해,  "trigger file" 을 사용할 수 있다. 이 파일은 실제로 재시작 연결을 check할 때 수정되어 있어야 하는 파일이다. (즉 특정 파일이 변경되었을 때만 재시작이 연결되도록 설정할 수 있다.)
- 파일을 변경하면 check가 시작되고(재시작과 연결된 파일인지 아닌지 check) Devtools가 뭔가를 해야한다고 감지하는 경우에만 재시작이 일어난다.
- trigger file을 사용하기 위해, trigger 파일 경로를 `spring.devtools.restart.trigger-file` 속성에 설정

> 모든 프로젝트가 동일한 방식으로 작동하도록 `spring.devtools.restart.trigger-file` 의 설정을 [global setting](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html#using-boot-devtools-globalsettings) 으로 할 수 있다.

### 20.2.6 재시작 Classloader 커스터마이징 하기
- [Restart vs Reload](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html#using-spring-boot-restart-vs-reload) 에서 설명했듯이, 재시작은 2개의 classloader로 구현되었다. 
- 대부분의 어플리케이션에서, 이런 접근방식을 잘 동작한다. 하지만 가끔 classloading 이슈가 생길 수 있다.
- 기본적으로, IDE상에 열려있는 프로젝트는 "restart" classloader로 로드되고, 일반 `.jar` 파일은 "base" classloader로 로드된다.
- IDE상에 모든 모듈을 불러오지 않고 멀티 프로젝트 구조로 작업하는 경우(*A 모듈이 B모듈을 jar 파일 형태로 참조하는 경우*), 커스터마이징이 필요할 수 있다. 이를 위해 `META-INF/spring-devtools.properties` 파일을 생성할 수 있다.
- `spring-devtools.properties` 파일은 `restart.exclude` 와 `restart.include` 로 시작하는 속성들을 담을 수 있다.
- `include` 요소들은 "restart" classloader에다 끌어올려야 할 항목이고, `exclude` 요소들은 "base" classloader 로 밀어넣어야 하는 항목이다.
- 속성의 값은 아래 예제와 같이 classpath에 적용되는 정규식 패턴이다.
```
restart.exclude.companycommonlibs=/mycorp-common-[\\w-]+\.jar
restart.include.projectcommon=/mycorp-myproj-[\\w-]+\.jar
```
> 모든 속성의 key는 유일해야 한다. 속성은 `restart.include.` 혹은 `restart.exclude.` 로 시작해야만 한다.

> classpath의 모든 `META-INF/spring-devtools.properties` 는 로드된다. 프로젝트 내부 또는 프로젝트가 사용하는 라이브러리들 안에 있는 파일들을 패키징 할 수 있다. (프로젝트가 사용하는 라이브러리 안에 들어 있는 `META-INF/spring-devtools.properties` 파일도 읽어들여서 패키징하는데 사용할 수 있다)

### 20.2.7 Known Limitations
- 재시작은 표준 `ObjectInputStream` 을 사용하여 역 직렬화 된 객체와는 잘 작동하지 않는다.
- 데이터를 역 직렬화 해야만 한다면, `Thread.currentThread().getContextClassLoader()` 를 조합하여 스프링의 `ConfigurableObjectInputStream` 를 사용해야 할 것이다.
- 불행하게도, 여러 제3자 라이브러리들은 context classloader를 고려하지 않고 역 직렬화를 한다. 어떤 문제를 발견한다면, 라이브러리 작성자에게 수정을 요청해야 한다.

## 20.3 LiveReload
- `spring-boot-devtools` 모듈은 리소스가 변경될 때 브라우저 새로고침을 trigger하는 내장 LiveReload 서버를 포함한다. 
- LiveReload 브라우저 확장기능은 크롬, 파이어폭스, 사파리에서 자유롭게 사용 가능하다. [livereload.com](https://livereload.com/extensions/)
- 어플리케이션 실행 시 LiveReload 서버 사용을 원하지 않는다면, `spring.devtools.livereload.enabled` 속성을 `false` 로 설정하면 된다.

> 

## 20.4 전역 설정
- `$HOME` 폴더에 `.spring-boot-devtools.properties` 이름의 파일을 추가하여 전역 devtools 설정을 구성할 수 있다.(파일 이름은 "."으로 시작)
- 이 파일에 추가된 모든 속성들은 시스템에서 devtools를 사용하는 모든 스프링 부트 어플리케이션에 적용된다. 예를 들면, 항상 trigger file을 사용하여 재시작 구성을 하려면 아래와 같이 속성을 추가하면 된다.
#### ~/.spring-boot-devtools.properties. 
```
spring.devtools.reload.trigger-file=.reloadtrigger
```

## 20.5 Remote Applications
- 스프링 부트 개발자 도구는 local 환경에만 적용되는 것은 아니다.
- 어플리케이션을 원격으로 실행할 때 여러 기능을 사용할 수 있다.
- 원격 기능은 opt-in 이다. 이 기능을 활성화 하려면 아래와 같이  `devtools` 가 패키징 된 아카이브에 포함되었는지 확인해야 한다.
```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
			<configuration>
				<excludeDevtools>false</excludeDevtools>
			</configuration>
		</plugin>
	</plugins>
</build>
```
- 아래 예제와 같이 `spring.devtools.remote.secret` 속성을 성정해야 한다.
```
spring.devtools.remote.secret=mysecret
```
> 원격 어플리케이션에서 `spring-boot-devtools` 를 활성화 하는 것은 보안적으로 위험할 수 있다. 운영 환경에선 절대 `spring-boot-devtools` 를 활성화 하면 안된다.

### 20.5.1 원격 클라이언트 어플리케이션 실행하기
- 원격 클라이언트 어플리케이션은 IDE와 함께 실행되도록 디자인 되어있다.
- 접속하려고 하는 원격 프로젝트와 동일한 classpath에서 `org.springframework.boot.devtools.RemoteSpringApplication` 를 실행해야 한다.
- 어플리케이션의 필수적인 단독 argument는 접속할 원격 URL이다.
- 예를 들어, Cloud Foundry에 `my-app` 이름의 프로젝트 배포하였고 Eclipse 또는 STS를 사용한다면 아래와 같이 할 것이다.
    + `Run` 메뉴에서 `Run Configurations…​` 를 선택
    + "launch configuration"에서 `Java Application` 만들기
    + `my-app` 프로젝트 열기
    + 메인 클래스에서 `org.springframework.boot.devtools.RemoteSpringApplication` 사용
    + `Program arguments`에 `https://myapp.cfapps.io` (또는 원격 URL) 추가  

- 실행 중인 원격 클라이언트는 아래 리스트와 유사할 수 있다.
```
  .   ____          _                                              __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _          ___               _      \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` |        | _ \___ _ __  ___| |_ ___ \ \ \ \
 \\/  ___)| |_)| | | | | || (_| []::::::[]   / -_) '  \/ _ \  _/ -_) ) ) ) )
  '  |____| .__|_| |_|_| |_\__, |        |_|_\___|_|_|_\___/\__\___|/ / / /
 =========|_|==============|___/===================================/_/_/_/
 :: Spring Boot Remote :: 2.0.3.RELEASE

2015-06-10 18:25:06.632  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Starting RemoteSpringApplication on pwmbp with PID 14938 (/Users/pwebb/projects/spring-boot/code/spring-boot-devtools/target/classes started by pwebb in /Users/pwebb/projects/spring-boot/code/spring-boot-samples/spring-boot-sample-devtools)
2015-06-10 18:25:06.671  INFO 14938 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@2a17b7b6: startup date [Wed Jun 10 18:25:06 PDT 2015]; root of context hierarchy
2015-06-10 18:25:07.043  WARN 14938 --- [           main] o.s.b.d.r.c.RemoteClientConfiguration    : The connection to http://localhost:8080 is insecure. You should use a URL starting with 'https://'.
2015-06-10 18:25:07.074  INFO 14938 --- [           main] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2015-06-10 18:25:07.130  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Started RemoteSpringApplication in 0.74 seconds (JVM running for 1.105)
```

> 원격 클라이언트가 실제 어플리케이션과 동일한 classpath를 사용하기 때문에 어플리케이션의 속성들을 직접 읽을 수 있다. 이것이 인증을 위해 `spring.devtools.remote.secret` 속성을 읽고 서버에 전달하는 방법이다.

> traffic이 암호화되고 암호가 유출되지 않도록 `https://` 를 접속 프로토콜로 사용하도록 권장한다.

> 원격 어플리케이션에 접근하기 위해 프록시를 사용해야 한다면, `spring.devtools.remote.proxy.host` 과 `spring.devtools.remote.proxy.port` 속성을 구성하면 된다.

### 20.5.2 원격 업데이트
- 원격 클라이언트는 로컬 재시작과 동일한 방식으로 어플리케이션의 classpath 변경사항을 모니터링 한다.
- 업데이트된 리소스가 원격 어플리케이션에 푸시되고(필요 시) 재시작과 연결된다.
- 로컬에 없는 클라우드 서비스를 사용하는 기능을 반복하여 사용 시 유용할 수 있다.
- 일반적으로 원격 업데이트 및 재시작은 전체 빌드와 배포의 주기보다 훨씬 빠르다.

> 파일은 원격 클라이언트가 실행 중일 때만 모니터링 된다. 원격 클라이언트를 시작하기 전에 파일을 변경하면 원격 서버로 푸시되지 않는다.

## 21. 운영 어플리케이션 패키징
- 실행가능한 jar 파일은 운영 배포에 사용될 수 있다.
- 필요한 것들을 자체적으로 모두 가지고 있기 때문에, 클라우드 기반 배포에 잘 맞는다.
- “production ready” (health, auditing, metric REST, JMX end-points) 기능을 위해 `spring-boot-actuator` 추가를 고려해라. 자세한 내용은 [Part V, “Spring Boot Actuator: Production-ready features”](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready.html) 을 참조


## 참고자료
[![스프링 부트 2.0 Day 3. 스프링 부트 스타터](http://img.youtube.com/vi/w9wqpnLHnkY/0.jpg)](https://www.youtube.com/watch?v=w9wqpnLHnkY) 스프링 부트 2.0 Day 3. 스프링 부트 스타터

[![스프링 부트 2.0 Day 4. @SpringBootApplication과 XML 빈 설정 파일 사용하기](http://img.youtube.com/vi/jftcS1BQ_0g/0.jpg)](https://www.youtube.com/watch?v=jftcS1BQ_0g) 스프링 부트 2.0 Day 4. @SpringBootApplication과 XML 빈 설정 파일 사용하기

[![스프링 부트 2.0 Day 5. spring-boot-devtools 그리고 릴로딩](http://img.youtube.com/vi/5BhWpx7RW-w/0.jpg)](https://www.youtube.com/watch?v=5BhWpx7RW-w) 스프링 부트 2.0 Day 5. spring-boot-devtools 그리고 릴로딩