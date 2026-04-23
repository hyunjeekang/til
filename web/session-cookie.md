# Session & Cookie

HTTP는 기본적으로 무상태(stateless) 프로토콜이다. 요청이 끝나면 연결이 끊어지므로, 서버는 다음 요청이 같은 클라이언트로부터 왔다는 것을 알 수 없다. 세션과 쿠키는 이 문제를 해결하기 위한 상태 유지 메커니즘이다.

<br>

## 1. Cookie

### 개념

서버가 생성하고 **브라우저가 저장**하는 데이터다.

- 서버가 `Set-Cookie` 헤더로 클라이언트에 전달한다.
- 브라우저는 이후 같은 서버에 요청할 때마다 쿠키를 자동으로 함께 전송한다.
- 문자열 데이터를 저장하며, 크기 제한이 있어 많은 데이터를 담기엔 무겁다.

  <img src = "https://developer.mozilla.org/shared-assets/images/diagrams/http/cookies/cookie-basic-example.png" height="400">

```java
// 서블릿에서 쿠키 생성 및 전달
Cookie cookie = new Cookie("userId", "hong");
cookie.setMaxAge(60 * 60 * 24); // 유효시간 (초 단위, 24시간)
response.addCookie(cookie);

// 쿠키 읽기
Cookie[] cookies = request.getCookies();
```

<br>

### 쿠키 주요 속성

#### name

쿠키를 구별하는 유일한 값이다. 동일한 이름의 쿠키가 설정되면 기존 쿠키를 덮어쓴다.

- 알파벳, 숫자, 하이픈, 언더스코어, 물결표, 점 등으로 구성한다.
- 공백 등 나머지 문자는 URL 인코딩이 필요하다.

#### value

쿠키가 저장하는 값이다. 생성 시 생성자에 전달하거나 `setValue()` 메서드로 설정한다.

```java
new Cookie("userName", "hong-gil-dong");
```

#### path

쿠키가 유효한 경로를 지정한다. 해당 경로와 그 하위 경로로 요청할 때만 쿠키가 전송된다.

- 미설정 시 현재 서블릿의 context root로 자동 설정된다.
- `/` (container root)로 지정하면 동일 도메인의 다른 애플리케이션까지 쿠키가 전달되므로 보안에 주의해야 한다.

```java
cookie.setPath("/");          // 전체 경로에서 유효
cookie.setPath("/myapp");     // /myapp 하위에서만 유효
```

> 브라우저 개발자 도구 → Application → Cookies에서 path 컬럼을 확인하며 의도한 범위가 맞는지 점검하는 습관을 들이자.

#### maxAge

쿠키의 유효 기간을 초 단위로 설정한다.

| 값 | 동작 |
|---|---|
| 양수 | 지정한 초 후 만료 (예: `60 * 60 * 24` = 24시간) |
| 음수 | 세션 쿠키로 처리 — 브라우저 종료 시 폐기 |
| 0 | 브라우저에 도착 즉시 폐기 (쿠키 삭제에 사용) |

```java
cookie.setMaxAge(60 * 60 * 24); // 24시간 유지
cookie.setMaxAge(-1);           // 브라우저 종료 시 삭제
cookie.setMaxAge(0);            // 즉시 삭제
```

<br>

### 보안 관련 속성 (SameSite / HttpOnly / Secure)

| 요청 상황 (출발지 → 목적지) | 요청 방식 | Strict | Lax (기본값) | None (+Secure) |
| :--- | :--- | :--- | :--- | :--- |
| 주소창 직접 입력 / 북마크 | Direct | 전송 | 전송 | 전송 |
| 같은 사이트 내 이동 (mysite → mysite) | 모든 방식 | 전송 | 전송 | 전송 |
| 타 사이트에서 링크 클릭 (google → mysite) | `<a href="mysite"> mysite</a>` | 차단 | 전송 | 전송 |
| 타 사이트에서 폼 전송 (GET) | `<form method="GET">` | 차단 | 전송 | 전송 |
| 타 사이트에서 폼 전송 (POST) | `<form method="POST">` | 차단 | 차단 | 전송 |
| 타 사이트에서 이미지 로딩 | `<img>` `<iframe>` | 차단 | 차단 | 전송 |
| 타 사이트에서 비동기 요청 | AJAX / fetch | 차단 | 차단 | 전송 |

