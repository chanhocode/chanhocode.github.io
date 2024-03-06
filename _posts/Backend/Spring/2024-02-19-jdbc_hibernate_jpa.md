---
title: "[SpringDB] JDBC, Hibernate, JPA 기초"
writer: chanho
date: 2024-02-19 23:00:00 +0900
categories: [Backend, Spring]
tags: [backend, spring, database]
---

# JDBC (Java Database Connectivity)

> 'JDBC'는 자바 언어를 위한 `데이터베이스 접근 기술`이다. 이는 `자바 프로그램이 다양한 유형의 데이터베이스와 상호작용할 수 있게 해주는 표준 SQL 인터페이스`를 제공한다. 'JDBC'는 데이터베이스 드라이버를 통해 데이터베이스와 통신하며 각 '데이터베이스 벤더'(소프트웨어와 관련된 제품 및 서비스를 제공하는 기업)는 자신의 데이터베이스에 맞는 'JDBC드라이버'를 제공한다.

<img width="709" alt="image" src="https://github.com/chanhocode/chanhocode.github.io/assets/105937460/a69e8e56-8c45-4d41-ba3e-9f4dfb27b1b5">

## JDBC 드라이버

> 자바 애플리케이션과 데이터베이스 간의 통신을 가능하게 하는 소프트웨어 컴포넌트이다. JDBC 드라이버는 JDBC API를 구현하며, 자바 애플리케이션에서 SQL 명령을 데이터베이스에 전달하고, 그 결과를 다시 애플리케이션으로 반환하는 역할을 한다.

## JDBC 작동 방식

1. 드라이버 로딩
   - 'JDBC'를 사용하기 위해 먼저 데이터베이스에 접근할 수 있도록 특정 '데이터베이스 드라이버'를 로드해야 한다. 이것은 'Class.forName()'메소드를 사용하여 수행된다.
2. 연결 생성
   - 드라이버가 로드되면, 'DriverManager.getConnection()' 메소드를 사용하여 데이터베이스에 연결한다. 이 메소드는 데이터베이스 URL, 사용자 이름, 비밀번호를 인자로 받아 연결(Connection)객체를 생성한다.
3. SQL 문 작성
   - 'Statement', 'PreparedStatment', 또는 'CallableStatement' 객체를 사용하여 수행
     - `Statement`: 정적 SQL문을 실행하는 데 사용된다.
     - `PreparedStatement`: 매개변수화된 쿼리를 실행하는 데 사용되며, SQL인젝션 공격으로부터 보호할 수 있는 이점이 있다.
     - `CallableSatement`: 데이터베이스의 저장 프로시저를 호출하는 데 사용된다.
4. 쿼리 실행 및 결과 처리
   - SQL문이 준비되면 'executeQuery()', 'executeUpdate()', 'execute()'메소드중 하나를 사용하여 쿼리를 실행한다.
     - `executeQuery()`: 데이터를 반환하는 SELECT쿼리에 사용되며, 'ResultSet' 객체로 결과를 받습니다.
     - `executeUpdate()`: 데이터를 수정하는 INSERT, UPDATE, DELETE 쿼리에 사용되며, 영향 받은 행의 수를 반환합니다.
     - `execute()`: 어떤 종류의 SQL문이든 실행할 수 있으며, 반환 값이 있는지 여부에 따라 'ResultSet'객체 또는 업데이트 카운터를 반환한다.
5. 자원 해제:
   - 모든 작업이 완료되면, 사용한 자원을 닫아야한다. 이것은 `close()`메소드를 사용하여 수행되며, 자원 누수를 방지한다.

# JPA (Java Persistence API)

> JPA는 'Java EE'에서 제공하는 `ORM(객체-관계 매핑)표준` 이다. 'JPA'를 사용하면 개발자는 데이터베이스 테이블을 자바 객체로 매핑하고, 이 객체들을 통해 데이터베이스와 상호작용할 수 있다. 'JPA'는 데이터베이스 작업을 추상화하고, 개발자가 객체 지향적인 방식으로 데이터에 접근할 수 있게 한다.

## Spring Data JPA

> 'Spring Framework'의 일부로, `JPA를 더 쉽고 편리하게 사용할 수 있게 해주는 모듈`이다. 이는 'JPA'의 기능을 확장하고, 리포지토리 계층의 구현을 단순화하여 개발자가 데이터 접근 로직에 더 집중할 수 있게 도와준다.

### JPA 사용 방법

#### 1. 의존성 추가

> JAP를 사용하기 위해서 'pon.xml'또는 'bunild.gradle' 파일에 JPA구현체의 의존성을 추가해야한다.

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

#### 2. 엔티티 클래스 생성

> 데이터베이스 테이블과 매핑될 자바 클래스(Entity)를 생성한다. 이 클래스에는 `@Entity` 어노테이션을 사용하고, 테이블의 각 컬럼에 해당하는 필드를 정의한다.

##### 주요 어노테이션 사용

- `@Id`: 엔티티의 기본키를 나타낸다.
- `@GeneratedValue`: 기본 키의 생성 전략을 지정한다.
- `@Column`: 필드와 데이터베이스 컬럼을 매핑한다.
- `@ManyToOne, @OneToMany 등`: 관계 매핑을 위한 어노테이션이다.

#### 3. persistence.xml 설정

> JPA 설정을 위해 'META-INF/persistence.xml'파일을 생성하고, 데이터베이스 연결 및 다양한 설정을 정의한다.

#### 4. EntityManager 사용

- `EntityManager`는 JPA의 핵심 인터페이스로, 엔티티의 생명주기를 관리한다.
- `EntityManagerFactory`를 사용하여 'EntityManager'인스턴스를 생성하고, 이를 통해 CRUD 작업을 수행한다.

#### 5. 트랜잭션 관리

> 데이터베이스 작업은 트랜잭션 내에서 수행 되어야한다. `EntityTransaction`객체를 사용하여 트랜잭션을 시작하고, 커밋 또는 롤백한다.

## ❓ JPA를 왜 사용하는가?

1. `ORM(Object-Relational Mapping)`: 개발자는 객체지향적인 방식으로 데이터를 관리할 수 있으며, 복잡한 SQL 쿼리를 작성할 필요가 없다.
2. `데이터베이스 독립성`: JPA는 데이터베이스에 독립적이다. 같은 코드로 다양한 데이터베이스 시스템과 작업할 수 있으며, 필요에 따라 데이터베이스를 교체할 때 코드 변경을 최소화할 수 있다.
3. `표준화`: Java EE 표준이므로, 다양한 JPA 구현체와 호환된다. 이는 코드의 이식성과 유지보수성을 향상 시킨다.
4. `자동화된 데이터베이스 작업`: JPA는 엔티티 상태의 변경을 감지하고, 자동으로 데이터베이스에 반영한다. 이는 개발자가 수동으로 데이터베이스 작업을 관리하는 부담을 줄여준다.
5. `쿼리 언어(JPQL)`: JPA는 JPQL(Java Persistence Query Language)를 제공하여, 데이터베이스 독립적이고 객체 중심적인 쿼리 작성을 가능하게 한다.

## Hibernate

> 'Hibernate'는 `JPA의 구현제` 중 하나로, 데이터베이스와의 상호작용을 관리하는 강력한 ORM 프레임워크이다. 'Hibernate'는 자바 객체와 데이터베이스 테이블 간의 매핑을 자동화하고, SQL 쿼리 생성, 데이터 검색 및 저장, 트랜잭션 관리 등을 처리한다.
