---
title: "[SpringDB] 트랜잭션 매니저(Transaction Manager)에 대해서"
writer: chanho
date: 2024-03-12 22:00:00 +0900
categories: [Backend, Spring]
tags: [backend, spring, database]
---

스프링이 트랜잭션 매니저(Transaction Manager)를 사용하는 주된 이유는 복잡한 트랜잭션 관리를 단순화하고, 다양한 데이터베이스 환경에 적용할 수 있는 유연한 해결책을 제공하기 위함이다.

- 복잡성 관리: 데이터베이스 연산이 복잡해질수록 트랜잭션 관리의 복잡성이 증가한다. 트랜잭션 매니저는 이를 `추상화`하여 사용자가 간편하게 트랜잭션을 관리할 수 있게 돕는다.
- 플랫폼 독립성: 스프링은 다양한 종류의 트랜잭션 매니저를 지원하여, 특정 데이터베이스 기술이나 플랫폼에 종속되지 않는다.

# 트랜잭션 매니저(Transaction Manager)

> 스프링 프레임워크에서 트랜잭션 매니저는 데이터베이스 작업을 효과적으로 관리하고, 일관성을 유지하는 데 중요한 역할을 한다. 트랜잭션은 여러 데이터베이스 연산들이 모두 성공적으로 수행되거나, 그렇지 않을 경우 모두 취소되어야 하는 하나의 작업 단위이다.

## 추상화와 트랜잭션 매니저

<img width="1091" alt="image" src="https://github.com/chanhocode/chanhocode.github.io/assets/105937460/25f6bce0-c46f-4951-9501-c7e3ff1e8e22">

> 스프링 트랜잭션 매니저는 데이터베이스 작업에 대한 트랜잭션 관리를 추상화 한다. 이 추상화는 개발자가 구체적인 트랜잭션 관리 기술이나 데이터베이스 `기술에 직접 종속되지 않도록` 해주며, 다양한 트랜잭션 관리 API(ex: JDBC, JPA, Hibernate ...)와 호환될 수 있도록 해준다.

```java
public interface PlatformTransactionManager extends TransactionManager {
     TransactionStatus getTransaction(@Nullable TransactionDefinition definition)
             throws TransactionException;
     void commit(TransactionStatus status) throws TransactionException;
     void rollback(TransactionStatus status) throws TransactionException;
}
```

- `getTransaction`: 트랜잭션을 시작한다.
- `commit`: 트랜잭션을 커밋한다.
- `rollbace`: 트랜잭션을 롤백한다.

## 동기화와 트랜잭션 매니저

> 트랜잭션 매니저는 크게 앞서 설명한 트랜잭션 추상화와 리소스 동기화 역할을 한다. 스프링은 트랜잭션 매니저를 통해 데이터베이스 연결과 트랜잭션 상태를 동기화한다. 이는 트랜잭션 내에서 실행되는 모든 데이터베이스 작업이 동일한 데이터베이스 연결과 트랜잭션을 공유하도록 보장한다.

- 트랜잭션 동기화 매니저
  > 스프링은 `TransactionSynchronizationManager`라는 클래스를 통해 `현재 스레드에 바인딩된 리소스와 트랜잭션 정보를 관리`한다. 이를 통해 트랜잭션 경계 내에서 리소스(ex: 데이터베이스 연결 ...)의 동기화가 이루어진다.

### 동작 방식

1. `리소스 바인딩`: 현재 스레드에 데이터베이스 연결과 같은 리소스 바인딩(데이터 소스를 통해 커넥션을 만들고 트랜잭션 시작)을 한다. 이를 통해 트랜잭션을 수행하는 동안 동일한 데이터베이스 연결이 사용되도록 보장한다.
2. `트랜잭션 상태 관리`: 매니저는 현재 트랜잭션의 상태 정보도 관리한다. 이는 트랜잭션이 활성화되어 있는지, 어떤 종류의 트랜잭션이 실행 중인지 등의 정보를 포함할 수 있다.
3. `스레드 로컬 저장소 사용`: 스레드 로컬 저장소를 사용하여 각 스레드 컨텍스트에 리소스와 트랜잭션 정보를 저장한다. 이는 서로 다른 스레드 간에 데이터가 공유되지 않도록 하여, 각 스레드에서 수행되는 트랜잭션이 다른 스레드의 트랜잭션과 독립적으로 작동할 수 있도록한다.
4. `트랜잭션 동기화 콜백`: 트랜잭션 동안 특정 이벤트(시작, 완료, 롤백 ...)가 발생할 때, 'TransactionSynchronizationManager'는 등록된 동기화 콜백(TransactionSynchronization)을 호출할 수 있다. 이를 통해 트랜잭션의 생명주기에 따른 추가적인 작업을 수행할 수 있다.

