[toc]

# item16. public클래스에서는 public 필드가 아닌 접근자 메소드를 사용하라

```java
public class Point {
    public int x;
    public int y;
}
```

이 방식은 객체지향프로그래밍 관점에서의 '캡슐화'에 대한 이점을 얻지 못한다. (구현과 API를 분리하지 않았다)

이 경우, 사용자는 해당 클래스 외부에서 public 필드인 x, y에 직접 접근이 가능해진다.



## **왜 문제일까?**

* 해당 API를 사용하는 클라이언트는 public 필드에 직접 접근이 가능하므로, API **내부 구현에 대해 의존성을 갖게 된다**.

* 클라이언트는 특정 기능을 사용하기 위하여 API를 호출하는데 API 자체가 아닌 내부 구현에 의존할 경우, 추후에 이 구현 방법 자체를 수정하기 매우 어려워진다. 

	> ''자주 변경되는 구체(Concret)클래스에 의존하지 마라" - 로버트 C. 마틴 



```java
public class Point {
    private int x;
    private int y;
    
    public int getX() {return x;}
    public int getY() {return y;}
    public void setX(int x){this.x = x;}
    public void setY(int y){this.y = y;}
}
```

위 방식은 public 접근자를 이용하여 필드에 간접 접근한다. 이 경우, 클래스 내부 표현 방식(= API 내부 구현)에 대한 유연성을 얻을 수 있다.

이와 마찬가지로, 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 **클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.**

> 그 외의 접근자를 통한 필드 접근의 이점들
>
> - **Encapsulation of behavior** associated with getting or setting the property - this allows additional functionality (like <u>validation</u>) to be added more easily later.
>
> 	
>
> - **Hiding the internal representation of the property** while exposing a property using an alternative representation.
>
> 
>
> - **Insulating your public interface from change** - allowing the <u>public interface to remain constant while the implementation changes without affecting existing consumers.</u>
>
> 
>
> - Providing a debugging interception point for when a property changes at runtime - debugging when and where a property changed to a particular value can be quite difficult without this in some languages.
>
> 
>
> - Improved **interoperability with libraries that are designed to operate against property getter/setters** - Mocking, Serialization, and WPF come to mind.
>
> 
>
> - ~~Allowing **inheritors to change the semantics of how the property behaves** and is exposed by overriding the getter/setter methods.~~
>
> 
>
> - Allowing the **getter/setter to be passed around as lambda expressions** rather than values.
>
> 
>
> - **Getters and setters can allow different access levels** - for example the get may be public, but the set could be protected.
>
> 출처 - https://stackoverflow.com/questions/1568091/why-use-getters-and-setters-accessors

## **데이터 필드의 노출**

위에서 살펴본 접근자 메소드의 이점들은 API의 내부 표현을 클라이언트에게 공개하지 않는 것에 대한 이점들이다.

그러한 맥락에서 **package-private 클래스 또는 private inner class라면 데이터 필드를 노출한다 해도 문제가 없다.**

<u>API를 사용하는 클라이언트가 API내부 구현에 대한 의존성을 갖지 못하기 때문이다.</u> 

(비록, package-private 클래스 또는 private inner class에 의존하는 상위 클래스들은 해당 클래스의 내부 구현에 의존하긴 하지만, 이것 또한 API내부 구현의 일종이다.)



Q. 그럼에도 불구하고 데이터 필드를 외부에 노출한다면 발생할 수 있는 피해는?

A. 동기화 이슈도 고려해야한다. 가변 객체에 여러 모듈들이 접근하여 값을 수정할 경우, 예상치 못한 동작을 할 수 있기 때문이다.



**Q. 그렇다면 public 클래스의 필드가 불변이라면 해당 필드를 노출했을 때 단점이 모두 없어질까?**

A. 아니다! API를 변경하지 않고서는 표현 방식을 바꿀 수 없고 필드를 읽을 때 부수작업(ex. validation)등을 수행할 수 없다는 문제점은 여전하다. 

