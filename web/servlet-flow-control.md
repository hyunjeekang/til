# 서블릿 흐름 제어

<br>

## 1. 서블릿 간 통신이 필요한 이유

하나의 서블릿이 모든 책임을 지는 대신 작업을 분리해서 다른 서블릿으로 데이터를 전달하고 처리를 위임해야 하는 상황이 자주 발생한다.

이때 개발자가 직접 다른 서블릿 인스턴스를 생성할 수는 없다. **서블릿 객체의 생성과 관리 권한은 WAS**에 있기 때문이다. 따라서 다른 서블릿을 호출하려면 WAS에게 실행을 위임하는 방식을 사용해야 한다.

<br>

## 2. 흐름 제어 방식

### Forward

현재 서블릿이 `request`, `response` 객체를 그대로 유지한 채 다른 서블릿이나 JSP로 제어권을 완전히 넘기는 방식이다.

- WAS 내부에서 이동하므로 브라우저 URL이 변경되지 않는다
- `request` 객체가 그대로 유지되어 데이터를 공유할 수 있다
- 서블릿 → JSP로 데이터를 전달할 때 주로 사용한다
- WAS 내부에서만 이동하므로 외부 서버나 다른 컨텍스트로는 이동 불가

```java
request.getRequestDispatcher("/WEB-INF/views/member.jsp")
       .forward(request, response);
```

![Forward 흐름](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FcCiYyn%2FbtrCHdWxymT%2FAAAAAAAAAAAAAAAAAAAAACTf1iy4jwkpMqM8pu5mP8pKMKqxr2Cmr-Udht2_R9Nr%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1777561199%26allow_ip%3D%26allow_referer%3D%26signature%3DrWcmzHprZ0qLwT15ZqMM%252BMo4I1g%253D)

<br>

### Include

다른 서블릿이나 JSP의 실행 결과를 현재 서블릿의 응답에 포함시키는 방식이다.

- Forward와 달리 제어권이 완전히 넘어가지 않는다. 포함된 리소스 실행이 끝나면 다시 원래 서블릿으로 돌아온다
- 공통 header, footer, navigation 등 반복되는 화면 조각을 삽입할 때 활용한다

```java
request.getRequestDispatcher("/WEB-INF/views/header.jsp")
       .include(request, response);
```

<br>

### Redirect

클라이언트(브라우저)에게 특정 주소로 다시 요청하라고 지시하는 방식이다.

- 브라우저가 새 URL로 재요청하므로 URL이 변경된다
- 요청이 한 번 끊어지고 새롭게 연결되므로 `request` 객체가 초기화된다
- WAS 내부 리소스뿐 아니라 외부 서버나 다른 컨텍스트로도 이동 가능하다
- `sendRedirect()` 를 호출해도 이후 코드가 계속 실행되므로, 조건문으로 처리하거나 메서드 마지막에 호출해야 한다

```java
response.sendRedirect(request.getContextPath() + "/member/list");
// context path를 붙여야 정확한 절대 경로로 이동된다.
```

![Redirect 흐름](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2F5mi7Q%2FbtrCIllqCJU%2FAAAAAAAAAAAAAAAAAAAAAAVj_J3Huzv4aYfhiRO9bPZeQT2aAZZ5OML2NLKXuGKV%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1777561199%26allow_ip%3D%26allow_referer%3D%26signature%3D3CfzKGEx1LxxOXu0GJlIW%252BktYlU%253D)

<br>

## 3. Forward vs Redirect 비교

| 구분 | Forward | Redirect |
|---|---|---|
| 이동 주체 | WAS 내부 | 클라이언트(브라우저) |
| URL 변경 | 없음 | 변경됨 |
| `request` 유지 | 유지됨 | 새로 생성됨 |
| 속도 | 빠름 (서버 내부 처리) | 상대적으로 느릴 수 있음 |
| 이동 범위 | 같은 컨텍스트 내부만 | 외부 서버·다른 컨텍스트 포함 전부 |
| 새로고침 시 | 이전 로직 재실행됨 (중복 위험) | 새 URL을 GET 재요청 (중복 x) |
| 사용 메서드 | `RequestDispatcher.forward()` | `HttpServletResponse.sendRedirect()` |
| 주요 사용처 | 조회·렌더링 (서블릿 → JSP) | INSERT/UPDATE 후 다른 페이지로 이동 |
<br>

### 공통 이동 처리 패턴

