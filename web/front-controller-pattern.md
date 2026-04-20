## 2. 서블릿 매핑의 발전 과정과 프론트 컨트롤러 패턴

### 1. 서블릿 매핑 방식의 발전
**서블릿 매핑** : 클라이언트가 특정 주소로 접속했을 때 어떤 서블릿을 실행할지 연결해 주는 작업 <br>
과거 웹 프로젝트는 톰캣의 설정 파일인 `web.xml`에서 이러한 매핑을 관리했다.

**Servlet 3.0 이전: web.xml을 이용한 매핑**<br>
새로운 서블릿을 만들 때마다 XML 파일에 긴 설정 코드를 직접 추가해야 했다. 프로젝트의 규모가 커질수록 설정 파일이 비대해지고 유지보수가 어려워지는 단점이 있었다.

```xml
<servlet>
    <servlet-name>memberServlet</servlet-name>
    <servlet-class>edu.ssafy.controller.MemberServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>memberServlet</servlet-name>
    <url-pattern>/member</url-pattern>
</servlet-mapping>
```

**Servlet 3.0 이후: 어노테이션 기반 매핑**
현재는 자바 클래스 선언부 상단에 `@WebServlet` 어노테이션을 추가하는 것만으로 매핑이 완료된다.

```java
@WebServlet("/member")
public class MemberServlet extends HttpServlet {
    // 서블릿 로직
}
```

### 2. 프론트 컨트롤러 패턴

초기 웹 개발 방식은 기능마다 별도의 서블릿 파일을 개별적으로 생성했다. 이 방식은 문자 인코딩 설정이나 보안 권한 검증 같은 공통 로직이 모든 서블릿 파일에 중복으로 작성되는 문제점이 있었다.

이를 해결하기 위해 등장한 아키텍처가 **프론트 컨트롤러 패턴**이다. 

**프론트 컨트롤러 도입 시 구조 변화**<br>
단 하나의 대표 서블릿이 클라이언트의 모든 요청을 가장 먼저 전달받는다. <br>이 진입점에서 공통적인 작업을 단 한 번만 수행한 뒤, 요청 파라미터 값을 분석하여 알맞은 세부 비즈니스 로직으로 흐름을 분기한다.

<img src = "https://velog.velcdn.com/images/pp8817/post/6d20fb62-754d-45a2-a617-078b010d2bfe/image.png">

```java
@WebServlet("/member")
public class FrontControllerServlet extends HttpServlet {
    
    protected void service(HttpServletRequest req, HttpServletResponse resp) {
        // 전역 인코딩 및 응답 형식 등 공통 로직을 한 번만 처리한다.
        req.setCharacterEncoding("UTF-8"); 
        resp.setContentType("text/html; charset=utf-8");
        
        // 클라이언트의 요청 목적을 파라미터로 식별한다.
        String action = req.getParameter("action");
        
        // 목적에 맞게 세부 로직으로 흐름을 라우팅한다.
        if ("insert".equals(action)) {
            memberInsert(req, resp); 
        } else if ("select".equals(action)) {
            memberSelect(req, resp); 
        }
    }
    
    private void memberInsert(HttpServletRequest req, HttpServletResponse resp) {
        // 회원 가입 비즈니스 로직
    }
}
```

### 3. 스프링 프레임워크와 디스패처 서블릿
개발자가 직접 프론트 컨트롤러를 구현할 경우 조건문이 무한정 길어지는 한계가 존재한다. 이에 따라 스프링 프레임워크는 완성형 프론트 컨트롤러인 **디스패처 서블릿**을 핵심 컴포넌트로 제공한다.

스프링 프레임워크는 단순히 서블릿 규격만 지원하는 것을 넘어, 애플리케이션을 구성하는 모든 클래스 객체를 스프링 컨테이너라는 공간에 담아 직접 관리하고 주입한다. 모든 요청은 디스패처 서블릿이라는 단일 통로를 거쳐 알맞은 하위 컨트롤러에 자동 배분되며, 개발자는 복잡한 라우팅 로직 대신 핵심 비즈니스 로직 구현에만 집중할 수 있게 된다.
