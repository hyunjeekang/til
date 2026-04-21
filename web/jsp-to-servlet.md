# JSP 내부 구조

<br>

## 1. JSP → Servlet 변환 구조

JSP 파일은 컴파일 과정을 거쳐 `HttpJspBase`를 상속받는 자바 파일로 변환된다. 개념상 `HttpServlet`을 상속받는다고 이해하면 된다.

```java
// ─────────────────────────────────────────
// <%@ %> Directive → 클래스 바깥 import문으로 변환
// ─────────────────────────────────────────
import java.util.*;
import javax.servlet.*;
import javax.servlet.http.*;

public class MyJsp_jsp extends HttpJspBase {

    // ─────────────────────────────────────
    // <%! %> Declaration → 클래스 멤버 영역으로 변환
    // ─────────────────────────────────────
    int globalCount = 0;

    public void myMethod() {
        // 메서드 선언 가능
    }
    // ⚠️ 여기는 클래스 바디이므로 if문, 메서드 호출 같은 실행문 단독 작성 불가

    public void _jspInit() { /* 초기화 */ }

    // ─────────────────────────────────────
    // 클라이언트 요청마다 실행되는 핵심 메서드
    // ─────────────────────────────────────
    public void _jspService(HttpServletRequest request, HttpServletResponse response) {

        // 내장 객체들이 여기서 자동 선언·초기화됨
        PageContext    pageContext  = _jspxFactory.getPageContext(this, request, response, ...);
        HttpSession    session      = pageContext.getSession();
        ServletContext application  = pageContext.getServletContext();
        JspWriter      out          = pageContext.getOut();

        // ─────────────────────────────────
        // <% %> Scriptlet → 이 메서드 내부로 복사
        // ─────────────────────────────────
        int localCount = 0;     // 지역 변수
        globalCount++;          // 멤버 변수 변경 가능
        myMethod();             // 메서드 호출 가능
        // ⚠️ 메서드 내부이므로 새 메서드 선언 불가

        out.write("<html><body>");

        // ─────────────────────────────────
        // <%= %> Expression → out.print() 인자로 변환
        // ─────────────────────────────────
        out.print(globalCount);  // <%= globalCount %>
        out.print(localCount);   // <%= localCount %>
        // ⚠️ 세미콜론 붙이면 컴파일 에러

        out.write("</body></html>");
    }

    public void _jspDestroy() { /* 소멸 */ }
}
```

### 요소별 변환 위치 정리

| JSP 태그 | 변환 위치 | 가능 | 불가능 |
|---|---|---|---|
| `<%@ %>` | 클래스 바깥 | import, 페이지 설정 | - |
| `<%! %>` | 클래스 멤버 영역 | 멤버 변수 선언, 메서드 정의 | 실행문, 내장 객체 사용 |
| `<% %>` | `_jspService()` 내부 | 실행문, 내장 객체 사용 | 메서드 선언 |
| `<%= %>` | `out.print()` 인자 | 변수·반환값 출력 | 세미콜론 |

<br>

## 2. 선언부에서 `request`를 쓸 수 없는 이유

`request`는 `_jspService()`의 매개변수이므로 메서드 바깥인 선언부에서는 존재 자체를 알 수 없다.

```java
public class MyJsp_jsp extends HttpJspBase {

    // <%! %> 선언부 → 클래스 멤버 영역
    // _jspService() 바깥이므로 request가 없는 공간
    // ❌ request.getAttribute("data") → "cannot find symbol" 컴파일 에러
    String myData = "";

    public void _jspService(HttpServletRequest request, HttpServletResponse response) {
        // ✅ 이 메서드의 매개변수로 request가 들어오므로 사용 가능
        // 스크립틀릿 코드가 이 안으로 들어오기 때문에 스크립틀릿에서도 사용 가능
        Object data = request.getAttribute("data");
    }
}
```

<br>

## 3. 내장 객체 (Implicit Object)

### 왜 선언 없이 바로 쓸 수 있나?

WAS가 JSP를 서블릿으로 변환할 때, `_jspService()` 최상단에 아래 코드를 자동으로 삽입하기 때문이다.

```java
public void _jspService(HttpServletRequest request, HttpServletResponse response) {

    // 개발자가 선언하지 않아도 자동 생성되는 내장 객체들
    PageContext    pageContext  = _jspxFactory.getPageContext(this, request, response, ...);
    HttpSession    session      = pageContext.getSession();
    ServletContext application  = pageContext.getServletContext();
    JspWriter      out          = pageContext.getOut();

    // 이 아래에 스크립틀릿·표현식 코드가 들어감
}
```

### 주요 내장 객체

| 객체 | 타입 | 역할 |
|---|---|---|
| `request` | `HttpServletRequest` | 클라이언트 요청 정보 (파라미터, 헤더 등) |
| `response` | `HttpServletResponse` | 클라이언트 응답 설정 (리다이렉트, 쿠키 등) |
| `session` | `HttpSession` | 세션 유지 (로그인 상태 등) |
| `application` | `ServletContext` | 애플리케이션 전체 공유 객체 |
| `out` | `JspWriter` | 화면 출력. 표현식 `<%= %>`이 내부적으로 이 객체를 사용 |
| `pageContext` | `PageContext` | 현재 페이지 컨텍스트 정보 |

내장 객체들은 모두 `_jspService()`의 지역 변수다. 스크립틀릿·표현식에서는 사용 가능하고, 선언부에서는 사용 불가다.

<br>

## 4. Web Scope

속성(attribute)을 저장하고 공유할 수 있는 네 가지 범위다.

![Web Scope](https://www.oreilly.com/api/v2/epubs/urn:orm:book:0596005636/files/httpatomoreillycomsourceoreillyimages137889.png)

| Scope | 객체 | 유지 범위 |
|---|---|---|
| page | `pageContext` | 현재 JSP 페이지 안에서만 |
| request | `request` | 하나의 요청(forward 포함) 동안 |
| session | `session` | 브라우저 세션이 유지되는 동안 |
| application | `application` | 서버가 실행되는 동안 (전체 공유) |

모든 Scope 객체에서 공통으로 사용하는 메서드:

```java
scope.setAttribute("key", value);   // 저장
scope.getAttribute("key");          // 조회
scope.removeAttribute("key");       // 삭제
scope.getAttributeNames();          // 전체 키 목록 (Enumeration)
```

<br>

## 5. Parameter vs Attribute

출처와 방향이 다르다!

| 구분 | Parameter | Attribute |
|---|---|---|
| 출처 | 클라이언트에서 전송 | 서버에서 직접 저장 |
| 전송 방법 | GET(URL), POST(폼 데이터), AJAX | `request.setAttribute(key, value)` |
| 조회 방법 | `request.getParameter("key")` → `String` | `request.getAttribute("key")` → `Object` |
| 수정 여부 | 불가, 읽기 전용 | `setAttribute()`, `removeAttribute()` 로 수정·삭제 가능 |
| `set` 메서드 | 없음, 서버가 클라이언트 값을 임의로 설정할 수 없음 | `setAttribute()` 있음 |
| 주요 용도 | 사용자 입력값, 폼 데이터, 검색어 | 서블릿 간 데이터 전달, DB 조회 결과 |

```java
// Parameter: 클라이언트 → 서버
// URL: /search?name=kim
String name = request.getParameter("name");  // "kim"

// Attribute: 서버 → 서버 (서블릿 → JSP 등)
request.setAttribute("user", userObject);    // 저장
Object user = request.getAttribute("user");  // 꺼내서 사용
```
