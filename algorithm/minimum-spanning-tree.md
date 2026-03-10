# Minimum Spanning Tree 최소 신장 트리

## 개요
**그래프에서 최소 비용 문제**
- 모든 정점을 연결하는 간선들의 가중치의 합이 최소가 되는 트리
- 두 정점 사이의 최소 비용의 경로 찾기

**신장 트리**<br>
n개의 정점으로 이루어진 무향 그래프에서 n개의 정점과 n-1개의 간선으로 이루어진 트리 <br>
사이클 발생 x

**최소 신장 트리**<br>
무향 가중치 그래프에서 신장 트리를 구성하는 **간선들의 가중치의 합이 최소**인 신장 트리

<img src="https://media.geeksforgeeks.org/wp-content/uploads/20240502002413/Producer.png">

<br>

## 최소 신장 트리 만들기
최소 신장 트리를 어떻게 찾아볼까?

#### 완전탐색<br>
  1. 간선 선택<br>
  2. 트리를 만족하는지 검사(사이클이 발생하는지 확인)<br>
  3. 최소값 갱신<br>

    가능은 하지만 경우의 수가 많아진다...
    E개의 정점에서 V-1개의 간선을 선택 -> 매우 비효율적!

<br>

### MST: Greedy Choice Problem

최소 신장 트리는 탐욕 알고리즘이 **항상 최적해를 보장**하는 문제이다.
그 이유는 그리디의 핵심 조건 두 가지를 완벽하게 충족하기 때문이다.

#### 1. 문제 분해 가능 (Optimal Substructure)

* MST의 일부분(서브 트리) 또한 그 정점들을 잇는 최소 비용의 트리여야 한다.
* 큰 그래프의 MST를 구하는 문제는, 결국 작은 정점 집합들을 최소 비용으로 연결해 나가는 **작은 문제들의 결합**으로 쪼개서 해결할 수 있다.

#### 2. 현재의 선택이 미래에 악영향을 주지 않음 (Greedy Choice Property)

* '컷 속성(Cut Property)'에 의해 어떤 시점에서든 사이클을 만들지 않는 **가장 저렴한 간선**을 선택하는 것은 나중에 더 좋은 간선을 선택할 기회를 박탈하지 않는다. 즉 미래에 악영향을 주지 않는다!
* 지금 가장 싼 간선을 골랐다고 해서, 나중에 연결해야 할 다른 정점들의 연결 비용이 갑자기 비싸지거나 연결이 불가능해지지 않는다.
  
<br>

### Kruskal vs Prim: Greedy 관점 차이

두 알고리즘은 '가장 적은 비용인 것을 고른다'는 전략은 같지만 **어디서** 찾느냐에 따라 관점이 다르다.

| 구분 | Kruskal | Prim |
| --- | --- | --- |
| **탐욕의 범위** | **전역적(Global) 탐욕** | **국소적(Local) 탐욕** |
| **선택 기준** | 그래프 전체 간선 중 **가장 가중치가 작은** 간선부터 검토 | 현재 트리에서 **확장 가능한** 간선 중 **가장 작은 간선** 선택 |
| **관점** | 적은 비용의 간선들을 모으다 보면 트리가 됨 | 지금 트리에서 가장 적은 비용으로 트리를 넓히기 |


<br><br>

## Kruskal Algorithm

**과정**
1. 초기화 : 모든 간선을 가중치에 따라 **오름차순** 정렬
2. **가중치가 가장 낮은 간선**부터 선택하며 트리를 증가시킴<br>
    `사이클이 존재하면` 남아 있는 간선 중 그 다음으로 가중치가 낮은 간선 선택
3. `n-1`개의 간선이 선택될 때까지 반복

<br>

2번에서 사이클이 존재하는지를 어떻게 확인할까?<br>
[서로소 집합(Union Find)](/algorithm/disjoint-set.md)을 통해 확인할 수 있다!

**Union Find과 KRUSKAL**
- `Find` 두 정점이 이미 같은 집합(트리)에 속해 있는지 확인
- `Union` 속해있지 않다면 두 정점을 하나의 집합으로 합침 

`Find`를 통해서 사이클 여부를 확인하고, `Union`을 통해서 노드를 연결한다!

<br>

