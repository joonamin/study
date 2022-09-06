# [item15] 클래스와 멤버의 접근 권한을 최소화하라

**잘 설계된 컴포넌트란?**

1. 캡슐화가 얼마나 잘 되었는지.
2. 노출되는 API와 실제 구현이 얼마나 잘 분리되었는지.
3. 메시지를 주고 받는 두 컴포넌트가 서로의 내부 동작을 신경쓰지 않는지.

**캡슐화를 잘 지켰을 때의 장점**

1. 서로의 구현을 몰라도 되기 때문에 병렬로 개발이 가능하며 개발 속도가 빨라진다.
2. 잘 분리되어 있는 컴포넌트는 관리포인트가 작다. 디버깅도 빨리 할 수 있고 다른 컴포넌트로의 교체도 빠르게 할 수 있다.
3. 잘 분리되어 있는 컴포넌트는 최적화도 그 컴포넌트만 하면 되기 때문에 좋다. (다른 컴포넌트에 영향을 주지 않고 해당 컴포넌트만 최적화할 수 있다.)
4. 외부 컴포넌트에 종속되지 않기 때문에 재사용성이 높다.
5. 전체 시스템이 완성되지 않아도 개별 컴포넌트를 검증할 수 있기 때문에 큰 시스템을 개발하는 난이도를 낮춰준다.

## 캡슐화의 핵심은 “접근제어자”

> 기본원칙: 모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다.
> 

**private**: 멤버를 선언한 톱레벨 클래스에서만 접근 가능

**package-private**: 멤버가 소속된 패키지 안의 모든 클래스에서 접근 가능

**protected**: package-private+하위클래스에서 접근 가능

**public**: 모든 곳에서 접근 가능

- 톱레벨 클래스 : 중첩 클래스가 아닌 클래스(즉, 단일 형태의 클래스)
- 중첩 클래스: 다른 클래스/인터페이스 내부에 선언된 클래스

톱레벨 클래스나 인터페이스에 부여할 수 있는 접근 수준은 package-private와 public 두 가지이다.

- public으로 선언하면 공개 API가 된다. → 하위호환을 위해 지속적으로 관리해주어야 한다.
- package-private으로 선언하면 해당 패키지 안에서만 이용할 수 있다. → 클라이언트에 아무런 피해 없이 다음 릴리스에서 수정, 교체, 제거할 수 있다.

**캡슐화 방법**

공개 API를 설계하고 그것만 public으로 지정한다 > 나머지는 private으로 만든다 > 구현 중 같은 패키지의 다른 클래스가 접근해야 하면 package-private 으로 풀어준다.

- 이 과정에서 너무 자주 풀어주는 일이 발생한다면 컴포넌트를 더 분해해야 하는 것일 수도 있다.

**+ 추가로 고려해야 할 사항**

1. 한 클래스에서만 사용하는 package-private 톱레벨 클래스나 인터페이스는 private 정적클래스로 중첩시켜본다. 이 중첩클래스는 포함된 톱레벨 클래스에서만 접근할 수 있다.
2. public일 필요가 없는 클래스는 package-private 클래스로 바꾸자.

**공개 API의 제약**

공개 API는 영원히 지원되어야 한다. 또, 필요에 따라 문서화되어야 한다. 그러므로 접근제어는 최대한 좁힐 수 있을 만큼 좁히자.

> 하지만 리스코프 원칙을 지키기 위해 하위 클래스는 상위 클래스보다 좁혀질 수 없다.
> 
- 리스코프 치환 원칙 : 상위 클래스의 인스턴스는 하위 클래스의 인스턴스로 대체해 사용할 수 있어야 한다.

## public 클래스의 인스턴스 필드가 public이 되어서는 안되는 이유

1. 변경에 매우 취약하다.
- 해당 필드와 관련된 모든 것은 불변식을 보장할 수 없게 된다.
1. 스레드 안전하지 않다.
- 인스턴스 변수는 힙에 할당되며 공유자원이다. 즉, 모든 스레드가 이 공유자원에 접근할 수 있다.
- public 인스턴스 변수는 필드가 수정될 때 Lock 획득 같은 “thread safe” 하도록 부가적인 작업을 할 수 없기 때문에 사용해서는 안된다.
    
    
- 인스턴스 변수란?

```java
public class test {

	int iv; // 인스턴스 변수
	static int cv; // 클래스 변수
	
	void method() {
		int lv; // 지역 변수
	}
}
```

![21226F42578D2F8137.jpg](%5Bitem15%5D%20%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A2%E1%84%89%E1%85%B3%E1%84%8B%E1%85%AA%20%E1%84%86%E1%85%A6%E1%86%B7%E1%84%87%E1%85%A5%E1%84%8B%E1%85%B4%20%E1%84%8C%E1%85%A5%E1%86%B8%E1%84%80%E1%85%B3%E1%86%AB%20%E1%84%80%E1%85%AF%E1%86%AB%E1%84%92%E1%85%A1%E1%86%AB%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8E%E1%85%AC%E1%84%89%E1%85%A9%E1%84%92%E1%85%AA%20c3877a39f8c84716b5a58da58c345ba4/21226F42578D2F8137.jpg)

1. public final 필드의 문제점
- private 필드라면 내부 구현 변경 시 private 필드를 삭제할 수 있고, Getter든 Setter든 해당 클래스 내부만 수정하면 된다. 하지만 public final 필드의 경우 내부 구현 변경시 이 필드가 직접 사용되고 있는 모든 소스코드를 찾아가 수정해야 한다.

**유일하게 허용되는 public static final 필드**

추상 개념을 완성하는데 꼭 필요한 “상수”

관용적으로 대문자알파벳과 _의 조합으로 이루어져 있으며 기본타입 또는 불변객체를 참조해야 한다.

> public static final 필드가 가변객체를 참조한다면?
> 

```java
public class Member2 {
	private static final long serialVersionUID = 1L;
	private static final Subject subject = new Subject("eunyeong");
	private String id;
	private String password;
	private String name;

	public Member2(String id, String password, String name) {
		this.id = id;
		this.password = password;
		this.name = name;
	}

	public Subject getSubject() {
		return subject;
	}

	public String getTeacher() {
		return subject.getTeacher();
	}
}
```

```java
public class Subject {
	private String teacher;
	public Subject(String teacher) {
		this.teacher=teacher;
	}
	public String getTeacher() {
		return this.teacher;
	}
	public void setTeacher(String teacher) {
		this.teacher = teacher;
	}
	
}
```

```java
//private static final 이 가변객체를 가리키면 수정될 수 있다.
public class ChangibleObject {
	public static void main(String[] args) {
		Member2 member = new Member2("eunyeong@naver.com", "1234", "정은영");
		String originTeacher = member.getTeacher();
		member.getSubject().setTeacher("hyunseo");
		
		String teacher = member.getTeacher();
		if(originTeacher.equals(member.getTeacher())) {
			System.out.println("equal");
		}
		else {
			System.out.println("not equal");
		}
	}

}
```

- 위와 같이 가변객체 Subject를 private static final로 참조한다면, 해당 참조만 그대로 유지하고 그 값만 변경하는 식으로 “불변”을 깰 수 있다.
- public 클래스는 상수용 public static final 필드 외에는 어떠한 public 필드도 가져서는 안된다.
- public static final 필드가 참조하는 객체가 불변인지 확인하라.