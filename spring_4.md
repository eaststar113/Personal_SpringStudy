## 스프링 핵심 원리 이해2 - 객체 지향 원리 적용


<img width="816" height="391" alt="image" src="https://github.com/user-attachments/assets/151dc32c-f90f-49fe-b197-388a75d052a5" />

```java
package hello.core.discount;
import hello.core.member.Grade;
import hello.core.member.Member;
public class RateDiscountPolicy implements DiscountPolicy {
    private int discountPercent = 10; //10% 할인
    @Override
    public int discount(Member member, int price) {
        if (member.getGrade() == Grade.VIP) {
            return price * discountPercent /  100;
        } else {
            return 0;
        }
    }
}
```

* @Override : 부모 클래스나 인터페이스가 가진 기능을 재정의해서 사용하겠다는 뜻, 인터페이스의 규칙을 따르는 것, 다형성


```java
public class OrderServiceImpl implements OrderService {
//    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
      private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
}
```
<img width="836" height="503" alt="image" src="https://github.com/user-attachments/assets/9002bc0a-7a06-4e25-95ee-5dc026d6f70c" />

<img width="851" height="580" alt="image" src="https://github.com/user-attachments/assets/94726d70-36a9-4dbf-9f69-0b1362a1f543" />

<img width="812" height="428" alt="image" src="https://github.com/user-attachments/assets/72f3c3ce-883f-4752-98c3-18f8a9a3d9e3" />

<hr>

```java
package hello.core.order;
import hello.core.discount.DiscountPolicy;
import hello.core.discount.FixDiscountPolicy;
import hello.core.member.Member;
import hello.core.member.MemberRepository;
import hello.core.member.MemoryMemberRepository;

public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository = new MemoryMemberRepository();
    private final DiscountPolicy discountPolicy = new FixDiscountPolicy(); ## 직접 본인이 할당 이게 문제임

    @Override
    public Order createOrder(Long memberId, String itemName, int itemPrice) {
      Member member = memberRepository.findById(memberId);
      int discountPrice = discountPolicy.discount(member, itemPrice);
      return new Order(memberId, itemName, itemPrice, discountPrice);
    }
}
```
역할의 분리가 필요 비즈니스 로직은 비즈니스 로직만 담당하듯이


```java
package hello.core;

import hello.core.discount.FixDiscountPolicy;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import hello.core.member.MemoryMemberRepository;
import hello.core.order.OrderService;
import hello.core.order.OrderServiceImpl;
public class AppConfig {

  public MemberService memberService() {
    return new MemberServiceImpl(new MemoryMemberRepository());
  }
  public OrderService orderService() {
    return new OrderServiceImpl(
    new MemoryMemberRepository(),
    new FixDiscountPolicy());
  }
}
    
```
* appconfig : 객체의 생성과 연결을 담당

```java
package hello.core.member;

public class MemberServiceImpl implements MemberService {

  private final MemberRepository memberRepository;

  public MemberServiceImpl(MemberRepository memberRepository) {
    this.memberRepository = memberRepository;  ## 인터페이스에만 의존하고 어떤 구현 객체가 들어올지 appconfig(외부)에서 결정되기에 생성자 주입이라고 함, DIP 완성, 다형성에 의해서 뭔가가 들어옴
  }
  public void join(Member member) {
    memberRepository.save(member);
  }
  public Member findMember(Long memberId) {
    return memberRepository.findById(memberId);
  }
}
```
* 생성자 : 객체가 생성될 때 딱 한 번 호출되는 특수한 메서드, 객체가 만들어질 때 호출되는 초기화 함수
* 생성자 주입: 생성자를 통해서 이 객체가 일하는 데 필요한 다른 객체(의존성)를 전달받는 방식. (의존관계 주입을 위한 방법)
* 의존관계 주입(DI): "부품을 밖에서 넣어준다"는 전략.

