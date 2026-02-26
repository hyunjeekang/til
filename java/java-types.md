# Java Types: Primitive vs Reference & Floating Point

### 1. Primitive Type vs Reference Type

Java는 성능을 위한 기본형과 객체 지향의 유연성을 위한 참조형을 모두 제공

| 구분 | Primitive Type | Reference Type |
| --- | --- | --- |
| **종류** | `int`, `long`, `double`, `boolean` 등 | `Integer`, `Long`, `Double`, `Boolean` 등 |
| **저장 위치** | **Stack** 영역에 실제 값 저장 | **Heap** 영역에 객체 저장, Stack은 주소값만 가짐 |
| **메모리** | 작음 (예: `int`는 4바이트) | 큼 (객체 헤더 오버헤드 포함, 약 16바이트 이상) |
| **기본값** | `0`, `0.0`, `false` 등 | `null` |
| **Generic** | 사용 불가 | **사용 가능 (Collection 저장 시 필수)** |

<br/>


### 2. 부동소수점(Floating Point)과 오차 처리

Java의 `float`와 `double`은 **IEEE 754 표준**을 따르며, 이진법으로 소수를 표현하기 때문에 정밀도 한계가 존재함.

#### 이진법 표현의 한계

* `0.1 + 0.2`를 계산하면 `0.30000000000000004`가 나옴
* 컴퓨터는 `0.1`을 정확한 이진수로 표현하지 못하고 무한소수로 처리하기 때문

#### Epsilon (ε)을 이용한 비교

두 실수가 "사실상 같은지" 판단할 때는 산술 연산자(`==`) 대신 허용 오차(`epsilon`)를 사용

* **기법:** `Math.abs(a - b) < epsilon`
* **기준값:** 보통 `1E-5` (0.00001) 정도를 쓰지만, 연산의 정밀도 요구사항에 따라 달라짐

```java
double a = 0.1 + 0.2;
double b = 0.3;
double epsilon = 1E-9;

if (Math.abs(a - b) < epsilon) {
    // 두 값은 같다고 간주함
}

```

<br/>

### 3. 관련 유틸리티 및 주의사항

* **Autoboxing / Unboxing:** Java 5부터 자동으로 변환해주지만, 루프 내에서 대량으로 발생하면 성능 저하의 주범
* **`Arrays.sort()`와 오차:** 실수를 정렬할 때는 `Double.compare()`를 사용해야 `-0.0`이나 `NaN` 같은 특수 케이스를 정확히 처리할 수 있음

