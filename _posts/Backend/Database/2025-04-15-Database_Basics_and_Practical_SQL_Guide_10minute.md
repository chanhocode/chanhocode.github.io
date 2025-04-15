---
title: [TIL] 데이터베이스 기초와 SQL 실전 활용 정리 :10-Minute SQL"
writer: chanho
date: 2025-04-15 09:30:00 +0900
categories: [Backend, Database]
tags: [backend, database]
---

> 데이터베이스를 사용하고 있지만, 깊게 공부한 적은 없는 것 같다는 생각이 들었다. 이에 시간이 날떄 한번씩 10분정도 분량의 SQL 지식에 대해서 A-Z까지 학습을 해보고자 한다. 오늘은 데이터베이스의 기본 개념부터 시작해, SQL 명령어의 분류와 역할, 그리고 PL/SQL의 활용까지 간략하게 살펴보겠다.

## 핵심 포인트
- 데이터베이스는 논리적으로 연관된 테이블들의 집합이다
- 필드, 레코드, 테이블은 관계형 데이터 구조의 기본 단위이다
- SQL 명령어는 DDL, DML, DCL, TCL 네 가지로 분류된다
- 기본키는 유일성을 보장하고, 참조키는 테이블 간 관계를 정의한다
- 인덱스는 검색 속도를 향상시키는 자료구조이다
- PL/SQL은 SQL을 절차형으로 확장하여 업무 로직을 구현할 수 있다
- 함수는 값을 반환하고, 프로시저는 복합 작업을 수행한다
- ERD는 테이블 구조와 관계를 시각적으로 표현하는 설계 도구이다

## 데이터베이스 구성 요소 이해
> 데이터베이스를 구성하는 구조적 단위를 이해하면, 전체 시스템의 설계를 보다 효율적으로 할 수 있다.

### 핵심 용어 정리
- **데이터(Data)**: 유무형의 정보를 구성하는 기본 단위
- **필드(Field)**: 데이터에 속성값을 부여하는 열(Column)
- **레코드(Record)**: 하나의 개체에 대한 여러 필드값의 집합 (행)
- **테이블(Table)**: 동일한 형식의 레코드가 모인 구조
- **데이터베이스(Database)**: 논리적으로 연관된 테이블들의 모음
- **DBMS**: 복잡한 SQL을 대신하여 데이터를 효율적으로 관리하는 시스템

### 주요 데이터 형식
| 타입 | 설명 |
|------|------|
| `NUMBER` | 숫자형 데이터 |
| `DATE` | 날짜 및 시간 데이터 |
| `CHAR(n)` | 고정 길이 문자열 |
| `VARCHAR(n)` | 가변 길이 문자열 |

## SQL 명령어 체계화
> 데이터베이스 조작을 위한 SQL 명령어는 목적에 따라 네 가지 유형으로 분류된다.

| 분류 | 명령어 | 설명 |
|------|--------|------|
| DDL (Data Definition Language) | `CREATE`, `ALTER`, `DROP` | 테이블 등 데이터 구조 정의 및 변경 |
| DML (Data Manipulation Language) | `SELECT`, `INSERT`, `UPDATE`, `DELETE` | 데이터 조회 및 조작 |
| DCL (Data Control Language) | `GRANT`, `REVOKE` | 사용자 권한 제어 |
| TCL (Transaction Control Language) | `COMMIT`, `ROLLBACK` | 트랜잭션 처리 및 복구 |

## 데이터 무결성과 성능 향상
> 단순한 데이터 저장을 넘어, 무결성과 성능을 동시에 확보하는 구조 설계가 중요하다.

### 기본키와 참조키
- **기본키**: 테이블 내에서 각 레코드를 고유하게 식별하는 필드 (중복 불가, NULL 불가)
- **참조키**: 다른 테이블의 기본키를 참조하여 관계를 설정하는 필드

### 인덱스(Index)
- 검색 속도 향상을 위한 보조 구조
- 자주 조회되는 필드에 인덱스를 설정하면 성능이 대폭 개선됨

```sql
CREATE INDEX idx_student_name ON student(name);
```

## PL/SQL을 통한 로직 구현
> PL/SQL은 SQL을 프로그래밍 언어 수준으로 확장하여 복잡한 로직과 트랜잭션을 처리할 수 있도록 지원한다.

### 함수 vs 프로시저

| 항목 | 함수(Function) | 프로시저(Procedure) |
|------|----------------|---------------------|
| 반환값 | O | 선택적 또는 없음 |
| 사용 목적 | 계산 및 값 반환 | 트랜잭션, 데이터 처리 로직 수행 |
| 예시 | `SUBSTR('데이터', 1, 2)` → '데' | 주문 등록, 사용자 등록 처리 |

### 예시: 프로시저 정의
```sql
CREATE OR REPLACE PROCEDURE register_student (
  p_id IN NUMBER,
  p_name IN VARCHAR2
)
IS
BEGIN
  INSERT INTO student (student_id, name) VALUES (p_id, p_name);
  COMMIT;
END;
```

## 실전 예제: 학생-강의-수강 시스템
> 앞서 학습한 내용을 바탕으로, 간단한 학사 관리 시스템을 설계하고 구현하는 과정을 살펴본다.

### ERD 기반 테이블 설계

- **STUDENT** (학생)
- **COURSE** (강의)
- **ENROLLMENT** (수강 관계)

```sql
CREATE TABLE student (
  student_id NUMBER PRIMARY KEY,
  name VARCHAR2(100),
  major VARCHAR2(50)
);

CREATE TABLE course (
  course_id NUMBER PRIMARY KEY,
  title VARCHAR2(100),
  credit NUMBER
);

CREATE TABLE enrollment (
  student_id NUMBER,
  course_id NUMBER,
  enroll_date DATE,
  PRIMARY KEY (student_id, course_id),
  FOREIGN KEY (student_id) REFERENCES student(student_id),
  FOREIGN KEY (course_id) REFERENCES course(course_id)
);
```

### 데이터 입력 및 조회 예시
```sql
-- 데이터 삽입
INSERT INTO student VALUES (1, '김철수', '컴퓨터공학');
INSERT INTO course VALUES (101, '데이터베이스', 3);
INSERT INTO enrollment VALUES (1, 101, SYSDATE);

-- 학생의 수강 강의 조회
SELECT s.name, c.title, e.enroll_date
FROM enrollment e
JOIN student s ON e.student_id = s.student_id
JOIN course c ON e.course_id = c.course_id
WHERE s.name = '김철수';
```

## 트랜잭션과 예외 처리
> 데이터베이스 작업의 신뢰성을 보장하기 위해, 트랜잭션 처리와 예외 처리 구조가 필요하다.

```sql
BEGIN
  INSERT INTO student VALUES (2, '이영희', '전자공학');
  INSERT INTO enrollment VALUES (2, 101, SYSDATE);
  COMMIT;
EXCEPTION
  WHEN OTHERS THEN
    ROLLBACK;
END;
```
