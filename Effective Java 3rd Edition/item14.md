[toc]

# item14. Comparable을 구현할지 고려하라

## Comparable Interface 정의

>```
>public interface Comparable<T>
>```
>
>This interface imposes a total ordering on the objects of each class that implements it. This ordering is referred to as the class's *natural ordering*, and the class's `compareTo` method is referred to as its *natural comparison method*.
>
>Lists (and arrays) of objects that implement this interface can be sorted automatically by [`Collections.sort`](dfile:///Users/kangminjun/Library/Application Support/Dash/DocSets/Java_SE11/Java.docset/Contents/Resources/Documents/java.base/java/util/Collections.html#sort(java.util.List)) (and [`Arrays.sort`](dfile:///Users/kangminjun/Library/Application Support/Dash/DocSets/Java_SE11/Java.docset/Contents/Resources/Documents/java.base/java/util/Arrays.html#sort(java.lang.Object[]))). Objects that implement this interface can be used as keys in a [sorted map](dfile:///Users/kangminjun/Library/Application Support/Dash/DocSets/Java_SE11/Java.docset/Contents/Resources/Documents/java.base/java/util/SortedMap.html) or as elements in a [sorted set](dfile:///Users/kangminjun/Library/Application Support/Dash/DocSets/Java_SE11/Java.docset/Contents/Resources/Documents/java.base/java/util/SortedSet.html), without the need to specify a [comparator](dfile:///Users/kangminjun/Library/Application Support/Dash/DocSets/Java_SE11/Java.docset/Contents/Resources/Documents/java.base/java/util/Comparator.html).

* `Comparable.compareTo(T)` 

	Comparable 인터페이스의 유일한 메소드.

	단순 동치성 뿐만 아니라, 순서(Order)까지 비교가 가능

	

* Comparable 인터페이스의 `compareTo`를 구현하는 것으로 객체간의 Ordering이 가능해진다.

> 즉, 동치성 비교 뿐만 아니라 인스턴스의 Ordering을 목적으로 한다면 해당 클래스는 Comparable 인터페이스를 구현해야한다.



## `compareTo(T)` 의 일반 규약

* `compareTo(T)` 는 주어진 객체보다 <u>작으면 음의 정수</u> , <u>같으면 0</u>, <u>크면 양의 정수</u> 를 반환한다. 만약 현재 인스턴스와 비교할 수 없는 인스턴스가 주어지면 `ClassCastException`을 던진다.



1. `Comparable`을 구현한 클래스는 모든 x, y에 대해 `sign(x.compareTo(y)) == -sign(y.compareTo(x))` 를 만족해야한다. 둘 중 하나의 수식에 Exception이 던져질 경우에만 나머지 하나도 Exception을 던진다.

이는 당연하다. $x < y$ 이면서 $x > y$ 일 경우는 비교 그 자체에 모순적이기 때문이다.



2. `Comparable`을 구현한 클래스는 **추이성을 보장**해야한다. `(x.compareTo(y) > 0 && y.compareTo(z) > 0)` 라면 `x.compareTo(z) > 0` 을 만족해야한다.



3. `Comparable`을 구현한 클래스는 모든 z에 대해 `x.compareTo(y) == 0` 이면 `sign(x.compareTo(z)) == sign(y.compareTo(z))`이다.



4. (권장) `(x.compareTo(y) == 0)` == `(x.equals(y))`

임의의 인스턴스 2개를 비교할 때, *주어진 객체보다 <u>작으면 음의 정수</u> , <u>같으면 0</u>, <u>크면 양의 정수</u> 를 반환한다.* 의 방식으로 구현할 경우 위 조건을 만족하게 된다. <u>하지만</u>, 단순하게 x가 y보다 크다면 1 그 외에는 모두 0을 반환한다고 정의할 경우에 위 권고사항을 만족하지 못한다.

권고 사항을 준수한다면, `compareTo(T)` 의 결과와 `equals(T)` 의 **결과의 일관성을 보장**할 수 있다. 간혹, `compareTo(T)`를 내부 연산으로 활용하는 컬렉션과 `equals(T)`를 내부 연산으로 활용하는 컬렉션을 혼용한다면 이 부분에서 예기치 못한 문제가 발생할 수 있으므로 4번 권고사항을 지키지 않게 설계한다면 그에 대한 description을 명시해야한다.



ex) `BigDecimal(String)` 

