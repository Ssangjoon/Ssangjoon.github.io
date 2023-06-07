---
title: spring security ContextHolder는 왜?
date: 2023-05-28 10:00:00 +0900
categories: [개인프로젝트]
tags: [개인프로젝트,GettingThingsDone,SpringSecurity]     # TAG names should always be lowercase
published: true
---
SpringSecurity를 무작적 써보며 애를 많이 먹었는데 그 중에 특히

마치 Authentication(인증 객체)를 담기 위한 저장공간 정도로 착각하여 한참을 헤메었다.…

`ContextHolder`는디폴트로 쓰레드를 단위로 이뤄진다.
때문에 이전 요청에서 저장한 인증과는 무효한데  왜 이전에 저장한 값이 있어도 null로 나오는지 이해가 안되기도 하였고, 

디비나 메모리에 저장하면 저 과정이 불필요한줄 알고 생략하였더니, `authenticationEntryPoint` 에에러**?** 에러가 나는 (지금 생각해보면 당연한 )결과가 나오기도…

저장 전략관련한 글을 보니 속성을 바꿀수도 있다고 나와있으나 아직 찾아보지는 그런 용도로 사용할 필요가 없어보인다. 아직까지는.