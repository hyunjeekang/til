
## 서블릿 필터 & 리스너

프론트 컨트롤러 패턴을 도입하면 각 서블릿에 흩어져 있던 공통 로직을 하나로 모을 수 있다. <br>하지만 프론트 컨트롤러에 도달하기조차 전에 처리해야 하는 전역적인 보안, 인코딩, 로깅 작업이나 애플리케이션 전체의 생명주기를 관리해야 할 때는 어떻게 해야 할까? <br>이때 등장하는 웹 컴포넌트가 **필터(Filter)**와 **리스너(Listener)**다.


### 1. Filter
필터는 클라이언트의 요청/응답이 서블릿(프론트 컨트롤러)/클라이언트에 전달되기 전에 흐름을 가로채어 부가적인 작업을 수행하는 컴포넌트다.

* **주요 용도:** 사용자 인증 및 권한 검사, 전역 로깅 처리, 데이터 인코딩, XSS 방어 등

* **Filter Chain:** 여러 개의 필터를 사슬처럼 연결하여 순차적으로 적용할 수 있다.

* **생명주기:** `init()`(초기화) -> `doFilter()`(실제 필터링 로직) -> `destroy()`(소멸)

---

### 2. 필터 - 요청 로깅 처리

모든 사용자의 요청 URI와 파라미터를 콘솔에 기록해야 한다고 가정해 보자.

**필터 X**
모든 서블릿 혹은 프론트 컨트롤러의 메서드 시작 부분에 아래와 같은 로깅 코드를 일일이 작성해야 한다.

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) {
    // 핵심 로직과 상관없는 로깅 코드가 서블릿을 차지함
    System.out.println("요청 URI: " + request.getRequestURI());
    request.getParameterMap().forEach((k, v) -> {
        System.out.println("파라미터: " + k + " = " + Arrays.toString(v));
    });
    
    // 이후 비즈니스 로직 수행...
}
```

**필터 O**
서블릿은 철저히 비즈니스 로직만 처리하도록 두고, 로깅은 필터로 분리하여 서블릿 실행 전에 처리한다.<br> `@WebFilter("/*")` 어노테이션을 통해 모든 요청에 이 필터를 적용할 수 있다.

```java
@WebFilter(dispatcherTypes = {DispatcherType.REQUEST}, urlPatterns = { "/*" })
public class LoggingFilter extends HttpFilter implements Filter {

    @Override
    protected void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        
        // 1. 서블릿에 도달하기 전 로깅
        System.out.printf("요청 정보 분석 %s, 요청 방식 : %s%n", request.getRequestURI(), request.getMethod());
        request.getParameterMap().forEach((k, v)->{
            System.out.printf("name : %s, v : %s\n", k, Arrays.toString(v));
        });
        
        // 2. 체인의 다음 필터나 최종 서블릿으로 흐름 넘기기
        chain.doFilter(request, response);
    }
}
```

---

### 3. 필터 - XSS 방어

게시판에서 악의적인 사용자가 내용에 `<script>alert('해킹');</script>`를 입력하여 전송한다고 가정해 보자. 

**필터 X**<br>
서블릿은 사용자가 보낸 문자열을 그대로 데이터베이스에 저장하거나 응답 화면에 출력한다.<br> 다른 사용자가 이 글을 읽는 순간 자바스크립트가 실행되어 심각한 보안 사고가 발생한다.<br>

**필터 O**<br>
필터에서 `HttpServletRequest` 파라미터 값을 읽는 `getParameter` 메서드의 동작 방식을 아예 조작해 버릴 수 있다. 이때 사용하는 것이 **Wrapper** 클래스다.

```java
@WebFilter("/*")
public class XssFilter extends HttpFilter implements Filter {

    @Override
    public void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain) 
            throws IOException, ServletException {
        
        // 기존 request 객체 대신 XssRequestWrapper 객체를 서블릿으로 넘겨준다.
        chain.doFilter(new XssRequestWrapper(request), response);
    }
    
    // 내부 클래스: 기존 HttpServletRequest를 래핑해서 기능 확장
    class XssRequestWrapper extends HttpServletRequestWrapper {
        
        public XssRequestWrapper(HttpServletRequest request) {
            super(request);
        }
        
        // getParameter 메서드를 재정의(Override)하여 태그 문자를 무력화
        @Override
        public String getParameter(String name) {
            String org = super.getParameter(name);
            if(org != null) {
                // HTML 태그로 인식되는 괄호를 HTML 엔티티 코드로 변환
                org = org.replace("<", "&lt;").replace(">", "&gt;");
            }
            return org;
        }
    }
}
```
이제 서블릿에서 `request.getParameter("content")`를 호출하면 원본 값이 아닌 안전한 문자열(`&lt;script&gt;...`)이 반환된다. 서블릿 코드를 수정하지 않고 전역적인 보안이 적용된 것이다.

---

### 4. Listener

필터가 클라이언트의 **요청**을 감시한다면, **리스너**는 웹 애플리케이션 내부에서 발생하는 주요 **이벤트**를 모니터링하는 관찰자 객체다. 

**리스너 X**<br>
웹 애플리케이션 구동 시 단 한 번, 엄청나게 무거운 작업(예: 데이터베이스 커넥션 풀 생성, 공통 환경설정 파일 로딩 등)을 미리 해두어야 한다고 가정하자. 리스너가 없다면 특정 서블릿의 `init()` 메서드에 이 코드를 넣어야 하지만, 사용자가 그 서블릿을 호출하기 전까지는 초기화가 이루어지지 않는 문제가 발생한다.<br>

**리스너 O**<br>
`ServletContextListener`를 구현하면 톰캣(WAS)이 웹 애플리케이션을 시작할 때와 종료할 때를 정확히 감지하여 무거운 자원의 초기화와 해제를 깔끔하게 수행할 수 있다.

```java
@WebListener
public class MyContextListener implements ServletContextListener {

    // 1. 웹 애플리케이션이 시작될 때 (가장 먼저 실행됨)
    @Override
    public void contextInitialized(ServletContextEvent sce) { 
         System.out.println("개별 서블릿에서 초기화하기 부담스러운 무거운 자원 초기화");
         // 예: DB 커넥션 풀 생성, 전역 설정 객체 생성 등
    }

    // 2. 웹 애플리케이션이 종료될 때
    @Override
    public void contextDestroyed(ServletContextEvent sce) { 
         System.out.println("init 과정에서 초기화한 자원에 대한 정리 작업");
         // 예: 메모리 누수 방지를 위한 커넥션 풀 종료 등
    }
}
```

* **필터(Filter):** 서블릿 밖에서 **요청과 응답의 흐름**을 가로채어 공통 작업(로깅, 보안)을 수행한다.
* **리스너(Listener):** 애플리케이션의 **생명주기 이벤트(시작/종료 등)**를 감지하여 전역적인 자원을 관리한다.
* 두 기술 모두 핵심 비즈니스 로직(서블릿)에 영향을 주지 않으면서 애플리케이션의 품질을 높이는 수단이다.