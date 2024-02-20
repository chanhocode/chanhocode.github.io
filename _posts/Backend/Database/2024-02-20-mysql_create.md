---
title: "[MySQL] 기본 조작하기 _ DB, TABLE, 데이터, 자료형"
writer: chanho
date: 2024-02-20 18:50:00 +0900
categories: [Backend, Database]
tags: [backend, mysql]
---

# 데이터베이스 생성 및 삭제

## 데이터 베이스 생성

```sql

$ CREATE DATABASE 'database_name';
-- 또는
$ CREATE SCHEMA 'database_name;'
```

> 'CREATE SCHEMA' 명령어와 'CREATE DATABASE'명령어는 'MySQL'에서 새로운 데이터베이스를 생성하는 기능을 수행한다. 그러나 'CREATE SCHEMA' 명령어는 데이터베이스를 생성하는 것뿐만 아니라 데이터베이스 스키마(구조)를 정의하는 데 좀 더 널리 사용되는 SQL 표준 용어 이다.

'CREATE DATABASE' 명령어는 단순히 새로운 데이터베이스를 생성하는 데 사용되는 반면 'CREATE SCHEMA' 명령어는 데이터베이스뿐만 아니라 해당 데이터베이스의 초기 구조(테이블, 뷰, 인덱스 등)를 정의하는 데 사용될 수 있다. 즉, 기능적으로는 동일하지만 'CREATE SCHEMA'는 좀 더 광범위한 사용이 가능하다.

### 문자셋과 정렬

```sql
$ CREATE SCHEMA `database_name` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

| 항목               | 설명                                                                      |
| ------------------ | ------------------------------------------------------------------------- |
| utf8mb4            | 한글을 포함한 전세계의 문자와 이모티콘 사용이 가능하다.                   |
| utf8mb4_general_ci | 가장 정확하지는 않지만 정렬 속도가 빠르다.                                |
| utf8mb4_bin        | 이진 비교를 수행하는 'utf8mb4'의 정렬 규칙이다. 문자열을 정확히 비교한다. |

[더많은 설정 알아보기](https://dev.mysql.com/doc/refman/8.0/en/charset-mysql.html)

## 데이터베이스 삭제

```sql
$ DROP DATABASE 'database_name';
```

## 데이터베이스 사용하기

```sql
$ use 'database_name';
```

# 자료형

## 숫자 자료형

### 정수형

| 자료형    | Byte | SIGNED                         | UNSIGNED          |
| --------- | ---- | ------------------------------ | ----------------- |
| `TINYINT` | 1    | -128 ~ 127                     | 0 ~ 255           |
| SMALLINT  | 2    | -32,768 ~ 32,767               | 0 ~ 65,535        |
| MEDIUMINT | 3    | -8,388,608 ~ 8,338,607         | 0 ~ 16,777,215    |
| `INT`     | 4    | -2,147,483,648 ~ 2,147,483,647 | 0 ~ 4,294,967,295 |
| BIGINT    | 8    | -2^63 ~ 2^63-1                 | 0 ~ 2^64-1        |

### 고정 소수점 : 정확한 값 표현

- DICIMAL(실수 부분 총 자릿수'최대 65', 소수 부분 자릿수)

### 부동 소수점 : 넓은 범위의 수, 부정확하지만 일반적으로는 충분히 정확하다.

- FLOAT
- DOUBLE (더 넓은 범위)

## 문자 자료형

### 문자열

| 자료형     | 설명        | 최대 바이트 |
| ---------- | ----------- | ----------- |
| CHAR(s)    | 고정 사이즈 | 255         |
| VARCHAR(s) | 가변 사이즈 | 65,535      |

#### 비교

|Value|CHAR(4)|StorageRequired|VARCHAR(4)|StorageRequired|
|''|'\_\_\_\_'|4bytes|''|1byte|
|'ab'|'ab\_\_'|4bytes|'ab'|3byte|

### 텍스트

| 자료형     | 최대 바이트 크기 |
| ---------- | ---------------- |
| TINYTEXT   | 255              |
| TEXT       | 65,535           |
| MEDIUMTEXT | 16,777,215       |
| LONGTEXT   | 4,294,967,295    |

## 시간 자료형

|자료형|설명|
|DATE|YYYY-MM-DD|
|TIME|HHH:MI:SS|
|DATETIME|YYYY-MM-DD HH:MI:SS|
|TIMESTAMP|YYYY-MM-DD HH:MI:SS|

# 테이블 생성 및 수정, 삭제하기

## 테이블 생성

```sql
CREATE TABLE tablename (
  id INT,
  p_name VARCHAR(10),
  p_age TINYINT
);
```

### 제약 조건

| 제약           | 의미                                                         |
| -------------- | ------------------------------------------------------------ |
| AUTO_INCREMENT | 새 행 생성시 자동으로 1씩 증가                               |
| PRIMARY KEY    | 중복 입력 불가, NULL 불가, 기본키 설정, 테이블당 하나만 가능 |
| UNIQUE         | 중복 입력 불가                                               |
| NOT NULL       | NULL 입력 불가                                               |
| UNSIGNED       | 양수만 가능(숫자 타입)                                       |
| DEFAULT        | 값 입력이 없을시 기본값 설정                                 |

## 테이블 수정

```sql
-- 테이블 명 변경
$ ALTER TABLE tablename RENAME TO tablename2;

-- 컬럼 자료형 변경
$ CHANGE COLUMN id id TINYINT;

-- 컬럼명 변경
$ CHANGE COLUMN p_name p_nickname VACHAR(10);

-- 컬럼 제거
$ DROP COLUMN age;

-- 컬럼 추가
$ ADD COLUMN birthday DATE AFTER p_name;
```

## 테이블 삭제

```sql
$ DROP TABLE tablename2;
```

# 데이터 추가 및 변경, 삭제

## 데이터 추가

```sql
$ INSET INTO tablename (id, p_name, p_age) VALUES (1, 'siru', 20);
```

## 데이터 변경

```sql
$ UPDATE tablename SET columnName = 'update' WHERE 조건
```

## 데이터 삭제

```sql
$ DELETE FROM tablename WHERE 조건;

-- 테이블 초기화
$ TRUNCATE tablename;
```

### TRUNCATE

> 테이블의 모든 데이터를 빠르게 제거하는 데 사용된다. 해당 명령어는 'DELETE'보다 빠르지만, 조건을 지정할 수 없다. 테이블의 모든 레코드를 단순하고 빠르게 삭제한다. 하지만, 테이블의 구조와 속성들은 그대로 유지한다.
