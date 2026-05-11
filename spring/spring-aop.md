# Spring AOP 

---

## 목차

1. [AOP란?](#1-aop란)
2. [핵심 용어](#2-핵심-용어)
3. [Pointcut 표현식](#3-pointcut-표현식)
4. [Advice 종류](#4-advice-종류)
5. [JoinPoint / ProceedingJoinPoint](#5-joinpoint--proceedingjoinpoint)
6. [프록시와 AOP 동작 원리](#6-프록시와-aop-동작-원리)
7. [@Async / @Cacheable 과 AOP](#7-async--cacheable-과-aop)
8. [Inner Call 문제](#8-inner-call-문제)
---

## 1. AOP란?

**AOP (Aspect-Oriented Programming, 관점 지향 프로그래밍)**

핵심 비즈니스 로직과 별도로, 여러 곳에서 공통으로 필요한 기능(횡단 관심사)을 분리해서 관리하는 프로그래밍 방식이다.

```
핵심 관심사(Core Concern)     : 회원 가입, 주문 처리 등 비즈니스 로직
횡단 관심사(Cross-cutting)    : 로깅, 보안, 트랜잭션, 암호화 등
```

예를 들어, 회원 가입 시 비밀번호 암호화는 "회원 가입"이라는 핵심 로직의 일부가 아니다.  
암호화를 `save()` 메서드 안에 넣으면 비즈니스 코드가 오염되므로 AOP로 분리한다.

```java
@Before("execution(void *..PointcutTestBean.save(com.ssafy.live.model.dto.Member)) && args(member)")
public void encryptPass(Member member) {
    member.setPassword(new StringBuilder(member.getPassword()).reverse().toString());
}
```

---

## 2. 핵심 용어

| 용어 | 설명 |
|------|------|
| **Aspect** | 횡단 관심사를 모아 놓은 모듈 |
| **Advice** | 실제로 실행될 부가 기능 코드 |
| **Target** | Advice가 적용될 원본 객체 |
| **Join Point** | Aspect의 Advice를 삽입할 수 있는 모든 위치 (메서드 호출 시점) |
| **Pointcut** | "어느 Join Point에 적용할지"를 정의하는 표현식 |
| **Proxy** | Target을 감싸서 Advice를 주입하는 대리 객체 (Spring이 자동 생성) |
| **Weaving** | Pointcut에 따라 Advice를 Target에 연결하는 과정 (Spring 컨텍스트 초기화 시 발생) |

---

## 3. Pointcut 표현식

`execution()` 표현식이 가장 많이 쓰인다.

### 기본 문법

```
execution( [접근제어자] 반환타입 [패키지.클래스.]메서드명(파라미터) )
```

### 표현식 예제

```java

// 특정 클래스의 특정 시그니처
@Before("execution(void *..PointcutTestBean.save(com.ssafy.live.model.dto.Member))")

// 반환타입이 String이 아니면서 s로 시작하는 메서드, 파라미터 1개
@Before("execution(!String com.ssafy..s*(*))")

// 반환타입 long, Bean으로 끝나는 클래스, 첫 파라미터 int, 나머지 임의
@Before("execution(long com..*Bean.*(int, ..))")

// 반환타입이 List<MyClass>인 모든 메서드
@Before("execution(java.util.List<com.example.MyClass> *(..))")

// 모든 메서드 (와일드카드)
@Before("execution(* *(..))")
```

### 표현식 패턴

| 패턴 | 의미 |
|------|------|
| `*` | 임의의 하나 |
| `..` | 0개 이상의 패키지 or 파라미터 |
| `*..클래스명` | 임의 패키지의 해당 클래스 |
| `(..)` | 파라미터 개수/타입 무관 |
| `(*)` | 파라미터 1개 |
| `(int, ..)` | 첫 번째는 int, 나머지 무관 |

### Pointcut 재사용 (@Pointcut)

```java
@Pointcut("execution(* com.ssafy.service.*.*(..))")
public void complexExecution() {
    // 내용은 필요 없음 - 표현식만 정의
}

@Before("com.ssafy.live.basic.aspect.PointcutAspect.complexExecution()")
public void logBeforeServiceLayerExecution(JoinPoint joinPoint) { ... }

@After("com.ssafy.live.basic.aspect.PointcutAspect.complexExecution()")
public void logAfterServiceLayerExecution(JoinPoint joinPoint) { ... }
```

`@Pointcut`으로 표현식에 이름을 붙여두면 여러 Advice에서 재사용할 수 있다.

---

## 4. Advice 종류

### 실행 시점 요약

```
메서드 호출
    │
    ▼
@Before ──────── 메서드 실행 전
    │
    ▼
[Target 메서드 실행]
    │
    ├── 정상 반환 → @AfterReturning
    ├── 예외 발생 → @AfterThrowing
    └── 항상      → @After
    
@Around : 메서드 호출 전후를 모두 감쌈 (proceed() 호출로 직접 실행)
```

---

### @Before

메서드 실행 **전**에 동작. 파라미터를 직접 수정할 수 있다.

```java
// save() 호출 전에 비밀번호 암호화
@Before("execution(void *..PointcutTestBean.save(com.ssafy.live.model.dto.Member)) && args(member)")
public void encryptPass(Member member) {
    member.setPassword(new StringBuilder(member.getPassword()).reverse().toString());
}
```

> `&& args(member)`: Pointcut에서 파라미터를 Advice 메서드로 바인딩하는 방법

---

### @AfterReturning

메서드가 **정상 반환된 후** 동작. 반환값에 접근하고 수정할 수 있다.

```java
// select() 후 반환되는 Member의 비밀번호를 마스킹
@AfterReturning(value = "execution(* *..PointcutTestBean.select(*))", returning = "member")
public void maskingPassword(Member member) {
    member.setPassword("*");
}
```

> `returning = "member"`: 반환값을 해당 이름의 파라미터로 받는다.  
> 반환값이 객체일 경우 내부 필드 수정은 가능하지만, 반환값 자체를 교체하려면 `@Around`를 써야 한다.

---

### @AfterThrowing

메서드에서 **예외가 발생했을 때** 동작. 예외 알림, 로깅 등에 활용.

```java
// factorial() 예외 발생 시 관리자 이메일 전송처럼 처리
@AfterThrowing(value = "execution(* *..PointcutTestBean.factorial(int)", throwing = "e")
public void notifyError(JoinPoint jp, RuntimeException e) {
    log.debug("signature: {}, exception: {}", jp.getSignature(), e.getMessage());
}
```

> `throwing = "e"`: 발생한 예외를 파라미터로 받는다.  
> `@AfterThrowing`은 예외를 잡는 게 아니라 가로채는 것임 -> 예외는 그대로 전파된다.

---

### @Around

메서드 호출 전후를 **모두 제어**한다.

```java
// add(int, int)의 파라미터와 반환값을 조작
@Around("execution(* *..PointcutTestBean.add(int, int))")
public Integer modifyAdd(ProceedingJoinPoint pjp) throws Throwable { // 예외 전파

    Object[] args = pjp.getArgs();
    args[0] = (Integer) args[0] * 10; // 1. 파라미터 완전 대체 가능
    Integer result = (Integer) pjp.proceed(args); // 2. 타겟의 메서드 호출
    return result % 2 == 0 ? result : result / 2; // 3. 결과 조작 가능
}

```

| `@Around`에서 할 수 있는 것 |
|-----------------------------|
| 타겟 메서드 실행 여부 결정 (`proceed()` 호출 안 하면 실행 안 됨) |
| 파라미터 값 변경 후 `proceed(args)` 전달 |
| 반환값 조작 |
| 예외 잡아서 처리하거나 다른 예외로 변환 |

---

### DAO 레이어 공통 로깅 (@Before + LoggingAspect)

```java
// DAO 레이어 전체에 로깅 적용
@Before("execution(* com.ssafy..dao..*(..))")
public void loggingDao(JoinPoint jp) {
    log.debug("method call", jp.getSignature(), Arrays.toString(jp.getArgs()));
}
```

`com.ssafy` 하위 어디든 `dao` 패키지 내 모든 메서드 호출 전에 로그를 남긴다.

---

## 5. JoinPoint / ProceedingJoinPoint

Advice 메서드가 "현재 가로챈 메서드의 정보"를 얻거나 "실제로 실행"하기 위해 쓰는 객체.

---

### JoinPoint (jp)

가로챈 메서드의 **정보를 읽기만** 할 수 있는 객체.  
`@Before`, `@After`, `@AfterReturning`, `@AfterThrowing`에서 사용.

```java
@Before("execution(* com.ssafy..dao..*(..))")
public void loggingDao(JoinPoint jp) {
    jp.getSignature() // 메서드 시그니처: "Member BasicMemberDao.selectDetail(Connection, String)"
    jp.getArgs()      // 파라미터 값: [con, "admin@ssafy.com"]
    jp.getTarget()    // 타겟 객체: BasicMemberDao 인스턴스
    jp.getThis()      // 프록시 객체
}
```

```java
@AfterThrowing(value = "...", throwing = "e")
public void notifyError(JoinPoint jp, RuntimeException e) {
    // jp로 "어떤 메서드에서" 예외가 났는지 알 수 있음
    log.debug("signature: {}, exception: {}", jp.getSignature(), e.getMessage());
}
```

---

### ProceedingJoinPoint (pjp)

`JoinPoint`를 상속하면서 **`proceed()`가 추가**된 객체. **`@Around` 전용.**  
`proceed()`가 타겟 메서드를 실제로 실행하는 열쇠다.

```java
@Around("execution(* *..PointcutTestBean.add(int, int))")
public Integer modifyAdd(ProceedingJoinPoint pjp) throws Throwable {

    Object[] args = pjp.getArgs();            // JoinPoint 기능은 그대로 사용 가능
    args[0] = (Integer) args[0] * 10;         // 파라미터 교체

    Integer result = (Integer) pjp.proceed(args); // "이제 진짜 메서드 실행해"
    return result % 2 == 0 ? result : result / 2; // 반환값 조작
}
```

`proceed()`를 **호출하지 않으면 타겟 메서드가 아예 실행되지 않는다.**  
이걸 활용하면 캐시 구현이 가능하다

```java
private final Map<Integer, Long> cache = new HashMap<>();

@Around("execution(* *..PointcutTestBean.factorial(int))")
public Long cacheFactorial(ProceedingJoinPoint pjp) throws Throwable {
    int n = (int) pjp.getArgs()[0];
    if (cache.containsKey(n)) {
        return cache.get(n);       // proceed() 안 부름 → 원본 메서드 실행 X
    }
    Long result = (Long) pjp.proceed(); // 첫 호출만 실제 실행
    cache.put(n, result);
    return result;
}
```

---

### 비교

| | `JoinPoint` | `ProceedingJoinPoint` |
|--|------------|----------------------|
| 사용 위치 | `@Before` / `@After` / `@AfterReturning` / `@AfterThrowing` | `@Around` 전용 |
| 메서드 정보 조회 | O | O (상속) |
| 타겟 메서드 실행 | X (자동으로 실행됨) | O (`proceed()` 직접 호출) |
| 실행 여부 결정 | X | O (`proceed()` 생략 시 실행 안 됨) |

> `@Around`에서 `throws Throwable`이 필수인 이유: 타겟 메서드가 어떤 예외를 던질지 모르기 때문에 Advice가 그대로 전파할 수 있어야 한다.

---

## 6. 프록시와 AOP 동작 원리

Spring AOP는 **프록시 기반**으로 동작한다. 실제 빈 대신 프록시 객체를 주입하고, 메서드 호출을 가로채서 Advice를 실행한다.

```
클라이언트 → [Proxy] → Advice 실행 → [Target 메서드] → 결과 반환
```

### CGLIB 프록시 확인

```java
@Test
public void isProxy() {
    log.debug(config.getClass().getName());
    // 출력: com.ssafy.live.aop.config.CustomConfiguration$$SpringCGLIB$$0
    //                                                         ^^^^^^^^^^^ CGLIB이 만든 서브클래스

    // AopUtils.isAopProxy()는 AOP 어드바이스 적용 여부 체크 → false
    log.debug("isproxyclass: {}", AopUtils.isAopProxy(config.getClass()));

    // 클래스 이름에 "$$" 포함 여부로 CGLIB 프록시 판별
    Assertions.assertTrue(config.getClass().getName().contains(ClassUtils.CGLIB_CLASS_SEPARATOR));
}
```

- `@Configuration` 클래스도 CGLIB 프록시로 감싸진다 → `@Bean` 메서드 중복 호출 방지
- AOP가 적용된 빈은 원본 클래스가 아닌 `$$SpringCGLIB$$0` 접미사 붙은 서브클래스 인스턴스

---

## 7. @Async / @Cacheable 과 AOP

이 두 어노테이션도 내부적으로 AOP 프록시를 활용한다.

### @Async - 비동기 실행

```java
@Configuration
@EnableAsync   // @Async 활성화
@EnableCaching // @Cacheable 활성화
public class CustomConfiguration { ... }
```

```java
@Async
public void heavyWork(int i) throws InterruptedException {
    Thread.sleep(1000);
    log.trace("{}에 의해 처리중 {}", Thread.currentThread().getName(), ptb.factorial(i));
}
```

```java
// ThreadPoolTaskExecutor 설정
@Bean
public TaskExecutor customTaskExecutor() {
    ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
    executor.setCorePoolSize(5);   // 기본 스레드 수
    executor.setQueueCapacity(25); // 대기열 용량
    executor.setMaxPoolSize(10);   // 대기열 초과 시 최대 확장 스레드 수
    return executor;
}
```

```java
@Test
public void asyncTest() throws InterruptedException {
    Assertions.assertTrue(AopUtils.isAopProxy(hbean)); // HeavyWorkBean도 프록시!
    log.debug("executor: {}", executor.getClass().getName());
    for (int i = 0; i < 10; i++) {
        hbean.heavyWork(i); // 각각 별도 스레드에서 실행됨
    }
}
```

### @Cacheable - 캐시 처리

```java
@Cacheable("guguCache") // 같은 인자로 호출 시 캐시된 결과 반환
public List<String> cacheableGugu(int n) {
    log.info("캐시 미적용: 실제 계산 수행 - n={}", n); // 첫 호출 시만 출력됨
    List<String> result = new ArrayList<>();
    for (int i = 1; i <= 9; i++) {
        result.add(n + " x " + i + " = " + (n * i));
    }
    return result;
}
```

---

## 8. Inner Call 문제

AOP는 **프록시를 통한 외부 호출**에만 적용된다.  
같은 클래스 내부에서 `this.메서드()`를 호출하면 프록시를 거치지 않아 AOP가 동작하지 않는다.

```java
public void methodA() {
    log.debug("methodA called");
}

public void methodB() {
    log.debug("methodB called");
    methodA(); // ← 내부 호출! 프록시를 거치지 않음
}
```

```java
@Before("execution(void *..HeavyWorkBean.method*())")
void prepareMethodA(JoinPoint jp) {
    log.debug("method 호출 준비: {}", jp.getSignature());
}
```

```java
@Test
public void innerCallTest1() {
    hbean.methodA(); // 프록시를 통한 외부 호출 → AOP 적용 O
}

@Test
public void innerCallTest2() {
    hbean.methodB(); // methodB는 AOP 적용 O
                     // methodB 내부의 methodA() 호출은 AOP 적용 X (inner call)
}
```

**결론**: `innerCallTest2`를 실행하면 Advice 로그가 `methodB`에 대해서만 찍히고,  
내부에서 호출된 `methodA`에 대해서는 찍히지 않는다.

---

## 한눈에 보는 Advice 선택 기준

```
메서드 실행 전에 뭔가 하고 싶다          → @Before
메서드 정상 반환 후 반환값을 가지고 처리  → @AfterReturning
예외 발생 시 별도 처리 (알림, 로깅)       → @AfterThrowing
정상/예외 무관하게 항상 후처리            → @After
전후 모두 제어 / 실행 여부 결정 / 반환값 교체 → @Around
```
