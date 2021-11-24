# **2021-11-23**
## **1. `namedtuple`**
### **1-1. 기본 설명**
`namedtuple` 인스턴스는 일반 튜플과 마찬가지로 메모리 효율적이다. 인스턴스마다 딕셔너리를 포함하고 있지 않기 때문이다. 각 `namedtuple`은 `namedtuple()` 팩토리 함수를 사용해 생성되는 자신의 클래스로 표현할 수 있다. 새 클래스의 이름과 각 요소에 대한 문자열이 인자로 올 수 있다.
```python
import collections

Person = collections.namedtuple('Person', 'name age')

bob = Person(name='Bob', age=30)
print('\nRepresentation:', bob)

jane = Person(name='Jane', age=29)
print('\nField by name:', jane.name)

print('\nFields by index:')
for p in [bob, jane]:
    print('{} is {} years old'.format(*p))
```

여기서 `Person`을 같게 하는데 헷갈리지 않기 위해 그렇게 한다고 한다

> __출력__
```
Representation: Person(name='Bob', age=30)

Field by name: Jane

Fields by index:
Bob is 30 years old
Jane is 29 years old
```

### **1-2. 상속에서의 활용**
```python
from collections import namedtuple


class Point(namedtuple('Point', 'row col')):
    pass


c = Point('test1', 'test2')
print(c.row)
```

> **출력**
```
test1
```

클래스에 상속될 경우 그 클래스의 ```namedtuple```로 활용되는 듯 하다.
## **2. `@property`**
### **2-1. 기본 설명**
일반적으로 인스턴스나 클래스의 속성에 접근하면 저장된 값이 반환된다. 프로퍼티`property`는 접근되는 순간 값이 계산되는 특수한 속성이다.

장식자 `@property`는 바로 다음에 나오는 메서드를 보통 메서드를 호출할 떄 붙여야 하는 괄호 `()` 없이 단순 속성인 것처럼 접근할 수 있게 한다.
```python
import math

class Circle(object):
    def __init__(self, radius):
        self.radius = radius
        # 원의 몇가지 프로퍼티들
    @property
    def area(self):
        return math.pi*self.radius**2
    @property
    def perimeter(self):
        return 2*math.pi*self.radius


c = Circle(4.0)
print("c.radius : ", c.radius)
print("c.area : ", c.area)
print("c.perimeter : ", c.perimeter)
```
> **출력**
```
c.radius :  4.0
c.area :  50.26548245743669
c.perimeter :  25.132741228718345
```

`c.area = 2` 와 같이 속성을 재정의하려고 시도할 경우

```AttributeError: can't set attribute```
에러가 발생한다

### **2-2. 속성 설정/삭제**
프로퍼티는 속성을 설정하거나 삭제하는 연산을 가로챌 수도 있다. 이를 위해서는 프로퍼티에 설정 `setter` 메서드나 삭제 `delete` 메서드를 추가하면 된다.
```python
class Foo(object):
    def __init__(self, name):
        self.__name = name

    @property
    def name(self):
        return self.__name

    @name.setter
    def name(self, value):
        if not isinstance(value, str):
            raise TypeError("Must be a string!")
        self.__name = value

    @name.deleter
    def name(self):
        raise TypeError("Can't delete name")


f = Foo("Guido")
n = f.name       # f.name()를 호출한다. - 얻기(get) 함수
f.name = "Monty" # 설정 함수인 name(f, "Monty") 을 호출한다.
f.name = 45      # 설정 함수인 name(f, 45)을 호출한다 -> Type Error
del f.name       # 삭제 함수인 name(f)을 호출한다 -> Type Error
```
속성 `name`은 `@property` 장식자와 관련 메서드를 사용해서 먼저 읽기 전용 프로퍼티로 정의된다. 이어서 나오는 `@name.setter`와 `@name.deleter` 장식자는 설정 및 삭제 연산을 위한 추가 메서드를 프로퍼티에 연결시킨다.

