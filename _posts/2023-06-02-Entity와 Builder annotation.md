---
title: Entity와 Builder annotation
date: 2023-06-02 10:00:00 +0900
categories: [개인프로젝트]
tags: [GettingThingsDone,JPA]     # TAG names should always be lowercase
published: false
---

> 엔티티 빌더인데 생성자에서 굳이 builder어노테이션을 쓰는 이유가 있을까요?
> 

뭐가 문제였을까 싶었는데 

Builder 패턴을 사용하면 다음과 같은 장점이 있다.

1. 인자가 많을 경우 쉽고 안전하게 객체를 생성할 수 있다.
2. 인자의 순서와 상관없이 객체를 생성할 수 있다.
3. **적절한 책임을 이름에 부여하여 가독성을 높일 수 있다.**

이름의 문제일까? 

아니면 클래스에 builder를 붙이지 않았냐는 말일까? 

builder를 쓰는 이유는 

[https://velog.io/@mooh2jj/올바른-엔티티-Builder-사용법](https://velog.io/@mooh2jj/%EC%98%AC%EB%B0%94%EB%A5%B8-%EC%97%94%ED%8B%B0%ED%8B%B0-Builder-%EC%82%AC%EC%9A%A9%EB%B2%95)

https://cheese10yun.github.io/spring-builder-pattern/

Entity 는 동적으로 객체 생성 시 java의 Reflection API를 활용하기 때문에 기본 생성자가 필요하며,

무분별한 생성을 막기위해 혹은, 

Entity 를 조회하기 위해 생성되는 proxy 객체는 직접 만든 객체 class를 상속하기 때문에 public이나 protected 기본 생성자를 선언해야한다.

private으로 허용하면 dto 객체가 lazy fetch 같은 비동기 조회 방식 일때 일관성을 가지지 못하는 문제가 발생할 수 있다고 한다.


builder를 쓰는 방식은 클래스에 붙이거나 생성자에 붙이거나인데 

클래스에 `Builder` 와`NoArgsConstructor`를 함께 쓰면 컴파일 에러가 난다. 

- 왜?
    
    `@builder`는 클래스 자체 붙이거나 생성자에 붙이는 방법이 있는데,`@builder`는 전체 생성자를 필요로함`@builder`는 `@NoArgsConstrator`어노테이션이나 다른 생성자가 존재한지 않다면 전체 생성자를 자동으로 만든다.
    
    => `@builder`와 `@NoArgsConstrator`를 클래스에 함께 쓰면 전체 생성자가 만들어지지 않고 컴파일에러가 발생
    

이를 위해 

`@AllArgsConstructor`를 달아주는 방법이 있지만 

이는 위험한 일이라고 한다. 

이유는 클래스에 존재하는 모든 필드에 대한 생성자를 자동으로 생성하면 인스턴스 멤버의 순서에 영향을 받기 때문에 변수의 순서를 바꾸면 입력값 순서도 바뀌게 되어 검출되지 않는 오류를 발생시킨다고 하다. 

뭔말일까 이게?

https://www.nowwatersblog.com/springboot/springstudy/lombok

여기에 더 잘 써져 있었는데 

# 결론

`@builder`는 클래스 자체 붙이거나 생성자에 붙이는 방법이 있는데,

클래스에 붙일 경우 `@builder`는 전체 생성자를 필요로함,

`@builder`는 @NoArgsConstrator어노테이션이나 다른 생성자가 존재한지 않다면 전체 생성자를 자동으로 만든다.=>

`@builder`와 @NoArgsConstrator를 클래스에 함께 쓰면 전체 생성자가 만들어지지 않고 컴파일에러가 발생 이에 따라

`@AllArgsConstructor`를 달아주면 전체 생성자를 만들어준다면 당장 에러는 해결 가능하지만 필드의 순서를 바꾸거나 할때 바로 검출되지 않는 에러가 발생할 수 있기 때문에,

`@AllArgsConstructor`를 붙이는 것은 지양해야 한다고 함그래서 생성자에 `@builder`를 쓰는게 나음

자바의 객체 생성방법

https://ssdragon.tistory.com/78