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
- 자바에서 "어딘가"는 JVM 내 Method Area에 해당한다
- 즉, 런타임에 Method Area의 정보를 읽어들여 다음 작업들을 수행할 수 있다.
  - 객체 생성
  - 메서드를 호출
- 성능 문제를 항상 고려해야한다.
- Spring에서 어노테이션 검사시 리플렉션 기능을 활용할 수 있다

<br>

# 2장, 생성자에 매개변수가 많다면 빌더를 고려하라.

- 필수 매개변수 & 선택적 매개변수가 많다면 점층적 생성자 패턴을 사용할 수 있다.

  매개변수가 적은 생성자에서 매개변수가 많은 생성자를 호출한다.
  ```java
  class Person{
    private final int age;
    private final int iq;
    private final int eq;

    Person(int age){
      this(age,10);
    }

    Person(int age,int iq){
      this(age,iq,10);
    }

    Person(int age,int iq, int eq){
      this.age = age;
      this.iq = iq;
      this.eq = eq;
    }
  }
  ```

  - 클라이언트 코드를 작성하기 어렵다.

    ```java
    new Person(180,70,0);
    ```

    - 각 매개변수의 의미를 한번에 파악하기 어려우며 클라이언트를 몇번째 값이 어떤 의미인지 파악하고 있어야 한다.(물론 코틀린은 이 문제를 해결함)

- 두번째 대안으로 자바빈즈 패턴을 사용할 수 있다.

  <img width="500" alt="image" src="https://user-images.githubusercontent.com/67682840/234449487-32b2459e-c1c2-4f77-be15-9ce97f7d6e4c.png">

  - 단점
    - 여러 메서드를 호출해야한다.
    - 객체가 완전히 생성되기까지 일관성이 무너진 상태이다.(도대체 어느 속성까지 초기화를 해야 온전한가?)

      자바빈즈 패턴을 사용하면 아래처럼 생성자 내에서 유효성 검사를 하지 못하므로 일관성이 무너지기 쉽다.

      <img width="582" alt="image" src="https://user-images.githubusercontent.com/67682840/234449936-247f0d9f-9cce-405a-9259-369f767fd6fc.png">

      - 이를 통해서 불변 객체를 생성하지 못하므로 런타임시 오류 가능성과 멀티쓰레드 환경에서의 안전성을 보장받지 못한다.(세터를 열어두는 것은 위험)
- 세번째 대안으로 빌더패턴을 사용할 수 있다.
  - 필수 매개변수로 빌더 객체를 생성하고
  - 선택 매개변수를 선택하고
  - build 메서드를 호출해 필요한 객체를 얻는다.
      
**왜 위 코드에서 private 생성자에 @Builder를 붙였을까?**

- @Builder는 클래스레벨, 생성자레벨에 사용가능
- 클래스레벨에 사용시, 모든 속성을 인자로 갖는 package-private 수준의 생성자가 생성된다.

  ```java
  @Builder
  public class BuildMe{
    private String username;
    private int age;
  }
  ```

  ```java
  public class BuildMe{
    private String username;
    private int age;

    BuildMe(String username, int age){
      this.username = username;
      this.age = age;
    }

    ...// builder 관련 코드
  }
  ```

- `BuildMe`는 내부 빌더클래스에서 접근할 것이므로 굳이 package-private일 필요는 없다. 이를 위해 클래스 레벨에서 @Builder를 사용하는 대신 생성자레벨에서 @Builder를 사용하자

- 교재에서 언급한대로 `checkArgument`를 통해 빌더의 생성자에서 입력매개변수를 검사함을 알 수 있다.(불변식)

**재귀적 타입 매개변수 제한**

- 다음 코드를 살펴보자

  <img width="538" alt="image" src="https://user-images.githubusercontent.com/67682840/234758838-cd5ff103-c468-4f0b-9072-7fd657f54526.png">

  - 제네릭에서 타입의 상한에 제한을 걸 때 자기 자신에게 제한을 거는 것을 재귀적인 매개변수 제한이라고 한다. 

- `sort` 에서 이러한 예시를 볼 수 있다. 이 코드를 이해하기 위해 `sort`를 살펴보자

  - 자바에서는 어떠한 타입이든 상관없이 타입간 비교가 가능하다면 정렬이 가능하다. 

    ```java
    public static <T> void sort(List<T> list){
      ...
      if(list.get(i).compareTo(list.get(j)))
      ...
    }
    ```

  - 이 코드의 문제점은 T가 비교가능한 값인지 알 수 없다는 점이다. 타입제한을 걸기 위해 `Comparable<T>` 인터페이스를 정의해보자

    ```java
    public interface Comparable{
      public int compareTo(int o);
    }
    ```

  - 이처럼 정의하면 int값에 대한 비교만 가능해진다. 우리가 원하는 것은 int뿐만 아니라 비교가능한 타입이 오기를 기대한다. 

    ```java
    public interface Comparable<T>{
      public int compareTo(T o);
    }
    ```

  - 어떤 타입 A가 `comparable<B>`의 서브타입이라면 무엇을 의미할까? 이것은 타입 A는 B타입과 비교가능한 메서드를 가지고 있다를 의미하고 다시말해 타입 A와 B의 비교가 가능해진다

  - 마지막으로 T가 `comparable<T>`의 서브타입이라면 타입 T는 T와 비교가능한 메서드를 가지고 있고 이는 타입 T끼리 비교가 가능하다는 의미이다.

  - 즉, 최종적으로 구현할 sort함수는 다음과 같은 모습을 한다

    ```java
    public static <T extends Comparable<T>> void sort(List<T> list){
      ...
    }
    ```
- 위 내용을 토대로 추상 클래스 builder를 이해해보자

  - `T extends Builder<T>`에서 T는 addTopping이나 self 메서드에서 사용되고 이러한 T는 아무타입이나 오는 대신 Builder나 Builder를 상속하는 하위 클래스가 오기를 기대한다.

    <img width="548" alt="image" src="https://user-images.githubusercontent.com/67682840/234761755-d8168768-3c24-4b7e-8b44-bcda306b3736.png">

  - Ny.Pizza의 Builder는 Pizza.Builder가 요구하는 메서드를 다 구현했고, `<Builder>`에 NyPizza.Builder 타입을 넘겨줌으로써 자신이 사용하는 메서드에서 다루는 타입은 Pizza.Builder의 하위타입임을 만족한다.

