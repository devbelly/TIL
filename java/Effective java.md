<p align="center">
  <img src="https://user-images.githubusercontent.com/67682840/233975701-e720b359-89d1-4ba7-9859-0d55f614cfbb.png">
</p>

# 1장, 생성자 대신 정적 팩터리 메서드를 고려하라.

- 반드시 생성자를 정적 팩터리 메서드로 대체해야하는 것은 아니다.

- 장점
  - 이름을 가질 수 있다. 
    - 이를 통해 반환될 객체의 특성을 잘 나타낼 수 있다. 
    - 메서드 시그니처가 같은 생성자가 여러개 필요한 경우에도 적용할 수 있다.
  - 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
    - 미리 생성한 인스턴스를 캐싱하여 리턴할 수 있다.
	- 플라이웨이트 패턴과 유사하다

> FlyWeight? 
>
> 생성 비용이 큰 객체를 자주 생성하는 경우 캐싱하여 메모리 사용을 캐싱하는 기법, 구현 시 세 파트로 나눌 수 있다
> - FlyWeight 인터페이스
> - FlyWeight 구현 클래스
> - Map을 통해 객체를 관리하는 클래스

<details>
<summary>코드</summary>

```java
interface Flyweight {
    void operation();
}

class ConcreteFlyweight implements Flyweight {
    private final String intrinsicState;

    public ConcreteFlyweight(String intrinsicState) {
        this.intrinsicState = intrinsicState;
    }

    @Override
    public void operation() {
        System.out.println("ConcreteFlyweight: " + intrinsicState);
    }
}

class FlyweightFactory {
    private Map<String, Flyweight> flyweights = new HashMap<>();

    public Flyweight getFlyweight(String key) {
        if (flyweights.containsKey(key)) {
            return flyweights.get(key);
        } else {
            Flyweight flyweight = new ConcreteFlyweight(key);
            flyweights.put(key, flyweight);
            return flyweight;
        }
    }
}

public class Client {
    public static void main(String[] args) {
        FlyweightFactory factory = new FlyweightFactory();

        Flyweight flyweight1 = factory.getFlyweight("A");
        flyweight1.operation(); // 출력 결과: "ConcreteFlyweight: A"

        Flyweight flyweight2 = factory.getFlyweight("A");
        flyweight2.operation(); // 출력 결과: "ConcreteFlyweight: A"

        System.out.println(flyweight1 == flyweight2); // 출력 결과: "true"
    }
}
```
</details>
		
  - 인터페이스 또는 상위타입을 리턴할 수 있다.
    - 클라이언트에서 인터페이스 기반 프레임워크를 사용하도록 할 수 있다.
      - 클라이언트는 Hibernate가 아닌 JPA와 같은 인터페이스 기반 프레임워크에 의존
  
> 인터페이스 기반 프레임워크
>
> 자바8 이전에는 인터페이스에 정적 팩토리 메서드를 사용할 수 없어서 인스턴스가 불가능한 클래스(companion class)를 만들어 해당 클래스에 정적 메서드 들을 넣어놓았다. Collections의 구현을 보면 다음과 같다.
```java
public class Collections {
    // Suppresses default constructor, ensuring non-instantiability.
    private Collections() {
    }
}
```
> 자바8 이후부터는 인터페이스에도 static 메서드를 사용할 수 있으므로 이전의 Collections와 같은 클래스를 생성하는 대신 인터페이스를 활용할 수 있게 되었다.

   - 전달되는 매개변수에 따라 리턴되는 클래스가 달라질 수 있다
     - 코드

		```java
		public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
			Enum<?>[] universe = getUniverse(elementType);
			if (universe == null)
				throw new ClassCastException(elementType + " not an enum");

			if (universe.length <= 64)
				return new RegularEnumSet<>(elementType, universe);
			else
				return new JumboEnumSet<>(elementType, universe);
		}
		```

		- 크기에 따라 RegularEnumSet, JumboEnumset을 리턴함을 알 수 있다.
		- 클라이언트는 구체적인 클래스에 의존하지 않으므로 유연성이 증가하게 된다.



**ServiceLoader** 

다섯번째 장점을 살펴보기전 ServiceLoader에 대해 알아보자. 런타임 시에 클래스를 동적으로 로드하고 서비스를 찾는데 사용된다. 예를 들어 아래와 같은 두 프로젝트가 있다고 가정하자. 아래서 살펴보는 프로젝트 A는 B를 참조하고 있다.
```
A/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com.example.a/
│   │   │       └── Service
│   │   └── resources/
│   └── test/
└── build.gradle

B/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com.example.b/
│   │   │       └── ServiceImpl.java
│   │   └── resources/
│   │       └── META-INF/
│   │           └── services/
│   │               └── com.example.a.Service(파일)
│   └── test/
└── build.gradle

```

Service 인터페이스와 

```java
Interface Service{
	void doService();
}
```

프로젝트 A를 사용하는 클라이언트에서 다음과 같이 코드를 사용한다고 해보자.

```java
ServiceLoader<Service> loader = ServiceLoader.load(Service.class);
Optional<Service> optional = loader.findFirst();
optional.ifPresent(h -> {
	sout(h.doService())
})
```

실행을 한다면