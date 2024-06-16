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
기본 기능
- Table Evaluation time 을 수정해서 과거 시간 조회 가능
- Graph 메트릭을 그래프로 조회 가능

필터
- 레이블을 기준으로 필터를 사용할 수 있다. 필터는 중괄호( {} ) 문법을 사용한다.
- uri=/log , method=GET 조건으로 필터
  - http_server_requests_seconds_count{uri="/log", method="GET"}
- /actuator/prometheus 는 제외한 조건으로 필터
  - http_server_requests_seconds_count{uri!="/actuator/prometheus"}

연산자 쿼리와 함수
- 다음과 같은 연산자를 지원한다.
```
+ (덧셈)
- (빼기)
* (곱셈)
/ (분할)
% (모듈로)
^ (승수/지수)
```

sum
- 값의 합계를 구한다.
예) sum(http_server_requests_seconds_count)

sum by
- sum by(method, status)(http_server_requests_seconds_count)
- SQL의 group by 기능과 유사하다.

결과
```json
{method="GET", status="404"} 3
{method="GET", status="200"} 120
```

count
- count(http_server_requests_seconds_count)

메트릭 자체의 수 카운트
- topk
- topk(3, http_server_requests_seconds_count)
- 상위 3개 메트릭 조회

오프셋 수정자
- http_server_requests_seconds_count offset 10m
- offset 10m 과 같이 나타낸다. 현재를 기준으로 특정 과거 시점의 데이터를 반환한다.

범위 벡터 선택기
- http_server_requests_seconds_count[1m]
- 마지막에 [1m] , [60s] 와 같이 표현한다. 지난 1분간의 모든 기록값을 선택한다.
- 참고로 범위 벡터 선택기는 차트에 바로 표현할 수 없다. 데이터로는 확인할 수 있다.

메트릭은 크게 보면 게이지와 카운터라는 2가지로 분류할 수 있다.
#### 게이지(Gauge)
- 임의로 오르내릴 수 있는 값
  - 예) CPU 사용량, 메모리 사용량, 사용중인 커넥션
- 값이 계속 변하는 게이지는 현재 값을 그대로 그래프로 표현하면 된다.

#### 카운터(Counter)
- 단순하게 증가하는 단일 누적 값
  - 예) HTTP 요청 수, 로그 발생
- 값이 단조롭게 증가하는 카운터는 increase() , rate() 등을 사용해서 표현하면 된다.
- 이렇게 하면 카운터에서 특정 시간에 얼마나 고객의 요청이 들어왔는지 확인할 수 있다.
 