**self()**

- 피자 추상 클래스에 속한 빌더는 `self()` 메서드를 제공한다
  
  <img width="535" alt="image" src="https://user-images.githubusercontent.com/67682840/234762514-83af677a-04bb-4628-a272-a70a976a4337.png">

- self() 대신 this를 리턴하게 되면 하위타입 빌더 사용시 항상 타입 캐스팅을 해야하는 문제가 있다.

**IllegalArgumentException**

- 잘못된 인자를 넘겨받았을 때 사용할 수 있는 런타임 예외
- CheckedException vs UncheckedException
  - CheckedExcpetion : `try-catch`로 예외를 처리하거나 다시 던져야한다. 복구 가능
  - UncheckedException : 클라이언트가 예외를 처리할 수 없을 때 사용. 복구 불가능
- 예외를 던질때는 잘못된 파라미터나 필드에 대한 정보를 넘겨주는 것이 좋다.
- 하위 계층에서 CheckException을 사용하면 상위 계층까지 해당 예외에 대한 의존성이 전파되므로 주의해서 사용해야한다.

**가변인수를 여러개 사용할 수 있다**

- 가변인수는 메서드에 하나만 쓸 수 있고 가장 마지막에 선언할 수 있다.
  
  ```java
  public class Person{
      public void test(int... grades){
        ...something..
      }
  }
  ```

- 빌더를 활용하면 각 메서드마다 가변인수를 하나씩 사용할 수 있으므로 여러개의 가변인수를 사용할 수 있다는 장점이 있다.

<br>

# 3장, private 생성자나 열거 타입으로 싱글턴임을 보장하라

- 싱글톤은 유일해야하는 시스템 컴포넌트이다.
- 클래스를 싱글톤으로 만들면 테스트하기 어렵다.

## 싱글톤을 만드는 방법

- public static 멤버를 통해 private 생성자에 접근하도록 한다.
- public static final 필드를 사용하거나 public static 메서드를 사용하는 방법이 있다.
- 단점
  - Reflection을 통해 private 생성자에 접근할 수 있다.
    
    ```java
    Constructor<Elvis> defaultConstructor = Elvis.class.getDeclaredConstructor();
    defaultConstructor.setAccessable(true);
    ...
    ```

    이 문제는 flag를 선언해 private 생성자에서 하나만 생성하도록 관리할 수 있다.
    
    ```java
    public class Elvis{
      public static final Elvis Instance = new Elvis();
      private Elvis(){
        if(flag){
          Throw new Exception();
        }
        flag=true;
      }
    }
    ```
    
  - 역직렬화시 새로운 인스턴스가 생성된다 

    - 객체를 직렬화를 통해 외부에 저장했다가 역직렬화로 다시 읽어올 수 있다.
    - 역직렬화 과정에서 동일한 객체가 생성될것이라고 기대할 수 있지만 실제로는 새로운 객체가 생성된다
    - `readResolve()`를 오버라이딩함으로써 이 문제를 해결할 수 있다.

      ```java
      private Object readResolve(){
        return INSTANCE;
      }
      ```

- public static 메서드

  ```java
     public class Elvis<T> {
      private static final Elvis INSTANCE = new Elvis();

      private Elvis() {
      }

      public static Elvis getInstance() { return INSTANCE; }
    } 
  ```

  - 필요하다면 쓰레드마다 객체를 생성해서 리턴하도록 수정할수 있다. 클라이언트 코드를 변경하지 않기 때문이다.

**정적 팩터리를 제네릭 싱글톤 팩토리로 만들 수 있다**

  - 제네릭을 사용하는 클래스에 대해 싱글톤으로 만들고 싶다면 `제네릭 싱글톤 팩토리`를 사용할 수 있다.
 
  - 원하는 타입을 사용할 수 있다는 장점이 있다.

    ```java
    public class Elvis<T> {
      private static final Elvis INSTANCE = new Elvis();

      private Elvis() {
      }

      public static <T> Elvis<T> getInstance() {
          return (Elvis<T>) INSTANCE;
      }

      public void leaveTheBuilding() {
          System.out.println("Whoa baby, I'm outta here!");
      }

      // 이 메서드는 보통 클래스 바깥(다른 클래스)에 작성해야 한다!
      public static void main(String[] args) {
          Elvis<Integer> elvis1 = Elvis.getInstance();
          Elvis<String> elvis2 = Elvis.getInstance();
          System.out.println(elvis1);
          System.out.println(elvis2);
      }
    } 
    ```

    [실행결과]
    ```
    effectivejava.chapter2.item3.staticfactory.Elvis@2f92e0f4
    effectivejava.chapter2.item3.staticfactory.Elvis@2f92e0f4    
    ```

**정적 팩토리의 메서드 참조를 공급자로 사용할 수 있다**

- Supplier는 LazyEvaluation을 위해 사용되는 기능

  <img width="263" alt="image" src="https://user-images.githubusercontent.com/67682840/235085486-739c5f2e-4fab-464a-8995-19e333daa025.png">

  - 인자로는 아무것도 받지 않고 객체를 리턴하는 모습임을 알수 있다.
  - `getInstance()`를 보면 인자로는 아무것도 받지않고 객체를 리턴하는 형태
  - 이러한 이유로 supplier를 사용하는 곳에 `Elvis::getInstacne`를 넘겨줘 사용할 수 있다.

- 코드
  
  ```java
  public static void test(Supplier<Elvis> test){
        test.get();
  }
  //test(Elvis::getInstance)와 같이 사용가능하다.
  ```

- Enum을 통한 싱글톤 객체 생성
  - 위에서 언급한 두가지 단점, 리플렉션과 역직렬화에 대한 문제가 해결되므로 Enum을 통한 싱글톤 객체 생성을 권장

  - 사용예시(배달의민족)

    <img width="853" alt="image" src="https://user-images.githubusercontent.com/67682840/235092495-72487329-f605-42fd-832f-f02f59e48649.png">

<br>

# 5장, 자원을 직접 명시하지말고 의존 객체 주입을 사용하라

