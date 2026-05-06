# Spring DI 정리

## 1. 의존 관계

한 객체가 다른 객체를 사용하는 관계를 **의존 관계**라고 한다.

```
MovieManager → MovieService (인터페이스)
                    ↑               ↑
          KoreaMovieService   GlobalMovieService
                    ↓
              MovieDao (인터페이스)
                    ↑
              MovieDaoImpl
```

- `MovieManager`는 `MovieService`에 의존한다.
- `KoreaMovieService`와 `GlobalMovieService`는 `MovieDao`에 의존한다.

의존 관계가 있으면 한 클래스 안에서 다른 클래스의 인스턴스가 필요하다.

---

## 2. DI의 필요성

DI 없이 직접 생성하면:

```java
// DI 없이 직접 생성 (나쁜 예)
public class KoreaMovieService {
    MovieDao movieDao = new MovieDaoImpl(); // 강한 결합
}
```

문제점:
- `KoreaMovieService`가 `MovieDaoImpl`이라는 **구체 클래스에 강하게 결합**된다.
- `MovieDaoImpl`을 다른 구현체로 교체하려면 서비스 코드를 직접 수정해야 한다.
- Spring이 관리하는 싱글톤 빈이 아닌 **별도 인스턴스**가 생성되므로 `Movie`가 저장된 HashMap이 공유되지 않는다.
- 테스트 시 목(Mock) 객체 주입이 불가능하다.

DI를 사용하면 객체 생성을 Spring에게 맡기고, 클래스는 **인터페이스에만 의존**하게 된다.

---

## 3. DI (Dependency Injection)

**의존 객체를 외부에서 주입받는 패턴**이다. 객체 스스로 의존 객체를 생성하지 않고, Spring IoC 컨테이너가 대신 생성하고 주입해준다.

```java
// DI 적용
@Service("koreaMovieService")
public class KoreaMovieService implements MovieService {

    MovieDao movieDao; // 인터페이스에만 의존

    @Autowired
    public KoreaMovieService(MovieDao movieDao) { // Spring이 주입해줌
        this.movieDao = movieDao;
    }
}
```

- `KoreaMovieService`는 `MovieDao` **인터페이스**만 알고 있다.
- `MovieDaoImpl`이라는 구체 클래스는 Spring이 알아서 찾아 주입한다.

---

## 4. Bean 관리 과정

```
1. AnnotationConfigApplicationContext 초기화
        ↓
2. @Configuration 클래스(MovieConfig) 읽기
        ↓
3. @ComponentScan("com.ssafy.hw") 실행
        ↓
4. 해당 패키지에서 스테레오타입 어노테이션(@Component, @Service, @Repository) 탐색
        ↓
5. 탐색된 클래스를 싱글톤 빈으로 등록
        - MovieDaoImpl    (@Repository)
        - KoreaMovieService  (@Service("koreaMovieService"))
        - GlobalMovieService (@Service("globalMovieService"))
        - MovieManager       (@Component)
        ↓
6. 의존 관계 분석 후 주입(@Autowired)
        ↓
7. context.getBean()으로 이미 완성된 빈을 꺼내 사용
```

```java
// MovieMain.java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MovieConfig.class);
MovieManager movieManager = context.getBean(MovieManager.class); // 이미 DI 완료된 빈
```

**싱글톤**: Spring은 기본적으로 빈을 싱글톤으로 관리한다. `MovieDaoImpl` 인스턴스는 애플리케이션 전체에서 하나만 존재하므로, `KoreaMovieService`와 `GlobalMovieService`가 **같은 HashMap을 공유**한다. 이 덕분에 한국 서비스로 등록한 영화를 글로벌 서비스에서도 조회할 수 있다.

---

## 5. 명시적 DI vs 묵시적 DI

### 묵시적 DI

스테레오타입 어노테이션 + `@ComponentScan`으로 Spring이 자동으로 빈을 탐색하고 등록한다.

```java
// MovieConfig.java
@Configuration
@ComponentScan("com.ssafy.hw") // 패키지 하위를 자동 스캔
public class MovieConfig {
    // 아무것도 작성하지 않아도 됨
}
```

```java
@Repository          // 자동으로 빈 등록됨
public class MovieDaoImpl implements MovieDao { ... }

@Service("koreaMovieService") // 자동으로 빈 등록됨
public class KoreaMovieService implements MovieService { ... }
```

