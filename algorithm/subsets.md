# Subsets

### subset (Basic DFS)
모든 원소에 대해 포함한다/포함하지 않는다 두 갈래 선택
-> Leaf Node까지 탐색

```python
def get_subsets(nums):
    result = []
    subset = []
    
    def backtrack(index):
        # [기저 조건] 모든 원소를 확인했을 때
        if index == len(nums):
            result.append(subset[:]) # 현재 부분집합 복사하여 추가
            return
        
        # 1. nums[index]를 포함하는 경우
        subset.append(nums[index])
        backtrack(index + 1)
        
        # 2. nums[index]를 포함하지 않는 경우
        subset.pop()
        backtrack(index + 1)
        
    backtrack(0)
    return result


nums = [1, 2, 3]
print(get_subsets(nums))
# 출력: [[1, 2, 3], [1, 2], [1, 3], [1], [2, 3], [2], [3], []]
```


### Subset Sum Problem (Optimization with Backtracking)

📌 **정수 집합(Negative integers included)** <br>
- 음수가 포함되어 있으면 현재 값이 목표값(K)보다 크더라도, 나중에 음수를 더해 다시 합이 줄어들 수 있음
- 결국 **모든 부분집합을 구해서** 합을 확인해야 한다.

📌 **정수 집합(Only Positive included)** <br>
- 숫자를 더할 수록 합은 무조건 커진다.
- 탐색 중간에 현재 합이 이미 목표값(K)을 초과했다면, 그 뒤의 원소는 더 볼 필요 없이 즉시 중단한다. (**Pruning / Backtracking**)

```python
def backtrack(index, current_sum):
    # [가지치기] 이미 목표 합을 넘었다면 즉시 종료
    if current_sum > target:
        return
    
    # [기저 조건] 목표 합에 도달한 경우
    if current_sum == target:
        # 결과 처리
        return

    # [기저 조건] 모든 원소를 확인한 경우
    if index == len(nums):
        return

    # 1. 포함하는 경우
    subset.append(nums[index])
    backtrack(index + 1, current_sum + nums[index])
    
    # 2. 포함하지 않는 경우 (Backtrack)
    subset.pop()
    backtrack(index + 1, current_sum)
```