> ![Screen Shot 2022-08-24 at 12.48.40 AM](https://raw.githubusercontent.com/joonamin/UpicImageRepo/master/uPic/Screen%20Shot%202022-08-24%20at%2012.48.40%20AM.png)
>
> 위와 같이 BigDecimal은 문자열이 실제 동일한 값(<u>0 <-> 0.00</u>)을 가진다고 해도 내부 표현방식이 다를 수 있기 떄문에 compareTo(T) 연산과 equals(T) 연산의 결과가 다를 수 있다. [<u>위의 경우</u>는 compareTo = true, equals = false]
>
> 이 경우에, equals를 내부 연산으로 사용하는 `HashSet` 에 0과 0.00을 추가할 경우에는 원소가 2개 저장되고 compareTo를 내부 연산으로 사용하는 `TreeSet`은 0과 0.00은 numerically equal이기 때문에 원소가 1개 저장된다.
>
> - #### compareTo
>
> 	```
> 	public int compareTo(BigDecimal val)
> 	```
>
> 	Compares this `BigDecimal` with the specified `BigDecimal`. Two `BigDecimal` objects that are equal in value but have a different scale (like 2.0 and 2.00) are considered equal by this method. This method is provided in preference to individual methods for each of the six boolean comparison operators (<, ==, >, >=, !=, <=). The suggested idiom for performing these comparisons is: `(x.compareTo(y)` <*op*> `0)`, where <*op*> is one of the six comparison operators.
>
> 	- **Specified by:**
>
> 		`compareTo` in interface `Comparable<BigDecimal>`
>
> 	- **Parameters:**
>
> 		`val` - `BigDecimal` to which this `BigDecimal` is to be compared.
>
> 	- **Returns:**
>
> 		-1, 0, or 1 as this `BigDecimal` is numerically less than, equal to, or greater than `val`.



## `compareTo(T)` 작성 요령

>  `Comparable`의 `compareTo()`와 `Comparator`의 `compare` 의 예시는 다음 소제목에서 다루겠다.

---

1. `compareTo(T)` 의 T는 `Comparable<T>` 제너릭 인터페이스의 타입이므로, **컴파일 타임**에 타입이 정해진다.

그렇기 때문에, 추가적으로 T에 대한 타입 체크 및 캐스팅 과정이 필요없다. (타입에 대한 문제 발생시 컴파일 불가능)



2. `equals()`와는 다르게 Ordering에 초점을 둔다. 객체 레퍼런스 필드를 비교하기 위해서는 레퍼런스가 가리키는 인스턴스의 `compareTo()` 를 재귀적으로 호출한다. 

이 때, <u>해당 인스턴스가 `Comparable` 인터페이스를 구현하지 않거나 Ordering을 기본 정렬 기준과 다르게 Customize하기 위해서라면 `Comparator` 을 정의하거나 자바가 제공하는 것을 선택한다.</u>

**보통 `Comparator`는 동일 클래스 간의 비교만을 허용하여 추이성 & 대칭성을 보장한다. (일반적인 패턴)**




3. `compareTo` 메소드에서 `<` , `>` 등을 사용하는 것 대신 Comparator의  `compare()` 을 통해 비교를 수행하자

자바 8부터 권장되는 방식인데, Comparator 함수형 인터페이스의 비교자 생성 메서드(ex. comparing-)을 통해 메소드 연쇄 방식으로 비교자를 생성할 수 있게 되었다.  이 방식을 `compareTo()` 와 결합하므로서 객체지향적인 compareTo() 설계가 가능해진다.




> - #### comparingInt
>
> 	```
> 	static <T> Comparator<T> comparingInt(ToIntFunction<? super T> keyExtractor)
> 	```
>
> 	Accepts a function that extracts an `int` sort key from a type `T`, and returns a `Comparator<T>` that compares by that sort key.
>
> 	The returned comparator is serializable if the specified function is also serializable.
>
> 	- **Type Parameters:**
>
> 		`T` - the type of element to be compared
>
> 	- **Parameters:**
>
> 		`keyExtractor` - the function used to extract the integer sort key
>
> 	- **Returns:**
>
> 		a comparator that compares by an extracted key
>
> 	- **Throws:**
>
> 		`NullPointerException` - if the argument is null
>
> 	- **Since:**
>
> 		1.8
>
> 	- **See Also:**
>
> 		[`comparing(Function)`](dfile:///Users/kangminjun/Library/Application Support/Dash/DocSets/Java_SE11/Java.docset/Contents/Resources/Documents/java.base/java/util/Comparator.html#comparing(java.util.function.Function))

ex) 비교자 생성 메소드를 활용한 비교자 활용

```java
private static final Comparator<PhoneNumber> COMP = Comparator<PhoneNumber>.comparingInt((PhoneNumber) pn -> pn.areaCode) // PhoneNumber객체에서 areaCode를 추출하여 compare()을 수행한다. (비교 1순위)
    .thenComparingInt(pn -> pn.prefix) // 연쇄적으로 comparator를 정의할 수 있다.  (비교 2순위)
    .thenComparingInt(pn -> pn.lineNum); // 연쇄적으로 comparator를 정의할 수 있다.(비교 3순위)

@Override
public int compareTo(PhoneNumber pn) {
    return COMP.compare(this, pn); // PhoneNumber객체에 대한 '비교'연산을 위임한다.
}
```



## 주의!

Comparator의 `compare(T, T)` 를 사용하여 객체를 비교할 경우 `-`기반의 연산을 피하자.

1. ex) `-` 기반의 compare()의 문제점

