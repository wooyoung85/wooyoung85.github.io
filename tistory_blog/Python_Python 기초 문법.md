> Python 기초 문법을 주제로 발표했던 자료 정리한 내용입니다.
---
# 파이썬
- 간단하고 쉬운 문법
- 인터프리터 언어
- 효율적인 자료 구조들과 객체 지향 프로그래밍 지원
- 동적 타이핑(typing) 
    * 실행 시점에 많은 것을 판단하고 결정
    * 규모가 커질수록 개발하기 힘듬 ㅠㅠ

> PYTHON 개발패턴 : WRITE(EDIT) - ~~COMPILE~~ - RUN

## 파이썬으로 무엇을 할 수 있을까?
> 시스템 관리, GUI, DB, 분산처리, 수치연산 등등 거의 모든 영역의 프로그래밍 가능

스터디에서는 아래 두가지 영역을 주로 학습할 예정 
1. 웹 프로그래밍 (Django Framework)
2. Data Science

## 어떤 파이썬 버전을 써야 할까?

- python2.7은 오랜시간 표준 (유명한 리눅스 배포판에는 2.x 버전이 기본적으로 설치되어 있을 정도)
- python2.7은 2020년까지만 필수 보안이슈들을 다루는 update를 계속 진행할 예정
- 파이썬을 처음 접한다면 고민없이 python3으로 ㄱㄱ

> Starting from <span style="color:green">Celery</span> 5.0 only Python 3.5+ will be supported.

> <span style="color:green">Faust</span> requires Python 3.6 or later for the new async/await syntax, and variable type annotations.

