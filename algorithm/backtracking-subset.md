# Backtracking - subset

### subset
```python
def get_subsets(nums):
    result = []
    subset = []
    
    def backtrack(index):
        # 기저 조건: 모든 원소를 확인했을 때
        if index == len(nums):
            result.append(subset[:]) # 현재 부분집합 복사하여 추가
            return
        
        # 1. nums[index]를 포함하는 경우
        subset.append(nums[index])
        backtrack(index + 1)
        
        # 2. nums[index]를 포함하지 않는 경우 (백트래킹)
        subset.pop()
        backtrack(index + 1)
        
    backtrack(0)
    return result

# 예시
nums = [1, 2, 3]
print(get_subsets(nums))
# 출력: [[1, 2, 3], [1, 2], [1, 3], [1], [2, 3], [2], [3], []]
```


### 부분 집합의 합 문제

**정수 집합의 경우**
음수의 경우 합이 줄어들 수 있으므로 목표합과 관계 없이 모든 부분집합의 합 구해야 함

**자연수 집합 경우**
목표 합에 도달하면 그 이후의 요소를 추가할 필요가 없음
backtracking 사용 가능 