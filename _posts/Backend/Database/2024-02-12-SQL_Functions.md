---
title: "[SQL] 연산자 및 함수들"
writer: chanho
date: 2024-02-12 18:44:00 +0900
categories: [Backend, Database]
tags: [backend, sql]
---

# SQL의 연산자

| 연산자                      | 의미                             |
| --------------------------- | -------------------------------- |
| +,-,\*,/                    | 더하기, 빼기, 곱하기, 나누기     |
| %, MOD                      | 나머지                           |
| IS                          | 양쪽이 모두 TRUE 또는 FALSE      |
| IS NOT                      | 한쪽은 TRUE 한쪽은 FALSE         |
| AND, &&                     | 양쪽이 모두 TRUE                 |
| OR, \|\|                    | 한쪽만 TRUE여도 TRUE             |
| =                           | 양쪽 값이 동일                   |
| !=, <>                      | 양쪽 같이 다름                   |
| >, <                        | (왼쪽, 오른쪽) 값이 큼           |
| >=, <=                      | (왼쪽, 오른쪽) 값이 같거나 더 큼 |
| BETWEEN {MIN} AND {MAX}     | 두 값 사이에 있음                |
| NOT BETWEEN {MIN} AND {MAX} | 두 값 사이가 아닌 곳에 있음      |
| IN(...)                     | 괄호 안의 값들 가운데 있음       |
| NOT IN(...)                 | 괄호 안의 값들 가운데 없음       |
| LIKE '...%...'              | 0~N개 문자를 가진 패턴           |
| LIKE '...\_...'             | \_ 갯수 만큼의 문자를 가진 패턴  |

## 사칙 연산

```sql
// (1)
SELECT 1 + 2;
// (2)
SELECT 2 - 1 AS Minus;
// (3)
SELECT 'A' + 1;
// (4)
SELECT '1' + 3;
```

- 1번

| 1+2 |
| --- |
| 3   |

- 2번

| Minus |
| ----- |
| 1     |

- 3번
  > 문자는 0으로 인식

| 'A'+1 |
| ----- |
| 1     |

- 4번
  > 숫자를 내용으로 가진 문자는 숫자로 치환

| '1'+3 |
| ----- |
| 4     |

### 응용

📌 Example

| Product | Price | Count |
| ------- | ----- | ----- |
| A       | 10    | 2     |
| B       | 20    | 3     |

```sql
SELECT Product, Price * Count AS Pay
FROM Example;
```

| Product | Pay |
| ------- | --- |
| A       | 20  |
| B       | 30  |

## 참 & 거짓

```sql
SELECT TRUE, FALSE;
```

| TRUE | FALSE |
| ---- | ----- |
| 1    | 0     |

### 응용

// Example
| Product | Price | Count |
| ------- | ----- | ----- |
| A | 10 | 2 |
| B | 20 | 3 |
| C | 20 | 5 |
| D | 20 | 7 |

```sql
SELECT Product, Count < 3 AS TEST
FROM Example
WHERE Count < 6;
```

| Product | TEST |
| ------- | ---- |
| A       | 1    |
| B       | 0    |
| C       | 0    |

> 이러한 연산자들을 SELECT 키워드 다음 혹은 WHERE 키워드 다음에 활용하여 다양한 데이터를 출력할 수 있다.

# 숫자 함수

| 함수                   | 의미                       |
| ---------------------- | -------------------------- |
| ROUND                  | 반올림                     |
| CEIL                   | 올림                       |
| FLOOR                  | 내림                       |
| ABS                    | 절대값                     |
| GREATEST               | (괄호 안에서) 가장 큰 값   |
| LEAST                  | (괄호 안에서) 가장 작은 값 |
| MAX                    | 가장 큰 값                 |
| MIN                    | 가장 작은 값               |
| COUNT                  | 갯수(NULL 미포함)          |
| SUM                    | 총합                       |
| AVG                    | 평균                       |
| POW(A, B), POWER(A, B) | A를 B만큼 제곱             |
| SQRT                   | 제곱근                     |
| TRUNCATE(N, n)         | N을 소수점 n자리 까지 선택 |

# 문자열 함수