서블릿에서 forward와 redirect를 하나의 흐름으로 처리하는 방식이다.

```java
String url = /* 이동할 경로 */;

if (url.startsWith("redirect:")) {
    url = url.substring(url.indexOf(":") + 1);
    response.sendRedirect(request.getContextPath() + url);
} else {
    request.getRequestDispatcher(url).forward(request, response);
}
```
<br>

### 언제 Redirect를 써야 할까?

회원 입력(INSERT) 후 Forward로 목록 페이지에 이동하면, URL이 여전히 `/member?action=insert` 상태다. F5를 누르면 INSERT가 다시 실행되어 중복 데이터가 생긴다. INSERT, UPDATE, DELETE 처리 후에는 반드시 Redirect를 사용하는 것이 좋다.
이 두 방식에 대해 더 자세히 알아보자.<br><br>

#### 회원 입력 페이지 -> 목록 페이지 `Forward`, `Response` 방식 비교

두 방식은 모두 클라이언트로부터 `request`를 받아 `response`를 다시 보낸다. 다만 서버가 브라우저에게 명령(`redirect`)을 내리는지, 결과물을 보여주냐에 따라 달라진다.

먼저 새로고침은 화면을 다시 그리는 것이 아니라 서버에 보냈던 **마지막 요청**을 다시 보내는 것이다.

**1. `Forward` 방식**
- 링크가 바뀌지 않는다.
- 사용자 마지막 요청 : `POST /addMember`
- 서버 : 멤버를 추가하고 화면을 던져준다.
- 주소 : `/addMember`

- 새로고침 이후 : 브라우저가 기억하는 마지막 요청인 `POST /addMember`(멤버 추가 요청)을 한 번 더 보낸다. (중복!)

**2. `Redirect` 빙식**
- 링크가 바뀐다.
- 서버 : 멤버를 추가하고 `302` 상태 코드로 리다이렉션을 명령한다
- 사용자 마지막 요청 : `GET / memberList`로 링크를 바꾸어서 이동
- 주소창 : `/addMember` -> `/memberList`

- 새로고침 이후 : 브라우저가 기억하는 마지막 요청인 `GET / memberList`을 다시 수행한다. (중복 POST 해결!)

#### 결론
글쓰기, 가입, 결제 등 행동을 했다면 **Redirect**<br>
목록, 상세 페이지 보기 등 데이터 조회 후 화면 로딩이라면 **Forward** <br>

<br>

## 4. JSP 호출 방법과 WEB-INF 보안

JSP를 호출하는 방법은 두 가지가 있다.

### 서블릿에서 JSP 호출 (권장)

```
클라이언트  →  서블릿 요청  →  서블릿이 내부적으로 JSP로 forward
```

JSP를 `WEB-INF` 하위에 두면 클라이언트가 URL로 직접 접근할 수 없다. `WEB-INF/` 하위 파일은 서버 내부에서만 접근 가능하다. `WEB-INF/classes/` 에 서블릿이 있으므로 서블릿은 JSP를 자유롭게 forward할 수 있다.

```
webapp/
  WEB-INF/
    classes/         ← 서블릿 .class 위치
    views/
      member.jsp     ← JSP는 여기에 (URL 직접 접근 차단)
```

### JSP 직접 호출 (비권장)

```html
<a href="/app/member.jsp">회원 입력</a>
```

JSP를 `webapp/` 바로 아래에 두면 URL로 직접 접근이 가능하다. 서블릿을 거치지 않으므로 인증·권한 체크를 우회할 수 있어 보안에 취약하다.

**JSP는 반드시 `WEB-INF` 하위에 두고, 서블릿을 통해서만 호출한다.**

<br>

## 5. Response Buffer와 Forward

Forward를 사용할 때 알아야 할 개념이다.

웹 서버는 응답 데이터를 즉시 전송하지 않고 버퍼에 모았다가 꽉 차면 일괄 전송(Flush)한다.

A 서블릿이 B 서블릿으로 제어권을 넘기면, 서버는 응답의 일관성을 위해 A 서블릿이 버퍼에 써둔 데이터를 전부 비운다. B 서블릿이 작성한 데이터만 클라이언트에게 전달하기 위해서다.

만약 A 서블릿의 버퍼가 이미 가득 차서 클라이언트에 일부 데이터가 전송된 후라면 제어권을 넘길 수 없어 에러가 발생한다.
