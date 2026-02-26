# Java Object Ordering: Comparable & Comparator

### 1. Comparable Interface

객체의 **Natural Ordering(기본 정렬 순서)**을 정의하기 위해 사용

* 클래스 자체에 정렬 로직을 내장시켜, `Collections.sort()`나 `Arrays.sort()` 호출 시 별도의 인자 없이도 정렬이 가능하게 만들기 위함
* 주로 숫자(오름차순), 문자열(사전순)처럼 보편적인 정렬 기준이 필요한 도메인 모델(Entity, DTO)에 구현
* **정의 방법**: `Comparable<T>` 인터페이스 상속 후 `compareTo(T o)` 메서드 오버라이딩

<br/>

#### `compareTo` 리턴값에 따른 정렬 메커니즘

Java의 정렬 알고리즘은 리턴되는 정수의 **부호**에 따라 위치를 결정

* **음수 (-1 등)**: 현재 객체(`this`)가 비교 대상(`o`)보다 **작다**고 판단 → 앞에 위치함.
* **0**: 두 객체가 **같다**고 판단.
* **양수 (+1 등)**: 현재 객체(`this`)가 비교 대상(`o`)보다 **크다**고 판단 → 뒤에 위치함.

> **Architect's Tip:** 직접 뺄셈(`this.id - o.id`)을 하면 정수 오버플로우 발생 위험이 있음. `Integer.compare(this.id, o.id)`나 `Double.compare()` 같은 Wrapper 클래스의 정적 메서드 활용을 권장함.

<br/>

### 2. Comparator & Lambda

기본 정렬 기준 외에 **특수 상황(역순, 다른 필드 기준 등)**에 맞는 정렬이 필요할 때 사용함.

* **람다 활용**: Java 8부터는 인터페이스를 직접 구현하지 않고 람다식으로 간결하게 표현함.

    ```java
    // 나이 기준 오름차순 정렬 (람다)
    list.sort((o1, o2) -> o1.getAge() - o2.getAge());

    // Comparator.comparing 활용
    list.sort(Comparator.comparingInt(Person::getAge).reversed());

    ```

<br/>

### 3. 요약

* **Comparable**: 클래스의 본질적인 정렬 기준이 있을 때 사용함 (내부 구현)
* **Comparator**: 정렬 기준이 가변적이거나, 외부 라이브러리 클래스를 정렬해야 할 때 사용함 (외부 주입)
* **정렬 안정성**: `Arrays.sort()`는 기본형엔 Dual-Pivot Quicksort를, 객체엔 Stable한 **TimSort**를 사용하여 정렬 후에도 동일 값의 상대적 순서가 유지됨.