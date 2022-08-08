# item2. 생성자에 매개변수가 많다면 빌더를 고려하라

생성자를 어떻게 설계하는 것이 효율적일까?

## 상황1. 해당 클래스의 대부분의 멤버변수는 기본값을 가진다.

* customize할 값이 없는 경우 멤버변수는 기본값을 가진다.

* ex) 대부분의 언어에서 parameter의 default value를 지원한다.

	* ```cpp
		// c++
		int add(int op1, int op2 = 0) {return op1 + op2;}  // 선언부
		add(10); // 10 
		add(10, 20); // 30
		```

	* ```python
		# python
		def add(op1, op2 = 0): #선언부
		    return op1 + op2
		add(op1 = 10) # 10 (keyword argument)
		add(op1 = 10, op2 = 20) # 30
		```

	* ```swift
		// swift
		func add(op1: Int, op2: Int = 0) {
			return op1 + op2
		}
		add(op1: 10); // 10
		add(op1: 10, op2: 20); // 30
		```

		parameter의 default value 설정이 가능할 경우, default value가 설정되어있지 않은 parameter(= *필수 값*)에 값을 넘기는 방식으로 작성이 가능하다. 

		이 경우에는, overload해야하는 생성자의 수가 현저히 줄어들게된다.

* 반면에 Java는?

	* parameter의 default value를 지정하는 별도의 syntax가 존재하지 않는다.

		* tricky한 방법을 사용한다

	* **[방법1]** **점층적 생성자 패턴(telescoping constructor pattern)**

		* 필수 값에 해당하는 인자들에 대한 생성자를 선언하고, optional한 인자들에 대한 생성자를 모두 overload

		* ```java
			public class Citizen {
			    private String country; // 기본 값: Korea 
			    private int age;	// 필수 값
			    private String name; // 기본 값: joonamin
			    
			    public Citizen(String country, int age, String name){
			        this.country = country;
			        this.age = age;
			        this.name = name;
			    }
			    public Citizen(int age, String name) {
			        this("Korea", age, name);
			    }
			    public Citizen(int age) {
			        this(age, "joonamin");
			    }
			}
			```

		* 필수값만 생성자에 넘겨준다면, 해당 생성자에서 더 좁은 범위의 생성자를 연쇄적으로 호출하면서 기본 값을 채워주는 느낌 (`telescope`)

	* **[방법1] 장점**

		* 안전성
		* 생성자를 호출하는 로직 내부에서 연쇄적으로 값을 설정하는 것이기 때문에, 역할 분리 측면의 `생성`  단계에서 특정 값을 가지는 인스턴스를 만든다는 점에서 적절하다.

	* **[방법1] 단점**

		* 유연성

			* 필수 값이 유동적으로 바뀌는 상황에 대처하기 어렵다.
				* 최악의 경우, 모든 생성자의 정의를 재설정해야한다.

		* 가독성

			* 점층적으로 생성자를 호출하는 것이기 때문에, parameter의 수가 많을 수록 가독성이 떨어진다.
			* 생성자에서 사용되는 parameter의 수가 많아질 수록, 복잡도가 증가한다.

		* 편의성

			* 동일한 타입의 매개변수가 연속적으로 parameter에 존재한다면, client의 부주의로 예기치못한 오류가 발생할 수 있다.

			

	* **[방법2] 자바 빈즈 패턴(JavaBeans Pattern)**

		* 매개변수가 없는 생성자로 인스턴스를 만든 이 후, **setter계열의 메서드**를 통해 내부 멤버를 초기화해주는 방법

		* ```java
			public class Citizen {
			    private String country; // 기본 값: Korea 
			    private int age;	// 필수 값
			    private String name; // 기본 값: joonamin
			    
			    public Citizen(){...}
			    public void setCountry(String country){this.country=country;}
			    public void setAge(int age){this.age = age;}
			    public void setName(String name){this.name = name;}
			}
			/// Belows are driver program ... 
			Citizen citizen = new Citizen();
			citizen.setCountry("Korea");
			citizen.setAge(10);
			citizen.setName("joonamin");
			```

	* **[방법2] 장점**
		* 인스턴스를 만드는 과정이 직관적이다.
		* 유연성이 높다
		* 편의성이 높다.
			* client는 더 이상 parameter의 순서에 얽매이지 않아도 된다.
	* **[방법2] 단점**
		* `12-15`줄까지의 명령어들이 atomic하지 않다.
			* 중간에 context switch가 일어날 경우, 예상치 못한 결과를 낳을 수 있다.
			* Consistency(일관성) 문제!!
		* Immutable class로 만들 수 없다.
			* 초기화를 위해 setter계열의 함수를 사용하기 때문

	

