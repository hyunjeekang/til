# Arrays.sort vs Collections.sort

### 1. 정렬 메서드별 시간 복잡도와 내부 알고리즘
자바에서 정렬을 할 때 어떤 데이터 타입을 넣느냐에 따라 내부 알고리즘과 시간 복잡도가 완전히 달라진다.

* **원시 타입 배열 정렬 (`int[]`, `double[]` 등)**
    * **메서드:** `Arrays.sort()`

    * **알고리즘:** Dual-Pivot Quicksort
    * **시간 복잡도:** 평균 $O(n \log n)$, **최악 $O(n^2)$**
    * **특징:** 값이 같은 원소의 원래 순서가 유지되지 않는 **불안정 정렬(Unstable Sort)**이다.


* **참조 타입(객체) 배열 및 컬렉션 정렬 (`Integer[]`, `String[]`, `ArrayList` 등)**
    * **메서드:** `Arrays.sort(Object[])` / `Collections.sort(List)`

    * **알고리즘:** TimSort (Merge Sort + Insertion Sort 결합)
    * **시간 복잡도:** 평균 및 **최악 모두 $O(n \log n)$** 보장
    * **특징:** 값이 같은 원소의 순서가 보장되는 **안정 정렬(Stable Sort)**이다.

### 2. 공식 문서로 확인하기

* **Delegation:** `Collections.sort(List)`는 직접 정렬을 수행하지 않고 내부적으로 `List.sort()` 메서드에 동작을 위임한다. [Collections-Method Detail](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html)에는 다음과 같이 명시되어 있다.
    > *"This implementation defers to the `List.sort(Comparator)` method using the specified list and a null comparator."* <br>
    > (해당 구현은 지정된 리스트와 null comparator를 사용하여 `List.sort(Comparator)` 메서드에 위임한다.)

* **TimSort의 특징:** 
  * 객체 배열에 사용되는 정렬은 Python의 Tim Peters가 만든 TimSort를 사용한다.

  * 완전히 무작위인 데이터에서는 일반적인 Merge Sort의 성능을 가진다. 
  * **배열이 이미 부분적으로 정렬되어 있는 경우(Partially sorted) 비교 횟수가 $n \log n$보다 훨씬 적어지며** 거의 $O(n)$에 가까운 속도를 낸다.

* **Mutually Comparable:** 
  * 정렬하려는 객체들은 반드시 `Comparable` 인터페이스를 구현하거나 정렬 시 `Comparator`를 명시해 주어야 한다. 

  * 그렇지 않으면 `ClassCastException`이 발생한다.

### 3. 메서드 사용 시 유의점 
1. **메모리:** 
    - `int`는 4바이트이지만 `Integer` 객체는 객체 헤더와 패딩 등으로 인해 16바이트 이상의 메모리를 소모한다. 

    - 배열의 크기가 커질수록 메모리 초과 위험이 커진다.
  
2. **캐시 지역성:** 
   - 원시 타입 배열은 메모리상에 데이터가 연속적으로 빈틈없이 붙어 있어 CPU 캐시 히트율이 높고 연산이 매우 빠르다. 

   - 참조 타입 배열은 메모리 곳곳에 흩어진 객체의 '주소'를 찾아가야 하므로 병목이 발생하고 느리다.