# Mockito : Mockito 시작하기

<br>

스프링 부트 2.2버전 이상의 프로젝트 생성 시 spring-boot-starter에서 자동으로 Mockito를 추가해준다.

<br>

스프링부트를 쓰지 않는다면 의존성을 직접 추가해준다.

```
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>3.1.0</version>
    <scope>test</scope>
</dependency>


<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-junit-jupiter</artifactId>
    <version>3.1.0</version>
    <scope>test</scope>
</dependency>

```

<br>

다음 세가지만 알면 Mock을 활용한 테스트를 쉽게 작성 가능하다.
- Mock 만드는 법
- Mock이 어떻게 작동하는지 관리하는 법
- Mock의 행동을 검증하는 법


## 출처 
---
백기선님의 더 자바, 애플리케이션을 테스트하는 다양한 방법 강의(인프런)를 참조하여 작성하였습니다.