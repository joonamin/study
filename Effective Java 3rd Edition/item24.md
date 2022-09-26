# item24. 멤버 클래스는 되도록 static으로 만들라

* 멤버 클래스: top-level 클래스 내부에 정의된 클래스(중첩 클래스의 일종)
* 일반적으로, 자신을 감싼 클래스 외부에서 사용되어진다면 해당 클래스는 top-level 클래스로 정의되어야한다.

```java

public class A {
    private final int number = 0;
    public class B {
        // B는 non-static member class
        // B는 A와 매우 밀접한 관계를 지니고, B 단독으로는 존재하지 못할 경우 이런식으로 표현
        private final double dNumber = 0.0;
    }
}
```



## 멤버 클래스 종류

1. 정적(static) 멤버 클래스 
2. 비정적 멤버 클래스
3. 익명 클래스
4. 지역 클래스



**정적 멤버 클래스는 자신을 감싸는 클래스의 private 멤버에도 접근이 가능하다.** 그 외에는 일반 클래스와 동일하다.

또한, 자신을 감싸는 클래스의 접근 규칙에 따라서 내부 필드를 참조할 수 있다. (ex. private 선언시, 자신과 자신을 감싸는 클래스에서만 접근 가능)

* 정적 멤버 클래스는 흔히 자신을 감싸는 클래스와 함께 쓰일 경우에만 유용한 public 도우미 클래스로 쓰인다.
* 정적 멤버 클래스는 인스턴스 레벨의 클래스가 아니다.
	* <u>바깥 인스턴스로의 숨은 외부 참조를 갖지 않는다.</u>
* ==멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자.==
	* 숨은 외부 참조에 의한 성능 저하 X
	* ❗ 가비지 컬렉트시에 바깥 클래스의 인스턴스를 수거하지 못하는 경우 발생
* 바깥 클래스가 표현하는 객체의 일부분(component)로 표현할 때 많이 사용한다.

---

**비정적 멤버 클래스는  인스턴스 레벨의 클래스이다.** 만약, 비정적 멤버 클래스의 멤버를 참조하고자 하는 경우, 해당 클래스를 감싸는 외부 클래스의 인스턴스화가 필요하다.

* 외부 클래스의 인스턴스를 통해서 비정적 멤버 클래스의 인스턴스화가 가능하다.

	* ex) `(클래스명).this` 
	* <u>외부 클래스의 인스턴스와 비정적 멤버 클래스는 암묵적으로 연결된다.</u>
	* 그로 인해, 비정적 멤버 클래스의 인스턴스화를 한다면 관계 정보에 대한 추가 정보가 인스턴스 내부에 존재하게된다. (overhead)

* 비정적 멤버 클래스의 인스턴스는 바깥 클래스가 인스턴스화 될 때 확립되며 더 이상 수정할 수 없다.

* 개념상, 중첩 클래스의 인스턴스가 **바깥 인스턴스에 대하여 의존적일** 경우 비정적 멤버 클래스를 사용한다.

* 주로, <u>어댑터</u>를 정의하기 위하여 자주 쓰인다.

	* 어떤 클래스의 인스턴스를 감싸 다른 클래스의 인스턴스로 보이게 하기 위함

	```java
	// 비정적 멤버 클래스를 이용한 어댑터 구현
	public interface Map<K, V> extends ... {
	    // ... 생략 ...
	    public Set<K> keySet() {
	        // 모든 Key값에 대하여 inner class의 도움을 받아서 Set형성
	        MySet mySet = new MySet(keys);
	        return mySet;
	    }
	    private class MySet extends Set<K> {
	        private final List<K> keyList = new ArrayList();
	        // ... 생략 ... 
	    }
	}
	```

	

---

**익명 클래스는 멤버 클래스가 아니며 선언과 동시에 인스턴스가 만들어진다.** 이러한 특징으로 인해, 코드의 어떠한 부분에서도 선언이 가능하다. 그리고 non-static context에서만 사용될 때만, 바깥 클래스의 인스턴스를 참조할 수 있다. 

```java
public class Example {
	private string name = "joon!";
    interface MyInterface {
        void doSomething();
    }
	public void minjunJoa() {
        MyInterface myClass = new MyInterface() {
            @Override
            public void doSomething() {
                System.out.println(name);
            }
        };
    }
    public static void main(String[] args) {
		// static context
        MyInterface myClass = new MyInterface() {
            @Override
            public void doSomething() {
                System.out.println("doSomething");
            }
        };
        myClass.doSomething();
    }
}
```

* 특징1. non-static context 내부에서 사용될 때만, 바깥 클래스의 인스턴스를 참조할 수 있다.
* 특징2. static context내부에서 사용되더라도 상수변수 이외에 정적 멤버는 가질 수 없다. (익명 클래스는 인스턴스 레벨이기 때문)
* 특징3. 람다와 마찬가지로 코드 길이가 길수록 가독성이 현저히 떨어진다.
* 특징4. 익명 클래스를 사용하는 클라이언트는 그 익명 클래스가 상위 타입에서 상속한 멤버 외에는 호출할 수 없다.

---

**지역 클래스는 지역 변수와 동일한 context에서 선언하여 사용할 수 있다.** (가장 드물긴 함)

```java
// Belows are driver Program
public class Main {
    public class MyClass {
        private int number;
        public MyClass() {this.number = 10;}
    };
    // 이런 느낌! :) 
    // int a;
    // a = 10;
    // 
	Myclass myClass;
    public Main(MyClass ref) {this.myClass = ref;}
    public Main() {this.myClass = new MyClass();}
    
}
```

---

공개 API에 중첩 클래스를 사용할 경우, 정적 멤버 클래스냐 비정적 멤버 클래스냐의 따른 중요성이 매우 커진다.

* 멤버 클래스 또한 공개API가 된다. 향후 release에서 static/non-static으로의 변경이 어려울 수 있다(하위 호환성)



