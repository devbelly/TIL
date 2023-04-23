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
  - FK를 관리하는 쪽, 연관관계의 주인인 객체에 `setTeam()`와 같이 연관관계를 설정하면 데이터베이스에 저장시 FK를 갖게 된다.
  - 그렇다면 주인이 아닌쪽은 `team.addmember()`와 같은 연관관계를 설정할 필요가 없는걸까?
    - 데이터베이스에는 영향을 미치지 않더라도 애플리케이션을 객체지향적으로 다루기 위해 논리적으로 연관관계를 설정해주는 것이 옳다.
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

# 3장, 영속성 관리

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

- 엔티티 매니저를 사용해서 엔티티를 영속성 컨텍스트에 등록해보자

  ```java
  EntityManager em = emf.createEntityManager();
  EntityTransaction tx = em.getTransaction();

  tx.begin();

  em.persist(memberA);
  em.persist(memberB);

  //커밋을 하는 시점에 데이터베이스에 INSERT SQL을 전송한다.
  tx.commit();
  ```

- JPA는 쓰기 지연 저장소를 통해 커밋 전까지 SQL을 전송하지 않는다.
- 위 코드를 실행하면 1차 캐시에 memberA, memberB가 저장되고 쓰기 지연 저장소에 쿼리가 저장된다

  <img width="655" alt="image" src="https://user-images.githubusercontent.com/67682840/232691156-3b627057-2509-4531-b6fd-0ce8966284ae.png">

