# ✨ 아이템 9: try-finally 보다는 try-with-resources 를 사용하라

자바 라이브러리는 close 메서드로 직접 호출해 자원을 닫아줘야하는 것이 많다. 자원 닫기는 클라이언트가 놓치기 쉽기 때문에 예측할 수 없는 성능 문제로 이어질 수 있다. 앞서 item08 에서 본 `finalizer/cleaner` 를 활용하여 안전망을 만들 수 있지만 믿을만한 구현체는 아니다.

자바7 에서 만들어진 try-with-resources 덕분에 쉬운 문법으로 해결되었다.

```java
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
        OutputStream out = new FileOutputStream(dst)) {
        
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ( (n = in.read(buf)) >= 0 ) {
            out.write(buf, 0, n);
        }
    }
}
```

1개 뿐만 아니라 복수의 자원도 짧으면서 직관적이게 활용하여 자원을 닫을 수 있다.
