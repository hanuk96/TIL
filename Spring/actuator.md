# actuator
- 지표(metric), 추적(trace), 감사(auditing)
- 모니터링
- 스프링 부트가 제공하는 액추에이터는 이런 프로덕션 준비 기능을 매우 편리하게 사용할 수 있는 다양한 편의 기능들을 제공한다. 
- 마이크로미터, 프로메테우스, 그라파나 같은 최근 유행하는 모니터링 시스템과 매우 쉽게 연동할수 있는 기능도 제공한다.

```
 implementation 'org.springframework.boot:spring-boot-starter-actuator' //actuator gradle에 추가
```
```
http://localhost:8080/actuator/health

{"status": "UP"} 
이 URL을 통해 현재 서비스의 헬스체크가 가능해짐


http://localhost:8080/actuator

이 URL을 통해 actuator가 제공하는 수 많은 기능을 확인할 수 있다.
health 는 헬스 정보를, beans 는 스프링 컨테이너에 등록된 빈을 보여준다
```

- 액츄에이터가 제공하는 기능 하나하나를 엔드포인트라 한다.

### EndPoint 설정
- 엔드포인트를 활성화
  - 엔드포인트를 활성화 한다는 것은 해당 기능 자체를 사용할지 말지 on , off 를 선택하는 것이다.  
- 엔드포인트를 노출
  - 엔드포인트를 노출하는 것은 활성화된 엔드포인트를 HTTP에 노출할지 아니면 JMX에 노출할지 선택하는 것이다.
- 물론 활성화가 되어있지 않으면 노출도 되지 않는다.
- 그런데 엔드포인트는 대부분 기본으로 활성화 되어 있다.( shutdown 제외)

#### health
- health 정보를 사용하면 애플리케이션에 문제가 발생했을 때 문제를 빠르게 인지할 수 있다.
- health 정보는 단순히 애플리케이션이 요청에 응답을 할 수 있는지 판단하는 것을 넘어서
 - 애플리케이션이 사용하는 데이터베이스의 응답
 - 디스크 사용량에는 문제가 없는지
```
management.endpoint.health.show-details=always
management:
 endpoint:
  health:
   show-details: always


http://localhost:8080/actuator/health
```

#### info
- info 엔드포인트는 애플리케이션의 기본 정보를 노출한다.
- 기본으로 제공하는 기능들은 다음과 같다.
 - java: 자바 런타임 정보
 - os: OS 정보
 - env: Environment 에서 info. 로 시작하는 정보
 - build: 빌드 정보, META-INF/build-info.properties 파일이 필요하다.
 - git: git 정보, git.properties 파일이 필요하다

#### logger
- loggers 엔드포인트를 사용하면 로깅과 관련된 정보를 확인하고, 또 실시간으로 변경할 수도 있다.
```java
@Slf4j
@RestController
public class LogController {
@GetMapping("/log")
public String log() {
  log.trace("trace log");
  log.debug("debug log");
  log.info("info log");
  log.warn("warn log");
  log.error("error log");
  return "ok";
 }
}
```
log level이 debug이다.
```yaml
logging:
 level:
  hello.controller: debug
```
아래로 호출시 "hello.controller"의 부분을 보면 debug로 설정되어있는것을 볼 수 있다.
```
http://localhost:8080/log

{
 "levels":[
 "OFF",
 "ERROR",
 "WARN",
 "INFO",
 "DEBUG",
 "TRACE"
 ],
 "loggers":{
 "ROOT":{
 "configuredLevel":"INFO",
 "effectiveLevel":"INFO"
 },
 "_org.springframework":{
 "effectiveLevel":"INFO"
 },
 "hello":{
 "effectiveLevel":"INFO"
 },
 "hello.ActuatorApplication":{
 "effectiveLevel":"INFO"
 },
 "hello.controller":{
 "configuredLevel":"DEBUG",
 "effectiveLevel":"DEBUG"
 },
 "hello.controller.LogController":{
 "effectiveLevel":"DEBUG"
 }
 }
}
```
##### 실시간으로 로그 레벨을 변경
- 그런데 서비스 운영중에 문제가 있어서 급하게 DEBUG 나 TRACE 로그를 남겨서 확인해야 확인하고 싶다면 어떻게 해야할까?
- 일반적으로는 로깅 설정을 변경하고, 서버를 다시 시작해야 한다.
- loggers 엔드포인트를 사용하면 애플리케이션을 다시 시작하지 않고, 실시간으로 로그 레벨을 변경할 수 있다.
```
(꼭! POST를 사용해야 한다.)
POST http://localhost:8080/actuator/loggers/hello.controller
```
content-type은 application/json이며 body는 다음과 같다.
```json
{
 "configuredLevel": "TRACE"
}
```
- 요청에 성공하면 204 응답이 온다.(별도의 응답 메시지는 없다.)
- GET으로 요청해서 확인해보면 configuredLevel 이 TRACE 로 변경된 것을 확인할 수 있다.
```json
GET http://localhost:8080/actuator/loggers/hello.controller

호출 결과
{
 "configuredLevel": "TRACE",
 "effectiveLevel": "TRACE"
}
```

#### HTTP 요청/응답 기록
- HTTP 요청과 응답의 과거 기록을 확인하고 싶다면 httpexchanges 엔드포인트를 사용하면 된다.
- HttpExchangeRepository 인터페이스의 구현체를 빈으로 등록하면 httpexchanges 엔드포인트를 사용할 수 있다.
 - InMemoryHttpExchangeRepository 추가 
```java
@SpringBootApplication
public class ActuatorApplication {
 public static void main(String[] args) {
  SpringApplication.run(ActuatorApplication.class, args);
 }

 @Bean
 public InMemoryHttpExchangeRepository httpExchangeRepository() {
  return new InMemoryHttpExchangeRepository();
 }
}
```
- 이 구현체는 최대 100개의 HTTP 요청을 제공한다.
- 최대 요청이 넘어가면 과거 요청을 삭제한다.
 - setCapacity()로 최대 요청수를 변경할 수 있다.
- 실행 url
```
http://localhost:8080/actuator/httpexchanges
```
- 단순하나 제한이 많다.
- 그래서 실제 운영 서비스에서는 모니터링 툴이나 핀포인트, Zipkin 같은 다른 기술을 사용하는 것이 좋다.

#### actuator와 보안
- 외부 인터넷 망이 공개된 곳에 액츄에이터의 엔드포인트를 공개하는 것은 보안상 좋은 방안이 아니다.
 - 너무 많은 정보를 제공하기 때문
#### 방법
1. 액츄에이터를 다른 포트에서 실행
 - 예를 들어서 외부 인터넷 망을 통해서 8080 포트에만 접근할 수 있고
 - 다른 포트는 내부망에서만 접근할 수 있다면 액츄에이터에 다른 포트를 설정하면 된다.
```yaml
management.server.port=9292
```
url path
```
http://localhost:9292/actuator
```
2. 액츄에이터 URL 경로에 인증 설정
- 포트를 분리하는 것이 어렵고 어쩔 수 없이 외부 인터넷 망을 통해서 접근해야 한다면
- /actuator 경로에 서블릿 필터, 또는 스프링 시큐티리를 통해서 인증된 사용자만 접근 가능하도록 추가 개발이 필요하다.
3. 내부망에서 엔드포인트를 변경
- 엔드포인트의 기본 경로를 변경하려면 다음과 같이 설정하면 된다.
```
management
 endpoints:
  web:
   base-path: "/manage"
```
- /actuator/{엔드포인트} 대신에 /manage/{엔드포인트} 로 변경된다.