## 트랜잭션 매니저의 역할

1. `일관성 유지`: 데이터베이스의 일관성을 유지하기 위해 모든 작업이 성공적으로 완료되거나, 실패 시 롤백이 이루어져야 한다.
2. `원자성 보장`: 트랜잭션 내의 모든 연산은 하나의 단위로 간주되어야 하며, 이들은 전부 실행되거나 전부 실행되지 않아야 한다.
3. `고립성 제공`: 동시에 실행되는 다른 트랜잭션의 작업으로 부터 독립적이어야 하며, 상호 간섭을 방지해야 한다.
4. `지속성 확보`: 트랜잭션이 성공적으로 완료되면, 그 결과는 영구적으로 반영되어야 한다.

## 제공하는 트랜잭션 매니저의 종류

1. `DataSourceTransactionManager`: JDBC 기반의 데이터베이스와 함께 사용된다.
2. `HibernateTransactionManager`: Hibernate를 사용하는 애플리케이션에 적합하다.
3. `JpaTransactionManager`: JPA 기반의 데이터 접근을 위해 설계되었다.
4. `JtaTransactionManager`: 분산 트랜잭션을 관리하기 위해 사용된다.

## 트랜잭션 매니저 사용하기

### 선언적 트랜잭션 관리

> 스프링에서 선언적 트랜잭션 관리를 하기 위해서는 `@Transactional` 어노테이션을 사용하여 메소드 또는 클래스 레벨에서 트랜잭션 경계를 정의할 수 있다. 이 접근법을 사용하면, 스프링이 자동으로 트랜잭션 관리를 처리하며, 개발자는 트랜잭션 관리 로직을 직접 작성할 필요가 없다.

```java
@Transactional
public void someDataServiceMethod() {
  // 데이터 액세스 코드
}
```

이 경우, 스프링은 이 메소드가 실행될 때 트랜잭션을 시작하고, 메소드 실행이 완료되면 트랜잭션을 커밋하거나 예외 발생 시 롤백한다.

`@Transactional`은 스프링프레임워크에서 제공하는 어노테이션으로, 선언적 트랜잭션 관리를 가능하게 한다. 이 어노테이션을 사용하면 메소드 또는 클래스 레벨에서 트랜잭션의 경게를 설정할 수 있으며, 스프링이 자동으로 해당 메소드의 실행을 트랜잭션으로 래핑합니다. 이를 통해 개발자는 트랜잭션 관리를 위한 복잡한 코드 작성없이 일관성을 유지할 수 있다.

> `@Transactional`어노테이션을 사용할 때는 스프링의 AOP(Aspect-Oriented Programming)기능이 활성화되어 있어야 하며, 트랜잭션 매니저가 올바르게 구성되어 있어야 한다. 이를 통해 데이터 접근 로직의 안정성을 높이면서, 개발자는 비즈니스 로직에 더 집중할 수 있다.

### 프로그래밍 방식 트랜잭션 관리(DataSourceUtils) 사용

> `DataSourceUtils.getConnection()`와 `DataSourceUtils.releaseConnection()`은 데이터 소스로부터 직접 연결을 얻고 해제하는 데 사용된다. 이 메소드들은 주로 프로그래매틱한 트랜잭션 관리에서 사용되며, 선언적 트랜잰션 관리를 사용하지 않는 경우 유용하다.

```java
Connection con = null;
try {
  con = DataSourceUtils.getConnection(dataSource);
} catch (SQLException ex) {
  // 예외 처리
} finally {
  DataSourceUtils.releaseConnection(con, dataSource);
}
```
