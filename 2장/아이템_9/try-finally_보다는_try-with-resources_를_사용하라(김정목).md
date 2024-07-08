# try-finally 보다는 try-with-resources 를 사용하라
</br>

자바 라이브러리에는 close 메서드를 호출해 직접 닫아줘야 하는 자원이 많다.  <br>
전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 try-finally가 쓰였다. 예외가 발생하거나 메서드에서 반환되는 경우를 포함해서 말이다. <br>

## 1️⃣ try-finally는 더 이상 자원을 회수하는 최선의 방책이 아니다. 
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

## 2️⃣ try-finally는 자원이 둘 이상이면 너무 지저분하다.
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

## 📌 try-with-resources는 자원을 회수하는 최선책이다.
```java
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            return br.readLine();
    }
}
```
<br>

## 📌 try-with-resources는 복수의 자원을 처리하는 짧고 매혹적이다.
try-with-resources는 짧고 읽기 수월할 뿐 아니라 문제를 진단하기도 훨씬 좋다. 
```java
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0) out.write(buf, 0, n);
    }
}
```

보통 try-finally 처럼 try-with-resources에서도 catch절을 쓸 수 있다. catch절을 넣으면 try문을 더 중첩하지 않고도 다수의 예외를 처리할 수 있다.

<br>

## 📌 try-with-resources + catch
```java
static String firstLineOfFile(String path, String defaultValue) {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } catch (IOException e) {
        return defaultVal;
    }
}
```

## ⭐️ 핵심 정리
꼭 회수해야 하는 자원을 다룰 때는 try-finally 말고, try-with-resources를 사용하는 것이 좋다. 예외는 없다. 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다. try-finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도, try-with-resources로는 정확하고 쉽게 자원을 회수할 수 있다.
