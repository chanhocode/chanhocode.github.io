---
title: "[SpringDB] 커넥션 풀(Connection Pool)과 DataSource에 대해서"
writer: chanho
date: 2024-03-06 15:00:00 +0900
categories: [Backend, Spring]
tags: [backend, spring, database]
---

커넥션 풀(Connection Pool)과 데이터 소스(DataSource)는 데이터베이스 연결과 관련된 기술이다. 이 기술은 일반적으로 데이터베이스와의 통신을 관리하는 데 사용된다.

# 커넥션 풀(Connection Pool)에 대해서

> 커넥션 풀(Connection Pool)은 데이터베이스와 같은 자원에 대한 연결을 관리하는 기술이다. `데이터베이스에 연결하는 과정은 시간과 자원을 많이 소모하기 때문에, 매번 연결을 새로 생성하는 것은 비효율` 적이다 커네션 풀은 `이 문제를 해결하기 위해 미리 여러 개의 연결을 생성하고 풀(Pool)에 저장한다.`

<img width="722" alt="image" src="https://github.com/chanhocode/chanhocode.github.io/assets/105937460/82adf834-f1ee-4790-8d31-ee7783aecd70">

어플리케이션이 데이터베이스에 접근할 필요가 있을 때, 커넥션 풀에서 미리 생성된 연결을 가져와 사용한다. 사용이 끝난 연결은 다시 풀에 반환되어 다른 요청에 재사용된다. 여기서 `주의할 점은 커넥션을 종료하는 것이 아니라 커넥션이 살아있는 상태로 커넥션 풀에 반환`한다. 이 과정은 자원을 효율적으로 사용할 수 있게 하며, 연결 생성과 소멸에 소요되는 시간을 줄여 성능을 향상시킨다.

# DataSource

> 데이터 소스는 `데이터베이스 연결을 생성하기 위한 정보와 로직을 캡슐화한 객체`이다. 일반적으로 JDBC와 같은 데이터베이스 연결 기술에서 사용된다. 데이터 소스는 데이터베이스 서버의 URL, 사용자 이름, 비밀번호와 같은 연결 정보를 포함하여, 필요에 따라 연결을 생성하고 관리한다. 데이터 소스는 종종 커넥션 풀과 함께 사용되어, 애플리케이션에서 필요한 연결을 커넥션 풀로부터 획득하는 역할을 한다.

- `연결 정보의 캡슐화`: 데이터베이스 URL, 사용자 이름, 비밀번호 등 연결에 필요한 정보를 캡슐화 한다.
- `연결 관리`: 데이터베이스 연결 및 연결 해제를 관리한다. 이는 커넥션 풀과 함께 사용될 때 특히 중요하다.
- `커넥션 풀 지원`: 대부분의 데이터 소스 구현체는 내부적으로 커넥션 풀을 사용해 연결을 관리한다. 이는 어플리케이션 성능을 향상시키는 데 도움이 된다.

# HiKariCP에 대해 알아보자

> HikariCP는 자바 어플리케이션에서 널리 사용되는 커넥션 풀 라이브러리이다. 이 라이브러리는 뛰어난 성능과 간결한 코드, 적은 오버헤드로 인기가 있다.

### 사용하기

- `의존성 추가`

```
<!-- Maven을 사용하는 경우 -->
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>버전</version>
</dependency>

// Gradle을 사용하는 경우
dependencies {
    implementation 'com.zaxxer:HikariCP:버전'
}
```

- `HikariCP 설정`: 'HikariConfig' 또는 'HikariDataSource' 객체를 설정한다. 여기서는 데이터베이스 연결에 필요한 정보(JDBC URL, 사용자 이름, 비밀번호 등)와 풀 관련 설정(최대 풀 크기, 커넥션 타임아웃 등)을 지정한다.

```java
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:mysql://localhost:3306/db");
config.setUsername("username");
config.setPassword("Password");

// 추가 설정
config.addDataSourceProperty("cachePrepStmts", "true");
config.addDataSourceProperty("prepStmtCacheSize", "250");
config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");

config.setMaximumPoolSize(10); // 최대 풀 크기 설정
config.setMinimumIdle(5); // 최소 유휴 커넥션 수
config.setConnectionTimeout(30000); // 커넥션 타임아웃
config.setIdleTimeout(600000); // 유휴 커넥션의 수명
config.setMaxLifetime(1800000); // 커넥션의 최대 수명

// HikariDataSource 생성
HikariDataSource dataSource = new HikariDataSource(config);
```

1. cachePrepStmts: Prepared Statements의 캐싱을 활성화다. Prepared Statements는 데이터베이스 서버에 사전 컴파일되어 재사용될 수 있기 때문에, 이 옵션을 'true'로 설정하면 성능이 향상될 수 있다.
2. prepStmtCacheSize: 커넥션 당 Prepared Statements 캐시의 크기를 설정한다. 이 값은 캐실할 수 있는 준비된 문의 최대 개수를 나타낸다. 예를 들어 '250'으로 설정하면, 각 커넥션은 최대 250개의 Prepared Statements를 캐시할 수 있다.
3. prepStmtCacheSqlLimit: 이 속성은 캐시에 저장할 수 있는 개별 Prepared Statements의 최대 길이를 설정한다. SQL문이 이 값 보다 길면 캐시되지 않는다. 예를 들어 '2048'로 설정시 최대 2048자 길이의 SQL문을 캐시할 수 있다.
