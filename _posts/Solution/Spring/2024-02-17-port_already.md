---
title: "[Solution] Spring - web server failed to start. port 8080 was already in use 해결"
writer: chanho
date: 2024-02-17 00:08:00 +0900
categories: [Solution, Spring]
tags: [solution, spring]
---

Spring에서 웹 서버를 실행시 시작 실패와 해당 포트가 이미 사용중이라는 오류를 마주치며 해결한 방법을 기록하고자 한다.

![port_already0](https://github.com/chanhocode/chanhocode.github.io/assets/105937460/34f9a979-8cc8-4094-a3ef-fceebe5d2115)

> 해당 오류는 일반적으로 8080 포트가 이미 다른 프로세스에 의해 사용되고 있을 때 발생한다.

## 사용 중인 포트 확인 및 프로세스 종료

- 포트 확인

```
# Window
$ netstat -ano | find "8080"

# Linux & Mac
$ sudo lsof -i :8080
or
$ netstat -tulnp | grep 8080
```

![port_already1](https://github.com/chanhocode/chanhocode.github.io/assets/105937460/6b06d9d2-33bf-4b55-a159-2658e5fe8fa5)

- 확인된 프로세스 종료

```
# Window
$ taskkill /F /PID 7120

# Linux & Mac
$ kill -9 7120
```

## 포트 번호 변경

> `application.properties`파일에서 `server.port`속성을 수정하여 다른 포트번호를 지정하여 충돌을 회피할 수 있다.

```
# application.properties
server.port=9090
```
