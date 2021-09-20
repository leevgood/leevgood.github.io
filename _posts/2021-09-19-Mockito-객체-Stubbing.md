# Mockito : Mockito 객체 Stubbing

<br>

## 모든 Mock 객체의 행동
-  null을 리턴한다. (Optional 타입은 Optional.empty 리턴)
-  Primitive 타입은 기본 Primitive 값.(예를 들어 boolean은 false)
-  콜렉션은 비어있는 콜렉션
-  void 메서드는 예외를 던지지 않고 아무런 일도 발생하지 않는다.

<br>

## Mock 객체를 조작해서
- 특정한 매개변수를 받은 경우 특정한 값을 리턴하거나 예외를 던지도록 할 수 있다.
    - https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#2
    - https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#3
- Void 메소드 특정 매개변수를 받거나 호출된 경우 예외를 발생 시킬 수 있다. 
    - https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#5
- 메소드가 동일한 매개변수로 여러번 호출될 때 각기 다르게 행동호도록 조작할 수도 있다.
    - https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#10

<br>

## 출처 
---
백기선님의 더 자바, 애플리케이션을 테스트하는 다양한 방법 강의(인프런)를 참조하여 작성하였습니다.