# 대규모 트래픽을 처리하는 API 서버를 설계해야 합니다. 예상되는 트래픽은 초당 수만 건의 요청이며, 사용자가 요청을 보낼 때마다 DB를 직접 조회하여 부하가 심해질 가능성이 있습니다. 이런 상황에서 API 서버의 성능을 최적화하고, 장애 발생 시에도 안정적으로 운영할 수 있는 방법을 설명해주세요.


> 대규모 트래픽을 처리하는 API 서버를 설계할 때는 다음과 같은 최적화 전략이 필요
>
1. **캐싱 (Redis, Local Cache)으로 DB 부하 줄이기**
2. **비동기 처리 (Kafka, WebFlux)로 트랜잭션 효율성 높이기**
3. **DB 최적화 (인덱싱, Read-Replica, 샤딩)**
4. **트래픽 제어 (Rate Limiting, 서킷 브레이커)**
5. **로드 밸런싱 및 수평 확장 (Auto Scaling, Kubernetes)**
6. **모니터링 및 장애 감지 (APM, 로그 분석)**

## 1. **캐싱을 적극 활용하여 DB 부하 감소**

DB 조회를 최소화하고 응답 속도를 높이기 위해 캐싱을 적극 활용한다

### 1.1 **Redis/Memcached 활용**

- 자주 조회되지만 자주 변경되지 않는 데이터(예: 상품 목록, 사용자 정보 등)는 **Redis** 또는 **Memcached**에 캐싱하여 DB 부하를 줄입니다.
- 캐싱할 데이터를 저장할 때는 **TTL(Time-to-Live)** 을 적절히 설정하여 데이터 정합성을 유지합니다.
- API 응답을 직접 캐싱하는 방식도 고려할 수 있습니다.

### 1.2 **Local Cache 활용 (Caffeine, Guava Cache)**

- 서버 내부에서도 빈번히 접근하는 데이터를 **Local Cache**에 저장하여 Redis 접근도 줄일 수 있습니다.
- LRU(Least Recently Used) 또는 TTL을 설정하여 캐시가 무한히 증가하는 것을 방지합니다.

### 1.3 **Write-Through, Write-Behind, Cache-Aside 패턴 적용**

- **Cache-Aside** 패턴: 데이터를 먼저 캐시에서 조회하고 없으면 DB에서 가져와 캐시에 저장하는 방식
- **Write-Through** 패턴: DB에 데이터를 저장할 때 캐시에도 동시에 업데이트
- **Write-Behind** 패턴: 캐시를 먼저 업데이트하고 비동기적으로 DB에 반영

## 2. **비동기 처리 및 큐 활용**

트래픽이 급증할 때 API 서버가 모든 요청을 동기적으로 처리하면 성능 저하가 발생할 수 있습니다.

### 2.1 **Kafka/RabbitMQ 등의 메시지 큐 활용**

- 즉각적인 응답이 필요 없는 요청(로그 저장, 알림 전송 등)은 **Kafka, RabbitMQ** 같은 메시지 큐에 넣고 비동기적으로 처리합니다.
- 트랜잭션이 필요한 경우 **idempotency key**를 활용하여 중복 처리를 방지합니다.

### 2.2 **비동기 API 설계 (CompletableFuture, WebFlux)**

- Java에서는 **CompletableFuture**나 **Spring WebFlux**(Reactor)를 사용하여 비동기 API를 구현할 수 있습니다.
- API가 외부 API 또는 DB를 호출하는 경우, **non-blocking I/O**를 사용하여 성능을 최적화할 수 있습니다.

## 3. **데이터베이스 최적화**

DB를 직접 조회하는 횟수를 줄이고, 트래픽이 집중될 때 성능 저하를 방지합니다.

### 3.1 **DB Connection Pool 사용**

- HikariCP를 사용하여 데이터베이스 커넥션을 효율적으로 관리합니다.
- Connection Pool의 크기를 적절히 조정하여 과부하를 방지합니다.

### 3.2 **인덱스 튜닝 및 쿼리 최적화**

