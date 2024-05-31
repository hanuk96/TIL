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
