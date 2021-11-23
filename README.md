# 2021-11-23
## 1. namedtuple
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

test
