### item17. 변경 가능성을 최소화하라

자바 플랫폼 라이브러리에는 다양한 불변 클래스가 있다. 

eg. String, 기본 타입의 박싱된 클래스들, BigInteger, BigDecimal



---



### 불변 클래스를 만들기 위한 다섯가지 규칙

클래스를 불변으로 만들기 위해서는 다음 다섯 가지 규칙을 따르면 된다.



1. 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.

    

2. 클래스를 확장할 수 없도록 한다. 
    - 하위 클래스에서 객체의 상태를 변하게 만드는 상황을 막아준다.

      

3. 모든 필드를 final로 선언한다. 

    

4. 모든 필드를 private으로 선언한다. 
    - 기술적으로는 기본 타입 필드가 불변 객체를 참조하는 필드를 public final로만 선언해도 불변 객체가 되지만, 이렇게 하면 다음 릴리스에서 내부 표현을 바꾸지 못하므로 권하지는 않는다. (만약, 클라이언트가 public final 필드를 쓰고 있다면 api 내부 코드를 마음대로 바꾸지 못한다.)

      

5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다. 
- 가변 객체를 참조하는 필드는 절대 클라이언트가 제공한 객체 참조를 가리키게 해서는 안 되며, 접근자 메서드가 그 필드를 그대로 반환해서도 안 된다.

- 생성자, 접근자, readObject 메서드 모두에서 방어적 복사를 수행하라.
    - `readObject` 메서드는 실질적으로 또 다른 public 생성자이다.
    - `ObjectInputStream`의 `readObject()` 메서드
      
        ![캡처.PNG](C:\Users\USER\Desktop\캡처.PNG.png)
        
    - 다음 코드는 파일에 쓰여진 객체를 읽는 코드이다. 파일 읽기 스트림(`FileInputStream`)을 열고 `ObjectInputStream`을 통해서 읽는다. `ObjectInputStream`에 `FileInputStream`을 전달하고 `readObject`를 이용해서 객체를 얻어온다. 이 때 `Object` 객체로 반환하므로 적절히 형변환을 해야 한다.
    
    ```java
    Account ruser=null;
    		
    FileInputStream fis=new FileInputStream("user.acc");
    ObjectInputStream ois=new ObjectInputStream(fis);
    ruser=(Account)ois.readObject();
    
    ois.close();
    ```
    



**`readObject`의 방어적 복사**

```java
public final class Period {
  private final Date start;
  private final Date end;
  
  public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    if (this.start.compareTo(this.end) > 0)
      throw new IllegalArgumentException(start + "after" + end); 
  }
  
  public Date start() {
    return new Date(start.getTime()); 
  }

  public Date end() {
    return new Date(end.getTime());
  }
}
```

- 불변인 날짜 범위 클래스를 만드는데 가변인 `Date` 필드를 이용했다.
- 따라서 불변식을 지키고 불변을 유지하기 위해 생성자와 접근자에서 `Date` 객체를 방어적으로 복사해야 한다.

- 이 클래스를 직렬화 하기로 결정했다고 가정하자.
    - 직렬화: 자바 시스템 내에서 사용하는 객체 또는 데이터를 자바시스템 외에서도 사용할 수 있도록 Byte 형태로 데이터를 변환하는 기술
- 간단하게 이 클래스 선언에 `implements Serializable`을 추가하면 이 클래스의 불변식을 보장하지 못하게 된다. `readObject` 메서드가 실질적으로 또 다른 public 생성자이기 때문이다.
- 따라서 `readObject` 메서드에서도 인수가 유효한지 검사해야 하고 필요하다면 매개변수를 방어적으로 복사해야 한다.

- 단순히 `Period` 클래스 선언에 `implements Serializable`만 추가했다고 가정해보자. 그러면 종료 시각이 시작 시각보다 앞서는 `Period` 인스턴스를 만들 수 있다. 이는 허용되지 않는 `Period` 인스턴스이다.