- 정적 유틸리티 클래스나 싱글톤은 테스트하기 어렵다.
  
  ```java
  public class Spellchecker{
    private final Lexicon dictionary = ...;

    public boolean isValid(String word){...}
    public List<String> suggestions(String type) { ... }
  }
  ```

  - 만약 언어별로 다른 사전을 필요료 한다면 어떻게 해야할까?
    - koreanSpellChecker 객체를 생성? 테스트 하기 어렵다.
    - 서브 클래스 생성? 유연성이 떨어진다.
    - final을 제거하고 필요할때마다 세터를 통해 적절한 dic 세팅? 멀티쓰레드 환경에서 사용하기 어렵다.
  - 객체를 주입받는 식으로 하자

    ```java
    public class SpellCheckeer{
      private final Lexicon dictionary = ...;

      public SpellChecker(Lexicon dictionary){
        this.dictionary = dictionary;
      }
      ...
    }
    ```
    - 테스트 시 `mockDictionary`를 주입을 수 있으므로 테스트하기 쉽다.
    - 기존 SpellChecker에 있는 코드를 재사용할 수 있으므로 유연성이 증가한다.
    - final을 통해 불변을 보장하여 멀티쓰레드 환경에서 안전하다
  
**29p, 이 패턴의 쓸만한 변형으로 생성자에 자원 팩터리를 넘겨주는 방식이 있다.**

- 팩터리란 호출할때마다 특정 타입의 인스턴스를 리턴하는 객체를 말한다.
- 즉, 팩터리 메서드 패턴을 구현한 것이다.
- 팩터리 메서드 패턴?
  - 클라이언트에서 직접 new 연산자를 통해 제품 객체를 생성하지 않는다.
  - 객체 생성을 팩터리에 위임하여 클라이언트가 다른 객체를 생성하고 싶다면 다른 팩터리를 주입받아 사용한다.
  - 그외 사용 예시?

**29p, Supplier 인터페이스가 팩터리를 표현한 완벽한 예이다**

- 팩터리 메서드는 인자 없이 리턴(Product)하기만 한다
- 이 구조는 Supplier와 일치한다.

**p29, 한정적 와일드카드 타입을 사용해 팩터리 타입의 매개변수를 제한하자**

- 팩터리 메서드가 구체적인 타입을 리턴한다.
- 만약 `Supplier<Product>` 처럼 적으면 Product의 하위타입을 사용할 수 없으므로
- `Supplier<? extends Product>`를 통해 팩터리 메서드 패턴을 완성하자
  - 하지만 Product가 구체적인 클래스가 아니라 인터페이스라면 굳이 사용할 필요는 없을 듯 하다.

<br>

# 6장, 불필요한 객체 생성을 피하자

- `String str = new String("test")`
- `String str = "test"`
- 첫 번째 코드는 Heap에 매번 새로운 객체를 생성하고 두번째는 String Constant Pool에서 문자열을 재사용해 매번 동일한 객체를 사용함을 보장한다.

- 정규표현식 예시

  ```java
  static boolean isRomanNumeral(String s){
    return s.matches("//정규표현식");
  }
  ```
  - 자바에서 생성하는 정규표현식용 Pattern은 유한 상태 머신을 내부적으로 사용해 매칭을 하기 때문에 생성 비용이 비싸다.
  - 즉, //정규표현식 에 해당하는 부분을 private static final로 분리하자
  - 해당 클래스가 사용되지 않는다면 불필요하게 Pattern 정규표현식이 초기화 되었다. 이를 lazy initialization으로 변경할 수도 있지만 큰 성능차이를 체감하긴 어려우므로 권장하지 않는다.

- 오토박싱 예시
  - 기본타입과 래퍼타입을 같이 사용할 때 자동으로 변환해주는 기술이다.
  - 오토 박싱은 기본타입과 해당 기본 타입을 래핑한 타입간 경계를 허물지만 위험할 때도 있다.
  
**p34, 데이터베이스 연결 같은 경우 생성 비용이 워낙 비싸서 재사용하는 편이 낫다**

- 애플리케이션과 데이터베이스를 연결하기 위해서 커넥션 객체를 사용한다.
- 이러한 커넥션 객체는 생성 비용이 굉장히 비싸기 때문에 객체 풀링을 하는 것이 좋다.
- Spring에서는 HikariCP를 통해 객체풀링을 지원한다. 덕분에 값비싼 커넥션 객체를 매번 재생성 하지 않아도 된다.

**가비지 컬렉션**

- JVM에서 참조되고 있지 않은 객체를 메모리에서 해제하는 기술
- 과정
  - 모든 참조변수들을 확인하여 마킹한다. MARK
  - 마킹되지 않은 객체들을 제거한다. SWEEP
  - 비어있는 메모리 공간을 압축한다. COMPACT
- 시점
  - Heap을 Young generation, Old Generation으로 나눌 수 있다.
  - Young Generation은 eden, s0, s1로 나눌 수 있다.
  - 처음에 eden에 객체가 생성. eden이 전부 차게 되면 s0으로 이동
  - 과정 반복해서 s0에 객체가 다 쌓이면 s1으로 이동
  - 특정 age가 넘게 되면 Old Generation으로 이동한다
- 고려해야할 포인트
  - Throughtput
  - Latency
  - Footprint
- Stop the world
  - JVM이 GC를 위해 애플리케이션 스레드를 중지하고 GC 관련 쓰레드만 동작 시키는 것
  - latency에 영향을 끼친다
- 종류
  - serial GC
  - parallel GC (java8)
  - CMS GC (deprecated)
  - G1 GC
  - ZGC

<br>

# 7장, 다 쓴 객체 참조를 해제하라

- 예시

  ```java
  public Object pop() {
    if(size == 0){
      throw new EmptyStackException();
    }
    return element[--size];
  }
  ```  

  - 스택이 커졌다가 줄어들 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 제거하지 못한다.
  - 객체의 참조를 살려두면 해당 객체가 참조하고 있는 다른 객체들 또한 제거하지 못한다.
    (가비지컬렉션 과정 중 Mark는 객체 그래프를 탐색해서 모든 객체들을 마킹)
  - 올바르게 구현하려면 null을 명시적으로 표시해주면 된다.
- 예시

  ```java
  public Object pop() {
    if(size == 0){
      throw new EmptyStackException();
    }
    element[size]=null;
    return element[--size];
  }
  ```  
  - 이렇게 일일이 null처리를 하면 프로그램을 필요이상으로 번잡하게 만든다.
  - 가장 올바른 방법은 참조변수를 유효범위 밖으로 밀어내는 것

**p38, 객체 참조를 null 처리하는 일은 예외적인 경우여야한다**

