# Spring Cloud Netflix



### MSA (Microservice Architecture)

단일 응용프로그램으로 나누어 작은 서비스들의 조합으로 구축하는 방법

각각의 서비스 별로 프로젝트가 생기게 된다. 

- 서로 간의 서비스 호출은 기존의 메서드 호출이 아닌 http, REST 방식으로 호출
- 트랜잭션은 어렵다.

MSA는 Cloud 환경과 밀접하게 관련이 있다. 

각각의 서비스를 물리서버에 올리게되면 관리가 힘들기 때문이다. 



### Monolithic Architecture

모든 것이 하나의 프로젝트로 묶여있다. 

일반적으로 MSA가 아니면, Monolithic으로 개발하고 있는 것이다.

- 빌드, 테스트 시간이 길어진다. 
- 하나의 서비스가 모든 서비스에 영향을 준다.
- 개발 언어에 종속적이다.



### Circuit Breaker

일명 누전 차단기, MSA에서도 한 서비스가 다른 서비스의 의존성이 있을 경우 영향을 끼칠 수 있다. 

특히, 하나의 서비스가 비정상적으로 동작할 경우 다른 서비스의 장애 전파하게 된다. 

이럴 때 Circuit breaker를 통해 에러를 핸들링한다. 



### API Gateway

API 서버 앞단에서 모든 API 서버들의 End-point를 단일화하여 묶어주는 역할.

MSA가 각각의 서비스를 API 형태로 제공하게 되면, CORS, 로깅, 모니터링 문제가 발생한다. 

이 문제를 API Gateway를 통해 극복한다. 

- API 요청을 한 곳에서 받아서 해당 서비스로 라우팅을 시켜준다. 
- public / private을 나누어 보안강화
- 서비스들의 공통된 로직을 처리해준다. 예) CORS, 모니터링



### Service Discovery

Auto-scaling을 통해 서버의 인스턴스 개수를 동적으로 관리하게 되는데, 

이를 감지하고 관리하는 역할이다. 

- 동작 과정
  1. 각 서비스가 실행될 때 Service Registry에 ip, port 등의 서버 정보를 저장한다.
  2. Client에서 http요청이 들어온다.
  3. Router는 정해진 시간마다 Service Registry에서 서비스 서버 정보를 가져온다.
  4. Router는 클라이언트 요청을 서비스한다.
  5. 주기적으로 ping을 통해 서버 health를 확인한다.



## Netflix

- Service Discovery : Eureka 인스턴스를 등록할 수 있으며, client는 Spring이 관리하는 빈을 사용하여 탐지할 수 있습니다.
- Service Discovery : 내장된 Eureka Server는 선언적 java config를 통하여 생성될 수 있습니다.
- Circuit Breaker : Hystrix clients는 간단한 어노테이션 기반으로 구축될 수 있습니다.
- Circuit Breaker : 선언적 java config로 내장된 Hystrix dashboard
- 선언적 REST Client : Feign은 JAX-RS 또는 Spring MVC 어노테이션으로 인터페이스를 동적으로 구현합니다.
- Client Side Load Balancer: Ribbon
- External Configuration : Spring 환경에서 Archaius로 연결 (Spring Boot를 사용하여 Netflix 구성 요소를 설정 가능)
- Router and Filter : Zuul 필터의 자동 등록 및 Reverse Proxy 생성 설정 접근에 대한 간단한 규칙



### Zuul

API Gateway로, 외부의 요청은 모두 Zuul로 들어오고 URL에 맞는 서비스를 라우팅 시켜준다. 

동일 서비스가 여러개일 경우, Ribbon이 로드밸런싱 역할을 한다. 



### Filter

로깅, 인증, 모니터링, CORS 등을 제공한다. 

- Authentication and Security
  - 클라이언트 요청시, 각 리소스에 대한 인증 요구 사항을 식별하고 이를 만족시키지 않는 요청은 거부
- Insights and Monitoring
  - 의미있는 데이터 및 통계 제공
- Dynamic Routing
  - 필요에 따라 요청을 다른 클러스터로 동적으로 라우팅
- Stress Testing
  - 성능 측정을 위해 점차적으로 클러스터 트래픽을 증가
- Load Shedding
  - 각 유형의 요청에 대해 용량을 할당하고, 초과하는 요청은 제한
- Static Response handling
  - 클러스터에서 오는 응답을 대신하여 API GATEWAY에서 응답 처리

***필터 종류*** 

- PRE Filter - 라우팅전에 실행되며 필터이다. 주로 logging, 인증등이 pre Filter에서 이루어 진다.

- ROUTING Filter - 요청에 대한 라우팅을 다루는 필터이다. Apache httpclient를 사용하여 정해진 Url로 보낼수 있고, Neflix Ribbon을 사용하여 동적으로 라우팅 할 수도 있다.

- POST Filter - 라우팅 후에 실행되는 필터이다. response에 HTTP header를 추가하거나, response에 대한 응답속도, Status Code, 등 응답에 대한 statistics and metrics을 수집한다.

- ERROR Filter - 에러 발생시 실행되는 필터이다.

### Eureka

URL 해석하여 해당 서비스의 서버 정보를 Zuul에게 전달한다. 



- http://woowabros.github.io/r&d/2017/06/13/apigateway.html
- http://alwayspr.tistory.com/22