- 이후 트랜잭션을 커밋하면 영속성 컨텍스트를 flush 한다

  ![image](https://user-images.githubusercontent.com/67682840/232696986-f413d506-b8c3-4162-8f97-25c72ecafb49.png)

### 엔티티 수정

- JPA는 변경 감지를 통해 엔티티를 수정한다.

  ```java
  EntityManager em = emf.createEntityManager();
  EntityTransaction tx = em.getTransaction();
  
  tx.begin();

  Member memberA = em.find(Member.class,"memberA");
  memberA.setUsername("hi");

  //em.update(memberA) 필요할까?

  tx.commit();
  ```

- 엔티티를 수정하기 위해선 단지 엔티티를 조회해서 데이터만 변경하면 된다.
- `em.update`가 필요할 것 같지만 필요하지 않다.

#### 변경감지

- 변경감지는 아래 순서
  - 영속성 컨텍스트는 영속 시점에 스냅샷을 저장한다
  - `flush()` 호출
  - 1차 캐시와 스냅샷을 비교하여 변경된 부분에 대해 UPDATE SQL을 생성
  - 생성한 SQL을 쓰기 지연 저장소에 저장
  - 데이터베이스에 SQL 전송

- 생성된 UPDATE SQL은 수정한 컬럼만 반영할까?

  ```sql
  UPDATE MEMBER -- 이런식으로 쿼리가 생성될까?
  SET
    NAME = ?
  WHERE
    id = ?
  ```

- JPA 기본전략은 수정하지 않은 모든 컬럼들을 업데이트 한다. 즉 수정 쿼리는 다음과 같이 생겼다.

  ```sql
  UPDATE MEMBER
  SET
    NAME = ?,
    AGE = ?,
    ...
  WHERE
    id = ?
  ```

- 모든 필드를 업데이트 하는 전략의 장점
  - 한 테이블에 대해 업데이트 쿼리 문장이 동일하므로 애플리케이션 로딩 시점에 미리 수정쿼리를 생성해 사용할 수 있다.
  - 데이터베이스에 동일한 쿼리를 보내면 데이터베이스는 이전에 파싱한 쿼리를 재사용할 수 있다.
    - chatgpt에선 데이터베이스는 최적화 기술로 쿼리 캐싱과 파싱된 쿼리 전략을 사용하는데 전달되는 파라미터가 달라지면 이러한 최적화 기능들이 제한된다고 설명한다
  
- 필요하다면 하이버네이트에서 제공하는 `@DynamicUpdate`를 사용하면 된다.
  - 테이블의 필드가 30개 이상 넘어가면 동적 쿼리가 성능상 이점을 제공할 수도 있다
  - 하지만 테이블 필드가 30개 이상 넘어가는 것은 설계상의 문제일 가능성도 있다.

### 엔티티 삭제

- 삭제를 위해선 엔티티를 조회한 후 삭제하면 된다.

  ```java
  Member memberA = em.find(Member.class,"memberA");
  em.remove(memberA)
  ```

- 업데이트 쿼리와 마찬가지로 삭제 쿼리를 쓰기 지연 저장소에 저장한 후 플러시 시점에 반영된다.

## 플러시

- 플러시는 영속성 컨텍스트의 변경내용을 데이터베이스에 반영하는 것. 실행 시 다음과 같은 일이 발생한다
  - 1차캐시와 스냅샷을 비교하여 UPDATE SQL을 생성한다
  - 쓰기 지연 저장소에 있는 SQL을 데이터베이스에 전송한다.

- 플러시는 아래 상황에서 발생한다
  - 강제로 `em.flush()` 호출
  - 트랜잭션 커밋 시
  - JPQL 실행시

- JPQL 실행 시 왜 `flush()`가 발생할까?

  ```java
  em.persist(memberA);
  em.persist(memberB);
  em.persist(memberC);

  query = em.createQuery("Select m from Member m",Member.class);
  List<Member> list = query.getResultList();
  ```

  - 객체 3개를 생성해서 persist하면 영속성 컨텍스트에는 존재하지만 데이터베이스에는 존재하지 않는다
  - 이때 JPQL을 실행하면 SQL로 바뀌어 데이터베이스에 질의를 날리게 되는데 데이터베이스에는 값이 없으므로 제대로 된 결과를 얻을 수 없다
  - 즉 JPQL 실행 전 자동으로 `em.flush()`를 호출한다

### 플러시모드 옵션

- 엔티티 매니저의 옵션으로 플러시모드를 직접 설정할 수 있다.
  - FlushMode.AUTO: 커밋을 하거나 쿼리 실행시 플러시가 발생한다
  - FlushMode.COMMIT: 커밋시 플러시가 발생한다
- 쿼리 실행시는 다음과 같다
  - JPQL
  - Criteria API
  - Native SQL
- 플러시를 하면 영속성 컨텍스트가 초기화되는 것이 아니라 데이터베이스에 동기화 된다는 점을 생각하자!

## 준영속

- 영속성 컨텍스트가 더는 관리하지 않는 상태는 준영속, 아래 상황에서 준영속이 된다.
  - `em.detach(memberA)`
  - `em.clear()`
  - `em.close()`

### em.DETACH

- 특정 엔티티와 연관된 1차 캐시와 쓰기 지연 저장소의 내용이 삭제된다.

  [before]
  <img width="542" alt="image" src="https://user-images.githubusercontent.com/67682840/232946113-fda77f11-1911-4ef4-9a20-30ab3c9b48f4.png">

  [after]
  <img width="514" alt="image" src="https://user-images.githubusercontent.com/67682840/232946205-3a64eae8-34a8-4c88-bac4-d18972d9c430.png">

### em.clear() & em.close()

- 영속성 컨텍스트가 관리하는 모든 1차 캐시와 쓰기 지연 저장소의 내용이 삭제된다.

  [after]
  <img width="566" alt="image" src="https://user-images.githubusercontent.com/67682840/232946465-7e5e73f2-c199-496a-921b-bf50dc0d00a4.png">

- 개발자가 직접 준영속으로 변경하는 경우는 드물다

### 준영속 상태의 특징

- 거의 비영속에 가깝다
- PK값이 있다
- 지연로딩을 사용할수 없다

### 병합: merge()

- 준영속 상태 엔티티를 영속으로 전환하기 위해서는 `merge()`를 사용하면 된다
- 실행결과는 ***새로운*** 영속상태의 엔티티를 반환한다.
- 아래 링크에서 persist와 merge의 차이에 대해 다루고 있다.
  - https://perfectacle.github.io/2021/06/13/entity-manager-persist-vs-merge/
  - 요약
    - JPQL 실행 시 managed entity의 연관객체에 대해 cascade가 발생한다
    - JPQL은 쓰기 지연 저장소에 자신과 관련이 있는 SQL만 flush한다
    - mother(영속상태)-children(비영속)인 상태에서 `motherRepository.save(mother)` 호출시 `.merge()`가 호출되고 연관객체까지 `.merge()`가 호출된다. `.merge()는 **새로운** 영속 상태 엔티티를 반환하고 컬렉션(mother.children)에 새로운 영속상태 엔티티를 대체하므로
    - 위 블로그에서 언급한대로 child의 레퍼런스가 달라진다.

  ```java
  static EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");

    public static void main(String[] args) {

        Member member = createMember("memberA","회원1");
        member.setUsername("회원명변경");
        mergeMember(member);
    }

    static Member createMember(String id,String username){
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx1 = em.getTransaction();

        tx1.begin();

        Member member = new Member();
        member.setId(id);
        member.setUsername(username);
        member.setAge(2);
        em.persist(member);

        tx1.commit();
        em.close();
        return member;
    }

    static void mergeMember(Member member){
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx2 = em.getTransaction();

        tx2.begin();

        Member mergedMember = em.merge(member);

        tx2.commit();

        System.out.println("준영속 " + member.getUsername());
        System.out.println("영속 " + mergedMember.getUsername());

        System.out.println(em.contains(member));
        System.out.println(em.contains(mergedMember));
    }
  ```

  [실행결과]
  ```
  준영속 회원명변경
  영속 회원명변경
  false
  true
  ```

  [그림]
  <img width="616" alt="image" src="https://user-images.githubusercontent.com/67682840/232951127-cf18cfc6-5321-4f9f-8a18-2178b84b0b96.png">

  1. merge(준영속) 호출
  2. 1차 캐시에 해당값이 있는지 조회, 없으므로 DB 조회 후 1차 캐시 + 스냅샷 저장
  3. 1차 캐시에 있는 값을 준영속 상태의 값으로 변경
  4. 반환
  5. 트랜잭션 종료 시 flush가 되며 스냅샷과 1차 캐시의 내용을 비교하여 SQL을 DB에 반영
   
- 새로운 엔티티를 반환하므로 기존 준영속 상태의 엔티티를 사용하지 않는다. 코드를 아래와 같이 변경
  
  `member = em.merge(member)`

#### 비영속 병합

- `merge()`는 비영속 엔티티도 영속 상태로 만들 수 있다.
- 즉, 병합은 준영속, 비영속을 신경쓰지 않는다.

<br>

# 4. 엔티티매핑

## @Entity

- 테이블과 매핑할 클래스는 @Entity 어노테이션을 필수로 붙여야 한다
- 기본생성자를 필수로 사용한다.
  - JPA에서는 객체를 생성할때 기본 생성자로 객체를 생성한다.
- final, enum, interface 클래스에는 사용할 수 없다

## @Table

- 엔티티와 매핑할 테이블에 대한 정보를 제공하기 위해 사용한다.
  - name: 테이블 이름을 지정한다. 생략 시 엔티티 이름을 사용한다.
  - uniqueConstraint: DDL 생성 시 유니크 제약조건을 건다. 스키마 자동생성시에만 사용된다.
## 다양한 매핑 사용

- 요구사항이 추가되었다.
  - 회원은 사용자와 어드민을 구분할 수 있어야 한다.
  - 회원가입일과 수정일이 있어야 한다.
  - 회원에 대한 설명란이 있어야 한다. 이 길이는 제한이 없다

  ```java
  @Enumerated(EnumType.STRING)
  private RoleType roleType;

  @Temporal
  private Date CreatedDate;

  @Temporal
  private Date UpdatedDate;

  @Lob
  private String description
  ```

  - enum 타입을 사용하기 위해서는 `@Enumerated`를 사용해야한다.
  - 날짜를 사용하기 위해서는 `@Temporal`을 사용해야한다.
  - `@Lob`을 사용하면 CLOB, BLOB 타입을 사용할 수 있따.

## 데이터베이스 스키마 자동생성

- jpa는 데이터베이스 스키마 자동생성을 지원한다
  - 매핑정보와 데이터베이스 방언을 확인하여 스키마를 생성한다.
  - `<property name = "hibernate.hbm2ddl.auto" value="create">`
    - 애플리케이션 실행시점에 자동으로 테이블을 생성한다.

    ```sql
    Hibernate: 
        drop table member if exists
    Hibernate: 
        create table member (
            id varchar(255) not null,
            age integer,
            created_date timestamp,
            description clob,
            last_modified_date timestamp,
            role_type varchar(255),
            name varchar(255),
            primary key (id)
        )
    ```

- `hibernate.hbm2ddl.auto`
  - create: DROP - CREATE
  - create-drop: DROP - CREATE - DROP
  - update: 테이블과 엔티티의 차이점을 확인하여 다른 부분이 있다면 업데이트를 한다
  - validate: 테이블과 엔티티의 차이점을 확인하여 다른 부분이 있다면 경고 메세지후 애플리케이션을 실행하지 않는다
  - none: 자동 생성 기능을 사용하지 않는다. 이는 유효하지 않은 값.

## 기본키 매핑

- 기본키를 직접 지정해줄 수도 있지만 데이터베이스가 제공하는 전략을 사용할 수도 있다.
  - 직접사용 : `@Id`만 사용한다
  - 자동생성 : `@Id` + `@GeneratedValue`
- 각 데이터베이스 벤더마다 제공하는 기본키 전략이 다르다.
  - Oracle : 데이터베이스 시퀀스
  - MySQL : Auto_Increment 제공

### IDENTITY

- 기본키 생성 전략을 데이터베이스에 위임한다.
- IDENTITY는 데이터를 데이터베이스에 INSERT한 후 기본키 값을 조회할 수 있다.
  - 이러한 이유로 추가적으로 데이터베이스를 조회해야한다
  - JDBC3에서 추가된 Statement.getGeneratedKeys()를 사용하면 데이터베이스에 저장과 동시에 값을 조회할 수 있다.
  - 하이버네이트는 위 메서드를 통해 데이터베이스에 한번만 연결한다.
- IDENTITY 식별자 생성 전략은 엔티티를 데이터베이스에 저장해야만 식별자를 구할 수 있다.
  - 따라서 `em.persist`호출 시 바로 INSERT 쿼리가 발생하며 쓰기 지연이 동작하지 않는다.
  - update에 대한 쓰기 지연은 동작하는 것 같다.

### SEQUENCE

- 시퀀스를 통해 기본키를 할당한다
- `em.persist()`시 데이터베이스에 시퀀스를 조회한 후 엔티티에 할당해 1차캐시에 저장한다
- 이후에 `.flush()`를 호출하면 데이터베이스에 쿼리가 날라간다.

  [실행결과]
  ```
  Hibernate: 
    create sequence hibernate_sequence start with 1 increment by 1
  //persist 전
  Hibernate: 
      call next value for hibernate_sequence
  //로직 종료
  Hibernate: 
      /* insert jpabook.start.Board
          */ insert 
          into
              board
              (description, id) 
          values
              (?, ?)
  //트랜잭션 종료
  ```

- 하나의 엔티티를 저장하기 위해 데이터베이스와 두번 통신해야한다
  - 최적화를 위해 allocationSize를 사용할 수 있다.
  - 50으로 설정해놓으면 1~50까지의 값은 메모리에서 시퀀스 값을 읽기 때문에 통신 비용이 발생하지 않는다.

### AUTO

- 다양한 기본키 생성 전략을 데이터베이스에 따라 자동으로 설정한다
  - Oracle : SEQUENCE
  - MySQL : IDENTITY
- 전략을 확정짓지 않은 개발초기에 사용하기 좋다.

### 키 선택 전략

- 키는 두가지 종류가 있다
  - 자연 키(natural key) : 비즈니스적으로 의미가 있는 키, ex) 전화번호, 주민등록번호 등
  - 대체 키(surrogate key) : 비즈니스적으로 아무 의미가 없는 키, ex) sequence, auto_increment, sequence table
- 자연 키를 사용하는 것 보다 대체 키 사용을 권장한다.

## 필드와 컬럼 매핑: 레퍼런스

### @Column

- 객체 필드와 테이블의 컬럼을 매핑할 때 사용한다.
- name : 객체 필드와 매핑할 테이블의 컬럼 이름을 지정한다.
- insertable, updatable: 객체를 테이블에 저장할 때 해당 필드는 저장, 업데이트 하지 않도록 한다. (false시, 실수방지용)
- table: 추후 소개
- nullable: ddl 시 not null 옵션 추가 (false시)
- unique: @Table의 uniqueConstraint와 동일, ddl시 alter unique ~ 추가
- columnDefinition: 지정한 값을 토대로 ddl 생성, ex) @Column(columnDefinition = "varchar(100) default 'EMPTY'")
- length
- precision, scale

### @Enumerated

- Enum 타입을 컬럼으로 사용하기 위해 사용한다.
 - EnumType.ORDINAL
  - 순서에 따라 숫자를 컬럼에 저장한다.
  - 저장되는 크기가 작다는 장점이 있지만 Enum 클래스 중간에 다른 값이 추가되면 문제가 발생한다.
 - EnumType.STRING
  - 이름을 기준으로 컬럼에 저장한다.
  - 저장되는 크기가 ORDINAL보다 크지만 Enum 클래스 중간에 다른 값이 추가되어도 문제가 발생하지 않는다
  - STRING 사용을 권장한다.

### @Transient

- 일시적인이라는 뜻
- 이 필드는 매핑하지 않는다, 객체에 일시적으로 값을 저장하고 데이터베이스에는 저장하지 않을 때 사용

<br> 

# 5장, 연관관계 기초 매핑

- ORM은 객체의 참조와 데이터베이스의 외래키의 매핑을 지원하는 것이 핵심 목적이다

### 객체 관계 매핑

- 회원과 팀이 다대일 관계로 맺어진다고 가정하자

  ```java
  public class Member{
    @Id @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    ...
  }
  ```

  ```java
  public class Team{
    @Id
    @Column(name = "TEAM_ID")
    private String id;

    private String name;
  }
  ```

### @JoinColumn

- 외래키와 매핑하기 위해 사용한다. 사용할 수 있는 속성은 다음과 같다.
  - name : 매핑할 외래키의 이름
  - referencedColumName: 외래키가 참조하는 대상 테이블의 컬럼명
    - `@JoinColumn(name = "TEAM_ID", referencedColumnName = "id")` 지정 시 id는 Team의 id를 가리킨다.

### @ManyToOne

- 다대일 관계를 나타낸다. 사용할 수 있는 속성은 다음과 같다.
  - optional: false로 설정하면 연관 엔티티가 항상 있어야 한다.
  - fetch: 글로벌 페치 전략을 설정한다.
  - cascade: 영속성 전이 기능을 사용한다.
  - targetEntity: 연관 엔티티의 타입정보를 설정한다. 제네릭의 타입추론으로 거의 사용하지 않는다.
    ```java
    @OneToMany
    private List<Member> members;

    @OneToMany(targetEntity = Member.class)
    private List members;
    ```

> @JoinColumn nullable vs @ManyToOne optional
>
> JoinColum이 nullable true이고 ManyToOne의 optional이 false라면 논리적으로 맞지 않아 왜 두가지 옵션이 존재하는지 잘 이해가 되지 않았다. 이를 이해하려면 JPA가 존재하는 이유에 대해서 다시 생각해보면 된다. JPA는 데이터베이스와 객체지향 사이의 차이를 매핑하기 위해 존재하는 기술이다. JoinColumn 의 nullable은 실제 데이터베이스의 컬럼에 null 값을 허용하는지 여부를 판단하는 물리적인 개념이라면 ManyToOne의 optional은 객체지향의 관점에서 연관 엔티티가 존재해야하는지를 가리킨다. 이는 논리적인 개념이다.

## 연관관계 사용

### 저장

```java
Team team = new Team("team1","팀1");
em.persist(team);

Member member1 = new Member("member1","회원1");
member1.setTeam(team);
em.persist(member1);
```

- 엔티티를 저장할때 연관된 모든 엔티티는 영속 상태여야한다.
- 코드에서 중요한 부분은 `member1.setTeam(team)`이다. 이 코드 덕분에 직접 team_id를 조회해서 저장하지 않아도 된다.

[실행된 쿼리]
```
insert 
        into
            member
            (team_id, username, member_id) 
        values
            ("team1", "회원1", "member1")
```

- 이를 통해 외래키를 가지고 있는 엔티티(여기서는 Member)는 연관관계 설정을 꼭 해야하는 것을 알 수 있다.

> 양방향 연관관계
>
> 외래키를 가지고 있는 엔티티는 연관관계 설정을 하면 데이터베이스에 자동으로 참조하는 엔티티의 id값을 확인하여 SQL을 생성해주는 것을 확인했다. 위 예제에서 Team 또한 @OneToMany로 양방향 관계를 상황이라면 연관관계를 설정해야할까? 이에 대한 대답은 JoinColumn의 nullable, ManyToOne의 optional과 비슷하다. 데이터베이스에 not null 제약조건을 위해 nullable을 설정하고 논리적인 관계를 위해 optional을 설정한 것처럼 물리적으로 FK를 관리하는 엔티티인 member에 `setTeam()` 메서드를 사용했다면 논리적인 관점, 즉 객체지향적인 관점을 위해서 @OneToMany에 `.addMember()` 메서드를 사용해 연관관계를 설정해주는 것이 옳다.

### 조회

- 객체를 통해 연관 엔티티를 조회, 객체 그래프 탐색
- JPQL 사용

### 수정

- 영속성 상태인 엔티티에 대해 값을 수정하면 트랜잭션 종료시 자동으로 update가 실행된다

### 삭제

- 연관된 엔티티를 삭제하기 위해서는 연관관계부터 제거해야한다. 만약 제거하지 않는다면 외래키 제약조건에 의해 오류가 발생한다. 

```java
public static void logic(EntityManager em) {
    Team team = new Team("team1","팀1");
    em.persist(team);

    Member member1 = new Member("member1","회원1");
    member1.setTeam(team);
    em.persist(member1);

    em.flush();

    em.remove(team);
}
```

[실행결과]

![image](https://user-images.githubusercontent.com/67682840/233524109-1f4d6dac-2e8c-4726-b568-65a03186d8c0.png)

올바른 결과를 위해선 `emflush()`와 `em.remove(team)` 사이에 `member1.setTeam(null)`을 추가하면 된다.

```java
public static void logic(EntityManager em) {
    Team team = new Team("team1","팀1");
    em.persist(team);

    Member member1 = new Member("member1","회원1");
    member1.setTeam(team);
    em.persist(member1);

    em.flush();
    member1.setTeam(null);
    em.remove(team);
}
```

[실행결과]

<img width="433" alt="image" src="https://user-images.githubusercontent.com/67682840/233524392-ceaf0bdb-88db-41ee-887d-0f40ed11ed39.png">

<br>

> 연관관계를 맺고 있는 두 엔티티가 있을 때, @JoinColumn의 nullable이 false인 경우는 없는걸까?
>
> 연관 엔티티를 삭제하기 위해서는 `setTeam(null)`와 같이 연관관계를 우선 끊어내야한다. team을 null로 설정하기 위해서는 FK가 nullable 해야한다. 그렇다면 연관 엔티티를 삭제하는 경우가 있다면 무조건 FK는 nullable해야하는걸까? <br>
> chatGPT에게 아래와 같은 답변을 받았다. 요약하자면 부모 엔티티가 삭제될 때 자식 엔티티를 모두 삭제하고 싶은 경우에 사용한다고 한다.

<img width="742" alt="image" src="https://user-images.githubusercontent.com/67682840/233526222-a12dc553-a5b0-4db1-9f71-0441e6fc7e76.png">

## 양방향 연관관계

- Team과 Member 엔티티를 양방향 연관관계로 맺기 위해 Team에 `@OneToMany`를 추가해보자.
- JPA는 List, Map, Set과 같은 여러 컬렉션을 지원한다.
- 데이터베이스는 외래키 하나로 양방향 연관관계가 성립하므로 `@OneToMany`를 추가했다고 해서 데이터베이스에 변화가 일어나지 않는다.

  ```java
  @OneToMany(mappedBy = "team")
  private List<Member> members = new ArrayList<>();
  ```

## 연관관계의 주인

- 단방향, ManyToOne만 사용했을때는 객체의 연관관계와 데이터베이스의 연관관계가 하나씩 있으므로 차이가 없었다.
- 양방향 연관관계, 정확히는 두 개의 단방향 연관관계를 사용한다면 객체의 참조는 두 곳(Team.members, Member.team)이지만 데이터베이스 연관관계는 하나이므로 차이가 발생한다.
- mappedBy로 이러한 차이를 해결하지 않으면 아래 사진처럼 FK 관리를 위해 테이블이 하나 더 생성된다.
  
  [mappedBy 미사용]

  <img width="201" alt="image" src="https://user-images.githubusercontent.com/67682840/233541226-1ad6b24d-f752-45fa-94fc-e626f4cc7af6.png">

  [mappedBy 사용]

  <img width="199" alt="image" src="https://user-images.githubusercontent.com/67682840/233541750-ff4489d3-4f6d-42fb-a162-2779e931cea7.png">

- 연관관계의 주인은 FK를 가지고 있는 엔티티
- 주인이 아닌쪽은 `mappedBy`를 통해 자신의 연관관계가 다른 연관관계에 의해 매핑되고 있음을 알려준다.
- 요약하자면 FK를 가지고 있는 객체의 연관관계만 데이터베이스 연관관계와 매핑될 수 있다!

## 양방향 연관관계 저장

- 양방향 연관관계 저장의 코드는 아래와 같다. 이는 단방향 연관관계 저장 코드와 완벽하게 일치한다.

  ```java
  Team team1 = new Team("team1","팀1");
  em.persist(team1);

  Member member1 = new Member("member1","회원1");
  member1.setTeam(team1);
  Member member2 = new Member("member2","회원2");
  member2.setTeam(team1);

  em.persist(member1);
  em.persist(member2);
  ```

- `team1.getMembers().add(member1)`와 같은 코드가 필요할 것 같지만 연관관계의 주인이 아니므로 JPA는 이를 무시한다.

## 양방향 연관관계의 주의점

- 위에서도 언급했지만 연관관계의 주인이 아닌곳에는 값을 매핑하고 연관관계의 주인인 곳에는 값을 매핑하지 않는 오류를 조심해야한다.
- 이에 대한 문제는 https://www.youtube.com/watch?v=brE0tYOV9jQ 에서도 다루니 궁금하면 살펴보자

### 순수한 객체까지 고려한 양방향 연관관계

- JPA는 ORM, 객체와 데이터베이스를 매핑해주기 위한 기술이다.
- 앞서 살펴봤듯이, `setTeam()`을 통해 연관관계를 설정하면 데이터베이스에는 문제가 없다
- 하지만 JPA는 데이터베이스 뿐만 아니라 객체지향도 고려하므로 `Team.getMembers().add(member1)`와 같은 코드를 추가해주는 것이 좋다.

## 정리

- 단방향 연관관계만으로 객체와 데이터베이스의 연관관계 매핑은 완료되었다.
- 양방향 연관관계 설정 시, 반대방향에서 객체 그래프 탐색 기능이 추가된것이다.
- 양뱡향 연관관계는 객체에서 양쪽 방향을 모두 관리해야한다
- 양방향 연관관계는 까다롭다. 단방향으로 설계하고 필요하면 양방향을 추가하도록 하자.

<br> 

# 6장, 다양한 연관관계 매핑

- 다중성에 대해 언급할 때 왼쪽에 있는 것이 연관관계의 주인이라고 가정하겠다.

## 다대일

### 다대일 단방향 관계

## 일대다

- 하나의 팀은 여러 회원을 참조하지만 회원은 팀을 참조를 하지 않는다.
- 그림은 아래와 같다.

  <img width="479" alt="image" src="https://user-images.githubusercontent.com/67682840/233627975-fe8e0f39-b50b-4ee6-97da-75b53e659045.png">

  - 일반적으로 자신과 매핑하는 테이블의 외래키를 관리하지만 일대다에서는 반대 테이블의 외래키를 관리하는 모습을 보인다
  - 이러한 이유는 데이터베이스는 일대다 관계에서 "다"쪽에 항상 외래키를 저장하기 때문이다
  - 하지만 "다"쪽 엔티티에 해당하는 멤버에 외래키와 매핑할 수 있는 참조필드가 없다.
  - "일"쪽에 members가 존재하므로 Team 엔티티에서 반대편 테이블의 외래키를 관리하는 형태이다

  ```java
  @Entity
  public class Team {
      @Id @GeneratedValue
      @Column(name="TEAM_ID")
      private Long id;

      private String name;

      @OneToMany
      @JoinColumn(name = "TEAM_ID")
      private List<Member> members = new ArrayList<>();

      public Team(){}
      public Team(String name){
          this.name = name;
      }

      public List<Member> getMembers(){
          return members;
      }
  }
  ```

### 일대다 단방향 매핑의 단점

- 매핑한 객체가 관리하는 외래키가 다른 테이블에 있다는 문제
- 매핑한 객체가 관리하는 외래키가 같은 테이블에 있다면 엔티티의 저장과 연관관계 관리를 insert 한번에 가능
- 다른 테이블에 있으므로 추가적인 update 쿼리가 발생한다.

  ```java
  public static void logic(EntityManager em) {
    Member member1 = new Member("member1");
    Member member2 = new Member("member2");

    Team teamA = new Team("team1");
    teamA.getMembers().add(member1);
    teamA.getMembers().add(member2);

    em.persist(member1);
    em.persist(member2);
    em.persist(teamA);
  }
  ```

  - member를 persist할 때 Team의 존재를 몰라 insert만 발생하지만
  - team을 persist할때 비로소 연관관계의 존재를 알게 되므로 추가적으로 update 쿼리가 발생하게 되는 것!

### 일대다 양방향

- 다대일 양방향 관계를 적극 사용하자...

## 일대일

- 일대일에서는 주 테이블에서 외래키를 관리할 수도 있고 참조 테이블에서 관리할 수도 있다.
  
### 주 테이블에 외래키

- 주 객체가 대상 객체를 참조하는 것처럼 주 테이블에 외래키를 두고 대상 테이블을 참조한다.
- 객체지향 개발자들이 선호한다.
- JPA도 주 테이블에 외래키를 두면 좀더 편리하게 매핑할 수 있다.

### 대상 테이블에 외래키

- 데이터베이스 개발자들이 선호하는 방식이다
- JPA는 일대일 단방향 관계에서 대상 테이블에 외래키를 두는 것을 허용하지 않는다.
  
## 다대다

- 데이터베이스는 다대다 관계를 중간에 관계테이블을 둬서 일대다, 다대일 관계로 풀어간다
  
  <img width="494" alt="image" src="https://user-images.githubusercontent.com/67682840/233756465-e70f3c96-4059-4e2b-8903-9cc9da363aee.png">

- 반면에 객체는 다대다 관계를 별도의 중간 객체 없이 바로 풀어나갈 수 있다. 양쪽 다 컬렉션을 사용하면 되기 때문이다

### 다대다 단방향

- 예시
  
  ```java
  @Entity
  public class Member {

      @Id
      @GeneratedValue
      @Column(name = "MEMBER_ID")
      private Long id;

      private String username;

      @ManyToMany
      @JoinTable(
              name = "MEMBER_PRODUCT",
              joinColumns = @JoinColumn(name = "MEMBER_ID"),
              inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID")
      )
      private List<Product> products = new ArrayList<Product>();

      ...
  }
  ```

  ```java
  @Entity
  public class Product {
      @Id @GeneratedValue
      @Column(name = "PRODUCT_ID")
      private Long id;

      private String name;
  }
  ```

- `@JoinTable`
  - name : 관계 테이블의 이름을 지정한다.
  - joinColumns : 현재 방향인 멤버와 매핑할 조인 컬럼 정보를 지정한다.
  - inverseJoinColumns : 반대 방향인 상품과 매핑할 조인 컬럼 정보를 지정한다.

- 중간 엔티티를 직접 작성하지 않아도 자동으로 생성됨을 알 수 있다.

  <img width="197" alt="image" src="https://user-images.githubusercontent.com/67682840/233757212-d9ef994d-b7c2-4f49-8621-c84e4fbb155e.png">

- 저장하는 코드는 다음과 같다.

  ```java
  public static void logic(EntityManager em) {
        Product product = new Product();
        em.persist(product);

        Member member1 = new Member("user1");
        member1.getProducts().add(product); // 연관관계 설정
        em.persist(member1);
    }
  ```

  - Product, Member, Member_Product 테이블에 INSERT 쿼리가 한번씩 발생함을 알 수 있다.

### 다대다: 매핑의 한계와 극복, 연결 엔티티 사용

- `@JoinTable`은 간결하지만 추가적인 필드를 사용하지 못하는 단점이 있다.
- 예를 들어 회원이 주문을 하면 추가적으로 주문한 날짜와 주문 수량에 대해서도 파악해야한다.
  - `JoinTable`을 사용하면 위 정보를 회원과 상품 엔티티에 매핑할 수 없다
- 이러한 이유로 다대다 관계를 표현할 수 있는 객체라 할지라도 일대다, 다대일 관계로 풀어야하고 이를 위해 연결 엔티티를 사용해야한다.
- 연관 엔티티는 다음과 같다
  
  ```java
  @Entity
  @IdClass(MemberProductId.class)
  public class MemberProduct{
    @Id
    @JoinColumn(name = "MEMBER_ID")
    @ManyToOne
    private Member member;

    @Id
    @JoinColumn(name = "PRODUCT_ID")
    @ManyToOne
    private Product product;

    private int amount;
    ...
  }
  ```

  - `@Id`와 `@JoinColumn`을 통해 부모 테이블의 기본키를 외래키로 사용하면서 자신의 기본키로 사용하고 있다.
  - 기본키로 MEMBER_ID와 PRODUCT_ID를 사용하고 있다. 이를 복합 기본키라고 부른다.
  - 복합 기본키를 사용하기 위해서는 식별자 클래스(MemberProductId.class)를 사용해야한다.

  ```java
  public class MemberProductId implements Serializable{
    ...
  }
  ```
  
> 식별관계
>
> 부모 테이블의 기본키를 통해 자신의 기본키와 외래키로 사용하는 모습을 보았다. 데이터베이스에서는 이를 식별관계라고 한다.

### 다대다: 새로운 기본 키 사용

- 복합키를 사용하는 대신 데이터베이스에서 제공하는 기본키를 사용하자.
- 비즈니스에 의존적이지도 않으며 복잡하게 복합키를 위한 테이블을 사용할 필요도 없다.
- MEMBER_PRODUCT 라는 이름보다는 ORDER 이라는 테이블 명이 더 어울린다
  - ORDER은 일부 데이터베이스의 예약어로 잡혀있으므로 ORDERS를 사용하기도 한다.

<br>

# 7장, 고급매핑

## 상속 관계 매핑

- 관계형 데이터베이스에는 객체에서 말하는 상속이라는 개념이 없지만 슈퍼타입, 서브타입이라는 모델링 기법이 존재
- ORM에서 말하는 상속 관계 객체의 상속과 데이터베이스의 슈퍼타입, 서브타입 관계를 매핑하는 것
- 슈퍼타입, 서브타입의 논리 관계를 실제 테이블로 풀어나갈 때는 세가지 전략이 존재한다.

### 조인전략

- 부모와 자식 클래스를 모두 테이블로 만든다
  
  <img width="401" alt="image" src="https://user-images.githubusercontent.com/67682840/233769759-d2333f92-29e0-4050-b097-57643e4d329d.png">

- 객체는 타입으로 구분할 수 있지만 데이터베이스는 타입이라는 개념이 없다. 구분을 위해 부모 테이블에 `DTYPE` 컬럼이 추가됨을 알 수 있다.
- 자식 테이블은 부모의 기본키를 외래키 + 기본키로 사용한다. 그림에서 ITEM_ID를 사용하는 모습을 볼 수 있다.
- 테이블을 여러개 만드므로 조회시 조인을 많이 사용한다. 이 때문에 조인전략이라고 부른다.
- 코드

  ```java
  @Entity
  @Inheritance(strategy = InheritanceType.JOINED)
  @DiscriminatorColumn(name = "DTYPE")
  public abstract class Item {
      @Id @GeneratedValue
      @Column(name = "ITEM_ID")
      private Long id;

      private String name;
      private int price;
  }

  @Entity
  @DiscriminatorValue("M")
  public class Movie extends Item{
      private String director;
      private String actor;
  }
  ```

  - `@Inheritance` : 상속 매핑시 부모 테이블에 `@Inheritance` 어노테이션을 사용해야한다.
  - `@DiscriminatorColumn` : 부모테이블에서 자식 테이블을 구분하기 위해 사용
  - `@DiscriminatorValue` : 위 컬럼에 들어갈 값을 지정

- 장점
  - 테이블이 정규화된다.

    정규화는 중복된 데이터를 제거하고 일관성과 무결성을 확보하는 것이다. 상속구조를 통해 효율적으로 저장하였으므로 정규화를 이뤘다고 볼 수 있다.
  
  - 저장공간의 효율

- 단점
  
  - 조회시 조인이 많이 사용되므로 성능저하문제가 발생할 수 있다.
  - 쿼리가 복잡하다
  - 데이터 저장 시 INSERT가 두 번 실행된다.

### 단일 테이블 전략

<img width="153" alt="image" src="https://user-images.githubusercontent.com/67682840/233818708-45a5138a-4bb0-49ec-b950-58c0621abaa4.png">

- 각 자식 엔티티 마다 클래스를 만드는 것이 아니라 하나의 거대한 테이블을 생성한다.
- 장점
  - JOIN을 사용하지 않아 성능이 뛰어나다
  - 쿼리가 단순하다
- 단점
  - 자식 엔티티가 사용하용하는 필드는 nullable 해야한다.
  - 단일 테이블에 모두 저장하므로 테이블이 커질 수 있다. 조회 성능이 오히려 느려질 수 있다.

    <img width="626" alt="image" src="https://user-images.githubusercontent.com/67682840/233818829-c3f7df77-664b-434e-8782-824c251e6d8b.png">

### 구현 클래스마다 테이블 전략

<img width="406" alt="image" src="https://user-images.githubusercontent.com/67682840/233818930-4ca6864a-c977-44fb-88cd-efcdd592e7e5.png">

- 장점
  - not null 제약조건을 사용할 수 있다.
  - 서브 타입을 구분해서 처리할 때 효과적이다.
- 단점
  - 여러 자식 테이블을 함께 조회할 때 성능이 느리다.
  - 자식 테이블을 통합적으로 쿼리를 날리기 어렵다.

- 데이터베이스 설계자와 ORM 전문가 둘다 추천하지 않는 전략이다.

> @MappedSuperclass vs @Inheritance

- MappedSuperclass는 base class에 선언된 속성이 마치 자식 클래스에 구현된 것처럼 하게 해주는 어노테이션이다.
- 이는 실제 데이터베이스에는 나타나지 않는, 객체에만 보이는 상속이 된다.
- @Inheritance는 OOP 세계에 존재하는 상속을 데이터베이스 테이블 구조로 실체화해준다.
  - 이를 통해 Strategy Pattern을 구현할 수 있다.
- @MappedSuperClass는 그마저도 @Embeddable로 대체할 수 있으며 차이점은 @Embeddable은 @Id를 포함할 수 없다는 것이다.
- https://stackoverflow.com/questions/9667703/jpa-implementing-model-hierarchy-mappedsuperclass-vs-inheritance

## @MappedSuperclass

- 위에서 살펴본 Inheritance는 실제 테이블과 매핑되지만(모두는 아님) MappedSuperClass는 매핑 정보를 상속하기 위해 사용한다
  
  <img width="526" alt="image" src="https://user-images.githubusercontent.com/67682840/233820964-d7fa0822-ad9d-4b22-8588-92bcaad058f2.png">

  ```java
  @MappedSuperclass
  public class BaseEntity {
      @Id @GeneratedValue
      private Long id;
      private String name;
  }
  @Entity
  public class Member extends BaseEntity{
      private String email;
  }
  ```

- 부모로부터 물려받은 매핑정보를 재정의 하기 위해선 `@AttritubeOveride`를 사용하면 된다.

  ```java
  @Entity
  @AttributeOverride(name = "id",column = @Column(name = "MEMBER_ID"))
  public class Member extends BaseEntity{
    private String email;
  }
  ```

- 둘 이상을 재정의 하려면 `@AttributeOverrides`를 사용하면 된다

## 복합키와 식별관계 매핑

- 데이터베이스 테이블의 관계는 외래키가 기본키에 포함되는지 여부에 따라 식별관계, 비식별관계로 나눌 수 있다.
  - 식별관계 : 부모테이블의 기본키를 자식 테이블의 기본키, 외래키로 사용
  - 비식별관계 : 부모테이블의 기본키를 자식 테이블의 외래키로만 사용
    - 필수적 비식별 관계 : 외래키에 null을 허용하지 않는다
    - 선택적 비식별 관계 : 외래키에 null을 허용한다.