**구현(java)**
```java
import java.util.*;

class Edge implements Comparable<Edge> {
    int from, to, weight;
    Edge(int from, int to, int weight) {
        this.from = from;
        this.to = to;
        this.weight = weight;
    }
    @Override
    public int compareTo(Edge o) {
        return this.weight - o.weight; // 가중치 오름차순 정렬
    }
}

public class KruskalMST {
    static int[] parent;

    public static int find(int x) {
        if (parent[x] == x) return x;
        return parent[x] = find(parent[x]); // 경로 압축
    }

    public static void union(int x, int y) {
        x = find(x);
        y = find(y);
        if (x != y) parent[y] = x;
    }

    public static void main(String[] args) {
        int V = 7; // 정점 개수
        List<Edge> edges = new ArrayList<>();
        // edges.add(new Edge(from, to, weight)); 형태로 간선 추가

        Collections.sort(edges); // 1. 간선 정렬 (탐욕적 선택의 준비)
        parent = new int[V + 1];
        for (int i = 1; i <= V; i++) parent[i] = i;

        int mstWeight = 0;
        int count = 0;

        for (Edge edge : edges) {
            if (find(edge.from) != find(edge.to)) { // 2. 사이클이 안 생기면 선택
                union(edge.from, edge.to);
                mstWeight += edge.weight;
                count++;
                if (count == V - 1) break; // n-1개 간선 선택 시 종료
            }
        }
        System.out.println("Kruskal MST 총 비용: " + mstWeight);
    }
}
```

## Prim Algorithm
하나의 정점에서 연결된 간선들 중에 **가장 비용이 적은** 길을 하나씩 선택하면서 MST를 만들어 가는 방식

- **임의의 정점 선택**해서 시작
- 선택한 정점과 인접하는 정점들 중의 **최소 비용의 간선이 존재하는 정점 선택**
- 모든 정점이 선택될 때까지 반복

**Priority Queue**<br>
현재 트리에서 갈 수 있는 가장 저렴한 길을 빠르게 찾을 수 있다.<br>
새로운 정점이 추가될 때마다 그 정점과 연결된 간선들을 `pq`에 추가<br>
가중치가 가장 낮은 간선을 뽑아 쓴다.

<br>

**구현(java)**
```java
import java.util.*;

class Node implements Comparable<Node> {
    int to, weight;
    Node(int to, int weight) {
        this.to = to;
        this.weight = weight;
    }
    @Override
    public int compareTo(Node o) {
        return this.weight - o.weight;
    }
}

public class PrimMST {
    public static void main(String[] args) {
        int V = 7;
        List<Node>[] adj = new ArrayList[V + 1]; // 인접 리스트
        for (int i = 1; i <= V; i++) adj[i] = new ArrayList<>();
        
        // adj[from].add(new Node(to, weight)); 형태로 그래프 구성

        boolean[] visited = new boolean[V + 1];
        PriorityQueue<Node> pq = new PriorityQueue<>();
        
        pq.add(new Node(1, 0)); // 시작 정점 설정 (비용 0)
        int mstWeight = 0;
        int count = 0;

        while (!pq.isEmpty()) {
            Node current = pq.poll();

            if (visited[current.to]) continue; // 이미 트리에 포함된 정점이면 패스

            visited[current.to] = true;
            mstWeight += current.weight;
            if (++count == V) break;

            for (Node next : adj[current.to]) {
                if (!visited[next.to]) {
                    pq.add(next); // 3. 아직 방문 안 한 정점으로의 간선들을 PQ에 담기
                }
            }
        }
        System.out.println("Prim MST 총 비용: " + mstWeight);
    }
}
```



###  KRUSKAL vs PRIM

| 비교  | Kruskal | Prim |
| --- | --- | --- |
| **중심 사고** | **간선(Edge)** 위주 | **정점(Vertex)** 위주 |
| **시간 복잡도** | $O(E \log E)$ | $O(E \log V)$ (우선순위 큐 기준) |
| **유리한 상황** | 간선이 적은 **희소 그래프(Sparse)** | 간선이 아주 많은 **밀집 그래프(Dense)** |
| **특징** | 간선 정렬이 시간의 대부분 | 시작 정점에 따라 탐색 순서가 달라짐 |

<br>

### MST vs Shortest Path

| 구분 | MST | Shortest Path |
| --- | --- | --- |
| **핵심 목표** | **모든 정점**을 잇는 **간선 가중치 총합** 최소화 | **시작점**에서 **각 정점**까지의 **거리** 최소화 |
| **알고리즘** | Kruskal, Prim | Dijkstra, Bellman-Ford |
| **결과물** | 하나의 트리 ($V-1$개의 간선) | 시작점 기준의 최단 경로 트리 |
| **응용 사례** | 통신망 설계, 도로 구축, 전기 회로 | 지도 앱 경로 탐색, 네트워크 패킷 라우팅 |
