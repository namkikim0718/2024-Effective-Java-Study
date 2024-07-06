# private 생성자나 열거 타입으로 싱글톤임을 보증하다.
</br>

## 싱글톤(Singleton) 이란? <br>
- 인스턴스를 오직 하나만 생성할 수 있는 클래스
</br></br>

## 싱글톤을 만드는 방법 3가지

1. private 생성자와 public static 필드
```java
public class Elly {
    public static final Elly INSTANCE = new Elly();
    private Elly() { }

    public void sleep() { }
    ...
}
Elly elly1 = Elly.INSTANCE;
Elly elly2 = Elly.INSTANCE;

System.out.println(elly1 == elly2); //true
System.out.println(elyl1.equals(elly2)); //true
```
- elly1 과 elly2 는 같은 객체를 반환
- public 이나 potected 생성자가 없으므로 Elly 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 단 하나뿐임을 보장한다.
- 예외는 딱 한가지가 존재 `AccessibleObject.setAccessbile` 을 통한 private 생성자 호출 (리플렉션 API)
```java
        try {
            // 리플렉션을 통해 private 생성자에 접근
            Constructor<Elly> constructor = Elly.class.getDeclaredConstructor();
            constructor.setAccessible(true); // private 접근 허용
            Elly elly2 = constructor.newInstance();

            // 두 인스턴스가 동일한지 확인
            System.out.println(elly1 == elly2); // false
        } catch (Exception e) {
            e.printStackTrace();
        }
```
- 예외 예시
</br></br>
장점 : 해당 클래스가 싱글톤임이 명백히 드러나며 간결하다.
</br></br>

2. private 생성자와 public static 정적 팩토리 메소드
```java
public class Elly {
    private static final Elly INSTANCE = new Elly();
    private Elly() { }

    public static Elly getInstance() {
        return INSTANCE;
    }
}
```
- Elly.getInstance() 는 항상 같은 객체의 참조를 반환한다.
- 즉 또 다른 Elly 는 만들어지지 않다.
</br></br>
장점
- API 를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.
  - getinstance() 메소드가 호출하는 스레드별로 다른 인스턴스를 넘겨주게 할 수 있다.
- 제네릭 싱글턴 팩토리로 만들 수 있다.
</br></br>

3. 원소가 하나인 Enum 타입 선언
- 1번 방식과 비슷하지만, 더 간결하고 추가 노력 없이 직렬화할 수 있다.
- 아주 복잡한 직렬화 상황이나, 리플렉션 공격에서도 제 2의 인스턴스 생성을 막을 수 있다.
```java
public enum Elly {
    INSTANCE;
    
    public void sleep() { ... }
}
```
</br></br>

## 결론
- 책에서는 열거 타입의 싱글턴이 가장 좋은 방법이라고는 하지만, 꼭 이를 고집할 필요는 없다.
- 사실 코드에 열거 타입이 있을 때 클래스 다이어그램만 보고 싱글톤인지 한 번에 알기 어려운 경우가 많기 떄문이다.
- 하지만 싱글턴의 제약 조건을 가장 잘 만족 시키는 것은 열거 타입이기 때문에 취향껏 사용하면 된다.
- 스프링부트에 경우에는 알아서 싱글톤을 보장하기 때문에 이를 직접 구현할 필요는 없을 것이다.