### 명시적 DI

`@Configuration` 클래스 안에서 `@Bean` 메서드로 직접 빈을 정의한다.

```java
@Configuration
public class MovieConfig {

    @Bean
    public MovieDao movieDao() {
        return new MovieDaoImpl();
    }

    @Bean("koreaMovieService")
    public MovieService koreaMovieService() {
        return new KoreaMovieService(movieDao()); // 직접 주입
    }
}
```

| 구분 | 묵시적 DI | 명시적 DI |
|------|-----------|-----------|
| 설정 위치 | 클래스에 어노테이션 | `@Configuration` 클래스의 `@Bean` 메서드 |
| 코드량 | 적음 | 많음 |
| 제어 유연성 | 낮음 | 높음 (생성 로직 커스터마이징 가능) |
| 주로 사용 | 만든 클래스 | 외부 라이브러리 클래스 (수정 불가) |

---

## 6. 주입 방식 비교

### 생성자 주입(권장)
```java
@Autowired
public KoreaMovieService(MovieDao movieDao) {
    this.movieDao = movieDao;
}
```

- **불변성**: 주입 후 값을 바꿀 수 없다 (`final` 선언 가능).
- **필수 의존성 보장**: 객체 생성 시점에 의존 객체가 없으면 즉시 오류 발생.
- **테스트 용이성**: `new KoreaMovieService(mockDao)`처럼 직접 생성 가능.
- **Spring 공식 권장 방식**.

### Setter 주입

```java
@Autowired
public void setMovieDao(MovieDao movieDao) {
    this.movieDao = movieDao;
}
```

- **선택적 의존성**에 적합하다 (없어도 객체 생성 가능).
- 주입 전에 메서드를 호출하면 `NullPointerException` 위험이 있다.

### 필드 주입

```java
@Autowired
private MovieDao movieDao;
```

- 코드가 가장 짧다.
- `final` 선언 불가, 테스트 시 Spring 컨테이너 없이 주입 불가능
- 권장하지 않는다.

---

## 7. @Component와 스테레오타입 어노테이션

`@Component`는 Spring이 빈으로 등록하도록 표시하는 기본 어노테이션이다. 역할에 따라 세분화된 어노테이션들이 있다.

| 어노테이션 | 적용 계층 | 이 프로젝트 사용 위치 |
|-----------|-----------|----------------------|
| `@Component` | 일반 컴포넌트 | `MovieManager` |
| `@Service` | 비즈니스 로직 계층 | `KoreaMovieService`, `GlobalMovieService` |
| `@Repository` | 데이터 접근 계층 | `MovieDaoImpl` |
| `@Controller` | 웹 요청 처리 계층 | (이 프로젝트에서 미사용) |

네 어노테이션 모두 내부적으로 `@Component`를 포함하므로 빈 등록 기능은 동일하다. <br>
`@Repository`는 데이터 접근 예외를 Spring 예외로 변환하는 추가 기능이 있다.

```java
@Repository           // DAO 계층
public class MovieDaoImpl implements MovieDao { ... }

@Service("koreaMovieService")  // 서비스 계층, 빈 이름 지정
public class KoreaMovieService implements MovieService { ... }

@Component            // 특정 계층 구분 없는 일반 컴포넌트
public class MovieManager { ... }
```

빈 이름은 기본적으로 **클래스명의 첫 글자를 소문자**로 바꾼 이름이 된다 (`MovieDaoImpl` → `movieDaoImpl`). <br>`@Service("koreaMovieService")`처럼 이름을 직접 지정할 수도 있다.

---

## 8. @Qualifier

같은 타입의 빈이 여러 개일 때 어떤 빈을 주입할지 지정한다.

위 예시에서는 `MovieService` 인터페이스를 구현한 빈이 두 개다.
- `koreaMovieService` (`KoreaMovieService`)
- `globalMovieService` (`GlobalMovieService`)

`@Autowired`만 쓰면 Spring은 어떤 빈을 주입해야 할지 몰라 오류가 발생한다.

```java
// MovieManager.java
@Autowired
public MovieManager(@Qualifier("koreaMovieService") MovieService movieService) {
    this.movieService = movieService;
}
```

`@Qualifier("koreaMovieService")`로 이름을 명시해 `KoreaMovieService`를 선택적으로 주입받는다.
