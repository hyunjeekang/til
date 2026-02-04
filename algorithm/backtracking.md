# Backtracking


### 코드

`nums`에서 2개 조합 구하는 코드

```python
def backtrack(start, path):
  if len(path) == 2:
    print(path)
    return

  for i in range(start, len(nums)):
    path.append(nums[i])
    backtrack(i+1, path)
    path.pop()

nums = [1,2,3]
backtrack(0,[])
```

### 구조

1. **종료 조건** : 원하는 개수를 다 뽑았는가?
2. **선택** : 현재 숫자를 결과 리스트에 넣는다.
3. **재귀 탐색** : 다음 숫자를 뽑으러 깊게 들어간다.
4. **복구** 넣었던 숫자를 다시 빼서 상태를 복구한다.


### 시뮬레이션


| 단계 | 상태 |  | 설명 |
| --- | --- | --- | --- |
| **Step 1** | `path = []` | `backtrack(0, [])` 호출 | 시작점 |
| **Step 2** | `path = [1]` | `backtrack(1, [1])` 호출 | 1을 선택하고 다음 칸으로 이동 |
| **Step 3** | `path = [1, 2]` | **종료 조건 만족** | `[1, 2]` 출력 후 리턴 |
| **Step 4** | `path = [1]` | `path.pop()` | **백트래킹:n** 2를 빼고 다음 후보(3)를 확인 |
| **Step 5** | `path = [1, 3]` | **종료 조건 만족** | `[1, 3]` 출력 후 리턴 |
| **Step 6** | `path = []` | `path.pop()` | 1까지 다 훑었으니 1을 빼고 첫 번째 칸에 2를 넣으러 감 |
