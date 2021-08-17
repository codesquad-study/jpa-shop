## CQRS

CQRS란 Command and Query Responsibility Segregation의 약자로 command와 query를 분리하는 패턴이다.

명령은 상태 변경의 작업을 하고, 조회는 상태 반환의 작업을 한다. 

이런 관점에서 보면 생성 등 command를 실행하는 메서드는 void 타입을 린턴하게 하여 상태 변경에 대한 책임만 가지도록 관리하는 것이 좋다. 

차선: id만 리턴하도록 설계

- https://docs.microsoft.com/ko-kr/azure/architecture/patterns/cqrs
- https://justhackem.wordpress.com/2016/09/17/what-is-cqrs/



## 상속 관계 매핑

객체의 상속 관계를 데이터베이스에 맵핑하는 방법

관계형 데이터베이스에는 상속이라는 개념이 없기 때문에 ORM([Object–relational mapping](https://en.wikipedia.org/wiki/Object–relational_mapping), 객체와 관계형 데이터베이스의 데이터를 자동으로 연결해주는 것)에서의 상속 관계 매핑은 객체의 상속 구조와 데이터베이스의 슈퍼 타입 - 서브 타입 관계를 매핑하는 것이다.

#### JOINED

엔티티 각각을 테이블로 만들고 자식 테이블이 부모 테이블의 기본 키를 받아서 기본 키 + 외래 키로 사용하는 전략

조회할 때 주로 사용

테이블에는 타입이 없기 때문에 타입을 구분하는 컬럼을 추가해야 한다.

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED) // 부모 클래스에 명시
@DiscriminatorColumn // 기본값 = DTYPE
public class Animal {
    @Id
    private long animalId;
    private String species;

    // constructor, getters, setters 
}

@Entity
@DiscriminatorValue("A") // 엔티티를 저장할 때 구분 컬럼에 입력할 값
@PrimaryKeyJoinColumn(name = "petId") // 자식테이블의 기본 키 컬럼명은 부모 테이블의 id 컬럼명을 기본으로 사용하지만, 변경하고 싶다면 이 어노테이션을 사용하면 된다.
public class Pet extends Animal {
    private String name;

    // constructor, getters, setters
}
```



#### SINGLE_TABLE

전략 설정 안하면 기본적으로 JPA에 의해 선택되는 전략이다.

테이블을 하나만 사용하고 구분 컬럼(DTYPE)으로 어떤 자식 데이터가 저장되었는지 구분하는 전략

조인을 사용하지 않으므로 속도가 가장 빠르다. But, 단일 테이블이 너무 커지만 상황에 따라 조회 성능이 느려질 수도 있다. 

자식 엔티티가 매핑한 컬럼은 모두 null을 허용해야 한다. 

@DiscriminatorValue를 지정 안 하면 엔티티 이름이 값으로 설정된다. 

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
public class MyProduct {
    @Id
    private long productId;
    private String name;

    // constructor, getters, setters
}
```



#### TABLE_PER_CLASS

자식 엔티티마다 테이블을 만드는 전략인데 잘 사용 안함

자식 테이블을 통합해서 쿼리하기 어렵고 여러 자식 테이블을 함께 조회할 때 성능이 저하됨

```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public class Vehicle {
    @Id
    private long vehicleId;

    private String manufacturer;

    // standard constructor, getters, setters
}
```



- https://www.baeldung.com/hibernate-inheritance



## 컬렉션의 초기화

> 컬렉션은 필드에서 바로 초기화 하는 것이 안전하다. null 문제에서 안전하다. 하이버네이트는 엔티티를 영속화 할 때, 컬랙션을 감싸서 하이버네이트가 제공하는 내장 컬렉션으로 변경한 다. 만약 getOrders() 처럼 임의의 메서드에서 컬력션을 잘못 생성하면 하이버네이트 내부 메커니즘에 문 제가 발생할 수 있다. 따라서 필드레벨에서 생성하는 것이 가장 안전하고, 코드도 간결하다

```java
    @ManyToMany
    @JoinTable(name = "category_item",
            joinColumns = @JoinColumn(name = "category_id"),
            inverseJoinColumns = @JoinColumn(name = "item_id"))
    private List<Item> items = new ArrayList<>();
```

```java
    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "parent_id")
    private Category parent;

    @OneToMany(mappedBy = "parent")
    private List<Category> child = new ArrayList<>();
```



 ## Value Object

- @Embedded, @Embeddable 중 하나만 있어도 동작한다.

https://github.com/janeljs/jane-daily-books/blob/main/DDD%20START!/Chapter%201/%EC%97%94%ED%8B%B0%ED%8B%B0%EC%99%80%20%EB%B0%B8%EB%A5%98.md

> @Setter 를 제거하고, 생성자에서 값을 모두 초기화해서 변경 불가능한 클래스를 만들자. JPA 스펙상 엔티티나 임베디드 타입( @Embeddable )은 자바 기본 생성자(default constructor)를 public 또는 protected 로 설정해야 한다. public 으로 두는 것 보다는 protected 로 설정하는 것이 그나마 더 안전 하다. > JPA가 이런 제약을 두는 이유는 JPA 구현 라이브러리가 객체를 생성할 때 리플랙션 같은 기술을 사용할 수 있도록 지원해야 하기 때문이다



## 연관관계

- 외래 키가 있는 주문을 연관관계의 주인으로 정하는 것이 좋다
- 주인이 아닌 쪽은 조회용, 거울
- 주인을 업데이트해야 값이 업데이트 된다.
- 연관관계 메서드
  - 양방향 연관관계는 연관관계의 주인을 정해줘야 한다. 양쪽 모두 값을 바꿀 수 있으면 혼란스러움
  - 위치는 control 하는쪽이 들고있는게 좋다. 주인쪽

```java
    //==연관관계 메서드==//
    public void setMember(Member member) {
        this.member = member;
        member.getOrders().add(this);
    }

    public void addOrderItem(OrderItem orderItem) {
        orderItems.add(orderItem);
        orderItem.setOrder(this);
    }

```



### 영속성 컨텍스트

`@PersistenceContext `: 엔티티 메니저( EntityManager ) 주입

`@PersistenceUnit` : 엔티티 메니터 팩토리( EntityManagerFactory ) 주입

- persist = 트랜잭션이 커밋되는 시점에 디비에 반영됨

  - persist해도 insert가 바로 안나감 database transaction이 commit될 때 insert되는 것

    em.flush() 하면 insert 쿼리는 나가고 그 이후에 롤백이 일어남

    @Rollback(false) 설정 가능

- find = 단건 조회 , 타입, pk

- jpql = from의 대상이 엔티티

- 파라미터 바인딩

```java
    public List<Member> findByName(String name) {
        return em.createQuery("select m from Member m where m.name = :name", Member.class)
                .setParameter("name", name)
                .getResultList();
    }
```



## @Transactional

- EntityManager를 통한 모든 데이터 변경은 transaction 안에서 일어나야 한다.

- 클래스보다 메서드 레벨이 먼저 적용된다.
- DB에서 멤버이름 unique constraint로  설정하여 최후 방어를 해야한다.

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@Transactional(readOnly = true) // 데이터의 변경이 없는 읽기 전용 메서드에 사용, 영속성 컨텍스트를 플러시 하지 않으므로 약간의 성능 향상(읽기 전용에는 다 적용)
public class MemberService {
    
    @Transactional
    public Long join(Member member) {
        validateDuplicateMember(member); 
        memberRepository.save(member);
        return member.getId();
    }

}
```



### fail 메서드

```java
 @Test(expected = IllegalStateException.class)
 public void 중복_회원_예외() throws Exception {
     fail("예외가 발생해야 한다.");
 }

```



### 메모리 디비 사용

- 아무 설정 없어도 jdbc:h2:mem:testdb로 동작함
- 테스트 시 사용하자

