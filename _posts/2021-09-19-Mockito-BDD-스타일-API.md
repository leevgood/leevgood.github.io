# Mockito : Mockito BDD 스타일 API

<br>

BDD : 애플리케이션이 어떻게 <b>행동</b>해야 하는지에 대한 이해를 구성하는 방법으로 TDD에서 창안하였다.

<br>

행동에 대한 스펙
- Title
- Narrrative(설명)
    - As a(어떤 역할) / I want(~하다) / so that(그래서)
- Acceptance criteria
    - given(주어진 상황) / when(~하면) / then(~이 반환)

<br>

when->given
```
given(memberService.findById(1L)).willReturn(Optional.of(member));
given(studyRepository.save(study)).willReturn(study);
```

<br>

verify->then
```
then(memberService).should(times(1)).notify(study);
then(memberService).shouldHaveNoMoreInteractions();
```

<br>

## 출처 
---
백기선님의 더 자바, 애플리케이션을 테스트하는 다양한 방법 강의(인프런)를 참조하여 작성하였습니다.