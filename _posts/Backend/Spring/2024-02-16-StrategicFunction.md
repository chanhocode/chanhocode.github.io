---
title: "[Spring] 스프링 부트의 전략적 기능들"
writer: chanho
date: 2024-02-16 22:50:00 +0900
categories: [Backend, Spring]
tags: [backend, spring]
---

스프링 부트(Spring Boot)는 현대적인 자바 애플리케이션 개발을 위해 설계된 프레임워크로, 개발자가 빠르고 효율적으로 애플리케이션을 개발하고 배포할 수 있도록 다양한 전략적 특징을 제공한다.

1. Auto-Configuration
   > Spring Boot는 'Classpath'에 기반하여 필요한 라이브러리를 자동으로 구성한다.
2. Starter Kits
   > Spring Boot는 다양한 'Stater'종속성을 제공한다. 이것은 특정 기능을 구현하는 데 필요한 라이브러리들을 미리 조합해 놓은 것으로, 예를 들어 'spring-boot-starter-web'은 웹 애플리케이션 개발에 필요한 모든 종속성을 포함하고 있다.
3. Embedded Server Support
   > Spring Boot는 Tomcat, Jetty, Undertow와 같은 내장 서버를 지원하여, 별도의 서버 설치 없이 애플리케이션을 실행할 수 있게 해준다. 이를 통해 애플리케이션을 JAR 파일로 패키징하고, 이를 어디서든 실행할 수 있다.
4. No-Code Configuration
   > Spring Boot는 XML 기반의 구성 파일 없이 자바 기반의 구성을 지원한다. 'application.properites' 또는 'application.yml' 파일을 통해 애플리케이션의 설정을 관리할 수 있다.
5. Actuator를 통한 모니터링과 관리
   > Spring Boot Actuator는 운영 중인 애플리케이션의 상태, 메트릭스, 환경 속성 등을 모니터링하고 관리하는 데 사용된다.
6. Profiles를 통한 환경 구분
   > 다양한 개발 및 운영 환경에 맞춰 애플리케이션 설정을 구분할 수 있다. 예를 들어, 개발 환경과 운영 환경에서 다른 데이터베이스 설정을 사용할 수 있다.
7. Cloud-Native 지원
   > Spring Boot는 클라우드 환경, 특히 마이크로서비스 아키텍처와의 통합에 최적화되어 있다.

# Profile

> Spring Boot 애플리케이션의 `다양한 환경(개발, 테스트, 운영)을 구분`하기 위해 사용된다. 예를 들어 '@Profile("development")' 애너테이션을 사용하여 특정 클래스나 설정이 개발 환경에서만 활성화되도록 할 수 있다. 프로파일을 통해 환경별로 다른 설정 값을 적용할 수 있어, 애플리케이션의 유연성을 높일 수 있다.

## `@Pfofile` 애너테이션을 사용하여 특정 프로파일 적용

```java
@Component
@Profile("development")
public class DevConfig { "개발 환경 구성 로직" }

@Component
@Profile("Production")
public class ProdConfig { "운영 환경 구성 로직 " }
```

## `application.properties`, `application.yml`파일에서 프로파일 적용

```application-development.profiles
string.datasource.url=jdbc:h2:mem:devdb
```

```application-production.profiles
string.datasource.url=jdbc:mysql
```

# Configuration Properties

> '@ConfigurationProperties' 애너테이션을 통해 외부 설정 파일(application.properties, application.yml)에 정의된 속성을 자바 클래스에 매핑하는 데 사용된다. 이를 통해 타입 안전한 방식으로 구성 데이터를 관리하고, 코드 내에서 쉽게 접근할 수 있다.

## 자주 사용되는 설정 소개

### 서버 설정

```
# 애플리케이션 서버의 포트 번호를 설정한다.
server.port=8080

# 애플리케이션의 컨텍스트 패스를 설정한다.
server.servlet.context-path=/myapp
```

### 데이터베이스 연결 설정

```
# 데이터베이스의 URL을 설정한다.
spring.datasource.url=jdbc:mysql://localhost:3306/mydb

# 데이터베이스 연결을 위한 사용자 이름과 비밀번호를 설정한다.
spring.datasource.username=dbuser
spring.datasource.password=dbpass

# 사용할 JDBC 드라이버를 설정한다.
spring.datasource.driver-class-name=com.mysql.Deiver
```

### JPA/Hibernate 설정

```
# Hibernate의 DDL 생성 전략을 설정한다.
spring.jpa.hibernate.ddl-auto=update

# JPA가 실행하는 SQL을 로그로 출력할지 여부를 설정합니다.
spring.jpa.show-sql=true
```

