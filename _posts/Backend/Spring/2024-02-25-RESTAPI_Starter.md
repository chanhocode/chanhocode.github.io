---
title: "[Spring] 백엔드에서 어떤일이 벌어지고 있는가? _ REST API & Springboot starter와 자동설정"
writer: chanho
date: 2024-02-25 15:00:00 +0900
categories: [Backend, Spring]
tags: [backend, spring]
---

# Spring에서 REST API 구현의 백엔드 처리에 대해서

- 컨트롤러(Controller): REST API는 `@RestController`어노테이션을 사용하여 정의된 컨트롤러를 통해 요청을 처리한다. 이 컨트롤러는 HTTP 요청을 받고 응답을 반환한다.
- 서비스(Service) 계층: 컨트롤러는 일반적으로 비즈니스 로직을 처리하는 서비스 계층을 호출한다. 서비스 계층은 데이터를 처리하고 비즈니스 규칙을 적용한다.
- 데이터 액세스: Spring은 `JpaRepository`와 같은 데이터 액세스 인터페이스를 제공하여 데이터베이스와의 상호 작용을 쉽게 만든다. 이를 통해 'CRUD'작업이 간소화된다.
- 데이터 전송 객체(DTO): 컨트롤러와 클라이언트 사이의 데이터 교환은 DTO를 통해 이루어진다. DTO는 필요한 데이터 필드만을 포함하여 네트워크를 통한 데이터 전송을 최적화 한다.
- JSON/XML 변환: Spring은 Jackson과 같은 라이브러리를 사용하여 객체를 JSON또는 XML로 직렬화하고 역직렬화 한다.

# Spring Starter와 자동설정

## Spring Starter

> Spring Boot는 'Starters'라는 개념을 도입하여 필요한 의존성을 쉽게 추가할 수 있게 했다. 예를 들어, 'spring-boot-starter-web'은 웹 애플리케이션 개발에 필요한 모든 의존성을 제공한다.

- 장점: Starter를 사용하면 개발자는 각 기능에 필요한 의존성을 일일이 찾아 추가할 필요가 없다. Starter가 필요한 모든 것을 제공하기 때문에, 개발자는 더 빠르고 효율적으로 개발을 시작할 수 있다.
- 사용방법: Starter는 프로젝트의 'pom.xml' 또는 'build.gradle' 파일에 의존성으로 추가됩니다. 예를 들어, Maven을 사용하는 경우 'pom.xml' 파일에 원하는 Starter의 의존성을 추가하면 된다.

## 자동 설정

> Spring Boot는 '@EnableAutoConfiguration'어노테이션을 사용하여 필요한 구성을 자동으로 설정한다. 이는 클래스패스(classpath)에 있는 의존성을 기반으로 작동하며, 개발자가 수동으로 많은 설정을 하지 않아도 된다.

- 작동 방식: 예를 들어, 'class path'에 h2 데이터베이스 의존성이 있다면, Spring Boot는 내장형 h2 데이터베이스를 자동으로 설정한다. 또한, Thymeleaf나 JPA 같은 라이브러리가 포함되어 있다면, 이에 맞는 자동 설정이 적용된다.
- 사용자 정의: 자동 설정은 매우 유용하지만, 때로는 특정 설정을 사용자가 직접 정의해야 할 필요가 있다. 이 경우, 자동 설정을 오버라이드 하거나 필요한 설정을 수동을 추가할 수 있다.

# 요청은 어떻게 처리되는지에 대해서

## DispatcherServlet - Front Controller Pattern

> Spring MVC에서 DispatcherServlet은 프론트 컨트롤러로서 작동한다. 모든 HTTP 요청은 먼저 이 DispatcherServlet을 통과한다. 이는 웹 애플리케이션에서 단일 진입접을 제공하며, 모든 요청을 적절한 핸들러(컨트롤러)에 전달하는 역할을 한다.