추가 메서드는 원본 프로퍼티 메서드의 이름과 동일해야 한다. 앞의 예에서 `name`의 실제 값은 `__name` 속성에 저장된다. 이 속성의 이름을 정하는 데에는 특별한 규칙은 없지만 프로퍼티와 구별하기 위해 프로퍼티의 이름과는 달라야 한다.
## **3. 정적 메서드와 클래스 메서드**
클래스 정의에서 모든 함수는 인스턴스에 대해서 작동하는 것으로 가정한다. 이 때문에 첫 번째 매개변수로서 항상 `self`가 전달된다. 이 밖에도 클래스 정의에 볼 수 있는 흔히 사용되는 두 종류의 메서드가 더 있다.
### **3-1. 정적 메서드 `static method`**
정적 메서드 `static method` 는 클래스에 의해 정의되는 네임스페이스에 들어 있는 보통의 함수다 정적 메서드는 인스턴스에 대해서 작동하지 않는다. 정적 메서드를 정의하려면 다음과 같이 `@staticmethod` 장식자를 사용하면 된다.
```python
class Foo(object):
    @staticmethod
    def add(x, y):
        return x + y
```
정적 메서드를 호출하려면 그저 클래스 이름을 앞에 붙이기만 하면 된다. 추가 정보를 넘겨줄 필요는 없다. 다음 예를 보자
```python
x = Foo.add(3, 4) # x = 7
```
정적 메서드는 다양한 방식으로 인스턴스를 생성하는 클래스를 작성할 떄 흔히 사용된다. 클래스에서 `__init__()` 함수는 단 하나만 존재할 수 있기 때문에 다음에서 볼 수 있듯이 다른 생성 함수를 정적 메서드로서 정의하는 경우가 종종 있다
```python
import time


class Date(object):
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day

    @staticmethod
    def now():
        t = time.localtime()
        return Date(t.tm_year, t.tm_mon, t.tm_mday)

    @staticmethod
    def tomorrow():
        t = time.localtime(time.time() + 86400)
        return Date(t.tm_year, t.tm_mon, t.tm_mday)


# 날짜 객체를 몇 개 생성하는 예
a = Date(1967, 4, 9)
b = Date.now()  # 정적 메서드 now()를 호출한다.
c = Date.tomorrow()  # 정적 메서드 tomorrow()를 호출한다.

```

### **3-2. 클래스 메서드 `class method`**
클래스 메서드 `class method`는 클래스 자체를 객체로 보고 클래스 객체에 대해 작동하는 메서드를 말한다. 클래스 메서드는 `@classmethod` 장식자를 사용해서 정의한다. 인스턴스 메서드와 다른 점은 첫 번째 인수로서 관례적으로 `cls`라는 이름을 가진 클래스가 전달된다는 점이다. 다음은 한 예이다
```python
class Times(object):
    factor = 1
    @classmethod
    def mul(cls, x):
        return cls.factor*x
    
class TwoTimes(Times):
    factor = 2
    
x = TwoTimes.mul(4)  # Times.mul(TwoTimes, 4)를 호출한다. 결과는 8이다.
```
앞의 예에서 `mul()`에 클래스 `TwoTimes`가 객체로 전달되었다. 앞의 예가 약간 난해하게 보일 수도 있을 것 같다. 클래스 메서드는 약간 복잡해 보이지만 실용적인 용도로 쓰인다. 예를 들어, 앞에서 정의한 `Date` 클래스로부터 상속을 받아서 다음과 같이 약간 커스터마이징 하였다고 하자.
```python
class EuroDate(Date):
    # 유럽의 날짜 표현 방식대로 문자열 변환을 수행해주도록 수정
    def __str__(self):
        return "%02d/%02d/%4d" % (self.day, self.month, self.year)
```
이 클래스는 `Date`에서 상속받았기 때문에 `Date`의 모든 기능을 갖게 된다. 그러나 `now()` 와 `tomorrow()` 메서드는 의도한 대로 작동하지 않는다. 예를 들어, 누군가가 `EuroDate.now()`를 호출하면 `EuroDate` 객체가 아닌 `Date` 객체가 반환된다. 다음과 같이 클래스 메서드를 사용함으로써 이 문제를 해결할 수 있다.
```python
import time
class Date(object):
    ...
    @classmethod
    def now(cls):
        t = time.localtime()
        # 적절한 타입의 객체를 생성한다.
        return cls(t.tm_year, t.tm_mon, t.tm_mday)
    
class EuroDate(Date):
    ...

a = Date.now()     # Date.now(Date)가 호출되어서 Date 객체가 반환된다
b = EuroDate.now() # Date.now(EuroDate)가 호출되어서 EuroDate 객체가 반환된다.
```
정적 메서드와 클래스 메서드를 사용할 때 파이썬에서는 이 메서드들을 인스턴스 메서드와 별개의 네임스페이스로 관리하지 않는다. 이 떄문에 다음과 같이 정적 메서드와 클래스 메서드를 인스턴스에 대해서 호출할 수 있다.
```python
d = Date(1967, 4, 9)
b = d.now() # Date.now(Date)를 호출한다
```

