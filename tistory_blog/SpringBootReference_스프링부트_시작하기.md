> 스프링 부트 레퍼런스 문서를 보면서 번역 및 정리를 한 문서 입니다.  
잘못 해석한 부분이 있을 수 있으니 참고하시기 바랍니다.


## 8. 스프링 부트 소개
- 독립적으로 실행되고, 운영(Production) 수준의 스프링 어플리케이션을 쉽게 구현 가능하도록 도와줌
- Spring Platform과 제3자 라이브러리에 대한 <u>**독단적인**</u> 의견이 반영되어 있기 때문에 최소한의 혼란을 겪으면서 시작할 수 있다.
- jar파일 war 파일 배포 모두 가능하다.
- 독단적인 기본 설정에서 벗어나는 설정도 가능
- `embedded servers`, `security`, `metrics`, `health checks`, `externalized configuration` 과 같은 비기능적인 요소도 제공한다.
- code generation은 없고 XML 설정은 필요 없다.

## 9. 시스템 요구사항
- Java 8이상 Spring 프레임워크는 5.0.6 이상
- Maven 3.2 이상, Gradle 4 이상

## 9.1 Sevlet Containers
스프링 부트는 다음과 같은 내장형 서블릿 컨테이너를 지원함
| Name | Servlet Version |
|:--------:|:--------:|
| Tomcat 8.5 | 3.1 |
| Jetty 9.4 | 3.1 |
| Undertow 1.4 | 3.1 |

## 10. 스프링 부트 설치
- 스프링 부트는 "classic" 자바 개발 도구 또는 설치된 command line tool과 함께 사용할 수 있다.
- Java SDK Version은 1.8 이상이어야 함 (Java Version Check는 `$java -version`)
- Spring Boot CLI를 사용하는 방법도 있다.

## 10.1 Java 개발자를 위한 설치 지침
- 다른 표준 자바 라이브러리와 같은 방식으로 스프링 부트를 사용하기 위해서는 `spring-boot-*.jar` 파일들이 classpath에 적절하게 포함되어 있어야 한다. (스프링 부트 라이브러리가 제대로 추가되면 설치가 된 것임)
- 스프링 부트는 특별한 통합 tool들이 필요 없고, 어떤 IDE나 Text Editor를 사용해도 됨
- 스프링 부트 어플리케이션은 특별할 것 없다. 그래서 다른 Java Program을 다루듯이 실행하고 디버깅하면 된다.

### 10.1.1 메이븐 설치
- 다양한 Package Manager를 통해 설치가 가능
- 스프링 부트의 의존성들은 `org.springframework.boot` groupId를 사용한다.
- 일반적인 메이븐 POM 파일은 `spring-boot-starter-parent`프로젝트를 상속하고 하나 이상의 "Starters"와의 의존관계를 정의하고 있다.
- 스프링 부트는 실행가능한 jars를 만들기 위해 Maven Plugin(옵션)을 제공한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>myproject</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<!-- 기본 상속 -->
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.0.BUILD-SNAPSHOT</version>
	</parent>

	<!-- 웹 Application을 위한 일반적인 종속성 추가 -->
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
	</dependencies>

	<!-- Package as an executable jar -->
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

	<!-- Spring repositories 추가 -->
	<!-- 릴리즈 버전일 경우 추가할 필요 없음 -->
	<repositories>
		<repository>
			<id>spring-snapshots</id>
			<url>https://repo.spring.io/snapshot</url>
			<snapshots><enabled>true</enabled></snapshots>
		</repository>
		<repository>
			<id>spring-milestones</id>
			<url>https://repo.spring.io/milestone</url>
		</repository>
	</repositories>
	<pluginRepositories>
		<pluginRepository>
			<id>spring-snapshots</id>
			<url>https://repo.spring.io/snapshot</url>
		</pluginRepository>
		<pluginRepository>
			<id>spring-milestones</id>
			<url>https://repo.spring.io/milestone</url>
		</pluginRepository>
	</pluginRepositories>
</project>
```

## 11. Spring Boot Application 게시
- [spring.io](https://spring.io/) 사이트에 스프링 부트 사용을 위한 많은 시작 가이드가 있다. 
- 가장 간단한 방법은 [start.spring.io](https://start.spring.io/) 로 가서 "Web" Dependency를 선택한 후 Project를 생성하면 된다.

- Java와 Maven 버전 확인
	```
	$ java -version
	java version "1.8.0_102"
	Java(TM) SE Runtime Environment (build 1.8.0_102-b14)
	Java HotSpot(TM) 64-Bit Server VM (build 25.102-b14, mixed mode)
	```
	```
	$ mvn -v
	Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-10T16:41:47+00:00)
	Maven home: /usr/local/Cellar/maven/3.3.9/libexec
	Java version: 1.8.0_102, vendor: Oracle Corporation
	```

## 11.1 POM 만들기
- Maven `pom.xml`(프로젝트 빌드 시 사용되는 recipe) 만들기

## 11.2 종속성 추가	
- 스프링 부트는 jar파일들을 classpath에 추가할 수 있게 해주는 여러개의 "Starters"를 제공한다.
- 예제 어플리케이션에서 POM의 parent 부분에 `spring-boot-starter-parent`(유용한 Maven 기본값을 제공함)를 사용하였다.
- `spring-boot-starter-parent`는 `dependency-management` 부분을 제공한다. 그래서 사용자는 parent에서 관리하고 있는 dependency의 경우 `version` tag를 생략할 수 있다.
- `mvn dependency:tree` 명령어로 프로젝트 종속성을 트리 구조로 볼 수 있다.

## 11.3 코드 입력
```java
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.web.bind.annotation.*;