- WHERE 절에 자주 사용하는 컬럼에는 **인덱스**를 추가하여 조회 성능을 향상합니다.
- 쿼리를 실행하기 전에 **EXPLAIN ANALYZE**를 활용하여 실행 계획을 분석합니다.

### 3.3 **데이터 샤딩 및 파티셔닝**

- **Sharding**: 특정 기준(예: 사용자 ID, 지역 등)으로 데이터를 분산 저장하여 단일 DB의 부하를 줄입니다.
- **Partitioning**: 큰 테이블을 여러 개의 파티션으로 나누어 쿼리 성능을 최적화합니다.

### 3.4 **Read-Replica 활용**

- **Master-Slave 구조**를 적용하여 쓰기 연산은 Master DB에서 수행하고, 읽기 연산은 Read-Replica에서 처리합니다.
- **Load Balancer**를 통해 여러 Read-Replica로 트래픽을 분산할 수 있습니다.

## 4. **트래픽 제어 및 장애 대응**

트래픽이 폭주하거나 서버 장애가 발생해도 서비스가 안정적으로 운영될 수 있도록 설계합니다.

### 4.1 **Rate Limiting (속도 제한)**

- API Gateway (Kong, Nginx, AWS API Gateway)에서 **Rate Limiting**을 설정하여 과도한 요청을 차단합니다.
- Redis 기반의 **Token Bucket** 또는 **Leaky Bucket** 알고리즘을 적용하여 특정 사용자의 요청량을 제한할 수 있습니다.

### 4.2 **서킷 브레이커 (Circuit Breaker)**

- 장애 발생 시 전체 서비스가 다운되지 않도록 **Resilience4j, Hystrix** 등의 서킷 브레이커 패턴을 적용합니다.
- 특정 서비스가 일정 시간 동안 장애 상태일 경우 요청을 차단하고, 복구되면 다시 트래픽을 허용합니다.

### 4.3 **Failover 및 장애 감지**

- **Health Check**를 설정하여 특정 인스턴스가 장애 발생 시 자동으로 다른 인스턴스로 트래픽을 전환합니다.
- **Auto Scaling**을 사용하여 트래픽 증가 시 서버를 자동으로 확장합니다.

## 5. **로드 밸런싱 및 서버 확장**

### 5.1 **로드 밸런서 활용**

- **Nginx, AWS ALB(Application Load Balancer)** 등을 사용하여 트래픽을 여러 서버에 분산합니다.
- API Gateway를 통해 인증, 로깅, CORS 처리 등을 중앙에서 관리할 수도 있습니다.

### 5.2 **수평 확장 (Horizontal Scaling)**

- 트래픽이 증가하면 API 서버를 **Auto Scaling Group**으로 설정하여 **자동 확장(Auto Scaling)**이 가능하도록 구성합니다.
- 쿠버네티스를 사용하여 Pod의 개수를 동적으로 조정할 수도 있습니다.

### 5.3 **DNS 기반 로드 밸런싱**

- **Cloudflare, AWS Route 53** 등의 DNS 기반 로드 밸런서를 활용하여 지역별 트래픽을 분산할 수 있습니다.

## 6. **모니터링 및 로깅**

트래픽 변화 및 장애를 실시간으로 감지하고 대응할 수 있도록 모니터링 시스템을 구축합니다.

### 6.1 **APM (Application Performance Monitoring) 도입**

- **Prometheus + Grafana**를 활용하여 **API 응답 시간, 에러율, DB 쿼리 성능**을 모니터링합니다.
- **Jaeger, Zipkin**을 사용하여 **분산 트레이싱**을 적용하면 요청 흐름을 추적할 수 있습니다.

### 6.2 **로그 수집 및 분석**

- **ELK Stack (Elasticsearch, Logstash, Kibana)** 또는 **AWS CloudWatch, Datadog** 등을 사용하여 로그를 수집하고 분석합니다.
- 중요 로그에 대해서는 **Kafka**를 사용하여 별도 저장 후 분석할 수도 있습니다.