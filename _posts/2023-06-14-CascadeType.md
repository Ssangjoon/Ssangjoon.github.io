---
title: Cascade에 대해
date: 2023-06-14 12:00:00 +0900
categories: [개인프로젝트]
tags: [study,jpa]     # TAG names should always be lowercase
published: false
---
팀 프로젝트 시 모임 삭제를 할 경우, 

cascade로? 해당 모임의 가입인원, 게시글, 댓글 등 관련 모든 정보를 삭제 시켰었다.   

그 당시 삭제를 어떻게 구현하였느냐는 질문에 나도 모르게 헛웃음을 지으며 cascade로 다 날렸다고 했는데 왜 cascade였고 지금이라면 어떻게 하고 싶냐는 질문에 벙찌게 되었다.

언젠가 귓동냥으로 cascade 남발하면 안된다는 말은 들었지만 이번기회에 다시 생각해보게 되었다.  

- 삭제가 무조건 나쁜 것은 아니며 정책에 따라 달라질 수 있으나
- 기본적으로 하루에도 수천건씩 쌓이는 데이터라면 삭제를 하고 소량이라면 비활성화
    - 많이 쌓이는 건 백업디비에 넘기고 리얼은 삭제
- 기본적으로 3개월 정도만 가지고 있는 테이블은 다 삭제 해주는게 좋다. (검색어?)

JPA 환경에서 관련 속성이 무엇이 있는지도 한번 알아보자

# 영속성 전이란?

특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때 사용하는 옵션

# What is Cascade?

영어 사전에서는 “계단식 폭포”라고 정의. 

즉 부모 엔티티부터 **연관된 자식 엔티티까지 상태를 전파**한다.

- **ALL** :
    - 아래의 모든 type을 포함
- **PERSIST** :
    - 부모 엔티티 저장 시 자식도 저장
- **REFRESH**
    - 다시 로드한다.
    - 영속된 상태에서 조회한 엔티티가 변경되더라도
    - refresh() 해서 db에 저장된 값을  override한다.
- **MERGE**
    - 트랜잭션이 종료되고 detach 상태에서 연관 엔티티를 추가하거나 변경된 이후에 부모 엔티티가 merge()를 수행하게 되면 변경사항이 적용된다.(연관 엔티티의 추가 및 수정 모두 반영됨)
        - 부모.merge() 연산 하게되면, 자식의 변경 사항도 수정된다.
        - Eager로딩일 경우에만 사용가능
- **REMOVE**
    - 부모 엔티티 삭제시 자식 엔티티도 모두 삭제한다.
    - JPA가 관리는 하나, commit 시점에만 삭제
- **DETACH**
    - 준영속 상태로 만들다.
    - 조회한 부모 엔티티가 detach상태가 되면 자식엔티티도..

**PERSIST** 와 **MERGE**가 비슷해 보이지만 

**PERSIST** 

- persist는 영속성 상태에 있지 않은 것을 자식 엔티티 까지 영속성 상태로 만드는게 핵심.

**MERGE**

- 준영속 상태 즉 영속성 컨텍스트에 올랐다가 내려온? 아이들은 여전히 식별자인 @Id가 남아있는데
- 이때 준영속 상태로 남아있는 식별자의 상태가 변경되면, persist()가 아닌 merge()발생
- merge는 비영속 상태의 부모 엔티티를 변경했을 때 비영속 상태의 merge를 수행한다.

## **CascadeType.REMOVE**

```java
@Entity
public class Team {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

		@OneToMany(
        mappedBy = "team",
        fetch = FetchType.LAZY,
        cascade = CascadeType.ALL   // { CascadeType.PERSIST, CascadeType.REMOVE }와 동일하다.
    )
    private List<Member> members = new ArrayList<>();

    public Team() {
    }

		public void addMember(Member member) {
        members.add(member);
        member.setTeam(this);
    }
}

// Member.java
@Entity
public class Member {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn
    private Team team;

    public Member() {
    }
}
```

```java
// JpaLearningTest.java
@DataJpaTest
public class JpaLearningTest {

    @Autowired
    private TeamRepository teamRepository;

    @Autowired
    private MemberRepository memberRepository;
}
```

### 부모 엔티티를 삭제

```java
@DisplayName("CascadeType.REMOVE - 부모 엔티티(Team)을 삭제하는 경우")
@Test
void cascadeType_Remove_InCaseOfTeamRemoval() {
    // given
    Member member1 = new Member();
    Member member2 = new Member();

    Team team = new Team();

    team.addMember(member1);
    team.addMember(member2);

    teamRepository.save(team);

    // when
    teamRepository.delete(team);

    // then
    List<Team> teams = teamRepository.findAll();
    List<Member> members = memberRepository.findAll();

    assertThat(teams).hasSize(0);
    assertThat(members).hasSize(0);
}
```

<details>
<summary>query</summary>
<div markdown="1">      
    ```
    Hibernate: 
        insert 
        into
            team
            (id, name) 
        values
            (null, ?)
    Hibernate: 
        insert 
        into
            member
            (id, name, team_id) 
        values
            (null, ?, ?)
    Hibernate: 
        insert 
        into
            member
            (id, name, team_id) 
        values
            (null, ?, ?)
    
    Hibernate: 
        delete 
        from
            member 
        where
            id=?
    Hibernate: 
        delete 
        from
            member 
        where
            id=?
    Hibernate: 
        delete 
        from
            team 
        where
            id=?
    ```

</div>
</details>
    
    