@RestController
@EnableAutoConfiguration
public class Example {

	@RequestMapping("/")
	String home() {
		return "Hello World!";
	}

	public static void main(String[] args) throws Exception {
		SpringApplication.run(Example.class, args);
	}

}
```
### 11.3.1 @RestController와 @RequestMapping 어노테이션
- @RestController
  + Spring Framework Stereotype Annotations
  +	Stereotype Annotation은 사람이 코드를 읽을 때와 스프링에게 이 클래스가 특정 역할을 맡고 있다는 것에 대해 힌트를 제공한다.	
  + 이 어노테이션이 붙은 클래스는 스프링이 웹 요청을 처리할 때 고려하게 된다.

- @RequestMapping
  + Spring MVC and REST Annotations
  + 이 어노테이션은 "routing" 정보를 제공한다.

### 11.3.2 @EnableAutoConfiguration 어노테이션
- Spring Boot Annotations
- 이 어노테이션은 스프링 부트에게 POM에 추가한 dependencies에 따라 스프링을 구성하는 방법을 추측하도록 알려준다.
- spring-boot-starter-web(Tomcat, Spring MVC) 때문에 auto-configuration은 사용자가 웹 어플리케이션 개발할 것으로 추측하고 그에 따라 스프링을 설정하게 된다.

> Starters and Auto-configuration
>  + Auto-configuration은 Starter와 잘 작동하도록 설계되었지만 두 개념이 직접적으로 연관은 없다.
>  + Starter 외에도 jar 종속성을 자유롭게 취사 선택할 수 있다.

### 11.3.3 "main" Method
  - 응용 프로그램 진입점
  - 메인 메서드는 `SpringApplication.run` 메서드를 Call하면서 SpringApplication 클래스에게 위임하게 된다.
  - SpringApplication은 응용 프로그램을 로드하고, 스프링을 시작하면서 자동 설정된 Tomcat web server를 시작한다.
  - `SpringApplication.run` 메서드를 실행할 때 반드시 `Example.class`(메인 메서드를 감싸고 있는 Class)를 인자값으로 넘겨줘야 한다.

## 11.4 예제 실행
```
$ mvn spring-boot:run

.   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
'  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::  (v2.1.0.BUILD-SNAPSHOT)
....... . . .
....... . . . (log output here)
....... . . .
........ Started Example in 2.222 seconds (JVM running for 6.514)
```
## 11.5 실행가능한 Jar파일 만들기
- 운영환경에서 실행 가능하도록 완벽하게 자체적으로 필요한 것(컴파일된 클래스, 모든 종속성)들을 담고 있어서 "fat jars"라고 부르기도 한다.
- 좀 더 깊은 내용은 [Appendix E. The Executable Jar Format](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#executable-jar)을 참고하면 된다.
- 실행 가능한 Jar를 생성하려면 `spring-boot-maven-plugin`을 pom.xml에 추가해야 함.
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
- pom.xml을 저장한 후 명령실행 창에서 `mvn package`을 실행
```
$ mvn package

[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building myproject 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] .... ..
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ myproject ---
[INFO] Building jar: /Users/developer/example/spring-boot-example/target/myproject-0.0.1-SNAPSHOT.jar
[INFO]
[INFO] --- spring-boot-maven-plugin:2.1.0.BUILD-SNAPSHOT:repackage (default) @ myproject ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```
- Target 폴더에 가면 `myproject-0.0.1-SNAPSHOT.jar` 파일을 볼 수 있다.(파일크기는 10MB 정도)
- Fat Jar 파일 안에 파일 목록이 궁금하면 `jar tvf` 명령어로 볼 수 있다.
```
$ jar tvf target/myproject-0.0.1-SNAPSHOT.jar
```
- Target 폴더에서 `myproject-0.0.1-SNAPSHOT.jar.original`과 이름이 유사한 파일을 볼 수 있는데 이는 스프링 부트가 repackaging 하기 전에 메이븐이 생성한 파일이다.
- 어플리케이션을 실행시키려면 아래와 같이 `java -jar`명령어로 실행하면 된다.
```
$ java -jar target/myproject-0.0.1-SNAPSHOT.jar

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v2.1.0.BUILD-SNAPSHOT)
....... . . .
....... . . . (log output here)
....... . . .
........ Started Example in 2.536 seconds (JVM running for 2.864)
```


## 참고자료
[Spring Boot Reference Guide](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/)

[A Guide to Spring Framework Annotations](https://dzone.com/articles/a-guide-to-spring-framework-annotations)

[![스프링 부트 2.0 Day 1. 스프링 부트 시작하기](http://img.youtube.com/vi/CnmTCMRTbxo/0.jpg)](https://www.youtube.com/watch?v=CnmTCMRTbxo) 스프링 부트 2.0 Day 1. 스프링 부트 시작하기

[![스프링 부트 2.0 Day 2. Executable JAR 어떻게 만들고 어떻게 동작하는가](http://img.youtube.com/vi/PicKx3lDGLk/0.jpg)](https://www.youtube.com/watch?v=PicKx3lDGLk) 스프링 부트 2.0 Day 2. Executable JAR 어떻게 만들고 어떻게 동작하는가