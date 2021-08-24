# 상품 도메인 개발

## 상품 엔티티 개발(비즈니스 로직 추가)



> 상품 리포지토리 개발

```java
    public void save(Item item) {
        if (item.getId() == null) {
            em.persist(item);
        } else {
            em.merge(item); // 업데이트 비슷한 개념
        }
    }
```

## 상품 서비스 개발 

> 상품서비스는 상품 레포지토리에 위임만 하는 단순한 서비스
> 경우에 따라서는 위임만 하는 건 서비스 따로 만들지 않고, 
> 컨트롤러에서 레포지토리에 바로 접근하는 것도 괜찮을 수 있음.



---

# 주문 도메인 개발

Order 엔티티는 여러 연관관계가 들어가고 복잡함

각 필드를 Setter 메서드로 개별 지정하면 관리 포인트가 늘어나므로, 별도의 통합된 생성 메서드가 있다면 좋음.

```java
public static Order createOrder(Member member, Delivery delivery, OrderItem... orderItems) {
        Order order = new Order();
        order.setMember(member);
        order.setDelivery(delivery);
        for (OrderItem orderItem : orderItems) {
            order.addOrderItem(orderItem);
        }
        order.setStatus(OrderStatus.ORDER);
        order.setOrderDate(LocalDateTime.now());
        return order;
}
```

- 생성메서드를 사용하면, 깔끔하고 좋은데 이거 외에도 생성자로 생성할 수도 있으니 막기위해 생성자를 protected 처리해줌.

## 주문 서비스 개발

>  cascade 범위 고민, 
>  order 같은 경우에 delivery와 orderItem을 모두 관리하는데 이런 상태에만 cascade를 사용해야함.



cascade

- 라이프 사이클이 동일하고, 다른 엔티티가 참조하지 않을 때 쓰는 것이 좋다.
- 여러 엔티티가 참조하는 경우에는 별도의 repository를 생성하는 것이 좋다.

```java
@NoArgsConstructor(access = AccessLevel.PROTECTED)
```

```java
@Transactional
public void cancelOrder(Long orderId){
    //주문 엔티티 조회
    Order order = orderRepository.findOne(orderId);
    //주문 취소
    order.cancel();
}

```

JPA의 경우 엔티티의 데이터를 변경하면, `더티 체킹(변경된 감지)`이 일어나면서 변경된 쿼리가 자동으로 날아감.

- 추후 공부필요



```
> 참고: 

주문 서비스의 주문과 주문 취소 메서드를 보면 비즈니스 로직 대부분이 엔티티에 있다. 

서비스 계층은 단순히 엔티티에 필요한 요청을 위임하는 역할을 한다. 
이처럼 엔티티가 비즈니스 로직을 가지고 객체 지향의 특성을 적극 활용하는 것을 도메인 모델 패턴(http://martinfowler.com/eaaCatalog/
domainModel.html)이라 한다. 


반대로 엔티티에는 비즈니스 로직이 거의 없고 서비스 계층에서 대부분
의 비즈니스 로직을 처리하는 것을 트랜잭션 스크립트 패턴(http://martinfowler.com/eaaCatalog/
transactionScript.html)이라 한다.
```

### 도메인 모델 패턴, 트랜잭션 스크립트 패턴

> 마틴 파울러가 소개한 패턴

### 도메인 모델 패턴

> 엔티티가 비즈니스 로직을 가지고 객체 지향의 특성을 적극 활용하는 것
>
> 각 객체에 객체가 수행해야하는 업무를 분담

- 장점
  - 객체간 관계를 맺을 수 있어, 제약하거나 로직의 단순화에 도움이됨.
  - 유지보수가 편리하며, 재사용성이 뛰어나다.
  - repository 없이 엔티티에 대한 단위 테스트를 할 수 있다.
- 단점
  - 객체들간의 꼼꼼한 설계가 필요해, 모델 구축이 쉽지 않다.

### 트랜잭션 스크립트 패턴

>  엔티티에 비즈니스 로직 X
>
> 서비스 계층에서 대부분의 비즈니스 로직 처리

- 장점

  - 구현이 매우 쉽다.
  - 강건한 로직이 해당 모듈에서만 구현되어야할 경우 side effect 를 예방할 수 있고, 좀 더 효율적인 코드를 작성할 수 있다.
  -  많은 개발자들의 몸에 익은 최적의 개발 방법중의 하나

- 단점

  - 서비스 메서드가 비대해지며, 테스트하기 어려워짐.
  - 도메인에 대한 분석/설계 개념이 약하기 때문에 코드의 중복 발생을 막기 어려워진다.
  - 구조가 복잡해질 수록 모듈화의 복잡도가 높아진다.

  

>두 패턴 중 하나가 좋다 나쁘다를 말하는게 아니라, 상황에 따라 유리한 걸 선택

## 주문 검색 기능 개발

### JPA 동적쿼리

- JPQL

  - 쿼리를 문자로 생성해야함.
  - 휴먼에러 가능성 O

- JPA Criteria

  - JPQL을 자바 코드로 작성하도록 도와주는 빌더 클래스 API

  - 자바 코드로 쿼리를 작성하므로, 문법 오류를 컴파일 단계에서 잡을 수 있음.

  - JPA 표준스펙이지만, 코드가 복잡하고, 직관적으로 이해하기 어려운 단점이 존재

    - QueryDSL

    

## 변경 감지와 병합(merge)

> 정말 중요한 개념

- ...



---



메모 



mapstruct

json non null







----

### 참고

https://javacan.tistory.com/entry/94

https://jins-dev.tistory.com/entry/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D%EC%9D%98-2%EA%B0%80%EC%A7%80-%EB%A1%9C%EC%A7%81-%ED%8C%A8%ED%84%B4-%EB%8F%84%EB%A9%94%EC%9D%B8-%EB%AA%A8%EB%8D%B8Domain-Model-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8Transaction-Script