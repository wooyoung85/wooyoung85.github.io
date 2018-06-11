## JUnit
- 테스트 방법에 @ 테스트 애노테이션 추가
- 클래스의 메소드가 결과를 내 보냅니다.

```java
import static org.junit.Assert.assertEquals;

import org.junit.Test;

public class CalculatorTest {
    @Test
    public void add() {
        Calculator cal = new Calculator();
        assertEquals (9, cal.add (6,3);
    }
}
```

## web application test
- Spring Mock MVC
- web integration test (Geb)

## 참고 사이트
- [Spring Boot Test](http://meetup.toast.com/posts/124)
- [시작하자 단위 테스트](https://www.slideshare.net/youngeunchoi12/springcamp2014-35347399)