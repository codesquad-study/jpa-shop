# 실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발

## 1. 😀 준영속 엔티티

`영속성 엔티티에서 관리하지 않는 엔티티`를 말합니다. 예시로 기존 식별자를 가지고 있는 영속성 엔티티를 직접 new 생성자를 통해 인스턴스화 및 해당 id를 할당한 엔티티는 준영속 엔티티에 해당됩니다.

```java
//이미 Book Entity에 1번이 할당되어 있는 상태
//이미 존재하는 엔티티의 Id를 임의로 할당하였다. = 준영속 엔티티
Book book = new Book();

book.setId(form.getId());
book.setName(form.getName());
book.setPrice(form.getPrice());
book.setStockQuantity(form.getStockQuantity());
book.setAuthor(form.getAuthor());
book.setIsbn(form.getIsbn());
```

<br><br>

## 2. 준영속 엔티티 수정 방법 : 변경 감지와 병합

 ### (1) 변경 감지(Dirty Checking)

-  영속성 엔티티를 호출해 값을 변경합니다. @Transactional 에노테이션을 선언한 경우, 해당 메서드가 정상 작동 시, 트랜잭션을 커밋합니다. JP커밋한 시점을 기준으로 JPA는 flush() 메서드를 실행하고 **변경 내역만을 감지해 최신 내용으로 영속성 엔티티의 값을 변경**시킵니다.
- `flush()` : 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영한다.

<br>

### (2) 병합 사용

- 병합은 준영속 상태의 엔티티를 영속 상태로 변경시 사용하는 기능입니다. 엔티티의 **모든 필드를 변경**하는 메서드입니다.

