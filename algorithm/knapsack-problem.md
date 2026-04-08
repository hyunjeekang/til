# Knapsack Problem


## 0/1 Knapsack


>`n`개 물건(무게 `w`, 가치 `v`, 각 물건은 **1개**), 배낭 용량 `W`<br>배낭에 담을 수 있는 물건의 **최대 가치** 

<br>

<img src = "https://media.geeksforgeeks.org/wp-content/uploads/20240805101207/Recursion-Tree-for-01-KnapSack.png" width="80%"><br>

<br>


### 완전 탐색 방법
- 물건들의 집합 `S`에 대한 모든 부분집합 구하기

- 총 무게가 `W` 초과하는 집합 버리고, 나머지 집합에서 총합이 가장 큰 집합 선택

- 물건의 개수가 증가하면 시간 복잡도가 지수적(`2^n`)으로 증가

<br>

### DP
- [DP 정리 참고](./dynamic-programming.md)

- 처음 문제 : `W` 용량 배낭에 최대 가치 넣기

- `w` 무게 물건 넣었을 때의 문제 : `W - w`용량 배낭에 최대 가치 넣기 <br>

    -> 최적 부분 문제 구조 <br>
    -> 가방의 남은 무게 기준! <br>

<br>


### DP 예제 코드

**Top-Down**
```java
import java.io.*;
import java.util.*;

class Solution {
    
    // 메모이제이션을 위한 2차원 배열 선언
    static Integer[][] memo;

    public static int knapsack(List<Integer> weights, List<Integer> values, int maxWeight) {
        int n = weights.size();
        
        // [물건의 개수][배낭의 최대 무게 + 1]
        // 물건의 인덱스가 0부터 n-1까지이므로 크기는 n으로 설정
        memo = new Integer[n][maxWeight + 1];
        
        // 재귀 호출 시작
        // 마지막 물건(n-1)부터 0번 물건까지 살펴보고 
        // 남은 무게가 maxWeight일 때의 최대 가치를 구하기

        return dp(n - 1, maxWeight, weights, values);
    }

    // 재귀적으로 최대 가치를 계산하는 함수
    private static int dp(int i, int w, List<Integer> weights, List<Integer> values) {
        
        // [Step 1] 기저 조건
        // 인덱스가 0 미만으로 내려가서 더 이상 살펴볼 물건이 없거나 
        // 배낭의 남은 용량이 0이라면 가치는 0
        if (i < 0 || w == 0) {
            return 0;
        }

        // [Step 2] 메모이제이션 체크
        // 이미 계산해둔 값이 있다면 바로 반환
        if (memo[i][w] != null) {
            return memo[i][w];
        }

        // [Step 3] 점화식 계산 및 기록

        // 현재 물건의 무게가 배낭의 남은 용량보다 커서 넣을 수 없는 경우
        if (weights.get(i) > w) {
            // 이 물건은 건너뛰고 이전 물건(i-1)으로 넘어감
            memo[i][w] = dp(i - 1, w, weights, values);
        } 

        // 배낭에 넣을 수 있는 경우
        else {
            memo[i][w] = Math.max(
                // 1. 현재 물건을 넣지 않는 경우의 최대 가치
                dp(i - 1, w, weights, values), 
                // 2. 현재 물건을 넣는 경우: (남은 무게 - 현재 물건 무게) 상태의 최대 가치 + 현재 물건의 가치
                dp(i - 1, w - weights.get(i), weights, values) + values.get(i)
            );
        }

        // 계산된 최댓값을 반환
        return memo[i][w];
    }

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st;

        // 1. 무게 입력
        st = new StringTokenizer(br.readLine());
        List<Integer> weights = new ArrayList<>();
        while(st.hasMoreTokens()) {
            weights.add(Integer.parseInt(st.nextToken()));
        }

        // 2. 가치 입력
        st = new StringTokenizer(br.readLine());
        List<Integer> values = new ArrayList<>();
        while(st.hasMoreTokens()) {
            values.add(Integer.parseInt(st.nextToken()));
        }

        // 3. 배낭 용량 입력
        int maxWeight = Integer.parseInt(br.readLine());

        // 결과 출력
        int res = knapsack(weights, values, maxWeight);
        System.out.println(res);
    }
}
```
<br>
<br>

