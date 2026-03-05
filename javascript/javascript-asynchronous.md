# Javascript Asynchronous

## 동기(Synchronous) vs 비동기(Asynchronous)

### 동기 (Synchronous)

* 코드가 위에서부터 아래로 순차적으로 실행된다.
* 흐름이 간단하고 직관적이다.
* **단점:** 특정 작업(예: 대용량 이미지 다운로드)이 오래 걸리거나 응답이 늦어지면 그 작업이 끝날 때까지 화면이 멈추고 다음 작업들이 지연된다. (Block 현상)

### 비동기 (Asynchronous)

* 특정 작업의 완료를 기다리지 않고, 다음 코드를 먼저 실행한다.
* 웹 페이지가 멈추지 않고 부드럽게 동작할 수 있도록 돕는 핵심 개념

<img src="https://kissflow.com/hubfs/synchronous-programming-vs-asynchronous-programming.webp" width = "80%">

**대표적인 비동기 함수:**

* `fetch`: 서버로부터 데이터를 받아오는 네트워크 API 호출
* `setTimeout`: 특정 시간 이후 콜백 함수를 실행
* `setInterval`: 특정 시간마다 콜백 함수를 주기적으로 실행

<br/><br/>

## 싱글 스레드인데 어떻게 비동기가 가능할까?

자바스크립트는 기본적으로 한 번에 하나의 작업만 처리할 수 있는 **싱글 스레드(Single Thread)** 언어이다.<br/>
그런데 어떻게 여러 비동기 작업들을 동시에 처리하는 것처럼 보일까??

그 비밀은 브라우저 환경이 제공하는 **이벤트 루프(Event Loop)** 에 있다.

<img src="https://media.licdn.com/dms/image/v2/D5612AQHIuZDc3cqPtg/article-cover_image-shrink_600_2000/article-cover_image-shrink_600_2000/0/1721189705579?e=2147483647&v=beta&t=7z1ivEBMlIOpeq4P2UUbbrj1T64ysIpkPv27efVvq60" width = "80%">

<br/>

### 핵심 요소 역할 정리

* **Call Stack (콜 스택)** 
  * 자바스크립트 코드가 실제로 실행되는 공간이다. **싱글 스레드** 이므로 한 번에 하나씩만 쌓이고 실행된다.

* **Web APIs:** 
  * 브라우저가 제공하는 기능 (`setTimeout`, `DOM 이벤트`, `AJAX/Fetch` 등)
  * 오래 걸리는 비동기 작업은 콜 스택에서 여기로 넘겨진다.

* **Task Queue (매크로태스크 큐):** 
  * Web API의 작업이 끝나면, 실행되어야 할 콜백 함수들이 순서를 기다리는 대기열이다. 
  * (`setTimeout` 등의 콜백)

* **Microtask Queue (마이크로태스크 큐):** 
  * Task Queue와 같지만 **우선순위가 더 높은** 대기열이다.
  * 주로 `Promise`의 `.then()`, `.catch()`, `.finally()` 콜백들이 이곳에 담긴다.

* **Event Loop (이벤트 루프):** 
  * 콜 스택이 비어있는지 지속적으로 확인한다.
  * 콜 스택이 비어있으면 큐에 대기 중인 작업을 콜 스택으로 끌어올려 실행시키는 관리자 역할을 수행!

<br/>

**이벤트 루프의 실행 우선순위**

1. 현재 **콜 스택**에 있는 모든 동기 코드를 끝까지 실행한다.
2. 콜 스택이 비어있으면, **마이크로태스크 큐**에 있는 모든 작업을 먼저 콜 스택으로 올려 실행한다.
3. 마이크로태스크 큐까지 완전히 비워지면, 그제야 **태스크 큐**에 있는 작업을 하나씩 가져와 실행한다.

<br/><br/>

## AJAX와 callback hell

**AJAX**란, 화면 전체를 갱신(새로고침)하지 않고 클라이언트와 서버 간에 데이터를 비동기적으로 교환하는 기술이다. 과거에는 `XMLHttpRequest` 객체를 주로 사용했다.

하지만 비동기 처리가 많아지면서 문제가 발생한다. 비동기 작업의 '순서'를 보장하기 위해 (예: 이미지 다운로드 -> 압축 -> 필터 적용 -> 저장) 콜백 함수 안에 콜백 함수를 계속 중첩해야 하는 것!