**p38, 캐시 역시 메모리 누수를 일으키는 주범이다**

  - 캐시를 활용하기 위해 Map을 사용한다.
  - 캐시 외부에서 Key를 참조하는 동안만 엔트리가 필요하다면 WeakHashMap을 사용하자.
  - Map의 Key는 래퍼타입이 오는데 primitive의 레퍼타입은 null로 처리하더라도 JVM 내부에 캐싱이 되어있어 WeakHashMap을 사용하더라도 삭제되지 않는다.
 
**p38, WeakHashMap**

- 레퍼런스 종류

  - Strong Reference
    - new 연산을 통해서 객체를 생성하는 것
    - GC의 대상이 아니다
    - 참조변수에 명시적으로 null을 설정하면 GC의 대상이 된다

  - Soft Reference
    - 생성하기 위해서는 생성자에 Strong ref를 넣어주면 된다.
    - 예시
      
      ```java
      //SoftReference는 JAVA에서 지원한다.
      Object strongRef = new Object();
      SoftReference<Object> soft = new SoftReference<>(strongRef);

      strongRef = null;

      System.gc();
      System.out.println(soft.get())  //제거되지 않는다, 메모리 충분
      ```
    - Object를 가리키는 strongRef가 사라졌을 때 GC의 대상이 된다
    - 단 메모리가 부족한 상황에서만 실제로 제거된다

  - Weak Reference
    - Soft와 전체적으로 생성하는 과정이 비슷하다
    - 예시
      
      ```java
      //WeakReference는 JAVA에서 지원한다.
      Object strongRef = new Object();
      WeakReference<Object> weak = new WeakReference<>(strongRef);

      strongRef = null;

      System.gc(); 
      System.out.println(weak.get()) // Weak 이므로 제거된다.
      ```
  - Phantom Reference
    - 객체가 Phantom Reachable 한 경우 객체 제거 + 생성자에서 사용한 큐에 레퍼런스를 넣어준다
    - 이후 큐에서 꺼내 삭제 작업을 진행할 수 있다.
    - finalize를 대신하는 자원정리용도, 객체들이 메모리 해제가 되는지 파악할 수 있다.
- 더이상 사용하지 않는 객체를 GC할때 자동으로 삭제해주는 Map

**ScheduledThreadPoolExecutor**

- 쓰레드를 실행하는 방법은 다음과 같다.
  
  ```java
  CustomRunnable task = new CustomRunnable();
  Thread thread = new Thread(task);
  thread.start();
  ```
  - Runnable, Thread 문제점
    - 저수준 API에 의존한다
    - 쓰레드의 반환값을 얻을 수 없다.
    - 매번 쓰레드를 생성 및 종료하는 오버헤드가 크다.

- 쓰레드의 반환값을 얻기 위해 `Callable`이 등장했다.
  - 코드

    ```java
    @FunctionalInterface
    public interface Callable<V> {
        V call() throws Exception;
    }
    ```

- `Callable`의 비동기 실행을 돕기 위해 `Future`이 등장했다.
  - 코드

    ```java
    // hello는 Callable 객체이다.
    // Future에서 제공하는 isDone, get, cancel로 task를 통제할 수 있다.
    Future<String> helloFuture = executorService.submit(hello);
    ```
- 매번 쓰레드를 생성하는 부담을 덜기 위해 쓰레드풀 개념이 JAVA5 부터 도입되었다.
  - Executor 인터페이스
    - 쓰레드풀 구현을 위한 인터페이스
    - 작업의 실행을 책임진다
    - 코드
      <img width="698" alt="image" src="https://user-images.githubusercontent.com/67682840/236634450-91c85fb5-3dd9-4041-bdd6-eec63b0f4e23.png">
  - ExecutorSerivce 인터페이스
    - 작업의 등록을 책임진다.
    - Executor를 상속받았기 때문에 작업의 등록 및 실행을 책임지게 된다.
  - 쓰레드풀 생성 시 고려해야할 점
    - I/O
    - CPU
  
<br>

# 8장, finalizer와 cleaner 사용을 피하라

- 객체를 생성한 후 리소스를 정리하지 않은 채 객체를 소멸하면 문제가 발생할 수 있다.
- 위 대안으로 finalizer와 cleaner가 등장했으나 둘의 사용을 권하진 않는다.
  
## finalizer

