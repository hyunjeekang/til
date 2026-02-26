# Java Data Structures

### 1. Array (배열)

연속된 메모리 공간을 점유하며 성능은 최상이나 유연성이 떨어짐

* **고정 크기:** 생성 시 크기 결정 후 변경 불가
* **시간 복잡도:** 인덱스 접근/수정은 `$O(1)$`

#### 메모리 구조 및 초기화

* **`int[] array = {1, 2, 3};`은 Heap에 들어가는가?**
  - **Heap에 들어감.** Java의 모든 배열은 객체이다. 변수는 Stack에, 실제 데이터는 Heap에 할당됨

<br/>

* **초기화 값**
  * **0** 기본형 배열은 타입별 기본값(0, false 등)으로, 참조형은 `null`로 자동 초기화됨


#### `java.util.Arrays` 유틸리티

* `Arrays.sort(arr)`: **Dual-Pivot Quicksort** 사용 ($O(N \log N)$)
* `Arrays.binarySearch(arr, key)`: 이진 탐색 수행 (정렬 필수)
* `Arrays.toString(arr)`: 배열 요소를 문자열로 변환

<br/>

### 2. Collection Framework & Wrapper Class

`Collection`은 객체만 저장 가능 <br/>
따라서 `int` 같은 기본형은 `Integer` 같은 **Wrapper Class**를 사용해야 함

* **이유:** 제네릭(`<>`)은 컴파일 시 타입 안정성을 체크하며 내부적으로 `Object` 기반으로 동작
* 기본형은 `Object`를 상속받지 않으므로 **Auto-Boxing**이 필요

<br/>

### 3. ArrayList (리스트)

* **내부 동작:** 가변 배열, 공간이 부족하면 기존 크기의 약 1.5배 배열을 새로 만들어 복사
* **성능:**
* 맨 뒤 추가(`add`): `$O(1)$`.
* 삭제/중간 삽입: 데이터를 시프트해야 하므로 `$O(N)$`


* **메서드:** `size()`, `isEmpty()`, `contains()`, `get(index)`, `set(index, val)`

<br/>

### 4. HashMap & HashSet

#### HashMap (Key-Value)

* **`map.get(key)`:** 키가 없으면 `null` 반환, `getOrDefault()` 사용 권장
* **핵심 메서드:** `put()`, `remove()`, `containsKey()`, `keySet()`

#### HashSet (Key Only)

* 중복 허용 안 함
* 내부적으로 `HashMap`의 Key 구조를 사용
* **메서드:** `add()`, `remove()`, `contains()`

<br/>

### 5. String & StringBuilder

#### String (Immutable)

* 불변 객체
* 수정 시 새로운 객체가 생성되어 메모리 낭비 가능성
* `replace()`, `substring()` 등은 모두 **신규 객체를 반환**함

#### StringBuilder & StringBuffer (Mutable)

* 내부 `char[]`를 직접 수정하여 효율적
* **StringBuilder:** 단일 스레드/코테용 (빠름)
* **StringBuffer:** 멀티 스레드용 (Thread-safe)

---
