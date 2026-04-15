# [BE] 빌드 도구(Maven)와 Spring Boot의 설정 자동화

## 1. 빌드 도구 메이븐(Maven)과 의존성 관리
과거에 직접 다운로드해야 했던 라이브러리를 효율적으로 관리하기 위해 빌드 도구를 사용한다.

* **MVN Repository**
  * 라이브러리들을 모아둔 저장소([https://mvnrepository.com/](https://mvnrepository.com/))

* **메이븐(Maven) 프로젝트 전환**
  * 일반 자바 웹 프로젝트를 'Convert to Maven Project' 기능을 통해 메이븐 프로젝트로 변환할 수 있다.

* **pom.xml과 의존성(Dependency)**
  * 프로젝트를 메이븐으로 변환하면 생성되는 핵심 설정 파일이 `pom.xml`이다.
  * 이 파일에 필요한 외부 라이브러리(Dependency)를 명시하여 가져올 수 있다.

* **로컬 캐싱 (.m2 폴더)**
  * 메이븐은 선언된 라이브러리들을 중앙 저장소에서 다운로드하여 내 컴퓨터의 `.m2` 폴더에 저장해두고 프로젝트에 적용한다.


## 2. pom.xml 기본 구조
아래는 MySQL 연동(mysql-connector-java)과 스프링 핵심 컨텍스트(spring-context) 라이브러리를 의존성으로 추가한 `pom.xml` 예시다.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>Backend</groupId>
	<artifactId>Backend</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>
	
	<dependencies>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>8.0.33</version>
			<scope>compile</scope>
		</dependency>
		
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>6.1.21</version>
			<scope>compile</scope>
		</dependency>
	</dependencies>
	
	<build>
		<plugins>
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.13.0</version>
				<configuration>
					<release>25</release>
				</configuration>
			</plugin>
			<plugin>
				<artifactId>maven-war-plugin</artifactId>
				<version>3.2.3</version>
			</plugin>
		</plugins>
	</build>
</project>
```


## 3. 스프링 부트(Spring Boot)의 설정 자동화
스프링(Spring) 프레임워크의 초기 세팅 부담을 덜어주기 위해 스프링 부트(Spring Boot)가 등장했다.

* **버전 호환성과 설정 자동화**
  * 스프링 부트는 여러 라이브러리 간의 버전 호환성을 맞춰주고, 개발자가 일일이 설정해야 했던 부분들을 편하게 자동화해 준다.

* **설정 자동화의 양면성**
  * 이러한 자동화 기능은 잘 쓰면 개발 생산성을 크게 높여준다.
  * 하지만 지금 당장 안 쓰고 나중에 쓸 라이브러리들의 설정까지 모조리 자동화되어 함께 돌아가는 경우도 발생할 수 있으므로, 내부 동작을 이해하고 제어할 줄 알아야 한다.