```java
public class BogusPeriod {
	//진짜 Period 인스턴스에서는 만들어질 수 없는 바이트 스트림
	private static final byte[] serializedForm = new byte[] { (byte) 0xac,
			(byte) 0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06, 0x50, 0x65, 0x72,
			0x69, 0x6f, 0x64, 0x40, 0x7e, (byte) 0xf8, 0x2b, 0x4f, 0x46,
			(byte) 0xc0, (byte) 0xf4, 0x02, 0x00, 0x02, 0x4c, 0x00, 0x03, 0x65,
			0x6e, 0x64, 0x74, 0x00, 0x10, 0x4c, 0x6a, 0x61, 0x76, 0x61, 0x2f,
			0x75, 0x74, 0x69, 0x6c, 0x2f, 0x44, 0x61, 0x74, 0x65, 0x3b, 0x4c,
			0x00, 0x05, 0x73, 0x74, 0x61, 0x72, 0x74, 0x71, 0x00, 0x7e, 0x00,
			0x01, 0x78, 0x70, 0x73, 0x72, 0x00, 0x0e, 0x6a, 0x61, 0x76, 0x61,
			0x2e, 0x75, 0x74, 0x69, 0x6c, 0x2e, 0x44, 0x61, 0x74, 0x65, 0x68,
			0x6a, (byte) 0x81, 0x01, 0x4b, 0x59, 0x74, 0x19, 0x03, 0x00, 0x00,
			0x78, 0x70, 0x77, 0x08, 0x00, 0x00, 0x00, 0x66, (byte) 0xdf, 0x6e,
			0x1e, 0x00, 0x78, 0x73, 0x71, 0x00, 0x7e, 0x00, 0x03, 0x77, 0x08,
			0x00, 0x00, 0x00, (byte) 0xd5, 0x17, 0x69, 0x22, 0x00, 0x78 };

	public static void main(String[] args) {
		Period p = (Period) deserialize(serializedForm);
		System.out.println(p);
	}

	// 주어진 직렬화 형태(바이트 스트림)로부터 객체를 만들어 반환한다.
	private static Object deserialize(byte[] sf) {
		try {
			InputStream is = new ByteArrayInputStream(sf);
			ObjectInputStream ois = new ObjectInputStream(is);
			return ois.readObject();
		} catch (Exception e) {
			throw new IllegalArgumentException(e);
		}
	}
}
```

- 이 코드를 실행하면 **Fri Jan 01 12:00:00 PST 1999 - Sun Jan 01 12:00:00 PST 1984** 를 출력한다.

  

**유효성 검사를 수행하는 `readObject` 메서드**

```java
private void readObject(objectInputStream s) throws IOException, ClassNotFoundException {
		s.defaultReadObject( );
		// 가변 객체들은 방어 복사한다.
		start = new date(start.getTime( ));
		end = new Date(end.getTime( ));

		// 불변식을 만족하는지 검사한다.
		if (start.compartTo(end) > 0) throw new InvalidObjectException(start + "after" + end);
}
```



**불변 복소수 클래스**

```java
public final class Complex {
    private final double re;
    private final double im;

    private Complex(double re, double im){
        this.re = re;
        this.im = im;
    }

    public static Complex valueOf(double re, double im){
        return new Complex(re, im);
    }

    public double realPart() {
        return re;
    }

    public double imaginaryPart(){
        return im;
    }

    public Complex plus(Complex c){
        return new Complex(re+c.re, im+c.im);
    }

    public Complex minus(Complex c){
        return new Complex(re-c.re, im-c.im);
    }

    public Complex times(Complex c){
        return new Complex(re*c.re - im*c.im, re*c.im + im*c.re);
    }

    public Complex divdedBy(Complex c){
        double tmp = c.re*c.re+c.im*c.im;
        return new Complex((re*c.re + im*c.im)/tmp,(im*c.re-re*c.im)/tmp);
    }

    @Override public boolean equals(Object o){
        if(o == this) return true;
        if(!(o instanceof Complex)) return false;
        Complex c = (Complex) o;

        return Double.compare(c.re, re) == 0 && Double.compare(c.im, im) == 0;
    }

    @Override
    public int hashCode(){
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override
    public String toString(){
        return "("+re+" + "+im+"i)";
    }
}
```

