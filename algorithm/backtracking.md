# Backtracking

<br/>

- [Backtracking](#backtracking)
    - [Summary](#summary)
    - [Brute-Force vs Backtracking](#brute-force-vs-backtracking)
    - [Pseudocode](#pseudocode)
    - [N-Queen ](#n-queen-)


<br/>

### Summary
모든 조합을 시도해서 문제의 해를 찾는다 (기본 골격: 완전 탐색) <br/>
가능성들을 트리처럼 구성 가능, 가지(선택지)중 하나의 가지 선택<br/>
선택 -> 유망 체크 -> 다음 선택 / 최근의 선택지로 돌아가서 다른 선택 -> 최종 상태 도달<br/>
보통 **재귀 함수**로 구현 (돌아가기 : 함수 호출 스택)

<br/>

### Brute-Force vs Backtracking

- 상태 공간 트리의 모든 가능한 경로를 탐색
- 상태 공간 트리 : 비선형 자료구조 -> BFS, DFS(완전탐색)도 가능

- **Brute-Force**
  - 모든 노드(해의 후보)를 방문
  - 답이 될 수 있는지 확인
  - 해답이 될 가능성이 전혀 없는 후손 노드들도 검색해야 함 -> 비효율

- **Backtracking**
  - 상태 공간 트리의 깊이 우선 검색 진행
  - 각 노드의 유망성 검사
  - 유망(promising)하지 않다면
  - 부모로 되돌아가(backtracking) 다음 자식 노드로 이동
  - 가지치기(pruning) : 유망하지 않은 노드를 포함하는 경로는 더이상 고려하지 x

<br/>

### Pseudocode
```
backtrack(node v)
  
  IF promising(v) == false then **return**
  
  IF there is a solution at v
    write the solution
  
  ELSE
    FOR each child u of v
      **backtrack( u )**
```
- 가능한 선택만 하므로 리프노드까지 도착한다면 해에 도달한 것
- 백트래킹 사용 시 일반적으로 경우의 수가 줄어들지만, 최악의 경우 지수함수 시간 요구

<br/>

### [N-Queen](https://www.acmicpc.net/problem/9663) <br/>
크기가 N × N인 체스판 위에 퀸 N개를 서로 공격할 수 없게 놓는 경우의 수<br/>
N이 주어졌을 때, 퀸을 놓는 방법의 수를 구하는 문제

**N = 8 이라면**
완전 탐색 시 
64칸 중 8칸을 뽑아야 함 `64C8` -> 약 44억가지<br/>
실제 정답은 92개뿐임

백트래킹!!