## 빌더(Builder) 패턴

* `점층적 생성자 패턴`의 장점(안전성) + `자바 빈즈 패턴`의 장점(가독성)을 결합한 생성 패턴

* ```java
	// 일반적인 빌더 패턴
	public class Citizen {
	    private String country = "Korea"; // 기본 값: Korea 
	    private int age;	// 필수 값
	    private String name = "joonamin"; // 기본 값: joonamin
	    
	    public Citizen(Builder builder) {
	        this.country = builder.country;
	        this.age = builder.age;
	        this.name = builder.name;
	    }
	
		public static class Builder {
	        private String country;
	        private int age;
	        private String name;
	        
	        public Builder(int age) {
	           this.age = age;
	        }
	
	        public Builder country(String value) {
	            this.country = value;
	            return this;
	        }
	        public Builder age(int value) {
	            this.age = value;
	            return this;
	        }
	        public Builder name(String value){
	            this.name = value;
	            return this;
	        }
	        public Citizen build() {
	            return new Citizen(this);
	        }
	    }
	}
	// Belows are driver program
	Citizen citizen = new Citezen.Builder(100)
	    					.country("America")
	   						.name("joon")
	    					.build();
	
	```

	* 이 방식은 `named parameter` 방식을 모방한 것! 

* ```java
	// 계층적으로 설계된 클래스와 잘 어울리는 빌더 패턴
	public abstract class Pizza {
	    
	    public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE}
	    final Set<Topping> toppings;
	    
	    // <T extends Builder<T>> := Builder의 subtype만 argument로 올 수 있다. (재귀적 타입 한정)
	    // 그 결과로서, 해당 클래스를 extends한 클래스는 안전하게 addTopping, Build(), self()를 호출가능
	    abstract static class Builder<T extends Builder<T>> {
	        EnumSet<Topping> toppings = EnumSet.noneof(Topping.class);
	        public T addTopping(Topping topping) {
	            toppings.add(Objects.requireNonNull(topping));
	            return self();
	        }
	        abstract Pizza Build();
	        
	        protected abstract T self();
	    }
	    
	    Pizza(Builder<?> builder) {
	        toppings = builder.toppings.clone();
	    }
	}
	
	public class NyPizza extends Pizza {
	    
	    public enum Size {SMALL, MEDIUM, LARGE}
	    private final Size size;
	    
	   	public static class Builder extends Pizza.Builder<Builder> {
	        private final Size size;
	        public Builder(Size size) {
	            this.size = Objects.requireNonNull(size);
	        }
	        
	        @Override
	        public NyPizza build() {
	            return new NyPizza(this);
	        }
	        
			@Override
	        protected Builder self() {return this;}
	    }
	    private NyPizza(Builder builder) {
	        super(builder); // 부모 클래스의 멤버는 부모 클래스의 Builder가 초기화
	        size = Builder.size;
	    }
	}
	
	public class Calzone extends Pizza {
	    private final boolean sauceInside;
	    
	    public static class Builder extends Pizza.Builder<Builder> {
	        private boolean sauceInside = false; // 기본값
	        public Builder sauceInside() {
	            sauceInside = true;
	            return this;
	        }
	        
	        @Override
	        public Calzone build() {
	            return new Calzone(this);
	        }
	        
	        @Override
	        protected Builder self() {return this;}
	    }
	    private Calzone(Builder builder) {
	        super(builder); // 부모 클래스의 멤버는 부모 클래스의 Builder가 초기화
	        sauceInside = builder.sauceInside;
	    }
	}
	```

	

* 장점

	1. 매개변수 타입에 대해서 유연하다.
	2. 매개변수 각각에 대해서 유효성 검사가 가능하다.
	3. 계층적으로 설계된 클래스와 함께 사용하기 좋다.
		* 빌더 패턴을 적용할 클래스의 내부에 타입에 맞는 빌더 클래스 정의
		* 부모 클래스의 멤버는 부모 클래스의 Builder에게 처리를 맡길 수 있다
		* 자신의 고유 멤버는 자신의 클래스의 Builder를 통해 처리한다.
	4. 가변인수 매개변수를 여러 개 사용할 수 있다.
		* 자바의 언어 스펙에 따라, method의 signiture에는 최대 1개의 가변인자를 포함 가능
		* 각각의 멤버 변수(=필드)에 Builder의 setter계열 함수를 적용할 경우,  사실 상 가변인자를 여러 개 포함한 것과 동일한 효과를 얻는다.

* 단점

	* Builder 패턴을 적용하기 위해, Overhead가 존재







