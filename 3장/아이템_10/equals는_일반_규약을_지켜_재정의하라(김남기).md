# equals는 일반 규약을 지켜 재정의하라.

> equals 메서드는 함부로 재정의를 했다가 끔찍한 결과를 초래할 수 있다.

</br> 
 

### 다음에서 열거한 상황중 하나에 해당한다면 재정의하지 않는 것이 최선이다.

- 각 인스턴스가 본질적으로 고유하다.
    - 값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스가 여기 해당한다.
    - 예시로 `Thread`의 `equals` 메서드는 이러한 클래스에 딱 맞게 구현되었다.
</br> 

- 인스턴스의 `논리적 동치성(logical equality)`를 검사할 일이 없다.
    - 정규표현식을 비교하는 `java.util.regex.Pattern`과 같은 논리적 동치성을 검사하는 것이 아니면 `Object`의 기본 `equals`만으로 해결된다.

</br> 

- 상위 클래스에서 재정의한 `equals`가 하위 클래스에도 딱 들어맞는다.
    - ex) `Set` 구현체는 `AbstractSet`이 구현한 `equals`를 상속받아 쓰고, `List` 구현체는 `AbstractList`로부터 상속받아 그대로 쓴다.

</br> 

- 클래스가 `private`이거나 `package-private`이고 `equals` 메서드를 호출할 일이 없다.

</br> 

***그렇다면 `equals`를 재정의해야 할 때는 언제일까?***

</br> 

객체 식별성(두 객체가 물리적으로 같은가)이 아니라 `논리적 동치성`을 확인해야 하는데, 상위 클래스의  `equals`가 `논리적 동치성`을 비교하도록 재정의되지 않았을 때다.

`Integer`와 `String`처럼 값을 표현하는 클래스는 프로그래머가 값이 같은지를 알고 싶어할 것이다.

</br> 

이 때, 논리적 동치성을 확인하도록 재정의하면 값이 같은지는 물론 `Map`의 키와 `Set`의 원소로도 사용할 수 있게 된다.

> Enum과 같이 논리적으로 같은 인스턴스가 2개 이상 만들어지지 않는다면 재정의하지 않아도 된다.
> 이런 상황에서는 논리적 동치성과 객체 식별성이 똑같은 의미가 된다.
 
</br> 

</br> 


## 동치관계란 무엇일까?

`Object` 명세에서 말하는 동치관계는 쉽게 말해 집합을 서로 같은 원소들로 이뤄진 부분집합으로 나눴을 때, 이 부분집합을 `동치류(동치 클래스)`라 한다.

`equals` 메서드가 쓸모 있으려면 모든 원소가 같은 동치류에 속한 어떤 원소와도 서로 교환할 수 있어야 한다.

</br> 

이제 동치관계를 만족 시키기 위한 다섯 요건을 보자.

</br> 

## 1. 반사성(reflexivity)

`null`이 아닌 모든 참조 값 x에 대해 `x.equals(x)`는 `true`다.

> 단순히 말하면 객체는 자기 자신과 같아야 한다는 뜻이다. 
> 이 요건은 일부로 어기는 경우가 아니라면 만족시키지 못하기가 더 어려워 보인다.


</br> 

## 2. 대칭성(symmetry)

`null`이 아닌 모든 참조 값 x, y에 대해 `x.equals(y)`가 `true`면 `y.equals(x)`도 `true`다.
두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다.

</br> 

```java
// 코드 10-1 잘못된 코드 - 대칭성 위배! (54-55쪽)
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    // 대칭성 위배!
    @Override public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
        if (o instanceof String)  // 한 방향으로만 작동한다!
            return s.equalsIgnoreCase((String) o);
        return false;
    }
```

위의 코드에 기반하여 다음 코드를 작성했다면 어떻게 될까?

</br> 

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";
```

`cis.equals(s)`는 `true`를 반환하지만, `s.equals(cis)`는 `false`를 반환할 것이다.

</br> 

***이는 대칭성을 명백히 위반한다!***

</br> 

이 문제를 해결하려면 `equals` 메서드를 `String`과도 연동하려는 꿈을 버려야한다.

```java
@Override public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString &&
            ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

</br> 

</br> 

## 3. 추이성(transitivity)

`null`이 아닌 모든 참조 값 x, y, z에 대해 `x.equals(y)`가 `true`이고, `y.equals(z)`도 `true`면 `x.equals(z)`도 `true`다.

```java
// 단순한 불변 2차원 정수 점(point) 클래스 (56쪽)
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

위의 코드를 확장해 색상을 더해보자.

</br> 

```java
// Point에 값 컴포넌트(color)를 추가 (56쪽)
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
```

이때 `equals` 메서드는 그대로 둔다면 색상 정보는 무시한 채 비교를 수행할 것이다.  아래는 또다른 `ColorPoint`와 비교하는 코드이다.

</br> 

```java
// 코드 10-2 잘못된 코드 - 대칭성 위배! (57쪽)
    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
