# Disjoint Set 서로소 집합

공통 원소가 없는 여러 개의 집합을 관리하기 위한 구조
`Union-Find` 알고리즘이라고도 불린다.
그래프 내의 `사이클 판별`, `최소 신장 트리(MST)`의 `크루스칼 알고리즘` 등에 사용

<br/>

## 핵심 개념

**Find**
특성 원소가 속한 집합의 대표자(루트 노드) 찾기
두 원소의 루트가 같다면 같은 집합에 속한 것
그래프에서 같은 그룹인지 확인

**Union**
두 개의 개별적인 집합을 하나로 합침
한쪽 노드를 다른쪽 루트노드의 자식으로 연결
그래프에서 간선을 연결

## 구현 - Naive Union-Find

**1. 초기화**

```java
public static void makeSets(){
    parents = new int[N];
    for(int i = 0 ; i < N; i++){
        parents[i] = i;
    }
}
```

`N`개의 서로소집합을 저장하기 위해 자기 자신을 부모 노드로 초기화한다.

**2. Find**

```java
public static int findSet(int a){
    if(a == parents[a]) return a;
    return findSet(parents[a]);
}
```

`a` 노드의 루트 노드를 찾는 함수이다.
각 노드의 부모 노드를 재귀적으로 탐색해 루트 노드를 리턴한다.

**3. Union**
```java
public static boolean union(int a, int b){
    int aRoot = findSet(a);
    int bRoot = findSet(b);

    if(aRoot == bRoot) return false;

    parents[bRoot] = aRoot;
    return true;
}
```

노드 `a`와 노드 `b`를 합집합 연산한다.
각 노드의 루트 노드를 찾고, 부모가 같다면 이미 같은 집합 내의 노드이므로 `false`를 리턴한다.
두 노드가 다른 집합의 원소라 합집합 연산이 가능하다면
노드 `b`의 루트 노드를 노드 `a`의 루트 노드로 변경해 같은 집합에 속하도록 합집합 연산한다.

**4. 시간 복잡도**

`find`가 루트까지 올라가야 하기 때문에 `O(N)`

| 연산    | 시간복잡도    |
| ----- | -------- |
| find  | **O(N)** |
| union | **O(N)** |

<br/><br/>

## 최적화

이렇게 진행하면 두 서로소 집합의 합집합 연산과 각 원소의 집합을 손쉽게 구분할 수 있을 것이다.
다만 위 진행 방식에는 두 가지 문제가 존재한다.

**find()의 시간복잡도?**
 현재 `union` 연산은 단순히 노드 `b`의 루트 노드를 노드 `a`의 루트 노드로 변경한다.
이렇게 부모를 연결하기만 하면 **트리가 한 쪽으로 편중(Skewed)하는 현상**이 발생할 수 있다. 
`findSet` 함수 호출 시 최악의 경우 `O(N)`만큼 부모 노드를 타고 올라가야지만 루트 노드를 찾을 수 있는 경우가 생기게 되어 비효율적이다. (`x → parent → parent → root`)

<br/>

### Path Compression
이러한 문제점을 경로 압축으로 해결할 수 있다.

```java
public static int findSet(int a) {
    if (a == parents[a]) return a;
    return parents[a] = findSet(parents[a]); // 경로 압축
}
```

`findSet` 수행 시에 찾은 루트 노드를 부모로 설정하는 것이다.
재귀를 타면서 부모를 갱신하게 되어, 다음에 `findSet(a)`를 호출할 때는 바로 루트 노드를 찾을 수 있다.
모든 노드가 루트에 직접 연결된 트리가 되는 것이다!

**Path Compression 적용 시의 시간복잡도**
| 연산    | 시간복잡도           |
| ----- | --------------- |
| find  | **O(log N)** |
| union | **O(log N)**    |

<img src="https://algocoding.wordpress.com/wp-content/uploads/2014/09/uf3_path_compression.png">

<br/>

### Union by Rank