- 사용하기 위해선 Object 클래스에 있는 `finalize` 메서드를 override 하면 된다.

  ![image](https://github.com/devbelly/dailycard-server/assets/67682840/57a9753f-715b-4e31-9c13-62456f190581)

- 객체에 대한 참조가 없다고 판단되면 가비지 컬렉터가 `finalize` 메서드를 호출한다

  ```java
  public class FinalizerIsBad {
    @Override
    protected void finalize() throws Throwable {
        System.out.println("");
    }

    public static void main(String[] args) {
        while(true){
            new FinalizerIsBad();
        }
    }
  }
  ```

- GC의 대상이 되면 Finalizer에서 관리하는 큐에 객체가 들어가게 된다.

  <img width="724" alt="image" src="https://github.com/devbelly/dailycard-server/assets/67682840/815c8ec3-69fe-4960-b513-e73ecf561c69">

- 리플렉션으로 큐의 크기를 확인해보면 참조되지 않는 객체들이 즉각적으로 제거되는 것이 아니라 쌓여있는 것을 확인할 수 있다.
  - 이 원인으로는 finalizer가 실행되는 쓰레드가 다른 쓰레드에 비해 우선순위가 낮기 때문이다.

- finalizer가 즉시 수행된다는 보장이 없다는 것을 확인할 수 있다.
  

## Cleaner

- finalizer의 대안으로 cleaner를 사용할 수 있다.
- 내부적으로 PhantomReference로 구현되어있다. 
- 코드
  
  ```java
  public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    // 청소가 필요한 자원. 절대 Room을 참조해서는 안 된다!
    private static class State implements Runnable {
        int numJunkPiles; // Number of junk piles in this room

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        // close 메서드나 cleaner가 호출한다.
        @Override public void run() {
            System.out.println("Cleaning room");
            numJunkPiles = 0;
        }
    }

    // 방의 상태. cleanable과 공유한다.
    private final State state;

    // cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override public void close() {
        cleanable.clean();
    }
  }
  ```

  - finalizer와 cleaner의 역할은 안전망 역할이다. Autoclosable을 구현했을지라도 클라이언트가 try-with-resource를 사용하지 않은 경우, gc될 수 있는 기회를 제공하기 위함이다.
    - 즉, 똑똑한 클라이언트가 try-with-resource를 사용했으면 close 메서드를 통해 clean이 호출되고
    - 멍청한 클라이언트가 구현을 깜박했다면 Room 생성시 cleaner에 register 했으므로 gc를 통해 정리될 기회를 갖게된다.
  - Runnable을 통해 자원 정리 방법을 정의한다.
  - `Cleaner.create()` 이후 `cleaner.register()`을 통해 GC시 할 Task를 알려준다
  - Room이 제거될 때 호출되므로 inner class 구현시 Room을 참조해선 안된다.

**p45, 정적이 아닌 중첩클래스는 자동으로 바깥 객체의 참조를 갖기 때문이다**

- 코드

  ```java
  public class OuterClass {
    class InnerClass {

    }

    public static void main(String[] args) {
        OuterClass outerClass = new OuterClass();
        InnerClass innerClass = outerClass.new InnerClass();

        System.out.println(innerClass);

        outerClass.printFiled();
    }

    private void printFiled() {
        Field[] declaredFields = InnerClass.class.getDeclaredFields();
        for(Field field : declaredFields) {
            System.out.println("field type:" + field.getType());
            System.out.println("field name:" + field.getName());
        }
    }
  }
  ```

  실행결과

  ```
  field type:OuterClass
  field name:this$0
  ```

  - inner 클래스에 어떠한 필드도 선언하지 않았지만 외부 클래스에 대한 참조가 있음을 알 수 있다.
  - OuterClass를 제거해야하는데 OuterClass를 참조하면 오류가 발생하므로 static을 사용해서 OuterClass에 대한 참조를 갖지 않도록 하자

<br>

- cleaner 역시 바로 실행된다는 보장이 없으므로 파일 닫기에 사용해선 안된다.
  - 시스템이 동시에 열 수 있는 파일 갯수에는 한계가 있기 때문이다.
  - 즉각적으로 닫지 않으면 예외가 발생할 수 있다.

- finalizer와 cleaner로 객체를 제거하면 AutoClosable을 사용한 것보다 더 느리다.

**p42, Finalizer 공격**

- 생성자나 직렬화 과정에서 예외가 발생하면 생성되다만 객체에서 finalizer가 수행될 수 있다.

  예시

  ```java
  public class Normal{
    private String name;
    public Normal(String name){
      if(name=="bad"){
        throw new IllegalArgumentException();
      }
    }
    public void func(){
      // something..
    }
  }
  ```

  ```java
  public class BadClass extends Normal{
    ...

    @Override
    protected void finalize() throws Throwable {
        this.func();
    }
  }
  ```

- BadClass 객체 생성 시 "bad"를 파라미터로 넘겨주면 예외가 발생할 것이다.
- 생성되다만 BadClass 객체는 gc시 finalize를 호출하는데, 이때 "bad"에게 허용되지 않는 `func()` 또한 호출한다.
- final 클래스가 아니라면 아무일도하지 않는 finalize final 메서드를 정의해서 오버라이딩을 막으면 된다.

<br>

# 9장, try-finally보다 try-with-resource를 사용하라

- 자바 라이브러리에는 close 메서드를 호출해 직접 닫아줘야 하는 지원이 많다
  - InputStream, OutputStream, Connection

- try-finally 방식에서 여러 자원을 사용한다면?

  <img width="515" alt="image" src="https://github.com/devbelly/TIL/assets/67682840/7d8eafd4-4c9f-4393-9960-0c8f523ea764">

  - 복잡하다
  - 테스트하기 어렵다
  - try와 finally 두 군데서 예외가 발생하면 예외 추적이 어렵다.
    - try-finally는 finally에서 발생한 예외만 보여주지만
    - try-with-resource는 두 예외 다 출력해준다

- 위 문제들은 try-with-finally을 통해 해결가능하다

  - 사용하기 위해서는 AutoClosable을 구현해야한다.
  - 코드

    ```java
    static void copy(String src, String dst) throws IOException{
      try(InputStream in = new FileInputStream(src);
          OutputStream out = new FileOutputStream(dst)){
          byte[] buf = new byte[128];
          int n;
          while((n=in.read(buf))>=0)
              out.write(buf,0,n);
      }
    }
    ```

  - 참고로 try-with-resource는 var를 사용하기에 적절하다

    ```java
    static void copy(String src, String dst) throws IOException{
        try(var in = new FileInputStream(src);
            var out = new FileOutputStream(dst)){
            byte[] buf = new byte[128];
            int n;
            while((n=in.read(buf))>=0)
                out.write(buf,0,n);
        }
    }
    ```

<br>

# 10장, equals는 일반 규약을 지켜 재정의하라

- 재정의하지 않아도 되는 경우는 다음과 같다
  - 각 인스턴스가 본질적으로 고유하다
    - 싱글톤 클래스, Enum 및 상태를 나타내지 않고 동작을 나타내는 클래스
  - 상위 클래스에서 재정의한 equals가 하위클래스에도 딱 맞을 때
    - List, Set..

- 반대로 재정의해야하는 경우는 위 상황과 정 반대라고 생각하면 된다
  - 논리적 동치성을 확인해야하고
  - 상태, 값을 나타내는 클래스이며
  - 상위 클래스에서 재정의 되지 않았을 때
  - @Entity

- 여러 클래스에서 equals는 동치관계를 만족한다고 가정하므로 아래 규약을 지켜야한다
  - 반사성
  - 대칭성
  - 추이성
  - 일관성
  - null-아님

- intellij 자동생성 코드

  ```java
  @Override
  public boolean equals(Object o) {
      if (this == o) return true;
      if (o == null || getClass() != o.getClass()) return false;
      Child child = (Child) o;
      return Objects.equals(id, child.id) && Objects.equals(member, child.member);
  }
  ```

  - `if(this==0) return true` : 반사성, 객체는 자기자신과 같아야한다
  - `if(o == null) return false` : null-아님을 만족한다

    - 메서드 오버라이딩을 위해 매개변수로 Object 클래스를 사용
    - 즉, 타입변환 전에 instanceof로 검사하는 과정이 필요, 이는 묵시적으로 null 검사를 해주므로 instanceof 코드를 활용하는 것이 적절하다

  - `getClass() != o.getClass()` : 리스코프 치환원칙을 위배하는 코드이다.

- 대칭성
  
  <img width="448" alt="image" src="https://github.com/devbelly/TIL/assets/67682840/6ece8263-e8c0-4c48-956a-670e34b7f633">

  - String을 설계할 때 당연히 CaseInsensitiveString을 고려해서 설계하지 않으므로 대칭성을 만족하지 않는 구현이다.

<br> 

# 11장, equals를 재정의하려거든 hashCode도 재정의하라

- equals를 재정의했다면 hashCode도 재정의해야한다
  - lombok에서 `@EqualsAndHashCode`를 지원한다.
- 규약은 다음과 같다.
  - equals가 같다고 판단한 두 객체는 hash code가 일치해야한다.
    - 만약 `Object.hashcode()`를 그대로 사용하면 다른 객체에 대해선 다른 해시값이 나온다.
    - 동치인 두 객체가 다른 해시값을 리턴하면 해시 기반의 컬렉션에 문제가 발생한다
  - equals가 다르다고 판단한 두 객체는 hash code가 일치해도 상관은 없으나 달라야한다
- 해시 값을 계산할 때 파생 필드를 제외한 핵심 필드만으로 해시값을 계산해야한다.
  - JPA에서 동치는 id값으로 계산한다.
  - 엔티티의 hashcode를 override한다면 어떤식으로 구현해야할까?

## 쓰레드 안정성

- lazy initialization을 한다면 쓰레드 안정성에 주의하자.

  ```java
  private int hashCode;

  @Override public int hashCode(){
    //logic
  }
  ```

- 방법1

  ```java
  private volatile int hashCode;

  @Override public int hashCode(){
    if(this.hashCode!=0){
      return hashCode;
    }
    //logic 공유자원과 관련없음
    synchronized(this){
      //logic 공유자원과 관련있음
    }
  }
  ```

  - critical section의 길이를 줄일 수 있다.
  - synchronized만 사용하면 다른 쓰레드가 캐시에 저장된 이상한 값을 읽어올 수 있다.
  - 이를 방지하기 위해 volatile 키워드 추가. 메인 메모리에서 값을 읽고 쓰는 것을 보장한다.

- 방법2, ThreadLocal 사용

  - 스프링에서는 트랜잭션 사용 시 ThreadLocal을 사용한다.(Connection 객체..)

- 방법3, hashCode를 사용하지 않고 클래스 모든 속성이 불변일 경우(final)

<br>

# 13장, clone 재정의는 주의해서 진행하라

- Cloneable은 복제해도 되는 클래스임을 명시하는 믹스인 인터페이스
- 실무에서 Cloneable을 구현한 클래스는 clone 메서드를 퍼블릭으로 제공한다
  - protected면 하위 클래스에서만 clone이 호출가능하므로 이상하다
- clone을 재정의 하는 곳에 생성자를 사용하면 안된다. 
  - `super.clone()`은 사용하는 위치에 따라 반환되는 객체의 타입이 다르기 때문이다.
  - 만약 생성자 방식으로 clone을 구현했다면 하위 클래스에서 문제가 발생할 것이다.
- 코드

  ```java
  @Override public PhoneNumber clone(){
    try{
      return (PhoneNumber) super.clone();
    } catch(CloneNotSupportedException e){
      throw new AssertionError();
    }
  }
  ```
- java는 오버라이딩하는 메서드에서 하위타입을 리턴해도 오버라이딩으로 인정한다.
  - ex) Object를 리턴하는 대신 PhoneNumber를 리턴하는 것도 인정해준다.  클라이언트에서는 clone이후에 형변환을 하지 않아도 된다

