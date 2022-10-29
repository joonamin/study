### item23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

태그 달린 클래스란 <u>2개 이상의 특정한 개념을 필드를 통해서 표현하는 클래스</u>를 말한다.

다음은 태그 달린 클래스이다.

```java
public class Figure {
    enum Shape {RECTANGLE, CIRCLE}

    //태그 필드 - 현재 모양을 나타낸다.
    private Shape shape;

    // 다음 필드들은 모양이 사각형일 때만 사용.
    private double length;
    private double width;

    // 다음 필드들은 모양이 원일 때만 싸용.
    private double radius;

    //원용 생성자
    public Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    //사각형용 생성자
    public Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    private double area() {
        switch (shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

---

#### 태그 달린 클래스의 단점

> 태그 달린 클래스는 장황하고 오류를 내기 쉽고 비효율적이다.

1. 열거 타입 선언, 태그 필드, switch 문 등 쓸데 없는 코드가 많다.

2. 여러 구현이 한 클래스에 혼합돼 있어서 가독성이 나쁘다.

3. 다른 의미를 위한 코드가 함께 존재하므로 메모리를 많이 사용한다.

4. 필드들을 final로 선언하려면 해당 의미에 쓰이지 않는 필드들까지 생성자에서 초기화해야 한다.

   &rarr; 쓰지 않는 필드를 초기화하는 불필요한 코드가 늘어난다.

   &rarr; 생성자가 엉뚱한 필드를 초기화해도 컴파일 타임이 아닌 런타임에서 문제가 생긴다.

5. 또 다른 의미를 추가하려면 코드를 수정해야 한다.

   새로운 의미를 추가할 때마다 모든 switch 문을 찾아 새 의미를 처리하는 코드를 추가해야 하는데 하나라도 빠트리면 런타임 문제가 생긴다.

6. 인스턴스의 타입만으로는 현재 나타내는 의미를 알 수 없다.

---

#### 태그 달린 클래스를 클래스 계층구조로 바꾸자

1. 계층 구조의 루트가 될 추상 클래스를 정의한다.

2. 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언한다.

3. 태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가한다.

4. 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 루트 클래스로 올린다.

5. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다.

```java
public class Figure { // 1) Figure 클래스를 계층 구조의 루트로 만든다.
    enum Shape {RECTANGLE, CIRCLE}

    //태그 필드 - 현재 모양을 나타낸다.
    private Shape shape;

    // 다음 필드들은 모양이 사각형일 때만 사용.
    private double length;
    private double width;

    // 다음 필드들은 모양이 원일 때만 싸용.
    private double radius;

    //원용 생성자
    public Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    //사각형용 생성자
    public Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    private double area() { //2) 루트 클래스의 추상 메서드로 선언
        switch (shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
// 5) 루트 클래스를 확장한 구체 클래스를 의미 별로 정의한다.
// Figure를 확장한 원(Circle) 클래스와 사각형(Rectangle)클래스를 만든다.
```

다음은 태그 달린 클래스를 클래스 계층 구조로 변환한 코드이다.

```java
abstract class Figure {
	//공통적으로 동작하는 메서드
    abstract double area();
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() { //하위 클래스에서 재 정의
        return Math.PI * (radius * radius);
    }
}

class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    double area() {
        return length * width;
    }
}
```

#### 태그 달린 클래스를 클래스 계층 구조로 변환했을 경우 장점

1. 간결하고 명확하며 쓸데 없는 코드가 없다.

2. 필드들은 모두 final이다.

3. 각 클래스의 생성자가 모든 필드를 남김없이 초기화하고 추상 메서드를 모두 구현했는지 컴파일러가 확인해준다.

4. 빼먹은 case문으로 인한 런타임오류가 없다.

5. 루트 클래스의 코드를 건드리지 않고도 독립적으로 계층구조를 확장하고 사용할 수 있다.

6. 타입이 의미별로 존재하여 변수의 의미를 명시하거나 제한할 수 있고, 특정 의미만 매개변수로 받을 수 있다.

7. 타입 사이의 자연스러운 계층 관계를 반영할 수 있어서 유연성은 물론 컴파일타임 타입 검사 능력을 높여준다.

   - 정사각형을 지원하도록 태그 달린 클래스를 수정해보자.

   - 클래스 계층 구조에서라면 정사각형이 사각형의 특별한 형태임을 간단하게 반영할 수 있다.

   - ```java
     class Square extends Rectangle {
         Square(double side) {
             super(side, side);
         }
     }
     ```
