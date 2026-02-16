# Bitmasking

**Definition**: 0(False)과 1(True)의 상태를 가지는 여러 값들을 **하나의 정수**로 관리하는 기법
**Advantages**: 메모리 사용량 최소화, 빠른 연산 속도, 간결한 집합 표현 가능

<br/>

### 1. Fundamental Knowledge

**Integer Bit Sizes & Ranges**
|  | Type | Bits | Range |
| --- | --- | --- | --- | 
| **C / C++** | `int` | 32-bit | 약 21억 |
|  | `long long` | 64-bit |  원소 60개 이상 관리 시 필수 |
| **Java** | `int` | 32-bit | 고정 크기 |
|  | `long` | 64-bit | 고정 크기 |
| **Python** | `int` | **가변 비트** | 메모리가 허용하는 한 무제한 **Overflow 걱정 없음** |

<br/>

**Bitwise Operators**

<img src="https://www.educative.io/api/edpresso/shot/6125638579781632/image/5467266204958720?page_type=collection_lesson" width = "60%">

| Operation | Symbol | Purpose |
| --- | --- | --- |
| **AND** | `&` | 특정 비트의 **포함 여부 확인** |
| **OR** | `\|` | 특정 비트 **추가(1로 설정)** |
| **XOR** | `^` | 특정 비트 **반전(0↔1)** |
| **NOT** | `~` | 모든 비트 반전 |
| **Shift** | `<<`, `>>` | 비트를 왼쪽/오른쪽으로 이동 |

    연산자 우선순위 유의하며 사용!

<br/>

### 2. Key Considerations

**Shift Operations & Overflow (C / Java)**

* `1 << 35` (X) → `int` 범위를 넘어가서 쓰레기 값이 나옴
* `1LL << 35` (O) → C++에서 `long long`으로 계산하도록 명시해야 함

<br/>

**MSB (Most Significant Bit)**:
* 가장 왼쪽 비트는 보통 **부호(Sign)**를 결정한다. (0: 양수, 1: 음수)
* `~0`을 수행하면 모든 비트가 반전되어 음수가 되는 이유가 바로 이 부호 비트 때문!

<br/>

**Arbitrary-precesion integers (Python)**:
* 파이썬은 `Arbitrary-precision integers`를 지원하여 비트가 아무리 길어져도(예: 100번째 비트 켜기) 자동으로 메모리를 할당해 처리한다. (연산 속도는 다른 언어에 비해 느릴 수 있음)

<br/>

**Signed vs Unsigned (C/C++)**:
* 비트마스킹을 할 때는 부호 비트(MSB)의 간섭을 피하기 위해 가급적 `unsigned` 타입을 사용하는 것이 좋다.
* `unsigned int` (32-bit), `unsigned long long` (64-bit)

<br/>


### 3. Practical Examples

**Subset Enumeration**
`N`개의 원소를 가진 집합의 모든 부분집합(`2**N`개) 순회

```python
# [0, 1, 2, 3]의 부분집합 구하기 (2**4)
def subset():
    for tmp in range(16):
        print("{", end=' ')
        for i in range(4):
            if tmp & (1 << i):
                print(i, end=' ')
        print("}", end = ' ')

def subset():
    n = 4
    for i in range(1 << n):
        subset = [j for j in range(n) if i & ( 1 << j)]
        print(subset)
```

<br/>

**예제: [BOJ 11723](https://www.acmicpc.net/problem/11723) 집합**

여러 개의 상태 값 관리

```python
if command == 'all':
    states = (1 << 21) - 1
elif command == 'empty':
    states = 0
else:
    x = int(line[1])
    if command == 'add':
        states |= (1 << x)
    elif command == 'remove':
        states &= ~(1 << x)
    elif command == 'check':
        print(1 if states & (1 << x) else 0)
    elif command == 'toggle':
        states ^= (1 << x)
```

    x번째 비트 켜기 : `state |= (1 << x)`
    x번째 비트 끄기 : `state &= ~(1 << x)`
    x번째 비트 확인 : `if state & (1 << x)`
    x번째 비트 반전 : `state ^= (1 << x)`
    전체 집합/공집합 : `(1 << n) - 1 / 0`

---

### 4. reference sites
[What is bit masking? - Educative](https://www.educative.io/answers/what-is-bit-masking)

---
