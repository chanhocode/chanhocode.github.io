---
title: "[Mybatis] 마이바티스란 무엇이고 간단한 사용법에 대해서"
writer: chanho
date: 2024-03-13 01:40:00 +0900
categories: [Backend, Spring]
tags: [backend, spring, database, mybatis]
---

`MyBatis`란 자바 프로그래밍 언어용 `데이터베이스 맵핑 툴(SQL Mapper)` 이다. 이를 사용하여, SQL 쿼리를 보다 쉽게 데이터베이스에 연결하고 실행할 수 있다. 'MyBatis'를 사용하는 주된 이유는 SQL 문을 프로그램 코드와 분리하여 관리할 수 있다는 점이다. 이렇게 하면, SQL 문을 좀더 유연하게 관리하고, 코드의 가독성과 유지 보수성을 향상시킬수 있다. 예를 들어, 데이터베이스 테이블에 데이터를 추가, 조회, 수정, 삭제하는 작업을 SQL 파일에 별도로 작성하고, 이 SQL 파일을 MyBatis를 통해 자바 코드와 연결한다.

> 'MyBatis'의 가장 큰 장점은 SQL을 XML에 편리하게 작성할 수 있고 또 동적 쿼리를 매우 편리하게 작성할 수 있다는 점이다.

`💡 SQL Mapper: SQL을 직접 작성해 해당 SQL문과 Object의 필드를 매핑하여 데이터를 다루는 Persistence Framework이다.`

- Persistence: 프로그램이 종료되더라도 사라지지 않는 데이터 특성을 말한다.
- Persistence Framework(영속성 프레임워크): 데이터베이스와 연동되는 시스템을 빠르게 개발하고 안정적인 구동을 보장해주는 프레임워크

# MyBatis 기본 동작

1. `SQL 매핑 파일 생성`: SQL 문과 이를 처리할 매핑 정보를 XML 파일이나 어노테이션 형태로 작성한다.
2. `설정 파일 작성`: 데이터베이스 연결 정보와 매핑 파일의 위치 등의 설정을 포함한다.
3. `세션 생성`: MyBatis는 SQL을 실행하기 위해 'SqlSession'이라는 객체를 사용한다. 이 객체는 설정 파일을 기반으로 생성된다.
4. `SQL 실행` SqlSession을 사용하여 SQL 매핑 파일에 정의된 SQL문을 실행한다.

# MyBatis 설정하기

## 의존성 추가

```
// Maven
<dependencies>
    <!-- MyBatis -->
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.x.x</version>
    </dependency>
    <!-- 데이터베이스 드라이버 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.x.x</version>
    </dependency>
    <!-- 기타 필요한 의존성 추가 -->
</dependencies>

// Gradle
 dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-jdbc' //MyBatis 추가
    implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:2.2.0'
}

```

## 데이터 소스 설정

> 데이터베이스 연결에 필요한 DataSource를 설정한다. 이는 데이터베이스 URL, Username, Password 등의 정보를 포함한다. Spring Boot를 사용하는 경우 'application.properties'또는 'application.yml'파일에 이러한 속성을 정의할 수 있다.

```properties
spring.datasource.url=jdbc:h2:tcp://localhost/~/hello
spring.datasource.username=sa
spring.datasource.password=

#MyBatis
mybatis.type-aliases-package=hello.service.domain
mybatis.configuration.map-underscore-to-camel-case=true
```

- mybatis.type-aliases-package
  - 마이바티스에서 타입 정보를 사용하기 위해서 패키지 이름을 명시 해야한다. 해당 속성에 명시하면 패키지 이름을 생략할 수 있다.
  - 여러 위치를 지정하려면 `,` 또는 `;`로 구분하면 된다.
- mybatis.configuration.map-underscore-to-camel-case
  - 언더바(\_)를 카멜로 자동 변경해주는 기능을 활성화 한다.

## MyBatis 설정: `SqlSessionFactoryBean` 구성

> MyBatis의 SQL 세션 팩토리를 설정한다. 이는 MyBatis의 'SqlSessionFactoryBean'을 사용하여 구성된다. 'SqlSessionFactoryBean'은 MyBatis 설정 파일의 경로, 데이터 소스, 마이바티스 매퍼 파일의 위치 등을 설정한다.