이것이 가능하다는 점은 `d.now()`가 인스턴스 `d`와는 아무런 관련이 없다는 점 떄문에 혼동을 가져올 수 있다. 이 부분은 파이썬의 객체 시스템이 스몰톡이나 루비같은 다른 객체지향 언어와 다른 점 중 하나이다. 다른 객체지향언어에서는 클래스 메서드와 인스턴스 메서드가 엄격하게 분리되어 있다.
# **2021-11-24**
## **1. 이동, 비트 연산자**
| 연산     | 설명                     |
| -------- | ------------------------ |
| `x << y` | 왼쪽 이동                |
| `x >> y` | 오른쪽 이동              |
| `x & y`  | 비트 `and`               |
| `x \| y` | 비트 `or`                |
| `x ^ y`  | 비트 `xor(exclusive or)` |
| `~x`     | 비트 `negation`          |
## **2. `assert` 와 `__debug__`**
`assert` 문으로 프로그램에 디버깅 코드를 넣을 수 있다. `assert`문의 일반적인 형식은 다음과 같다
```python
assert test [, msg]
```
여기서 `test`는 `True`와 `False`로 평가되는 표현식이다. 만약 `test`가 `False`로 평가되면 `assert`문에 지정한 메시지인 `msg`와 함께 `AssertionError`에러가 발생한다. 다음은 한 예이다
```python
def write_date(file, data):
    assert file, "write_data: file not defined!"
    ...
```
프로그램이 올바르게 작동하기 위해서 반드시 수행되어야 하는 코드에는 `assert` 문을 사용하지 않도록 한다. 왜냐하면 파이썬이 최적화 모드(인터프리터에 `-o` 옵션)로 실행될 때 `assert` 문이 실행되지 않기 때문이다. 특히 사용자의 입력을 확인하는 데 `assert` 문을 사용하면 안 된다. `assert`문은 항상 참이어야 하는 것들을 검사하는 데 사용한다. 참이 아닌 것이 있으면 사용자의 오류가 아니라 프로그램의 버그 때문인 것이다.

예를 들어, 앞의 예에서 `wrtie_data()` 가 최종 사용자에 의해서 사용되는 함수라면 `assert` 문은 `if`문과 에러 처리 코드로 대체되어야 한다.

`assert` 와 더불어 파이썬에서는 읽기 전용 내장 변수인 `__debug__` 가 제공된다. 이 변수는 인터프리터가 최적 모드(인터프리터에 `-O` 옵션)로 수행되지 않는 한 `True`로 설정된다. 필요에 따라 프로그램에서 이 변수를 확인할 수 있고, 이 변수가 설정된 경우 추가적인 에러 검사 과정을 수행할 수도 있다. 인터프리터에서 `__debug__` 변수가 구현되는 방식은 최적화되어 있어서 추가적인 `if` 문의 제어 흐름 로직은 코드에 실제로 포함되지 않는다. 파이썬이 일반 모드로 실행되는 경우 `if __debug__`문 아래에 있는 문장들은 `if`문 없이 프로그램 코드에 직접 나타난다. 최적화 모드에서는 `if __debug__` 문과 관련된 모든 문장들이 프로그램에서 완전히 제거된다.

`assert` 문과 `__debug__` 를 사용하면 프로그램을 두 가지 모드를 통해 효율적으로 개발할 수 있다. 예를 들어, 디버그 모드에서는 코드가 올바르게 작동하는지를 검증하기 위한 단언 및 버그 검사 코드를 자유롭게 사용할 수 있다. 최적화 모드에서는 이러한 모든 추가적인 검사들이 자동으로 제거되기 때문에 성능에 영향이 없다.
## **2. 객체 비교와 순서 매기기**
### **2-1 객체 검사 및 해싱을 위한 특수 메서드**
| 메서드           | 설명                                         |
| ---------------- | -------------------------------------------- |
| `__bool__(self)` | 진리값 검사를 위해 `False` 나 `True` 를 반환 |
| `__hash__(self)` | 정수 해시 색인을 계산                        |