`union`시 **높이가 낮은 트리를 높은 트리 밑에 붙이도록** 강제하는 것이다

왜 높이가 낮은 트리를 높이가 높은 트리 밑에 붙이는 것이 좋을까?

두 개의 트리 `A`(높이 : 2), `B`(높이 : 1)을 합친다고 가정해 보자.

1. **낮은 트리(B)를 높은 트리(A)에 붙일 때**
    전체 트리의 높이가 여전히 2로 동일 
    높이가 더 큰 트리(`A`) 밑에 짧은 트리(`B`)가 들어가기 때문에 기존 A의 깊이보다 더 깊어지지 않는다.

2. **높은 트리(A)를 낮은 트리(B)에 붙일 때**
    전체 트리의 높이가 3으로 늘어난다
    `B`가 루트가 되면서 그 밑의 `A` 노드들이 깊이가 1씩 더 깊어진다.

`Union by Rank` 적용 시 트리 높이가 **log₂N 이하**로 제한된다.

<br/>

**Union by Rank 적용 시 시간 복잡도**

| 연산    | 시간복잡도        |
| ----- | ------------ |
| find  | **O(log N)** |
| union | **O(log N)** |

```java
int[] parents;
int[] rank; // 각 트리의 높이를 저장

public void union(int a, int b) {
    int rootA = findSet(a);
    int rootB = findSet(b);

    if (rootA != rootB) {
        // 1. Rank가 더 높은 쪽을 부모로 결정
        if (rank[rootA] > rank[rootB]) {
            parents[rootB] = rootA;
        } else if (rank[rootA] < rank[rootB]) {
            parents[rootA] = rootB;
        } else {
            // 2. Rank가 같다면 한쪽을 부모로 정하고 Rank를 1 높임
            parents[rootB] = rootA;
            rank[rootA]++;
        }
    }
}
```

<img src="https://algocoding.wordpress.com/wp-content/uploads/2014/09/uf4_union_by_rank.png">

<br/><br/>

## 구현 - 최적화

기본 구현 + Path Compression + Rank

```java
public class DisjointSet {
    private int[] parent;
    private int[] rank;

    public DisjointSet(int n) {
        parent = new int[n + 1];
        rank = new int[n + 1];
        // 초기화
        for (int i = 1; i <= n; i++) {
            parent[i] = i;
            rank[i] = 0;
        }
    }

    // find
    public int find(int x) {
        if (parent[x] == x) {
            return x;
        }
        // path compression
        return parent[x] = find(parent[x]);
    }

    // union
    public void union(int x, int y) {
        int rootX = find(x);
        int rootY = find(y);

        if (rootX != rootY) {
            // Rank
            if (rank[rootX] < rank[rootY]) {    // x
                parent[rootX] = rootY;
            } else if (rank[rootX] > rank[rootY]) {
                parent[rootY] = rootX;
            } else {
                parent[rootY] = rootX;
                rank[rootX]++;
            }
        }
    }

    public static void main(String[] args) {
        DisjointSet ds = new DisjointSet(5);

        ds.union(1, 2);
        ds.union(2, 3);
        
        System.out.println("1과 3은 같은 집합인가? " + (ds.find(1) == ds.find(3))); // true
        System.out.println("1과 4는 같은 집합인가? " + (ds.find(1) == ds.find(4))); // false
    }
}
```

**Union by Rank + Path Compression 적용 시 시간복잡도**

| 연산    | 시간복잡도       |
| ----- | ----------- |
| find  | **O(α(N))** |
| union | **O(α(N))** |

여기서 `α(N)` 은`Inverse Ackermann Function` 이다.
매우 느리게 증가하는 함수라고 생각하면 된다.

| N       | α(N) |
| ------- | ---- |
| 10      | 3    |
| 10⁶     | 4    |
| 10⁶⁰⁰   | 4    |
| 10⁶⁰⁰⁰⁰ | 5    |

`α(N) < 5`일 경우 거의 거의 `O(1)`라고 보면 된다

<br/>
