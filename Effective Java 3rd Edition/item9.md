# [item9] try-finally보다는 try-with-resource를 사용하라

- 꼭 회수해야 하는 자원을 다룰 때는 try-finally 말고, try-with-resources문을 사용한다.
- 코드가 더 간결해지고 놓치는 예외정보 없이 사용할 수 있다.

## try-finally 문을 사용할 경우

```java
public class FirstError extends RuntimeException {
}
```

```java
public class SecondException extends RuntimeException {
}
```

```java
public class MyResource implements AutoCloseable {

    public void doSomething() throws FirstError {
        System.out.println("doing something");
        throw new FirstError();
    }

    @Override
    public void close() throws SecondException {
        System.out.println("clean my resource");
        throw new SecondException();
    }
}
```

```java
public class AppRunner {

	public static void main(String[] args) {
		MyResource myResource = new MyResource();
		
//		myResource.doSomething();
//		myResource.close();
		
		try {
			myResource.doSomething();
		}finally {
			myResource.close();
		}
	}
}
```

```java
doing something
Exception in thread "main" clean my resource
item9.SecondException
	at item9.MyResource.close(MyResource.java:13)
	at item9.AppRunner.main(AppRunner.java:12)
```

- 예외는 try문과 finally문 안에서 모두 발생하지만 두번째 에러인 close에 대한 예외 정보만 확인할 수 있다.
- 스택 추적 내역에 첫 번째 예외에 관한 정보는 남지 않게 된다.
- 처음 발생한 예외를 확인할 수 없기 때문에 디버깅이 어렵다.

```java
public class AppRunner {

	public static void main(String[] args) {
		MyResource myResource = new MyResource();

		try {
			myResource.doSomething();
			MyResource myResource2 = null;
			try {
				myResource2 = new MyResource();
				myResource2.doSomething();
			}finally {
				if(myResource2 != null) {
					myResource2.close();
				}
			}
		}finally {
			myResource.close();
		}
	}
}
```

- 자원이 둘 이상이면 코드가 복잡해진다.

## try-with-resources 문을 사용할 경우

```java
public class AppRunner2 {

	public static void main(String[] args) {
		try (MyResource myResource = new MyResource()){
			myResource.doSomething();
		}
	}
}
```

- 괄호() 안에 객체를 생성하는 문장을 넣으면 try 블럭을 벗어나는 순간 자동적으로 close()가 호출된다.
- 단, 클래스가 AutoCloseable 이라는 인터페이스를 구현한 것이어야만 한다.

```java
public interface AutoCloseable {
    void close() throws Exception;
}
```

- 두 문장 이상을 넣을 경우 ;으로 구분한다.

```java
doing something
clean my resource
Exception in thread "main" item9.FirstError
	at item9.MyResource.doSomething(MyResource.java:7)
	at item9.AppRunner2.main(AppRunner2.java:7)
	Suppressed: item9.SecondException
		at item9.MyResource.close(MyResource.java:13)
		at item9.AppRunner2.main(AppRunner2.java:8)
```

- 짧고 읽기 수월하다.
- readLine과 close 양 쪽에서 에러가 발생하면 close에서 발생한 예외는 숨겨지고 readLine에서 발생한 예외가 기록된다.
- 버려지는 것이 아니라 스택 추적 내역에 ‘숨겨졌다(suppressed)’는 꼬리표를 달고 출력된다.

- 자바 7에서 Throwable에 추가된 getSuppressed 메서드를 이용하면 프로그램 코드에서 가져올 수도 있다.

```java
void addSuppressed(Throwwable exception) 
Throwable[] getSuppressed() 
```

- addSuppressed() : suppressed된 예외를 추가
- getSuppressed() : suppressed된 예외(배열)를 반환