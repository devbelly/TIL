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
  - 자식 엔티티가 사용하는 필드는 nullable 해야한다.
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

### 복합키 : 비식별 관계 매핑

- 아래처럼 작성하면 오류가 발생한다.

  ```java
  @Entity
  public class Hello{
    @Id
    private String id1;
    @Id
    private String id2;
  }
  ```

- 영속성 컨텍스트에 엔티티를 저장할 때 엔티티의 식별자를 사용
- 식별자들을 구분할때는 equals와 hashcode를 사용한다.
- 복합키를 사용할때는 복합키를 위한 클래스를 사용하므로 해당 클래스에 equals와 hashcode를 사용해야한다.
- 복합키를 지원하는 방법은 아래 두가지가 있다.
  - `@Idclass`
  - `@EmbeddedId`

#### IdClass

- 코드
  
  ```java
  @Entity
  @IdClass(ParentId.class)
  public class Parent {
      @Id
      @Column(name = "PARENT_ID1")
      private String id1;
      @Id
      @Column(name = "PARENT_ID2")
      private String id2;

      @Column
      private String name;
  }
  ```

- 복합키 클래스 코드

  ```java
  public class ParentId implements Serializable {
    private String id1;
    private String id2;

    public ParentId(){}

    public ParentId(String id1, String id2){
        this.id1 = id1;
        this.id2 = id2;
    }
  }
  ```

- 복합키 클래스는 다음을 만족해야한다
  - Serializable을 구현
  - equals & override 메서드 오버라이드
  - 기본 생성자
  - 복합키 클래스의 속성의 이름과 클래스의 속성의 이름이 같아야 한다.(id1,id2가 동일함을 알 수 있다)
  - 클래스 식별자는 public 이여야 한다

- 이를 참조하는 자식 클래스

  ```java
  @Entity
  public class Child{
    @Id
    private String id;

    @ManyToOne
    @JoinColumns({
      @JoinColumn(name = "PARENT_ID1",referencedColumnName = "PARENT_ID1"),
      @JoinColumn(name = "PARENT_ID2",referencedColumnName = "PARENT_ID2")
    })
    private Parent parent;
  }
  ```

- 데이터 저장 시

  ```java
  public static void logic(EntityManager em) {
      Parent parent = new Parent();
      parent.setId1("myId1");
      parent.setId2("myId2");
      parent.setName("parent");
      em.persist(parent);
  }
  ```

  -  직접 ParentId.class를 생성하지 않아도 영속성 컨텍스트에 엔티티를 저장하는 시점에 ParentId 객체를 생성을 한다.
  
- 데이터베이스와 관련이 있는 접근이다.

#### @EmbeddedId

- @Idclass보다 객체지향적인 접근이다.
- 클래스 내에 바로 ParentId를 사용할 수 있다.

  ```java
  @Entity
  public class Parent{
    @EmbeddedId
    private ParentId parentId;

    ...
  }

  @Embeddable
  public class ParentId implements Serializable {
    @Column(name = "PARENT_ID1")
    private String id1;
    @Column(name = "PARENT_ID2")
    private String id2;
  }
  ```

#### 복합키와 equals(), hashCode()

- 영속성 컨텍스트는 식별자를 키로 저장하여 엔티티를 관리한다
- 기본적으로 사용하는 equals는 Object 클래스에 구현되어 있고 이는 참조값을 비교(동일성 비교)한다
- 식별자가 같다면 같은 엔티티로 구분(동일성 비교)해야하므로 equals와 hashCode를 필수적으로 구현하자.

### 식별, 비식별 관계의 장단점

- 식별관계보다는 비식별 관계가 갖는 이점이 많다.
- 식별관계는 부모의 기본키가 자식 테이블에 전파되는 단점이 있다.
- 식별관계는 기본키로 비즈니스적으로 의미가 있는 자연키 컬럼 조합을 조합하는 경우가 많다.
  - 비식별관계는 비즈니스와 전혀 관련 없는 대리키를 사용한다.
  - 하지만 비즈니스 요구사항은 언제든 변할 수 있으므로 대리키를 사용하는 것이 더 좋다.

- 정리
  - 새로운 프로젝트를 할 계획이 있다면 Long 타입의 대리키를 사용하는 것이 좋다.
  - Long은 920경 정도이므로 안전하다.
  - 선택적 비식별 관계보다 필수적 비식별 관계를 사용하는 것이 더 좋다.
    - NOT NULL을 사용해서 INNER JOIN을 사용하기 때문이다.

## 조인 테이블

- 테이블끼리 연관관계를 맺는 방법은 다음 두가지이다
  - 외래키
  - 조인 테이블 사용