`💡 'mybatis-spring-boot-starter'를 사용할 경우 'SqlSessionFactory'는 자동으로 구성되지만, 특정 경우에 개발자가 추가적인 설정을 제공할 필요가 있을 수 있다. 이 경우, Java구성 클래스에서 '@Bean' 어노테이션을 사용하여 'SqlSessionFactroy'빈을 명시적으로 정의할 수 있다.`

### Spring과 통합된 MyBatis 설정하기

```java
@Configuration // 스프링 구성 파일 명시
public class MyBatisConfig {
  @Autowired
  private DataSource dataSource; // 데이터베이스 연결 정보(URL, USERNAME ...)를 포함한 객체

  @Bean
  public SqlSessionFactory sqlSessionFactory() throws Exception {
    // SqlSessionFactoryBean: 'SqlSessionFactory'를 생성하기 위한 헬퍼 클래스, 이 객체는 SQL 세션을 생성하고, MyBatis와 데이터베이스 간의 상호 작용을 관리
    SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();

    sessionFactory.setDataSource(dataSource); // DataSource 설정, 데이터베이스 연결을 관리
    sessionFactory.setMapperLocations(
      new PathMatchingResoucePatternResolver().getResouces("classpaht:mappers/*.xml")
    ); // 매퍼 파일들의 위치를 지정, 이 파일들은 SQL 쿼리와 자바메소드를 매핑하는 데 사용
    return sessionFactory.getObject();
  }
}
```

- `Spring 프레임워크와의 통합`: 이 설정은 Spring의 의존성 주입 기능을 활용하여 'DataSource'를 자동으로 주입하고, Spring 컨테이너에 'SqlSessionFactory'빈을 등록한다.
- `매퍼 위치 설정`: MyBatis 매퍼 파일들의 위치를 지정한다. 이를 통해 SQL 매핑이 이루어진다.