`__bool__()` 메서드는 진리값 검사를 수행하는 데 사용되며 `True` 또는 `False`를 반환해야 한다. 이 메서드가 정의되지 않을 경우에는 진리값을 결정하기 위해서 `__len()__` 메서드가 대신 사용된다.

```python
class Truth:
    def __init__(self, num):
        self.num = num

    def __bool__(self):
        return self.num != 0


print(bool(Truth(0)))  # False
print(bool(Truth(3)))  # True
```

`__hash__()` 메서드는 사전에서 키로서 쓰일 수 있는 객체에서 정의해야 하는 메서드이다. 반환되는 값은 동일한 두 객체에 대해서 동일한 값을 갖는 정수여야 한다. 또한, 변경 가능한 객체에서는 이 메서드를 정의하면 안 된다. 객체에 변경이 가해지면 해시 값이 변경되어 이후의 사전 검색에서 객체를 찾을 수 없기 때문이다.
```python
class Obj:
    def __init__(self, a, b):
        self.a = a
        self.b = b

    def _key(self):
        return (self.a, self.b)

    def __eq__(self, other):
        return self._key() == other._key()

    def __hash__(self):
        return hash(self._key())


o1 = Obj(1, 2)
o2 = Obj(3, 4)
print(hash(o1))  # -3550055125485641917
print(hash(o2))  # 1079245023883434373

d = {o1: 1, o2: 2}
```

### **2-2. 비교를 위한 메서드**
| 메서드                | 결과              |
| --------------------- | ----------------- |
| `__it__(self, other)` | `self` < `other`  |
| `__le__(self, other)` | `self` <= `other` |
| `__gt__(self, other)` | `self` > `other`  |
| `__ge__(self, other)` | `self` >= `other` |
| `__eq__(self, other)` | `self` == `other` |
| `__ne__(self, other)` | `self` != `other` |

객체는 하나 이상의 관계 연산자 `<`, `>`, `<=`, `>=`, `==`, `!=` 를 구현할수 있다. 각 메서드는 두 개의 인수를 받으며, 불리언 값, 리스트, 또는 다른 아무 파이썬 타입 등 어떤 종류의 객체이든 반환할 수 있다. 예를 들어, 수치 관련 패키지의 경우 두 행렬의 원소별 비교를 수행할 때 그 결과로서 행렬을 반환할 수 있다. 비교가 수행될 수 없는 경우에는 예외가 발생되기도 한다.

```python
class Compare:
    def __init__(self, n):
        self.n = n

    def __eq__(self, other):
        return self.n == other

    def __lt__(self, other):
        return self.n < other

    def __le__(self, other):
        return self.n <= other


c = Compare(10)
print(c < 10)   # False / __lt__()
print(c <= 10)  # True  / __le__()
print(c > 10)   # Error : __gt__() is not defined
print(c == 10)  # True  / __eq__()
print(c != 10)  # False / __ne__() or not __eq__()
```
## **3. `isinstance(object, classobj)`**
`object` 가 `classobj` 또는 `classobj` 의 하위 클래스의 인스턴스이거나 추상 기반 클래스인 `classobj` 에 속할 경우 `True` 를 반환한다. `classobj` 매개변수는 타입이거나 클래스의 튜플일 수도 있다. 예를 들어, `isinstance(s, (list, tuple))` `s` 가 튜플이거나 리스트이면 `True` 를 반환한다.
```python
class A(object): pass
class B(A): pass
class C(object): pass

a = A()
b = B()
c = C()

print(isinstance(a, A))  # True
print(isinstance(b, A))  # True  / B는 A로부터 파생
print(isinstance(b, C))  # False / B는 C로부터 파생되지 않았다.
```
## **4. 사전의 메서드와 연산**
|항목|설명|
|-|-|
|`len(m)`|`m`에 있는 항목 개수를 반환한다|
|`m[k]`|키 `k`로 `m`의 항목을 반환한다.|
|`m[k] = x`|`m[k]`를 `x`로 설정한다.|
|`del m[k]`|`m`에서 `m[k]`를 제거한다|
|`k in m`|키 `k`가 `m`에 있으면 `True`를 반환한다.|
|`m.clear()`|`m`에서 모든 항목을 제거한다|
|`m.copy()`|`m`의 복사본을 생성한다|
|`m.fromkeys(s, [,value])`|순서열 `s`에서 키를 가져오고 모든 값을 `value`로 설정한 새로운 사전을 생성한다|
|`m.get(k [,v])`|`m[k]`가 있으면 `m[k]`를 반환하고 아니면 `v`를 반환한다|
|`m.items()`|`m`의 모든 (키, 값) 쌍들로 구성되는 순서열을 반환한다.|
|`m.keys()`|`m`의 모든 키들의 순서열을 반환한다|
|`m.pop(k [,default])`|`m[k]`가 있으면 `m[k]`를 반환하고 `m`에서 제거한다. `m[k]`가 없을 경우 `default`가 제공되면 `default`를 반환하고, 아니면 `KeyError` 예외를 발생시킨다|
|`m.popitem()`|`m`에서 임의의 (키, 값) 쌍을 제거하고 이를 튜플로서 반환한다.|
|`m.setdefault(k [,v])`|`m[k]`가 있으면 `m[k]`를 반환한다. 없으면, `v`를 반환하고 `m[k]`를 `v`로 설정한다.|
|`m.update(b)`|`b`에 있는 모든 객체를 `m`에 추가한다.|
|`m.values()`|`m`에 있는 모든 값으로 구성되는 순서열을 반환한다.|

