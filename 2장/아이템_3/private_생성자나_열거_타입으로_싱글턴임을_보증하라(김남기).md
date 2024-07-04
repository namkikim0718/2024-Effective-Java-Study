# private 생성자나 열거 타입으로 싱글턴임을 보증하라.
</br>

> `싱글턴(singleton)`이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.

그러나 인터페이스를 구현해서 만든 싱글턴이 아니라 클래스 자체를 싱글턴으로 만들면 그 인스턴스를 가짜(mock) 구현으로 대체할 수 없어 테스트하기가 어려워질 수 있다.

</br>

## 싱글턴을 만드는 방식
우선 생성자는 `private`으로 감춰두고, 유일한 인스턴스에 접근할 수 있는 수단으로 `public static` 멤버를 하나 마련해둔다.

</br>

### 1. `public static` 멤버가 `final` 필드인 방식
```java
// 코드 3-1 public static final 필드 방식의 싱글턴 (23쪽)
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() { }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }
```
위의 방식은 `public`이나 `protected`생성자가 없으므로 Elvis 클래스가 초기화될 때 딱 한번만 생성되므로 전체 시스템에서 하나뿐임이 보장된다.
그리고 `public static` 필드가 `final`이라 절대로 다른 객체를 참조할 수 없다.
</br>

```
<장점>
1. 해당 코드가 싱글턴임이 API에 명백히 드러난다.
2. 간결하다.
```
</br>

> ___<예외 상황>___ </br>
> 권한이 있는 클라이언트가 `리플렉션 API(아이템 65)`인 `AccessibleObject.setAccessible`을 사용해 private 생성자를 호출할 수 있다.</br>
> -> 이러한 공격을 방어하려면 두번째 객체가 생성되려 할 때 예외를 던지게 하면 된다.

</br></br>

### 2. 정적 팩터리 메서드를 `public static` 멤버로 제공하는 방식
```java
// 코드 3-2 정적 팩터리 방식의 싱글턴 (24쪽)
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }
```
위 방식은 `Elvis.getInstance`가 항상 같은 객체의 참조를 반환하므로 다른 인스턴스가 결코 만들어지지 않는다.(리플렉션을 통한 예외는 동일하게 적용)
</br>

```
<장점>
1. API를 바꾸지 않고도 언제든지 싱글턴이 아니게 변경할 수 있다.
2. 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다는 장점이 있다.(아이템 30)
3. 정적 팩터리의 메서드 참조를 공급자(suplier)로 사용할 수 있다.(아이템 43, 44)
```
</br>

#### <직렬화 방법>
위의 두가지 방식으로 만든 싱글턴 클래스를 직렬화하려면 단순히 `Serialiazable`을 구현한다고 선언하는 것 만으로는 부족하다.
모든 인스턴스 필드를 `일시적(transient)`이라고 선언하고 `readResolve` 메서드를 제공해야 한다.
이렇게 하지 않으면 역직렬화 하는 과정에서 새로운 인스턴스가 만들어진다.
`정적 팩터리 메서드 방식`에서는 가짜 Elvis가 탄생한다는 의미이다. 이를 막기 위해서는 다음 `readResolve` 메서드를 추가해야한다.

```java
// 싱글턴임을 보장해주는 readResolve 메서드
private Object readResolve() {
  // '진짜' Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
  return INSTANCE;
}
```
</br></br>

### 3. 원소가 하나인 열거 타입을 선언하는 방식(권장)
```java
// 열거 타입 방식의 싱글턴 - 바람직한 방법 (25쪽)
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() {
        System.out.println("기다려 자기야, 지금 나갈께!");
    }
```
`public 필드 방식`과 유사하지만, 더 간결하고, 쉽게 직렬화할 수 있으며, 예외 상황이던 리플렉션 공격도 완벽히 막아준다.
___대부분의 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.___

> 단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야한다면 이 방법은 사용할 수 없다.(열거 타입이 다른 인터페이스를 구현하도록 선언할 수는 있다)
