# try-finally 보다는 try-with-resources 를 사용하라
</br>

자바 라이브러리에는 close 메서드를 호출해 직접 닫아줘야 하는 자원이 많다.  <br>
전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 try-finally가 쓰였다. 예ㅚ가 발생하거나 메서드에서 반환되는 경우를 포함해서 말이다. <br>

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

## 📌 try-with-resources는 자원을 회수하는 최석책이다.
```java
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            return br.readLine();
    }
}
```
<br>

## 📌 try-with-resources는 복수의 자원을 처리하는 짧고 매혹적이다.
```java
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0) out.write(buf, 0, n);
    }
}
```

여기서 catch 절도 함께 쓰면 좋다.

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
