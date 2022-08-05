# [item4] 인스턴스화를 막으려거든 private 생성자를 사용하라

* 인스턴스 상태와 무관하게, 항상 특정한 기능을 제공해주기 위한 static멤버들로 구성된 클래스는 인스턴스화가 불필요하다. 
	* ex) `java.awt.event.WindowAdapter` , `java.base.java.lang.Math` 등등..
* 인스턴스화가 필요없는 static멤버로 구성된 클래스는 인스턴스화를 막는 것이 효율적이다.
	* 성능면에서, 인스턴스화를 허용하는 것은 인스턴스화에 따른 오버헤드가 존재
	* 이것을 막는 방법은 크게 2가지가 존재



## 방법 1. 추상 클래스(abstract class)

```java
public abstract class MyAbstractClass {
	private static final AUTHOR = "joon";
   	public static String f(String name){return "hello" + name;}
    public MyAbstractClass(){}
    // 아 여기안에 추상 메서드가 1개 이상 존재하는구나! 이걸로 직접 인스턴스화를 막아야겠다
    //public abstract abstF();
};
```

* 일반적으로 추상 클래스는 1개 이상의 추상 메소드가 존재해야하지만, **간혹 완성된 클래스에서도 인스턴스화를 막기 위해 추상 클래스로 만드는 경우가 존재한다.** (ex. `WindowAdapter`)

* 하지만, abstract 클래스를 상속받은 **구체 클래스에서의 인스턴스화를 막지는 못한다.**

	* ex)

		```java
		public class MyConcreteClass extends MyAbstractClass {
		    MyConcreteClass(){
		    	// super() 가 자동으로 추가됨
		        ...
		    }
		}
		```

	* 구체 클래스로 인스턴스가 생성될 경우, 자신의 부모 클래스의 생성자를 호출하기 때문에 `MyAbstractClass`의 생성자가 호출된다. 이 경우, 구체 클래스의 인스턴스가 만들어질때 `MyAbstractClass`의 인스턴스화가 이루어진다.

* 인스턴스화를 완전하게 막는 방법은 아니다.

* 또한, 추상 클래스의 목적인 `자식 클래스에서 상속하여 오버라이딩 하여 사용` 으로 오해의 소지가 존재한다.

## 방법2. private 생성자

```java
public class MyClass {
    private static final AUTHOR = "joon";
    public static String f(String name){return "hello" + name;}
    private MyClass(){throws new IllegalAccessException("인스턴스화 불가능");} // !!!
}
```

* `new()` 연산자로 인스턴스를 생성할 경우, 내부적으로 생성자 함수를 통해 인스턴스의 값을 초기화한다.
* 이 과정에서 `private` 으로 생성자가 선언될 경우, 클래스 외부에서 호출이 불가능하므로 해당 클래스 외부에서 인스턴스화가 불가능하다.
* 하지만, 동일 클래스 내부에서 해당 생성자를 호출할 경우에는 인스턴스화를 막을 수 없으므로 실수로 사용하는 것을 방지하기 위해 호출될 경우 예외를 던져준다.