<br>

# 15장, 클래스와 멤버의 접근 권한을 최소화하라

- 캡슐화가 잘 된 컴포넌트가 설계가 잘 된 컴포넌트라고 할 수 있다.

> [!faq]- 캡슐화란 무엇일까?
> - 필드나 메서드들을 하나의 클래스로 묶어 외부로부터 접근을 제한하는 것
> - 핵심은 클래스 내부가 변경이 되더라도 클래스를 사용하는 쪽에서는 영향을 받지 않는 것
> - 이를 통해 클래스의 결합도를 낮출 수 있다.

**정보은닉의 장점**
  - 시스템 개발속도를 높인다. 여러 컴포넌트를 병렬적으로 개발할 수 있다.
    - 캡슐화를 위해서는 자연스럽게 인터페이스를 사용
    - 인터페이스를 사용하는 쪽 / 인터페이스를 구현하는 쪽
  - 시스템 관리비용을 낮춘다.
    - 인터페이스를 살펴보면 시스템 구성을 파악하기 용이하다
    - 인터페이스가 있다면 다른 컴포넌트로 교체하기도 쉽다.
  - 성능최적화에 도움을 준다
    - 성능 향상에 도움을 주지는 않는다.
    - 성능에 병목이 되는 모듈을 빠르게 파악할 수 있어 최적화에 도움을 준다.
  - 소프트웨어 재사용성을 높인다.
  - 개발 난이도를 낮춘다.

- 자바에서는 접근 제어자를 통해 정보은닉을 제공한다.
- 소프트웨어가 제대로 작동하는 범위에서 클래스 또는 멤버의 접근성을 가장 낮게 설정해야한다.

> p24, 한 클래스에서만 사용하는 package-private 톱레벨 클래스나 인터페이스는 이를 사용하는 클래스 안에 private static으로 중첩시켜보자.

> [!faq]- 왜 private class가 아닌 private static class일까?
> - private class는 outer class에 대한 참조를 가지고 있기 때문이다.
> - 참조를 가지고 있으므로 서로 독립적이지 않다.

- 위 내용보다 중요한 것은 톱 레벨 클래스가 public 으로 공개되지 않아야 하는 경우에는 package-private으로 관리하는 것

> [!note] p98, Serializable을 구현한 클래스는 필드들도 의도치 않게 공개 API가 될 수 있다.
> - 객체가 직렬화되면 클래스 및 필드 정보들이 바이트 스트림으로 변환되어 저장된다
> - 바이트 스트림은 클래스의 구조와 필드이름과 함께 저장되므로 바이트 스트림을 살펴보면 필드 내용도 파악할 수 있다.
> 

