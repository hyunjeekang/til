# JavaScript Engine: Scope & Memory Lifecycle

### 1. Runtime Environment: Global Object

자바스크립트 엔진이 실행되는 환경(Context)에 따라 최상위 루트 객체가 결정된다.

* **Browser:** `window` 객체
* **Node.js:** `global` 객체

---

### 2. Variable Lifecycle & Scope Architecture

#### `var`: Function-Level Scope

* **동작 원리:** 블록(`{ }`)을 무시하고 **함수 단위**로 메모리 공간을 확보한다.
* **호이스팅(Hoisting):** 컴파일 단계에서 **선언부를 스코프 최상단으로 끌어올리며 `undefined`로 즉시 초기화**한다.
* **위험성:** 의도치 않은 전역 변수화(Global Pollution)와 재선언 허용으로 인한 런타임 버그 유발 가능성이 크다.

#### `let` & `const`: Block-Level Scope

* **동작 원리:** `{ }` 내부에서만 유효한 엄격한 가독 범위를 가진다.
* **TDZ (Temporal Dead Zone):** 호이스팅은 발생하지만, 선언문 이전 접근 시 **ReferenceError**를 발생시켜 논리적 선행을 강제한다. (초기화 전 사용 금지)

#### `Undeclared` (암묵적 전역)

* 키워드 없이 `i = 10`과 같이 할당할 경우, 엔진은 스코프 체인을 따라 최상위까지 탐색 후 없으면 **전역 객체(`window`)의 속성**으로 강제 등록한다.
---

### 3. Advanced Concepts

#### Scope Chain (식별자 결정)

함수가 중첩될 경우, 엔진은 내부에서 외부 방향으로 변수를 찾아 나간다. 이를 스코프 체이닝이라 한다.

> **Case Study: 접근 제어 오류**

```javascript
var a = function() {
    var b = 20; // 함수 'a'의 지역 변수
}
a();
console.log(b); // ReferenceError: b is not defined

```

* **분석:** `b`는 `a`의 실행 컨텍스트 내부에 캡슐화되어 있다. 외부(Global Context)에서는 내부 하위 스코프에 직접 접근할 수 없으므로 참조 오류가 발생한다.

#### Closure (클로저)

함수가 생성될 당시의 **어휘적 환경(Lexical Environment)**을 기억하는 메커니즘이다. <br/>
상위 함수가 실행 종료되어 스택에서 제거되더라도, 하위 함수가 참조하고 있는 상위 스코프의 변수는 가비지 컬렉션(GC) 대상에서 제외되어 **메모리에 유지**된다.

---

### 4. Code Quiz (Review)

**Q1. 호이스팅 시점의 값 확인**

```javascript
console.log(x); // 결과: ?
var x = "Architecture";

```

* **정답:** `undefined` (선언과 동시에 초기화됨)

**Q2. TDZ에 의한 런타임 에러**

```javascript
console.log(y); // 결과: ?
let y = "Engineering";

```

* **정답:** `ReferenceError` (호이스팅은 되었으나 초기화 전 접근 불가 구역)

**Q3. 클로저를 통한 상태 유지**

```javascript
const counter = (() => {
    let count = 0;
    return () => ++count;
})();
console.log(counter()); // ?
console.log(counter()); // ?

```

* **정답:** `1`, `2` (내부 함수가 소멸된 상위 스코프의 `count`를 유지함)

---