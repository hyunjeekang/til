# Topology Sort - 위상 정렬

<br/>

### 1. Concept and Characteristics

- **정의:** 방향 그래프의 모든 노드를 **방향성에 어긋나지 않게 순서대로 나열**하는 것
- **목적:** 선후 관계가 있는 작업의 순서를 결정
- **필수 조건:** **DAG (Directed Acyclic Graph)**
    - **방향성**이 있어야 함
    - **Cycle**이 없어야 함 (시작점을 찾을 수 없기 때문)

<img src="https://inginious.org/course/competitive-programming/graphs-toposort/anim.gif" width = "80%">

<br/>

### 2. Key Terminology

* **진입 차수(In-degree):** 특정 노드로 **들어오는** 간선의 개수 (나보다 먼저 처리되어야 하는 노드 수)
* **진출 차수(Out-degree):** 특정 노드에서 **나가는** 간선의 개수 (내가 먼저 처리되어야 처리할 수 있는 노드의 수)

<br/>


### 3. Algorithm Implementation

#### **① BFS (Kahn's Algorithm)**

**진입 차수(In-degree)** 를 활용

1. 진입 차수가 **0**인 노드를 모두 큐(Queue)에 삽입
2. 큐가 빌 때까지 다음 과정을 반복
* 큐에서 원소를 꺼내 결과 리스트에 담기
* 해당 노드와 연결된 모든 간선을 제거
* 새롭게 **진입 차수가 0이 된 노드**를 큐에 삽입


**결과:** 큐에서 꺼낸 순서가 위상 정렬의 결과 <br/>
(만약 모든 노드를 방문하기 전에 큐가 빈다면 사이클이 존재하는 것)

<img src = "https://iq.opengenus.org/content/images/2020/03/algo.gif" width = "80%">

<br/>

#### **② DFS**

**진출 차수(Out-degree)** 를 활용

1. 방문하지 않은 노드에 대해 DFS를 수행
2. DFS가 해당 노드의 모든 인접 노드를 확인하고 **리턴(종료)되기 직전**에 스택(또는 리스트)에 노드를 기록
3. 모든 노드 방문 후, 기록된 순서를 **역순**으로 뒤집으면 위상 정렬 결과가 됨

<img src = "https://media.licdn.com/dms/image/v2/D4E22AQFLJPDEwmzAMw/feedshare-shrink_800/B4EZaWmMYPHYAg-/0/1746283339974?e=2147483647&v=beta&t=8SVy3KuAOhL_ni81zWxTbLNBX_os-qKFUtzpK1eBFII" width="50%">

<br/>

### Implementation (Python)

#### **BFS (Kahn's Algorithm)**

```python
from collections import deque

def topology_sort_bfs(v, adj, indegree):
    result = []
    q = deque()

    # 1. 진입 차수가 0인 노드 큐에 삽입
    for i in range(1, v + 1):
        if indegree[i] == 0:
            q.append(i)

    while q:
        now = q.popleft()
        result.append(now)
        
        # 2. 해당 노드와 연결된 노드들의 진입 차수 -1
        for next_node in adj[now]:
            indegree[next_node] -= 1
            # 3. 새롭게 진입 차수가 0이 된 노드 삽입
            if indegree[next_node] == 0:
                q.append(next_node)

    # 사이클 체크: 결과 리스트의 길이가 노드 개수와 다르면 사이클 발생
    if len(result) != v:
        return "Cycle!"
    return result

```

<br/>

#### **DFS**

```python
def topology_sort_dfs_stable(v, adj):
    visited = [False] * (v + 1)
    in_recursion = [False] * (v + 1) # Track nodes in current recursion stack
    stack = []

    def dfs(now):
        visited[now] = True
        in_recursion[now] = True # Add to current path
        
        for next_node in adj[now]:
            if not visited[next_node]:
                if not dfs(next_node): # Cycle detected in deeper call
                    return False
            elif in_recursion[next_node]:
                # If visited AND still in recursion stack, it's a Back Edge!
                return False
        
        in_recursion[now] = False # Remove from path before returning
        stack.append(now)
        return True

    for i in range(1, v + 1):
        if not visited[i]:
            if not dfs(i):
                return "Cycle Detected!"
                
    return stack[::-1]
```

<br/>


### 5. Performance and Results

* **시간 복잡도:**  (V: 정점 개수, E: 간선 개수)
  * 모든 노드를 확인, 각 노드에서 나가는 간선을 한 번씩 모두 확인하기 때문


* **특징:** 위상 정렬의 결과는 하나가 아닐 수 있음
  * 진입 차수가 0인 노드가 여러 개일 경우 어느 것을 먼저 선택하느냐에 따라 달라짐

<br/>

### 6. References
[Graphs - Topological sort](https://inginious.org/course/competitive-programming/graphs-toposort?lang=pt) <br/>
[Topological Sort for Course Schedule](https://media.licdn.com/dms/image/v2/D4E22AQFLJPDEwmzAMw/feedshare-shrink_800/B4EZaWmMYPHYAg-/0/1746283339974?e=2147483647&v=beta&t=8SVy3KuAOhL_ni81zWxTbLNBX_os-qKFUtzpK1eBFII)<br/>
[Topological Sorting using Kahn's Algorithm](https://iq.opengenus.org/content/images/2020/03/algo.gif)<br/>
