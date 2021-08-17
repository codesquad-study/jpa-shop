### JPA and ORM

JPA(Java Persistence API) : 자바 진영의 ORM 기술 표준

ORM(object-relational mapping) : 객체와 관계형 DB를 매핑

- RDB에 데이터를 읽고 쓰는 처리 → 객체에 데이터를 읽고 쓰는 방식으로 구현한 기술

<br>

### 왜 등장?

JAVA는 객체지향 O, **BUT** DB나 SQL은 객체지향 X

✅ 객체를 DB에 저장하고 관리할 때, 개발자가 직접 SQL을 작성하지 않아도 된다.

→ 데이터베이스를 의식하며 코드를 작성하지 않아도 되기 때문에 비지니스 로직에 더 집중할 수 있게 되었다.

→ 쿼리문을 효율적으로 짜기 위해서 비지니스 로직을 희생하거나 vice versa의 경우도 발생하지 않음

✅ JPA가 객체를 자동으로 Mapping 해줌

→ 패러다임 불일치 해결

→ 컴파일 타임에 오류를 확인할 수 있다. 디버깅도 편리

<br>

### Entity

데이터베이스에서 영속적으로 저장된 데이터를 매핑한 자바 객체

→ EntityManager에 의해 데이터베이스의 데이터와 동기화된다

🤔 **영속성(persistence)**은 데이터를 생성한 프로그램의 실행이 종료되더라도 사라지지 않는 데이터의 특성

<br>

### EntityManager

Entity를 필요에 따라 데이터베이스와 동기화하는 역할을 한다

- 영속성 컨텍스트(Persistence Context)라는 Entity 관리 영역이 존재
- 데이터베이스의 캐시와 같은 역할(EntityManager가 작업을 수행하더라도 즉시 데이터베이스에 반영되지는 않는다)
- Entity의 상태를 변경하거나 데이터베이스와의 동기화를 위한 API 제공

**find(entity class, primary key)**

- 기본키로 Entity 검색 후 반환
- 영속성 컨텍스트에서 해당하는 Entity 없을 경우 → SQL Select 쿼리를 날려 데이터를 취득하고 Entity를 생성해서 반환

**persist(entity)**

- 어플리케이션에서 생성한 인스턴스를 Entity로 영속성 컨텍스트에서 관리
- (insert의 경우) persist 메서드를 실행한 시점에 SQL 쿼리가 실제로 실행되지는 않고 영속성 컨텍스트에 축적된다

**flush()**

- 영속성 컨텍스트에 축적된 모든 Entity 변경 정보를 데이터베이스에 강제적으로 동기화
- flush()를 호출하지 않는다면 보통 동기화는, 트랜잭션을 커밋할 때 이루어진다

<br>

## 엔티티 클래스 개발

### OneToOne : 어디다 FK를 두어야 하나?

김영한님 개인적인 판단 기준은 → 얼마나 더 액세스를 많이 하느냐

예를 들어 Delivery와 Order가 1:1 관계라면, 보통은 Order를 통해서 Delivery 정보를 함께 조회하는 일이 많다

따라서 Order가 FK를 가지도록 설계

<br>

### **임베디드 타입(복합 값 타입)**

```java
public class Delivery {
// ..
  private String city;
	private String street;
	private String zipcode;
}
```

대신 ﻿3개의 연관된 값들을 묶어 하나의 클래스로 만든다

- **@Embeddable** : 값 타입을 정의하는 곳에 표시
- **@Embedded** : 값 타입을 사용하는 곳에 표시

```
public class Delivery {
// ..@Embedded
    private Address address;
}

@Embeddable
public class Address {
	private String city;
	private String street;
	private String zipcode;
}
```

**임베디드 타입의 장점**

(+) 다른 클래스에도 재사용이 가능하다 → 예를 들면 Address 클래스를 Delivery 뿐만 아니라 Member 클래스에서도 재사용 가능하다

(+) 연관된 값들을 모아놓아 응집성이 높고, 해당 값 타입에 관련된 메소드만 모아놓을 수있어 관리하기 편리하다

<br>

## 엔티티 설계시 주의할 점

### ManyToMany : 사용하지 말자

- 예를 들어 category_item 테이블에는 FK 두 개 이외에는 부가적인 값을 추가할 수 없다

  운영적인 관점에서도 좋지 않다

