# Path Finding

### 1. Shortest Path (최단 경로 탐색)

**BFS (Breadth-First Search):**
* **Condition:** **모든 간선의 가중치가 동일할 때** 사용
* **Mechanism:** 시작점에서 가까운 노드부터 레벨별로 탐색, 큐(`Queue`)를 사용함 -> 목표 노드를 처음 만나는 순간이 곧 **최단 경로**임을 보장
* **Efficiency:** $O(V + E)$

<center><img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*GChPGXvZQiVwjok9EvKPIA.gif" width = "90%"></center>

<br/>

**Dijkstra's Algorithm:** <br/>

* **Condition:** **간선의 가중치가 서로 다를 때** (가중치는 양수) 사용
* **Mechanism:** 우선순위 큐(`Priority Queue`)를 사용하여 **현재 가장 비용이 적은 노드**를 먼저 확정하며 탐색하는 그리디 방식
* **Efficiency:** $O(E \log V)$ <br/>

<center><img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*3aibaGt1-zimnwreliwX0A.gif" width = "90%"></center>
<br/>

<br/>

### 2. Gemeral Path Finding (모든 경로 탐색)
**DFS (Depth-First Search)**
* **Condition:** 특정 경로 존재 여부 확인하거나, 경로의 특징(거쳐온 노드 속성)을 유지해야할 떄 사용
* **Mechanism:** 스택(`Stack`)이나 재귀를 사용하여 **한 방향으로 끝까지** 파고든다.
* **Efficiency:** $O(V + E)$ <br/>

<center><img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*yBXw4Q8rSMRqGYC-iZI0yg.gif" width = "90%"></center>

---

<br/>

### Decision Making

| Problem Requirement | Strategy | Data Structure |
| --- | --- | --- |
| **All Possible Combinations** | **Exhaustive Search** (DFS) | Stack / Recursion |
| **Optimal Path (Weight = 1)** | **Shortest Path** (BFS) | Queue |
| **Optimal Path (Weight > 1)** | **Shortest Path** (Dijkstra) | Priority Queue |

---
<br/>


**Reference Sites** <br/>

[Path Finding Algorithms](https://medium.com/omarelgabrys-blog/path-finding-algorithms-f65a8902eb40)

---