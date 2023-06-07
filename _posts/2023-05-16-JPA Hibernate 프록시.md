---
title: JPA Hibernate 프록시
date: 2023-05-16 18:00:00 +0900
categories: [개인프로젝트]
tags: [개인프로젝트,GettingThingsDone,JPA,Transaction]      # TAG names should always be lowercase
published: true
---


앞선 글에서의 에러 해결 후 궁금해서 관련 개념에 대해 좀 더 찾아 보았다. 
---

https://tecoble.techcourse.co.kr/post/2022-10-17-jpa-hibernate-proxy/

[https://velog.io/@ohzzi/JPA-프록시의-사실과-오해](https://velog.io/@ohzzi/JPA-%ED%94%84%EB%A1%9D%EC%8B%9C%EC%9D%98-%EC%82%AC%EC%8B%A4%EA%B3%BC-%EC%98%A4%ED%95%B4)

[https://velog.io/@ynoolee/JPAProxy-와-지연로딩](https://velog.io/@ynoolee/JPAProxy-%EC%99%80-%EC%A7%80%EC%97%B0%EB%A1%9C%EB%94%A9)

---
위 글들을 참고하였고 내가 읽기 편하게 조금 바꾸었음을 알림


# **JPA 프록시(Proxy) 란?**

JPA 프록시에 대해 알아보기에 앞서, **프록시란** 무엇일까? 
프록시는 ‘대신하다‘라는 의미를 가지고 있는 단어 즉,  동작을 대신해주는 가짜 객체의 개념이라고 생각하면 된다. 

대표적으로 스프링에서도 초기화를 지연시켜준다든가, 트랜잭션을 적용한다든가 하는 부가 기능을 추가할 때 프록시 기술을 사용하는 것을 생각하면 이해하기 쉽다.

**하이버네이트**는 **지연 로딩을 구현하기 위해 프록시를 사용**한다. 
지연 로딩을 하려면 연관된 엔티티의 실제 데이터가 필요할 때 까지 조회를 미뤄야 하는데, 그렇다고 엔티티의 필드에 null 값을 넣어 둘 수는 없기에 하이버네이트는 지연 로딩을 사용하는 연관관계 자리에 프록시 객체를 주입하여 실제 객체가 들어있는 것처럼 동작하도록 한다.

덕분에 우리는 연관관계 자리에 프록시 객체가 들어있든 실제 객체가 들어있든 신경쓰지 않고 사용할 수 있다. 
참고로 프록시 객체는 지연 로딩을 사용하는 것 외에도 `em.getReference`를 호출하여 프록시를 호출할 수도 있음.

# **프록시는 실제 객체의 상속본이다**

프록시가 어떻게 실제 객체처럼 동작을 할 수 있을까? 이는 프록시가 실제 객체를 상속한 타입을 가지고 있기 때문이다. 그리고 프록시 객체는 실제 객체에 대한 참조를 보관하여, 프록시 객체의 메서드를 호출했을 때 실제 객체의 메서드를 호출한다. 

프록시 객체가 실제 객체의 상속본인 것은 JPA 엔티티 생성의 중요한 규칙을 만들어내기도 했다. 바로 ‘기본 생성자는 최소 protected 접근 제한자를 가져야 한다.’ 는 규칙과 ‘엔티티 클래스는 final로 정의할 수 없다. 이.’ 만약 기본 생성자가 private이면 프록시 생성 시 `super`를 호출할 수 없을 것이고, 엔티티를 final로 선언한다면 상속이 불가능하게 된다?

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0e094683-92ba-457e-af6f-5adf5d2c8cf8/Untitled.png)

- 실제 클래스를 상속 받아서 만들며
- 실제 객체의 참조를 보관한

# **프록시의 초기화**

그런데 최초 지연 로딩 시점에는 당연히 참조 값이 없다. 때문에 실제 객체의 메서드를 호출할 필요가 있을 때 데이터베이스를 조회해서 참조 값을 채우게 되는데요, 이를 프록시 객체를 초기화한다고 한다. 앞서 말했듯이 실제 객체의 메서드를 호출할 필요가 있을 때 select 쿼리를 실행하여 실제 객체를 데이터베이스에서 조회해오고, 참조 값을 저장하게 된다. 

```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column
    private String name;

    @JoinColumn(name = "team_id")
    @ManyToOne(fetch = FetchType.LAZY)
    private Team team;

    public Member(String name, Team team) {
        this.name = name;
        this.team = team;
    }
    ...
}

@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class Team {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column
    private String name;

    public Team(String name) {
        this.name = name;
    }
    ...
}
```

```java
System.out.println("========");
Team team = member.getTeam();
System.out.println(team.getName()); // 이 시점에 프록시 초기화!
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dc595870-5dcf-4ec3-9d06-5415916d58d2/Untitled.png)

Member를 조회해오게 되면 team 필드 자리에는 프록시가 들어가 있다. 이 때 프록시 Team의 getName 메서드를 호출하게 되면 select 쿼리가 실행되고 프록시가 초기화다.

단, 이 때 프록시가 실제 객체를 참조하게 되는 것이지 프록시가 실제 객체로 바뀌지는 않는다는 점을 주의하자. 참고로 프록시를 초기화하지 못하는 경우도 있다. 프록시의 초기화는 영속성 컨텍스트의 도움을 받다. 때문에 영속성 컨텍스트의 관리를 받지 못하는 상황, 즉 준영속 상태의 프록시를 초기화 한다거나 OSIV 옵션이 꺼져 있는 상황에(기본값으로는 켜져 있습니다.) 트랜잭션 바깥에서 프록시를 초기화 하려 하는 경우 `LazyInitializationException`을 만나게 되실 수 있다. 때문에 프록시를 초기화할 때는 반드시 프록시가 `영속 상태`여야 한다는 점에 주의!

# 하이버네이트에서 프록시.getId()는 프록시를 초기화 하지 않는다

프록시 객체는 엔티티 객체의 데이터를 사용할 때 초기화됩니다. 예를 들어 `team.getName`을 호출하면 해당 팀에 대한 select 쿼리가 날아가게 됩니다. 또한, 엔티티의 `AccessType`이 `Property`가 아닌 `Field`라면 식별자 호출을 할 때도 초기화됩니다.

그런데 하이버네이트에서 식별자를 꺼내는 `getId`를 호출할 때는 초기화되지 않습니다.

그런데 getName이 아닌 getId를 호출할 때도 초기화될까요? 테스트해보도록 하겠습니다.

```java
System.out.println("========");
Team team = member.getTeam();
System.out.println(team.getId());
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/105e0c0f-0ec7-415b-9d30-fca52ca8773d/Untitled.png)