- protected로 접근 범위가 변경되는 순간 공개 API가 되어 내부 동작을 문서화 해야할 수도 있다.
- 접근 범위를 더 좁힐 수 없는 경우도 있다. 

> [!note] p98, 상위 클래스의 메서드를 재정의 할때는 그 접근 수준을 상위 클래스보다 좁게할 수 없다.
> - 부모 객체가 사용되는 곳에 자식 객체도 사용할 수 있어야한다.
> - 메서드 재정의 시 접근 범위를 좁게하면 부모 객체를 사용하는 쪽에서 자식 객체로 대체할 때 접근이 안된다

- public 클래스에서 public 필드 만들지 말자
	- 불변성을 지키기 어렵다. 제한된 범위내에서 필드를 다루기 어렵다
	- thread-safe 하지 않다.
	- 상수인 경우 `public static final`로 공개해도 좋다.

> [!note]  p100, 자바 9에서는 모듈 시스템이라는 개념이 도입되면서 두 가지 암묵적인 접근 수준이 추가되었다.
> - 모듈 시스템을 사용하려면 `module-info.java`에 서로 의존하고 있는 모듈을 정의하면 된다.
> - 모듈을 내보내는 쪽에선 export, 모듈을 사용하는 쪽에서는 require를 명시해야 모듈 내에 정의된 패키지에 접근할 수 있다.
> - `public`, `protected` 인 클래스가 있더라도 모듈 간 사용을 명시하지 않으면 외부에서도 접근이 불가능한 새로운 접근 수준이 발생한다.
> - 하지만 `모듈-모듈아님` 관계에서는 일반적인 `public`, `protected` 처럼 접근이 가능해지므로 주의하자


# 16장. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

- Bad case
	```java
	public class Point{
		public double x;
		public double y;
	}
	```

	- 필드에 직접 접근하면 불변식을 보장할 수 없다.
	- 부수 작업을 수행할 수 없다.

- Good case
	```java
	class Point{
		private double x;
		private double y;

		public void setX(double candX){
			if(candX>=100.0){
				throw IllegalArgumentException();
			}
			this.x = candX;
		}
		// setY, getX, getY 등
	}
	```
	- 부수 작업을 수행함으로써 불변식을 보장할 수 있다.

- public class에서 bad case처럼 작성하면 public 필드에 접근하는 클라이언트들이 생기게 된다.
- package-private class에서는 bad case처럼 작성해도 문제가 없다.

> [!note] p103. 내부를 노출한 Dimension 클래스의 심각한 성능문제는 오늘날까지도 해결되지 못했다.
> - 가변 객체를 사용하려면 프로그램의 동작 수행을 보장하기 위해 복사해서 사용한다
> - 만일 메서드 내에서 복사해서 사용하지 않는다면 `myLogic` 수행 시 `dim.witdh`값을 보장할 수 없다
> 	```java
> 	Dimension dim = new Dimension(10,20);
> 	methodA(dim);
> 	myLogic(dim.width);
> 	...
> 	```
> - 복사가 자주 일어나면 성능문제로 직결된다


# 17장. 변경가능성을 최소화하라

- 클래스를 불변으로 만드는 방법
	- 객체의 상태를 변경할 수 있는 메서드를 제공하지 않는다.
	- 클래스를 확장할 수 없도록 한다.
	- 모든 필드를 final로 선언한다.
	- 모든 필드를 private으로 선언한다.
	- 자신 외에는 내부의 가변 컴포넌트에 접근하지 않도록 한다.
		- 방어적 복사를 통해 제공 

> [!faq]- 클래스를 확장하면 어떤 문제가 발생할까?
> - 서브 클래스는 가변적인 객체로 설정할 수 있다
> - 부모 클래스가 불변인데 서브 클래스가 가변인 상황

> [!faq]- 클래스를 확장하지 못하게 하는 방법?
> - 클래스 final 선언
> - 생성자를 private, package-private으로 두고 public 메서드로 생성자 접근
> 	- 클라이언트 입장에선 불변 클래스와 다름이 없다.

> [!note] p105. 모든 필드를 final로 선언한다. 시스템이 강제하는~
> - 자바 언어 명세의 메모리 모델 부분에 잘 나와있다.
> 	- 컴퓨터구조
> - 메모리 모델이 허용하는 범위 내에서 프로그램 실행 순서는 구현체(JVM)의 자유
> - final 변수는 초기화가 이루어져야지 사용하는 쓰레드에서 필드에 접근을 하게 됨. 순서가 강제된다
> 	- 동영상

- 불변 객체의 장점
	- 쓰레드 안전하여 따로 동기화할 필요가 없다 & 불변 객체는 안심하고 공유할 수 있다.
		- 자주 사용되는 객체를 캐싱하여 제공할 수 있다.
		- 정적 팩터리 메서드를 열어두면 클라이언트 코드의 변경 없이 캐싱 기능을 추가할 수 있다.
	- 객체에 대한 검증이 단순해진다.
	- 불변 객체를 구성요소로 사용하면 이점이 많다.
		- Map, Set이 원하는 범위의 객체들을 담도록 보장할 수 있다. 즉 불변식을 유지할 수 있다
	- 불변 객체는 그 자체로 실패 원자성을 제공한다
		
> [!faq]- 실패 원자성?
> - 호출된 메서드가 실패하더라도 객체가 이전의 상태를 유지하는 것
> - 객체에서 예외가 발생하더라도 그 객체를 정상적으로 사용할 수 있다.
> - 예시
> ```java
> public BadObject method(BadObject object){
> 	object.x += 10;
> 	// 예외가 발생할 로직
> 	return object;
> }
> ```
> ```java
> public GoodObject method(GoodObject object){
> 	GoodObject ret = new GoodObject(object.x+10);
> 	//예외가 발생할 로직
> 	return ret;
> }
> ```
> - 불변 객체로 실패 원자성을 제공할 수도 있고, 가변 객체여도 입력 인자를 검사하는 로직을 추가하면 가능하다

- 불변 객체의 단점
	- 값이 일부만 달라져도 객체를 새로 생성해야한다.
	- `immu.plus().minus().()...` 으로 체이닝 하면 중간에 불변 객체를 계속 생성하게 된다.
	- 해결책
		- 다단계 연산을 지원하자. 이를 돕기 위해 가변 동반 클래스를 사용할 수도 있다

