# [item7] 다 쓴 객체 참조를 해제하라

자바는 가비지 컬렉터를 갖춘 언어이지만 메모리 관리를 해야 한다.

## 메모리 누수가 일어나는 위치는 어디인가?

```java
package com.effectivejava;

import java.util.Arrays;
import java.util.EmptyStackException;

public class Stack {
	private Object[] elements;
	private int size = 0;
	private static final int DEFAULT_INTIAL_CAPACITY = 16;

	public Stack() {
		elements = new Object[DEFAULT_INTIAL_CAPACITY];
	}

	public void push(Object e) {
		ensureCapacity();
		elements[size++] = e;
	}

	/*
	 * 원소를 위한 공간을 적어도 하나 이상 확보한다. 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
	 */
	private void ensureCapacity() {
		if (elements.length == size) {
			elements = Arrays.copyOf(elements, 2 * size + 1);
		}

	}

	**public Object pop() {**
		if (size == 0)
			throw new EmptyStackException();
		return elements[--size];
	}

}
```

- 위 코드는 메모리 누수가 일어난다. 프로그램을 오래 실행하면 가비지 컬렉션 활동과 메모리 사용량이 늘어나 결국 성능이 저하된다.
- 이 코드에서 스택이 커졌다가 줄어들었을 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다. 이 스택이 객체들의 다 쓴 참조를 여전히 가지고 있기 때문이다.
- 여기서 다 쓴 참조는 elements 배열의 ‘활성영역’ 밖의 참조들이다. 활성영역은 인덱스가 size보다 작은 원소들로 구성된다.
- 따라서 해당 참조를 다 썼을 때 null 처리(참조 해제)해야 한다.

## 제대로 구현한 pop 메서드

```java
public Object pop() {
		if (size == 0)
			throw new EmptyStackException();
		Object result = elements[--size];
		elements[size]=null; //다 쓴 참조 해제
		return result;
	}
```

- 모든 객체를 null 처리 하는 것이 아닌 객체 참조를 null 처리하는 일은 예외적인 경우여야 한다.
- 다 쓴 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 유효 범위(scope) 밖으로 밀어내는 것이다.
    - 그 예로 지역변수를 선언할 때 해당 scope에서만 선언하도록 만들어야 한다. for문 내부에, 사용하는 변수는 그 외부에 선언을 해놓으면 GC가 자동으로 정리해주지 못한다.
    

### Stack  클래스가 메모리 누수에 취약한 이유는 **스택이 자기 메모리를 직접 관리하기 때문이다.**

- 이 스택은 (객체 자체가 아닌 객체 참조를 담는) elements 배열로 저장소 풀을 만들어 원소들을 관리한다. 배열의 활성 영역에 속한 원소들이 사용되고 비활성 영역은 쓰이지 않는다.
- 가비지 컬렉터는 비활성 영역에서 참조하는 객체도 똑같이 유효한 객체이다.
- **따라서 자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야 한다.**

### **객체 참조를 캐시에 넣고 메모리 해제를 해주지 않으면 메모리 누수가 일어날 수 있다.**

- 캐시 엔트리의 유효 기간을 정확히 정의하기 어렵기 때문에 시간이 지날수록 엔트리의 가치를 떨어뜨리는 방식을 사용한다.
    - 이 방식에서는 쓰지 않는 엔트리를 메모리에서 해제해야 한다.
        - [https://d2.naver.com/helloworld/1326256](https://d2.naver.com/helloworld/1326256)
    - ScheduledThreadPoolExecutor 같은 백그라운드 스레드를 활용한다.
    - 캐시에 새 엔트리를 추가할 때 부수 작업으로 수행한다. 예로 LinkedHashMap은 removeEldestEntry메서드를 써서 처리한다.
    
    ```java
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest){ 
    		return false;
    }
    ```
    
    - removeEldestEntry() 는 들어온 순서를 기억하고, eldest는 LinkedHashMap 에 들어온지 가장 오래된 값이다.
    
    ```java
    @Override
    protected boolean removeEldestEntry( Entry<Character,Character> eldest ) {
    		return size() == 6 ? true : false;
    }
    ```
    
    - 위와 같이 오버라이드 하여 사용하는데 LinkedHashMap 의 size 가 6이 되면, 가장 오래된 값을 지우고, 그 자리에 방금 들어온 값을 대체하는 코드이다.
    

### **리스너 혹은 콜백**

- 클라이언트가 콜백을 등록만 하고 해지하지 않는다면 콜백은 쌓인다.
- 이 때 콜백을 약한 참조로 저장하면 GC가 즉시 수거해간다.
- 예로 WeakHashMap에 키로 저장한다.