| 함수                          | 의미                                           |
| ----------------------------- | ---------------------------------------------- |
| UCASE, UPPER                  | 모두 대문자로                                  |
| LCASE, LOWER                  | 모두 소문자로                                  |
| CONCAT(...)                   | 괄호 안의 내용 이어 붙임                       |
| CONCAT_WS(S, ...)             | 괄호 안의 내용 S로 이어 붙이기                 |
| SUBSTR, SUBSTRING             | 주어진 값에 따라 문자열 자르기                 |
| LEFT                          | 왼쪽 부터 N글자                                |
| RIGHT                         | 오른쪽 부터 N글자                              |
| LENGTH                        | 문자열의 바이트 길이                           |
| CHAR_LENGTH, CHARACTER_LENGTH | 문자열의 문자 길이                             |
| TRIM                          | 양쪽 공백 제거                                 |
| LTRIM                         | 왼쪽 공백 제거                                 |
| RTRIM                         | 오른쪽 공백 제거                               |
| LPAD(S, N, P)                 | S가 N글자가 될 때까지 P를 왼쪽에 이어 붙이기   |
| RPAD(S, N, P)                 | S가 N글자가 될 때까지 P를 오른쪽에 이어 붙이기 |
| REPLACE(S,A,B)                | S중 A를 B로 변경                               |
| INSTR(S, s)                   | S중 s의 첫 위치를 반환, 없다면 0               |
| CAST(A AS T), CONVERT(A,T)    | A를 T자료형으로 변환                           |

# 시간/날짜 관련 함수

| 함수                       | 의미                                  | 예제                                                   |
| -------------------------- | ------------------------------------- | ------------------------------------------------------ |
| CURRENT_DATE(), CURDATE()  | 현재 날짜 변환                        |                                                        |
| CURRENT_TIME(), CURTIME()  | 현재 시간 반환                        |                                                        |
| CURRENT_TIMESTAMP(), NOW() | 현재 시간과 날짜 반환                 |                                                        |
| DATE()                     | 문자열에 따라 날짜 생성               | DATE('2024-02-12')                                     |
| TIME()                     | 문자열에 따라 시간 생성               | TIME('21:21:21')                                       |
| YEAR()                     | DATETIME값의 년도 반환                |                                                        |
| MONTHNAME()                | DATETIME값의 월(영문) 반환            | MONTHNAME('2024-07-03') -> July                        |
| MONTH()                    | DATETIME값의 월 반환                  |                                                        |
| WEEKDAY()                  | DATETIME값의 요일값 (월요일: 0)       | WEEKDAY('2024-02-16') -> 6                             |
| DAYNAME()                  | DATETIME값의 요일명 반환              | DAYNAME('2024-02-16') -> Friday                        |
| DAYOFMONTH(), DAY()        | DATETIME값의 날짜(일) 반환            |                                                        |
| HOUR()                     | DATETIME의 시 반환                    |                                                        |
| MINUTE()                   | DATETIME의 분 반환                    |                                                        |
| SECOND()                   | DATETIME의 초 반환                    |                                                        |
| ADDDATE(), DATE_ADD()      | 시간/날짜 더하기                      | ADDDATE('2024-02-12', INTERVAL 1 YEAR) -> '2025-02-12' |
| SUBDATE(), DATE_SUB()      | 시간/날짜 빼기                        |                                                        |
| DATEDIFF()                 | 두 시간/날짜 간 일수차                | DATEDIFF('2024-02-12', '2024-01-12') -> 31             |
| TIMEDIFF()                 | 두 시간/날짜 간 시간차                |                                                        |
| LAST_DAY                   | 해당 달의 마지막 날짜                 |                                                        |
| STR_TO_DATE(S,F)           | S를 F형식으로 해석하여 시간/날짜 생성 | STR_TO_DATE('2024-02-12 21:21:21', '%Y-%m-%d %T')      |

- DATE_FORMAT : 시간/날짜를 지정한 형식으로 반환한다.

| 형식   | 의미           |
| ------ | -------------- |
| %Y     | 4자리 연도     |
| %y     | 2자리 연도     |
| %M     | 월(영문)       |
| %m     | 월(숫자)       |
| %D     | 일(영문)       |
| %d, %e | 일(숫자)       |
| %T     | hh:mm:ss       |
| %r     | hh:mm:ss AM/PM |
| %H, %k | 24시           |
| %h, %l | 12시           |
| %i     | 분             |
| %S, $s | 초             |
| %p     | AM/PM          |

`DATE_FORMAT(NOW(), '%Y년 %m월 %d일 %p %h시 %i분 %s초'); -> 2024년 02월 12일 PM 01시 01분 02초`

# 기타 함수

- `IF(조건, T, F)` : 조건이 참이라면 T, 거짓이면 F반환

📍 복잡한 조건은 `CASE`문 사용

```sql
SELECT
CASE
  WHEN a > 10 THEM '10보다 크다.'
  WHEN a == 10 THEM '10이다.'
  ELSE '10보다 작다.'
END;

SELECT
  IF(1 > 0, '1은 0보다 크다.', '1은 0보다 작다');
```

- `IFNULL(A,B)` : A가 NULL일 경우 B 출력

```sql
SELECT(NULL, 'A'); -> A
```
