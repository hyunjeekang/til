# MVC 패턴

역할에 따라 코드를 세 모듈로 분리하는 아키텍처 패턴이다. 하나의 서블릿이 모든 처리를 담당하면 코드가 비대해지고 유지보수가 어려워지는 문제를 해결하기 위해 사용한다.

<br>

## 1. 구성 요소

| 구성 요소 | 담당 | 역할 |
|---|---|---|
| Controller | Servlet | 클라이언트 요청을 받아 Model과 View의 흐름을 조정한다 |
| Model | Service / DAO | 비즈니스 로직 처리. 필요 시 DB 연동을 담당한다 |
| View | JSP | 사용자에게 보여주는 화면(UI)을 담당한다 |

### Controller

클라이언트의 요청을 직접 받는 진입점이다. 어떤 Model을 호출할지 결정하고, 처리 결과를 어떤 View로 넘길지 조정한다. 비즈니스 로직을 직접 처리하지 않는다.

### Model

비즈니스 로직과 데이터 처리를 담당한다. 크게 두 계층으로 나뉜다.

- **Service**
  - 업무 로직 처리
  - 여러 DAO를 조합해 하나의 작업 단위를 구성

- **DAO (Data Access Object)**
  - DB와 직접 통신
  - SQL 실행, 결과 반환이 이 계층의 역할

### View

사용자 인터페이스만 담당한다. JSP로 구현하며, Controller가 `request`나 `session`에 담아 전달한 데이터를 꺼내 화면에 렌더링하는 역할만 한다. 비즈니스 로직이 들어가면 안 된다!

<br>

## 2. Model 1 vs Model 2

### Model 1

JSP가 Controller 역할까지 함께 수행하는 방식이다. 요청 처리 로직과 화면 출력 코드가 한 파일에 섞여 있어 간단한 구조에서는 빠르게 개발할 수 있지만, 규모가 커지면 유지보수가 어려워진다.

```
클라이언트 → JSP (요청 처리 + 화면 출력) → DB
```

![Model 1](https://download.oracle.com/otn_hosted_doc/jdeveloper/1012/developing_mvc_applications/images/struts_model1.gif)

### Model 2

엄격한 MVC 분리를 적용한 방식이다. Servlet이 Controller를 담당하고, JSP는 화면만 관리한다. 역할이 명확히 분리되어 코드 수정 범위를 최소화할 수 있다.

```
클라이언트 → Servlet (Controller) → Model → Servlet → JSP (View) → 클라이언트
```

![Model 2](https://download.oracle.com/otn_hosted_doc/jdeveloper/1012/developing_mvc_applications/images/struts_model2.gif)

| 구분 | Model 1 | Model 2 |
|---|---|---|
| Controller | JSP | Servlet |
| View | JSP | JSP |
| 코드 분리 | 낮음 | 높음 |
| 유지보수 | 어려움 | 용이 |
| 적합한 규모 | 소규모 | 중·대규모 |

<br>

## 3. Model 2 요청 처리 흐름

일반적인 작업 순서는 아래와 같다.

1. 클라이언트가 URL로 요청을 보낸다.
2. **Controller(Servlet)** 가 요청을 받아 파라미터를 파악하고, 어떤 작업인지 판단한다.
3. **Model(Service/DAO)** 을 호출해 비즈니스 로직을 처리하고 결과를 받는다.
4. Controller가 결과 데이터를 `request.setAttribute()` 로 담는다.
5. **View(JSP)** 로 forward하거나, 다른 URL로 redirect한다.
6. JSP가 데이터를 꺼내 HTML을 렌더링하고 클라이언트에 응답한다.