**Bottom-Up : 2D**
```java
import java.io.*;
import java.util.*;

class Solution {
    public static int knapsack(List<Integer> weights, List<Integer> values, int remainingWeight) {
        int n = weights.size();
        
        // maxValue[i][j]: 1번부터 i번째 물건까지 고려
        // 배낭의 최대 용량이 j일 때 얻을 수 있는 최대 가치
        int[][] maxValue = new int[n + 1][remainingWeight + 1];
        
        // 모든 물건을 차례대로 확인
        for (int i = 0; i <= n; i++) {
            // 0부터 최대 허용 무게까지 모든 경우의 수를 확인합니다.
            for (int w = 0; w <= remainingWeight; w++) {
                
                // 물건을 아예 고려하지 않거나 배낭 용량이 0이면 최대 가치는 0
                if (i == 0 || w == 0) {
                    maxValue[i][w] = 0;
                } 

                // 현재 물건의 무게가 배낭의 남은 용량보다 커서 넣을 수 없는 경우
                else if (w < weights.get(i - 1)) {
                    // 이전 물건(i-1)까지 고려했을 때의 최대 가치 그대로 가져오기 
                    maxValue[i][w] = maxValue[i - 1][w];
                } 

                // 배낭에 물건을 넣을 수 있는 경우:
                else {
                    maxValue[i][w] = Math.max(
                        // 1. 물건을 넣는 경우
                        // 현재 물건의 가치 + (현재 배낭 용량 - 현재 물건 무게)로 얻을 수 있었던 최대 가치
                        
                        values.get(i - 1) + maxValue[i - 1][w - weights.get(i - 1)],
                        // 2. 물건을 넣지 않는 경우
                        // 이전 물건(i-1)까지 고려했을 때의 최대 가치

                        maxValue[i - 1][w]
                    );
                }
            }
        }
        // 배낭 용량이 maxWeight일 때의 최대 가치
        return maxValue[n][remainingWeight];
    }

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st;

        // 1. 무게 입력
        st = new StringTokenizer(br.readLine());
        List<Integer> weights = new ArrayList<>();
        while(st.hasMoreTokens()) {
            weights.add(Integer.parseInt(st.nextToken()));
        }

        // 2. 가치 입력
        st = new StringTokenizer(br.readLine());
        List<Integer> values = new ArrayList<>();
        while(st.hasMoreTokens()) {
            values.add(Integer.parseInt(st.nextToken()));
        }

        // 3. 배낭 최대 용량 입력
        int maxWeight = Integer.parseInt(br.readLine());

        int res = knapsack(weights, values, maxWeight);
        System.out.println(res);
    }
}
```
<br>
<br>


**Bottom-Up : 1D**
```java
import java.io.*;
import java.util.*;

class Solution {
    public static int knapsack(List<Integer> weights, List<Integer> values, int maxWeight) {
        
        // dp 배열을 생성하고 -1로 초기화
        // 무게가 0일 때의 가치는 0으로 설정
        int[] dp = new int[maxWeight + 1];
        Arrays.fill(dp, -1);
        dp[0] = 0;
        
        // 모든 물건을 차례대로 반복하며 확인
        for (int i = 0; i < weights.size(); i++) {
            
            // 0/1 배낭 문제에서는 하나의 물건을 중복해서 넣을 수 없으므로
            // 반드시 '최대 무게부터 현재 물건의 무게까지' 거꾸로(역순으로) 반복해야 함
            for (int j = maxWeight; j >= weights.get(i); j--) {
                
                // 이전 물건들을 이용해 해당 무게(j - 현재 물건 무게)를 만들 수 있는 경우에만 갱신 시도
                if (dp[j - weights.get(i)] != -1) {
                    // 기존에 구한 j 무게의 가치와 (현재 물건을 넣었을 때의 가치) 중 더 큰 값으로 갱신
                    dp[j] = Math.max(dp[j], dp[j - weights.get(i)] + values.get(i));
                }
            }
        }
        
        // dp 배열을 전체 순회하며 가장 큰 값(최대 가치)을 찾는다
        int maxValue = -1;
        for (int i = 0; i <= maxWeight; i++) {
            maxValue = Math.max(maxValue, dp[i]);
        }
        
        return maxValue;
    }

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st;

        // 1. 무게 입력
        st = new StringTokenizer(br.readLine());
        List<Integer> weights = new ArrayList<>();
        while(st.hasMoreTokens()) {
            weights.add(Integer.parseInt(st.nextToken()));
        }

        // 2. 가치 입력
        st = new StringTokenizer(br.readLine());
        List<Integer> values = new ArrayList<>();
        while(st.hasMoreTokens()) {
            values.add(Integer.parseInt(st.nextToken()));
        }

        // 3. 최대 무게 입력
        int maxWeight = Integer.parseInt(br.readLine());

        int res = knapsack(weights, values, maxWeight);
        System.out.println(res);
    }
}
```

## Fractional Knapsack
> 물건을 부분적으로 담는 것이 허용되는 문제
