# try-finally보다는 try-with-resources를 사용하라

자바 라이브러리에는 InputStream, OutputStream, java.sql.Connection 등 close메서드를 호출해 직접 닫아줘야 하는 자원이 많다.

전통적으로 이러한 자원을 닫기 위한 수단으로 try-finally가 사용되었다.

## try-finally - 자원을 회수하는 최선의 방책이 아니다

```java
static String firstLineOfFile(String path) throws IOException{
	BufferedReader br = new BufferedReader(new FileReader(path));
	try {
		return br.readLine();
	} finally {
		br.close();
	}
}
```
자원이 둘 이상이면 try-finally 방식은 코드가 매우 복잡해진다.
```java
static void copy(String src, String dst) throws IOException{
	InputStream in = new FileInputStream(src);
	try {
		OutputStream out = new FileOutputStream(dst);
		try {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while((n = in.read(buf)) >= 0){
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

- 예외는 try 블록과 finally 블록 모두에서 발생할 수 있다. 
- 기기에 물리적인 문제가 생긴다면 firstLineOfFile 메서드 안의 readLine 메서드가 예외를 던지고, 같은 이유로 close 메서드도 실패할 것이다.
- 이런 상황이라면 두 번째 예외가 첫 번째 예외를 덮어버린다.
- 그러면 스택 추적 내역에 첫 번째 예외에 관한 정보는 남지 않게 되어, 실제 시스템에서의 디버깅을 몹시 어렵게 한다.

## try-with-resources - 자원을 회수하는 최선책
위의 문제들을 해결하기 위해 자바7부터 생겼다.   
이 구조를 사용하려면 해당 자원이 AutoCloseable 인터페이스를 구현해야한다.   
AutoCloseable은 단순히 void를 반환하는 close 메서드 하나만 정의한 인터페이스이다. (자바 라이브러리들이 이미 구현,확장해두었다.)
```java
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```
```java
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
    	 OutputStream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
            	out.write(buf, 0, n);
    }
}
```
- 위의 try-finally로 작성했던 코드를 try-with-resources로 바꾼 코드이다.
- 코드가 더 짧고 문제를 진단하기도 훨씬 좋다.
- readLine과 close 호출 양쪽에서 예외가 발생하면, close에서 발생한 예외는 숨겨지고 readLine에서 발생한 예외가 기록된다.
- 숨겨진 예외들도 그냥 버러지지는 않고, 스택 추적 내역에 '숨겨졌다(supressed)'는 꼬리표를 달고 출력된다.
- try-with-resources도 catch 절을 쓸 수 있다. catch 절을 사용하면 try 문을 중첩시키지 않아도 다수의 예외를 처리할 수 있다.

## 결론
꼭 회수해야 할 자원을 다룰 때는 try-with-resources를 사용하자.
코드는 더 짧고 분명해지고, 예외 정보도 훨씬 유용하다.