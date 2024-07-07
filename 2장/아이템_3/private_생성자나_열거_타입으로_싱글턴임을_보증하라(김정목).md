# private 생성자나 열거 타입으로 싱글턴임을 보증하라. <br>

싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다. 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다. 

## 📌 싱글턴의 한계
1️⃣ private 생성자를 가지고 있기 때문에 상속할 수 없다.
2️⃣ 싱글턴은 테스트하기가 어렵다.
3️⃣ 서버환경에서는 싱글턴이 하나만 만들어지는 것을 보장하지 못한다.
4️⃣ 싱글톤의 사용은 전역 상태를 만들 수 있기 때문에 바람직하지 못하다.

## 📌 싱글턴 사용 이유
한 번의 객체 생성으로 재사용이 가능하기 때문에 메모리 낭비를 방지할 수 있다.

## 📌 싱글턴을 만드는 방식
1️⃣ public static 멤버가 final 필드인 방식
```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() { ... }

	public void leaveTheBuilding()
}
```

### 위와 같이 작성하면 장점
1. 해당 클래스가 싱글턴임이 API에 명백히 드러난다.
2. 간결하다.

2️⃣ 정적 팩터리 메서드를 public static 멤버로 제공하는 방식
```java
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	private Elvis() { ... }
	public static Elvis getInstance() { return INSTANCE; }

	public void leaveTheBuilding()
}
```


// 공부 부족.. 