- 각 속성을 반환하는 접근자만 제공할 뿐 변경자는 없다.

  

1. 사칙연산 메서드(`plus`, `minus`, `times`, `dividedBy`)들은 인스턴스 자신은 수정하지 않고 새로운 `Complex` 인스턴스를 만들어 반환한다. → 함수형 프로그래밍
    - **함수형 프로그래밍**: 피연산자에 함수를 적용해 그 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 패턴
    
    - 함수형 프로그래밍을 한다면 코드에서 불변이 되는 영역이 높아진다.
    
      ↔ 절차적 혹은 명령형 프로그래밍: 메서드에서 피연산자인 자신을 수정해 자신의 상태가 변하게 된다.
    
      
    
2. 메서드 이름으로 add 같은 동사 대신 plus 같은 전치사를 사용하였다. 
    - 해당 메서드가 객체의 값을 변경하지 않는다는 사실을 강조하려는 의도이다.
    
      
    
    ---



#### 불변클래스(객체)의 장점

1. 불변 객체는 단순하다.
- 모든 생성자가 클래스 불변식을 보장한다면 그 클래스를 사용하는 프로그래머가 다른 노력을 들이지 않아도 불변이다.

  
2. 불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요가 없다.

- 불변 객체에 대해서는 그 어떤 스레드도 다른 스레드에 영향을 줄 수 없으므로 불변 객체는 안심하고 공유할 수 있다.

  

3. 불변 클래스라면 한번 만든 인스턴스를 최대한 재활용 하는 것을 권한다.

- 자주 쓰이는 값들을 상수(`public static final`)로 제공하기

```java
public static final Complex ZERO = new Complex(0, 0);
 public static final Complex ONE  = new Complex(1, 0);
 public static final Complex I    = new Complex(0, 1);
```

- 여기에 불변 클래스는 자주 사용되는 인스턴스를 캐싱하여 같은 인스턴스를 중복 생성하지 않게 해주는 정적 팩터리를 제공할 수 있다.
    - eg. 박싱된 기본 타입 클래스와 `BigInteger`
      
        ![2.PNG](C:\Users\USER\Desktop\2.PNG.png)
        
    - 이렇게 정적 팩터리를 사용하면 여러 클라이언트가 인스턴스를 공유하여 메모리 사용량과 가비지 컬렉션 비용이 줄어든다.
    
        
    
4. 불변 객체는 자유롭게 공유할 수 있으므로 방어적 복사도 필요 없다.

- 아무리 복사해봐야 원본과 똑같으니 복사 자체가 의미가 없다.

- 따라서 불변 클래스는 `clone` 메서드나 복사 생성자를 제공하지 않는 것이 좋다.

  

5. 불변 객체는 자유롭게 공유할 수 있고 불변 객체끼리는 내부 데이터를 공유할 수 있다.

- eg. `BigInteger` 클래스
    - 값의 부호(sign)와 크기(magnitude)를 따로 표현한다.
    
    - 부호에는 `int` 변수를, 크기(절대값)에 는 `int` 배열을 사용한다.
    
      
    
- `negate` 메서드는 크기가 같고 부호만 반대인 새로운 `BigInteger`를 생성한다.
    
    - 이 때 배열은 비록 가변이지만 복사하지 않고 원본 인스턴스와 공유해도 된다.
    
    - 결과적으로 새로운 `BigInteger` 인스턴스는 원본 인스턴스가 가리키는 내부 배열을 그대로 가리킨다.
    
      

6. 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 좋다.

- 이러한 객체는 구조가 아무리 복잡해도 불변식을 유지하기 수월하다.

- eg. 불변 객체는 맵의 키와 집합의 원소로 쓰기에 좋다.

  

7. 불변 객체는 그 자체로 실패 원자성을 제공한다.

- 상태가 변하지 않으므로 불일치 상태에 빠질 가능성이 없다.

  > 실패 원자성: 메서드에서 예외가 발생한 후에도 그 객체는 여전히 메서드 호출 전과 똑같은 유효한 상태여야 한다.

  

  ---

  

