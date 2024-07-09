# try-finally?
java에서는 close 를 호출하여 닫아줘야 하는 자원이 있다

```
`InputStream`, `OutputStream`, `java.sql.Connection` 등
```

코드를 보자.

```java
public static String inputString() throws IOException {
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    String result = br.readLine();
    br.close();
    return result;
}
```

위 코드의 문제점은 예상치 못하게 `IOException` 이 발생하여 프로그램이 종료될 수 있다.

이를 `try-finally` 문을 사용하여 수정했다.

```java
public static String inputString() throws IOException {
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

자원을 한 개 더 사용해보자.
```java
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);

    try {
        OutputStream out = new FileOutputStream(dst);

        try {
            byte[] buf = new FileOutputStream(dst);
            int n;
            while ((n = in.read(buf)) >= 0) out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```

누가봐도 지저분하다 ㅎㅎ

여기서 결점은 `try 블록`과 `finally 블록` 안에서 모두 발생할 수 있다.

# try-with-resources 를 사용하자
`try-finally` 에서 발생하는 문제점을 보완하기 위해 `try-with-resources` 를 사용하자!

```java
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            return br.readLine();
    }
}
```

위 코드에 복수 자원을 처리하는 `try-with-resources` 를 적용하면

```java
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0) out.write(buf, 0, n);
    }
}
```

읽기 수월하고 문제 진단하기도 좋다.

## catch 절도 같이 사용하자
catch 절을 사용하여 다수의 예외를 처리할 수 있다.

위 코드를 살짝 수정해보자
```java
static String firstLineOfFile(String path, String defaultValue) {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } catch (IOException e) {
        return defaultVal;
    }
}
```

`catch` 절을 사용해서 예외처리를 해결할 수 있다.

## 정리
close 를 통해 회수해야 하는 자원을 다룰 때는 `try-finally` 말고, `try-with-resources` 를 사용하자. 보다 더 쉽고 정확하게 가독성 있는 코드를 만들 수 있다.