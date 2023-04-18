# 자바 ORM 표준 JPA 프로그래밍 

<p align="center">
  <img src="https://camo.githubusercontent.com/3073829860a866037e0ca12f244a9eb5462e4cfa/687474703a2f2f696d6167652e6b796f626f626f6f6b2e636f2e6b722f696d616765732f626f6f6b2f786c617267652f3333302f78393738383936303737373333302e6a7067">
</p>

# 1장

## 애플리케이션에서 SQL을 직접 다루면 발생하는 문제

- DAO로 계층을 분리했더라도 실제로 어떤 SQL이 실행되는지 확인해야하므로 진정한 의미의 계층 분할이 어렵다.
- 이로 인해 SQL 의존적인 개발을 피하기 어렵다.

## 패러다임 불일치

- 어플리케이션이 복잡해지는 것을 대처할 수 있게 해주는 것은 객체지향 프로그래밍
- 하지만 데이터베이스에는 객체지향에서 다루는 추상화, 상속, 다형성 개념이 없음
  
## 상속

- 객체지향 프로그래밍에는 상속이 존재하지만 데이터베이스에는 상속이 존재하지 않는다.
- 그나마 슈퍼타입, 서브타입 개념을 도입하면 해결가능하다
- 객체를 저장하기 위해서는 `ITEM` 와 `ALBUM` 테이블 두 곳에 저장해야함
- 객체를 조회하기 위해서는 두 테이블을 JOIN 해야함
- 이렇듯, 패러다임 불일치로 인해 소비해야하는 코드량이 증가한다.

## JPA와 상속

- JPA에서는 상속과 관련된 패러다임 불일치를 해결해준다.
- `jpa.persist(album)`을 실행하면 알아서 `ITEM`와 `ALBUM`에 INSERT를 실행.
- `jpa.find(Album.class,albumId)`를 실행하면 알아서 `ITEM`과 `ALBUM`을 JOIN

## 연관관계

- 객체는 참조를 통해 다른 객체에 접근
- 데이터베이스는 외래키를 보관해여 JOIN을 통해 접근

## 객체를 테이블에 맞추어 모델링

- `MEMBER`와 `TEAM` 테이블의 컬럼에 따라 `Member`객체를 아래와 같이 만들었다
  ```java
  class Member{
    String id;
    String teamId;
    String name;
    ...
  }
  ```
- 객체를 조회, 저장할때는 문제가 안됨. 하지만 연관객체에 접근할때 참조가 없으므로 연관객체를 다룰때 문제가 발생한다.
  - 이는 객체지향 프로그래밍의 장점을 잃게 된다.
  
## 객체지향 모델링

- 객체는 참조를 통해 관계를 맺으므로 `Member` 클래스를 다음과 같이 수정해야한다.
  ```java
  class Member{
    String id;
    Team team;
    String name;
  }
  ```
- 객체를 조회, 저장하기가 까다롭다.
  - 객체는 연관객체에 대한 참조만 있으면 되지만
  - 데이터베이스는 외래키로 연관관계를 맺기 때문이다.
- 개발자는 이러한 불일치를 중간에서 해결해주어야한다.

## JPA와 연관관계

- JPA를 사용하기 이전, 다음 정보들을 불러온 후 `Team` 테이블에 저장한다.
  ```java
  member.getId();
  member.getTeam().getId();
  member.getName();
  ```
- JPA를 사용하면 연관관계를 설정한 후 저장만 하면 된다. 알아서 `team` 참조를 외래키로 변환하여 INSERT SQL을 작성해준다.
  ```java
  member.setTeam(team)
  jpa.persist(member)
  ```
- JPA를 사용하기 이전, 조회를 하려면 `MEMBER`와 `TEAM` 테이블을 JOIN해서 정보들을 확인하고 객체들을 생성한 다음 직접 연관관계를 설정해주어야 한다.
  ```java
  // SQL 실행
  Member member = new Member();
  Team team = new Team();
  // SQL에서 받은 정보들을 통해 객체에 값을 설정
  member.setTeam(team);
  ```
- JPA를 사용하면 조회시 외래키를 참조로 변환하는 일을 대신해준다.
  ```java
  String memberId = "1L";
  jpa.findById(Member.class,memberId);
  ```

## 객체 그래프 탐색

- 하나의 객체에서 참조를 통해 다른 객체에 접근하는 것을 객체 그래프 탐색이라고 한다.
- 객체는 마음껏 객체 그래프를 탐색할 수 있어야한다.
  - `member.getOrder().getOrderItem()`
- 하지만 실행되는 SQL에 따라 탐색할 수 있는 객체그래프의 범위가 정해진다. (어느 테이블을 JOIN하느냐에 따라 접근가능한 범위가 결정된다)
-  개발자는 어디까지 접근가능한지 알기 위해서는 결국 데이터 접근 계층인 DAO를 열어서 SQL을 확인해야하는 문제가 있다.
- `member`와 연관된 객체 그래프를 모두 메모리에 올리는 것은 비현실적이므로 결국 상황에 따라 적절한 SQL 메서드를 정의해야한다
  - `memberDao.getMember()`
  - `memberDao.getMemberWithTeam()`

