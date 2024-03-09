# @PostConstruct / @PreDestroy

결제승인 로직간 DB에 있는 인증 key값을 가져오는 @Repository Bean이 존재

해당 Bean은 결제를 할때마다 호출하여, 성능개선을 위해 @PostConstruct annotation을 사용
<br><br><br>

spring bean 초기화시 맨 처음 한번만 호출되어, 두 번 이상 호출되지 않는 환경변수와 같은 값을 가져오는 Bean에 사용되면 좋을것 같다 생각하였다.

먼저, 스프링 Bean의 생명주기는 아래와 같다.
<br><br>

```
스프링 컨테이너 생성 => 스프링 Bean의 생성 => 의존성 주입 => 초기화 콜백 => 사용 => 소멸전 콜백 => 스프링 종료
```

(bean 생성자 주입의 경우에는 생성과 주입이 동시에 일어남 - compile time에 에러 확인가능)

Bean의 생명주기에서, @PostConstruct는 Spring application 로딩시 '초기화 콜백'으로 한번 사용된다.

스프링은 `CommonAnnotationBeanPostProcessor` 라는 빈 후처리기를 자동으로 등록하는데, 

여기에서 `@PostConstruct` 애노테이션이 붙은 메서드를 호출한다.
<br><br>
아래의 코드의 UserRepository라는 Bean이 생성되고, 의존성을 주입한뒤 초기화 되는걸 볼 수 있다.

<br>

### @PostConstruct
```java
@Component
public class DbInit {

    @Autowired
    private UserRepository userRepository;

    @PostConstruct
    private void postConstruct() {
        User admin = new User("admin", "admin password");
        User normalUser = new User("user", "user password");
        userRepository.save(admin, normalUser);
    }
}
```

### @PreDestroy
```java
@Component
public class UserRepository {

    private DbConnection dbConnection;
    @PreDestroy
    public void preDestroy() {
        dbConnection.close();
    }
}
```
