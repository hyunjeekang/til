## 1. 웹 서버와 WAS, 서블릿의 생명주기

### 1. 웹 서버와 웹 애플리케이션 서버의 역할 분담
클라이언트의 요청을 처리하는 서버는 정적 데이터 처리와 동적 데이터 처리로 역할을 나누어 분담한다.

* **웹 서버**
    * 아파치, 엔진엑스 등
    * 미리 저장된 HTML, CSS, 이미지 같은 정적인 자원을 즉각적으로 반환하는 역할을 수행한다. 
    * 이 핵심 모듈을 Http Demon (Httpd) 이라 부른다.<br><br>

  <img src = "https://miro.medium.com/v2/resize:fit:720/format:webp/1*FvZY_VP3QdqcKbBsZZ4dCA.jpeg"> <br><br>

* **웹 애플리케이션 서버 (WAS)**
    * 톰캣
    * 데이터베이스 조회, 비즈니스 로직 연산 등 웹 서버가 단독으로 처리할 수 없는 동적인 요청을 처리한다.

* **처리 흐름**
    * 클라이언트가 요청을 보내면 웹 서버가 먼저 분석하고, 동적인 처리가 필요한 경우 데이터를 WAS에 전달하여 연산을 수행한 뒤 결과를 반환한다.

  <img src = "https://gmlwjd9405.github.io/images/web/static-vs-dynamic.png"><br><img src="https://gmlwjd9405.github.io/images/web/webserver-vs-was1.png"><br><br>

### 2. 서블릿과 서블릿 컨테이너

**서블릿 & 서블릿 컨테이너**
- **서블릿** : 자바 기반의 웹 환경에서 동적인 페이지를 생성하기 위한 표준 기술

- **서블릿 컨테이너** : 서블릿 객체의 생성부터 소멸까지 전체 생명주기를 전담하여 관리하는 시스템 엔진 <br><br>


**제어의 역전 원칙**<br>
- 일반적인 자바 프로그램은 개발자가 작성한 `main` 함수부터 실행된다. 반면 백엔드 웹 애플리케이션에는 `main` 함수가 존재하지 않는다. 

- 실제 프로그램의 흐름을 제어하는 주체는 **서블릿 컨테이너 내부**에 존재하며, 컨테이너가 개발자가 작성한 클래스를 직접 생성하고 실행한다. 

- 이처럼 프로그램의 주도권이 시스템에 있는 것을 제어의 역전이라 한다. <br><br>

### 3. 서블릿 생명주기

  <img src = "https://media.geeksforgeeks.org/wp-content/uploads/20250205142106489023/State-of-sevlet-life-cycle.webp"><br>

서블릿 컨테이너는 특정 서블릿 객체를 단 하나만 생성하는 방식으로 관리한다. <br>
이 서블릿의 생명주기는 크게 네 가지 단계로 구성된다.

**1 서블릿 로딩 및 인스턴스 생성**
- **로딩**
  - 서블릿 컨테이너가 서블릿 클래스를 메모리에 로드한다.
  - 애플리케이션 시작 또는 첫 번째 클라이언트 요청 시 로드된다.

- **인스턴스 생성**: 컨테이너는 매개변수가 없는 생성자를 사용하여 서블릿의 인스턴스를 생성한다.<br><br>


**2 서블릿 초기화**
- 인스턴스 생성이 완료되면 컨테이너는 서블릿을 초기화한다. (`init`)

- 초기화 단계는 서블릿의 수명 주기 동안 단 한 번만 실행된다.

- 이때 서블릿이 요청을 처리할 수 있도록 준비 작업을 수행한다. <br><br>


**3 요청 처리**

- 요청 및 응답 객체
  - 각 요청에 대해 `ServletRequest` 및 `ServletResponse` 객체가 생성된다.
  - HTTP의 경우 `HttpServletRequest` 및 `HttpServletResponse` 객체가 사용된다.

- `service()` 메서드
  - 컨테이너는 각 요청에 대해 `service(ServletRequest req, ServletResponse res)`를 호출한다.
  - HTTP 요청 유형(GET, POST, PUT, DELETE)을 결정한다.
  - 요청을 `doGet()`, `doPost()` 등과 같은 적절한 메서드에 위임한다.<br><br>

**4 서블릿 삭제**
- 서블릿 컨테이너가 서블릿을 제거하기로 결정하면 자원 정리를 시작한다.
- 컨테이너는 `service()` 메서드를 실행하는 모든 스레드가 작업을 완료하도록 대기한다.
- 컨테이너는 서블릿이 리소스를 해제할 수 있도록 `destroy()` 메서드를 호출한다.
- `destroy()`가 실행된 후, 서블릿 컨테이너는 서블릿 인스턴스에 대한 모든 참조를 해제하여 가비지 컬렉션 대상이 되도록 한다.<br><br>

### 4. 서블릿 생명주기 메서드

* **init() 메서드**
    * 서블릿이 인스턴스화될 때 자원 초기화를 위해 1회 호출된다. 
    * 일반적인 자바 생성자는 `ServletException`과 같은 서블릿 전용 예외를 발생시킬 수 없기 때문에, 안전한 예외 처리를 위해 초기화 작업은 반드시 `init()` 메서드를 사용해야 한다.

* **service() 메서드**
    * 클라이언트와 서버를 연결하며 실제 요청과 응답을 처리한다.
    *  컨테이너에 의해 호출되며, 내부적으로 HTTP 요청 방식을 판단하여 `doGet()`, `doPost()` 등의 세부 메서드로 작업을 위임한다.

* **destroy() 메서드**
    * 서블릿의 수명 주기 마지막에 1회 호출되며, 사용했던 리소스를 안전하게 정리하는 역할을 수행한다.

**자바 서블릿 구현 예제**

```java
import jakarta.servlet.*;
import jakarta.servlet.http.*;
import java.io.*;

public class AdvanceJavaConcepts extends HttpServlet {
    private String output;

    @Override
    public void init() throws ServletException {
        // 서블릿이 처음 로드될 때 출력 변수를 초기화
        output = "Servlet Concepts";
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) 
        throws ServletException, IOException {
        
        resp.setContentType("text/html");
        
        // 클라이언트의 GET 요청에 응답하고 결과를 HTML 형식으로 출력
        try (PrintWriter out = resp.getWriter()) {
            out.println(output); 
        }
    }

    @Override
    public void destroy() {
        // 서블릿이 제거되기 전 자원 해제 및 정리 작업을 수행
        System.out.println("Servlet destroyed");
    }
}
```

위 코드에서 `output` 변수는 `init()` 단계에서 한 번만 초기화되고 이후에는 값이 변경되지 않는다. 여러 스레드가 동시에 `doGet()`에 접근하더라도 값을 읽기만 하므로 멀티 스레드 환경에서 안전하게 동작하게 된다.