## JPA와 객체 그래프 탐색

- JPA에서는 객체 그래프를 마음껏 탐색할 수 있다.
  - `member.getOrder().getOrderItem()`
- JPA는 연관객체를 사용하는 시점에 SELECT SQL을 실행한다. 이를 지연로딩이라함
- 설정에 따라 즉시로딩, 지연로딩을 설정할 수 있다.  

## 비교

- 데이터베이스는 PK를 기준으로 Row를 구분한다
- 객체의 비교는 동일성과 동등성이 있다.
  - 동일성은 `==` 연산자를 사용한다. 객체들의 주소값을 비교해 같은 인스턴스인지 판단
  - 동등성은 `.equals()` 메서드를 사용한다. 객체 내부적으로 가지고 있는 값이 동등한지 판단한다.
- JPA에서는 같은 트랜잭션 내에서 동일한 객체가 조회됨을 보장한다.

## JPA란 무엇인가?

- 자바 ORM 기술에 대한 API 표준 명세, 쉽게 말해서 인터페이스들을 모아놓은 것.
- ORM은 Object Relational Mapping의 약자, 객체와 데이터베이스 사이의 패러다임 불일치를 해결해준다.
  - 여러 ORM 프레임워크가 있지만 자바 진영에서는 하이버네이트를 사용한다.
  <img width="782" alt="image" src="https://user-images.githubusercontent.com/67682840/232647403-87fb54ad-d002-4114-abb8-2362cc139f2b.png">


## JPA를 사용해야하는 이유

- 생산성, 패러다임 불일치를 위해 매핑하는 코드를 직접 작성하지 않아도 된다.
- 유지보수, 하나의 필드가 추가되면 JDBC API를 사용한 모든 코드들 또한 수정해야하지만 JPA를 사용하면 그럴 필요가 업다.
- 패러다임 불일치 해결, 앞서 살펴봤던 패러다임 불일치 문제를 해결해준다.
- 성능 최적화, 애플리케이션과 데이터베이스 사이에 JPA가 위치하므로 성능 최적화의 기회가 있다.
- 벤더 독립성, 데이터베이스마다 페이징하는 방법이 달라 코드가 데이터베이스에 의존하게 되지만 JPA를 사용하면 사용하는 데이터베이스를 변경하겠다고 설정만 하면 된다. 
- 표준, JPA 기술 표준을 따르는 구현체를 손쉽게 변경할 수 있다.

<br>

# JPA 시작

## 객체 매핑 정보

- `@Entity`
- `@Table`
- `@Id`
- `@Column`
- 매핑정보가 없는 필드
  - 필드명을 기준으로 컬럼명과 매핑
  - 데이터베이스가 대소문자를 구분한다면 명시적으로 `@Column(name = "AGE")`

## Persistence.xml

- JPA는 persistence.xml을 통해서 실행에 필요한 정보들을 저장한다.
- `META-INF/persistence.xml`에 위치하면 추가적인 설정 없이 해당 파일을 인식할 수 있다.
- xml로 작성되어 있으며 아래와 같은 구조를 띤다
  - persistence
    - persistence-unit
      - properties
  
## 데이터베이스 방언

- JPA는 데이터베이스에 종속적이지 않은 기술
- 각 데이터베이스마다 문법이 조금씩 다르다. JPA에서는 데이터베이스 만의 특별한 문법을 방언이라고 한다.
- 하이버네이트를 포함한 여러 구현체들은 방언 클래스를 제공한다. 아래는 하이버네이트에서 제공하는 방언의 예시이다.
  - H2: org.hibernate.dialect.H2Dialect
  - 오라클 10g: org.hibernate.dialect.Oracle10gDialect
  <img width="601" alt="image" src="https://user-images.githubusercontent.com/67682840/232667963-d8741fb7-2c42-46bc-8881-1ea7c03c6d70.png">
  - `Dialect`를 살펴보면 추상 클래스로 되어있다.

## 애플리케이션 개발

```java
pulbic class JpaMain {
    
    public static void main(String[] args) {
    // [엔티티 매니저 팩토리] - 생성
    EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
    // [엔티티 매니저] - 생성
    EntityManger em = emf.createEntityManager();
    // [트랜잭션] - 획득
    EntityTransaction tx = em.getTransaction();

    try {
        tx.begin(); // [트랜잭션] - 시작
        logic(em) // 비즈니스 로직 실행
        tx.commit(); // [트랜잭션] - 커밋
    } catch (Exception e) {
        tx.rollback(); // [트랜잭션] - 롤백
    } finally {
        em.close(); // [엔티티 매니저 종료]
    }
    emf.close(); // [엔티티 매니저 팩토리 종료]-
    }
    
    //비지스 로직
    private static void logic (EntityManager em) {...}
}
```