이는 하이버네이트 설정 중 `hibernate.jpa.compliance.proxy` 때문입니다.

> hibernate.jpa.compliance.proxy (e.g. true or false (default value))
The JPA spec says that a javax.persistence.EntityNotFoundException should be thrown when accessing an entity Proxy which does not have an associated table row in the database.
Traditionally, Hibernate does not initialize an entity proxy when accessing its identifier since we already know the identifier value, hence we can save a database roundtrip.
If enabled Hibernate will initialize the entity proxy even when accessing its identifier.
> 

하이버네이트는 JPA 명세와는 다르게, 식별자를 호출할 때는 엔티티를 초기화하지 않습니다. 만약 식별자를 호출할 때 엔티티를 초기화하고 싶다면 `hibernate.jpa.compliance.proxy` 값을 true로 설정해주어야 합니다.

다만 여기서 주의하실 점은, `team.id`는 null이라는 점입니다. 프록시 객체는 기본적으로 모든 필드 값을 null로 가지고 있습니다. 하지만 프록시 객체는 추가로 interceptor(ByteBuddyInterceptor)를 가지고 있는데요, 이 인터셉터가 id값, 원래 엔티티의 타입 정보 등을 가지고 있습니다. 그리고 `target` 이라는 필드가 있는데요, 프록시를 초기화하고 나면 select 쿼리의 결과로 생성된 엔티티를 바로 이 `target`에 저장하게 됩니다. 때문에 select 쿼리를 통해 프록시가 초기화되지 않은 상태에서 프록시는 id값은 알 수 있지만 target이 없기 때문에 id를 제외한 다른 필드 값은 알 수가 없는 것입니다.

그리고 방금 select 쿼리를 날리고 나면 `target`에 쿼리의 결과 엔티티를 저장한다고 했죠? 그래서 한 번 프록시로 만들어지면 프록시가 초기화된다 해도 실제 엔티티로 바뀌지 않습니다. 단지 실제 엔티티의 `참조` 만 연결될 뿐이죠.

****프록시의 equals를 재정의 할 때는 instanceof와 getId를 사용할 것****

…

# **프록시가 생성되면 영속성 컨텍스트는 프록시를 반환한다**

프록시로 만들어진 엔티티에 대한 조회가 들어온다면, 영속성 컨텍스트는 실제 엔티티와 프록시 객체 중 어떤 객체를 반환해야 할까? 영속성 컨텍스트의 특징으로 `동일성 보장`이 있다. 
그런데 위에서 한 번 프록시로 만들어진 객체는 프록시 초기화를 하더라도 실제 엔티티로 변환되지 않고 참조값만 가지게 된다고 언급했다. 때문에 동일성을 보장해주기 위해서 한 트랜잭션 내에서 최초 생성이 프록시로 된 엔티티는 이후 초기화 여부에 상관 없이 영속성 컨텍스트가 무조건 같은 프록시 객체를 반환해주게 다.

```java
Team team = teamRepository.findById(1L).get();
assertThat(team).isInstanceOf(HibernateProxy.class);
```