# [item13] clone 재정의는 주의해서 진행하라

- 클래스에서 clone을 재정의 하기 위해서는 해당 클래스에 Cloneable 인터페이스를 상속받아 구현해야 한다.
- 그런데 clone 메소드는 Cloneable 인터페이스가 아닌 Object에 선언되어 있다.
- Cloneable 인터페이스에는 아무것도 선언되어 있지 않은 빈 인터페이스이다.
- **그렇다면 Cloneable 인터페이스의 역할은 무엇일까?**

## Cloneable 인터페이스의 역할

- Cloneable 인터페이스는 상속받은 클래스가 복제해도 되는 클래스임을 명시하는 믹스인 인터페이스이다. (단지, ‘clone에 의해 복사할 수 있다’ 라는 표시로서 사용되고 있다.)
    - **믹스인**이란 클래스가 본인의 기능 이외에 추가로 구현할 수 있는 자료형으로, 어떤 선택적 기능을 제공한다는 사실을 선언하기 위해 쓰인다.
    - A라는 클래스가 있다면 Cloneable과 같은 mixin interface를 구현하게 하면 A클래스가 가지는 본연의 기능에 다른 기능을 덧붙이기 때문에 (mix-in) 믹스인이라고 한다.

**Cloneable은 정상적인 믹스인 인터페이스라고 할 수 없는데 clone() 메서드를 직접 제공하는 게 아닌  Object의 clone의 동작 방식을 결정하기 때문이다.**

- 자바 디자인 실패 사례 중 하나로써 따라하지 말아야 할 방식 중 하나라고 한다.

**Cloneable 인터페이스는 빈 인터페이스로 대상을 표시하는데만 사용되고 Cloneable 인터페이스에는 clone()이 없다. clone()은 Object 클래스에 존재하는데 다음과 같다.**

```java
protected native Object clone() throws CloneNotSupportedException;
```

**Cloneable을 상속한 클래스의 clone메소드를 호출하면 해당 클래스를 필드의 내용을 그대로 복사하여 반환한다. (shallow copy)**

- 만약, Cloneable을 상속받지 않고 clone 메소드를 호출하였다면 ‘CloneNotSupportedException’을 던진다.

## Object clone 메소드의 일반 규약

1. x.clone() != x는 참이다. 원본 객체와 복사 객체는 서로 다른 객체이다.
2. x.clone().getClass()==x.getClass()는 참이다. 하지만 반드시 만족해야 하는 것은 아니다.
3. x.clone().equals(x)는 참이지만 필수는 아니다. ⇒ 동치성을 보장하지 않는다.
4. x.clone().getClass()==x.getClass(), super.clone()을 호출해 얻은 객체를 clone 메소드가 반환한다면, 이 식은 참이다. 관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상의 반환 전에 수정해야 할 수도 있다.

- Object에 정의된 clone 메서드의 일반 규약도 허술하다.
    - 동치성을 보장하지 않는다면 복사한 의미가 있을까? 같은 객체라고 할 수 있을까? 명시된 일반 규약은 생성자로도 충분히 인스턴스를 반환할 수 있다.
    
- 하지만 만약, clone 메소드가 super.clone이 아닌 생성자를 호출해 얻은 인스턴스를 반환하더라도 컴파일시에 문제가 되지 않지만 해당 클래스의 하위 클래스에서 super.clone을 호출한다면 하위 클래스 타입 객체를 반환하지 않고 상위 클래스 타입 객체를 반환하여 잘못된 클래스의 객체가 만들어지는 불상사가 생길 수 있다.
    - 클래스 B가 클래스 A를 상속할 때, 하위 클래스인 B의 clone은 B 타입 객체를 반환해야 한다. 그런데 A의 clone이 자신의 생성자, 즉 new A(…)로 생성한 객체를 반환한다면 B의 clone도 A 타입 객체를 반환할 수 밖에 없다. 달리 말해 super.clone을 연쇄적으로 호출하도록 구현해두면 clone이 처음 호출된 상위 클래스의 객체가 만들어진다.
    

