# 프로메테우스와 그라파나
### 프로메테우스
- 애플리케이션에서 발생한 메트릭을 그 순간만 확인하는 것이 아니라 과거 이력까지 함께 확인하려면 메트릭을 보관하는 DB가 필요하다.
- 이렇게 하려면 어디선가 메트릭을 지속해서 수집하고 DB에 저장해야 한다. 프로메테우스가 바로 이런 역할을 담당한다.
### 그라파나
- 프로메테우스가 DB라고 하면, 이 DB에 있는 데이터를 불러서 사용자가 보기 편하게 보여주는 대시보드가 필요하다. 
- 그라파나는 매우 유연하고, 데이터를 그래프로 보여주는 툴이다.
- 수 많은 그래프를 제공하고, 프로메테우스를 포함한 다양한 데이터소스를 지원한다.

![image](https://github.com/hanuk96/TIL/assets/12428689/3de5834b-9e04-4af2-bd8c-d15544eb988e)

#### 프로메테우스 - application 설정
1. application 설정: 프로메테우스가 애플리케이션의 메트릭을 가져갈 수 있도록 애플리케이션에서 프로메테우스 포멧에 맞추어 메트릭 만들기
2. 프로메테우스 설정: 프로메테우스가 우리 애플리케이션의 메트릭을 주기적으로 수집하도록 설정

- application 설정하기
```gradle
implementation 'io.micrometer:micrometer-registry-prometheus' //추가

프로메테우스 endpoint
http://localhost:8080/actuator/prometheus
```
포멧 차이
- jvm.info -> jvm_info : 프로메테우스는 . 대신에 _ 포멧을 사용한다. . 대신에 _ 포멧으로 변환된 것을 확인할 수 있다.
- logback.events -> logback_events_total : 로그수 처럼 지속해서 숫자가 증가하는 메트릭을 카운터라 한다.
  - 프로메테우스는 카운터 메트릭의 마지막에는 관례상 _total 을 붙인다.
- http.server.requests 이 메트릭은 내부에 요청수, 시간 합, 최대 시간 정보를 가지고 있었다. 프로메테우스에서는 다음 3가지로 분리된다.
  - http_server_requests_seconds_count : 요청 수
  - http_server_requests_seconds_sum : 시간 합(요청수의 시간을 합함)
  - http_server_requests_seconds_max : 최대 시간(가장 오래걸린 요청 수)
 
- 프로메테우스 설정하기
- 프로메테우스는 /actuator/prometheus를 1초에 한번씩 호출해서 애플리케이션의 메트릭들을 수집한다.
```yaml
global:
 scrape_interval: 15s
 evaluation_interval: 15s

alerting:
 alertmanagers:
 - static_configs:
 - targets:
 # - alertmanager:9093

rule_files:
scrape_configs:
 - job_name: "prometheus"
 static_configs:
 - targets: ["localhost:9090"]

 #추가
 - job_name: "spring-actuator" # 수집하는 이름(임의로 지정)
   metrics_path: '/actuator/prometheus' # 수집할 경로
   scrape_interval: 1s # 수집할 주기
   static_configs: # 수집할 서버의 IP / PORT
     - targets: ['localhost:8080']
```

#### 프로메테우스 - 기본 기능
