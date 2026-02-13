# Exhaustive-Search 

완전탐색: 가능한 **모든 경우의 수를 탐색**

**DFS (Depth-First Search)**

* **Concept:** 갈 수 있는 데까지 깊게 파고드는 방식
* **Best for:** 경로의 특징을 저장해야 하거나, 그래프의 모든 노드를 방문해야 할 때
* **Constraint:** 최단 거리를 보장하지 않으므로, 모든 경로를 다 확인한 뒤에야 최소 비용을 알 수 있음

**Backtracking (Optimization of DFS)**

* **Concept:** DFS로 탐색하되, **조건에 맞지 않는 경로는 즉시 포기(Pruning)** 하고 되돌아오는 기법<br/><br/>

**BFS (Breadth-First Search)**

* **Concept:** 시작점에서 가까운 노드부터 레벨별로 **모든 정점을 빠짐없이 방문**하는 방식
* 답이 여러 개일 때, 그중 **가장 얕은 깊이(최소 단계)** 에 있는 답을 먼저 찾아야 하는 완전 탐색 문제에 적합

<br/>


**Summary**



| Category | Algorithm | Core Goal | Data Structure |
| --- | --- | --- | --- |
| **Exhaustive Search** | DFS | 모든 경로/조합 탐색 | Stack / Recursion |
| **Exhaustive Search** | BFS | 모든 상태를 레벨별로 탐색 | Queue |
| **Optimization** | Backtracking | 가망 없는 경로 차단(가지치기) | DFS + Condition |

---