## clone 메소드 재정의

### 가변 객체를 참조하지 않는 클래스의 clone 재정의

```java
public PhoneNumber clone() {
		try {
			return (PhoneNumber) super.clone();
		} catch (CloneNotSupportedException e){
			throw new AssertionError(); //일어날 수 없는 일이다
		}
	}
```

- PhoneNumber의 클래스 선언에 Cloneable을 구현한다고 추가해야한다.
- Object의 clone메서드는 Object를 반환하지만 PhoneNumber의 clone 메서드는 PhoneNumber를 반환하게 한다.
- super.clone에서 얻은 객체를 반환하기 전에 PhoneNumer로 형변환하였다.

```java
 public class Foo implements Cloneable { //clone 메소드를 재정의 하기 위해서 Cloneable 인터페이스를 상속한다. 
    int num;

    public Foo() {
        System.out.println("---------------------");
        System.out.println("Foo constructor");
        System.out.println("---------------------");
    }

    public Foo(int num) {
        System.out.println("---------------------");
        System.out.println("Foo constructor");
        System.out.println("---------------------");
        this.num = num;
    }

    @Override
    public Foo clone() {
        try {
            System.out.println("---------------------");
            System.out.println("Foo Clone");
            System.out.println("---------------------");
            return (Foo) super.clone();
        } catch (CloneNotSupportedException e) { //Object의 clone메소드는 CloneNotSupportedException을 처리하도록 선언되어있다.
            throw new RuntimeException();
        }
    }
}
```

- Foo 클래스의 clone 메소드는 super.clone을 호출하여 Foo 클래스로 캐스팅하고 반환한다.
- 이 때 super.clone은 Object의 clone을 호출하는데 Object의 clone은 native 메소드로 Foo 클래스의 각 필드를 기준으로 생성자를 호출하지 않고 객체를 복사한다. (각 필드를 ‘=’ 를 이용해서 복사한다고 생각하면 쉽게 이해할 수 있다.)
- 이와 같이 **모든 필드가 기본 타입이거나 , 불변 객체를 참조한다면**, 위와 같이 super.clone 메서드만 호출하고 Foo 객체로 변환해서 반환하기만 해도 문제가 없다.
    - primitive 값을 복사할 때는 상수값을 가져와 복사하기 때문이다.

- 그런데 만약 클래스의 필드가 가변 객체를 참조하는 필드일 때 단순하게 super.clone만 반환하면 어떻게 될까?

### 가변 객체를 참조하는 클래스의 clone 재정의