> [!note] p111. 모든 필드가 private final 이여야 한다는 규칙은 성능 개선을 위해 조금 완화 할 수 있다.
> - final은 객체 생성시 반드시 초기화 되어야하는 필드
> - lazy evaluation을 사용할 수 없으므로 final을 풀어서 완화할 수도 있다.
> ```java
> private int hashCode;
> ```
> - hashCode를 사용하는 시점에만 계산을 하고 이후에는 캐싱된 hashCode를 사용하자

# 18장. 상속보다는 컴포지션을 사용하라

- 다른 패키지의 구체 클래스를 상속하는 일은 위험하다.
	- 내부가 어떻게 구현되어있는지 파악하고 구현을 하게 되므로 캡슐화가 깨진다
	- 다른 패키지의 상위 클래스 구현이 바뀌면?

- 책 예시(pdf+)
	```java
	public boolean addAll(Collection<? extends E> c) {  
	    boolean modified = false;  
	    Iterator var3 = c.iterator();  
	  
	    while(var3.hasNext()) {  
	        E e = var3.next();  
	        if (this.add(e)) {  
	            modified = true;  
	        }  
	    }  
	  
	    return modified;  
	}
	```

> [!faq]- `this.add(e)`에서 오버라이딩한 add가 호출되는 이유?
> - dynamic dispatch
> - 정의
> >Single dispatch is a way to choose the implementation of a method based on the receiver runtime type
> - receiver인 this가 런타임에 가리키는 객체는 InstrumentedHashSet이므로 해당 클래스 내에서 구현한 메서드가 사용된다!

- 책에서 언급한 addCount가 제대로 기록되지 않는 문제 해결을 위해 `addAll`을 사용하지 않는 것도 내부 구현을 알고 있다는 가정
- 상위 클래스에 새로운 메서드가 추가된다.
	- 오버라이딩이 강제는 아니므로 알아차리기도 어려울뿐더러
	- 만일 보안적인 문제로 추가되는 원소에 제한을 둬야하는 상황이면 관련된 모든 메서드를 오버라이딩 해야한다.
- 결론은 상위 클래스의 변화에 취약하므로 컴포지션을 사용하자

# 19장. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

- 상속용 클래스는 재정의할 수 있는 메서드를 어떻게 이용하는지 문서화 해야한다.
	- 상속이 가지는 문제 때문에 내부 구현방식을 설명해야만 한다
	- `remove() & iterator()`
- 내부 구현에서 사용되는 hook을 `protected`로 공개해야할 수도 있다.
	- `removeRange()`
		- `List`의 최종사용자는 `removeRange()`에 관심이 없다
		- `AbstractList`를 상속하는 클래스를 만드는 클라이언트가 `clear()` 구현시 편리함을 제공하기 위해서
- 상위 클래스의 생성자에서 재정의 가능한 메서드를 호출해선 안된다.
	- 하위 클래스에서 메서드 오버라이딩 후 객체를 생성하면
		- 하위 클래스 생성자 호출 → 상위 클래스 생성자 호출 → 하위 클래스에서 재정의한 메서드 호출 → 상위 클래스 생성자 종료 → 하위 클래스 생성자 종료
		- 동작이 이상해진다.
- 상속의 복잡함
	- 특별한 이유가 없다면 상속을 금지해서 클라이언트가 잘못 사용하는 것을 막자
	- 상속을 허용한다면 클래스 내부에서 재정의 가능한 메서드를 사용하지 않고 이를 명시

# 20장. 추상 클래스보다 인터페이스를 우선하라

- 기존 클래스에 추가로 구현한 인터페이스를 끼워넣기 쉽다.
	- AutoCloseable, Comparable 등 기존 클래스에 해당 인터페이스를 구현
	- 추상 클래스라면 제약이 많이 따르게 된다.
- 믹스인 정의에 안성맞춤이다.
	- 추상 클래스라면 제약이 따르게 된다.
		- 믹스인을 추가하고 싶은 클래스가 다른 클래스를 상속하고 있는 경우

> [!faq]- 믹스인이란?
> - 기존 클래스의 기능에 부가적인 기능을 더하는 효과가 있다.

- 인터페이스를 조합해 계층 구조가 없는 클래스를 만들기 쉽다.
- 래퍼 클래스와 함께 사용하면 기능을 향상 시키기 좋다.
- 인터페이스에서 제공하는 메서드가 너무 명확하면 디폴드 메서드로 직접 제공할 수도 있다.
	- `removeIf()`

- 인터페이스의 한계
	- 필드를 활용한 디폴트 메서드를 제공하지 못한다.
	- private 멤버를 가질 수 없다(private 메서드는 예외이다.)
	- `equals()`, `hashCode()`, `toString()`을 디폴트로 제공할 수 없다.

- 위 단점을 극복하기 위해 추상 골격 구현 클래스와 인터페이스를 함께 사용할 수 있다.
	- 타입을 추상 클래스로 선언하는 데에서 오는 단점을 극복가능! (인터페이스 타입 선언)
	- 필드를 활용한 디폴트 메서드 제공가능 + private 멤버 문제 해결
	- 구현의 수고를 덜어준다.
		- `List` 를 오버라이딩 하면 수많은 메서드를 직접 구현해야한다
		- `List`를 상속하는 `AbstractList`를 사용하면 간단해진다.
	- 다중 상속을 시뮬레이션 할 수 있다.

# 21. 인터페이스는 구현하는 쪽을 생각해 설계하라

- 기존 인터페이스에 디폴트 메서드 구현을 추가하는 것은 위험하다.
	- 아파치의 SynchronizedCollection
		- `removeIf`를 구현하지 않았으므로 디폴트 메서드를 물려받는다.
		- 래퍼 클래스 역할을 하는 SynchronizedCollection은 `removeIf`에 락을 걸지 않았으므로 모든 메서드의 동기화를 보장하지 않게 된다.
- 런타임 오류가 발생할 수도 있다.
	- 예시
- 결론
	- 기존의 인터페이스에 디폴트 메서드를 추가하는 것은 심사숙고
	- 새로운 인터페이스에 디폴트 메서드를 추가하는 것은 클라이언트가 인터페이스를 더 잘 사용할 수 있도록 해준다.

# 22. 인터페이스는 타입을 정의하는 용도로만 사용하자

- 상수만을 선언하는 인터페이스 사용을 금지
- 상수 전용 클래스나 enum을 통해서 해결하자