# EL (Expression Language)

JSP에서 스크립틀릿(`<% %>`) 없이 데이터를 출력하기 위해 도입된 표현 언어다. 4가지 웹 스코프에 저장된 attribute와 파라미터를 간결하게 꺼내 쓸 수 있다.

```jsp
<%-- 기존 방식 --%>
<%= request.getAttribute("name") %>

<%-- EL 방식 --%>
${name}
```

---

## 1. 기본 문법

```
${표현식}
```

값이 없을 경우 `null` 대신 **빈 문자열("")** 로 출력된다.

---

## 2. 스코프 탐색 순서

`${name}` 처럼 스코프를 명시하지 않으면 좁은 범위부터 순서대로 탐색해서 처음 발견된 값을 사용한다.

```
page  →  request  →  session  →  application
```

더 큰 범위의 값을 명시적으로 가져오려면 스코프를 지정한다.

```jsp
${sessionScope.name}
${requestScope.name}
${applicationScope.name}
${pageScope.name}
```

---

## 3. 내장 객체

| 내장 객체 | 설명 |
|---|---|
| `pageScope` | page scope의 attribute 맵 |
| `requestScope` | request scope의 attribute 맵 |
| `sessionScope` | session scope의 attribute 맵 |
| `applicationScope` | application scope의 attribute 맵 |
| `param` | 요청 파라미터 (`request.getParameter()` 대응) |
| `paramValues` | 파라미터가 여러 값일 때 사용 |
| `header` | 요청 헤더 |
| `headerValues` | 헤더가 여러 값일 때 사용 |
| `cookie` | 쿠키 |
| `pageContext` | PageContext 객체 (context path 등 접근 시 사용) |

```jsp
${param.name}                          <%-- request.getParameter("name") --%>
${header["accept-language"]}           <%-- 헤더 접근 (하이픈 포함 키는 [] 사용) --%>
${pageContext.request.contextPath}     <%-- context path --%>
```

---

## 4. 객체 접근 (프로퍼티)

서블릿에서 객체를 attribute로 넘기면, EL로 getter를 통해 필드 값에 접근할 수 있다.

```java
// 서블릿
request.setAttribute("user", memberObj);
```

```jsp
<%-- 기존 방식 --%>
<%= ((Member)request.getAttribute("user")).getUserName() %>

<%-- EL 방식 --%>
${user.userName}
```

`${user.userName}` 은 `user` 객체의 `getUserName()` 메서드를 호출한다. EL은 `.userName` 을 보고 앞 글자를 대문자로 바꿔 `getUserName()` 을 자동으로 찾아 실행한다.

자바에서 **필드(변수) + Getter/Setter 메서드의 조합**을 프로퍼티(Property)라고 부른다. EL은 이 프로퍼티 규칙을 따른다.

### 접근 방법 두 가지

```jsp
${user.name}       <%-- 점(.) 접근: 프로퍼티 이름으로 접근 --%>
${user["name"]}    <%-- 괄호([]) 접근: 문자열 키로 접근. 동적 키나 특수문자 포함 시 사용 --%>
```

Map 계열 객체는 key로 접근한다.

```jsp
${person.name}         <%-- map.get("name") --%>
${person["name"]}      <%-- 동일 --%>
```

---

## 5. 연산자

### 산술 연산

문자열에 대한 결합 연산(`+`)은 지원하지 않는다. 정수 간 나눗셈도 소수점 결과를 반환한다.

```jsp
${num1 + num2}    <%-- 문자열 "10" + 숫자 20 = 30 (자동 타입 변환) --%>
${num1 / num2}    <%-- 0.5 (소수점 반환) --%>
${num1 div num2}  <%-- 동일 (div 키워드 사용 가능) --%>
${num1 mod num2}  <%-- 나머지 --%>
```

### 비교 연산

문자 데이터는 사전식(lexicographic) 비교를 수행한다.

```jsp
${num1 > 9}        <%-- true --%>
${num1 gt 9}       <%-- 동일 (gt, lt, ge, le, eq, ne 키워드 사용 가능) --%>
```

### 논리 / 삼항 연산

```jsp
${a && b}          <%-- and 키워드도 가능 --%>
${a || b}          <%-- or 키워드도 가능 --%>
${!a}              <%-- not 키워드도 가능 --%>
${a ? "yes" : "no"}
```

### empty 연산자

데이터 존재 여부를 확인하는 단항 연산자다. 아래 경우에 `true`를 반환한다.

- `null`
- 빈 문자열 `""`
- 길이가 0인 배열
- 비어 있는 Collection 객체

```jsp
${empty arr}       <%-- arr가 null이거나 빈 컬렉션이면 true --%>
${!empty param.name}   <%-- name 파라미터가 있으면 true --%>
```

---

## 6. 주의 - EL과 JavaScript 충돌

EL의 `${}` 문법은 자바스크립트 템플릿 리터럴(backtick 문자열)의 `${}` 와 표기가 같다. JSP 파일에서 자바스크립트를 작성할 때 주의가 필요하다.

```jsp
<script>
    // 일반 따옴표 문자열: EL이 먼저 처리됨
    let org = "${org}";   // 서버에서 EL 값으로 치환된 후 브라우저로 전달

    // 백틱 문자열: 백틱은 JSP/EL이 처리하지 않음
    let msg = `I am in ${org}`;  // 브라우저에서 JS 템플릿 리터럴로 처리
                                  // 단, 이 시점의 ${org}는 JS 변수 org를 참조
</script>
```

EL 값을 JS 변수에 담고, 그 변수를 백틱 안에서 사용하는 패턴이 안전하다.