```java
public class Stack implements Cloneable {
    private Foo[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        System.out.println("---------------");
        System.out.println("Stack constructor");
        this.elements = new Foo[DEFAULT_INITIAL_CAPACITY];
    }

    public Foo[] getElements() {
        return elements;
    }

    public int getSize() {
        return size;
    }

    public void push(Foo e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Foo pop() {
        if(size == 0) {
            throw new EmptyStackException();
        }
        Foo result = elements[--size];
        elements[size] = null;
        return result;
    }

    private void ensureCapacity() {
        if(elements.length == size) {
            elements = Arrays.copyOf(elements,2 * size + 1);
            System.out.println(elements.length);
        }
    }

    @Override
    public String toString() {
        StringBuilder stringBuilder = new StringBuilder();
        int i = 0;
        while (elements[i] != null) {
            stringBuilder.append(elements[i].num);
            stringBuilder.append(",");
            i++;
        }

        return stringBuilder.toString();
    }

    @Override
    public Stack clone() {
        try {
            return (Stack) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

- Stack 클래스는 Foo 배열을 필드로 가지고 있는 클래스이다. clone 메소드를 재정의하기 위해 Cloneable을 상속받았고 재정의한 clone 메소드는 super.clone을 반환한다.

- 위 코드에서 clone을 통해 반환된 Stack 인스턴스의 size 필드는 기본 타입이므로 값이 정상적으로 복사되어 서로 다른 값을 갖지만, 복사된 elements 배열은 원본 Stack 인스턴스의 elements 배열을 참조할 것이다. 원본이나 복제본 하나를 수정하면 다른 하나도 수정되어 불변식을 해친다.
    - (**shallow copy** - ‘주소값’ 복사, 참조하고 있는 실제 값은 같음 ↔ deep copy : ‘실제 값’을 새로운 메모리 공간에 복사)

![다운로드](https://user-images.githubusercontent.com/48662662/189886536-07d07edd-0bcf-4844-8799-4909803d48a7.png)
Stack 클래스의 clone 테스트

![img](https://user-images.githubusercontent.com/48662662/189886687-4867a2c7-d906-483c-b230-0e48f14ac43b.png)
테스트 결과

- 다음은 Stack 객체를 하나 생성하고 2개의 Foo 객체를 push한다. 그리고 clone으로 새로운 Stack 객체를 만든 뒤 cloneStack 객체에 1개의 객체를 push한다.
- 기존의 객체와 clone한 객체의 elements는 달라야 하는게 정상이라고 생각할 수 있다. 하지만 test 결과를 보자.
- 위와 같이 테스트는 성공한다. Stack 클래스의 int size 필드는 기본타입으로 값이 정상적으로 복사되어 서로 다른 값을 갖지만, 참조 필드인 element의 경우 같은 주소를 가지게 된다.

- clone 메서드는 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.

이를 가장 간단한 해결할 방법은 다음과 같다.

### 가변 상태를 참조하는 클래스용 clone 메소드

```java
@Override
    public Stack clone() {
        try {
						// super.clone()만 호출한다면
						// 복사는 되지만 가변 객체인 Elements가 같은 주소값을 갖는 얕은 복사가 된다
            Stack stack = (Stack) super.clone();
						// elements.clone 대신 배열의 clone을 사용할 수 있다.
            stack.elements = this.elements.clone();
            return stack;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
```

- elements 배열에 clone() 메서드를 이용해 따로 복사해주면 된다. Stack의 clone 메서드는 제대로 동작하려면 스택 내부 정보를 복사해야 하는데, 가장 쉬운 방법은 elements 배열의 clone을 재귀적으로 호출해주는 것이다. 다시 말하면, 내부변수들이 참조하고 있는 객체들을 Deep Copy를 해줘야 한다는 것이다.
- clone()은 원본 배열과는 별개의 주소값을 가진 새로운 배열을 만든다.

```java
int[] scores = { 1, 2, 3, 4, 5 };
int[] newScores = scores.clone();

System.out.println(Arrays.toString(scores)); // [1, 2, 3, 4, 5]
System.out.println(Arrays.toString(newScores)); // [1, 2, 3, 4, 5]

// 원본 배열 값 변경
scores[0] = 100;

System.out.println(Arrays.toString(scores)); // [100, 2, 3, 4, 5]
System.out.println(Arrays.toString(newScores)); // [1, 2, 3, 4, 5]

// 복사된 배열 값 변경
newScores[4] = 200;

System.out.println(Arrays.toString(scores)); // [100, 2, 3, 4, 5]
System.out.println(Arrays.toString(newScores)); // [1, 2, 3, 4, 200]
```

- scores.clone() 메서드를 이용해서 새로운 배열을 생성했다. newScores 변수에는 scores 배열과 다른 참조 주소 값을 갖는 배열을 가리키고 있다.

**→ 헷갈린 부분: Object의 clone은 원본 객체의 필드값과 동일한 값을 가지는 새로운 객체를 생성한다. 따라서 원본 객체와 새로운 객체는 서로 다른 주소값을 가진다. 하지만 필드값은 shallow copy 되기 때문에(’=’로 복사해주는 방식) primitive 타입은 상수값을 복사하기 때문에 상관없지만 가변 필드의 경우에는 같은 주소값을 가진 필드가 복사되는 것이다.**

- 하지만 이 방법도 단점이 존재하는데 기존 객체와 복사된 객체의 객체 배열이 서로 다른 주소를 가지고 있어도 배열의 원소가 가지고 있는 Foo라는 객체는 같은 객체이다.
- 이 경우 반복자를 이용해 기존 elements가 가지고 있는 원소들을 복사된 배열에 맞게 새로 생성하면 쉽게 해결할 수 있다.

- 그런데 만약 **elements가 final**이었다면 위의 방식을 사용할 수 없다. final 필드애는 새로운 값을 할당할 수 없기 때문이다.
- **Cloneable 아키텍처는 ‘가변 객체를 참조하는 필드는 final로 선언하라’ 라는 일반 용법과 충돌하게 된다.**
- 근본적인 문제

## clone 재정의시 주의사항

- clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.
- 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final 한정자를 제거해야 할 수도 있다.
- 재정의될 수 있는 메소드를 호출하지 않아야 한다.

## 복제 기능은 생성자와 팩터리를 이용하자

### 복사생성자

```java
public Yum(Yum yum){...};
```

- 복사 생성자: 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자

### 복사 팩터리

```java
public static Yum newInstance(Yum yum) {...};
```

- 복사 팩터리: 복사 생성자를 모방한 정적 팩터리이다.

- 복사 생성자와 복사 팩터리는 위의 clone의 모든 단점을 해결할 수 있다.
- 또한 해당 클래스가 구현한 ‘인터페이스’ 타입의 인스턴스를 인수로 받을 수 있다.
    - 메서드가 인터페이스 타입의 인수를 받을 수 있다는 것은 메서드 호출 시 해당 인터페이스를 구현한 클래스의 인스턴스를 인수로 받을 수 있다는 것이다.
- 또한 원본의 구현 타입에 얽매이지 않고 복제본의 타입을 직접 선택할 수 있다.
    - HashSet 객체 s를 treeSet 타입으로 복제할 수 있다.

- 단, 배열만은 clone메서드 방식이 가장 깔끔하기 때문에 예외로 둔다.

+) 참고

### 믹스인

- **믹스인**이란 클래스가 본인의 기능 이외에 추가하여 구현할 수 있는 자료형으로, 어떤 선택적 기능을 제공한다는 사실을 선언하기 위해 쓰인다.
- 다른 클래스의 부모클래스가 되지 않으면서 다른 클래스에서 사용할 수 있는 메서드를 포함하는 클래스이다.
- 포함(has-a)로 설명된다.
- 코드 재사용성을 높여준다.
- 자바코드에서는 다중 상속의 제한이 없는 인터페이스로 구현하기 용이하다.

```java
public interface Singer {
  AudioClip sing(Song s);
}

public interface Songwriter {
  Song compose(int chartPosition);
}

public interface SingerSongwriter extends Singer, Songwriter {
  AudioClip strum();
  void actSensitive();
}
```

- 예를 들어 현실에서는 가수 겸 작곡가인 경우도 있는데 이런 경우를 인터페이스로 구현할 경우 쉽게 구현 가능하다.

### native 메소드

native 메소드: 자바에서 특화된 메소드로써, 다른 언어로 구현되어 있으며 native 라는 지시자(예약어)를 가지는 메소드를 말함

### Reference

[https://minhyeokism.tistory.com/27](https://minhyeokism.tistory.com/27)

[https://javabom.tistory.com/15](https://javabom.tistory.com/15)

[https://velog.io/@tomato2532/Object.clone-얕은-복사-깊은-복사-복사-생성자](https://velog.io/@tomato2532/Object.clone-%EC%96%95%EC%9D%80-%EB%B3%B5%EC%82%AC-%EA%B9%8A%EC%9D%80-%EB%B3%B5%EC%82%AC-%EB%B3%B5%EC%82%AC-%EC%83%9D%EC%84%B1%EC%9E%90)

[https://ss-o.tistory.com/136](https://ss-o.tistory.com/136)

[https://doublesprogramming.tistory.com/157](https://doublesprogramming.tistory.com/157)
