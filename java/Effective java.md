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

Service 인터페이스와 ServiceImpl 클래스는 다음과 같다.

```java
Interface Service{
	void doService();
}

class ServiceImpl implments Service{
    @Override
    public void doService(){
        System.out.println("work!");
    }
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

실행을 한다면 `work!`가 출력된다. 이제 다섯번째 장점을 다시 살펴보자

- 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 없어도 된다.
  - 위 클라이언트 코드를 작성하는 시점에는 ServiceImpl.class가 없어도 된다. 런타임시에만 존재하면 되기 때문이다.
  - 마찬가지로 A 프로젝트는 ServiceImpl.class는 의존하지 않아 인터페이스 기반으로 프로그래밍을 할 수 있다.
  - 서비스 제공자 프레임워크를 만드는 근간이 된다.

**서비스 제공자 프레임워크**

  - 목적은 확장 가능한 애플리케이션을 설계하는 것
    - 다양한 구현체가 생성될 수 있는 인터페이스를 서비스 제공자 인터페이스(Service)
    - SPI를 구현하는 구현체
    - 서비스 제공 API, 구현체를 등록하는 방법
    - 서비스 접근 API, 구현체를 사용하는 방법

  - Spring에서 서비스 제공 API는 `@Configuration`, 서비스 접근 API는 `.getBean()`에 해당한다
  - JAVA 예제에서 서비스 제공 API는 `META-INF/services`, 서비스 접근 API는 클라이언트 코드에서 ServiceLoader에 해당한다.(구현체에 접근하므로)

- 단점
  - 정적 팩터리 메서드만 제공하면 상속이 불가능하다
    - 생성자가 private이므로 하위 클래스 사용이 불가능하다.

        ```java
        public class Coffee(){
            private Coffee()
            ...
        }
        //불가능
        public class DeliciousCoffee() extends Coffee(){

        }
        ```
    - 직접 하위 클래스를 생성하는 대신 컴포지션을 통해 우회할 수 있다. 컴포지션을 유도한다는 점에서 장점으로 해석될 수도 있다.
      
        ```java
        public class Coffee(){
            private Coffee()
            ...
        }
        //가능
        public class DeliciousCoffee(){
            Coffee coffee;
        }
        ```
  - 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
    - 터미널에 `mvn javadoc:javadoc` 명령어가 있는데 이는 주석을 기반으로 문서화 해주는 기능
    - 생성자 같은 경우는 Constructor라고 따로 빼놓고 설명해주지만 정적 팩터리 메서드는 메서드와 구분이 안간다.
    - 이름을 기반으로 프로그래머를 배려하자
      - from : 매개변수 하나를 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드
      - of : 매개변수를 여러개 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드

        [daangn-server]
        ```java
        public class ProductImage extends AuditingCreateUpdateEntity {

            @Id
            @GeneratedValue(strategy = GenerationType.IDENTITY)
            private Long id;

            @ManyToOne(fetch = FetchType.LAZY)
            @JoinColumn(name = "product_id")
            private Product product;

            @Column(nullable = false, length = 250)
            private String imageUrl;

            public static ProductImage of(Product product, String imageUrl) {
                return new ProductImage(product, imageUrl);
            }

            private ProductImage(Product product, String imageUrl) {
                checkArgument(product != null, "product 값은 필수입니다.");
                checkArgument(isNotEmpty(imageUrl), "imageUrl 값은 필수입니다.");
                checkArgument(imageUrl.length() <= 250, "imageUrl 값은 250자 이하여야 합니다.");

                this.product = product;
                this.imageUrl = imageUrl;
            }
        }
        ```

**p9. 열거타입은 인스턴스가 하나만 만들어짐을 보장한다**

-  `this.orderStatus == OrderStatus.DELIVERED` 와 같이 주소비교가 가능해진다.
-  Enum을 통해 Type Safety를 제공할 수 있다.

**p9. 객체가 자주 요청된다면 플라이웨이트 패턴을 사용할 수 있다**

- 여러개의 비슷한 객체를 생성하는 메모리 비용을 아끼는 패턴이다
- Java의 String Constant Pool이 플라이웨이트 패턴을 사용한다
  - 스트링 객체 대신 리터럴로 생성된 문자열을 재사용할 수 있어 메모리를 아끼게 된다.

**p10. 자바8부터 인터페이스가 정적 메서드를 가질 수 없다는 제한이 풀렸기 때문에 동반 클래스를 굳이 사용하지 않아도 된다**

- java8 부터 인터페이스 내부에 객체 없이 접근할 수 있게 해주는 static 메서드나 객체로 생성한 후 사용할 수 있는 default 메서드 기능이 추가되었다.
  -  이 기능을 통해 인터페이스의 기능들이 풍부해졌다

    ```java
    // List Interface, 기존에는 직접 리스트를 순회했지만 기본 메서드 덕에 편하게 사용가능해졌다.
    default void replaceAll(UnaryOperator<E> operator) {
      Objects.requireNonNull(operator);
      final ListIterator<E> li = this.listIterator();
      while (li.hasNext()) {
          li.set(operator.apply(li.next()));
      }
    }
    ```

**p12. 서비스 제공자 인터페이스가 없다면 각 구현체를 인터페이스로 만들 때 리플렉션을 사용해야한다**

- 리플렉션이란 어딘가에 "반사된" 정보들을 통해 클래스나 객체를 조작하는 것.
- 자바에서 "어딘가"는 JVM 내 Class Loader에 해당한다
- 즉, 런타임에 Class Loader의 정보를 읽어들여 다음 작업들을 수행할 수 있다.
  - 객체 생성
  - 메서드를 호출
- 성능 문제를 항상 고려해야한다.
- Spring에서 어노테이션 검사시 리플렉션 기능을 활용할 수 있다