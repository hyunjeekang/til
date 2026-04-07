# Dynamic Programming

<img src="https://cdn.emre.me/2019-09-07-fibonacci-number.png"> <br><br>


`fibo(0)`, `fibo(1)` 등 중복 호출 많음!  <br>
-> **메모이제이션** : 이전 계산 값을 저장해서 실행 속도 최적화  <br>

**DP : 최적화 문제 해결 알고리즘** <br>
- DP 적용 

    - 중복 부분문제 구조 : 작은 문제 해결 -> 큰 문제 해결 

    - 최적 부분문제 구조 : 어떤 문제 해가 최적일 때 작은 문제들의 해 역시 최적이어야 한다

- 접근 방식

    - 최적해 구조 특성 파악
    - 최적해 값 재귀적 정의
    - 최적해 값 계산 

- 재귀의 시간 복잡도를 줄일 수 있다

<br>

**하향식: 재귀 + 메모이제이션(Memoization)**

```java
    static long[] memo = new long[100];

    private static long fibonacci(int n){
        // 기저 조건: 가장 작은 문제
        if(n == 1 || n == 2) return 1;
        
        // 이미 계산된 경우 -> 바로 값 리턴 
        if(memo[n] != 0) return memo[n];

        // 계산 안 된 경우 -> 계산 후 배열에 저장
        memo[n] = fibonacci(n-1) + fibonacci(n-2);
        return memo[n];
    }
```
- 하향식 재귀가 깊어지면 메모리 초과날 수 있음


<br>

**상향식: 반복 + 타뷸레이션(Tabulation)**

```java
    private static long fibonacci(int n) {

        // dp table
        long[] dp = new long[n+1];

        // 기저 조건
        dp[1] = 1;
        if(n >= 2){
            dp[2] = 1;
        }

        // 반복문
        for(int i = 3; i <= n; i++){
            dp[i] = dp[i-1] + dp[i-2];
        }

        return dp[n];
        
    }
```

<br>
