---
title: "[Spring] 커스텀 예외처리 간단하게 생성하기 _ Custom Exception"
writer: chanho
date: 2024-02-25 18:00:00 +0900
categories: [Backend, Spring]
tags: [backend, spring]
---

Spring Boot에서 유효하지 않은 요청이 전송되거나 서버 내부에 오류가 발생했을 경우, 개발자가 별도의 오류페이지를 구성하지 않았다면 기본적으로 제공하는 간단한 오류페이지인 `Whitelabel Error Page`가 발생한다.

![whitelabel](https://github.com/chanhocode/chanhocode.github.io/assets/105937460/ca62bae7-fbb8-4b8e-9e30-fafb2fc8e972)

# ErrorCode 작성하기

> 커스텀 예외처리에서 표시할 내용 및 부가적인 부분에 관한 정의를 한다.

```java
public class ErrorDetails {
	private LocalDateTime timestamp;
	private String message;
	private String detail;

	public ErrorDetails(LocalDateTime timestamp, String message, String detail) {
		super();
		this.timestamp = timestamp;
		this.message = message;
		this.detail = detail;
	}

	public LocalDateTime getTimestamp() {
		return timestamp;
	}
	public String getMessage() {
		return message;
	}
	public String getDetail() {
		return detail;
	}

}
```

# @ControllerAdvice를 사용한 전역 예외 처리기 사용하기

> 전역 예외 처리기를 사용하여 애플리케이션 전체에서 발생하는 예외를 처리하고 사용자 정의 응답을 반환할 수 있다.

```java
@ControllerAdvice
public class CustomizedResponseEntityExceptionHandler  extends ResponseEntityExceptionHandler{

	@ExceptionHandler(Exception.class)
	public final ResponseEntity<ErrorDetails> handleAllExceptions(Exception ex, WebRequest request) throws Exception {
		ErrorDetails errorDetails = new ErrorDetails(LocalDateTime.now(),
				ex.getMessage(),
				request.getDescription(false));

		return new ResponseEntity<ErrorDetails>(errorDetails, HttpStatus.INTERNAL_SERVER_ERROR);

	}

	@ExceptionHandler(UserNotFoundException.class)
	public final ResponseEntity<ErrorDetails> handleUserNotFoundExceptions(Exception ex, WebRequest request) throws Exception {
		ErrorDetails errorDetails = new ErrorDetails(LocalDateTime.now(),
				ex.getMessage(),
				request.getDescription(false));

		return new ResponseEntity<ErrorDetails>(errorDetails, HttpStatus.NOT_FOUND);

	}
}
```

> 해당 코드는 Spring Boot에서 전역 예외 처리를 구현한 클래스로 `ResponseEntityExceptionHandler`를 확장하며 여러 유형의 예외 처리 로직을 정의할 수 있다.

- `@ControllerAdvice`: 어플리케이션의 모든 컨트롤러에서 발생하는 예외를 처리할 수 있다.
- `@ExceptionHandler`: 특정 예외를 처리하기 위한 메소드를 정의하는 데 사용되는 어노테이션 이다.

![image](https://github.com/chanhocode/chanhocode.github.io/assets/105937460/ebd09039-903f-41cf-9a28-311cab218f74)
