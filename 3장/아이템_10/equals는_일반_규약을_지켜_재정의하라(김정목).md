# equals는 일반 규약을 지켜 재정의하라.
equals 메서드는 재정의하기 쉬워 보이지만 자칫하면 끔찍한 결과를 초래한다. 그렇기 때문에 재정의하지 않는 것이 가장 좋은 방법이다. <br><br>

1️⃣ 각 인스턴스가 본질적으로 고유하다. <br>
-> 각 인스턴스가 본질적으로 고유하다. <br>
2️⃣ 인스턴스의 '논리적 동치성' 을 검사할 일이 없다. <br>
-> equals를 재정의해서 논리적 동치성을 검사하는 방법이 있다고 쳐도 설계자는 클라이언트가 이 방식을 원하지 않거나 애초에 필요하지 않다고 판단할 수 있다. <br>
3️⃣ 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다. <br>
-> 대부분의 Set 구현체는 AbstractSet이 구현한 equals를 상속받아 쓰고, List 구현체들은 AbstractList로부터, Map 구현체들은 AbstractMap으로부터 상속받아 그대로 쓴다. <br>
4️⃣ 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다. <br>
<br>

그렇다면 equals를 재정의해야 할 때는 언제일까? <br>
<br>
바로 "객체 식별성이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때" 이다. <br>
두 값 객체를 equals로 비교하는 프로그래머는 객체가 같은지가 아니라 값이 같은지를 알고 싶어 할 것이다. 이럴 때 equals가 논리적 동치성을 확인하돌고 재정의해두면, 그 인스턴스는 값을 비교하길 원하는 프로그래머의 기대에 부응할 수 있다. <br>
<br>

equals 메서드를 재정의할 때는 반드시 일반 규약을 따라야 한다. 아래는 Object 명세에 적힌 규약이다. <br>
- 반사성(reflexivity): null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.
- 대칭성(symmetry): null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true다.
- 추이성(transitivity): null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true다.
- 일관성(consistency): null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
- null-아님: null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.

<br><br>

그렇다면 Object 명세에서 말하는 동치관계란 무엇일까? <br>
동치 관계를 만족시키기 위한 다섯 요건을 하나씩 살펴보자.<br>
- 반사성: 객체는 자기 자신과 같아야 한다는 뜻이다.
- 대칭성: 두 객체는 서로에 대한 동치 여부에 똑같이 대답해야 한다는 뜻이다.

```java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    @Override public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
        if (o instanceof String)  
            return s.equalsIgnoreCase((String) o);
        return false;
    }
```

위 코드를 기반으로 아래 코드를 작성해 보면, 

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";
```

cis.equals(s) 는 true를 반환한다. 하짐나 s.equals(cis)는 false를 반환하며 대칭성을 위반한다. <br>

위와 같은 문제를 해결하기 위해서는 CaseInsensitiveString의 equals를 String과도 연동하겠다는 허황한 꿈을 버려야 한다. 그렇다면 아래와 같은 코드가 나온다. <br>

```java
@Override public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString &&
            ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```
<br>

- 추이성 : 첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 객체도 같아야 한다는 뜻이다. <br>
```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point)o;
        return p.x == x && p.y == y;
    }
```

위 코드를 확장해서 점에 색상을 더해보았다.

</br> 

```java
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
```

이때 그대로 둔다면 색상 정보는 무시한 채 비교를 수행한다.  <br>

```java
    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
```

위 코드는 일반 Point를 ColorPoint에 비교한 결과와 그 둘을 바꿔 비교한 결과가 다를 수 있다. <br>