## Python Implementations
파이썬 코드를 동작시키는 인터프리터는 다양한 언어와 framework기반에서 구현되어 있음
- CPython (표준, C)
- [PyPy](https://pypy.org/) (Python???)
- Jython (자바)
- IronPython, PythonNET (닷넷)
- 기타 등등

> 기본적으론 CPython, 퍼포먼스가 필요할 때는 PyPy, 호환성이 필요할 때는 Jython, IronPython, PythonNET

## Python 설치  
- http://python.org/ (http://python.org/)
- Anaconda (https://www.continuum.io/downloads)   
    python 기본 패키지 외 데이터분석에 유용한 외부 패키지들을 같이 포함되어 있음

## 실습환경
- jupyter notebook 사용
---
# Jupyter
- jupyter는 ipython을 웹에서 사용할 수 있도록 개선한 인터프리터 환경
- python외에도 다양한 kernel을 함께 사용할 수 있는 웹 기반의 데이터 분석도구로 발전하고 있음
- 설치 : `pip install jupyter`
- 실행 : `jupyter notebook`
- 아나콘다를 설치했다면 jupyter가 포함되어 있기 때문에 그냥 사용하면 됨

## Jupyter 단축키

명령 | short cut
:---|:---:
편집모드 진입 | enter
편집모드 탈출 | esc
셀 실행 | ctrl + enter
셀 실행하고 다음 새로운 셀 생성 | shift + enter
도움말 | shift + tab
자동완성 | tab
셀 복사 | c
셀 붙여넣기 | v
셀 삭제 | dd

## Jupyter 이해하기
- document하나가 별도의 python process로 구동 (재기동 가능)
- document내에 cell들은 한 process에서 구동되기 때문에, 같은 상태, 메모리를 공유

> jupyter(ipython)에서 in/out과 stdout, stderr는 다르게 표시됨
---
# Coding Convention
- 상수는 대문자
- 클래스명은 CamelCase
- 변수와 함수는 _으로 구분된 단어의 조합
- `_` 한 글자는 dummy 변수를 표현할 때 사용 (다국어모듈에서도 사용)
- `_`로 시작하는 이름은 외부에 노출하고 싶지 않을때 사용 (클래스, 모듈의 private의미, 등)
- `__`로 시작하는 이름은 가급적 사용 X (python의 예약어들이 많아서 문제가 생길 수 있음)

## Indentation
#### <span style="color:red">굉장히 중요함!!!</span>

- 들여쓰기 자체가 문법
- 탭 혹은 스페이스를 이용하여 들여쓰기 할 수 있음
    ```python
    def hello():
        print('world')
        if True:
            print('hello')
            print('hello')
    ```

#### [예제] 아래 문장을 python에서는 어떻게 표현할까?

```java
if (condition) {}
```

**pass statement를 사용하면 됨** (문법규칙을 충족시키기 위해 아무것도 하지 않는 문장임을 표시할 때 사용)
```python
if condition:
    pass
```
---
# 기본문법
주석, 변수, 변수타입

## 주석
3가지 방식으로 표현 가능
```python
# this is a comment
'''this is a comment'''
"""이것도 주석"""
```

## 변수
- 변수는 선언시점에 반드시 어떤 값이든 할당해야 함
- 타입은 지정 X
- 값을 지정하는 시점에 타입이 동적으로 결정(변경)
    > python3.6에서는 static type checking을 도와주는 문법이 추가됨
    ```python
    a : str = 'hello'
    b : int = 3
    ```

- 변수를 재정의하면 타입이 바뀜
    ```python
    a = None  # 아무것도 아님을 표현할 때 사용, Java의 NULL과 같은 의미
    a = 1
    a = "str"
    ```
- 변수의 모든 참조가 삭제되면 python이 자동으로 메모리에서 해제함. (garbage collector)

## 변수타입
숫자, 문자열, List, Dictionary, Tuple, Set

### 숫자
int, float, bool

**python은 숫자뿐만 아니라 전부 객체로 인식** (심지어 함수도 객체🙉)
```python
# dir 함수 : 해당 객체가 어떤 변수와 메소드(method)를 가지고 있는지 나열
>>> dir(3)[:3]
['__abs__', '__add__', '__and__']
```

**bool타입은 숫자이면서 객체**
```python
>>> isinstance(True, int), isinstance(True, object)
(True, True)
```

### 문자열
포매팅, 인덱싱, 슬라이싱

#### 문자열 포매팅
```python
name = '홍길동'
age = 30
print("나의 이름은 %s입니다. 나이는 %d입니다." % (name, age))
print("나의 이름은 {0}입니다. 나이는 {1}입니다.".format(name, age))
print(f'나의 이름은 {name}입니다. 나이는 {age}입니다.')
```

#### 문자열 인덱싱
**"파이썬은 0부터 숫자를 센다."**
```python
>>> a = "Life is too short, You need Python"
>>> a[0]
'L'
>>> a[12]
's'
>>> a[-1]
'n'
```

#### 문자열 슬라이싱
```python
>>> a = "20010331Rainy"
>>> year = a[:4]
>>> day = a[4:8]
>>> weather = a[8:]
>>> year
'2001'
>>> day
'0331'
>>> weather
'Rainy'
```

#### [예제] "Pithon"이라는 문자열을 "Python"으로 바꾸려면?
```python
>>> a = "Pithon"
>>> a[1]
'i'
>>> a[1] = 'y'
```
a라는 변수에 "Pithon"이라는 문자열을 대입하고 a[1]의 값이 i니까 a[1]을 y로 바꾸어 주는 로직은....  
당연히 에러가 발생함!!!  
왜냐하면 **문자열의 요소값은 바꿀 수 있는 값이 아님**  
(문자열, 튜플 등 의 자료형은 그 요소값을 변경할 수 없기 때문에 **immutable한 자료형**이라고도 함) 

👉 위에서 살펴본 슬라이싱 기법을 이용하면 됨
```python
>>> a = "Pithon"
>>> a[:1]
'P'
>>> a[2:]
'thon'
>>> a[:1] + 'y' + a[2:]
'Python'
```

#### 형변환
```python
# 문자열 -> 숫자, 소수 -> 정수
>>> int("345"), int(3.45)
(345, 3)
```

```python
# 정수 -> 소수
>>> float(3)
3.0
```

```python
# 숫자 -> 문자열
>>> str(345)
'345'
```

```python
# python은 묵시적인 형변환을 지원하지 않기 때문에 연산시에는 주의가 필요합니다.
>>> try:
...     print(str(345) + "hello")
... except TypeError as e:
...     print(e)

345hello
```

### List
- 기존 프로그래밍 언어로 치자면 배열과 비슷
- 길이의 제한 X
- 무엇이든! 섞어서! 담기 가능 --> 실제로 이렇게 쓸 일은 거의 없음
- [자세한 내용은 여기를 참고](https://wikidocs.net/14)

    ```python
    >>> mylist = [1,2]
    >>> mylist.append(1)
    >>> mylist
    [1, 2, 1]
    ```

    ```python
    >>> mylist = list("helloworld")
    >>> print(mylist)
    >>> for char in mylist:
    ...     print(char, end="")
    ['h', 'e', 'l', 'l', 'o', 'w', 'o', 'r', 'l', 'd']
    helloworld
    ```

    ```python
    >>> a = [1,2,3]
    >>> b = a[:1] + a[1:]
    >>> a == b, id(a) == id(b)
    (True, False)
    ```

    ```python
    >>> id([]), id(["1","3"]), id([1,2,3])
    (4393359240, 4393359240, 4393359240)
    ```

### Dictionary
- key와 value를 한 쌍으로 갖는 자료형
- [자세한 내용은 여기를 참고](https://wikidocs.net/16)

    ```python
    >>> phonebook = {
    ...     "John" : 938477566,
    ...     "Jack" : 938377264,
    ...     "Jill s" : 947662781
    ... }
    >>> phonebook["John"] = 838477561
    >>> phonebook
    {'John': 838477561, 'Jack': 938377264, 'Jill s': 947662781}
    ```

    ```python
    >>> phonebook = dict(John=938477566, Jack=938377264, Jill=947662781)
    >>> print(len(phonebook))
    3

    >>> for entry in phonebook:
    ...     print(entry, phonebook[entry])
    John 938477566
    Jack 938377264
    Jill 947662781

    >>> for name, number in phonebook.items():
    ...     print("Phone number of %s is %d" % (name, number))    
    Phone number of John is 938477566
    Phone number of Jack is 938377264
    Phone number of Jill is 947662781
    ```
### Tuple
#### 튜플(tuple)은 리스트와 거의 비슷함

#### 리스트와 튜플 차이점
- 리스트는 대괄호`[]`로 둘러싸지만 튜플은 괄호`()` 사용
- 리스트는 그 값의 생성, 삭제, 수정이 가능하지만 튜플은 그 값을 바꿀 수 없음!!! (Immutable)

#### 튜플이 굳이 왜 필요할까??
- 리스트보다 빠름
- 이 데이터는 변하지 않을 거라는 믿음(자신감)
- 외부(DB, 네트웍)에서 온 조회성 데이터같은 것을 표현하기에 적합
- 함수의 argument는 tuple로 전달됨  
  (positional 형태로 전달되는 인자들은 args라는 tuple에 저장되며, keyword 형태로 전달되는 인자들은 kwargs라는 dict에 저장)
- 변하지 않기 때문에 hashable할 수 있고, dictionary의 key로도 쓰일 수 있음

    ```python
    >>> t1 = (1, 2, 'a', 'b')
    >>> del t1[0]
    ---------------------------------------------------------------------------
    TypeError                                 Traceback (most recent call last)
    <ipython-input-4-f66ca102cafc> in <module>
        1 t1 = (1, 2, 'a', 'b')
    ----> 2 del t1[0]

    TypeError: 'tuple' object doesn't support item deletion
    ```

### Set
- 중복을 허용하지 X
- 순서가 X(Unordered).

#### 리스트나 튜플은 순서가 있기 때문에 인덱싱을 가능하지만  Set 자료형은 순서가 없기(unordered) 때문에 인덱싱으로 값을 얻을 수 없음 (이런 점은 딕셔너리와 비슷)

> ※ 중복을 허용하지 않는 set의 특징은 자료형의 중복을 제거하기 위한 필터 역할로 종종 사용되기도 함

#### set 자료형이 정말 유용하게 사용되는 경우는 다음과 같이 교집합, 합집합, 차집합을 구할 때

```python
# 교집합, 합집합, 차집합 구하기
>>> s1 = set([1, 2, 3, 4, 5, 6])
>>> s2 = set([4, 5, 6, 7, 8, 9])

>>> print(s1 & s2)
>>> print(s1.intersection(s2))
>>> print(s1 | s2)
>>> print(s1.union(s2))
>>> print(s1 - s2)
>>> print(s2.difference(s1))
{4, 5, 6}
{4, 5, 6}
{1, 2, 3, 4, 5, 6, 7, 8, 9}
{1, 2, 3, 4, 5, 6, 7, 8, 9}
{1, 2, 3}
{8, 9, 7}
```

```python
# 1개 추가
>>> s1.add(7)
>>> print(s1)
{1, 2, 3, 4, 5, 6, 7}

# 여러 개 추가
>>> s1.update([8, 9])
>>> print(s1)
{1, 2, 3, 4, 5, 6, 7, 8, 9}

# 특정 값 제거
>>> s1.remove(9)
>>> print(s1)
{1, 2, 3, 4, 5, 6, 7, 8}
```
