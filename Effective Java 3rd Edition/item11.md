# [item11]equals를 재정의하려거든 hsahCode도 재정의하라

- **equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.**

## hashCode 일반 규약

- equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
- **equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.**
- equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.
    - 해시 충돌이 발생했을때 체이닝, 개방 주소법을 사용하는데 체이닝 과정에서 연결리스트형태로 충돌을 처리하므로 O(N)의 시간이 걸린다.
        - [https://j3sung.tistory.com/759](https://j3sung.tistory.com/759)

두번째 조항을 다시 말하면, 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.

하지만 hashCode 재정의를 잘못했을 때 이 조항이 문제가 된다. 

### equals만 재정의할 경우

```java
package item11;

public class Car {
	private final String name;

	public Car(String name) {
		this.name = name;
	}

//	@Override
//	public int hashCode() {
//		final int prime = 31;
//		int result = 1;
//		result = prime * result + ((name == null) ? 0 : name.hashCode());
//		return result;
//	}

	@Override
	public boolean equals(Object obj) {
		if (this == obj)
			return true;
		if (obj == null)
			return false;
		if (getClass() != obj.getClass())
			return false;
		Car other = (Car) obj;
		if (name == null) {
			if (other.name != null)
				return false;
		} else if (!name.equals(other.name))
			return false;
		return true;
	}

}
```

```java
public class CarTest {

	public static void main(String[] args) {
		Car car1 = new Car("foo");
		Car car2 = new Car("foo");

		// true 출력
		System.out.println(car1.equals(car2));
	}
}
```

- equals를 재정의했기 때문에 Car 객체의 name이 같은 car1, car2 객체는 논리적으로 같은 객체로 판단된다.

```java
public class CarTest {

	public static void main(String[] args) {
		List<Car> cars = new ArrayList<>();
	    cars.add(new Car("foo"));
	    cars.add(new Car("foo"));

	    System.out.println(cars.size()); // 2
	}
}
```

- List는 중복 객체를 허용하므로 출력 결과는 2가 된다.

```java
public class CarTest {

	public static void main(String[] args) {
		Set<Car> cars = new HashSet<>();
		cars.add(new Car("foo"));
		cars.add(new Car("foo"));

		System.out.println(cars.size()); // 2
	}
}
```

- List를 Set으로 바꿔 중복값을 허용하지 않도록 바꾸고 출력하면 1이 아닌 2가 나온다.
- **hashCode를 equals와 함께 재정의하지 않으면 hash값을 사용하는 Collection(HashSet, HashMap, HashTable)을 사용할 때 문제가 발생한다.**

## hashCode 동작과정

![2020-07-29-equals-and-hashcode.png](%5Bitem11%5Dequals%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8C%E1%85%A2%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%8B%E1%85%B4%E1%84%92%E1%85%A1%E1%84%85%E1%85%A7%E1%84%80%E1%85%A5%E1%84%83%E1%85%B3%E1%86%AB%20hsahCode%E1%84%83%E1%85%A9%20%E1%84%8C%E1%85%A2%E1%84%8C%E1%85%A5%2091f4af2e2cd64d5b9d1230fc4341e544/2020-07-29-equals-and-hashcode.png)

- hash 값을 사용하는 Collection(HashMap, HashSet, HashTable)은 객체가 논리적으로 같은지 비교할 때 위와 같은 과정을 거친다.
- 위 동작과정을 보면 hashCode 메서드의 리턴 값이 우선 일치하고 equals 메서드의 리턴 값이 true여야 논리적으로 같은 객체라고 판단한다.
- 앞 예제에서 HashSet에 Car 객체를 추가할 때도 위와 같은 과정으로 중복 여부를 판단하고 HashSet에 추가된다. 하지만 Car 클래스에는 hashCode 메서드가 재정의 되어있지 않아서 Object 클래스의 hashCode 메서드가 사용되었다.
- Object 클래스의 hashCode 메서드는 객체의 고유한 주소 값을 int 값으로 변환하기 때문에 객체마다 다른 값을 리턴한다. 두 개의 Car 객체는 equals로 비교도 하기 전에 서로 다른 hashCode 메서드의 리턴 값으로 인해 다른 객체로 판단된 것이다.

```java
Map<PhoneNumber, String> m = new HashMap<>();
		m.put(new PhoneNumber(707,867,5309), "제니"); 
		m.get(new PhoneNumber(707,867,5309)); // null 반환
```

- hashCode를 재정의하지 않았기 때문에 두 객체는 논리적 동치이지만 서로 다른 해시코드를 반환한다.

## hashCode 재정의

- 해시 함수는 서로 다른 인스턴스에 다른 해시코드를 반환해야 한다. → hashCode의 세 번째 규약이 요구하는 속성
- 이상적인 해시 함수는 주어진 (서로 다른) 인스턴스들을 32비트 정수 범위에 균일하게 분배해야 한다.

```java
	@Override
	public int hashCode() {
		final int prime = 31;
		int result = 1;
		result = prime * result + ((name == null) ? 0 : name.hashCode());
		return result;
	}
```

## 좋은 hashCode를 작성하는 방법

1. int 변수 result를 선언한 후 값 c로 초기화한다. 이 때 c는 해당 객체의 equals 비교에 사용되는 첫번째 필드(첫번째 핵심필드)를 단계 2.a방식으로 계산한 해시코드이다. 
    - 핵심필드 : equals 비교에 사용되는 필드
    - 예제에서 c는 1이다.
2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.
    1. 해당 객체의 해시코드 c를 계산한다.
        1. 기본 타입 필드라면, Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본 타입의 박싱 클래스이다.
        2. 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 hashCode를 재귀적으로 호출한다. 계산이 더 복합해질 것 같으면, 이 필드의 표준형(canonial representation)을 만들어 그 표준형의 hashCode를 호출한다. 필드의 값이 null이면 0을 사용한다(다른 상수도 괜찮지만 전통적으로 0을 사용한다.)
        3. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, 단계 2.b 방식으로 갱신한다. 배열에 핵심 원소가 하나도 없다면 단순히 상수(0을 추천한다)를 사용한다. 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.
    2. 단계 2.a에서 계산한 해시코드 c로 result를 갱신한다. 코드로는 다음과 같다.
        - result = 31 * result + c;
        - 31*result는 필드를 곱하는 순서에 따라 result 값이 달라지게 한다. → 클래스에 비슷한 필드가 여러 개일 때 해시 효과를 크게 높여준다.
            - String의 hashCode를 곱셈 없이 구현한다면 모든 아나그램(구성하는 철자가 같고 그 순서만 다른 문자열)의 해시코드가 같아진다.
        - 곱할 숫자가 31인 이유는 31이 홀수이면서 소수이기 때문이다.
            - 짝수이고 오버플로가 발생한다면 정보를 읽게 된다. 2를 곱하는 것은 시프트 연산과 같은 결과를 내기 때문이다.
3. result를 반환한다.

> **구현 참고 사항**
> 
- 파생 필드는 해시코드 계산에서 제외해도 된다.
    - 파생 필드: 다른 필드로부터 계산해낼 수 있는 필드는 모두 무시해도 된다.
- equals 비교에 사용되지 않은 필드는 ‘반드시’  제외해야 한다. 제외하지 않으면 두번째 hashCode 규약을 위반할 수 있다.

- 해시 충돌이 더욱 적은 방법을 꼭 써야 한다면 구아바의 com.google.common.hash.Hashing을 참고하면 된다.

> **클래스가 불변이고 해시코드를 계산하는 비용이 크다면 캐싱하는 방식을 고려해야 한다.**
> 
- 이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해두어야 한다.
- 해시의 키로 사용되지 않는 경우라면 hashCode가 처음 불릴 때 계산하는 지연 초기화(lazy initialization) 전략을 사용해도 좋다.
- 필드 초기화를 할 때는 스레드 안정성까지 고려해야 한다.

```java
package item11;

public class PhoneNumberTest {

	private int hashCode; // 자동으로 0으로 초기화된다.

	@Override
	public int hashCode() {
		int result = hashCode;
		if (result == 0) {
			result = Short.hashCode(areaCode);
			result = 31 * result + Short.hashCode(prefix);
			result = 31 * result + Short.hashCode(lineNum);
			hashCode = result;
		}
		return result;
	}

}
```

> **성능을 높이기 위해 해시코드를 계산할 때 핵심 필드를 생략해서는 안된다.**
> 
- 속도는 빨라지지만 해시 품질이 나빠져 해시테이블의 성능을 떨어트린다.
- 어떤 필드는 특정 영역에 몰린 인스턴스들의 해시코드를 넓은 범위로 고르게 퍼트려주는 효과가 있을 수도 있는데 이런 필드를 생략한다면 해당 영역의 수 많은 인스턴스가 단 몇 개의 해시코드로 집중되어 해시테이블의 속도가 선형으로 느려질 수 있다.

## 한줄로 구현한 hashCode 함수

- Object  클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메서드인 **hash**를 제공한다.
- 하지만 앞서 구현했던 hashCode보다는 속도가 더 느리다.
    - 입력 인수를 담기 위한 배열이 만들어지고, 입력 중 기본 타입이 있다면 박싱과 언박싱도 거쳐야 하기 때문이다.

```java
@Override 
public int hashCode() {
		return Objects.hash(lineNum, prefix, areaCode);
	}
```

> hashCode 가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자. 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산방식을 바꿀 수도 있다.
> 

## REFERENCE

[https://tecoble.techcourse.co.kr/post/2020-07-29-equals-and-hashCode/](https://tecoble.techcourse.co.kr/post/2020-07-29-equals-and-hashCode/)