``` java
// 해시 코드 값의 차를 기준으로 하는 비교자 (추이성 위반)
static Comparator<Object> hashCodeOrder = new Comparator<>() {
  @Override
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode(); // 32bit정수 - 32bit정수 = 32bit 저장시 오버플로우 가능성 존재
    }
};
```

2. 위 코드를 개선한 방법 (Wrapper Class의 기본 `compare()` 활용)

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
  @Override
    public int compare(Object o1, Object o2) {
		return Integer.compare(o1.hashCode(), o2.hashCode()); // 정수값 비교
    }
};
```

3. 메소드의 body가 3줄 이하라면 람다 기반의 `비교자 생성 메소드`를 사용하는 것이 가독성이 좋다.

```java
// KeyExtractor는 functional Interface
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```

---

## 💡 `compareTo(T)` vs `compare(T, T)` 

* `compareTo(T)` 

`Comparable<T> ` interface를 기반으로 구현. 해당 클래스가 가지는 **기본적인 순서(natural order)**를 정의할 때 사용한다.


* `compare(T, T)` 

`Comparator<T>` interface를 기반으로 구현. 2개의 오브젝트의 Ordering을 위하여 사용된다. 일반적으로 기본적인 순서외의 순서를 커스터마이징하고 싶거나 Ordering 기준을 여러가지 정의하고 싶을 때 사용한다.

인스턴스 레벨에서의 호출이 아니기 때문에 오버헤드를 줄일 수 있다. (*단순 비교를 위해서 임시 객체를 생성할 필요가 X*)

이 때문에, 비교 기반의 많은 컬렉션 클래스(ex. TreeSet, PriorityQueue...)등은 Comparator를 생성자의 인자로 받기도 한다.

---

사실 `compareTo(T)`와 `compare(T, T)` 는 대척점으로 사용되는 것이 아니다. 

일반적으로, Ordering 기반의 컬렉션 클래스들은 비교 연산을 수행함에 있어, `Comparator`가 존재하지 않는 경우 기본적으로 `compareTo()`에 의한 natural ordering을 진행하고 그 외에는 `Comparator`의 정의된 `compare()` 을 이용한 비교를 수행한다.

> ```
> priority queue is ordered by comparator, or by the elements'
>      * natural ordering, if comparator is null: For each node n in the
>      * heap and each descendant d of n, n <= d. 
>      (생략)
> ```

[PriorityQueue.java](http://hg.openjdk.java.net/jdk8/jdk8/jdk/file/687fd7c7986d/src/share/classes/java/util/PriorityQueue.java) 

> ```
> * @param comparator the comparator that will be used to order this set.
>      *        If {@code null}, the {@linkplain Comparable natural
>      *        ordering} of the elements will be used.
> ```

[TreeSet.java](https://hg.openjdk.java.net/jdk8/jdk8/jdk/file/687fd7c7986d/src/share/classes/java/util/TreeSet.java)