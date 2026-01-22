# [python] Arbitrary Argument Lists

### 1. Arbitrary Arguments Lists (`*args` - 위치 인자)

인자의 개수가 가변적일 때 사용한다.
위치(positional) 인자들을 **튜플(tuple)** 형태로 묶어서 함수 내로 전달한다.

매개변수 앞에 별표(*)를 붙인다. 관례적으로 `*args`를 사용하지만, `*numbers`처럼 다른 이름을 사용해도 된다.

```python
def my_function(arg1, *args):
    print(f"First argument: {arg1}")
    print(f"Arbitrary positional arguments (as a tuple) : {args}")

my_function('hello', 1, 2, 3, 4)

# First argument: hello
#Arbitrary positional arguments (as a tuple): (1, 2, 3, 4)
```

<br>

### 2. Arbitrary Keyword Arguments Lists (`**kwargs` - 키워드 인자)

이름을 가진 인자들(키워드 형태, `key = value`)을 유연하게 받을 때 사용한다.
인자들을 **딕셔너리(dictionary)** 형태로 묶어 함수 내로 전달한다.

매개변수 앞에 별표 두 개(`**`)를 붙인다. 관례적으로 `**kwargs`를 사용.

```python
def print_info(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_info(name="Alice", age=25, city="Seoul")

# name : Alice
# age : 25
# city : Seoul
```
