## Geb 관련 영상
[![[SpringCamp2013] Spring MVC TEST 어렵지 않아요!](http://img.youtube.com/vi/k_88ADbuJqQ/0.jpg)](https://www.youtube.com/watch?v=k_88ADbuJqQ)

## PCSELL 프로젝트 기반 Geb Project
#### [PCSELL](https://github.com/wooyoung85/pcsell) 사이트를 먼저 실행 후 [Geb 테스트 프로젝트](https://github.com/wooyoung85/pcsell-integration-test)를 실행

#### git bash를 사용하여 실행
<pre>$ ./gradlew test</pre>


- GebConfig.groovy (테스트 설정 파일)
    - selenium Web 드라이버 사용
    - 테스트 할 사이트 URL 설정
```java
import org.openqa.selenium.chrome.ChromeDriver
import org.openqa.selenium.chrome.ChromeOptions
import org.openqa.selenium.firefox.FirefoxDriver

waiting {
    timeout = 2
}

environments {
    // run via “./gradlew chromeTest”
    // See: http://code.google.com/p/selenium/wiki/ChromeDriver
    chrome {
        driver = { new ChromeDriver() }
    }

    // run via “./gradlew chromeHeadlessTest”
    // See: http://code.google.com/p/selenium/wiki/ChromeDriver
    chromeHeadless {
        driver = {
            ChromeOptions o = new ChromeOptions()
            o.addArguments('headless')
            new ChromeDriver(o)
        }
    }

    // run via “./gradlew firefoxTest”
    // See: http://code.google.com/p/selenium/wiki/FirefoxDriver
    firefox {
        atCheckWaiting = 1
        driver = { new FirefoxDriver() }
    }
}

// To run the tests with all browsers just run “./gradlew test”
baseUrl = "http://localhost:8080"
```

- GebishOrgSpec.groovy (테스트 시나리오 정의 파일)
    - <code>When:</code> 테스트 케이스
    - <code>Then:</code> 테스트 기대 결과
```java
import geb.spock.GebSpec

class GebishOrgSpec extends GebSpec {

    def "can get to the current Book of Geb"() {
        when:
        to PcSellHomePage

        then:
        manualsMenu.jumbo.text().startsWith("임직원")

        when:
        manualsMenu.menus[1].click()

        then:
        at PCSellLoginPage
    }

    def "go login page"() {
        when:
        to PcSellHomePage

        then:
        manualsMenu.jumbo.text().startsWith("임직원")

        when:
        manualsMenu.menus[2].click()

        then:
        at PCSellLoginPage
    }

    def "Login Test"() {
        when:
        to LoginTest

        then:
        loginBtn.text().startsWith("Login")

        when:
        loginAsTester()

        then:
        at PCSellLoginPage
    }
}
```

## 참고사이트
- [Very Groovy browser automation. Web Testing, Screen Scraping and more](http://www.gebish.org/)<br/>
- [통합테스트 유틸리티 Geb](http://shy-blg.tistory.com/entry/%ED%86%B5%ED%95%A9%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%9C%A0%ED%8B%B8%EB%A6%AC%ED%8B%B0-Geb)