## **5. 패키지**
모듈을 공통의 이름으로 묶는데 패키지를 사용한다. 패키지를 사용하면 모듈 사이에 이름 충돌 문제를 해결할 수 있다. 패키지를 만들려면 패키지 이름을 가진 디렉터리를 만들고 이 디렉터리에 `__init__.py` 파일을 생성하면 된다. 그런 다음 필요에 따라 소스 파일, 컴파일된 확장 기능, 하위 패키지 등을 추가하면 된다. 다음은 패키지를 구성하는 예를 보여준다.

```
Graphics/
    __init__.py
    Primitive/
        __init__.py
        lines.py
        fill.py
        text.py
        ...
    Graph2d/
        __init__.py
        plot2d.py
    Graph3d/
        __init__.py
        plot3d.py
    Formats/
        __init__.py
        gif.py
        png.py
        tiff.py
        jpeg.py
```
`import` 문으로 패키지로부터 모듈을 몇 가지 형태로 로드할 수 있다.
* `import Graphics.Primite.fill`
  * 하위 모듈인 `Graphics.Primitive.fill`을 로드한다. 이 모듈의 내용에 접근하려면 `Graphics.Primitive.fill.floodfill(img, x, y, color)` 같이 이름을 직접 써주어야 한다.
* `from Graphics.Primitive import fill`
  * 하위 모듈인 `fill`을 로드하며 앞쪽 패키지 이름 없이 사용할 수 있다. 예를 들면 `fill.floodfill(img, x, y, color)`
* `from Graphics.Primitive.fill import floodfill`
  * 하위 모듈 `fill`을 로드하며 `floodfill` 함수를 바로 사용할 수 있다. 예를 들어, `floodfill(img, x, y, color)`

패키지의 일부가 처음으로 로드될 때 `__init__.py` 파일에 있는 코드가 실행된다. 이 파일은 비어있을 수 있고 패키지를 초기화하는 코드를 담을 수도 있다. `import`를 실행하는 동안 만나게 되는 모든 `__init__.py` 파일이 차례대로 실행된다. 즉, `import Graphics.Primitive.fill`이 실행되면 먼저 `Graphics` 디렉터리에 있는 `__init__.py` 파일이 실행되고 다음으로 `Primitive` 디렉터리에 있는 `__init__.py` 파일이 실행 된다.

