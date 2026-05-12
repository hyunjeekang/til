# Spring MVC

## 구조도

```
                         Spring MVC 요청 처리 흐름
┌────────┐
│ Client │
└───┬────┘
    │ ① HTTP Request
    ▼
┌─────────────────────────────────────────────────────┐
│                  DispatcherServlet                  │  ← Front Controller
│              (모든 요청의 단일 진입점)                │
└──┬──────────────────────────────────────────────────┘
   │                          ▲
   │ ② 핸들러 조회             │ ③ Handler 반환
   ▼                          │
┌──────────────────┐          │
│  HandlerMapping  │──────────┘
│  (URL → Handler  │
│   매핑 테이블)    │
└──────────────────┘

   │ ④ 처리 위임
   ▼
┌──────────────────┐
│  HandlerAdapter  │  ← Handler를 실제로 호출하는 중간 다리
└──┬───────────────┘
   │ ⑤ 요청 처리 요청          ▲
   ▼                          │ ⑥ ModelAndView 반환
┌──────────────────────────────────────────────────────┐
│              @Controller (Handler)                   │
│  - 파라미터 분석 (@RequestParam 등)                   │
│  - 비즈니스 로직 호출 (Service 연결)                  │
│  - 결과를 Model에 저장                               │
│  - view 이름(문자열) 리턴                            │
└──────────────────────┬───────────────────────────────┘
                       │
             ┌─────────┘
             │  Service 호출
             ▼
       ┌───────────┐
       │  Service  │  ← 비즈니스 로직 담당
       └───────────┘

   │ ⑦ view 이름 전달
   ▼
┌──────────────────┐
│   ViewResolver   │  ← view 이름 → 실제 파일 경로로 변환
│  prefix + name   │    ex) "index" → /WEB-INF/views/index.jsp
│       + suffix   │
└──┬───────────────┘
   │ ⑧ View 반환
   ▼
┌──────────────────┐
│    View (JSP)    │  ← Model 데이터 참조하여 HTML 생성
└──┬───────────────┘
   │ ⑨ HTTP Response
   ▼
┌────────┐
│ Client │
└────────┘
```

---

## Infrastructure Components

| 컴포넌트 | 역할 | 차이점 |
|---|---|---|
| **HandlerMapping** | URL을 보고 "어떤 핸들러(컨트롤러)가 처리할지" 찾아서 반환 | **찾는** 역할 |
| **HandlerAdapter** | 찾아낸 핸들러를 실제로 **호출**하는 역할 | **실행**하는 역할 |
| **ViewResolver** | 컨트롤러가 반환한 view 이름 문자열을 실제 View 파일로 변환 | |

---

## 프로젝트 구조 (Spring MVC + JSP)

```
src/
├── main/
│   ├── java/com/
│   │   ├── Application.java       ← Spring Boot 진입점
│   │   ├── Controller.java         ← Handler (@Controller)
│   │   └── model/service/
│   │       └── Service.java        ← 비즈니스 로직
│   │
│   ├── resources/
│   │   ├── static/                       ← 정적 리소스 (css, js, img, html)
│   │   │   └── common.css
│   │   ├── templates/                    ← Thymeleaf/Mustache 템플릿 (미사용 시 비워둠)
│   │   └── application.properties        ← ViewResolver 설정 등
│   │
│   └── webapp/WEB-INF/
│       └── views/
│           └── index.jsp                 ← JSP 뷰 파일
```

> JSP는 `src/main/resources/templates`가 아닌  
> `src/main/webapp/WEB-INF/views/` 에 위치해야 한다.  
> (보안상 WEB-INF 하위는 직접 URL 접근 불가 → 반드시 Controller 통해서만 접근)

---

## ViewResolver 설정

`application.properties`에서 빈(Bean) 직접 등록 없이 설정 가능.

```properties
# application.properties
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

view 이름 `"index"` 반환 시 → `/WEB-INF/views/` + `index` + `.jsp` = `/WEB-INF/views/index.jsp` 로 포워드

---

## 주요 어노테이션

### @Controller

Handler 역할. 클라이언트 요청을 받아들이고 처리 결과를 반환한다.  
하나의 클래스 안에 여러 개의 요청 처리 메서드를 작성할 수 있다.

```java
@Controller
@RequestMapping("/simple")
@RequiredArgsConstructor
public class SimpleController {

    private final SimpleService service;  // @RequiredArgsConstructor → 생성자 주입

