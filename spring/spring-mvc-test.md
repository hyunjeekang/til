# Spring MVC 테스트 & 주요 기능 정리

## 1. Spring MVC 테스트 방식

### 기존 방식 vs Mock 방식

**기존 방식**: 대상을 직접 호출

```
Member, Email → JUnit → Controller
HttpServletRequest, HttpServletResponse → Controller
```

**Mock 방식**: WAS에서 Spring MVC와 소통하는 부분을 가상으로 제공

```
JUnit ──MockRequest/MockResponse──▶ MockMvc ◀──HttpServletRequest/HttpServletResponse── @Controller
```

---

## 2. MockMvc 테스트 구성

### 주요 어노테이션

| 어노테이션 | 역할 |
|-----------|------|
| `@WebMvcTest` | Controller만 로드하는 Web Slice 테스트. Service 등 다른 Bean은 로드하지 않음 |
| `@MockitoBean` | 실제 Bean 대신 Mockito가 만든 가짜 객체를 주입. stub 설정이 없으면 기본값(null, 0 등) 반환 |

```java
@WebMvcTest(value = SomeController.class)
class SomeControllerTest {

    @Autowired
    MockMvc mockMvc;  // 가상 웹 환경 객체

    @MockitoBean
    SomeService service;  // 가짜 서비스 객체

    @BeforeEach
    void setUp() {
        // stub 설정: 특정 인자로 호출 시 반환값 지정
        when(service.echo("Hello")).thenReturn("echo: Hello");
    }
}
```

> **@MockitoBean 주의**: stub 설정이 없는 메서드는 null을 반환한다.  
> 인자를 특정 인스턴스로 매칭하기 어려울 때는 `any()` 사용.
> ```java
> when(service.upload(any(), any())).thenReturn(Map.of("desc", "설명", ...));
> ```

---

### 테스트 3단계: Request → Perform → Expect

#### 1단계: Request 빌드 (`MockMvcRequestBuilders`)

`get()`, `post()`, `put()`, `delete()`, `multipart()` 등 HTTP 메서드별 정적 메서드 제공.

```java
MockMvcRequestBuilders.get("/some/path")
    .param("key", "value")      // 쿼리 파라미터
    .header("Authorization", "Bearer xxx")
    .cookie(new Cookie("name", "hong"))
    .sessionAttr("user", userObj)
    .flashAttr("msg", "success")
    .contentType(MediaType.APPLICATION_JSON)
    .content("{\"name\":\"hong\"}")
```

#### 2단계: Perform

```java
ResultActions actions = mockMvc.perform(request);
```

#### 3단계: Expect (`andExpect`) — JUnit의 assertion 역할

```java
actions
    .andExpect(status().isOk())               // HTTP 상태 코드
    .andExpect(view().name("index"))           // 뷰 이름
    .andExpect(model().attribute("key", val)) // 모델 attribute
    .andExpect(content().contentType(MediaType.APPLICATION_JSON))
    .andExpect(jsonPath("$.name", equalTo("hong")))  // JSON 값 검증
    .andExpect(redirectedUrl("/"))            // 리다이렉트 URL
    .andExpect(flash().attribute("key", val)) // flash attribute
    .andExpect(request().sessionAttribute("key", val)) // 세션 attribute
    .andExpect(handler().methodName("echo")); // 핸들러 메서드명
```

---

### @ResponseBody 테스트 (JSON 응답)

`contentType`이 `application/json`인 응답은 `jsonPath`로 검증한다.

```java
.andExpect(content().contentType(MediaType.APPLICATION_JSON))
.andExpect(jsonPath("$.name", equalTo("hong")))
.andExpect(jsonPath("$.hobby[2]", equalTo("reading")))
```

---

## 3. Controller 주요 어노테이션

### @RequestParam

`HttpServletRequest`의 파라미터를 메서드 인자로 자동 바인딩. 자동 형변환 지원.

```java
@GetMapping("/add")
String add(@RequestParam int a, @RequestParam int b, Model model) {
    model.addAttribute("result", a + b);
    return "index";
}
```

---

### @ModelAttribute

파라미터들을 DTO의 프로퍼티 이름 기반으로 자동 연결해 객체를 생성한다.

