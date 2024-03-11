# Point Cut
패턴을 통해 부가 기능을 적용하지 않을지 판단하는 필터링 로직으로 주로 클래스와 메서드 이름으로 필터링 한다.

이를 위해 AspectJ는 포인트컷 표현식을 사용하는데, AspectJ pointcut expression이라고 한다.

- [중요] `execution` : 메소드 실행 조인 포인트를 매칭한다. 스프링 AOP에서 가장 많이 사용하고, 기능도 복잡하다.
- `within` : 특정타입내의 조인 포인트를 매칭한다.
- `args` : 인자가 주어진 타입의 인스턴스인 조인 포인트
- `this` : 스프링 빈 객체(스프링 AOP 프록시)를 대상으로 하는 조인 포인트
- `target` : Target 객체(스프링 AOP 프록시가 가리키는 실제 대상)를 대상으로 하는 조인 포인트
- `@target` : 실행 객체의 클래스에 주어진 타입의 애노테이션이 있는 조인 포인트 `@within` : 주어진 애노테이션이 있는 타입 내 조인 포인트
- `@annotation` : 메서드가 주어진 애노테이션을 가지고 있는 조인 포인트를 매칭
- `@args` : 전달된 실제 인수의 런타임 타입이 주어진 타입의 애노테이션을 갖는 조인 포인트
- `bean` : 스프링 전용 포인트컷 지시자, 빈의 이름으로 포인트컷을 지정한다.

### Execution
execution(접근제어자? 반환타입 선언타입?메서드이름(파라미터) 예외?)
- 메소드 실행 조인 포인트를 매칭한다.
- ?는 생략할 수 있다.
- `*` 같은 패턴을 지정할 수 있다.

가장 많은 제약조건을 둔 execution 표현식

```java
execution(public String hello.aop.member.MemberServiceImpl.hello(String))
```

- 접근제어자: `public`
- 반환타입: `String`
- 선언타입: `hello.aop.member.MemberServiceImpl`
- 메서드이름: `hello`
- 파라미터: `(String)`
- 예외: 생략

가장 많은 기능을 허용한 execution 표현식

```java
execution(* *(..))
```

- `*` 은 아무 값이 들어와도 된다는 뜻이다.
- 파라미터에서 `..` 은 파라미터의 타입과 파라미터 수가 상관없다는 뜻이다.

```java
execution(* hello.aop.member..*.*(..))
```

패키지에서 `.` , `..` 의 차이를 이해해야 한다.
- `.` : 정확하게 해당 위치의 패키지
- `..` : 해당 위치의 패키지와 그 하위 패키지도 포함

```java
public interface MemberService {
    String hello(String param);
}

@Component
public class MemberServiceImpl implements MemberService {

    @Override
    @MethodAop("test value")
    public String hello(String param) {
        return "ok";
    }

    public String internal(String param) {
        return "ok";
    }
}
```

위와 같이 MemberService를 MemberServiceImpl이 상속받았을때

```java
execution(* hello.aop.member.MemberService.*(..)) - (o)
```

위 표현식에서 MemberServiceImpl 또한 매칭된다. 다형성에의해 부모 = 자식이 주입 가능하다.

추가로 부모 타입을 표현식에 선언한 경우 부모 타입에서 선언한 메서드가 자식 타입에 있어야 매칭에 성공한다. 

그래서 부모 타입에 있는 `hello(String)` 메서드는 매칭에 성공하지만, 부모 타입에 없는 `internal(String)` 는 매칭에 실패한다.
