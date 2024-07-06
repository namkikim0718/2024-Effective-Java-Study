# try-finally 보다는 try-with-resources 를 사용하라
</br>

### JAVA 라이브러리의 직접 닫아줘야 하는 자원 <br>
- 자바 라이브러리에는 close 메소드를 호출해 직접 닫아줘야 하는 자원이 많다.
- InputStream , OutputStream , java.sql.Connection ...

### try-with-resources 사용법
- 특정 클래스에서 사용하기 위해서는 AutoClosable 인터페이스를 구현해야 한다.
- 이 인터페이스를 구현하면 try-with-resources 를 사용할 때 try 가 끝나는 시점에 close() 가 자동으로 호출된다.
```java
try (Resourc resource = new Resource()) {
  // 리소스를 사용하는 코드
```
- try 가 끝나면 자동으로 AutoClosable 의 close() 가 호출이 된다.
- 즉 해당 인터페이스를 구현하는 메소드에서 자원을 반납하는 방법을 여기에 정의해주면 된다.
- 이 메서드에서 예외를 던지지 않는다.

</br>

### 전통적인 자원 회수 방식 -> try-finally
- 전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 try-finally 가 쓰인다.
- 이는 예외가 발생하거나 메서드에서 반환되는 경우까지도 포함하여 사용되었다.
```java
    static String firstLineOfFile(String path) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(path));
        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }
```
</br></br>

### 자원이 둘 이상이면 try-fianlly 방식은 너무 지저분해진다.

```java
private static final int BUFFER_SIZE = 0; // 편의상 0으로 지정

    static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0) {
                    out.write(buf, 0, n);
                }
            } finally {
                out.close();
            }
        } finally {
            in.close();
        }
    }
```
- 예외는 try 블록과 finally 블록 모두에서 발생할 수 있는데, 만약 기계의 물리적인 결함으로 readline() 이 실패하는 경우 try 에서 readLine() 대한 실패가, finally 에서 close() 에 대한 실패가 발생한다.
- 이러한 경우는 시스템상에 close() 에 대한 예외처리만 잡기 떄문에 물리적 결함에 대한 오류 파악이 매우 어렵다는 단점이 있다.

</br></br>

### 복수의 자원을 처리하는 try-with-resources

```java
private static final int BUFFER_SIZE = 0; // 편의상 0으로 지정

    static void copy(String src, String dst) throws IOException {
        try (InputStream in = new FileInputStream(src);
             OutputStream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0) {
                out.write(buf, 0, n);
            }
        }
    }
```
- 만약 예외가 발생한다면, close 에서 발생한 예외는 숨겨지고 readLine 에서 발생한 예외가 기록된다.
- 이렇게 숨겨진 예외들은 그냥 버려지지는 않고, 스택 추적 내역에 출력이 ㅗ딘다.
- 자바 7에서 Throwable 에 추가된 getSuppressed 메서드를 이용하면 프로그램 코드에서 가져올 수도 있다.

</br></br>

### try-with-resources 를 catch 절과 함께 쓰기

```java
    static String firstLineOfFile(String path, String defaultVal) {
        try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            return br.readLine();
        } catch (IOException e) {
            return defaultVal;
        }
    }
```
- 보통의 try-finally 에서처럼 try-with-resources 에서도 catch 절을 쓸 수 있다.
- catch 절 덕분에 try 문을 더 중첩하지 않고도 다수의 예외를 처리할 수 있다.

</br></br>

### try-with-resources 장점
1. `리소스 누수 방지`
- 모든 리소스가 제대로 닫히도록 보장
- 실수로 finally 블록을 적지 않거나, finally 블럭 안에서 자원 해제 코드를 누락하는 문제를 예방

2. `코드 간결성 및 가독성 향상`
- 명시적인 close() 호출이 필요 없어 코드가 더 간결하고 읽기 쉬워진다.

3. `스코프 범위 한정`

4. `조금 더 빠른 해제`
- try-catch-finally 는 catch 이후에 자원을 반납하지만 try-with-resources 구문은 try 블럭이 끝나는 즉시 close() 호출한다.

</br></br>

## 정리
- 꼭 회수해야 하는 자원을 다룰 때는 try-fianlly 말고, try-with-resources 를 사용하자.
- 코드는 더 짧고 분명해지면서, 만들어지는 예외 정보도 훨씬 유용하다.
- try-finally 로 작성하면 실용적이지 못할 만큼 코드가 지저분해지지만 try-with-resources 로는 정확하고 쉽게 자원을 회수할 수 있다.
  











