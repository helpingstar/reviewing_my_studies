# **2021-11-23**
## **1. namedtuple**
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
## **2. @property**
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
n = f.name  # f.name()를 호출한다. - 얻기(get) 함수
f.name = "Monty"  # 설정 함수인 name(f, "Monty") 을 호출한다.
f.name = 45  # 설정 함수인 name(f, 45)을 호출한다 -> Type Error
del f.name  # 삭제 함수인 name(f)을 호출한다 -> Type Error
```
속성 `name`은 `@property` 장식자와 관련 메서드를 사용해서 먼저 읽기 전용 프로퍼티로 정의된다. 이어서 나오는 `@name.setter`와 `@name.deleter` 장식자는 설정 및 삭제 연산을 위한 추가 메서드를 프로퍼티에 연결시킨다.

추가 메서드는 원본 프로퍼티 메서드의 이름과 동일해야 한다. 앞의 예에서 `name`의 실제 값은 `__name` 속성에 저장된다. 이 속성의 이름을 정하는 데에는 특별한 규칙은 없지만 프로퍼티와 구별하기 위해 프로퍼티의 이름과는 달라야 한다.
