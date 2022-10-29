

### item 21.인터페이스는  구현하는 쪽을 생각해 설계하라



자바 8부터 디폴트 메서드를  사용할 수 있게 되었다.

디폴트 메서드를 선언하면, 그 인터페이스를 구현한 후 디폴트 메서드를 재정의하지 않은 모든 클래스에서 디폴트 구현이 쓰이게 된다.

하지만 자바 7까지는 모든 클래스가 "현재의 인터페이스에 새로운 메서드가 추가될 일은 영원히 없다"고 가정하고 작성되어 디폴트 메소드로 인해 문제점이 발생하기도 한다.



**생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하는 것은 어렵다.**



---



> 자바 8의 `Collection` 인터페이스에 추가된 `removeIf` 메서드

```java
	//true를 반환하면 remove 호출해 원소 제거
	default boolean removeIf(Predicate<? super E> filter) { 
		Objects.requireNonNull(filter); 
		boolean result = false; 
		for(Iterator<E> it = iterator(); it.hasNext(); ) { 
			if(filter.test(it.next())) { 
				it.remove(); 
				result = true; 
			} 
		} 
		return result; 
	}
```

- 이 메서드는 주어진 불리언 함수(predicat; 프레디키트)가 `true`를 반환하는 모든 원소를 제거한다.
- 디폴트 구현은 반복자를 이용해 순회하면서 각 원소를 인수로 넣어 프레디키트를 호출하고, 프레디키트가 `true`를 반환하면 반복자의 `remove` 메서드를 호출해 그 원소를 제거한다.



#### 아파치의 org.apach.commons.collections4.collection.SynchronizedCollection

**removeIf 메서드**는 `org.apache.commons.collections4.collection.SynchronizedCollection`와 호환되지 않는다.

- 아파치 커먼즈 라이브러리의 이 클래스는 `java.util`의 `Collections.synchronized Collection` 정적 팩터리 메서드가 반환하는 클래스와 비슷하다.

  - 차이점은 아파치 버전은 (컬렉션 대신) 클라이언트가 제공한 객체로 락을 거는 기능을 추가로 제공한다.

    &rarr; 모든 메서드에서 주어진 락 객체로 동기화한 후 내부 컬렉션 객체에 기능을 위임하는 래퍼 클래스이다.

- 아파치의 `SynchronizedCollection` 클래스는 `removeIf` 메서드를 재정의하지 않고 있다.

  - 이 클래스를 자바 8과 함께 사용하여 `removeIf`의 디폴트 구현을 물려받게 된다면  모든 메서드 호출을 알아서 동기화해주지 못한다. `removeIf`의 구현은 동기화에 관해 알지 못하므로 락 객체를 사용할 수 없다.

  - 따라서 `SynchronizedCollection` 인스턴스를 여러 스레드가 공유하는 환경에서 한 스레드가 `removeIf`를 호출하면 `ConcurrentModificationException`이 발생하거나 다른 예기치 못한 결과로 이어질 수 있다.

    > `ConcurrentModificationException` 은 대부분 반복문 내부에서 Collection 객체에 remove 메소드 호출 시 발생한다. 반복문 내부에서 타깃 컬렉션의 길이가 변하게 되어 처음 참조한 변수와 같지 않기 때문에 발생하게 되는 것.

    

---



#### 자바 플랫폼 라이브러리에서의 예방책

자바 플랫폼 라이브러리에서 위와 같은 문제를 예방하기 위해

구현한 인터페이스의 디폴트 메서드를 재정의하고, 다른 메서드에서는 디폴트 메서드를 호출하기 전에 필요한 작업을 수행하도록 한다.

e.g. `Collections.synchronizedCollection`이 반환하는 `package-private` 클래스들은 `removeIf`를 재정의하고, 이를 호출하는 다른 메서드들은 디폴트 구현을 호출하기 전에 동기화하도록 한다.



하지만 자바 플랫폼에 속하지 않은 기존 컬렉션 구현체들은 아직 수정되지 못한 것들이 많다.

**디폴트 메서드는 컴파일에 성공하더라도 기존 구현체에 런타임 오류를 일으킬 수 있다.**



---



#### 인터페이스를 설계할 때는 세심한 주의를 기울여야 한다.

- 기존 인터페이스일 경우

  - 기존 인터페이스에 디폴트 메서드로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니면 피해야 한다.

  - 또한, 추가하려는 디폴트 메서드가 기존 구현체들과 충돌하지 않을지 고려해야 한다.

    

- 새로운 인터페이스일 경우
  - 반면, 새로운 인터페이스를 만드는 경우라면 표준적인 메서드 구현을 제공하는 데 유용한 수단이며, 해당 인터페이스를 더 쉽게 구현해 활용할 수 있게 만들어준다.
  - 새로운 인터페이스는 릴리스 전에 반드시 테스트를 거쳐야 한다. 인터페이스는 서로 다른 방식으로 최소한 세가지는 구현해봐야 하며 각 인터페이스의 인스턴스를 다양한 작업에 활용하는 클라이언트도 여러개 만들어 봐야 한다. 