```javascript
// 콜백 지옥 (Callback Hell) 발생!
getImage('./image.png', (image, err) => {
    if (err) throw new Error(err);
    compressImage(image, (compressedImage, err) => {
        if(err) throw new Error(err);
        applyFilter(compressedImage, (filteredImage, err)=> {
            if(err) throw new Error(err);
            saveImage(filteredImage, (res, err) => {
                if(err) throw new Error(err);
                console.log("Successfully saved image!");
            });
        });
    });
});

```

<img src="https://binarymusings.org/wp-content/uploads/2025/11/shock-wave.gif">

<br/><br/>

## Promise

콜백 지옥을 해결하기 위해 ES6에서 **Promise**가 등장한다. Promise는 '비동기 작업의 최종 완료 또는 실패를 나타내는 객체'이다. 함수에 콜백을 직접 전달하는 대신, 결과 객체(Promise)에 콜백을 첨부하는 방식이다.

<img src="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/promises.png">

### Promise의 3가지 상태 (PromiseStatus)

* `pending` : 이행하지도, 거부하지도 않은 대기(초기) 상태
* `fulfilled` : 연산이 성공적으로 완료됨 (결과값 반환)
* `rejected` : 연산이 실패함 (에러 반환)

### Promise Chaining

`.then()`을 사용하면 결과를 아래로 계속 연결할 수 있어 코드가 간결(?)해진다.

* `.then()` : Promise가 해결(resolved)된 이후 호출
* `.catch()` : Promise가 거부(rejected)된 이후 에러 처리로 호출
* `.finally()` : 성공/실패 여부와 상관없이 무조건 마지막에 호출

**콜백 지옥을 Promise로 개선한 코드**

```javascript
getImage('./image.png')
    .then(image => compressImage(image))
    .then(compressedImage => applyFilter(compressedImage))
    .then(filteredImage => saveImage(filteredImage))
    .then(res => console.log("Successfully saved image!"))
    .catch(err => console.error(err));

```

<br/><br/>

## async / await

Promise 덕분에 콜백 hell은 피했지만, 꼬리에 꼬리를 무는 `Then Hell`이 발생하기도 한다. 이를 해결하기 위해 ES8에서 **가장 직관적이고 동기 코드처럼 읽히는 `async / await`**가 도입되었다.

* 함수 앞에 `async`를 선언하면 그 함수는 자동으로 Promise를 반환한다.
* Promise 비동기 함수 앞에 `await`를 붙이면, 해당 작업이 완료될 때까지 함수의 실행을 일시 중지(suspend)하고 기다린다.

```javascript
const processImageAsync = async () => {
    try {
        // 비동기지만 await를 만나면 동기 코드처럼 순서를 기다리게 된다.
        const image = await getImage('./image.png');
        const compressed = await compressImage(image);
        const filtered = await applyFilter(compressed);
        const res = await saveImage(filtered);
        
        console.log("성공!", res);
    } catch (e) {
        console.log("실패", e); // .catch() 역할
    } finally {
        console.log("끝.."); // .finally() 역할
    }
};

processImageAsync();

```

<br/><br/>

## Fetch API

현대 자바스크립트에서 네트워크 요청을 수행할 때 가장 기본적으로 사용하는 내장 API이며, **Promise를 기반으로 동작**한다.

```javascript
// fetch는 Promise<Response>를 반환합니다.
fetch('https://jsonplaceholder.typicode.com/posts/1', {
  method: 'PATCH',
  body: JSON.stringify({
    title: 'foo',
  }),
  headers: {
    'Content-type': 'application/json; charset=UTF-8',
  },
})
  .then((response) => response.json()) // json() 파싱 역시 Promise를 반환
  .then((json) => console.log(json));

```

이를 방금 배운 `async / await`로 리팩토링해보았다.

```javascript
const updatePost = async () => {
    try {
        const response = await fetch('https://jsonplaceholder.typicode.com/posts/1', {
            method: 'PATCH',
            // ... 
        });
        const json = await response.json();
        console.log(json);
    } catch (error) {
        console.error("통신 에러:", error);
    }
}

```

<br/>

#### References

* [Synchronous vs. Asynchronous](https://kissflow.com/application-development/asynchronous-vs-synchronous-programming/)
* [JavaScript Visualized: Promises & Async/Await](https://medium.com/@lydiahallie/javascript-visualized-promises-async-await-a3f1aad8a943)
* [자바스크립트의 핵심 '비동기' 완벽 이해](https://inpa.tistory.com/entry/%F0%9F%8C%90-js-async)
* [MDN: Promise](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise)
* [[새로운 기능] fetch API](https://goodteacher.tistory.com/506)
