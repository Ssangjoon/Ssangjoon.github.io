---
title: 왜 update문이 날아가는가? - dirtychecking
date: 2023-05-17 17:00:00 +0900
categories: [개인프로젝트]
tags: [GettingThingsDone,JPA,Transaction,Curiosity]     # TAG names should always be lowercase
published: false
---

# Curiosity

```java
public void update(String userName, String name, String email) {
        this.userName = userName;
        this.name = name;
        this.email = email;
    }
```

프로젝트 초반에 update를 어떻게 해야할까 고민하다 Entity 내에서 위와 같이 코드를 작성하였다.   

나는 저게 무슨 jpa 전용 메서드인 줄만 알고 그냥 넘어갔었다. (ㅇㅅㅇ..ㅎ…)

다행히 나중에라도 어째서 쿼리문이 날아가는지 궁금해졌다. 

# Cause

## Drity Checking

상태변경 검사라고 보면 된다. 

더티체킹이 일어나는 환경은 아래 두가지 조건이 충족되어야 하는데

- 영속 상태 안에 있는 엔티티
- Transaction안에서 엔티티를 변경하는 경우

```java
@Service
public class ExampleService {
     @Transactional
     public void updateInfo(Long id, String name) {
          User user = userRepository.findById(id);
          user.setName(name);
     }
}
```

```java
@Service
public class ExampleService {
     public void updateInfo(Long id, String name) {
          EntityManager em = entityManagerFactory.createEntityManager();
          EntityTransaction tx = em.getTransaction();
          // 1. 트랜잭션 시작
          tx.begin();
          // 2.User 엔티티를 조회 & User 스냅샷 생성
          User user = em.find(User.class, id);
          // 3.User 엔티티의 name을 변경
          user.setName(name);
          // 4. 트랜잭션
          // 5.User 스냅샷과 최종 user의 내용을 비교해서 Dirty를 Checking 해서 Update Query 발생
          tx.commit();
     }
}
```

**Transaction을 사용하지 않아서 반영되지 못한 내용을 반영하고 싶은 경우,**

**원하는 시점에 save, saveAndFlush를 사용.**

```java
public void updateInfo() { 
    User user = userRepository.findById(2L)
        .orElseThrow(() -> new ErrorCodeException(ErrorType.USER_IS_NOT_EXISTING));
    user.setEmail("hello@gmail.com");
    userRepository.saveAndFlush(user);
    System.out.println(userRepository.existsByEmail("hello@gmail.com")); // true
} 
```

## 그렇다면 JPA는 어떻게 엔티티들의 변경을 감지할 수 있을까?

- 엔티티를 조회하면 해당 엔티티의 조회 상태 그대로 스냅샷을 만들어 놓는다.
- 그리고 트랜잭션이 끝나는 시점에는 이 스냅샷과 비교해서 비교하여 변경을 감지한다.
- 변경이 감지되었다면 update query 를 데이터 베이스로 전달한다.

말이 조금 어렵다. 좀 찾아보다가 

https://frogand.tistory.com/175

위 글이 도움이 되었는데, 

엔티티 내부에 객체의 값을 변경하는 메서드를 호출하면 update문이 나가는 상황.

save()메서드가 없이 객체의 값을 변경했을 뿐인데 update문이 나간다.

# Result

결론은 동일하다. 

트랜잭션이 끝나는 시점, 변화가 있는 모든 엔티티 객체를 데이터 베이스에 자동으로 반영해준다.

조회한 시점에서 스냅샷을 만들고 해당 스냅샷을 기준으로 비교하여 변경을 감지하기에 가능한 것이었다. 

(트랜잭션에 대해 학습해봐야겠다.)