---
title: "[Spring] URI를 Location 헤더에 담아서 반환하기 _ ResponseEntity, Locationgi"
writer: chanho
date: 2024-02-25 16:50:00 +0900
categories: [Backend, Spring]
tags: [backend, spring]
---

RESTfull 웹 서비스에서 새로운 리소스가 생성되었을 때 그 리소스의 위치를 나타내는 URI를 'Location'헤더에 담아서 반환할 수 있다. 이는, 클라이언트가 해당 위치를 사용하여 새로 생성된 리소스에 접근 할 수 있다.

```java
URI location = ServletUriComponentsBuilder.fromCurrentRequest().path("/{id}").buildAndExpand(savedUser.getId()).toUri();
return ResponseEntity.created(location).build();
```

# 반환 하는 것

> 해당 코드는 새로 생성된 리소스의 URI를 'Location'헤더에 담은 HTTP 응답을 반환한다. 즉, 'ResponseEntity' 객체가 반환되며, 이 객체는 HTTP 상태 코드 201(Created)와 함께 생성된 리소스의 URI를 포함한다.

## 코드 설명

- `ServletUriComponentBuilder.fromCurrentRequest()`: 현재 HTTP 요청의 정보를 바탕으로 'UriComponentsBuilder'인스턴스를 생성한다. 이는 현재 요청의 URI를 기반으로 새로운 URI를 만드는 데 사용된다.

- `.path("/{id}")`: 현재 URL에 경로 변수 '{id}'를 추가한다. 예를 들어, 현재 요청이 'http://example.com/users'였다면, 이 메소드는 'http://example.com/users/{id}'형태의 URI를 생성하는 데 사용된다.

- `.buildAndExpand(savedUser.getId())`: '{id}' 부분을 'savedUser.getId()'의 값으로 치환하여 실제 URI를 만든다. 여기서 'savedUser'는 새로 저장된 객체를 가리키며, 'getId()'메소드는 해당 사용자의 고유 식별자를 반환한다.

- `toUri()`: 최종적으로 생성된 URI 객체로 변환한다.

- `ResponseEntity.created(location).build()`: HTTP 상태 코드 201(Created)와 함께 응답을 생성한다. 이때 'location'변수에는 위에서 생성된 URI가 들어있다. 'build()'메소드는 'ResponseEntity'객체를 최종적으로 만든다.

## ResponseEntity

> Spring Framework에서 HTTP 요청에 대한 응답을 나타내는 클래스이다. 이 객체를 사용하여 HTTP 상태 코드, 헤더, 응답 본문 등을 포함한 완전한 HTTP 응답을 생성할 수 있다.

### 생성 방법

- `ResponseEntity.ok()`: HTTP 상태 코드 200 Ok를 나타내는 응답을 생성
- `ResponseEntity.status(HttpStatus)`: 다양한 HTTP 상태 코드를 설정할 수 있다.
- `ResponseEntity.created(URI)`: 201 Created 상태와 'Location'헤더를 함께 반환 하는 데 사용

# 사용하는 이유

1. `RESTful API 설계 원칙 준수`: REST 아키텍처에서는 새로운 리소스를 생성할 때 클라이언트에게 해당 리소스의 URI를 제공하는 것이 일반적이다. 이는 클라이언트가 나중에 해당 리소스에 접근하거나 참조할 수 있게 해준다.
2. `효율적인 Client-Server 커뮤니케이션`: 클라이언트가 리소스 생성 요청을 보낸 후, 서버가 생성된 리소스 위치를 알려주면 클라이언트는 추가 요청을 통해 리소스를 조회할 수 있다. 이는 네트워크 트래픽과 서버의 처리 부하를 줄이는 데 도움이 된다.

# 활용

- `Client`: 클라이언트는 HTTP 응답의 'Location'헤더에서 URI를 추출하여 새로 생성된 리소스에 접근할 수 있다. 예를 들어, 사용자 정보를 등록한 후 해당 사용자의 상세 정보 페이지로 리다이렉션할 때 이 URI를 사용할 수 있다.

- `Server`: 서버는 이 방법을 사용하여 새로 생성된 리소스에 대한 참조를 제공한다. 이는 특히 데이터베이스에 새로운 엔트리를 추가한 후, 해당 엔트리의 고유 식별자를 클라이언트에게 알려줄 때 유용하다.

# ❓ URI & URL

## URI(Uniform Resource Identifier, 통합 자원 식별자)

- URI는 인터넷 상의 자원을 식별하는 데 사용되는 문자열이다. 이는 자원의 이름 또는 주소를 나타내며, 웹 주소뿐만 아니라 다양한 형태의 자원을 식별할 수 있다.
- URI는 두 가지 주요 형태를 가진다. (URL, URN)

## URL(Uniform Resouce Locator, 통합 자원 지시자)

- URL은 인터넷 상의 자원이 어디에 위치해 있는지를 나타내는 URI의 한 형태이다. 쉽게 말해, 특정 웹페이지나 파일등을 찾아가는 데 사용되는 인터넷 주소이다.
- URL은 자원의 위치와 접근 방법을 제공한다. 예를 들자면 'http://www.example.com/index.html'은 'http'프로토콜을 사용하여 'www.example.com'호스트의 'index.html'파일에 접근하는 방법을 나타낸다.
