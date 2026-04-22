# JSP (Java Server Pages)

서블릿은 자바 코드 안에 HTML을 일일이 `out.write()`로 작성해야 했다. JSP는 이 불편함을 해소하기 위해 등장한 뷰 템플릿 기술로, HTML 안에 자바 코드를 삽입하는 방식을 사용한다.

<br>

## 1. 동작 원리

JSP 파일은 그 자체로 실행되지 않는다. 최초 요청 시 서블릿으로 변환된 후 실행되며, 이후 요청부터는 컴파일된 `.class`를 바로 사용한다.

```
.jsp 파일
  → 서블릿 소스코드로 변환  (.java)
  → 바이트코드로 컴파일     (.class)
  → WAS 실행

이후 요청 → .class 바로 실행 (변환·컴파일 생략)
```

<br>

## 2. 생명주기

WAS가 JSP 요청을 처리하는 흐름이다.

```
JSP 요청 수신
  │
  ├─ 서블릿 클래스 없음  →  서블릿 생성  →  객체 생성
  │
  └─ 이미 존재함  ─────────────────────────────┐
                                            ↓
                     jspInit()  →  _jspService()  →  jspDestroy()
```

![JSP Life Cycle](https://iq.opengenus.org/content/images/2019/05/jsplifecycle.png)

| 메서드 | 역할 |
|---|---|
| `jspInit()` | 최초 생성 시 1회 실행. 초기화 담당 |
| `_jspService()` | 클라이언트 요청마다 실행. 핵심 로직 처리 |
| `jspDestroy()` | 서블릿 소멸 시 실행. 자원 정리 |

<br>

## 3. 구성 요소

JSP는 네 가지 태그로 자바 코드를 HTML 안에 포함시킨다. 이 태그들은 **서버에서 처리되어 HTML로 변환**되며 브라우저까지 내려가지 않는다.

| 태그 | 이름 | 역할 | 변환 위치 |
|---|---|---|---|
| `<%@ %>` | Directive (지시자) | 페이지 속성, import 설정 | 클래스 바깥 (import문) |
| `<%! %>` | Declaration (선언부) | 멤버 변수·메서드 선언 | 클래스 멤버 영역 |
| `<% %>` | Scriptlet (스크립틀릿) | 자바 로직 실행 | `_jspService()` 메서드 내부 |
| `<%= %>` | Expression (표현식) | 값·반환값 화면 출력 | `out.print()` 인자로 변환 |

### Directive `<%@ %>`

JSP 페이지 전체에 적용되는 설정을 지정한다.

| 종류 | 설명 |
|---|---|
| `<%@ page %>` | 인코딩, contentType, import 등 기본 설정 |
| `<%@ include %>` | 다른 파일을 현재 JSP에 포함 |
| `<%@ taglib %>` | 커스텀 태그 라이브러리 선언 |

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<%@ page import="java.util.*" %>
```

### Declaration `<%! %>`

클래스의 멤버 변수나 메서드를 선언할 때 사용한다. `_jspService()` 메서드 바깥에 위치하므로 `request`, `session` 같은 내장 객체는 사용 불가고, 실행문도 단독으로 작성할 수 없다.

```jsp
<%!
    int visitCount = 0;

    public String greet() {
        return "Hello!";
    }
%>
```

### Scriptlet `<% %>`

`_jspService()` 내부로 변환되어 실행문을 자유롭게 작성할 수 있다. 여기서 선언한 변수는 지역 변수가 되고, 내장 객체도 그대로 사용 가능하다. 단, 메서드 내부이므로 새로운 메서드 선언은 불가능하다.

```jsp
<%
    int count = 0;
    visitCount++;
    String name = request.getParameter("name");
%>
```

### Expression `<%= %>`

변수나 메서드 반환값을 화면에 출력할 때 사용한다. 내부적으로 `out.print(...)` 로 변환된다.

```jsp
<%= visitCount %>    <%-- out.print(visitCount); 로 변환 --%>
<%= greet() %>       <%-- out.print(greet()); 로 변환 --%>
```

> **xx세미콜론xx** `<%= a; %>` 는 `out.print(a;);` 가 되어 컴파일 에러가 발생한다.

<br>

## 4. JSP vs Servlet 비교

| 구분 | Servlet | JSP |
|---|---|---|
| 코드 구조 | Java 코드 안에 HTML 포함 | HTML 안에 Java 코드 삽입 |
| 실행 과정 | 컴파일된 `.class`를 WAS가 직접 실행 | 최초 요청 시 서블릿으로 변환 후 실행 |
| UI 작업 | 복잡하고 번거로움 | HTML처럼 작성하면서 필요한 곳에 Java 삽입 |
| 주요 역할 | 비즈니스 로직 처리 (Controller) | 화면 렌더링 (View) |

<br>

## 5. Action Tags
 
스크립틀릿 없이 XML 형태의 태그로 JSP 기능을 사용할 수 있도록 JSP 스펙에 내장된 태그들이다. `<%@ taglib %>` 선언 없이 바로 사용한다.
 
### jsp:include
 
다른 JSP 파일을 현재 페이지에 포함시킨다. directive의 `<%@ include %>`와 동작 방식이 다르다.
 
| 구분 | `<%@ include file="..." %>` | `<jsp:include page="..." />` |
|---|---|---|
| 처리 시점 | 컴파일 전 (정적 결합) | 실행 시 (동적 결합) |
| 서블릿 생성 | 두 JSP가 하나로 합쳐진 뒤 서블릿 1개 생성 | 각각 별도의 서블릿으로 생성, 실행 중 포함 |
| 변수 공유 | 가능 (같은 클래스 내부) | 불가능 (별개의 서블릿) |
| 주요 용도 | 변경이 없는 공통 코드 삽입 | 동적으로 변하는 화면 조각 삽입 |
 
```jsp
<%-- 실행 시마다 동적으로 header.jsp를 불러온다 --%>
<jsp:include page="/fragments/header.jsp" />
```
 
### jsp:useBean
 
JSP에서 자바 객체를 생성하거나 가져올 때 사용한다. 지정한 scope에 이미 해당 객체가 있으면 새로 생성하지 않고 기존 객체를 반환한다.
 
```jsp
<%-- 스크립틀릿 방식 --%>
<% MemberDto m = new MemberDto(); %>
 
<%-- Action Tag 방식 --%>
<jsp:useBean id="m" class="edu.ssafy.dto.MemberDto" scope="request" />
```
 
`scope`는 `page`(기본값), `request`, `session`, `application` 중 하나를 지정한다. 지정한 scope의 `setAttribute`로 저장되는 것과 같다.
 
### jsp:setProperty / jsp:getProperty
 
`useBean`으로 생성한 객체의 프로퍼티(getter/setter)에 접근할 때 사용한다.
 
```jsp
<jsp:useBean id="m" class="edu.ssafy.dto.MemberDto" scope="request" />
<jsp:setProperty property="id" name="m" value="hong" />
<jsp:getProperty property="id" name="m" />
```
 
<br>
 
## 6. isErrorPage
 
에러 처리 전용 JSP를 만들 때 사용하는 page directive 속성이다.
 
```jsp
<%@ page isErrorPage="true" %>
```
 
`isErrorPage="true"` 로 설정하면 `exception` 내장 객체가 생성된다. 이 객체로 발생한 예외 정보를 꺼낼 수 있다.
 
```jsp
<%@ page isErrorPage="true" %>
<p>오류가 발생했습니다: <%= exception.getMessage() %></p>
```
 
에러가 발생한 JSP에서는 `errorPage` 속성으로 에러 페이지를 지정한다.
 
```jsp
<%@ page errorPage="/error/error.jsp" %>
```