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
일반적으로 인스턴스나 클래스의 속성에 접근하면 저장된 값이 반환된다. 프로퍼티(`property`)는 접근되는 순간 값이 계산되는 특수한 속성이다.

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
        t = time.localtime(time.time()+86400)
        return Date(t.tm_year, t.tm_mon, t.tm_mday)

# 날짜 객체를 몇 개 생성하는 예
a = Date(1967, 4, 9)
b = Date.now()      # 정적 메서드 now()를 호출한다.
c = Date.tomorrow() # 정적 메서드 tomorrow()를 호출한다.
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