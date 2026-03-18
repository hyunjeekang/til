# Map

<img src = "https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhbm3QylZhYLRtYDoEGgSHtFn6tstKPFZaF9OZi0WTWjtAlmUDRK_YnjDxaEfQDdEmXA7UWTiFhNa-MtqJpsu6ccsNW4i7y2XySrNizhL5qaHPyIo6R4UYHHvdyT6CADGwPn2f54W3s4IEb/s1600/map-interface.jpg">

### TreeMap

내부적으로 레드-블랙 트리 자가 균형 이진 탐색 트리로 구현된다<br/>

<img src = "https://upload.wikimedia.org/wikipedia/commons/6/66/Red-black_tree_example.svg"> <br/> <br/>

**TreeMap 특징**
- 자동 정렬
  - 오름차순으로 자동 정렬된다
- 양방향 탐색
  - 최솟값(가장 왼쪽 노드), 최댓값(가장 오른쪽 노드) 탐색이 매우 빠르다.
- 시간 복잡도
  - 삽입, 삭제, 검색 모두 $O(\log N)$

**주요 메서드**

- `firstKey()`
  - 현재 트리에서 가장 작은(첫 번째) 키를 반환
- `lastKey()`
  - 현재 트리에서 가장 큰(마지막) 키를 반환
- `put(K, V)`
  - 데이터 넣기 (이미 있다면 덮어쓰기)
- `remove(K)`
  - 해당 키를 삭제
- `getOrDefault(K, D)`
  - 키가 있으면 그 값, 없으면 기본값(D) 반환

### references
- [TreeMap in Java](https://www.scaler.com/topics/treemap-in-java/)