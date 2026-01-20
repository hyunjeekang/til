# [python] `for-else`

for 루프 안에서

1. `break` 만난 경우 : `else` 실행 **x**
2. `break` 만나지 않은 경우 : `else` 실행 **o**

```python
for n in data:
    if n == target:
        break
else:
    print("못 찾을 경우 print된다")
```