#### 단점

1. 값이 다르면 반드시 독립된 객체로 만들어야 한다.
- 값의 가짓수가 많다면 모두 독립된 객체로 만들어야 하므로 큰 비용이 든다.

  

eg. 1,000,000 비트짜리 `BigInteger`에서 비트 하나를 바꿔야 한다고 가정하자.

```java

BigInteger moby = ...;
moby = moby.flipBit(0);

```

- `fitBit` 메서드는 원본과 단 하나의 비트만 다른 1,000,000 크기의 새로운 `BigInteger` 인스턴스를 생성한다.

```java
BitSet moby = ...;
moby.flip(0);
```

- `BitSet`도 `BigInteger`처럼 임의 길이의 비트 순열을 표현하지만, `BigInteger`와는 달리 ‘가변’이다.
- `BitSet` 클래스는 원하는 비트 하나만 상수 시간 안에 바꿔주는 메서드를 제공한다.



원하는 객체를 완성하기까지의 단계가 많고, 그 중간 단계에서 만들어진 객체들이 모두 버려진다면 성능 문제가 발생한다. 해결방법은 다음과 같다.

1. 흔히 쓰일 다단계 연산을 예측하여 기본 기능으로 제공한다. 
- eg. `BigInteger`는 모듈러 지수 같은 다단계 연산 속도를 높여주는 가변 동반 클래스(companion class)를 `package-private`으로 두고 있다.

  

**BigInteger들의 가변 동반 클래스의 동작 방식**

- `BitSieve`, `SignededMutableBigInteger`, `MutableBigInteger` 가변 동반 클래스들은 모두 `package-private` 접근제한이다.
- 클라이언트는 직접적으로 저 클래스들을 조작할 방법이 없고 `BigInteger`클래스의 메서드들이 사용한다.
- `BigInteger` 의 메서드를 사용하면, 그 내부에서 가변 동반 클래스들을 사용하고 그 결과를 다시 `BigInteger`로 캐스팅해서 반환하는 방식.

```java
public BigInteger gcd(BigInteger val) {
    if (val.signum == 0)
        return this.abs();
    else if (this.signum == 0)
        return val.abs();

    MutableBigInteger a = new MutableBigInteger(this);
    MutableBigInteger b = new MutableBigInteger(val);

    MutableBigInteger result = a.hybridGCD(b);

    return result.toBigInteger(1);
}

/**
 * Convert this MutableBigInteger to a BigInteger object.
 */
BigInteger toBigInteger(int sign) {    // <-- 접근제한자 package-private 에 주목
    if (intLen == 0 || sign == 0)
        return BigInteger.ZERO;
    return new BigInteger(getMagnitudeArray(), sign);
    // 이마저도 불변을 지키기 위해서 방어적 복사를 진행함
}
```



2. 클라이언트들이 원하는 복잡한 연산들을 예측할 수 없다면 클래스를 public으로 제공해야 한다.

   

**불변 클래스를 만드는 또 다른 설계 방법**

- 모든 생성자를 `private` 혹은 `package-private`로 만들고 public 정적 팩터리를 제공한다.

```java
public class Complex{
	private final double re;
    private final double im;
    
    private Complex(double re, double im){
    	this.re=re;
        this.im=im;
	}
    
    public static Complex valueOf(double re, double im){
    	return new Complex(re, im);
	}
	...
}
```



**Reference**

[https://donghyeon.dev/이펙티브자바/2021/11/09/readObject-메서드는-방어적으로-작성하자/](https://donghyeon.dev/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%EC%9E%90%EB%B0%94/2021/11/09/readObject-%EB%A9%94%EC%84%9C%EB%93%9C%EB%8A%94-%EB%B0%A9%EC%96%B4%EC%A0%81%EC%9C%BC%EB%A1%9C-%EC%9E%91%EC%84%B1%ED%95%98%EC%9E%90/)

[https://reakwon.tistory.com/160](https://reakwon.tistory.com/160)

[https://github.com/JunHyeok96/effective-java/issues/15](https://github.com/JunHyeok96/effective-java/issues/15)