패키지와 관련해서 다음 문장을 처리할 떄 약간의 문제가 있을 수 있다.
```python
from Graphics.Primitive import *
```
보통 프로그래머는 패키지와 연관된 모든 하위 모듈을 현제 네임스페이스로 임포트하려고 이 문장을 사용한다. 그러나 시스템마다 파일 이름과 관련된 규칙이 다르기 때문에(특히 대 소문자 구별 여부) 어떤 모듈을 임포트해야 할지를 정확하게 결정하게 어렵다. 그 결과, 이 문장은 단순히 `Primitive` 디렉터리에 있는 `__init__.py` 파일 안에서 정의된 이름을 임포트한다. 이러한 작동 방식을 바꾸려면 패키지와 연관된 모듈 이름을 담는 리스트인 `__all__`을 정의하면 된다. 다음과 같이 패키지의 `__init__.py` 파일에 이 리스트를 정의하면 된다.
```python
# Graphics/Primitive/__init__.py

__all__ = ["lines", "text", "fill"]
```
이제 사용자가 
```python
from Graphics.Primitive import *
```
문을 실행하면 나열된 모든 하위 모듈이 기대했던 대로 로드된다.

한 하위 모듈에서 동일한 패키지의 다른 하위 모듈을 임포트하려는 경우에도 문제가 발생할 수 있다. 예를 들어 `Graphics.Primitive.fill` 모듈에서 `Graphics.Primitive.lines` 모듈을 임포트하려고 한다고 하자. 이렇게 하려면 단순하게 완전히 한정된 이름(예를 들어, `from Graphics.Primitive import lines`) 을 사용하거나 아니면 다음과 같이 패키지에 상대적인 임포트를 사용해야 한다.
```python
# fill.py

from . import lines
```
점 `.` 은 현재 모듈과 동일한 디렉터리를 가리킨다. 따라서, 이 문장은 파일 `fill.py` 와 동일한 디렉터리에 있는 모듈 `lines`를 찾는다. 패키지의 하위 모듈을 임포트하려고 `import module` 같은 문장을 사용하지 않도록 한다. 파이썬 3에서는 `import`에 절대 경로가 사용된 것으로 가정하고 `module`을 표준 라이브러리에서 로드하려고 시도한다. 상대적인 임포트를 사용하면 더 명확하게 의도를 표현할 수 있다. 예를 들어 `Graphics.Graph2D.plot2d` 에서 `Graphics.Primitives.lines` 모듈을 임포트하려면 다음과 같이 하면 된다.

```python
# plot2d.py

from ..Primitives import lines
```
여기서 `..`은 디렉터리 밖으로 한 수준 벗어나게 하고 `Primitives`는 다른 패키지 디렉터리로 들어가게 한다.
`import`문 중 `from module import symbol` 형태로만 상대적인 임포트를 지정할 수 있다. 따라서 `import ..Primitive.lines` 나 `import .lines`는 문법 오류이다. 또한 `symbol`은 유효한 식별자여야 한다. 따라서, `from .. import Primitives.lines`도 적법하지 않다. 마지막으로, 상대적인 임포트는 패키지 안에서만 사용해야 한다. 단순히 다른 디렉터리에 있는 모듈을 가리키는 데 상대적인 임포트를 사용하면 안 된다.

패키지 이름을 임포트한다고 해서 그 안에 있는 모든 하위 모듈이 임포트되는 것은 아니다. 예를 들어, 다음 코드는 제대로 작동하지 않는다. 
```python
import Graphics

Graphics.Primitive.fill.floodfill(img, x, y, color)  # 실패
```
이 문제를 해결하려면 `import Graphics`가 `Graphics` 디렉터리에 있는 `__init__.py` 파일을 실행하기 떄문에 다음과 같이 상대적인 임포트를 사용하여 모든 하위 모듈을 자동으로 로드하게 만들어야 한다.
```python
# Graphics.__init__.py

from . import Primitive, Graph2d, Graph3d
```
```python
# Graphics/Primitive/__init__.py

from . import lines, fill, text, ...
```
이제 `import Graphics` 문은 모든 하위 모듈을 임포트하고 완전히 한정된 이름을 사용해서 해당 모듈에 접근할 수 있게 되었다. 상대적인 임포트는 앞에서 설명한대로 사용해야 한다. 간단히 `import module` 같은 문장을 사용하면 표준라이브러리 모듈이 로드된다.

# 참고 책
*데이비드 M. 비즐리, 『파이썬 완벽 가이드』, 송인철, 송현제 옮김, 인사이트(2012)*

*더그 헬먼, 『예제로 배우는 파이썬 표준 라이브러리』, 권석기, 김우현 옮김, 에이콘(2020)*

*이강성, 『파이썬3 바이블』, 프리렉(2013)*