    // ...
}
```

---

### @RequestMapping

URL과 메서드를 연결하는 어노테이션이다.

| 위치 | 의미 |
|---|---|
| **클래스** 레벨 | 소속된 모든 메서드의 URL prefix |
| **메서드** 레벨 | 해당 메서드에만 적용되는 URL |

```java
@Controller
@RequestMapping("/simple")         // 모든 메서드의 prefix: /simple
public class SimpleController {

    @GetMapping                    // GET /simple
    public String home() {
        return "index";
    }

    @GetMapping("/echo")           // GET /simple/echo
    public String echo(...) { ... }

    @GetMapping("/add")            // GET /simple/add
    public String add(...) { ... }
}
```

HTTP 메서드 지정 방법:
```java
// 풀 표기
@RequestMapping(value = "/echo", method = RequestMethod.GET)

// 축약형 (권장)
@GetMapping("/echo")
@PostMapping("/echo")
```

---

### @RequestParam

쿼리 파라미터(`?key=value`)를 메서드 파라미터로 바인딩한다.  
파라미터 이름이 같으면 자동 매핑, primitive/wrapper/String 타입 모두 사용 가능.

```java
// GET /simple/echo?msg=hello
@GetMapping("/echo")
public String echo(@RequestParam String msg, Model model) {
    String result = service.echo(msg);
    model.addAttribute("result", result);
    return "index";
}

// GET /simple/add?a=3&b=5
@GetMapping("/add")
public String add(@RequestParam int a, @RequestParam int b, Model model) {
    int result = service.add(a, b);
    model.addAttribute("result", result);
    return "index";
}
```

---

### Model

컨트롤러 → View로 데이터를 전달할 때 사용한다.  
`model.addAttribute("키", 값)` 으로 저장 → JSP에서 `${키}` 로 참조.

```jsp
<%-- index.jsp --%>
<h2>결과: ${result}</h2>
```

---

## 요청 처리 메서드 반환 형태

### 1. Forward (논리적 view 이름 반환)

컨트롤러가 view 이름 문자열을 반환 → ViewResolver가 실제 파일 경로로 변환 → 포워드.  
URL은 바뀌지 않는다.

```java
@GetMapping
public String home() {
    return "index";  // /WEB-INF/views/index.jsp 로 포워드
}
```

### 2. Redirect

view 이름 앞에 `redirect:` 접두사를 붙이면 클라이언트에게 리다이렉트 응답(302)을 보낸다.  
URL이 변경되며, 상대/절대 경로 및 외부 URL 모두 가능.

```java
@GetMapping("redirect")
public String redirect() {
    return "redirect:/simple";      // 절대경로 리다이렉트
    // return "redirect:../other";  // 상대경로
    // return "redirect:https://www.naver.com";  // 외부 URL
}
```

| | Forward | Redirect |
|---|---|---|
| 브라우저 URL | 변경 안 됨 | 변경됨 |
| HTTP 상태 코드 | 200 | 302 |
| Model 데이터 | JSP에서 사용 가능 | 전달 안 됨 (새 요청) |
| 사용 시점 | 결과 화면 출력 | 폼 중복 제출 방지, 다른 페이지 이동 |

### 3. @ResponseBody (JSON 응답)

View를 거치지 않고 반환 값을 HTTP 응답 body에 직접 직렬화.  
내부적으로 Jackson(`ObjectMapper`)이 Java 객체 → JSON 변환을 담당한다.

```java
// GET /simple/json
// 응답: {"name":"hong","pass":"1234","age":10}
@GetMapping("/json")
public @ResponseBody Map<String, Object> json() {
    return Map.of("name", "hong", "pass", "1234", "age", 10);
}
```

> `@RestController` = `@Controller` + 모든 메서드에 `@ResponseBody` 적용  
> REST API 전용 컨트롤러를 만들 때는 `@RestController`를 사용한다.

---

## Service 계층

비즈니스 로직은 Controller가 아닌 Service에서 처리.  
Controller는 요청/응답 흐름만 담당하고, 실제 로직은 Service에 위임.

```java
@Service
public class SimpleService {

    public String echo(String msg) {
        return "echo: " + msg;
    }

    public int add(int a, int b) {
        return a + b;
    }
}
```

```java
// Controller에서 Service 주입 (생성자 주입 방식)
@RequiredArgsConstructor
public class SimpleController {
    private final SimpleService service;  // Lombok이 생성자 자동 생성

    @GetMapping("/echo")
    public String echo(@RequestParam String msg, Model model) {
        String result = service.echo(msg);   // Service에 위임
        model.addAttribute("result", result);
        return "index";
    }
}
```

---