```

이렇게 되면 `Point`의 `equals`는 색상을 무시하고, `ColorPoint`의 `equals`는 입력 매개변수의 클래스 종류가 다르다며 매번 `false`만 반환할 것이다.

</br> 

```java
// 코드 10-3 잘못된 코드 - 추이성 위배! (57쪽)
    @Override public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;

        // o가 일반 Point면 색상을 무시하고 비교한다.
        if (!(o instanceof ColorPoint))
            return o.equals(this);

        // o가 ColorPoint면 색상까지 비교한다.
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
```

이 방법은 대칭성은 지켜주지만, 추이성은 깨버린다.

</br> 

또한 이 방식은 ***무한 재귀*** 에 빠질 위험도 있다.

`Point`의 또 다른 하위 클래스로 `SmellPoint`를 만들고 `equals`는 같은 방식으로 구현한 후, 
```java
myColorPoint.equals(mySmellPoint)
```
를 호출 하면 `StackOverflowError` 를 일으킨다.

</br> 

***그렇다면 해법은 무엇일까?***

</br> 

→ 객체 지향적 추상화의 이점을 포기하지 않는 한 구체 클래스를 확장해 새로운 값을 추가하면서 `equals` 규약을 만족시킬 방법은 없다.

> `equals` 안의 `instanceof` 검사를 `getclass` 검사로 바꾸면 규약도 지키고 구체 클래스를 상속할 수 있지 않을까?

</br> 


```java
// 잘못된 코드 - 리스코프 치환 원칙 위배! (59쪽)
    @Override public boolean equals(Object o) {
        if (o == null || o.getClass() != getClass())
            return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
```

위의 `equals`는 같은 구현 클래스의 객체와 비교할 때만 `true`를 반환한다.

괜찮아 보이지만 실제로 활용할 수는 없다.

</br> 

리스코프 치환 원칙에 따르면 `Point`의 하위 클래스는 정의상 여전히 `Point`이므로 어디서든 `Point`로써 활용될 수 있어야 하는데, 위의 코드는 그것을 위반한다.

</br> 

하지만, 우회 방법으로 상속 대신 컴포지션을 사용하는 것이 있다.

`Point`를 상속하는 대신 `Point`를 `ColorPoint`의 `private` 필드로 두고, 일반 `Point`를 반환하는 `뷰(view)`메서드를 `public`으로 추가하는 것이다.

```java
/**
  * 이 ColorPoint의 Point 뷰를 반환한다.
  */
public Point asPoint() {
    return point;
}
```

> ***추상 클래스*** 의 하위 클래스에서 라면 `equals` 규약을 지키면서도 값을 추가할 수 있다.
> 상위 클래스를 직접 인스턴스로 만드는게 불가능 하다면 지금까지의 문제들은 일어나지 않는다.
 
</br> 

</br> 


## 4. 일관성(consistency)

`null`이 아닌 모든 참조 값 x, y에 대해 `x.equals(y)`를 반복해서 호출하면 항상 `true`를 반환하거나 항상 `false`를 반환한다.

</br> 

두 객체가 같다면 수정되지 않는 한 앞으로도 영원히 같아야 한다.

클래스가 불변이든 가변이든 `equals`의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다.

</br> 

예시로 `java.net.URL`의 `equals`는 주어진 `URL`과 매핑된 호스트의 `IP` 주소를 이용해 비교하는데, 호스트 이름을 `IP` 주소로 바꾸려면 네트워크를 통해야 하므로, 그 결과가 항상 같다고 보장할 수 없다.

이런 문제를 피하려면 `equals`는 항시 메모리에 존재하는 객체만을 사용한 `결정적 계산`만 수행해야 한다.

</br> 

</br> 

## 5. null-아님

모든 객체가 `null`과 같이 않아야 한다.

> `o == null` 과 같이 명시적인 검사는 필요하지 않고, `instanceof`를 이용해 묵시적인 검사를 진행하면 된다.

</br> 

</br> 


## 양질의 equals 메서드 구현방법

1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형 변환한다.
4. 입력 객체와 자기 자신의 대응되는 ‘핵심’ 필드들이 모두 일치하는지 하나씩 검사한다.

</br> 

다음은 올바르게 작성해본 `equals` 메서드의 예이다.

```java
// 코드 10-6 전형적인 equals 메서드의 예 (64쪽)
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix   = rangeCheck(prefix,   999, "프리픽스");
        this.lineNum  = rangeCheck(lineNum, 9999, "가입자 번호");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max)
            throw new IllegalArgumentException(arg + ": " + val);
        return (short) val;
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }

    // 나머지 코드는 생략 - hashCode 메서드는 꼭 필요하다(아이템 11)!
}
```

</br> 

</br> 

## 마지막 주의사항

- `equals`를 재정의할 땐 `hashCode`도 반드시 재정의하자.

</br> 

- 너무 복잡하게 해결하려 들지 말자.
    - 필드들의 동치성만 검사해도 규약을 어렵지 않게 지킬 수 있다.
    - 일반적으로 별칭(alias)은 비교하지 않는게 좋다.

</br> 

- `Object` 외의 타입을 매개변수로 받는 `equals` 메서드는 선언하지 말자.
    - `Object` 타입이 아니라면 *재정의* 가 아니라 *다중정의* 하는 것이다.
    - `@Override` 애너테이션을 일관되게 사용하자.

</br> 

</br> 

## 핵심 정리

꼭 필요한 경우가 아니면 `equals`를 재정의하지 말자. 
웬만해서는 `Object`의 `equals`가 우리가 원하는 비교를 정확히 수행해준다.
재정의해야 할 때는 그 클래스의 핵심 필드 모두를 빠짐없이, 다섯가지 규약을 확실히 지켜가며 비교해야 한다.
