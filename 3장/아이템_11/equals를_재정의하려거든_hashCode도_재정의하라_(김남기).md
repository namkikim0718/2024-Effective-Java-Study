# equals를 재정의하려거든 hashCode도 재정의하라.
</br>

`equals`를 재정의한 클래스 모두에서 `hashCode`도 재정의하지 않으면 `hashCode` 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 `HashMap`이나 `HashSet`같은 컬렉션의 원소로 사용할 때 문제를 일으킨다.
</br>
</br>

## Object 명세에서 발췌한 규약
</br>

```
1. equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇번을 호출해도 항상 같은 값을 반환해야한다.(어플리케이션 다시 실행하면 값이 달라져도 상관 X)

2. equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.

3. equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다.(다른 객체면 다른 값을 반환해야 성능이 좋아진다)
```
</br>
</br>

### hashCode 재정의를 잘못했을 때 크게 문제가 되는 조항은 2번 규약이다.
논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.
</br>

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");

// 실행하면 null을 반환할 것이다!
m.get(new PhoneNumber(707, 867, 5309));
```

`PhoneNumber` 클래스는 `hashCode`를 재정의하지 않았기 때문에 논리적 동치인 두 객체가 서로 다른 해시코드를 반환하여 2번 규약을 지키지 못한다.

> 두 인스턴스를 같은 버킷에 담더라도 `HashMap`은 해시코드가 다른 엔트리끼리 동치성 비교를 시도조차 하지 않도록 최적화되어 있어 여전히 `null`을 반환하게 된다.
</br>
</br>


### 만약 아래의 코드로 hashCode를 구현하면 어떨까?

```java
// 최악의 hashCode 구현
@Override public int hashCode() {return 42;}
```
</br>

모든 객체에 대해서 똑같은 값만 내어주어 해시테이블의 버킷 하나에 모두 담겨 마치 연결 리스트처럼 동작한다.
그 결과 해시테이블은 평균 수행 시간이 `O(1)`에서 `O(n)`으로 느려져 객체가 많아지면 쓸 수 없다.
</br>
</br>

## 좋은 hashCode를 작성하는 간단한 요령
</br>

1. int 변수 `result`를 선언한 후 값 c 로 초기화 한다.
2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.
    1. 해당 필드의 해시코드 c를 계산한다.
        1. 기본 타입 필드라면, `Type.hashCode(f)`를 수행한다.(`Type`은 박싱 클래스)
        2. 참조 타입 필드면서 이 클래스의 `equals` 메서드가 이 필드의 `equals`를 재귀적으로 호출해 비교한다면, 이 필드의 `hashCode`를 재귀적으로 호출한다. (복잡하면 `표준형`을 만들어 사용, `null`이면 0을 사용)
        3. 필드가 배열이라면 핵심 원소 각각을 별도 필드처럼 다룬다. (배열에 핵심 원소가 하나도 없으면 0을 사용, 모든 원소가 핵심 원소라면 `Arrays.hashCode`를 사용)
    2. 단계 2.a에서 계산한 해시코드 c로 `result`를 갱신한다.
        
        ```java
        result = 31 * result + c;
        ```
        
3. `result`를 반환한다.
</br>

> 파생 필드는 해시코드 계산에서 제외해도 되며 `equals` 비교에 사용되지 않은 필드는 *“**반드시”*** 제외해야한다.
</br>
</br>


### 전형적인 hashCode 메서드

```java
// 코드 11-2 전형적인 hashCode 메서드 (70쪽)
    @Override public int hashCode() {
        int result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        return result;
    }
```
</br>

이 메서드는 `PhoneNumber` 인스턴스의 핵심 필드 3개만을 사용해 간단한 계산만 수행한다.
그 과정에 비결정적 요소는 전혀 없으므로 동치인 인스턴스들은 같은 해시코드를 가질 것이 확실하다.
</br>

> 단, 해시 충돌이 더욱 적은 방법을 꼭 써야 한다면 구아바의 `com.google.common.hash.Hasing`을 참고하자!
</br>
</br>


### Objects의 hash 메서드를 사용한 한 줄짜리 hashCode 메서드

```java
// 코드 11-3 한 줄짜리 hashCode 메서드 - 성능이 살짝 아쉽다. (71쪽)
    @Override public int hashCode() {
        return Objects.hash(lineNum, prefix, areaCode);
    }
```
</br>

`Objects` 클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메서드인 `hash`를 제공한다.
이 메서드를 활용하면 `hashCode` 함수를 단 한 줄로 작성할 수 있다. (속도는 더 느리므로 성능에 민감하지 않은 상황에서만 사용하자)
</br>
</br>

### 해시코드를 지연 초기화하는 hashCode 메서드

```java
   // 해시코드를 지연 초기화하는 hashCode 메서드 - 스레드 안정성까지 고려해야 한다. (71쪽)
    private int hashCode; // 자동으로 0으로 초기화된다.

    @Override public int hashCode() {
        int result = hashCode;
        if (result == 0) {
            result = Short.hashCode(areaCode);
            result = 31 * result + Short.hashCode(prefix);
            result = 31 * result + Short.hashCode(lineNum);
            hashCode = result;
        }
        return result;
    }
```
</br>

클래스가 불변이고 해시코드를 계산하는 비용이 크면, 매번 새로 계산하기 보다는 캐싱하는 방식을 고려해야 한다.
</br>

> 필드를 지연 초기화하려면 그 클래스의 스레드 안정성까지 고려해야 한다.
> `hashCode` 필드의 초깃값은 흔히 생성되는 객체의 해시코드와는 달라야 함에 유념하자.
</br>
</br>


### 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다.
</br>

→ 속도야 빨라지겠지만, 해시 품질이 나빠져 해시테이블의 성능을 심각하게 떨어뜨릴 수도 있다.

</br>
</br>

### hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자.
### 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.
</br>

> `String`과 `Integer`를 포함해 자바 라이브러리의 많은 클래스들이 `hashCode` 메서드가 반환하는 정확한 값을 알려주는 것은 바람직하지 않은 실수다!
