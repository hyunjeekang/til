# Session & Cookie

HTTP는 기본적으로 무상태(stateless) 프로토콜이다. 요청이 끝나면 연결이 끊어지므로, 서버는 다음 요청이 같은 클라이언트로부터 왔다는 것을 알 수 없다. 세션과 쿠키는 이 문제를 해결하기 위한 상태 유지 메커니즘이다.

<br>

## 1. Cookie

### 개념

서버가 생성하고 **브라우저가 저장**하는 데이터다.

- 서버가 `Set-Cookie` 헤더로 클라이언트에 전달한다.
- 브라우저는 이후 같은 서버에 요청할 때마다 쿠키를 자동으로 함께 전송한다.
- 문자열 데이터를 저장하며, 크기 제한이 있어 많은 데이터를 담기엔 무겁다.

```java
// 서블릿에서 쿠키 생성 및 전달
Cookie cookie = new Cookie("userId", "hong");
cookie.setMaxAge(60 * 60 * 24); // 유효시간 (초 단위, 24시간)
response.addCookie(cookie);

// 쿠키 읽기
Cookie[] cookies = request.getCookies();
```

<br>

## 2. Session

### 개념

서버가 생성하고 **서버가 저장**하는 데이터다. 쿠키의 보안·용량 문제를 해결하기 위해 사용한다.

- 실제 데이터는 서버에 저장하고, 브라우저에는 세션을 식별하는 **JSESSIONID** 쿠키만 전달한다.
- 브라우저가 다음 요청 시 JSESSIONID를 함께 보내면, 서버는 이 키로 해당 세션을 찾아 열어준다.

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

>*메시지를 표시한 뒤 즉시 `c:remove`로 세션에서 제거하지 않으면 이후 페이지에서도 계속 alert이 뜬다.

<br>

## 5. Web Scope 전체 정리

| Scope | 객체 | 유지 범위 | 주요 용도 |
|---|---|---|---|
| page | `pageContext` | 현재 JSP 하나 | JSP 내부 임시 변수 |
| request | `request` | 하나의 요청 (forward 포함) | 서블릿 → JSP 데이터 전달 |
| session | `session` | 브라우저 세션 동안 | 로그인 상태, 사용자 정보 |
| application | `application` | 서버 실행 동안 (전체 공유) | 공통 설정, 방문자 수 등 |

> `page` scope는 JSP에만 있다. `<%@ include %>` 로 합쳐진 JSP들은 같은 `page` scope를 공유하지만, `<jsp:include>` 는 각자 별개의 `page` scope를 가진다.
