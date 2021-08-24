### 도메인 모델 패턴

- 엔티티가 대부분의 비즈니스 로직을 담당
- 객체지향의 특성 활용
- 서비스 계층은 엔티티에 필요한 요청을 위임
- repository 없이 엔티티에 대한 단위 테스트를 할 수 있다.
- https://martinfowler.com/eaaCatalog/domainModel.html



### 트랜잭션 스크립트 패턴

- 서비스 계층이 대부분의 비즈니스 로직을 처리
- 엔티티에는 비즈니스 로직이 거의 없음

- https://martinfowler.com/eaaCatalog/transactionScript.html

참고: https://javacan.tistory.com/entry/94



### 생성 메서드

- 생성자를 protected로 선언해주어 생성메서드 외의 부분에서 객체를 생성하는 것을 방지하자.

```java
//==생성 메서드==//
 public static Order createOrder(Member member, Delivery delivery,OrderItem... orderItems) {
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

>  Tip! 생성 메서드, 비즈니스 로직, 조회 메서드 분리하여 작성하자.



### cascade

- 라이프 사이클이 동일하고 다른 엔티티가 참조하지 않을 때 쓰는 것이 좋다. 
- 여러 엔티티가 참조하는 경우에는 별도의 repository를 생성하는 것이 좋다.



### Validation

- 에러가 있어도 form 데이터는 유지된다. 
- Thymeleaf와 spring의 integration

```java
 @PostMapping(value = "/members/new")
 public String create(@Valid MemberForm form, BindingResult result) {
     if (result.hasErrors()) {
     return "members/createMemberForm";
     }
     Address address = new Address(form.getCity(), form.getStreet(), form.getZipcode());
     Member member = new Member();
     member.setName(form.getName());
     member.setAddress(address);
     memberService.join(member);
     return "redirect:/";
 }

```





### 준영속 엔티티

- 영속성 컨텍스트가 더는 관리하지 않는 엔티티

```java
 Book book = new Book();
 book.setId(form.getId());
 book.setName(form.getName());
 book.setPrice(form.getPrice());
 book.setStockQuantity(form.getStockQuantity());
 book.setAuthor(form.getAuthor());
 book.setIsbn(form.getIsbn());
```



### Merge

- 모든 속성(필드를) 변경 
  - DB에 null로 업데이트 될 위험성

```java
    public void save(Item item) {
        if (item.getId() == null) {
            em.persist(item);
        } else {
            em.merge(item);
        }
    }
```

1. merge(item) 실행
2. item의 id를 가져와 1차 캐시(영속성 컨텍스트)에서 엔티티를 조회, 없다면 DB에서 가지고 와 1차 캐시에 저장한다.
3. 파라미터로 넘어온 item의 데이터를 mergeItem에 채워넣는다.
4. 영속 상태인 mergeItem을 반환한다.

> 정리하면 영속 엔티티를 조회한 뒤 영속 엔티티의 값을 준영속 엔티티의 값으로 교체(병합)한다. 그리고 트랜잭션 커밋 시점에 변경 감지로 인해 DB에 update sql문이 날아간다.



### Dirty Checking

- 원하는 속성만 선택하여 변경

- JPA는 데이터를 변경하고 나면 알아서 트랜잭션 커밋 시점에 변경을 감지하여(dirty checking) 데이터베이스에 업데이트 쿼리를 날려준다.

```java
@Transactional
void update(Item itemParam) { //itemParam: 준영속 엔티티
    Item findItem = em.find(Item.class, itemParam.getId()); // 같은 엔티티를 조회
    findItem.setPrice(itemParam.getPrice()); 
}
```



### Model Attribute

```java
    @GetMapping("/orders")
    public String orderList(@ModelAttribute("orderSearch") OrderSearch orderSearch, Model model) {
        List<Order> orders = orderService.findOrders(orderSearch);
        model.addAttribute("orders", orders);

        return "order/orderList";
    }
```



### 정리

- 한 프로젝트 내에서도 도메인 모델 패턴과 트랜잭션 스크립트 패턴이 양립할 수 있다.

- 데이터를 가지고 있는 쪽에 비즈니스 메서드가 있는 것이 좋다. e.g. addStock(quantity)

- 엔티티는 핵심 비즈니스 로직만 가지고 있고, 화면을 위한 로직은 없어야 한다.
- 엔티티를 변경할 때는 항상 변경 감지를 사용해야 한다.

- 상태 변경 시 set 메서드 말고 의미있는 메서드를 사용해야 추적이 쉽다.

- 트랜잭션 안에서 비즈니스 로직을 실행해야 영속 상태를 유지할 수 있음
  - 컨트롤러에서 조회한 뒤 서비스로 객체를 넘기면 영속 상태가 끝나버린 객체가 넘어간다.