```java
Cookie cookie = new Cookie("sessionId", "abc123");

// 1. 값이 있는 속성: "속성=값" 형태
cookie.setAttribute("SameSite", "Strict");    // SameSite=Strict;
cookie.setAttribute("Domain", ".mysite.com"); // Domain=.mysite.com;

// 2. 값이 없는 속성 (boolean 속성) - "속성"만 표시
cookie.setAttribute("HttpOnly", "");          // HttpOnly;
cookie.setAttribute("Secure", "");            // Secure;

// 3. 속성 제거 (아예 없어짐)
cookie.setAttribute("SameSite", null);        // SameSite 사라짐
```

**Set-Cookie 헤더:**
```
Set-Cookie: sessionId=abc123; SameSite=Strict; HttpOnly; Secure;
```

<br>

#### SameSite (CSRF 방어)

다른 사이트에서 내 사이트로 요청을 보낼 때 쿠키를 함께 전송할지 결정한다. `Strict` `Lax` `None` 값을 가지며 기본값은 `Lax`다. 악의적인 사이트에서 강제로 요청을 보내는 CSRF 공격을 방어하는 데 사용된다.

#### Domain (쿠키의 활동 반경)

이 쿠키가 어떤 도메인으로 갈 때 전송될지 지정한다. `.mysite.com`으로 설정하면 `www.mysite.com` `api.mysite.com` `shop.mysite.com` 같은 서브도메인 요청에도 쿠키가 함께 전송된다. 지정하지 않으면 쿠키를 생성한 정확한 주소로만 전송된다.

#### HttpOnly (XSS 방어)

브라우저에서 자바스크립트로 쿠키를 읽지 못하게 막는다. XSS 공격으로 `document.cookie`를 통해 세션 ID를 탈취하는 것을 방지한다. 오직 HTTP 통신 시에만 서버로 전달된다.

#### Secure (도청 방어)

HTTPS 연결일 때만 쿠키를 전송하도록 강제한다. HTTP 통신은 평문으로 전달되므로 중간에서 쿠키를 가로챌 수 있는데, `Secure` 설정으로 이를 방지한다.

<br>

## 2. Session

### 개념

서버가 생성하고 **서버가 저장**하는 데이터다. 쿠키의 보안·용량 문제를 해결하기 위해 사용한다.

- 실제 데이터는 서버에 저장하고, 브라우저에는 세션을 식별하는 **JSESSIONID** 쿠키만 전달한다.
- 브라우저가 다음 요청 시 JSESSIONID를 함께 보내면 서버는 이 키로 해당 세션을 찾아 열어준다.

<br>

### 세션 생성 흐름

```
브라우저 최초 요청
  → 서버가 Session 객체 생성
  → 세션 ID(JSESSIONID) 생성
  → 브라우저에 JSESSIONID 쿠키 전달

이후 요청
  → 브라우저가 JSESSIONID 쿠키와 함께 요청
  → 서버가 해당 세션을 찾아 사용
```

서버는 연결된 브라우저 수만큼 세션 객체를 각각 유지한다.

<br>

### 세션 생명주기

| 이벤트 | 동작 |
|---|---|
| 브라우저 최초 요청 | 세션 생성 |
| 요청/응답 완료 | `request` 객체는 소멸, **세션은 유지** |
| 브라우저 종료 | 세션 소멸 (JSESSIONID 쿠키가 사라짐) |
| 세션 타임아웃 | 설정된 시간 동안 요청이 없으면 서버가 세션 삭제 |
| `session.invalidate()` | 강제 세션 소멸 (로그아웃에 사용) |

<br>

### 세션 사용법

