# ThreadLocal
ThreadLocal이란 해당하는 Thread만 접근할 수 있는 특별한 저장소를 말한다. 

트래픽이 많은 서비스에서 동시성 처리를 위해 ThreadLocal을 사용하거나, Transaction manager의 datasource 정보를 보관하는 등 다양한 요청에서 사용한다.

<img width="600" alt="image" src="https://github.com/hanuk96/TIL/assets/12428689/7c27516c-9c84-4c73-b2ad-2b09bb4a197c">

동시성 이슈는 Spring Bean처럼 Singleton인 instance 값, Static memory영역의 값을 여러 Thread가 동시에 변경하면서 발생한다.

```java
@Slf4j
@NoArgsConstructor
 public class FieldServiceTest {
    //로그를 저장하는 로직
    Private final FieldService fieldService;

    @Test
    void field() {
        log.info("main start");

        //Thread A 실행
        Runnable userA = () -> {
            fieldService.logic("userA");
        };

        //Thread B 실행
        Runnable userB = () -> {
            fieldService.logic("userB");
        };

        Thread threadA = new Thread(userA);
        threadA.setName("thread-A");

        Thread threadB = new Thread(userB);
        threadB.setName("thread-B");

        threadA.start(); //A실행
        threadB.start(); //B실행
        //동시성 문제가 발생
        sleep(3000); //메인 쓰레드 종료 대기
        log.info("main exit");
      }

      private void sleep(int millis) {
         try {
             Thread.sleep(millis);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
    }
}
```

로직상 Thread A의 결과가 Thread B의 결과에 영향을 주어선 안된다. 하지만,

```
[Test worker] main start
[Thread-A] 저장 name=userA -> nameStore=null
[Thread-B] 저장 name=userB -> nameStore=userA
[Thread-A] 조회 nameStore=userB
[Thread-B] 조회 nameStore=userB
[Test worker] main exit
```

결과적으로 Thread A 입장에서는 저장한 데이터와 조회한 데이터가 다른 문제가 발생한다.

<img width="300" alt="image" src="https://github.com/hanuk96/TIL/assets/12428689/64722afc-b457-423f-9dc3-6b3bb646de44">
<img width="300" alt="image" src="https://github.com/hanuk96/TIL/assets/12428689/0df3f271-fef9-4aa7-b7bb-eb7cd039f4ae">
<img width="300" alt="image" src="https://github.com/hanuk96/TIL/assets/12428689/2aab354a-b99b-4ee2-8a2a-97d1b1437db4">
