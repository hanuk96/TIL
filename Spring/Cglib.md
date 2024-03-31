# Cglib
Code Generator Library
- 바이트코드를 조작해서 동적으로 클래스를 생성하는 기술을 제공하는 라이브러리이다.
- CGLIB를 사용하면 인터페이스가 없어도 구체 클래스만 가지고 동적 프록시를 만들어낼 수 있다.

우리가 CGLIB를 직접 사용하는 경우는 거의 없다. 스프링의 `ProxyFactory` 가 이 기술을 편리하게 사용하게 도와준다.

구체 클래스

```java
@Slf4j
 public class ConcreteService {
  public void call() {
    log.info("ConcreteService 호출");
  }
}
```

Cglib를 적용한 Interceptor로, `proxy.invoke(target, args)` 를 이용해 실제 대상을 동적으로 호출한다.

참고로 `method` 를 사용해도 되지만, CGLIB는 성능상 `MethodProxy proxy` 를 사용하는 것을 권장한다.

```java
@Slf4j
public class TimeMethodInterceptor implements MethodInterceptor {

    private final Object target;

    public TimeMethodInterceptor(Object target) {
        this.target = target;
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        Object result = methodProxy.invoke(target, args);

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;

        log.info("TimeProxy 종료 resultTime={}", resultTime);
        return result;
    }
}

```

프록시를 생성하는 예제

```java
public void cglib() {
        ConcreteService target = new ConcreteService();

        Enhancer enhancer = new Enhancer();

        //어떤 구체클래스를 입력받을지 명시
        enhancer.setSuperclass(ConcreteService.class);
        //프록시에 적용할 로직 할당
        enhancer.setCallback(new TimeMethodInterceptor(target));
        ConcreteService proxy = (ConcreteService) enhancer.create();
        log.info("targetClass={}", target.getClass());
        log.info("proxyClass={}", proxy.getClass());

        proxy.call();

}
```

결과

```
CglibTest - targetClass=class hello.proxy.common.service.ConcreteService
CglibTest - proxyClass=class hello.proxy.common.service.ConcreteService$
$EnhancerByCGLIB$$25d6b0e3
TimeMethodInterceptor - TimeProxy 실행
ConcreteService - ConcreteService 호출 TimeMethodInterceptor - TimeProxy 종료 resultTime=9
```