```java
// 세션 가져오기 (없으면 새로 생성)
HttpSession session = request.getSession();

// 세션에 데이터 저장
session.setAttribute("loginUser", member);

// 세션에서 데이터 꺼내기
Member loginUser = (Member) session.getAttribute("loginUser");

// 세션에서 데이터 삭제
session.removeAttribute("loginUser");

// 세션 전체 무효화 (로그아웃)
session.invalidate();
```

JSP에서는 내장 객체 `session`으로 바로 접근한다.

```jsp
${sessionScope.loginUser.name}님 환영합니다.
```

<br>

### invalidate vs removeAttribute

| 메서드 | 동작 |
|---|---|
| `session.removeAttribute("key")` | 세션 자체는 유지하고 특정 데이터만 제거 |
| `session.invalidate()` | 세션 전체를 파괴하고 모든 데이터를 삭제 |

로그아웃 처리에는 `invalidate()`를 사용한다. 특정 데이터만 지우고 세션을 유지해야 할 때는 `removeAttribute()`를 사용한다.

<br>

### 세션 시간 관련 메서드

세션은 마지막 요청 시각을 기준으로 유효 기간을 계산한다. 요청이 들어올 때마다 타이머가 초기화되므로 사용 중인 세션은 만료되지 않는다.

```java
session.getCreationTime();            // 세션 최초 생성 시각 (ms)
session.getLastAccessedTime();        // 마지막 요청 시각 (ms)
session.getMaxInactiveInterval();     // 최대 유효 기간 (초)
session.setMaxInactiveInterval(60*30); // 최대 유효 기간 설정 (초 단위, 여기서는 30분)
```

> 유효 기간은 `getLastAccessedTime()` 이후 `getMaxInactiveInterval()` 초가 지나도록 요청이 없으면 만료된다.

<br>

## 3. Cookie vs Session 비교

| 구분 | Cookie | Session |
|---|---|---|
| 저장 위치 | 브라우저 | 서버 |
| 생성 주체 | 서버 (브라우저가 보관) | 서버 |
| 저장 가능 데이터 | 문자열만 | 모든 자바 객체 |
| 보안 | 낮음 (브라우저에 노출) | 높음 (서버에 저장, 키만 노출) |
| 용량 | 제한적 (4KB) | 서버 메모리 한도 내 |
| 유지 기간 | 설정된 만료일까지 | 브라우저 종료 또는 타임아웃까지 |
| 주요 용도 | 자동 로그인, 다크모드 설정 등 | 로그인 상태 유지 등 |

<br>

## 4. PRG 패턴에서 session을 쓰는 이유

POST → Redirect → GET 흐름에서 `request` 객체는 Redirect 시점에 소멸한다. 따라서 "등록 완료" 같은 메시지를 다음 페이지에 전달하려면 `session`에 담아야 한다.

```java
// POST 처리 후 (MemberController)
HttpSession session = request.getSession();
session.setAttribute("alertMsg", "등록되었습니다. 로그인 후 사용해주세요.");
redirect(request, response, "/");
```

```jsp
<%-- Redirect 후 도착한 index.jsp --%>
<c:if test="${!empty sessionScope.alertMsg}">
    <script>alert('${sessionScope.alertMsg}');</script>
    <c:remove var="alertMsg" scope="session" />
</c:if>
```

> 메시지를 표시한 뒤 즉시 `c:remove`로 세션에서 제거하지 않으면 이후 페이지에서도 계속 alert이 뜬다.

<br>

## 5. Web Scope 전체 정리

| Scope | 객체 | 유지 범위 | 주요 용도 |
|---|---|---|---|
| page | `pageContext` | 현재 JSP 하나 | JSP 내부 임시 변수 |
| request | `request` | 하나의 요청 (forward 포함) | 서블릿 → JSP 데이터 전달 |
| session | `session` | 브라우저 세션 동안 | 로그인 상태, 사용자 정보 |
| application | `application` | 서버 실행 동안 (전체 공유) | 공통 설정, 방문자 수 등 |

> `page` scope는 JSP에만 있다. `<%@ include %>`로 합쳐진 JSP들은 같은 `page` scope를 공유하지만, `<jsp:include>`는 각자 별개의 `page` scope를 가진다.