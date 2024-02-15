---
title: "[Spring] Bean의 즉시 로딩과 지연 로딩"
writer: chanho
date: 2024-02-15 12:50:00 +0900
categories: [Backend, Spring]
tags: [backend, spring]
---

스프링(Spring) 프레임워크에서 '즉시 로딩(Eager Loading)'과 '지연 로딩(Lazy Loading)'으로 빈(Bean)을 초기화 하는 방식을 설명하고자 한다.

# 지연 로딩(Lazy Loading)

> 지연 로딩은 빈이 실제로 필요할 때까지 생성을 지연시키는 방법으로 `빈에 대한 요청이 들어올 때 해당 빈을 생성하고 초기화 한다.` 이 방식은 시작 시간을 줄이고 메모리를 최적화 할 수 있지만, 빈이 처음 사용될 때 초기화 과정으로 인해 지연이 발생할 수 있으며, `의존성 문제가 런타임에 발생`할 수 있다.

- 시작 시간 단축
  > 대규모 애플리케이션에서 애플리케이션의 시작 시간을 단축하기 위해 일부 구성요소를 지연 로딩하여 속도를 높일 수 있다.
- 메모리 사용 최적화
  > 자주 사용되지 않는 서비스나 구성요소는 지연 로딩을 사용해 필요할 때만 메모리에 로드되도록 하여 전체적인 메모리 사용량을 줄일 수 있다.
- 조건부 기능
  > 특정 조건에서만 필요한 기능(ex. 특정 사용자 권한에 따른 서비스)은 지연 로딩을 사용하여 해당 조건이 충족될 때만 로드하게 할 수 있다.

# 즉시 로딩(Eager Loading)

> 즉시 로딩은 스프링프레임워크의 기본적인 빈 초기화 방식으로, 스프링 컨테이너가 시작될 때 모든 싱글톤 빈(Singleton Bean)을 미리 생성하는 방식이다. 이 방식의 장점은 `애플리케이션이 실행될 때 모든 의존성이 미리 확인되어 애플리케이션 실행 중에 발생할 수 있는 문제를 줄일 수 있다.` 단점으로는 애플리케이션의 시작 시간을 증가시키고, 시작 시에 필요로 하지 않는 빈들까지 초기화하여 불필요한 메모리를 차지할 수 있다는 점이 있다.

## 즉시 로딩 사용

- 핵심 서비스 구성요소
  > 데이터베이스 연결, 메시지 큐 설정, 핵심 서비스 로직 등 애플리케이션의 기본적인 기능을 담당하는 중요한 구성요소들은 즉시 로딩을 통해 애플리케이션 시작 시 바로 사용할 수 있게 한다.
- 의존성 검사
  > 애플리케이션의 여러 구성요소 간의 의존성이 복잡할 때, 시작 시 모든 구성요소를 로드하여 의존성 문제를 미리 발견하고 해결한다.
- 성능 최적화
  > 사용자 요청 처리 시 지연을 최소화하기 위해 필요한 서비스나 컴포넌트를 사전에 로드한다.

## 💡 항상 즉시 로딩을 추천하는 이유?

`스프링 구성에 오류가 있을 경우 즉시 초기화를 사용하면 애플리케이션이 시작할 때 오류를 바로 확인할 수 있다.`

# 사용 예제

```java
@Component
class ClassA {

}

@Component
@Lazy
class ClassB {
	private ClassA classA;

	public ClassB(ClassA classA) {
		// Logic
		System.out.println("Some Initialization logic");
		this.classA = classA;
	}

	public void doSomething() {
		System.out.println("Do Something");
	}
}

@Configuration
@ComponentScan
public class LazyInitializationLauncherApplication {
	public static void main(String[] args) {
		try (var context = new AnnotationConfigApplicationContext
				(LazyInitializationLauncherApplication.class)) {

			System.out.println("Initialization of context is completed");

			context.getBean(ClassB.class).doSomething();

		} catch (BeansException e) {
			e.printStackTrace();
		}
	}
}

```

- @Lazy 미사용 : `default: 즉시 로딩`

```
// 출력
Some Initialization logic
Initialization of context is completed
Do Something
```

- @Lazy

```
// 출력
Initialization of context is completed
Some Initialization logic
Do Something
```