- 상황(외래키)

  ![image](https://user-images.githubusercontent.com/67682840/233905122-27d85414-b589-411c-9bde-80e2f4f2c602.png)

  - 회원은 사물함 하나를 사용할 수 있다.
  - 회원이 사물함을 사용하기 전까지 두 테이블은 관련이 없다.
  - 선택적 비식별 관계이므로 두 테이블 JOIN 시 OUTER 연산을 사용한다.
  - 회원과 사물함이 가끔 관계를 맺는다면 많은 외래키가 NULL로 저장되는 단점이 있다.

- 상황(조인 테이블)

  <img width="514" alt="image" src="https://user-images.githubusercontent.com/67682840/233905613-93cedc01-96a9-4c98-8909-484eaa825b24.png">

  - 두 엔티티에 외래키를 위한 속성이 없다.
  - 단점은 테이블을 새로이 만들어야 하는 점이다.
  - MEMBER와 LOCKER를 조인하려면 추가적으로 MEMBER_LOCKER까지 조인해야한다.

<br>

# 8장, 프록시와 연관관계 관리

## 프록시

- 연관관계를 맺는다고 해서 항상 엔티티를 사용하는 것은 아니다.
- 사용하지도 않은 연관관계를 데이터베이스에서 가져오는 것은 비효율적
- JPA에서는 이러한 문제를 해결하기 위해 사용하는 시점에 쿼리를 날리는 지연로딩 기능을 지원한다.
- 데이터베이스에서 조회를 지연하기 위해서는 가짜 객체를 사용해야하는데 이를 프록시라고 한다.

> 참고
>
> JPA 표준 명세는 지연 로딩의 구현방법을 구현체에 위힘했다. 하이버네이트는 프록시와 바이트코드를 수정하는 방법을 제공한다.

### 프록시 기초

- `Member member = em.find(Member.class,"member1")` 을 사용하면 객체를 실제로 사용하는 것과는 상관없이 영속성 컨텍스트를 확인한 후 해당 객체가 없으면 데이터베이스에 쿼리를 날리게 된다.
- 실제 객체를 사용하는 시점까지 미루고 싶다면 `em.find()` 대신 `em.getReference()`를 사용하면 된다.

- 특징

  <img width="361" alt="image" src="https://user-images.githubusercontent.com/67682840/233927327-2c0016fb-41aa-4ea1-a147-ef79c3516559.png">

  - 프록시는 원본 클래스를 기반으로 구현하므로 사용하는 입장에서는 프록시인지 구분하기 어렵다
  - 프록시는 실제 객체에 대한 참조를 가지고 있어 메서드 호출시 실제 객체의 메서드를 호출한다.

- 프록시 예상 코드

  ```java
  class MemberProxy extends Member{
    Member target = null;

    public String getName(){
      if(target == null){

        // 2. 초기화 요청
        // 3. DB 조회
        // 4. 실제 엔티티 생성
      }
    }
    ...
  }
  ```

- 실행단계

  <img width="617" alt="image" src="https://user-images.githubusercontent.com/67682840/233928855-e178cda7-4824-48f7-b777-8e4a2bfaef0b.png">

  - proxy.getName() 호출
  - target이 null이므로 영속성 컨텍스트를 통해 엔티티 생성 요청
  - 데이터베이스를 조회해서 실제 객체 생성
  - target.getName() 호출

- 특징
  - 영속성 컨텍스트에 실제 객체가 존재한다면 `getReference()`를 호출해도 실제 객체가 리턴된다.
  - 프록시는 원본 객체를 상속했으므로 타입 체크시 주의해야한다
  - 프록시는 단 한번 초기화 되는데 이는 영속성 컨텍스트의 도움을 받는다. 만일 "준영속" 상태에서 영속성 컨텍스트의 도움을 받을 수 없다면 LazyInitializeException 예외가 발생한다.

### 프록시와 식별자

- 영속성 컨텍스트에 초기화를 요청하기 위해서는 식별자를 알아야한다
- 위 이유로 프록시 객체는 id 값을 가지고 있어 `Proxy.getId()`를 호출하더라도 초기화 되지 않는다
  - @Access(AccessType.PROPERTY)인 경우에만 해당하고
  - @Access(AccessType.FIELD)는 id를 호출하더라도 초기화 된다.

## 즉시로딩과 지연로딩

- 즉시로딩
  - 엔티티를 조회할 때 연관 엔티티도 함께 조회한다
  - @ManyToOne(fetch = FetchType.EAGER)
- 지연로딩
  - 연관 엔티티를 사용하는 시점에 JPQ가 SQL을 사용한다
  - @ManyToOne(fetch = FetchType.LAZY)

### 즉시로딩

- Team : Member = 1 : N 관계 일때 실행되는 쿼리는 다음과 같다.

  <img width="434" alt="image" src="https://user-images.githubusercontent.com/67682840/234152763-1551469b-ff94-4ca3-b317-1736141b380d.png">

  - 두 테이블을 조회하므로 SQL이 두번 실행 될 것 같지만 실제로는 OUTER JOIN을 사용해 한번만 실행된다
  - OUTER JOIN인 이유는 선택적 관계이기 때문이다. `@JoinColumn(nullable=false)` 또는 `@ManyToOne(optional = false)`를 추가하면 INNER JOIN으로 변경됨을 알 수 있다.

### 지연로딩

- 위 상황과 같다. 연관 엔티티인 Team에 접근할 때 SQL이 추가적으로 실행된다.

  <img width="359" alt="image" src="https://user-images.githubusercontent.com/67682840/234154141-c0986a83-09c0-4bf5-87e3-41e222676c76.png">

## 지연로딩 활용

- 상황에 따라 지연로딩을 할지 즉시로딩을 할지 결정해야한다. 아래와 같은 상황이 있다고 해보자
- Member와 연관된 Order는 가끔 사용되어서 LAZY로 결정했다.
- Member와 연관된 Team은 항상 같이 사용되어 EAGER로 결정했다.
- 위 상황을 반영한 코드는 다음과 간다.

  ```java
  @Entity
  public class Member {
      @Id
      private Long id;
      private String username;
      private Integer age;

      @ManyToOne(fetch = FetchType.EAGER)
      private Team team;

      @OneToMany(mappedBy = "member", fetch = FetchType.LAZY)
      private List<Order> orders;
  }
  ```
 
  `em.find(Member.class,~)`를 호출하면 다음과 같다. SQL에서 order와 관련된 내용은 찾아볼 수 없다.

  <img width="378" alt="image" src="https://user-images.githubusercontent.com/67682840/234157227-c379cacd-7dd6-43e0-95af-d3f8faad3ba8.png">

### 프록시와 컬렉션 레퍼

- `em.find(Member.class)`로 조회한 후 orders 객체의 클래스를 출력하면 `List<Order>` 대신 PersistenceBag이 나온다.

  ```
  org.hibernate.collection.internal.PersistentBag
  ```

  - 하이버네이트는 엔티티를 영속화 할 때 해당 엔티티에 컬렉션이 있으면 대상을 관리할 목적으로 내장 컬렉션으로 변경하는데 이를 컬렉션 레퍼라고 한다
  - 프록시와 하는 역할이 비슷하다

### 기본패치전략

- xxx To One : EAGER
- xxx to Many : LAZY
- 추천하는 방법은 모든 연관관계를 LAZY로 설정한 후 실제 운영함에 따라서 EAGER로 최적화할지 결정하는 것이 좋다.

### N+1 문제

- Cafe : Coffee = 1 : N 관계가 있다고 가정
- Cafe는 아래와 같이 작성되었다.

  ```java
  @Entity
  public class Cafe {
      @Id
      private Long id;

      @Column
      private String name;

      @OneToMany(mappedBy = "cafe",fetch = FetchType.EAGER)
      private List<Coffee> list = new ArrayList<>();
  }
  ```

- Cafe와 연관관계를 맺고 있는 커피가 1개 있다고 가정할 때, `em.find(Cafe.class,1L)`은 SELECT 가 몇번 실행될까?

  - 실행결과

    ```
    Hibernate: 
    select
        cafe0_.id as id1_0_0_,
        cafe0_.name as name2_0_0_,
        list1_.CAFE_ID as cafe_id2_1_1_,
        list1_.id as id1_1_1_,
        list1_.id as id1_1_2_,
        list1_.CAFE_ID as cafe_id2_1_2_ 
    from
        Cafe cafe0_ 
    left outer join
        Coffee list1_ 
            on cafe0_.id=list1_.CAFE_ID 
    where
        cafe0_.id=?
    ```

  - Cafe를 하나 조회했을 때 연관엔티티가 select에 포함되는 이유는 EAGER 때문이다.

- `em.createQuery("seelct c from Cafe c where c.id = 1").getSingleResult()` 또한 하나의 객체를 가져오는데, EAGER 이므로 SELECT가 한번 실행될까?

  - 실행결과

    ```java
     select
            cafe0_.id as id1_0_,
            cafe0_.name as name2_0_ 
        from
            Cafe cafe0_ 
        where
            cafe0_.id=1
    Hibernate: 
        select
            list0_.CAFE_ID as cafe_id2_1_0_,
            list0_.id as id1_1_0_,
            list0_.id as id1_1_1_,
            list0_.CAFE_ID as cafe_id2_1_1_ 
        from
            Coffee list0_ 
        where
            list0_.CAFE_ID=?
    ```

    - SELECT가 두번 실행되는 모습을 알 수 있다.


- 원인
  - JPQL은 글로벌 Fetch 전략을 무시하고 JPQL로만 SQL을 생성하기 때문이다
  - JPARepository에서 인터페이스 메서드를 사용해도 마찬가지 결과이다. 이는 메서드명을 기준으로 JPQL을 생성하기 때문이다.

- EAGER인 경우
  - JPQL에서 SQL을 생성해 쿼리 실행
  - Fetch전략이 EAGER 이므로 연관 엔티티 사용 여부에 관계 없이 즉시 하위 엔티티를 추가조회

- Lazy인 경우
  - JPQL에서 SQL을 생성해 쿼리 실행
  - Fetch전략이 LAZY 이므로 즉시 하위 엔티티를 조회하지는 않지만 연관 엔티티를 사용하는 시점에 쿼리가 발생해 결국 N+1 문제 발생

- 해결책
  - Fetch Join
    - 직접 JPQL에 fetch join을 명시하면 된다.
  - `@EntityGraph`
  - 자세한겐 뒤에서 다루도록 하자.

### 컬렉션에 FetchType.EAGER 사용시 주의점

- 컬렉션을 하나 이상 즉시로딩 하는 것을 권장하지 않는다

  위 내용을 확인하기 위해 아래처럼 작성했다.

  ```java
  @Entity
  public class Member {
      @Id
      private Long id;
      private String username;
      private Integer age;

      @ManyToOne(fetch = FetchType.LAZY)
      private Team team;

      @OneToMany(mappedBy = "member", fetch = FetchType.EAGER)
      private List<Order> orders = new ArrayList<>();

      @OneToMany(mappedBy = "member",fetch = FetchType.EAGER)
      private List<Child> children = new ArrayList<>() ;
  }
  ```

  실행하면 MultipleBagException 이라는 것이 발생한다.

  <img width="1419" alt="image" src="https://user-images.githubusercontent.com/67682840/234175600-a4216917-dc6c-486e-afd0-c2e4eb2ec672.png">

  - MultipleBagException?
    - BagType을 동시에 Fetch 해 올때 발생하는 문제
    - Bag(MultiSet)란 Set과 같이 순서가 없고 List와 같이 중복을 허용하는 자료구조
    - 자바에서는 Bag이 없기 때문에 하이버네이트에서는 List를 Bag로 사용하고 있다.
    - 문제를 해결하기 위해선 둘 중 하나를 List로 변경해야한다.
    - JPA는 BagType이 두개이상 로드되는 것을 허용하지 않는다!

  - 다시 원론으로 돌아와서 각 테이블의 데이터가 N, M이라면 쿼리의 실행결과가 N*M개가 되므로 애플리케이션의 성능저하가 발생한다.

- 컬렉션 즉시로딩은 항상 OUTER JOIN을 사용한다.
  
## 영속성 전이 : CASCADE

- 특정 엔티티를 영속 상태로 만들기 위해서는 연관 엔티티는 영속화 되어있어야 한다
- Parent : Child = 1 : N 관계에서 부모를 영속화 한 후 자식을 영속화 해야한다.
- CascadeType.PERSIST 옵션을 사용하면 부모 엔티티만 영속화해도 자식 엔티티가 영속화 된다.

  [코드]
  ```java
  @Entity
  public class Cafe {
      @Id @GeneratedValue
      private Long id;

      @Column
      private String name;

      @OneToMany(mappedBy = "cafe", cascade = CascadeType.PERSIST)
      private List<Coffee> list = new ArrayList<>();
  }
  ```

  ```java
  //PERSIST 예시
  public static void logic(EntityManager em) {

      Cafe cafe = new Cafe();
      cafe.setName("ediya");

      Coffee iceCoffee = new Coffee();
      iceCoffee.setCafe(cafe);
      cafe.getList().add(iceCoffee);

      Coffee hotCoffee = new Coffee();
      hotCoffee.setCafe(cafe);
      cafe.getList().add(hotCoffee);

      em.persist(cafe);
    }  
  ```

- Cascade.DELETE를 사용하면 부모 객체만 가져와서 삭제하면 된다. 

  [전]
  ```java
  Cafe cafe = em.find(Cafe.class,1L);
  Coffee iceCoffee = em.find(Coffee.class,2L);
  Coffee hotCoffee = em.find(Coffee.class,3L);

  em.remove(iceCoffee);
  em.remove(hotCoffee);
  em.remove(cafe);
  ```

  [후]
  ```java
  Cafe cafe = em.find(Cafe.class,1L);
  em.remove(cafe);
  ```

- 영속성 전이를 설정해도 `flush()` 시점에 전이가 발생한다.

## 고아 객체

- 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제하는 기능을 제공한다.
- 부모 엔티티의 컬렉션에서 자식 엔티티의 참조를 제거하면 자식 엔티티가 자동으로 삭제된다

  [코드]
  ```java
  public class Cafe {
    @Id @GeneratedValue
    private Long id;

    @Column
    private String name;

    @OneToMany(mappedBy = "cafe",orphanRemoval = true)
    private List<Coffee> list = new ArrayList<>();
  }
  ```

  책에서는 다음 코드를 실행하면 delete 쿼리가 발생한다고 한다.
  ```java
  Cafe cafe = em.find(Cafe.class,1L);
  cafe.getList().remove(0);
  ```

  하지만 delete 쿼리가 발생하지 않았다! JPA 스펙상에서는 동작해야하는데 하이버네이트는 PERSIST와 orphanRemoval이 관계를 맺고 있어서 발생하는 문제라고 한다. 
  - https://www.inflearn.com/questions/137740/orphanremoval%EA%B3%BC-cascade%EC%9D%98-%EA%B4%80%EA%B3%84

  orphanRemoval 자체를 잘 사용하진 않지만 정상적으로 동작하길 기대한다면 `cascade = CascadeType.PERSIST`를 추가하자

- 부모 엔티티를 삭제하면 연관된 자식 엔티티도 같이 삭제된다는 점에서는 `cascade = CascadeType.REMOVE`와 동일하게 동작한다.

## 영속성 전이 + 고아 객체, 생명주기

- CascadeType.ALL + orphanRemoval = true를 사용하면 부모 엔티티에서 자식 엔티티의 생명주기를 관리할 수 있게 된다.

<br>

# 9장, 값 타입

## 기본 값 타입

## 임베디드 타입

- 자바에서 기본적으로 제공하는 타입 대신 직접 정의해서 사용할 수도 있다.
- 여러개의 변수를 좀 더 의미있는 단위로 만들 수 있다.

  ```java
  @Entity
  public class Member {
    @Id @GeneratedValue
    private Long id;

    @Embedded
    private Address address;
  }
  ```

  ```java
  @Embeddable
  @Getter @Setter
  public class Address {
      private String city;
      private String street;
      private String zipCode;
  }
  ```

- 임베디드 타입은 기본생성자가 필수이다.(이유는 잘 모르겠다.)

### @AttributeOverride: 속성 재정의

- 한 엔티티에 동일한 임베디드 타입을 사용하면 컬럼명이 겹친다.
- `@AttributeOverrides`를 사용해서 중복되는 컬럼명의 이름을 변경한다
  
### 값 타입 공유

- 임베디드 타입을 공유하게 되면 Side Effect가 발생할 수 있따.

  ```java
  public static void logic(EntityManager em) {
    Member member = new Member();
    Address address = new Address();
    address.setCity("city");
    address.setStreet("street");
    address.setZipCode("zip Code");
    member.setAddress(address);

    em.persist(member);

    Member member2 = new Member();
    address.setCity("New City");
    member2.setAddress(address);

    em.persist(member2);
  }
  ```

  - 실행하게 되면 member의 city 또한 New City로 변경된다.
  - 자바에는 기본타입과 객체타입이 있는데 기본타입은 `=`연산자 사용시 자동으로 '복사'가 이루어진다
  - 하지만 값 타입은 객체타입이므로 `=`연산자 사용시 주소값을 복사하므로 공유참조 문제가 발생한다.

### 불변 객체

- 조회는 가능하지만 수정이 불가능한 객체를 불변객체라고 한다
- 여러가지 방법으로 구현이 가능하지만 생성자로만 값을 설정하고 세터를 열어두지 않는다.
- 불변이라는 작은 규약으로 부작용이라는 나쁜 결과를 막을 수 있다.

### 값 타입의 비교

- `int a = 10`, `int b = 10`에서 a와 b가 같으므로
- Address 객체에서 안에 있는 필드의 값들이 동일하다면 같다고 판단해야한다
- 즉, 값 타입을 사용할때는 `equals` 메서드를 재정의 하자.
  - `equals`를 재정의 했다면 `hashCode` 또한 재정의하자. 해시 기반의 자료구조에서는 hashCode도 사용해 비교하기 때문이다.

### 값 타입 컬렉션

- 값 타입 하나 이상을 저장하려고 하면 다음 오류가 발생한다.

  <img width="594" alt="image" src="https://user-images.githubusercontent.com/67682840/235408773-0ca72dd7-a364-416d-854e-2f7de872ec25.png">

- 값 타입을 하나 이상 저장하려면 컬렉션에 보관하고 `@ElementCollection`, `@CollectionTable`을 어노테이션을 사용해야한다.

  <img width="726" alt="image" src="https://user-images.githubusercontent.com/67682840/235409111-2de231e9-a6ba-429c-9280-472970e13c1f.png">

- 값 타입 컬렉션은 영속성 전이 + 고아 객체 제거 기능이 필수로 들어가있다.

### 값 타입 컬렉션의 제약

- 엔티티는 식별자를 기준으로 데이터베이스에서 검색하면 되므로 크게 문제가 안된다.
- 값타입도 컬렉션이 아닌 단일 형태면 엔티티를 검색해 수정하면 된다.
- 하지만 값 타입 컬렉션의 경우 테이블로 저장하게 된다
  - 테이블에 식별자가 없으므로 값 타입 컬렉션중 값 하나를 수정하게 되면 값 타입 컬렉션 테이블에 관련된 모든 데이터를 삭제한 후 컬렉션에 있는 값의 갯수만큼 INSERT가 발생한다

    이 코드를 실행하면 INSERT가 2번 발생한다.
    ```java
    public static void logic(EntityManager em) {
      Member member = new Member();

      List<Address> addresses = member.getAddress();
      Address address1 = new Address("city1","street1","zipCode1");
      Address address2 = new Address("city2","street2","zipCode2");
      addresses.add(address1);
      addresses.add(address2);

      em.persist(member);
      em.flush();
      em.clear();

      Member findMember = em.find(Member.class,1L);
      findMember.getAddress().get(0).setStreet("new Street");
    }
    ```

- 결론
  - 값 타입 컬렉션 테이블에 데이터가 많다면 일대다 관계를 사용할 것을 권장한다
  - 영속성 전이 + 고아 객체 제거 옵션을 사용한다면 값 타입 컬렉션처럼 사용할 수 있다.

### 정리

- 식별자가 필요하고 지속적으로 추적하고 변경하려면 값 타입이 아닌 엔티티로 사용하자

<br>

# 10장, 객체지향 쿼리 언어

- 애플리케이션을 개발하다보면 복잡한 쿼리를 작성해야할 필요성이 있다.
- 데이터베이스의 모든 데이터를 메모리에 올려서 필터링 하기는 현실적으로 무리가 있다.
- ORM을 사용하면 데이터베이스 중심이 아닌 객체지향적으로 개발을 하게 된다.
- 이러한 이유로 객체지향 쿼리를 작성할 필요가 있다.
- JPQL은 객체지향 쿼리로 다음과 같은 특징이 있다.
  - 데이터베이스 중심이 아닌 객체를 중심으로 쿼리를 작성한다.
  - 특정 데이터베이스에 의존적이지 않다.
- QueryDSL, Criteria 등 JPQL를 편리하게 사용해주는 도구이므로 JPQL에 대해 잘 알아두는것이 좋다.

## JPQL

- Member를 조회하는 JPQL은 다음과 같다.

  ```java
  String jpql = "select m from Member m where m.id = 1L";
  Member jpqlMember = em.createQuery(jpql,Member.class).getSingleResult();
  ```

- 위 코드를 실행해보면 영속성 컨텍스트에 Member가 존재하는 것과 관계없이 무조건 `SELECT`를 실행한다
- JPQL vs find
  - find는 영속성 컨텍스트를 우선적으로 조회하고 해당 엔티티가 없다면 데이터베이스에서 조회
  - JPQL은 데이터베이스에서 우선적으로 조회하고 해당 엔티티가 영속성 컨텍스트에 있다면 데이터베이스에서 조회한 결과를 버린다.

### Criteria

- JPQL을 생성해주는 쿼리 빌더
- JPQL을 문자가 아닌 코드로 생성
  - JPQL을 직접 작성하면 오타가 있어도 잡아내지 못함, 런타임때 해당 문제를 발견할 수 있음
  - Criteria로 작성하면 컴파일 시점에 오타를 잡아낼 수 있다.
- 객체의 필드 (Member의 username 필드)까지는 코드로 관리하지 못하는데 MetaModel API를 사용하면 필드까지 코드로 관리할 수 있다.
- 장점도 많지만 사용하기 복잡하다

### JPQL 특징

- SQL과 비슷하게 SELECT, UPDATE, DELETE를 사용할 수 있다.
- INSERT는 `em.persist`를 사용하면 되므로 없다.
- JPQL의 UPDATE나 DELETE는 벌크연산이다.
- `SELECT m FROM Member AS m where m.username = 'Hello'`
  - 대소문자 구분
    - 엔티티와 속성은 대소문자를 구분한다(Member, username)
    - JPQL 문법은 대소문자를 구분하지 않는다
  - 별칭 사용은 필수
    - `select username from Member m` 이라고 작성하면 `m.username`이 아니므로 오류가 발생한다.
  
**TypedQuery,Query**

- 생성한 JPQL을 실행하려면 쿼리 객체가 필요
- 반환할 타입을 명확히 할 수 있으면 TypedQuery, 아니라면 Query를 리턴한다
- TypedQuery 예시
  
  ```java
  String jpql = "select m from Member m where m.id = 1L";
  TypedQuery<Member> typedQuery = em.createQuery(jpql,Member.class);
  Member findMember = typedQuery.getSingleResult();
  ```

- Query 예시

  ```java
  String jpql = "select m.id, m.username from Member m where m.id = 1L";
  Query query = em.createQuery(jpql);
  Object object = query.getSingleResult();
  Object[] result = (Object[])object;

  System.out.println(result[0]);
  System.out.println(result[1]);
  ```
  - 검색 결과가 Long와 String 이므로 리턴타입이 명확하지 않아 Query를 사용하는 것이 옳다.


**결과 조회**

- 실제 데이터베이스에 조회를 한다
  - `getResultList()` : 조회 결과를 리스트형태로 반환, 없다면 비어있는 리스트 반환
  - `getSingleResult()` : 조회 결과가 단 하나가 아니라면 예외 발생

### 파라미터 바인딩

- 위치기반 파라미터 바인딩

  ```java
  String jpql = "select m from Member m where m.id = ?1";
  Member jpqlMember = em.createQuery(jpql,Member.class)
          .setParameter(1,1L)
          .getSingleResult();
  ```

- 이름기반 파라미터 바인딩

  ```java
  String jpql = "select m from Member m where m.id = :username";
  Member jpqlMember = em.createQuery(jpql,Member.class)
          .setParameter("username",1L)
          .getSingleResult();
  ```

- JPQL을 직접 작성하면 
  - 악의적인 사용자에 의해 SQL Injection의 위험이 있고
  - JPA와 데이터베이스에서 사용하는 쿼리 파싱을 재사용하지 못하므로 성능도 저하한다.
  - 파라미터 바인딩은 선택이 아닌 필수!

## 프로젝션

- SELECT 절에서 조회할 대상을 지정하는 것이 프로젝션
- 엔티티, 임베디드 타입, 스칼라 타입이 올 수 있다.
- 엔티티가 프로젝션 대상이라면 영속성 컨텍스트로 관리된다.
- 임베디드 타입은 값 타입 이므로 영속성 컨텍스트로 관리되지 않는다.  

### new

- 스칼라 타입을 프로젝션 대상으로 두면 `Object[]`를 사용해야한다.
- 이를 직접 다루는 것은 불편하므로 DTO 객체를 만들어서 사용하도록 하자.
- 코드
  
  ```java
  public static void logic(EntityManager em) {
    String jpql = "SELECT new jpabook.start.UserDTO(m.username,m.age) FROM Member m";
    TypedQuery<UserDTO> query = em.createQuery(jpql,UserDTO.class);
    List<UserDTO> list = query.getResultList();
    for(UserDTO x : list){
        System.out.println(x.getUsername());
        System.out.println(x.getAge());
    }
  }
  ```

**세타 조인**

- WHERE 절을 사용해서 세타 조인을 지원한다.
- 내부 조인만 지원한다
  - 조건을 만족하는 결과만 반환한다
- 전혀 관련없는 엔티티도 조인할수 있다.
- 코드
  
  ```java
  em.createQuery("select m, c from Member m, Coffee c where m.id = c.id").getResultList();
  ```

  ```
  select
      member0_.id as id1_1_0_,
      coffee1_.id as id1_0_1_,
      member0_.age as age2_1_0_,
      member0_.username as username3_1_0_ 
  from
      Member member0_ cross 
  join
      Coffee coffee1_ 
  where
      member0_.id=coffee1_.id
  ```

**ON**

- JPA 2.1 부터 JPQL join on을 지원한다.
- JOIN을 하기전 필터링 한다
- 내부조인은 WHERE을 사용하는 것과 결과가 같으므로 외부조인시에만 사용된다
  - 외부조인은 조건에 맞지않더라도(ON 절을 만족하지 않더라도) 결과값을 포함해서 리턴한다.
  - 하지만 내부조인은 조건에 맞는 결과행만 반환하므로 WHERE을 사용한 것과 결과가 같다
  - WHERE은 JOIN 이후 모든 행에 대해 필터링을 하기 때문에 외부조인 ON 과 WHERE은 다르다

### 페치 조인

- JPQL에서 성능 최적화를 위해 제공하는 기능
- 연관 엔티티나 컬렉션을 한 번에 조회하는 기술이다.

**엔티티 페치 조인**

- `em.createQuery("select m from Member m join fetch m.team")`
- `m.team` 이후에 별칭을 사용하지 않았다. 패치 조인은 별칭을 사용할 수 없다.
- 하지만 하이버네이트는 별칭을 허용한다
- 실행결과
  
  ```
  select
    member0_.id as id1_1_0_,
    team1_.id as id1_2_1_,
    member0_.age as age2_1_0_,
    member0_.TEAM_ID as team_id4_1_0_,
    member0_.username as username3_1_0_,
    team1_.teamName as teamname2_2_1_ 
  from
      Member member0_ 
  inner join
      Team team1_ 
          on member0_.TEAM_ID=team1_.id   
  ```

  select m 만 했는데도 member도 같이 선택이 되었음을 알 수 있다.
- 팀과 멤버를 지연로딩으로 설정하더라도 fetch join을 사용하면 연관 엔티티를 프록시가 아닌 실제 객체로 가져온다.
- 엔티티는 영속성 컨텍스트로 관리되므로 멤버가 준영속 상태여도 팀은 영속성 컨텍스트로 관리된다.

**컬렉션 페치 조인**

- JPQL은 JOIN 시 연관 엔티티를 갖고 조인을 하게 된다
- Team에서 members를 페치조인하는 쿼리를 작성해보자
- 코드
  
  ```java
  List<Team> list = em.createQuery("select t from Team t join fetch t.members",Team.class).getResultList();
  ```
- 그림

  <img width="613" alt="image" src="https://user-images.githubusercontent.com/67682840/235600525-3fa159b0-1ecb-437a-9be4-f201b9c830da.png">

- 원하는 결과는 하나의 팀 객체에 여러 멤버들이 담기는 것을 기대한다
- 하지만 그림 10.7과 같이 일대다조인 특성상 결과값이 증가한다.
- 실행결과
  
  ```
  alpha jpabook.start.Team@a565cbd
  memberA
  memberB
  alpha jpabook.start.Team@a565cbd
  memberA
  memberB
  ```

- 동일한 팀이 두번 조회되었다.
- 이를 방지하기 위해서는 distinct를 추가하자
  - 데이터베이스 SQL에서 distinct를 사용하고
  - 애플리케이션에서 다시 한번 중복을 제거해주는 효과가 있다.

**페치 조인의 특징과 한계**

- SQL한번으로 연관된 엔티티를 한 번에 조회할 수 있어서 성능을 최적화할 수 있다.
- 페치 조인은 글로벌 로딩 전략보다 앞선다.
- 즉, 지연로딩을 설정해도 JPQL에서 페치 조인을 사용하면 연관 엔티티를 한 번에 조회한다.
- 글로벌 로딩 전략은 LAZY로, 필요한 경우 페치 조인을 사용하는 것이 적절하다.

**페치 대상에 별칭을 줄 수 없다**
- JPA는 페치 대상에 별칭을 주지 않도록 한다.
  - Eclipselink는 별칭, on 사용가능
  - 하이버네이트는 별칭 사용가능하지만 on 사용 불가능
- 별칭을 잘못 사용하면 데이터 무결성이 깨질수 있고 특히 2차 캐시를 사용하는 경우 문제가 발생한다.
  - 예를 들어 Team : Member = 1 : 2 
  - `select distinct from Team t join fetch t.member where member.id = 1L` 와 같이 작성
  - JPQL 상으로는 팀에게 연관된 멤버는 1개, 하지만 객체상으론 2개의 멤버가 있을 것이라고 기대
  - 데이터 무결성이 깨진다.
  - 굳이 사용하려면 stateless session을 사용해서 1, 2차 캐시를 무효화하자.
  
**둘 이상이 컬렉션을 페치할 수 없다**
- 구현체에 따라 되기도 한다.
- 하이버네이트에서는 둘 이상의 컬렉션을 가지고 오려고 하면 카테시안곱으로 만들어지므로 multibagfetchexception 예외가 발생한다.

**컬렉션을 페치조인 하면 페이징 API(setFirstResult, setMaxResult)를 사용할 수 없다**
- 일대다가 아닌 일대일, 다대일은 페치 조인을 사용하더라도 페이징 API를 사용할 수 있다.

### 경로표현식

- 쉽게말해 .을 통해서 필드나 프로퍼티에 접근하는 것
- 필드의 종류는 세가지가 있다.
  - 상태 필드 : 단순히 값을 저장하기 위한 필드
  - 단일 값 연관필드: @ManyToOne, @OneToOne을 위한 엔티티
  - 컬렉션 값 연관필드: @OneToMany, @ManyToMany를 위한 엔티티

**경로 표현식과 특정**
- 상태 필드 경로 : 경로 탐색의 끝이다. 더는 탐색할 수 없다.
- 단일 값 연관 경로 : 묵시적으로 내부조인이 일어난다, 계속해서 탐색할 수 있다. 

  [코드]
  ```java
  em.createQuery("select m.team from Member m",Team.class).getSingleResult();
  ```

  [실행결과]
  ```java
  select
      team1_.id as id1_2_,
      team1_.teamName as teamname2_2_ 
  from
      Member member0_ 
  inner join
      Team team1_ 
          on member0_.TEAM_ID=team1_.id
  ```

  - 명시적으로 join을 사용하지 않았어도 inner join이 사용된 것을 알 수 있다.
  - team과 연관된 엔티티가 있다면 m.team.~ 로 계속 탐색이 가능하다.

- 컬렉션 값 연관 경로 : 묵시적으로 내부조인이 일어난다, 계속해서 탐색할 수 없다.
  
  - `select t.members from Team t` 가능
  - `select t.members.username from Team t` 불가능
    - 계속해서 탐색하고 싶다면 `select m.username from Team t join t.members m` 와 같이 명시적 조인 이후 탐색을 계속해야한다.
  
**특징**

- 조인이 성능에 미치는 영향이 크므로 명시적 조인을 통해 유지보수성을 높이자.

### Named Query : 정적쿼리

- JPQL은 두 가지로 나눌 수 있다.
  - 동적 쿼리 : `em.createQuery`처럼 JPQL을 문자로 완성해서 직접 넘기는 것을 동적 쿼리라고 한다. 런타임에 특정 조건에 따라 JPQL을 동적으로 구성할 수 있다.
  - 정적 쿼리 : 미리 정의한 쿼리에 이름을 부여해서 필요할때 사용

- 장점
  - 애플리케이션 로딩 시점에 JPQL 문법 오류 체크 + 쿼리 파싱
    - 애플리케이션 로딩 시점은 SessionFactory를 만드는 시점이다.
  - 사용하는 시점에는 파싱된 쿼리를 재사용
    - 성능 최적화에 도움이 된다.

> EntityManger
>
> - 영속성 컨텍스트와 관련이 있다.
> - 영속성 컨텍스트는 엔티티 객체의 집합이다.
> - EntityManager은 영속성 컨텍스트 내에 존재하는 엔티티들을 다루기 위한 API를 제공하는 인터페이스

> Persistence Unit
>
> - 단일 데이터베이스와 매핑되는 모든 클래스들에 대한 집합을 정의한다.

- 예시

  ```java
  @NamedQuery(
        name = "Member.findByUsername",
        query = "select m from Member m where m.username = :username"
  )
  public class Member {
      @Id @GeneratedValue
      private Long id;
  }

  //사용
  em.createNamedQuery("Member.findByUsername",Member.class)
    .setParameter("username","memberA");
  ```

- `Member.findByUsername`이라고 이름 지은 이유는 NameQuery가 영속성 유닛단위로 관리되기 때문이다

> @Retention
>
> 어노테이션이 어느 시점까지 남아있을지 결정한다
> - SOURCE : .java 파일까지 어노테이션이 남아있고 .class로 컴파일 되는 시점엔 어노테이션 정보가 사라진다. 롬복의 getter, setter가 이에 해당한다.
> - CLASS : .class 파일까지는 어노테이션이 남아있고 런타임 시점에는 어노테이션 정보가 사라진다. 롬복의 nonnull이 이에 해당한다. 
> - RUNTIME : 런타임 시점까지 어노테이션이 남아있다. 런타임에 리플렉션을 통해 어노테이션 정보를 확인할 수 있다.

> QueryHint
>
> - JPA가 하이버네이트에게 힌트를 제공할 수 있다. JPA 공식 지원은 아니기 때문이다. 
> - JPA에서는 동일 트랜잭션 내에서 엔티티에 대해 더티체킹을 한다. 더티 체킹을 하기 위해서는 내부적으로 엔티티 두개를 유지해야한다.
> - 하지만 엔티티를 조회용도로만 사용하기 위해서는 하이버네이트에게 readOnly라는 힌트를 제공해서 엔티티를 내부적으로 하나만 유지하게 할 수 있다. 

##  Criteria

- JPQL을 코드로 관리할 수 있게 해주는 기술
- JPQL을 직접 생성하는 것 보다 안전하다
- 막상 사용하면 복잡하다는 단점이 있다.

### 쿼리루트

- 예제

  [코드]
  ```java
  CriteriaBuilder cb = em.getCriteriaBuilder();
  CriteriaQuery<Member> cq = cb.createQuery(Member.class);

  Root<Member> m = cq.from(Member.class);
  ```

- m이 쿼리루트
- 조회의 시작점
- JPQL의 별칭과 같다고 이해해도 좋다.

### Criteria 쿼리 생성

- CriteriaBuilder를 통해서 CriteriaQuery를 생성해야한다
  
  `cb.createQuery(Member.class)`

- 반환타입을 지정하면 `em.createQuery(cq)` 시 타입을 지정해줄 필요가 없다.

### 조회
### DISTINCT
### 튜플
### 집합
### 정렬
### 조인

- join() 메서드와 JoinType 클래스를 사용한다
  
- 예제
  
  ```java
  Root<Member> m = cq.from(Member.class);
  Join<Member,Team> t = m.join("team",JoinType.INNER);

  cq.multiselect(m,t)
          .where(cb.equal(t.get("teamName"),"alpha"));
  ```

### 서브쿼리

- 간단한 예시

  ```java
  CriteriaBuilder cb = em.getCriteriaBuilder();
  CriteriaQuery<Member> mainQuery = cb.createQuery(Member.class);
  Subquery<Double> subquery = mainQuery.subquery(Double.class);

  Root<Member> m2 = subquery.from(Member.class);
  subquery.select(cb.avg(m2.<Integer>get("age")));

  Root<Member> m = mainQuery.from(Member.class);
  mainQuery.select(m)
          .where(cb.ge(m.<Integer>get("age"),subquery));

  ```

- 상호 관련 서브 쿼리

  - 서브쿼리에서 메인쿼리의 정보를 사용하려면 메인쿼리의 별칭을 얻어 사용해야한다.
  - `Root<Member> subM = subquery.correlate(m)` 을 통해서 메인 쿼리의 별칭을 얻을 수 있다.

### 메타모델

- criteria가 코드 기반의 JPQL 빌더임에도 불구하고 문자열을 사용하는 곳이 있다.

  `m.get("age")`

- 이런 부분들은 metamodel을 사용해서 해결할 수 있다.
  
  `m.get(Member_.age)`

- 다양한 툴을 통해 메타모델을 생성할 수 있지만 maven 기준으로는 dependency에 `hibernate-jpamodelgen`을 추가하면 된다.

  
## QueryDSL

- Criteria보다 설정이 간편하다
- Criteria에서 CriteriaQuery 객체를 만든 것처럼 QueryDSL에서도 JPAQuery를 만들어야한다
- 예제

  ```java
  EntityManager em = emf.createEntityManager();
  
  JPAQuery query = new JPAQuery(em);
  QMember qmember = new QMember("m");

  List<Member> members = query
    .from(qmember)
    .where(qmember.username.eq("test"))
    .list(qmember);
  
  ```
- `QMember` 클래스에서 제공하는 기본적인 쿼리타입이 있다.

  <img width="606" alt="image" src="https://github.com/devbelly/dailycard-server/assets/67682840/681e555f-b06e-474d-90c8-25a1cbea3107">

  - 쿼리 타입에서 variable은 JPQL내에서 별칭으로 사용된다.
  - 위 이유로 같은 엔티티를 조인하거나 서브쿼리를 사용하면 같은 별칭이 사용되므로 직접 지정해서 사용하는 것이 좋다.

### 결과조회

- uniqueResult : 단건 조회, 검색 결과가 없으면 null, 여러개면 예외발생
- singleResult : uniqueResult와 같지만 여러개면 맨 첫번째 결과 리턴
- list : 리스트를 반환, 없다면 비어있는 리스트 반환.
- listResults : 페이징에 사용된다
  - `SearchResult<?>` 를 반환한다
  - 전체 데이터 갯수를 확인하기 위한 쿼리가 추가적으로 나간다.

### 프로젝션

- select절에 조회대상을 지정하는 것을 프로젝션이라고 한다
- JDSL과는 다르게 마지막 결과조회를 하는 쪽에서 프로젝션 대상을 지정해준다
- 프로젝션 대상이 하나면 해당 타입으로 변환된다.

  ```java
  JPAQuery query = new JPAQuery(em);
  QMember qMember = new QMember("m");
  List<String> results = query
          .from(qMember)
          .list(qMember.username);
  ```

- 프로젝션 대상이 여러개면 Tuple을 사용한다.

  ```java
  JPAQuery query = new JPAQuery(em);
  QMember qMember = new QMember("m");
  List<Tuple> results = query
          .from(qMember)
          .list(qMember.username,qMember.id);

  for (Tuple tp : results){
      System.out.println(tp.get(qMember.id));
      System.out.println(tp.get(qMember.username));
  }
  ```

### 수정, 삭제 배치쿼리

- QueryDsl은 배치쿼리도 지원한다. 영속성 컨텍스트를 무시하고 바로 데이터베이스에 요청을 날리므로 주의하자.


### 네이티브 SQL

- 어떠한 이유로 JPQL을 사용할 수 없는 경우 JPA는 직접 쿼리를 작성할 수 있는 방법들을 열어두었다.
- 네이티브 SQL을 사용하면 엔티티를 조회할 수 있고 JPA가 지원하는 영속성 컨텍스트를 사용할수 있다.

**엔티티 조회**

- `em.createNativeQuery(sql, Member.class)`, JPQL처럼 클래스명을 명시하면 된다.

### 벌크연산

- 단건 업데이트는 더티체킹, 단건 삭제는 `em.remove()`를 사용하면 된다.
- 하지만 여러 건에 대해 적용하기 위해서는 벌크 연산을 해야한다
- 단, 벌크연산은 영속성 컨텍스트를 무시하고 바로 데이터베이스에 쿼리를 날리므로 주의해야한다
- 벌크연산은 `executeUpdate` 메서드를 사용한다.
- 예시

  ```java
  Member findMember = em.createQuery("select m from Member m where m.age = 40",Member.class).getSingleResult();
  
  System.out.println(findMember.getAge());

  em.createQuery("update Member m set m.age = m.age + 100").executeUpdate();

  System.out.println("벌크연산 직후 나이");
  System.out.println(findMember.getAge());
  ```

  출력결과
  ```
  40
  벌크연산 직후 나이
  40
  ```

**해결책**

- `em.refresh(findMember)`
  - select 쿼리가 한번 더 나가게 된다.
- 벌크연산 후 영속성 컨텍스트 초기화
  - 벌크연산은 영속성 컨텍스트와 2차캐시를 무시하므로 주의해서 사용하자.

### 영속성 컨텍스트와 JPQL

- JPQL의 조회대상은 엔티티, 임베디드 타입, 값 타입
- 조회대상이 엔티티라면 영속성 컨텍스트로 관리되지만 임베디드 타입, 값 타입은 영속성 컨텍스트로 관리되지 않는다.

### JPQL로 조회한 엔티티와 영속성 컨텍스트

- JPQL은 영속성 컨텍스트의 존재유무와 상관없이 일단 데이터베이스에 쿼리를 날린다.
- JPQL로 조회한 결과가 영속성 컨텍스트에 존재할 경우, 데이터베이스에서 조회해온 결과를 버린다.
  - 데이터베이스에서 조회해온 결과를 사용하면 영속성 컨텍스트에 더티체킹이 꼬일 수 있다.
- 즉, 영속성 컨텍스트에 있는 엔티티를 벌크연산 후 JPQL로 결과를 조회하려고 하면 원래 결과와 달라진다.

### JPQL과 플러시 모드

- 플러시는 변경내역을 데이터베이스에 반영하는 과정
- 기본 옵션은 AUTO, 이는 커밋과 쿼리시 플러시하는 옵션
- 다른 옵션으로는 COMMIT이 있다. 최적화시 사용하는데 조심해서 사용해야한다.
- `em.flushMode()`처럼 엔티티 매니저에 플러시 모드를 설정할 수도 있고 `em.createQuery().setFlushMode()` 처럼 쿼리에 플러시 모드를 설정할 수도 있다. 쿼리에 설정하는 것이 우선권을 갖는다.

<br>

# 11장, 웹 애플리케이션 제작

**PersistenceContext**

- 순수 자바환경에서는 EntityMangerFactory에서 EntityManager를 생성
- 스프링과 같은 컨테이너 환경에서는 `@PersistenceContext`를 통해 컨테이너에서 관리되는 EntityManager를 사용할 수 있다.

**Transactional**

- 서비스 계층에서 사용되면 로직 시작 시 트랜잭션을 열고 끝날때 커밋한다
- 테스트에서 사용되면 로직 종료 시 데이터베이스에 있는 내용들이 롤백된다

<br>

# 12장, spring data jpa

- 기능 개발을 하다보면 반복적으로 이루어지는 CRUD가 있다.
- JpaRepository에서 CRUD를 위한 메서드를 제공한다.
- 코드

  ```java
  public MemberRepository extends JpaRepository<Member,Long>{
    Member findByUsername(String username);
  }
  ```

- 스프링 데이터 JPA는 메소드 명을 기반으로 JPQL을 생성해준다.

  ```java
  select m from Member m where username = :username
  ```

- JpaRepository가 제공하는 기능들을 살펴보자
  - `save(S)` : 새로운 엔티티는 저장하고 이미 있는 엔티티는 수정한다.
  - `delete(T)` : 엔티티 하나를 삭제한다. 내부적으로 EntityManager.delete() 호출
  - `findOne(Id)` : 엔티티 하나를 조회한다. 내부적으로 EntityManager.find() 호출
  - `getOne(ID)` : 엔티티를 프록시로 조회한다. 내부에서 EntityManager.getReference() 호출

## 쿼리 메서드 기능

- 스프링 데이터 JPA는 메서드 이름만으로 쿼리를 생성하는 기능, JPQL을 생성한다
- 앞서 살펴봤듯이 JPQL의 종류에 정적쿼리와 동적쿼리가 있었다.
- 쿼리 메서드도 세가지 기능이 있다.
  - 메소드 이름으로 쿼리 생성
  - 메소드 이름으로 JPA NamedQuery 호출
  - `@Query` 어노테이션을 사용해서 레포지토리에 직접 정의

### 메소드 이름으로 쿼리 생성

- 유저의 이름과 나이만으로 검색하는 예제

  ```java
  public interface MemberRepository extends JpaRepository<Member,Long>{
    Member findByUsernameAndAge(String username, Long age);
  }
  ```
  ```
  select m from Member m where m.username = ?1 and m.age = ?2
  ```

### JPA NamedQuery

- NamedQuery란 정적쿼리
- 말 그대로 쿼리에 이름을 붙여서 사용하는 방법
- 생성하기 위해서는 `@NamedQuery` 어노테이션이나 xml을 정의하는 방법이 있다.
- 코드

  ```java
  @Entity
  @NamedQuery(
    name = "Member.findByUsername",
    query = "select m from Member m where m.username = :username")
  public class Member{
    ...
  }   
  ```

  ```java
  // JPA에서 호출
  em.createNamedQuery("Member.findByUsername",Member.class)
    .setParameter("username",회원1)
    .getResultList();
  ```

  ```java
  // Spring data jpa에서 호출
  public interface MemberRepository extends JpaRepository<Member,Long>{
    List<Member> findByUsername(@Param String username)
  }
  ```

  - 엔티티명.메서드명 으로 NamedQuery가 있는지 확인한다
  - 없다면 런타임에 JPQL을 생성한다.

### @Query, 리포지토리 메서드에 쿼리 정의

- 이름 없는 NamedQuery 방식
- NamedQuery와 마찬가지로 실행시점에 문법오류를 발견할 수 있는 장점이 있다.
- 코드

  ```java
  @Query("select m from Member m where m.username = :username")
  Member findByUsername(@Param String username);
  ```

### 벌크연산

- JPA에서는 `em.createQuery("...").executeUpdate()`를 사용했다.
- Spring Data Jpa에서는 `@Modifying`을 사용한다.
- 코드

  ```java
  @Modifying
  @Query("update Product p set p.price = p.price * 1.1 where p.stockAmount < :stockAmount")
  int bulkPriceUp(@Param("stockAmount") Long amount);
  ```

- 벌크연산 이후 영속성 컨텍스트를 초기화하고 싶다면 `clearAutomatically = true`를 사용할 수 있다. 기본값은 false

### 반환 타입

- 여러건 조회, 반환 결과 없다면 빈 컬렉션
- 단건 조회, 반환 결과가 없다면 null
  - 단건 조회시 Spring Data Jpa는 내부적으로 `.getSingleResult()`를 사용
  - 반환되는 결과가 없다면 `NoResultException` 예외가 발생하지만 개발자입장에선 다루기가 어렵기 때문에 Spring Data Jpa에서는 null로 반환된다. 

### 페이징과 정렬

- 페이징과 정렬을 위해 쿼리 메서드에 Sort와 Pageable 파라미터를 추가적으로 넣을 수 있다.

  ```java
  List<Member> findByNameStartingWith(String name, Pageable pageable);
  ```

- Pageable 파라미터를 사용할 경우, 반환값으로 `Page<Member>` 또는 `List<Member>`를 사용할 수 있다.
  - `Page<Member>`를 반환값으로 사용한다면 전체 데이터 파악을 위해 count 쿼리가 발생한다.

### 명세

- DDD에서 Specification 개념이 도입되었다.
- 스프링 데이터 JPA는 Jpa Criteria를 통해 제공
- Predicate는 참, 거짓으로 평가되는 술어. 검색조건이 하나의 Predicate일 수 있다.
- Composite Pattern으로 구현되어서 여러개의 Specification을 사용할 수 있다.
- Specifications 클래스는 Specification을 연결할 수 있도록 도와주는 클래스
  - and, or, not 등을 제공한다.
- Specifiation 구현체는 인자로 제공되는 CriteriaBuilder, root 등을 활용해서 적절한 Predicate를 생성하면 된다.

> Specifications
>
> - Specification을 체이닝 할 수 있도록 도와준다.
> - 특이한 점은 생성자를 private으로 막고 static 생성자를 사용하는데, static 생성자는 where이다. 순서를 강제할 때도 static 생성자를 사용할 수 있다.
>

### 도메인 클래스 컨버터 기능

- 컨트롤러에서 사용자 아이디가 넘어온다면 회원 객체를 찾기 위해 Repository를 활용할 것이다.
- 도메인 클래스 컨버터 기능을 활용한다면 자동으로 객체와 연관된 레포지토리에서 찾아서 객체 매개변수와 매핑을 해준다.

  <img width="609" alt="image" src="https://github.com/devbelly/TIL/assets/67682840/e2adf942-715b-454e-8353-91d668c08d65">

### SimpleJpaRepository

- 스프링 데이터 Jpa에서 제공하는 인터페이스는 SimpleJpaRepository가 구현한다.

- `@Repository` : Jpa가 던지는 예외를 스프링 예외로 변환한다

  - 서비스 계층이 데이터 접근 기술(Jpa)에 의존적인 것은 좋지 않다.
  - 스프링은 이 문제를 해결하기 위해 AOP를 사용하고 스프링 예외로 변환한다.

- `@Transaction(readOnly = true)`

  - 기본적으로 모든 메서드에 트랜잭션이 적용되어있다.
  - 데이터를 조회하는 메서드에 readOnly 옵션이 적용되어있는데 트랜잭션 종료시점에 플러시를 하지 않아 약간의 성능 향상을 얻을 수 있다.

<br>

# 13장, 웹 애플리케이션과 영속성 관리

## 트랜잭션 범위의 영속성 컨텍스트

- 순수 자바 애플리케이션은 트랜잭션과 영속성 컨텍스트 관리를 직접해야한다
- 스프링에서 JPA를 사용하면 스프링이 제공하는 전략에 따라 트랜잭션과 엔티티매니저를 사용한다.
- 스프링이 기본적으로 제공하는 전략은 **트랜잭션 범위의 영속성 컨텍스트**
  - 트랜잭션과 영속성 컨텍스트의 범위가 일치한다.
  - 트랜잭션을 시작하면 영속성 컨텍스트를 생성하고 종료되면 영속성 컨텍스트도 종료된다.
  - 같은 트랜잭션은 같은 영속성 컨텍스트에 접근한다.

- 트랜잭션이 같으면 사용하는 영속성 컨텍스트는 동일하다.

  ```java
  @Transactional
  public void methodName(){
    repo1.find();
    repo2.find();
  }
  ```

  - repo1, repo2는 각각 다른 영속성 컨텍스트 객체를 사용하지만 동일한 트랜잭션 내에 존재하므로 결국 사용하는 영속성 컨텍스트는 동일하다.

## 준영속 상태와 지연로딩

- 컨테이너의 기본전략, 트랜잭션 범위의 영속성 컨텍스트 전략은 서비스 계층까지만 영속성 컨텍스트가 존재하고 프레젠테이션 계층에서는 영속성 컨텍스트가 존재하지 않음
- 문제가 되는 경우는 프레젠테이션 계층에서 지연로딩으로 설정된 연관객체에 대한 정보를 사용하는 경우이다.
  - 프록시로 존재하는 연관객체에 접근하여 메서드를 호출하면 영속성 컨텍스트에 초기화 요청을 보냄
  - 하지만 프레젠테이션 계층에서는 영속성 컨텍스트가 존재하지 않으므로 LazyInitializationExxception이 발생한다.
- **준영속 상태의 지연로딩**문제를 해결하기 위한 방법은 두가지가 있다.
  - 뷰가 필요한 엔티티를 미리 로딩하는 방법
  - OSIV 

- 뷰가 필요한 엔티티를 미리 로딩하는 방법은 시점에 따라 세가지로 나눌 수 있다.
  - 글로벌 패치 전략(Fetch.EAGER)
  - Fetch Join 전략
  - 강제로 초기화

  **1. 글로벌 패치 전략**
  
  - 연관 엔티티를 즉시 로딩하도록 설정한다
  - 문제점
    - 모든 뷰에서 연관 엔티티를 필요로 하는 것이 아니면 낭비이다.
    - N+1 문제가 발생한다.  

**N+1 문제**
- `em.find()`를 호출하면 EAGER로 설정된 연관 엔티티를 가져올때 JOIN을 사용한다.
- `em.find(Order.class,1L)` 호출시
 
  <img width="490" alt="image" src="https://github.com/devbelly/TIL/assets/67682840/025780b5-187e-4765-865d-b6e4d278af99">

- 문제는 JPQL을 사용한다면? `em.createQuery("select t from Team t")`
- 실행결과

  <img width="1260" alt="image" src="https://github.com/devbelly/TIL/assets/67682840/1439a0ea-16f6-4a3c-8275-0d49b8c39497">

- 원인
  - JPQL은 글로벌 페치 전략(Eager,Lazy)와 상관없이 JPQL만 확인하여 SQL을 생성
  - Member이 10개 조회됨, 각 Member마다 Team이 Eager이므로 영속성 컨텍스트 확인
  - 영속성 컨텍스트에 아무것도 없으므로 team과 관련된 select 쿼리 발생
  - 처음에 조회한 데이터수 만큼 다시 조회를 하는 문제가 발생!! 이를 N+1 문제라고 한다

- Fetch Join을 통해서 해결가능
  - 하지만 Fetch Join을 통해 준영속 엔티티의 지연로딩 문제를 해결하면..
  - 프레젠테이션 계층이 간접적으로 데이터 계층을 침범하는 문제가 발생한다
    - `service.findMember`
    - `service.findMemberWithTeam`
- 이처럼 데이터 계층이 프레젠테이션 계층을 위해 로직을 추가하는 것은 옳지 않다.
  - facade 계층을 추가해서 해결하자.

### FACADE 계층 추가

- 프레젠테이션에서 사용하는 준영속 엔티티를 서비스 계층에서 초기화하면 논리적 결합이 생긴다.
- FACADE 계층을 추가한다. 

  <img width="641" alt="image" src="https://github.com/devbelly/TIL/assets/67682840/39261b4a-bb59-492c-baee-893b597ea02d">

  - 컨트롤러에서 사용하는 준영속 엔티티 초기화에 대한 책임을 갖는다.
  - 프레젠테이션 계층과 도메인 모델 계층간 논리적 의존성을 분리
  
## OSIV

- 퍼사드를 도입하더라도 프레젠테이션 계층이 퍼사드에 의존하게 된다.
- 본질적인 문제는 뷰에서는 영속성 컨텍스트가 없기 때문이다.
- OSIV는 영속성 컨텍스트를 뷰까지 열어둔다.

### 과거 OSIV : 요청당 트랜잭션

- 요청이 시작되면 영속성 컨텍스트를 만들고 트랜잭션을 시작, 종료시 영속성 컨텍스트와 트랜잭션을 종료
- 뷰에서도 영속성 컨텍스트를 사용하므로 미리 초기화할 필요가 없다.
- 문제점
  - 화면에 보여주기 위해서 엔티티를 잠시 변경했지만 데이터베이스에 커밋된다.
- 해결책
  - DTO
  - 래퍼클래스
  
### 스프링 OSIV : 비즈니스 계층에서만 트랜잭션을 유지

- 요청당 트랜잭션 방식 문제점은 트랜잭션과 영속성 컨텍스트의 범위가 일치한다는 것.
- 프레젠테이션 계층에서 수정이 데이터베이스에 반영되지 않으면서 영속성 컨텍스트를 사용
  - 영속성 컨텍스트와 트랜잭션을 분리하자
- 스프링 프레임워크가 제공하는 OSIV의 작동방식

  <img width="605" alt="image" src="https://github.com/devbelly/TIL/assets/67682840/e48797e5-90da-44f9-8b8f-61b3acbc3d8b">

  - 필터나 인터셉터에서 영속성 컨텍스트를 생성, 트랜잭션을 시작하진 않는다.
  - 서비스 계층에서 앞서 생성해둔 영속성 컨텍스트를 통해 트랜잭션 시작
  - 서비스 계층 로직 종료 이후 트랜잭션 종료
  - 컨트롤러와 뷰에서는 영속성 컨텍스트가 살아있어 엔티티를 변경하더라도 트랜잭션에 영향을 미치지 않는다.
- 영속성 컨텍스트에 있는 엔티티를 수정후 flush를 호출하면 예외 발생
- 하지만 Nontransactional read는 상관없다.
  - 프록시 객체를 조회하는 것은 Nontransactional read
- 주의점
  - 스프링이 제공하는 OSIV를 사용 중, 컨트롤러에서 다음과 같은 로직
  - 코드

    ```java
    //controller 로직 시작
    Entity entity = ..
    entity.setName("xxx");

    //새로운 비즈니스 로직 시작..
    anotherService.method();
    ```

# 14장, 컬렉션과 부가기능

## 컬렉션

- 자바에서 사용하는 컬렉션의 종류는 Collection, List, Set, Map 등이 있다.
- 하이버네이트는 자바 컬렉션을 그대로 사용하는 것이 아닌 내장 컬렉션으로 래핑한 후 사용한다.
- 위 이유로 컬렉션 필드는 엔티티 작성시 즉시 초기화 하는 것이 좋다. `private List<Member> members = new ArrayList<>();`

### Collection, List

- 래퍼클래스로 PersistentBag을 사용한다.
- 중복허용, 순서없음
- 리스트에 엔티티를 추가하더라도 지연 로딩된 컬렉션을 초기화 하지 않는다.

### Set

- 래퍼클래스로 PersistentSet을 사용한다
- 중복허용안함, 순서없음
- Set에 엔티티를 추가하면 중복검사를 위해 equals와 hashCode를 사용한다.
  - Set에 엔티티를 추가하면 지연 로딩된 컬렉션을 초기화한다.(중복 방지를 위해 원소들을 확인해야하기 때문이다)

**@OrderColumn**의 단점
- Team : Member = 1 : N 상황, Team의 members 필드에 @OrderColumn이 있다고 가정
- 일대다 테이블의 특성 때문에 `@OrderColumn(name = "POSITION")`을 Team에 작성했다 하더라도 POSITION 컬럼은 Member 테이블에 저장된다.
  - 이 이유로 Member은 Position 컬럼에 대한 정보를 모르고 오직 `team.members`에 접근해야지 POSITION 컬럼에 대한 update sql이 발생한다.
- 순서가 존재하므로 리스트의 두번째 값을 제거했다면 그 뒤에있는 값들의 POSITION이 변경되어야한다. 즉 추가적인 SQL이 발생
- 위 단점들로 인해 실무에서는 OrderBy를 선호한다

## 엔티티 그래프

- 연관 엔티티를 가져올때는 글로벌 패치전략 EAGER을 사용하거나 패치조인을 사용할 수 있다.
- 실무에서 EAGER은 변경가능성이 낮고 애플리케이션 전체에 영향을 주므로 패치조인을 사용한다.
- 패치조인은 비슷한 쿼리를 여러개 작성한다는 단점이 있다.
  - `select o from Order o join fetch o.member`
  - `select o from Order o join fetch o.orderItems`
- 이 원인은 JPQL이 데이터 조회 + 연관 엔티티 조회 두가지 기능을 수행하기 때문이다.
- JPA 2.1 부터 도입된 엔티티 그래프 기능을 통해 기능을 분리하자.


### NamedEntityGraph

- 코드
  
  ```java
  @NamedEntityGraph(
        name="Team.withMember",
        attributeNodes = {
                @NamedAttributeNode("members")
        }
  )
  public class Team {
      @Id @GeneratedValue
      private Long id;

      private String teamName;

      @OneToMany(mappedBy = "team")
      private List<Member> members = new ArrayList<>();
  }
  ```

- Team을 조회할 때 객체 그래프로 Member도 설정했으므로 Lazy로 설정했을지라도 members도 같이 조회하게 된다.
- 사용
  
  ```java
  //1. em.find에서 EntityGraph를 사용하는 방법
  EntityGraph graph = em.getEntityGraph("Team.member");

  Map hints = new HashMap();
  hints.put("javax.persistence.fetchgraph",graph);

  Member member = em.find(Member.class,1L,hints)
  ```

- Member에서 Team도 객체 그래프로 같이 조회할 때 `em.find()`를 사용한다면 inner join이 사용된다.

  - JPQL으로 EntityGraph를 탐색하면 outer join이 사용된다.(만약 싫다면 fetch join을 명시하자.)

### 동적 엔티티 그래프

- `em.createEntityGraph()`를 사용하자
- 코드

  ```java
  EntityGraph<Member> graph = em.createEntityGraph(Member.class);
  graph.addAttributeNodes("team");

  Map hints = new HashMap();
  hints.put("javax.persistence.fetchgraph",graph);

  Member findMember = em.find(Member.class,1L,hints);
  ```

- fetchgraph vs loadgraph
  - fetchgraph는 `addAttributeNode`로 추가한 속성만 가져온다
  - loadgraph는 추가한 속성뿐만 아니라 fetch.EAGER까지 가져온다.  

<br>

# 15장, 고급 주제와 성능 최적화

## 엔티티 비교

- Test 클래스에 @Transactional을 사용하면 메서드 종료시 플러시를 하지 않고 데이터베이스에 커밋도 하지 않는다.
  - 실행되는 로그를 보고 싶다면 강제로 `em.flush()`를 호출하자.
- @Transactional이 겹쳐서 사용된다면?
  - 예시
  
    ```java
    @Transactional
    method {
      //@Transactional
      method2()
    }
    ```
  
  - 여러가지 전략이 있지만 Propagation.REQUIRED 전략을 기본으로 사용한다.
    - 사용중인 트랜잭션이 있다면 해당 트랜잭션에 참여하고
    - 사용중인 트랜잭션이 없다면 새로운 트랜잭션을 시작한다.

- 동일성 비교는 같은 영속성 컨텍스트에 존재하는 엔티티끼리 가능
- 서로 다른 영속성 컨텍스트 끼리 비교할 경우 비즈니스키를 활용해 equals 비교를 하자.  

## 프록시 심화주제

### 영속성 컨텍스트와 프록시

- 이전에 살펴봤듯, 동일한 영속성 컨텍스트 내에서는 엔티티 동일성을 보장함을 확인했다.
- 프록시로 조회한 후 원본 객체를 조회하면 프록시가 조회된다.
  
  ```java
  public static void logic(EntityManager em) {

    Member member = new Member();
    em.persist(member);

    em.flush();
    em.clear();

    Member refMember = em.getReference(Member.class,1L);;
    Member findMember = em.find(Member.class,1L);

    System.out.println(refMember.getClass());
    System.out.println(findMember.getClass());
    System.out.println(refMember==findMember);
  }
  ```

  결과
  ```java
  class jpabook.start.Member$HibernateProxy$EXIylM7v
  class jpabook.start.Member$HibernateProxy$EXIylM7v
  true
  ```

## 성능 최적화

### N+1 문제

- Member : Order = 1 : N 인 상황
- 코드

  ```java
  public class Member {
    @Id @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "member", fetch = FetchType.EAGER)
    private List<Order> orders = new ArrayList<>();
  }

  public class Order {
    @Id
    private Long id;

    @ManyToOne
    private Member member;
  }
  ```

- `em.find(Member.class,1L)`을 하면 글로벌 패치 전략 EAGER에 따라 단 한번의 쿼리로 잘 가져온다.

  <img width="497" alt="image" src="https://github.com/devbelly/TIL/assets/67682840/9ab931ed-f112-4fca-8662-0a39588dd833">

- 문제는 JPQL을 사용할 때 발생한다. `em.createQuery(~)`를 사용하면 어떻게 될까?

  <img width="401" alt="image" src="https://github.com/devbelly/TIL/assets/67682840/58a79369-865c-4986-a2a1-a2996322fcda">

  - JPQL은 작성한 SQL 문법을 기반으로 쿼리를 생성하기 때문에 글로벌 패치 전략을 무시하고 작성한다.
  - EAGER이 나중에 작동, 하나의 회원에 연관된 ORDER를 가져오기 위해 추가 쿼리가 발생
  - 만약 회원이 한명이 아니라 5명? 아래처럼 총 5개의 쿼리가 발생한다.

    <img width="1533" alt="image" src="https://github.com/devbelly/TIL/assets/67682840/1f9cae04-dea7-4d1d-91b6-27619583d100">

- 처음 실행한 SQL의 결과수만큼 추가 쿼리를 발생하는 문제가 N+1 문제
- 지연로딩으로 설정하더라도 추가 쿼리가 나중에 발생할 뿐 피할 수 없는 문제이다.

### 해결책

- fetch join
- batch size
  
  - `@BatchSize`를 사용하면 IN 절을 통해 쿼리 횟수를 줄일 수 있다.
  - 예를 들어 Member가 10명이라면 추가 쿼리가 10번 발생
  - BatchSize를 5로 설정하면 추가쿼리가 2번만 발생한다.

    <img width="1594" alt="image" src="https://github.com/devbelly/TIL/assets/67682840/c7cb58cf-a9aa-46cf-bb69-3805d2b2bf42">

- subquery

  - @Fetch(FetchMode.SUBSELECT) 를 사용하면 서브쿼리를 통해 연관 데이터를 가져온다.

- 모든 연관관계를 LAZY로 설정하고 필요할때만 fetch join을 사용해서 가져오도록 하자.

### 읽기전용쿼리의 성능 최적화

- 엔티티가 영속성 컨텍스트에서 관리되면 얻을 수 있는 이점이 많다.
- 하지만 그만큼 영속성 컨텍스트에서 관리해야하는 것이 많다. 
  - 예를 들어 변경감지를 사용하기 위해선 스냅샷을 저장, 메모리를 더 사용해야한다.
- 최적화 방법
  - 스칼라 타입으로 조회, 엔티티를 조회한 것이 아니므로 영속성 컨텍스트에 저장하지 않는다.
  - `setHint`에 readOnly를 추가
  - `@Transactional(readOnly)`
    - FLUSH 모드를 manual로 설정한다.(직접 flush를 호출하기 전까지 실행되지 않음)
    - flush만 안하므로 스냅샷은 가지고 있다. 단지 비교만 안할뿐
    

> 하이버네이트 세션

- JPA 엔티티 매니저는 AUTO, COMMIT 모드를 지원하고 MANUAL을 지원하지 않지만 하이버네이트 세션은 MANUAL을 지원한다.
- 사용자가 flush를 호출하기 전까지는 플러시가 발생하지 않는다.
- 하이버네이트 세션은 JPA 엔티티 매니저를 하이버네이트로 구현한 것.
  - `entityManager.unwrap(Session.class)` 로 구할 수 있다.

<br>

**정리**
- 메모리를 절약하는 방법
  - 스칼라 타입을 조회
  - 읽기전용 쿼리 힌트 사용
- flush를 하지 않아서 최적화
  - 읽기 전용 트랜잭션 사용
  - 트랜잭션 밖에서 읽기
- 스프링에서는 읽기 전용 쿼리 힌트 + 읽기 전용 트랜잭션을 사용한다.

### 배치처리

- 수만건 이상의 데이터를 영속성 컨텍스트에서 동시에 관리하면 메모리 부족 문제가 발생
- 적절한 단위로 끊어서 `em.flush()`, `em.clear()`를 해야한다.

**배치 수정**

- 많은 데이터를 조회한 후 수정
- 페이징 방식과 커서 방식이 있다.
- 페이징 방식

  코드
  ```java
  int pageSize = 1000;
  for(int i=0;i<100_000/pageSize;++i){
      List<Member> lists = em.createQuery("select m from Member m")
              .setFirstResult(i*pageSize)
              .setMaxResults(pageSize)
              .getResultList();

      for(Member ret : lists){
          ret.setAge(20);
      }

      em.flush();
      em.clear();
  }
  ```

- 스크롤방식
  - 하이버네이트 세션 기능이므로 unwrap 사용, 추후 공부

### 하이버네이트 무상태 세션

- 영속성 컨텍스트가 없고 2차 캐시도 없다.
- 즉, 영속성 컨텍스트를 초기화하거나 플러시할 필요도 없다
- 수동적으로 update 메서드를 호출해야한다.

### 트랜잭션 단위의 쓰기 지연 장

- 개발의 편의성 증가
- 한번의 네트워크 통신 수만건의 메서드를 호출하는 것만큼 굉장히 무거운 연산
  - 네트워크 통신 횟수를 줄여준다.
- 가장 중요한 것은 데이터베이스 로우에 걸리는 락을 최소화 하는 것

  코드
  ```
  update(member);
  logicA();
  logicB();
  commit();
  ```

  - JPA를 사용하지 않았다면 `update(member)`에서 락을 걸고 commit 될때까지 락을 유지
  - JPA 쓰기 지연을 사용했다면 commit 직전에 update를 실행해서 락이 걸리는 시간을 최소화한다.

<br>

