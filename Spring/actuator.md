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
