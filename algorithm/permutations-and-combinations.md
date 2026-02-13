# Permutations and Combinations

| 유형 | Key Idea | Implementation | Analogy |
| --- | --- | --- | --- |
| **순열** (Permutation) | 순서가 다르면 다른 경우, 중복 허용 X | `visited` 배열을 사용하여 이미 선택한 원소 제외 | 3명의 후보 중 1, 2, 3위 뽑기 |
| **중복순열** (Product) | 순서가 다르면 다른 경우, 중복 허용 O | `visited` 체크 없이 모든 원소를 매번 탐색 | 3자리의 디지털 비밀번호 설정 |
| **조합** (Combination) | 순서가 달라도 구성이 같으면 동일, 중복 X | `start` 인덱스를 넘겨주어 현재 이후의 원소만 탐색 | 로또 번호 6개 뽑기 |
| **중복조합** (Comb. with Rep.) | 순서가 달라도 구성이 같으면 동일, 중복 O | `start` 인덱스를 넘기되, 재귀 호출 시 현재 `i`를 포함 | 같은 과일을 여러 개 담는 바구니 |

---

### python code

`depth`(현재까지 뽑은 개수)와 `R`(목표 개수)을 기반으로 동작

```python
# 1. 순열 (Permutation)
def DFS(depth):
    if depth == R:
        print(result)
        return
    for i in range(N):
        if not visited[i]:
            visited[i] = True
            result[depth] = array[i]
            DFS(depth + 1)
            visited[i] = False # 원복 (Backtracking)

# 2. 중복순열 (Product)
def DFS(depth):
    if depth == R:
        print(result)
        return
    for i in range(N):
        result[depth] = array[i]
        DFS(depth + 1) # visited 체크 없음

# 3. 조합 (Combination)
def DFS(depth, start):
    if depth == R:
        print(result)
        return
    for i in range(start, N):
        result[depth] = array[i]
        DFS(depth + 1, i + 1) # 'i + 1'을 넘겨 중복 방지

# 4. 중복조합 (Combination with replacement)
def DFS(depth, start):
    if depth == R:
        print(result)
        return
    for i in range(start, N):
        result[depth] = array[i]
        DFS(depth + 1, i) # 'i'를 넘겨 현재 원소 다시 선택 가능

```