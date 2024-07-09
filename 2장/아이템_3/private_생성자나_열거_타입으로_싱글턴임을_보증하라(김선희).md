# 싱글턴이란?
인스턴스를 오직 하나만 생성할 수 있는 클래스

* 함수(무상태 객체)
* 설계상 유일해야 하는 시스템 컴포넌트

## 싱글턴의 단점
클라이언트가 테스트하기 어렵다.

## 싱글턴을 만드는 방식
생성자는 private 으로 감추고, 인스턴스에 접근할 때는 `public static` 멤버를 만든다

### 1️⃣ public static 멤버가 final 필드인 경우
```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {
        //
    }

    public void leaveTheBuilding() {
        //
    }
}
```

* private 생성자는 `Elvis.INSTANCE` 를 초기화할 때 딱 한 번만 호출된다
* public 이나 protected 생성자가 없으므로 Elvis 클래스가 초기화될 때 인스턴스가 전체 시스템에서 하나뿐임을 보장한다

이렇게 되면 클라이언트는 어떤 방식으로든 객체를 생성할 수 없다.

하지만, 예외가 있다.
* `예외` : 권한이 있는 클라이언트가 `AccessibleObject.setAccessible` 을 사용해 private 생성자를 호출할 수 있다.
* `방어` : 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던진다.

### public static 멤버가 final 필드인 경우 장점
1. 해당 클래스가 final 이므로 싱글턴임이 명백하게 드러난다는 것이다.
2. 간결함이다.

### 2️⃣ 정적 팩터리 메서드를 public static 멤버로 제공
```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {
        //
    }

    public static Elvis getInstance() {
        return INSTANCE;
    }

    public void leaveTheBuilding() {
        //
    }
}
```

### 3️⃣ 원소가 하나인 열거 타입을 선언
```java
public class Elvis {
    INSTANCE;
}
```

### 원소가 하나인 열거 타입을 선언의 장점
* public 필드 방식보다 비슷하지만, 더 간결하고, 추가 노력 없이 직렬화할 수 있다
* 리플렉션 공격에서도 인스턴스가 생기는 일을 완벽히 막아준다
* **원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다**

### 단 만드려는 싱글턴이 Enum 이외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다

### 정적 팩터리 메서드를 public static 멤버로 제공하는 경우 장점
1. API를 변경하지 않고도 싱글턴이 아니게 변경할 수 있다
2. 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다
3. 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다

> 굳이 이러한 장점이 필요하지 않다면 `public` 필드 방식이 좋다

### public static 멤버가 final 필드인 경우 + 정적 팩터리 메서드를 public static 멤버로 제공하는 경우 단점
`문제` : 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어진다.

-> 도통 무슨 말인지 이해가 안가네

`해결` : readResolve 메서드 추가
```java
public class Elvis {
    INSTANCE;

    private Object readResolve() {
        return INSTANCE;
    }
}
```