### 로깅 설정

```
# 로깅 레벨을 설정한다.
logging.level.root=WARN

# 특정 패키지 또는 클래스의 로깅 레벨을 설정한다.
logging.level.org.springframework.web=DEBUG
```

### 스프링 시큐리티 설정

```
# 기본 사용자 이름과 비밀번호를 설정한다.
spring.security.user.name=admin
spring.security.password=adminpass
```

# Embedded Server (WAR, JAR)

> Spring Boot는 Tomcat, Jetty, Undertow와 같은 내장 서버를 제공하여, 별도의 서버 설치 없이 독립적인 애플리케이션으로 실행 할 수 있게 한다. WAR(웹 애플리케이션 아카이브) 또는 JAR(자바 아카이브) 형식으로 패키징할 수 있으며, 특히 JAR 파일로 패키징하면 실행 가능한 Stand alone Application으로 배포할 수 있다.

## WAR (Web Application Archive)

> WAR형식은 웹 애플리케이션을 포함하는데 사용되는 Java EE 표준 포맷입니다. 이 포맷은 웹 애플리케이션의 서블릿, JSP, HTML, Javascript, CSS 파일 등을 포함한다. 전통적인 Java EE 웹 애플리케이션은 WAR 파일 형태로 패키징되어 외부의 서버에 배포된다.

## JAR (Java Archive)

> JAR 파일은 자바 클래스 파일, 관련 메타데이터, 리소스 파일을 압축한 아카이브 파일을 압축한 아카이브 파일 형식이다. Spring Boot는 실행 가능한 JAR 파일을 지원하며, `내장 서버와 애플리케이션 코드를 하나의 JAR 파일로 패키징`할 수 있게 해준다. 이는 `별도의 서버 설치 없이도 애플리케이션을 실행`할 수 있으며, 마이크로 서비스 아키텍처와 같이 독립적인 서비스를 개발하는 데 효과적이다.

# Actuator

> Spring Boot Actuator는 애플리케이션의 상태와 성능을 모니터링하고 관리할 수 있는 여러 엔드포인트를 제공한다. 이를 통해 애플리케이션의 건강 상태, 메트릭스, 로그 레벨 설정, 환경 속성 등을 실시간으로 확인하고 관리할 수 있다.

## 모든 Actuator

```
management.endpoints.web.exposure.include=*
```

```json
{
  "_links": {
    "self": { "href": "http://localhost:8080/actuator", "templated": false },
    "beans": {
      "href": "http://localhost:8080/actuator/beans",
      "templated": false
    },
    "caches-cache": {
      "href": "http://localhost:8080/actuator/caches/{cache}",
      "templated": true
    },
    "caches": {
      "href": "http://localhost:8080/actuator/caches",
      "templated": false
    },
    "health": {
      "href": "http://localhost:8080/actuator/health",
      "templated": false
    },
    "health-path": {
      "href": "http://localhost:8080/actuator/health/{*path}",
      "templated": true
    },
    "info": {
      "href": "http://localhost:8080/actuator/info",
      "templated": false
    },
    "conditions": {
      "href": "http://localhost:8080/actuator/conditions",
      "templated": false
    },
    "configprops": {
      "href": "http://localhost:8080/actuator/configprops",
      "templated": false
    },
    "configprops-prefix": {
      "href": "http://localhost:8080/actuator/configprops/{prefix}",
      "templated": true
    },
    "env": { "href": "http://localhost:8080/actuator/env", "templated": false },
    "env-toMatch": {
      "href": "http://localhost:8080/actuator/env/{toMatch}",
      "templated": true
    },
    "loggers": {
      "href": "http://localhost:8080/actuator/loggers",
      "templated": false
    },
    "loggers-name": {
      "href": "http://localhost:8080/actuator/loggers/{name}",
      "templated": true
    },
    "heapdump": {
      "href": "http://localhost:8080/actuator/heapdump",
      "templated": false
    },
    "threaddump": {
      "href": "http://localhost:8080/actuator/threaddump",
      "templated": false
    },
    "metrics-requiredMetricName": {
      "href": "http://localhost:8080/actuator/metrics/{requiredMetricName}",
      "templated": true
    },
    "metrics": {
      "href": "http://localhost:8080/actuator/metrics",
      "templated": false
    },
    "scheduledtasks": {
      "href": "http://localhost:8080/actuator/scheduledtasks",
      "templated": false
    },
    "mappings": {
      "href": "http://localhost:8080/actuator/mappings",
      "templated": false
    }
  }
}
```