- N:N은 1:N과 N:1로 쪼개자

<br>

### JPA : 기본생성자는 필요하다

JPA 스펙상 엔티티(`@Entity`) 혹은 임베디드 타입(`@Embeddable`)은 자바 기본 생성자(`public` 혹은 `protected`)를 필요로 한다

- JPA 구현 라이브러리가 객체를 생성할 때 리플렉션과 같은 기술을 사용하기 때문

<br>

### Enum 사용 시 주의할 점

- `@Enumerated` 어노테이션을 붙여줘라!
- `EnumType.Ordinal` : 정의한 순서대로 인덱스가 매겨지는데 값이 중간에 끼여드는 등 유지 보수 시에 코드에 예상치 못한 사이드 이펙트가 발생할 수 있기 때문에 되도록 사용하지 않는 것이 좋다

```java
@Enumerated(EnumType.STRING)
private OrderStatus status; // 주문상태 [Order, Cancel]
```

<br>

### **모든 연관관계는 지연로딩으로 설정! ⭐️⭐️⭐️**

즉시로딩( EAGER )은 예측이 어렵고, 어떤 SQL이 실행될지 추적하기 어렵다.

특히 JPQL을 실행할 때 N+1 문제가 자주 발생한다. 따라서, 실무에서 **모든 연관관계는 지연로딩( LAZY )으로 설정**해야 한다.

```java
@Entity
public class Order {

		@ManyToOne(fetch = FetchType.EAGER)
		@JoinColumn(name = "member_id")
		private Member member;
		
}
```

위와 같은 경우 order 조회할 때

1. em.find()로 조회 → 괜춘

2. JPQL 사용

   ```sql
   SELECT o FROM order o;
   ```

   - SQL로 그대로 번역, fetch 옵션 다 무시
   - N + 1 문제 발생!
     - 예를 들어서 DB에 order 정보가 총 100개가 있음
     - JPQL을 통해 그대로 번역된 SQL문이 order를 조회하는데 결과가 100개 나옴
     - ...? Eager 발견
     - 이후 각 order 대한 Member 정보를 조회하기 위해 쿼리문을 각각 날린다(총 100번)
     - → N(각 order에 대한 member 조회 쿼리, 총 100개) + 1(전체 order를 조회하기 위한 쿼리문 1개)

```
@ManyToOne` , `@OneToOne`→ 패치 전략 기본 설정이 `EAGER
@OneToMany` → 패치 전략 기본 설정이 `LAZY
```

그래서 `@ManyToOne` , `@OneToOne` 사용할 때는 반드시 직접 `LAZY` 설정을 해줘야한다

<br>

### 연관관계 메소드

원래는 연관관계를 유지하기 위해서 일일이 날코딩 해야하지만

```java
Member member = new Member();
Order order = new Order();

member.getOrders().add(order);
order.setMember(member);
```

대신에 Order 클래스에 아래와 같이 **연관관계 메소드**를 넣어준다

```java
// Order 클래스

@ManyToOne
@JoinColumn(name = "member_id")
private Member member;

@OneToMany(mappedBy = "order")
private List<OrderItem> orderItems = new ArrayList<>();

public void setMember(Member member) {
		this.member = member;
		member.getOrders().add(this);
}

public void addOrderItem(OrderItem orderItem){
		this.orderItem.add(orderItem);
		orderItem.setOrder(this);
}
```

<br>

## 회원 서비스 개발

### `@Transactional` 두 가지 종류

- javax 제공
- 스프링 제공
  - 만약 이미 코드가 스프링에 의존적이라면 스프링에서 제공하는 트랜잭션 어노테이션 사용을 권장
  - javax에서 제공하는 것보다 기능이 더 많다

<br>

### 멀티 쓰레드 문제

```java
@Transactional
    public Long join(Member member) {
        validateDuplicateMember(member); // 중복 회원 검증
        memberRepository.save(member);
        return member.getId();
    }
```

- 트랜잭션 처리를 한다고 해도, 멀티스레드 환경에서 두 스레드 모두 `validateDuplicateMember()`를 통과하는 경우가 생길 수도 있다.
- 때문에 데이터베이스 이름 필드에 unique 제약조건을 걸어서 이중으로 '중복된 이름의 회원이 가입'되는 상황을 방어하는 것이 좋다

<br>

- 