- 엔티티 매니저 설정
- 트랜잭션 관리
- 비즈니스 로직

### 엔티티 매니저 설정

- 엔티티 매니저 팩토리 생성
  - `EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook")`
  - `META-INF/Persistence.xml`에 있는 `persistence-unit`의 정보를 읽는다.
  - JPA를 실행하기 위한 기반 객체를 만들고 데이터베이스에 따라 커넥션풀도 사용하므로 애플리케이션에서 딱 하나만 생성해야한다.
- 엔티티 매니저 생성
  - `EntityManager em = emf.createEntityManger()`
  - 내부적으로 데이터소스를 가지고 있다. 기본 CRUD 기능은 `EntityManager`를 통해 가능하다.
  - 데이터베이스와 밀접한 관련이 있으므로 쓰레드끼리 재사용해선 안된다.
- 종료
  - 사용이 끝났다면 종료해야한다.

### 트랜잭션 관리

- Transaction API를 받아와 실행한다.
  - `EntityTransaction tx = em.getTransaction()`
  ```java
  EntityTransaction tx = em.getTransaction();
  try {
      tx.begin(); // 트랜잭션 시작
      logic(em); // 비즈니스 로직 실행
      tx.commit(); // 트랜잭션 커밋
  }catch (Exception e) {
      tx.rollback(); // 예외발생시 트랜잭션 롤백
  }
  ```

### JPQL

- 단건조회, 수정, 삭제, 생성은 SQL을 사용하지 않고 `em`으로 가능하다.
- JPA는 객체지향적이므로 여러건 조회시 한 테이블의 모든 정보를 메모리에 올려서 사용해야하는데 이는 현실적으로 불가능하다.
- 필요한 정보만 불러오기 위해서 검색조건이 포함되어야하는데 JPA에서는 JPQL을 제공함으로써 이러한 문제를 해결한다
  ```java
  TypedQuery<Member> query = em.createQuery("select m from Member m",Member.class)
  ```
- SQL은 테이블 중심적으로 쿼리를 작성하지만 JPQL은 객체와 필드를 중심으로 쿼리가 작성된다.
  
<br>

# 영속성 관리

## 엔티티 매니저 팩토리와 엔티티 매니저

- `emf`는 생성 비용이 크다.
- `emf`는 Thread Safe 하므로 애플리케이션 내에서 공유를 해도 된다
- `em`은 생성 비용이 크지 않다
- `em`은 Thread Safe 하지 않으므로 쓰레드간 공유시 동시성 문제가 발생한다.
- `em`은 필요한 시점에 커넥션을 커넥션풀에서 얻어 사용한다.

## 영속성 컨텍스트

- JPA를 이해하는데 중요한 개념
- 엔티티를 영구적으로 저장하는 환경이다.
- 하나의 `em` 마다 하나의 영속성 컨텍스트가 생긴다고 생각하자
  - 여러 `em`이 하나의 영속성 컨텍스트를 보는 경우도 있다.

### 엔티티의 생명주기

- 비영속: 영속성 컨텍스트와 전혀 관련 없는 상태
- 영속: 영속성 컨텍스트가 관리하고 있는 상태
  - `em.persist(member)`
  - `em.find`
  - JPQL
- 준영속: 영속성 컨텍스트가 관리했다가 분리된 상태
- 삭제: 삭제된 상태

## 영속성 컨텍스트의 특징

### 엔티티 조회

- 영속성 컨텍스트에는 1차 캐시가 존재한다. 이는 Map과 같이 키는 식별자 값으로 되어있고 값은 객체를 저장하는 형태이다.
- 아래 코드를 실행해보자

  ```java
  Member member = new Member();
  member.setId("member1");
  member.setUsername("회원1");

  em.persist(member)
  ```
  
  실행 후 영속성 컨텍스트는 다음과 같은 그림이다. 데이터베이스에는 저장되지 않았다.

  <img width="533" alt="image" src="https://user-images.githubusercontent.com/67682840/232681007-18799600-5930-4856-b385-48e9d70e7994.png">

- 이 상태에서 조회를 해보자 `Member member = em.find(Member.class, "member1");`
  - 1차 캐시에 해당 id가 존재하는지 확인한다
  - 없다면 데이터베이스를 조회해서 엔티티를 생성한다.
  - 이후 1차 캐시에 해당 엔티티를 저장한 후 엔티티를 리턴한다
- 즉, 1차 캐시를 통해 성능상의 이점을 가질 수 있다.

### 동일성 보장

- 다음 코드의 실행 결과를 생각해보자

  ```java
  Member a = em.find(Member.class,"member1");
  Member b = em.find(Member.class,"member1");
  ```

- JPA는 엔티티 동일성을 보장하기 때문에 결과는 `true` 이다.
  - 앞에서 설명한대로 동등성은 객체가 가지고 있는 값이 일치하는 것을 의미한다.

### 엔티티 등록

