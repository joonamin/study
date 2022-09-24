

### item 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

상속을 고려한 설계와 문서화란 무엇일까?



**1. 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지(자기사용) 문서로 남겨야 한다.** 

- 메서드에서 클래스 자신의 또 다른 메서드를 호출할 수 있는데 호출되는 메서드가 재정의 가능 메서드라면 이것을 문서에 남겨야 한다.

- 어떤 순서로 호출하는지, 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지 써야 한다.

  

> Implementation Requirements
>
> 메서드의 내부 동작을 설명하는 곳
>
> 메서드 주석에 `@implSpec` 태그를 붙여주면 자바독 도구가 생성해준다.



다음은 java.util.AbstractCollection에서 발췌한 예이다.

**public boolean remove (Object o)**

> 주어진 원소가 이 컬렉션 안에 있다면 그 인스턴스를 하나 제거한다(선택적 동작). 
>
> 더 정확하게 말하면, 이 컬렉션 안에 '`Object.equals(o, 2)`가 참인 원소' e가 하나 이상 있다면 그 중 하나를 제거한다. 주어진 원소가 컬렉션 안에 있었다면(즉 호출 결과가 이 컬렉션이 변경됐다면) `true`를 반환한다.
>
> Implementation Requirements: 이 메서드는 컬렉션을 순회하며 주어진 원소를 찾도록 구현되었다. 주어진 원소를 찾으면 반복자의 `remove` 메서드를 사용해 컬렉션에서 제거한다. 이 컬렉션이 주어진 객체를 갖고 있으나, 이 컬렉션의 `iterator` 메서드가 반환한 반복자가 `remove` 메서드를 구현하지 않았다면 `UnsupportedOperationException`을 던지니 주의하자.

- `iterator` 메서드를 재정의하면 `remove` 메서드의 동작에 영향을 줌을 알 수 있다.

- `iterator` 메서드로 얻은 반복자의 동작이 `remove` 메서드의 동작에 주는 영향도 정확히 설명했다.

  

---



**2. 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 protected 메서드 형태로 공개해야 할 수도 있다.**

**protected void removeRange(int fromIndex, int toIndex)**

> `fromIndex`(포함)부터 `toIndex`(미포함)까지의 모든 원소를 이 리스트에서 제거한다.toIndex 이후의 원소들은 앞으로 (index만큼씩) 당겨진다. 이 호출로 리스트는 `toIndex - fromIndex`만큼 짧아진다.(`toIndex == fromIndex`라면 아무런 효과가 없다.)
> 
> 이 리스트 훅은 이 리스트의 부분리스트에 정의된 `clear` 연산이 이 메서드를 호출한다. 리스트 구현의 내부 구조를 활용하도록 이 메서드를 재정의하면 이 리스트와 부분리스트의 `clear` 연산 성능을 크게 개선할 수 있다.
> 
> Implementation Requirements: 이 메서드는 `fromIndex`에서 시작하는 리스트 반복자를 얻어 모든 원소를 제거할 때까지 `ListIterator.next`와 `ListIterator.remove`를 반복호출하도록 구현되었다.
> 
> 주의: `ListIterator.remove`가 선형 시간이 걸리면 이 구현의 성능은 제곱에 비례한다.
> 
> Parameters: 
> 
> `fromIndex`: 제거할 첫 원소의 인덱스
> 
> `toIndex`: 제거할 마지막 원소의 다음 인덱스

- List 구현체의 사용자는 `removeRange` 메서드를 알 필요가 없지만 하위 클래스에서 부분리스트의  `clear`메서드를 고성능으로 만들기 쉽게 하기 위해서 이 메서드를 제공한다.

- `rangeRange` 메서드가 없다면 하위클래스에서 `clear` 메서드를 호출하면 (제거할 원소 수의) 제곱에 비례해 성능이 느려지거나 부분리스트의 메커니즘을 새로 구현해야 했을 것이다.

  

---



**상속용 클래스를 설계할 때 어떤 메서드를 protected로 노출해야 할지는 어떻게 결정할까?**



- protected 메서드 하나하나가 내부 구현에 해당하므로 그 수는 가능한 적어야 한다.

- 상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 유일하다.
    - 이러한 검증에는 하위 클래스 3개 정도가 적당하고 이 중 하나 이상은 제 3자가 작성해봐야 한다.
    
      

---



1. 상속용으로 설계한 클래스는 배포 전에 반드시 하위 클래스를 만들어 검증해야 한다.

   

1. 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안된다.

   아래는 문제가 되는 예이다.

   

```java
public class Super {
    // 잘못된 예 - 생성자가 재정의 가능 메서드를 호출한다.
    public Super() {
        overrideMe();
    }

    public void overrideMe(){
func();
    }
private void func(){
//...
}
}
```

```java
public class Sub extends Super {
    // 초기화되지 않은 final 필드.
    // 생성자에서 초기화한다.
    private final Instant instant;

    public Sub() {
        instant = Instant.now();
    }

    // 재정의 가능 메서드.
    // 상위 클래스의 생성자가 호출한다.
    @Override
    public void overrideMe() {
        System.out.println(instant);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
		
		// 결과 : null
				2022-09-13T10:27:34.940726200Z
    }
}
```

- `instant`를 두 번 출력하는 것이 아닌 첫 번째는 `null`이 출력된다.

  

3. `clone`과 `readObject` 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안된다.

   - `readObject`의 경우 하위 클래스의 상태가 미처 역직렬화되기 전에 재정의한 메서드부터 호출하게 된다.
   - `clone`의 경우 하위클래스의 clone 메서드가 복제본의 상태를 (올바른 상태로) 수정하기 전에 재정의한 메서드를 호출한다.

   

4. Serializable 을 구현한 상속용 클래스가 `readResolve`나 `writeReplace` 메서드를 갖는다면 이 메서드들은 `private`이 아닌 `protected`로 선언해야 한다. 

   - `private`으로 선언한다면 하위 클래스에서 무시되기 때문이다.

   - 상속을 허용하기 위해 내부 구현을 클래스 API로 공개하는 예 중 하나이다.

     

5. 상속용으로 설계하지 않은 클래스는 상속을 금지한다.

   - 클래스를 final로 선언한다.

   - 모든 생성자를 private나 package-private으로 선언하고 public 정적 팩터리를 만들어준다.

     

6. 상속용으로 설계하지 않은 클래스를 상속을 허용해야 한다면 클래스 내부에서는 재정의 가능 메서드를 사용하지 않게 만들고 이 사실을 문서로 남겨야 한다.



**클래스의 동작을 유지하면서 재정의 가능 메서드를 사용하는 코드를 제거할 수 있는 기계적인 방법**

1) 각각의 재정의 가능 메서드는 자신의 본문 코드를 private ‘도우미 메서드’로 옮기고 이 도우미 메서드를 호출하도록 수정한다.

2) 재정의 가능 메서드를 호출하는 다른 코드들도 모두 이 도우미 메서드를 직접 호출하도록 수정한다.