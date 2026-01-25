# [python] `zip()`

```python
N, M = map(int, input().split())
grid1 = [list(map(int, input().split())) for _ in range(N)]
grid2 = [list(map(int, input().split())) for _ in range(N)]

# 풀이1 : 2중 for문
def solve1():
    for r in range(N):
        for c in range(M):
            grid1[r][c] += grid2[r][c]
    for row in grid1: print(*row)

# 풀이 2 : zip
def solve2():
    result = [[c+d for c, d in zip(a, b)] for a, b in zip(grid1, grid2)]
    for row in result: print(*row)

# # 1. 두 개의 3x3 행렬 정의
# grid1 = [
#     [1, 1, 1],
#     [2, 2, 2],
#     [0, 1, 0]
# ]

# grid2 = [
#     [3, 3, 3],
#     [4, 4, 4],
#     [5, 5, 100]
# ]

# # 결과를 담을 빈 리스트
# final_result = []

# print('[ grid 1 ]')
# for row in grid1: print(row)
# print()
# print('[ grid 2 ]')
# for row in grid2: print(row)
# print()

# print("덧셈 시작!\n")

# # zip(grid1, grid2): 행 단위로 짝 짓기
# # enumerate는 몇 번째 행인지 표시하기 위해 추가
# for row_idx, (row_a, row_b) in enumerate(zip(grid1, grid2)):
#     print(f"[{row_idx+1}번째 줄]")
#     print(f"  - grid1의 행: {row_a}")
#     print(f"  - grid2의 행: {row_b}")
#     print(f"  -> 이 두 행을 zip으로 묶어 숫자끼리 더함")
    
#     current_row_result = [] # 계산된 한 줄을 담을 임시 리스트
    
#     # zip(row_a, row_b): 각 행 안의 숫자(Element) 단위로 짝 짓기
#     for val_a, val_b in zip(row_a, row_b):
#         sum_val = val_a + val_b
#         print(f"    * 계산: {val_a} + {val_b} = {sum_val}")
#         current_row_result.append(sum_val)
    
#     print(f"  => {row_idx+1}번째 줄 결과: {current_row_result}\n")
#     final_result.append(current_row_result)

# print("=== 최종 결과 행렬 ===")
# for row in final_result:
#     print(row)
```