[추가: XML으로 설정 또는 Java설정](https://mybatis.org/mybatis-3/ko/getting-started.html)

## 매퍼 인터페이스 스캔: `@MapperScan` 사용

> '@MapperScan' 어노테이션은 MyBatis 매퍼 인터페이스가 위치한 패키지를 지정한다. 이렇게 하면 Spring이 해당 패키지를 스캔하여 매퍼 인터페이스를 자동으로 찾아 빈으로 등록한다.

```java
@SpringBootApplication
@MapperScan("com.hello.mapper")
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(hello.class, args);
    }
}
```

## 트랜잭션 관리자 설정: 'DataSourceTransactionManager'사용

```java
@Configuration
public class TransactionConfig {
  @Autowired
  private DataSource dataSource;

  @Bean
  public DataSourceTransactionManager transactionManager() {
    return new DataSourceTransactionManager(dataSource);
  }
}
```

> `DataSourceTransactionManager`: 이 클래스는 JDBC기반의 데이터베이스 트랜잭션 관리를 위한 Spring의 구현체이다. 이 객체는 주어진 'DataSource'를 사용하여 트랜잭션을 관리한다.

# MyBatis 적용하기

## 데이터 저장 예시를 활용한 이해

### Mapper Interface

> MyBatis에서는 SQL 쿼리와 자바 메소드를 연결하는 매퍼 인터페이스를 정의한다.

```java
package com.hello.mapper;

public interface HelloMapper {
  void save(User user);
}
```

### Mapper XML

> 매퍼 인터페이스의 각 메소드에 해당하는 SQL 쿼리를 정의하는 XML 파일을 작성한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.hello.mapper.HelloMapper">
  <insert id="save">
    INSERT INTO users (name, email) VALUES (#{name}, #{email})
  </insert>
</mapper>
```

> 해당 파일은 'resource/mappers' 디렉토리에 저장되며, 이전에 설정한 'setMapperLocations'에서 이 위치를 지정하였다.

### 서비스 클래스

> 매퍼 인터페이스를 사용하여 실제 비즈니스 로직을 구현하는 서비스 클래스를 작성한다.

```java
@Service
public class HelloService {
  @Autowired
  private HelloMapper helloMapper;

  public void saveUser(User user) {
    helloMapper.save(user);
  }
}
```

### 컨트롤러

> 최종적으로, 사용자의 요청을 받아 서비스 로직을 호출하는 컨트롤러를 작성한다.

```java
@RestController
@RequestMapping("/users")
public class HelloController {
  @Autowired
  private HelloService helloService;

  @PostMapping
  public void saveUser(@RequestBody User user) {
    helloService.saveUser(user);
  }
}
```

### 전체 흐름

1. 클라이언트는 '/users'경로로 POST 요청을 보낸다.
2. 컨트롤러가 요청을 받고, 'HelloService'의 'saveUser'메소드를 호출한다.
3. 'HelloService'는 'HelloMapper'의 'save' 메소드를 호출하여 데이터베이스에 사용자 정보를 저장한다.
4. 'save'메소드는 매퍼 XML에 정의된 SQL 쿼리를 실행한다.

## 기본 파라미터 바인딩: `#{}`

> '#{}' 구문은 SQL 쿼리 내에서 파라미터를 바인딩하는 데 사용된다. 이 구문을 사용하면 MyBatis가 자동으로 파라미터 값을 SQL 쿼리에 안전하게 삽입한다.

```xml
<select id="selectUser" resultType="User">
  SELECT * FROM users WHERE id = #{id}
</select>
```

해당 코드에서 '#{id}'는 메소드 호출 시 제공된 파라미터의 'id'값을 SQL 쿼리에 바인딩한다.

## 동적 SQL 구문

> MyBatis는 동적 쿼리를 작성할 수 있는 여러 XML 태그를 제공한다. 이를 사용하면 SQL 쿼리를 보다 유연하게 작성할 수 있다.

### `<if>`

: 특정 조건이 참일 때만 SQL의 일부를 포함시키고 싶을 때 사용한다.

```xml
<select id="selectUsersByName" resultType="User">
  SELECT * FROM users
  <where>
    <if test="name != null">
      name = #{name}
    </if>
  </where>
</select>
```

> 여기서, 'if test="name != null"'는 name 파라미터가 null이 아닐 경우에만 해당 조건을 쿼리에 포함시킨다.

### `<where>`

: SQL 쿼리의 'WHERE'절을 동적으로 구성한다. 'where'태그는 자동으로 앞에 오는 'AND'또는 'OR'키워드를 관리한다.

```xml
<select id="findUser" resultType="User">
  SELECT * FROM users
  <where>
    <if test="name != null">
      name = #{name}
    </if>
    <if test="email != null">
      AND email = #{email}
    </if>
  </where>
</select>
```

> 여기서 'where'태그는 필요한 경우에만 'whrer'절을 추가하고, 각 'if'조건 앞의 'AND'나 'OR'을 적절하게 관리한다.

### `<choose>, <when>, <otherwise>`

: 'if-else' 또는 'switch'문과 유사하게 사용할 수 있다. 여러 조건 중 하나를 선택하여 실행한다.

```xml
<select id="findUserByStatus" resultType="User">
  SELECT * FROM users
  <where>
    <choose>
      <when test="status == 'ACTIVE'">
        status = 'ACTIVE'
      </when>
      <when test="status == 'INACTIVE'">
        status = 'INACTIVE'
      </when>
      <otherwise>
        status = 'PENDING'
      </otherwise>
    </choose>
  </where>
</select>
```

> 여기서 'choose'태그는 제공된 조건들 'when'중 하나를 선택하고, 해당 조건이 없을 경우 'otherwise'절을 사용한다.

### `<foreach>`

: 컬렉션을 반복처리할 때 사용한다. 주로 'IN'절을 동적으로 생성할 때 유용하다.

```xml
<select id="selectUsersInList" resultType="User">
  SELECT * FROM users
  WHERE id IN
  <foreach item="item" index="index" collection="list"
           open="(" separator="," close=")">
    #{item}
  </foreach>
</select>
```

> 여기서 'foreach'태그는 주어진 'list' 컬렉션의 각 요소에 대해 반복적으로 SQL 조각을 생성한다.