delete 쿼리가 총 3번 나가는 걸 확인할 수 있다. 즉, Team(부모)가 삭제될 때 Member(자식)도 영속성 전이 옵션으로 인해 함께 삭제

### 부모 엔티티에서 자식 엔티티를 제거

```java
@DisplayName("CascadeType.REMOVE - 부모 엔티티(Team)에서 자식 엔티티(Member)를 제거하는 경우")
@Test
void cascadeType_Remove_InCaseOfMemberRemovalFromTeam() {
    // given
    Member member1 = new Member();
    Member member2 = new Member();

    Team team = new Team();

    team.addMember(member1);
    team.addMember(member2);

    teamRepository.save(team);

    // when
    team.getMembers().remove(0);

    // then
    List<Team> teams = teamRepository.findAll();
    List<Member> members = memberRepository.findAll();

    assertThat(teams).hasSize(1);
    assertThat(members).hasSize(2);
}
```

<details>
<summary>query</summary>
<div markdown="1">   
    ```
    Hibernate: 
        insert 
        into
            team
            (id, name) 
        values
            (null, ?)
    Hibernate: 
        insert 
        into
            member
            (id, name, team_id) 
        values
            (null, ?, ?)
    Hibernate: 
        insert 
        into
            member
            (id, name, team_id) 
        values
            (null, ?, ?)
    ```
    

</div>
</details>

delete 쿼리가 전혀 나가지 않는다. 영속성 전이 삭제 옵션은 부모와 자식의 관계가 끊어졌다 해서 자식을 삭제하지 않기 때문이다.

# What is (orphanrRemoval = true)?

## 고아객체(orphanrRemoval )

*부모 엔티티와 연관관계가 끊어진 자식 엔티티를 의미한다.* 

- 고아 객체 제거 : 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제.
- **orphanRemoval = true**
- Parent parent1 = em.find(Parent.class, id);parent1.getChildren().remove(0);자식 엔티티를 컬렉션에서 제거함 ➡ 부모객체와의 연관관계 끊어짐
- DELETE FROM CHILD WHERE ID = ? ➡ DELETE 쿼리 발생한다.

## ****orphanRemoval = true****

`orphanRemoval = true` 또한 부모 엔티티가 삭제되면 자식 엔티티도 삭제된다. 

따라서 `CascadeType.PERSIST`를 함께 사용하면, 이때도 부모가 자식의 전체 생명 주기를 관리하게 된다.

한편, 이 옵션의 경우에는 부모 엔티티가 자식 엔티티의 관계를 제거하면 자식은 고아로 취급되어 그대로 사라진다.

학습 테스트를 위해 Team 엔티티에 고아 객체 옵션을 추가

```java
@Entity
public class Team {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(
        mappedBy = "team",
        fetch = FetchType.LAZY,
        cascade = CascadeType.PERSIST,
        orphanRemoval = true
    )
    private List<Member> members = new ArrayList<>();
}
```

### 이전과 동일하게 부모 엔티티를 삭제

<details>
<summary>query</summary>
<div markdown="1">     
    ```
    // DML
    Hibernate: 
        insert 
        into
            team
            (id, name) 
        values
            (null, ?)
    Hibernate: 
        insert 
        into
            member
            (id, name, team_id) 
        values
            (null, ?, ?)
    Hibernate: 
        insert 
        into
            member
            (id, name, team_id) 
        values
            (null, ?, ?)
    
    Hibernate: 
        select
            team0_.id as id1_1_,
            team0_.name as name2_1_ 
        from
            team team0_
    
    Hibernate: 
        delete 
        from
            member 
        where
            id=?
    ```
</div>
</details>
    

이전과는 다르게 delete 쿼리가 1번 나간다. 고아 객체 옵션은 부모와 자식의 관계가 끊어지면 자식을 고아로 취급하고 자식을 삭제하기 때문이다.

# 결과 비교

- 부모 엔티티 삭제
    - `CascadeType.REMOVE`와 `orphanRemoval = true`는 부모 엔티티를 삭제하면 자식 엔티티도 삭제한다.
- 부모 엔티티에서 자식 엔티티 제거
    - `CascadeType.REMOVE`는 자식 엔티티가 그대로 남아있는 반면, `orphanRemoval = true`는 자식 엔티티를 제거한다.

## 정리

1. 부모 엔티티가 삭제되면 자식 엔티티도 삭제된다 => REMOVE
2. 부모 엔티티에서 자식만 삭제하고 싶다 => orphanRemoval = true

## 주의점

두 케이스 모두 자식 엔티티에 딱 하나의 부모 엔티티가 연관되어 있는 경우에만 사용해야 한다.

예를 들어 Member(자식)을 Team(부모)도 알고 Parent(부모)도 알고 있다면, `CascadeType.REMOVE` 또는 `orphanRemoval = true`를 조심할 필요가 있다. 자식 엔티티를 삭제할 상황이 아닌데도 어느 한쪽의 부모 엔티티를 삭제했거나 부모 엔티티로부터 제거됐다고 자식이 삭제되는 불상사가 일어날 수 있기 때문이다.

그러므로 `@OneToMany`에서 활용할 때 주의를 기울이고, @ManyToMany에서는 활용을 지양하자.

- 게시판의 댓글일 경우 사용가능하고, 파일 엔티티의 경우 공용으로 ? 사용하기에 쓰면 안됨

---

https://tecoble.techcourse.co.kr/post/2021-08-15-jpa-cascadetype-remove-vs-orphanremoval-true/

https://prodo-developer.tistory.com/151

https://yeon-kr.tistory.com/196