- 세부 로직

  1. merge(updateMember) 메서드 호출 시, 1차 캐시(없으면 DB)에서 엔티티를 호출합니다.
  2.  호출한 엔티티의 값을 updateMember의 값으로 호출한 엔티티의 값을 모두 변경시키고 값이 변경된 엔티티를 반환합니다.

  ![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/d0fb1836-bbb9-40fd-a2c0-cfb98bf07f00/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210824%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210824T094343Z&X-Amz-Expires=86400&X-Amz-Signature=5cdf1a2a902b781d795df09a0d24e6d7012953051ef74c7d21abe9642d05ba7a&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

<br>

### (3) 변경 감지와 병합의 차이점

- setter는 하나의 필드만 변경하는 메서드입니다. 해당 메서드를 사용할 경우, 변경하는 로직의 응집도가 낮아지고 유지보수를 하는데 어려움이 있습니다. 이를 해결하기 위해서는 의미있는 메서드를 선언해 응집도를 높히고 유지보수를 편리할 수 있도록 합시다!

```java
public void change(int price, String name, int stockQuanty){
	setPrice(price);
	setName(name);
	setStockQuantity(stockQuantity):
}
```

<br><br>

## 3. Cascade Option

### (1) cascade ?

- **연관된 엔티티도 함께 영속 상태를 만들고 싶은 경우** 사용하는 옵션입니다.  즉, 엔티티 저장할 때 자식 엔티티도 함께 저장이 가능합니다.

- Cascade option을 추가하지 않은. 경우 : saveNoCascade 메서드

  ```java
  private static void saveNoCascade(EntityManager em) {
  	Parent parent = new Parent();
  	em.persist(parent);
  
  	Child child1 = new Child();
  	child1.setParent(parent);
  	parent.getChildren().add(child1);
  	em.persist(child1);
  
  	Child child2 = new Child();
  	child2.setParent(parent);
  	parent.getChildren().add(child2);
  	em.persist(child2);
  }
  ```
  <br>

- Cascade option을 추가한 경우 : saveWithCascade 메서드

  ```java
  private static void saveWithCascade(EntityManager em) {
  	Parent parent = new Parent();
  
  	Child child1 = new Child();
  	Child child2 = new Child();
  
  	child1.setParent(parent);
  	child2.setParent(parent);
  
  	parent.getChildren().add(child1);
  	parent.getChildren().add(child2);
  
  	em.persist(parent);	
  }
  ```

  <br>

### (2) CascadeType

```java
public enum CascadeType {
	ALL, //모두 적용
	PERSIST,//영속
	MERGE, // 병합
	REMOVE, //삭제 : flush()를 호출할 때 전이가 발생한다.
	REFRESH, //REFRESH
	DETACH //DETACH
}
```

<br><br>

## 4. DTO를 사용하자

**Q. Entity 대신 DTO를 사용 이유를 말해주세요.**

만약 Entity를 그대로 사용할 경우, **도메인의 모든 속성이 외부에 노출**됩니다. 도메인 객체의 경우, 해당 UI에서 사용하지 않을 **불필요한 데이터까지 보유**하고 있는 경우가 있기 때문입니다. 또한 User와 같은 민감한 정보가 외부에 노출되면 치명적인 보안 문제에 부딪힐 수 있습니다. 하지만 DTO를 사용하게 될 경우, Entity의 모든  속성을 반환하지 않아도 되기 때문에 불필요한 데이터를 외부에 노출하지 않을 수 있습니다.

또한 Entity를 사용할 경우, UI계층의 상태 변경을 유발합니다. Domain의 요구사항의 변경으로 내부 필드를 변경할 경우, View와 강하게 결합해 Front-End의 JSON과 js code을 추가적으로 변경해야 합니다. 이는 유지보수 면에서 치명적입니다. 하지만 **DTO를 사용하면 View와의 결합도가 줄어듭니다.** 이는 Domain Field를 변경하더라도 View ㅣLayer의 코드 변경에 영향을 미치지 않는 것을 의미합니다.

<br><br>

## 5. RequestBody vs ModelAttribute

### 1. RequestBody

- JSON, XML, Text 등 데이터가 적합한 `HttpMessageConverter`로 파싱하여 Java 객체로 변환한다.

- @RequestBody를 사용할 객체는 필드를 바인딩할 생성자나 setter가 필요없다.

  - 왜냐하면, Objectmapper를 통해 JSON값을 Java 객체에 역직렬화하기 때문이다.

    (역직렬화 : 생성자를 거치지 않고 리플렉션을 통해 객체를 구성하는 메커니즘이다.)

  - 하지만, 직렬화를 하기 위해 기본 생성자가 필수로 필요하고 데이터 바인딩을 위한 getter나 setter 중 한가지는 정의 되어있어야 한다.

<br>

### 2. ModelAttribute

- Http 파라미터들 데이터를 Java Object로 맵핑한다.
- 객체 필드에 데이터를 바인딩할 수 있는 생성자 또는 setter 메서드가 필요하다.
- Query String 및 Form 형식이 아닌 데이터는 처리할 수 없다.

<br><br>

## 6. 기타 등등

**Q1. default constructor를 사용하지 않고 create method 한가지 방식을 사용한 이유에 대해서 설명해주세요.**

**Ans.** 기본 생성자를 생성해 setter를 이용할 경우, `객체를 생성 및 초기화하는 로직이 분산`됩니다. 로직이 분산된다는 이야기는 코드가 흩어진다는 이야기이며 이는 유지보수의 어려움을 유발합니다. 이를 방지하기 위해 `create` method를 선언하고 기본 생성자를 `protected`로 변경하였습니다. 이를 사용할 경우, 생성 및 초기화의 로직의 응집도를 높힐 수 있습니다.

<br>

**Q2. `도메인 모델 패턴`과  `트랜잭션 스크립트 패턴`에 대해서 간략히 설명해 주세요.**

**Ans.** 도메인 모델 패턴은  **서비스** 계층은 단순히 **엔티티에 필요한 요청을 위임하는 역할**을 하고 **핵심 비즈니스 로직**을 **엔티티에게 할당**하는 패턴입니다. 반면, 트랜잭션 스크립트 패턴은 엔티티에 **비즈니스 로직이 거의 없고 서비스 계층에서 대부분의 비즈니스 로직을 처리하는 방식**을 말합니다.

<br><br>

## Reference

- [Javable] DTO의 사용 범위에 대하여 : https://www.notion.so/a4e657e9231a4f0686a32ebe435d0bde#318a78438b684d82991cae638ed285c2
- [Javable] RequestBody vs ModelAttribute : https://woowacourse.github.io/tecoble/post/2021-05-11-requestbody-modelattribute/
- [Spring docs] MappingJackson2HttpMessageConvertor : https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/converter/json/MappingJackson2HttpMessageConverter.html

- @RequestBody와 HttpEntity : https://hooongs.tistory.com/90