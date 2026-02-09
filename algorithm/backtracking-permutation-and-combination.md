# Backtracking - permuation & combination

## permutation

```python
num_list = [1, 2, 3]
visited = [False for _ in range(len(num_list))]
perm = []
r = 2

def permutation(depth):
    if depth == r:
        print(perm)
        return
    for i in range(len(num_list)):
        if visited[i] == False:
            visited[i] = True
            perm.append(num_list[i])
            permuatation(depth + 1)
            perm.pop()
            visited[i] = False

    permutation(0)
```

visited -> 방문 여부 표시 -> 순서

1. 종료 조건 : depth == 찾으려는 개수
2. 탐색 범위 : 매번 처음부터 끝까지
3. 재귀호출 (`permutation(depth + 1)`)

**과정**
`num_list = [1,2,3]` 에서 2개뽑는 경우
1. 선택 : `1` 선택, `visited[0] = True` -> `perm = [1]`
2. 재귀 : 다음 칸 채우기 위해 `permutation(1)` 호출
3. 아직 안 쓴 `2` 선택 -> `perm = [1, 2]`
4. 종료 조건: `depth == r`이 되어,`[1, 2]` 출력
5. 복구(Backtrack) : `2`를 `pop`하고, `visited[1] = False`로 되돌린다.
6. 반복: 다시 다른 숫자인 `3` 넣는다.


## combination

```python
num_list = [1,2,3,4,5]
comb = []
r = 3

def combination(start, depth):
    if depth == r:
        print(comb)
        return
    for i in range(start, len(num_list)):   # 앞쪽 안 봄
        comb.append(num_list[i])
        combination(i+1, depth+1)
        comb.pop()
    
combination(0, 0)
```

순서 x -> `visited` 필요 없음

1. 종료 조건: `depth == 찾으려는 개수`
2. 탐색 범위: 현재 지점부터 끝까지 (앞쪽은 안 봄)
3. 재귀 호출 : `combination(i+1, depth+1)`

**과정**
1. 시작 : `combination(0, 0)`
2. 선택1 : `i=0`인 `1` 선택 -> `comb = [1]`
3. 재귀1 : 다음 수 선택하기 위해 `combination(1, 1)` 호출
   - 선택2 : `i=1`인 `2` 선택 -> `comb = [1]`
   - 종료 조건: `depth == 2`가 되어 `[1, 2]`출력 후 리턴
   - 복구 : `2`를 `pop` -> `comb = [1]`
   - 선택3 : `i=2`인 `3` 선택 -> `comb = [1, 3]`
   - 종료 조건: `depth == 2`가 되어 `[1, 3]`출력 후 리턴
   - 복구 : `3`를 `pop` -> `comb = [1]`
4. 복구 : `1`를 `pop` -> `comb = [ ]`
5. 선택4 : `i=1`인 `2` 선택 -> `comb = [2]`
6. 재귀2 : 다음 수 선택하기 위해 `combination(2, 1)` 호출
   - 선택5: `i=2`인 `3` 선택 -> `comb = [2, 3]`
   - 종료 조건: `depth == 2`가 되어 `[2, 3]`출력 후 리턴
   - 복구 : `3`를 `pop` -> `comb = [2]`
7. 복구: 루프의 `2`를 `pop` -> `comb = []`
8. 선택 6 : `i=2`인 `3` 선택 -> `comb = [3]`
9. 재귀3 : `combination(3, 1)`을 호출하지만, `for`문의 범위가 끝나서 아무 일 없이 리턴
10. 종료