- 작동 방식
  : 요청이 들어오면 DispatcherServlet은 요청 정보를 분석하여 어떤 컨트롤러가 요청을 처리할지 결정한다. 이를 위해 핸들러 매핑(Handler Mapping)을 사용한다.

## ➡️ Mapping Servlet

> 핸들러 매핑은 URL과 HTTP 메소드를 기반으로 적절한 컨트롤러 메소드를 찾는 역할을 한다. 예를 들어, '@RequestMapping' 어노테이션이 달린 메소드를 찾아 요청을 매핑한다.

- 프로세스
  : DispatcherServlet은 핸들러 매핑을 조회하여 요청에 해당하는 컨트롤러 메소드를 찾습니다. 그리고 이 메소드를 실행하여 요청을 처리한다.

## ➡️ Auto Configuration(DispatcherServletAutoConfiguration)

> Spring Boot에서는 DispatcherServletAutoConfiguration을 통해 DispatcherServlet이 자동으로 구성된다. 이는 Spring Boot의 자동 설정 기능의 일부로, 개발자가 별도의 구성 없이도 DispatcherServlet을 사용할 수 있게 해준다.

- 세부 사항
  : 이 자동 설정은 DispatcherServlet의 인스턴스를 생성하고, 스프링 애플리케이션 컨텍스트에 등록한다. 또한, 필요한 번들을 설정한다.

# 자동으로 JSON으로 변환되어 반환되는 과정

> DispatcherServlet이 URL 매핑을 확인하고, 매핑된 메소드를 실행하는 것에서 시작하여, '@ResponseBody'어노테이션과 'JacksonHttpMessageConverters'를 통해 Bean 객체가 JSON으로 변환되는 과정까지 이루어진다.

## DispatcherServlet URL 매핑 확인

DispatcherServlet은 들어오는 모든 HTTP 요청을 받아 처리한다. 이 과정에서 요청된 URL을 확인하고, 해당 URL에 매핑된 컨트롤러의 메소드를 찾는다.

## ➡️ 매핑된 메소드 실행

매핑된 컨트롤러 메소드가 실행된다. 이 메소드는 비즈니스 로직을 처리하고, 결과를 Bean객체로 반환할 수 있다.

## ➡️ @ResponseBody 어노테이션

컨트롤러 메소드에 '@ResponseBody' 어노테이션이 붙어 있으면, Spring은 해당 메소드의 반환 값을 HTTP 응답 본문으로 직접 사용한다. 즉, 이 어노테이션은 메소드가 반환하는 Bean객체를 HTTP응답의 본문으로 전송하도록 지시한다.

## ➡️ JacksonHttpMessageConverters

Spring Boot은 기본적으로 Jackson 라이브러리를 사용하여 객체를 JSON으로 변환한다. 이 변환은 'JacksonHttpMessageConverters'에 의해 처리된다. Spring Boot의 자동 설정은 'JacksonHttpMessageConverters'를 애플리케이션 컨텍스트에 자동으로 등록한다. 이는 'JacksonHttpMessageConvertersConfiguration'을 통해 이루어진다.

## ➡️ JSON으로의 자동 변환

'@ResponseBody' 어노테이션과 'JacksonHttpMessageConverters'가 설정되어 있으면, Spring은 Bean객체를 자동으로 JSON형식으로 변환하여 HTTP응답으로 반환한다. 이 과정은 내부적으로 Jackson라이브러리를 사용하여 Bean 객체의 데이터를 JSON형태로 직렬화 한다.

# 오류 매핑은 어디에서 설정하는지에 대해서 : ErrorMvcAutoConfiguration

## ErrorMvcAutoConfiguration

> Spring Boot은 'ErrorMvcAutoConfiguration'을 통해 기본적인 오류 처리 메커니즘을 제공한다. 이 설정은 오류가 발생했을 때, 표준 오류 페이지를 보여주는 'BasicErrorController'를 자동으로 등록한다. 또한 '/error' 경로로 모든 오류를 매핑한다. 즉, 어플리케이션에서 처리되지 않은 모든 오류는 '/error'경로로 리다이렉션 된다.
