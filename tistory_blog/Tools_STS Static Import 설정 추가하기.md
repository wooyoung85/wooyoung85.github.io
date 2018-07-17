STS나 이클립스에서 테스트 코드를 작성할 때 Static Import가 잘 안될 경우가 있다.  
아무리 쎄게 `Ctrl` + `Shift` + `O` 를 눌러도 Import가 안된다.

테스트 코드 작성할 때마다 매번 import를 추가하기도 번거로우니 이클립스 설정을 추가하여 해결했다.  

### 예제코드
- `assertThat` 
  - junit에서 제공하는 함수
  - `assertThat(T actual, Matcher<? super T> matcher)`의 형태로 메서드를 사용하여 두 값을 비교할 수 있다.
- `is`
  - Hamcrest에서 제공하는 Matcher
  - Decorates another Matcher(다른 매처를 꾸며줘서 좀 더 읽기 쉬운 코드를 만들어 줌)
  - `is(T value)` 의 형태로 사용하면 `is(equalTo(value)))` 와 같다.

```java
import static org.hamcrest.CoreMatchers.is;
import static org.junit.Assert.assertThat;

...

@Test(expected=EmptyResultDataAccessException.class)
public void getUserFailure() throws SQLException, ClassNotFoundException{
    dao.deleteAll();
    assertThat(dao.getCount(), is(0));
    dao.get("unknown_id");
}
```

### 해결 방법

`Window > Preferences > Java > Editor > Content Assist > Favorites`
