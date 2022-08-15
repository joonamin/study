# [item5] 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

많은 클래스가 하나 이상의 자원에 의존한다. 이 때 어떻게 설계하는 것이 유연성을 높일 수 있을까?

- 클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다.
- 또한 자원을 클래스가 직접 만들지 못하도록 해야 한다.
- **필요한 자원을(혹은 그 자원을 만들어주는 팩터리를) 생성자에 (혹은 정적 팩터리나 빌더에) 넘겨주어야 한다.**
- 이를 **의존성 객체 주입**이라고 한다.
- 유연성, 재사용성, 테스트 용이성

## 정적 유틸리티를 잘못 사용한 예

- 유연하지 않고 테스트하기 어렵다.

```java
public class SpellChecker {
	private static final Lexicon dictionary = new KoreanDictionary();

	private SpellChecker() {} 

	public static boolean isValid(String word) {
		throw new UnsupportedOperationException();
	}

	public static List<String> suggestions(String typo) {
		throw new UnsupportedOperationException();
	}

}
```

```java
public interface Lexicon { }
```

```java
public class KoreanDictionary implements Lexicon 
```

## 싱글턴을 잘못 사용한 예

- 유연하지 않고 테스트하기 어렵다.

```java
public class SpellChecker2 {
	 private final Lexicon dictionary = new KoreanDictionary();

	    private SpellChecker2() {}

	    public static final SpellChecker2 INSTANCE = new SpellChecker2() {};

	    public boolean isValid(String word) {
	        throw new UnsupportedOperationException();
	    }

	    public List<String> suggestions(String typo) {
	        throw new UnsupportedOperationException();
	    }
}
```

- **사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.**
- 맞춤법 검사기는 사전(dictionary)에 의존하는데 사전이 언어별로 따로 있다. SpellChecker가 여러 사전을 사용할 수 있도록 만들어보자.
- 클래스(SpellChecker)가 여러 자원 인스턴스를 지원해야 하고, 클라이언트가 원하는 자원(dictionary)를 사용햐여 한다.
- 이 때 **인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식을 쓴다.**

## 적절한 구현

```java
public class SpellChecker3 {
	private final Lexicon dictionary;

    **public SpellChecker3(Lexicon dictionary) {**
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) {
        throw new UnsupportedOperationException();
    }
    
    public List<String> suggestions(String typo) {
        throw new UnsupportedOperationException();
    }
		
		public static void main(String[] args) {
				Lexicon lexicon = new KoreanDictionary();
				//Lexicon lexicon = new TestDictionary();
				SpellChecker3 spellChecker = new SpellChecker3(lexicon);
		}
		
}
```

```java
public class KoreanDictionary implements Lexicon {}
```

- 의존 객체 주입은 생성자, 정적 팩터리, 빌더 모두에 똑같이 응용할 수 있다.

- 변형으로 생성자에 자원 팩터리를 넘겨주는 방식이 있다.
    - 팩터리 : 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체
    - **팩토리 메서드 패턴**: 부모 클래스에 알려지지 않은 구체 클래스를 생성하는 패턴이며, 자식 클래스가 어떤 객체를 생성할지 결정하는 패턴
    
    ```java
    public abstract class Pizza {
    	public abstract int getPrice();
    
    	public enum PizzaType{
    		HamMushroom, Deluxe, Seafood
    	}
    
    	public static final Pizza PizzaFactory(PizzaType pizzaType) {
    		switch (pizzaType){
    			case HamMushroom :
    				return new HamMushroomPizza();
    			case Deluxe :
    				return new DeluxePizza();
    			case Seafood :
    				return new SeafoodPizza();
    		}
    		throw new RuntimeException("The pizza type " +
    			pizzaType.toString() + "  is not Recognized.");
    	}
    
    }
    ```
    
    ```java
    public class HamMushroomPizza extends Pizza {
    	private int price = 15_000;
    
    	@Override
    	public int getPrice() {
    		return price;
    	}
    }
    ```
    
    ```java
    	public class DeluxePizza extends Pizza {
    	private int price = 20_000;
    
    	@Override
    	public int getPrice() {
    		return price;
    	}
    }
    ```
    
    ```java
    public class SeafoodPizza extends Pizza {
    	private int price = 23_000;
    
    	@Override
    	public int getPrice() {
    		return price;
    	}
    }
    ```
    
    - **Supplier<T> 인터페이스**
    
    ```java
    Mosaic create(Supplier<? extends Tile> tileFactory) {...}
    ```
    
    - 위 예는 클라이언트가 제공한 팩터리가 생성한 Tile들로 구성된 Mosaic를 만드는 메서드이다.

- 의존 객체 주입이 유연성과 테스트 용이성을 개선해주지만, 의존성이 수천개나 되는 큰 프로젝트에서는 코드를 어지럽게 만들 수도 있다.
    - 대거(Dagger), 주스(Guice), 스프링(Spring) 같은 의존 객체 주입 프레임워크를 사용하면 해결할 수 있다.