**일반 클래스** 처리 순서:
1. 기본 생성자로 객체 생성
2. setter를 통해 프로퍼티 설정
3. 모델에 attribute 자동 추가 (클래스 이름의 camelCase를 키로 사용)

**record** 처리: 생성자 인자와 파라미터 이름을 매칭해 바로 객체 생성.

```java
@PostMapping("/regist")
String regist(@ModelAttribute Member member, HttpSession session) {
    session.setAttribute("member", member);
    return "redirect:/";
}
```

---

### @CookieValue

쿠키의 name에 매핑된 value를 메서드 인자로 바인딩. `@RequestParam`처럼 자동 형변환 지원.

```java
@GetMapping("/cookie")
@ResponseBody
Map<String, Object> cookie(@CookieValue String name, @CookieValue int age) {
    return Map.of("name", name, "age", age);
}
```

---

## 4. Scope: Request / Flash / Session

| Scope | 유효 범위 |
|-------|----------|
| Request | 현재 요청 1회 |
| Flash | Redirect가 완료될 때까지 (세션에 임시 저장 → redirect 후 request로 이동, 세션에서는 삭제) |
| Session | 세션 유지 기간 전체 |

**수명: request < flash < session**

```java
@GetMapping("/redir-attr")
String redirAttr(Model model, HttpSession session, RedirectAttributes redir) {
    model.addAttribute("byModel", "value");         // request scope
    session.setAttribute("bySession", "value");     // session scope
    redir.addAttribute("byParam", "value");         // redirect param (URL에 붙음)
    redir.addFlashAttribute("byFlash", "value");    // flash scope
    return "redirect:/";
}
```

---

## 5. 예외 처리

### @ControllerAdvice + @ExceptionHandler

예외 처리 우선순위:

```
DispatcherServlet
  ├─ HandlerMapping
  ├─ HandlerAdapter → Controller → @ControllerAdvice  (1순위)
  └─ BasicErrorController                              (2순위, /error)
```

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(RuntimeException.class)
    public String handleRuntimeException(RuntimeException e, Model model) {
        model.addAttribute("error", e.getMessage());
        return "error";
    }
}
```

> `@ControllerAdvice(assignableTypes = SomeController.class)`로 특정 Controller만 대상으로 지정할 수도 있다.

---

## 6. 파일 업로드

### MultipartFile

웹 앱에서 파일 업로드 시 사용하는 객체. 파일은 대용량이므로 POST + `enctype="multipart/form-data"` 필수.

```html
<form method="post" enctype="multipart/form-data" action="/upload">
    <input type="file" name="file">
    <button>업로드</button>
</form>
```

```java
@PostMapping("/upload")
String upload(@RequestParam MultipartFile file, @RequestParam String desc, Model model)
        throws IOException {
    String fileName = file.getOriginalFilename();
    file.transferTo(new File(savePath, fileName));  // 서버 로컬에 저장
    model.addAttribute("fileName", fileName);
    return "index";
}
```

---

### 정적 리소스 매핑 (WebMvcConfigurer)

업로드된 파일을 클라이언트에서 URL로 접근하려면 가상 경로 → 물리 경로 매핑이 필요하다.

```java
@Configuration
public class MvcConfig implements WebMvcConfigurer {

    @Value("${spring.servlet.multipart.location}")
    String uploadPath;

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/upload/**")
                .addResourceLocations("file:" + uploadPath + "/");
    }
}
```

- 클라이언트: `/upload/image.png` 요청
- 서버: `uploadPath/image.png` 파일 반환

---

### 파일 다운로드 — `<a download>` 속성

```html
<a href="/upload/image.png" download>다운로드</a>
```

---

### Base64 인코딩 (Ajax 전송 시)

파일을 Ajax로 전달할 때 바이너리를 문자열로 변환해 전송.

```java
String mimeType = file.getContentType();
String base64 = "data:%s;base64,%s".formatted(
    mimeType,
    Base64.getEncoder().encodeToString(file.getBytes())
);
```

```html
<!-- 이미지 미리보기 -->
<img src="data:image/png;base64,iVBORw0KGgo...">
```
