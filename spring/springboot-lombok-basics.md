# Spring Boot

## 목차

1. [Lombok](#1-lombok)
2. [Builder 패턴](#2-builder-패턴)
3. [Spring에서 빈으로 관리할 대상들](#3-spring에서-빈으로-관리하는-대상)
4. [Spring Boot 주요 어노테이션](#4-spring-boot-주요-어노테이션)
5. [Bean Scope](#5-bean-scope)
6. [Blank Final](#6-blank-final)

---

## 1. Lombok

### 개념

Lombok은 반복적인 코드(getter, setter, 생성자, toString 등)를 어노테이션으로 자동 생성해주는 라이브러리다. 컴파일 타임에 바이트코드를 생성하므로 런타임 오버헤드가 없다.

### 의존성 추가 (pom.xml)

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

### 주요 어노테이션

#### `@Getter` / `@Setter`

모든 필드에 대한 getter/setter를 자동 생성한다.

```java
// 실습 코드: Page.java
@Getter
@Setter
@ToString
public class Page<T> {
    private int currentPage;
    private int listSize;
    private int pageSize;
    // getter, setter, toString 자동 생성
}
```

#### `@Data`

`@Getter + @Setter + @ToString + @EqualsAndHashCode + @RequiredArgsConstructor`를 한 번에 적용하는 복합 어노테이션.

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Address {
    private int ano;
    private String street;
    private String zipcode;
    private int mno;
}
```

**사용 전 코드**
```java
// 수동 작성 코드
public class Address {
    private int ano;
    private String street;
    // ...
    public int getAno() { return ano; }
    public void setAno(int ano) { this.ano = ano; }
    public String getStreet() { return street; }
    // ... 반복
    @Override
    public String toString() { return "Address [ano=" + ano + " ...]"; }
}
```

#### `@NoArgsConstructor` / `@AllArgsConstructor`

- `@NoArgsConstructor`: 파라미터 없는 기본 생성자 생성
- `@AllArgsConstructor`: 모든 필드를 파라미터로 받는 생성자 생성

```java
@NoArgsConstructor   // Address()
@AllArgsConstructor  // Address(int ano, String street, String zipcode, int mno)
public class Address { ... }
```

#### `@RequiredArgsConstructor`

`final` 필드 또는 `@NonNull` 필드만을 파라미터로 받는 생성자를 자동 생성한다. **의존성 주입에서 핵심적으로 활용된다.**

```java
@Service
@RequiredArgsConstructor
public class BasicMemberService implements MemberService {
    private final MemberDao mDao;    // final → 생성자 주입 대상
    private final AddressDao aDao;   // final → 생성자 주입 대상
    private final DBUtil util;       // final → 생성자 주입 대상
    // 자동 생성: BasicMemberService(MemberDao mDao, AddressDao aDao, DBUtil util)
}
```

#### `@NonNull`

필드 또는 파라미터에 `null` 값이 들어오면 `NullPointerException`을 던지는 null 체크 코드를 자동 삽입한다.

```java
@Data
@RequiredArgsConstructor
public class Member {
    private int mno;
    private @NonNull String name;     // null 허용 안 함
    private @NonNull String email;    // null 허용 안 함
    private @NonNull String password; // null 허용 안 함
    private String role;              // null 허용
}
```

`@RequiredArgsConstructor`와 함께 사용하면 `@NonNull` 필드도 생성자 파라미터로 포함된다.

#### `@ToString.Exclude` / `@EqualsAndHashCode.Exclude`

`@Data`가 자동 생성하는 `toString()` 또는 `equals()/hashCode()`에서 특정 필드를 제외한다.

```java
@Data
public class Member {
    private int mno;
    private String name;

    @ToString.Exclude           // toString()에서 제외 → 순환 참조 방지
    @EqualsAndHashCode.Exclude  // equals/hashCode에서 제외 → 동일성 비교 기준 제외
    private List<Address> addresses = new ArrayList<>();
}
```

> 양방향 연관관계에서 `@ToString`을 그대로 쓰면 무한 순환 참조가 발생할 수 있으므로 반드시 제외해야 한다.

#### `@Slf4j`

`private static final Logger log = LoggerFactory.getLogger(...)` 선언을 자동 생성한다.

```java
@SpringBootTest
@Slf4j
public class BeanCreateTest {
    @Test
    void beanTest() {
        log.info("테스트 시작"); // Lombok이 log 필드를 자동 생성
    }
}
```

---

## 2. Builder 패턴

### 개념

객체 생성 시 파라미터가 많을 경우, 생성자 대신 메서드 체이닝 방식으로 가독성 있게 객체를 생성하는 패턴이다.

### Lombok `@Builder`

`@Builder` 어노테이션을 클래스에 붙이면 내부 `Builder` 정적 클래스와 `builder()` 메서드를 자동으로 생성한다.

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Address {
    private int ano;
    private String street;
    private String zipcode;
    private int mno;
}
```

**사용 예시:**
```java
// 생성자 사용 (파라미터 순서를 알아야 함 -> 가독성 낮음)
Address addr1 = new Address(1, "서울시 강남구", "06000", 10);

// Builder 사용 (필드 이름 명시 -> 가독성 높아짐)
Address addr2 = Address.builder()
    .ano(1)
    .street("서울시 강남구")
    .zipcode("06000")
    .mno(10)
    .build();
```

### 생성자에만 `@Builder` 적용

클래스 전체가 아닌 특정 생성자에만 Builder를 적용할 수 있다.

```java
@Data
@NoArgsConstructor
public class SearchCondition {
    private String key;
    private String word;
    private int currentPage;

    @Builder  // 이 생성자에만 Builder 패턴 적용
    public SearchCondition(String key, String word, int currentPage) {
        this.key = key;
        this.word = word;
        this.currentPage = currentPage;
    }
}
```

### `@Builder`와 `@NoArgsConstructor` 함께 사용 시 주의사항

`@Builder`는 내부적으로 `@AllArgsConstructor`를 생성하는데, `@NoArgsConstructor`와 충돌할 수 있다. 이 경우 `@AllArgsConstructor`를 명시적으로 함께 선언해야 한다.

```java
@Data
@NoArgsConstructor      // 기본 생성자
@AllArgsConstructor     // 전체 생성자 (Builder 내부에서 사용)
@Builder                // 빌더 패턴
public class Address { ... }
```

---

## 3. Spring에서 빈으로 관리하는 대상

### Spring Bean이란?

Spring IoC 컨테이너가 생성하고 관리하는 객체를 **빈(Bean)** 이라 한다. 개발자가 `new`로 직접 생성하는 대신, 컨테이너가 객체의 생명주기를 관리한다.

### 수동 싱글톤 패턴 vs Spring Boot 적용 비교

#### 수동 싱글톤 패턴

```java
public class BasicMemberService implements MemberService {
    // 직접 싱글톤 구현
    private static BasicMemberService service = new BasicMemberService();
    public static BasicMemberService getService() { return service; }
    
    // 의존 객체도 직접 가져옴
    private MemberDao mDao = BasicMemberDao.getDao();
    private AddressDao aDao = BasicAddressDao.getDao();
    private DBUtil util = DBUtil.getUtil();
}

// 사용 측 (AuthController.java)
private final MemberService mService = BasicMemberService.getService();
```

**문제점:**
- 객체 간 결합도가 높다 (구체 클래스에 의존)
- 테스트 시 의존 객체를 교체하기 어렵다
- 싱글톤 구현을 매 클래스마다 반복해야 한다

#### Spring Boot: IoC 컨테이너 위임

```java
@Service // Bean으로 관리 
@RequiredArgsConstructor
public class BasicMemberService implements MemberService {
    private final MemberDao mDao;    // 인터페이스에 의존
    private final AddressDao aDao;
    private final DBUtil util;
    // Spring이 알아서 구현체를 찾아 주입
}
```
### 의존성 주입 DI

#### 생성자 주입

```java
// @RequiredArgsConstructor가 생성자를 자동 생성하므로 @Autowired 생략 가능
@Service
@RequiredArgsConstructor
public class BasicMemberService {
    private final MemberDao mDao;    // final → 불변 보장
    private final AddressDao aDao;
    private final DBUtil util;
}
```

> 생성자 주입이 권장되는 이유: final 선언으로 불변성 보장, 순환 의존성 컴파일 타임 감지, 테스트 용이성

---

## 4. Spring Boot 주요 어노테이션

### `@SpringBootApplication`

Spring Boot 애플리케이션의 진입점 클래스에 선언한다. 아래 세 어노테이션의 복합체다.

```java
@SpringBootApplication
public class BeOnline0507SpringBootApplication {
    public static void main(String[] args) {
        SpringApplication.run(BeOnline0507SpringBootApplication.class, args);
    }
}
```

| 내포된 어노테이션 | 역할 |
|---|---|
| `@SpringBootConfiguration` | 설정 클래스임을 선언 (`@Configuration`의 특수화) |
| `@EnableAutoConfiguration` | 클래스패스 기반으로 자동 빈 설정 활성화 |
| `@ComponentScan` | 현재 패키지부터 하위 패키지를 스캔하여 `@Component` 계열 빈 등록 |

### `@Value`

`application.properties`의 값을 필드에 주입한다.

```properties
# application.properties
manager.id=hong
manager.age=30
manager.pass=1234
manager.roles=admin, manager, guest
```

```java
@SpringBootTest
@Slf4j
class BeOnline0507SpringBootApplicationTests {
    @Value("${manager.id}")
    private String id;

    @Value("${manager.age}")
    private int age;           // 타입 자동 변환

    @Value("${manager.roles}")
    private List<String> roles; // , 로 분리하여 리스트로 주입

    @Test
    void contextLoads() {
        log.info("id={}, age={}, roles={}", id, age, roles);
    }
}
```

### `@Autowired`

타입 기반으로 빈을 자동 주입한다. 생성자가 하나면 생략 가능하다.

```java
@SpringBootTest
@Slf4j
public class BeanCreateTest {
    @Autowired
    MemberService service;  // Spring이 MemberService 구현체를 찾아 주입

    @Test
    void beanTest() {
        Assertions.assertNotNull(service);
        log.info("service={}", service);
    }
}
```

### `@SpringBootTest`

테스트 시 전체 Spring Application Context를 로드한다. 실제 빈 주입과 설정을 테스트에서 그대로 사용할 수 있다.

### `@ServletComponentScan`

`@WebServlet`, `@WebFilter`, `@WebListener` 어노테이션이 붙은 클래스를 스캔하여 서블릿 컴포넌트로 등록한다. 기존 서블릿 기반 코드를 Spring Boot에 통합할 때 사용한다.

```java
// application 진입점에 선언
@SpringBootApplication
@ServletComponentScan // Servlet, Filter, Listener 스캔 활성화
public class BeOnline0507SpringBootApplication { ... }
```

---

## 5. Bean Scope

### 개념

Bean Scope는 Spring 컨테이너가 빈 인스턴스를 몇 개 생성하고 어떤 범위에서 공유할지 정의한다.

### 주요 Scope 종류

| Scope | 설명 | 인스턴스 수 |
|---|---|---|
| `singleton` (기본값) | 컨테이너 당 하나의 인스턴스 | 1개 |
| `prototype` | 요청할 때마다 새 인스턴스 생성 | 요청 수만큼 |
| `request` | HTTP 요청 당 하나 (웹 환경) | 요청 당 1개 |
| `session` | HTTP 세션 당 하나 (웹 환경) | 세션 당 1개 |
| `application` | ServletContext 당 하나 (웹 환경) | 1개 |

### Singleton Scope (기본값)

Spring 빈의 기본 스코프. 컨테이너에 빈을 최초 요청할 때 한 번 생성하고, 이후 요청에는 동일한 인스턴스를 반환한다.

```java
// @Scope 생략 = singleton
@Service
public class BasicMemberService implements MemberService { ... }

// 명시적으로 선언하면
@Service
@Scope("singleton")
public class BasicMemberService implements MemberService { ... }
```

**비교**
```java
// 직접 싱글톤 구현
private static BasicMemberService service = new BasicMemberService();
public static BasicMemberService getService() { return service; }

// Spring Boot: 컨테이너가 싱글톤 관리 (개발자 코드 불필요)
@Service
public class BasicMemberService { ... }
```

### Prototype Scope

요청할 때마다 새 인스턴스를 생성한다. 상태를 가지는 객체에 사용한다.

```java
@Component
@Scope("prototype")
public class SomeStatefulBean {
    private int state; // 각 사용처마다 독립적인 상태 유지
}
```

### Scope 선언 방법

```java
import org.springframework.context.annotation.Scope;

@Component
@Scope("prototype")          // 문자열로 선언
public class MyBean { ... }

// 또는 상수 사용
@Scope(ConfigurableBeanFactory.SCOPE_SINGLETON)
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
```

### 주의사항: Singleton에서 Prototype 빈 주입

Singleton 빈이 Prototype 빈을 의존성 주입으로 받으면 Prototype임에도 불구하고 Singleton처럼 동작하게 된다. <br> (Singleton이 한 번만 생성될 때 Prototype도 한 번만 주입되기 때문)

```java
@Service // singleton
public class SingletonService {
    @Autowired
    private PrototypeBean prototypeBean; // 매번 새로 만들어지길 원하지만 그렇지 않음!
}
```

이를 해결하려면 `ObjectProvider`, `ApplicationContext`, 또는 `@Lookup`을 사용해야 한다.

---

## 6. Blank Final

### 개념

`blank final`은 선언 시점에 초기화하지 않고, **나중에 반드시 한 번만 초기화되어야 하는 `final` 필드**를 말한다. 초기화는 생성자에서 이루어진다.

```java
// 일반 final: 선언과 동시에 초기화
private final String url = "jdbc:mysql://localhost:3306/ssafy";

// blank final: 선언만 하고 초기화는 생성자에서
private final MemberDao mDao; // ← blank final
```

### 예시

#### blank final 아닌 코드

```java
public class AuthController extends HttpServlet {
    // 선언, 초기화 동시에 → blank final X
    private final MemberService mService = BasicMemberService.getService();
    private final AddressService aService = BasicAddressService.getService();
}
```

#### blank final

```java
@RequiredArgsConstructor
public class AuthController extends HttpServlet {
    // 선언만 하고 초기화 X → blank final
    private final MemberService mService;
    private final AddressService aService;
    // → @RequiredArgsConstructor가 생성자를 생성하고
    //   Spring이 그 생성자를 통해 주입(초기화)함
}
```

`@RequiredArgsConstructor`가 생성하는 코드
```java
// Lombok이 컴파일 타임에 자동으로 생성하는 생성자
public AuthController(MemberService mService, AddressService aService) {
    this.mService = mService; // blank final 초기화
    this.aService = aService; // blank final 초기화
}
```

### blank final의 규칙

1. **반드시 생성자에서 초기화해야 한다.** 초기화하지 않으면 컴파일 에러가 발생한다.
2. **한 번만 초기화할 수 있다.** 이미 초기화된 blank final에 값을 재할당하면 컴파일 에러가 발생한다.
3. **인스턴스 변수라면 모든 생성자에서 초기화해야 한다.**

```java
public class Example {
    private final int value;  // blank final

    public Example(int value) {
        this.value = value;   // OK: 생성자에서 초기화
    }

    public void update(int v) {
        // this.value = v;    // 컴파일 에러: final 필드 재할당 불가
    }
}
```

### blank final을 생성자 주입에 사용하는 이유

```java
@Service
@RequiredArgsConstructor
public class BasicMemberService {
    private final MemberDao mDao;    // blank final
    private final AddressDao aDao;   // blank final
    private final DBUtil util;       // blank final
}
```

| 장점 | 설명 |
|---|---|
| **불변성 보장** | 생성 후 의존 객체가 바뀌지 않음 |
| **NPE 방지** | 생성자 주입 실패 시 앱 시작 단계에서 바로 오류 발생 |
| **순환 의존성 감지** | 컨테이너 초기화 시점에 순환 참조를 감지 |
| **테스트 용이** | 테스트에서 생성자로 모의 객체를 쉽게 주입 가능 |

---

## Spring Boot, Lombok 적용 

```
기존 코드                             적용 후 
─────────────────────────────────────────────────────────────
수동 싱글톤 (static 필드)         →  @Component / @Service / @Repository
new 로 직접 생성                   →  Spring IoC 컨테이너가 관리
구체 클래스에 의존                 →  인터페이스에 의존 (DI)
getter/setter 수동 작성 (80줄+)   →  @Data, @Getter, @Setter (Lombok)
반복적인 생성자 작성               →  @RequiredArgsConstructor (Lombok)
선언 시 초기화 (일반 final)       →  blank final + 생성자 주입
설정 하드코딩                      →  application.properties + @Value
```