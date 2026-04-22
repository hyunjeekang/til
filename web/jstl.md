# JSTL (JSP Standard Tag Library)

JSP에서 자주 사용하는 기능(변수 선언, 조건문, 반복문 등)을 태그 형태로 제공하는 서드파티 라이브러리다. 스크립틀릿을 걷어내고 태그 중심으로 JSP를 작성할 수 있게 해준다.

---

## 1. 사용 준비

### 의존성 추가 (pom.xml 또는 build.gradle)

```xml
<!-- Maven -->
<dependency>
    <groupId>jakarta.servlet.jsp.jstl</groupId>
    <artifactId>jakarta.servlet.jsp.jstl-api</artifactId>
    <version>3.0.0</version>
</dependency>
```

### JSP 파일에서 선언

사용할 JSP 파일 상단에 taglib 지시자를 선언해야 한다.

```jsp
<%@ taglib prefix="c" uri="jakarta.tags.core" %>
```

- `prefix`: 태그를 사용할 때 앞에 붙이는 접두사. 관례적으로 `c`를 사용한다.
- `uri`: 태그 라이브러리를 식별하는 고유 문자열. 라이브러리 로드에 사용된다.

---

## 2. 주요 라이브러리

| 라이브러리 | 주요 기능 | 접두사 | URI |
|---|---|---|---|
| Core | 변수 선언, 조건문, 반복문, URL 처리 | `c` | `jakarta.tags.core` |
| Formatting | 날짜·숫자 형식, 국제화(i18n) | `fmt` | `jakarta.tags.fmt` |
| SQL | DB 쿼리 실행 | `sql` | `jakarta.tags.sql` |
| XML | XML 처리 | `x` | `jakarta.tags.xml` |

실무에서는 Core와 Formatting을 주로 사용한다.

---

## 3. Core 태그

### c:set / c:remove

EL 변수를 선언하거나 삭제한다.

```jsp
<%-- 변수 선언 --%>
<c:set var="name" value="kang" scope="request" />
<c:set var="root" value="${pageContext.request.contextPath}" />

<%-- 태그 바디로 값 지정 --%>
<c:set var="msg" scope="page">
    안녕하세요
</c:set>

<%-- 변수 삭제 --%>
<c:remove var="name" scope="request" />
```

| 속성 | 설명 | 기본값 |
|---|---|---|
| `var` | 변수 이름 | 필수 |
| `value` | 저장할 값 | - |
| `scope` | 저장할 범위 (page/request/session/application) | `page` |

scope를 명시하지 않으면 page scope에 저장된다. EL에서 `${name}` 으로 조회할 때 page부터 탐색하므로 바로 꺼낼 수 있다.

---

### c:if

단순 조건 분기에 사용한다. else에 해당하는 태그가 없으므로, else가 필요하면 `c:choose`를 사용한다.

```jsp
<c:if test="${!empty param.loginUser}">
    ${param.loginUser}님 반갑습니다.
</c:if>

<c:if test="${empty param.loginUser}">
    로그인이 필요합니다.
</c:if>
```

`test` 결과를 변수에 저장해서 재사용할 수 있다.

```jsp
<c:if test="${param.age >= 18}" var="isAdult" />
<c:if test="${isAdult}">성인 인증 완료</c:if>
```

---

### c:choose / c:when / c:otherwise

자바의 if-else if-else 또는 switch에 해당한다.

```jsp
<c:set var="point" value="18500" />
<c:set var="visitCount" value="300" />

<c:choose>
    <c:when test="${point >= 15000 && visitCount >= 100}">VIP</c:when>
    <c:when test="${point >= 10000 || visitCount >= 50}">골드</c:when>
    <c:when test="${point >= 5000}">실버</c:when>
    <c:otherwise>일반 회원</c:otherwise>
</c:choose>
```

---

### c:forEach

배열, List, Map 등 컬렉션을 순회하거나 숫자 범위를 반복할 때 사용한다.

```jsp
<%-- 숫자 범위 반복 --%>
<c:forEach var="i" begin="0" end="10" step="2">
    <c:set var="sum" value="${sum + i}" />
</c:forEach>
<div>0부터 10까지 짝수의 합: ${sum}</div>
```

| 속성 | 설명 |
|---|---|
| `var` | 현재 항목을 담을 변수 이름 |
| `items` | 순회할 컬렉션 |
| `begin` | 시작 인덱스 또는 시작 값 |
| `end` | 종료 인덱스 또는 종료 값 |
| `step` | 증가 단위 (기본값 1) |
| `varStatus` | 반복 상태 정보를 담는 변수 |

#### varStatus 주요 프로퍼티

| 프로퍼티 | 설명 |
|---|---|
| `count` | 1부터 시작하는 반복 횟수 |
| `index` | 0부터 시작하는 인덱스 |
| `first` | 첫 번째 항목이면 true |
| `last` | 마지막 항목이면 true |

```jsp
<%-- 배열 순회 --%>
<ul>
    <c:forEach var="item" items="<%=strings%>">
        <li>${item}</li>
    </c:forEach>
</ul>

<%-- Map 순회 --%>
<c:forEach var="entry" items="${person}" varStatus="status">
    <li>${status.count}번째: ${entry.key} = ${entry.value}</li>
</c:forEach>

<%-- List<Member> 순회 --%>
<c:forEach var="member" items="${members}">
    <tr>
        <td>${member.name}</td>
        <td>${member.email}</td>
    </tr>
</c:forEach>
```

---

### c:forTokens

구분자로 나뉜 문자열을 반복할 때 사용한다.

```jsp
<c:forTokens var="city" items="서울,부산,대구" delims=",">
    <span>${city}</span>
</c:forTokens>
```

---

## 4. EL과 함께 쓰는 패턴

context path를 변수로 선언해두고 링크에서 재사용하는 패턴이다.

```jsp
<c:set var="root" value="${pageContext.request.contextPath}" />

<a href="${root}/member?action=member-regist-form">회원가입</a>
<a href="${root}/auth?action=member-list">회원 목록</a>
```

스크립틀릿 없이 context path를 안전하게 사용할 수 있다.
