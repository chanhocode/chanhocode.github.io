---
title: "[MySQL] 맥에서 MySQL 설치 및 기본 셋팅 하기"
writer: chanho
date: 2024-02-20 17:50:00 +0900
categories: [Backend, Database]
tags: [backend, mysql]
---

![mysql](https://github.com/chanhocode/chanhocode.github.io/assets/105937460/14edf8fc-2ea4-4580-a98f-ce043c39a459)

# Homebrew를 통한 MySQL 설치

- Homebrew가 없다면 [설치](https://brew.sh/)

- brew를 통해 설치

```
$ brew update # 설치전 업데이트
$ brew install mysql # 설치
```

- 설치 확인

```
$ mysql --version
```

- MySQL 시작 및 종료

```
# 서버 시작
$ mysql.server start

------
# 성공적으로 시작시 아래와 같은 문구 출력
Statring MySQL
  SUCCESS!
------

# 서버 종료
$ mysql.server stop
```

- Homebrew를 통해 데몬으로 실행하기
  > 백그라운드에서 'MySql'을 실행시키고 싶다면, 'Homebrew'를 통해 실행 하면 된다.

```
# 서버 실행
$ brew services start mysql

# 재실행
$ brew services restart mysql

# 종료
brew service stop

# + 데몬 실행중인 프로그램 확인하기
$ brew service list
```

# 초기 설정 하기

```
$ mysql_secure_installation

------
1. VALIDATE PASSWORD COMPONENT
  (복잡한 비밀번호 여부) : (y/n) // 간단한 개발 용도로 사용할 것 이므로 "n"
2. set the password : // 1234
3. Remove anonymous users?
  (익명 사용자를 삭제 할 것인가?) : (y/n) // y
4. Disallow root login remotely?
  (원격 접속을 허용하지 않을 것인가?) : (y"허용 하지 않음"/n) // y
5. Remove test database and access to it?
  (test 데이터베이스를 삭제 할 것인가?) : (y/n) // y
5. Reload privilege tables now?
  (모든 변경 사항을 적용하고 MySQL 서버를 다시 시작 유무) : (y/n) // y

# 설정이 성공적으로 완료시 "All done!" 문구 출력
```

## 익명 사용자 제거

> MySQL은 기본적으로 '익명' 사용자를 포함하고 있습니다. 이 사용자는 비밀번호 없이 MySQL에 접근할 수 있기 때문에, 보안상의 이유로 제거하는 것이 좋습니다.

## 루트 사용자 원격 접속 비활성화

> 보안을 위해, 루트 사용자가 원격에서 MySQL에 접근하는 것을 일반적으로 제한합니다.

## 테스트 데이터베이스 제거

> MySQL에는 테스트 목적으로 사용되는 'test' 데이터베이스가 포함되어 있습니다. 이 데이터베이스는 모든 사용자가 접근할 수 있으므로, 보안을 위해 제거하는 것이 좋습니다.

# MySQL 접속하기

```
# 접속
# mysql -h[자신의 ip주소] -u[계정명] -p[비밀번호] [데이터베이스 명]

$ mysql -u root -p

# 특정 사용자로 접속시
$ mysql -u [사용자] -p

# 특정 DB 접속시
$ mysql -u root -p [데이터베이스 명]
```

# 계정 생성 및 삭제

```
# 계정 생성
$ CREATE USER 'username'@'localhost' IDENTIFIED BY 'password'

# 바로 적용
$ FLUSH PRIVILEGES;

# 계정목록 확인
$ SELECT User, Host FROM mysql.user;

# 계정 삭제하기
$ DROP USER 'username'@'%'
```

## 권한 부여하기

```
# 모든 권한 부여하기
$ grant all privileges on [DB명, 모든 DB(*)].* to 'username'@'localhost';
```

- 'all privileges': 데이터베이스의 모든 권한
