# @Aspect
자동 프록시 생성기( `AnnotationAwareAspectJAutoProxyCreator` )는 `Advisor` 를 자동으로 찾아와서 필요한 곳에 프록시를 생성하고 적용해준다.

자동 프록시 생성기는 추가로 하나의 역할을 더 하는데, 이름 앞에 `AnnotationAware` 가 붙어 있듯이 `@Aspect` 를 `Advisor` 로 변환해서 저장하는 기능도 한다. 

<img width="800" alt="image" src="https://github.com/hanuk96/TIL/assets/12428689/3efaca67-2190-4849-8140-cddf8ff82091">

자동 프록시 생성기의 작동 순서

**1. 생성:** 스프링 빈 대상이 되는 객체를 생성한다. ( `@Bean` , 컴포넌트 스캔 모두 포함)

**2. 전달:** 생성된 객체를 빈 저장소에 등록하기 직전에 빈 후처리기에 전달한다.

**3-1. Advisor 빈 조회:** 스프링 컨테이너에서 `Advisor` 빈을 모두 조회한다.

**3-2. @Aspect Advisor 조회:** `@Aspect` 어드바이저 빌더 내부에 저장된 `Advisor` 를 모두 조회한다.

**4. 프록시 적용 대상 체크:** 앞서 3-1, 3-2에서 조회한 `Advisor` 에 포함되어 있는 포인트컷을 사용해서 해당 객체가 프록시를 적용할 대상인지 아닌지 판단한다. 

이때 객체의 클래스 정보는 물론이고, 해당 객체의 모든 메서드를 포인트컷에 하나하나 모두 매칭하여 조건이 하나라도 만족하면 프록시 적용 대상이 된다.

**5. 프록시 생성:** 프록시 적용 대상이면 프록시를 생성하고 프록시를 반환하여 스프링 빈으로 등록한다.

만약 프록시 적용 대상이 아니라면 원본 객체를 반환해서 원본 객체를 스프링 빈으로 등록한다.

**6. 빈 등록:** 반환된 객체는 스프링 빈으로 등록된다.

<img width="800" alt="image" src="https://github.com/hanuk96/TIL/assets/12428689/0b1c9e9c-3527-43f4-bc93-2bedb7f9e734">

스프링 AOP는 AspectJ의 문법을 차용하고, 프록시 방식의 AOP를 제공한다. AspectJ 프레임워크를 직접 사용하는 것이 아니다.

@Aspect를 사용할때, 어드바이스간 순서를 지정하고 싶다면, 클래스 단위로 @Order를 지정해 줄 수 있다. 숫자가 작을수록 먼저 실행된다.

```java
@Slf4j
public class AspectV5Order {

    @Aspect
    @Order(2)
    public static class LogAspect {
        @Around("hello.aop.order.aop.Pointcuts.allOrder()")
        public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
            log.info("[log] {}", joinPoint.getSignature()); //join point 시그니처
            return joinPoint.proceed();
        }
    }

    @Aspect
    @Order(1)
    public static class TxAspect {
        @Around("hello.aop.order.aop.Pointcuts.orderAndService()")
        public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {

            try {
                log.info("[트랜잭션 시작] {}", joinPoint.getSignature());
                Object result = joinPoint.proceed();
                log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());
                return result;
            } catch (Exception e) {
                log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());
                throw e;
            } finally {
                log.info("[리소스 릴리즈] {}", joinPoint.getSignature());
            }
        }
    }
}
```

Advice는 아래와 같은 종류로 나누어진다.

```java
@Aspect
public class AspectAdvice {

    @Around("hello.aop.order.aop.Pointcuts.orderAndService()")
    public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {

        try {
            //@Before
            log.info("[트랜잭션 시작] {}", joinPoint.getSignature());
            Object result = joinPoint.proceed();
            //@AfterReturning
            log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());
            return result;
        } catch (Exception e) {
            //@AfterThrowing
            log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());
            throw e;
        } finally {
            //@After
            log.info("[리소스 릴리즈] {}", joinPoint.getSignature());
        }
    }

    @Before("hello.aop.order.aop.Pointcuts.orderAndService()")
    public void doBefore(JoinPoint joinPoint) {
        log.info("[before] {}", joinPoint.getSignature());
    }

    @AfterReturning(value = "hello.aop.order.aop.Pointcuts.orderAndService()", returning = "result")
    public void doReturn(JoinPoint joinPoint, Object result) {
        log.info("[return] {} return={}", joinPoint.getSignature(), result);
    }

    @AfterThrowing(value = "hello.aop.order.aop.Pointcuts.orderAndService()", throwing = "ex")
    public void doThrowing(JoinPoint joinPoint, Exception ex) {
        log.info("[ex] {} message={}", ex);
    }

    @After(value = "hello.aop.order.aop.Pointcuts.orderAndService()")
    public void doAfter(JoinPoint joinPoint) {
        log.info("[after] {}", joinPoint.getSignature());
    }

}
```

포인트컷을 따로 분리한다.

메서드 이름과 파라미터를 합쳐서 포인트컷 시그니쳐라 한다. 코드 내용은 비워둔다.

포인트컷은 이렇게 조합할 수 있다. `&&` (AND), `||` (OR), `!` (NOT) 3가지 조합이 가능하다.

```java
public class Pointcuts {

    //hello.aop.order 패키지와 하위 패키지
    @Pointcut("execution(* hello.aop.order..*(..))")
    public void allOrder(){} //pointcut signature

    //클래스 이름 패턴이 *Service
    @Pointcut("execution(* *..*Service.*(..))")
    public void allService(){}

    //allOrder && allService
    @Pointcut("allOrder() && allService()")
    public void orderAndService() {}

}
```

각 어드바이스의 호출, 리턴 순서이다.

<img width="800" alt="image" src="https://github.com/hanuk96/TIL/assets/12428689/b8ac7af7-9a8e-4cc4-a8f2-9382235c5236">

`@Around` 하나만 있어도 모든 기능을 수행할 수 있다. 왜 다른 Advice가 존재할까?

```java
 @Around("hello.aop.order.aop.Pointcuts.orderAndService()")
 public void doBefore(ProceedingJoinPoint joinPoint) {
     log.info("[before] {}", joinPoint.getSignature());
 }
```

이 코드는 타켓을 호출하지 않는 문제가 있다.

이 코드를 개발한 의도는 타켓 실행전에 로그를 출력하는 것이다. 그런데 `@Around` 는 반드시 `joinPoint.proceed()` 를 호출해야 한다. 

만약 실수로 호출하지 않으면 타켓이 호출되지 않는 치명적인 버그가 발생한다.

```java
 @Before("hello.aop.order.aop.Pointcuts.orderAndService()")
 public void doBefore(JoinPoint joinPoint) {
     log.info("[before] {}", joinPoint.getSignature());
 }
```

`@Before` 는 `joinPoint.proceed()` 를 호출하는 고민을 하지 않아도 된다.
