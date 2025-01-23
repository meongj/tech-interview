# 사용하기 편리하고 자원활용 효율적인 API 구조의 특징은 무엇이라고 생각하시는지 설명해주세요. 그리고 본인이 프로젝트에서 적용한 내용을 알려주세요.
<br>

### 1. **사용하기 편리한 API 구조의 특징**

- **일관성(Consistency)**
    - RESTful 원칙을 준수하여 리소스를 명확히 표현 (예: `GET /products/{id}`)
    - 응답 형식(JSON, XML)과 HTTP 상태 코드(200, 400, 404 등)의 일관된 사용
    - 표준적인 명명 규칙 사용 (`/api/v1/users`, `/api/v1/orders`)
- **직관적인 인터페이스(Intuitive Interface)**
    - 직관적인 URL 및 명확한 요청/응답 스펙 제공
    - Swagger/OpenAPI 등을 활용한 문서화 제공
    - 적절한 에러 메시지 및 가이드를 제공하여 디버깅 용이
- **확장성(Extensibility)**
    - 새로운 기능을 쉽게 추가할 수 있도록 버저닝 적용 (예: `/api/v1/`, `/api/v2/`)
    - 비즈니스 로직과 API 계층의 분리
- **보안(Security)**
    - OAuth2, JWT 등의 인증 방식 적용
    - 데이터 검증 및 입력 유효성 검사 적용
    - Rate limiting 및 IP 화이트리스트 적용

<br>

### 2. **자원 활용이 효율적인 API 구조의 특징**

- **최적화된 데이터 전송(Optimized Data Transfer)**
    - 필요한 데이터만 제공 (필드 선택 기능, `?fields=id,name` 쿼리 제공)
    - 페이징 및 정렬 (`?page=1&size=20&sort=createdAt,desc`)을 활용하여 대량 데이터 요청 제한
    - Gzip 압축 적용을 통한 네트워크 비용 절감
- **캐싱 전략 적용(Caching Strategy)**
    - ETag, Last-Modified 헤더를 활용한 클라이언트 측 캐싱 적용
    - Redis와 같은 인메모리 캐시 적용을 통한 DB 부하 감소
- **비동기 처리(Asynchronous Processing)**
    - WebSocket, Kafka, RabbitMQ 등을 활용한 비동기 이벤트 기반 처리
    - 장시간 소요되는 작업에 대한 비동기 API (예: Batch 처리)
- **효율적인 DB 접근(Optimized Database Access)**
    - N+1 문제 방지를 위한 JPA 페치 조인 적용
    - 적절한 인덱스 설정을 통한 성능 최적화
    - 데이터 정규화 및 필요시 CQRS 패턴 적용

<br>

### 3. **내가 프로젝트에서 적용한 사례**

"제가 진행한 프로젝트에서는 API 사용의 편의성과 자원 활용 최적화를 위해 다음과 같은 전략을 적용했습니다."

1. **일관된 RESTful API 설계 및 문서화**
    - `Swagger`를 통해 API 문서를 자동화하여 협업을 원활하게 진행
    - HTTP 상태 코드 및 에러 메시지를 표준화하여 클라이언트의 디버깅 편의성 강화
2. **JWT 인증 및 보안 정책 적용**
    - 로그인 후 `au_token`과 `csrf_token`을 쿠키에 저장하여 보안 강화
    - API Rate Limiting을 적용해 DDoS 방어 강화
3. **캐싱 및 성능 최적화**
    - 자주 조회되는 데이터를 `Redis`를 이용해 캐싱하여 데이터베이스 부하를 감소
    - 페이징 처리 및 인덱스를 적용하여 대량 데이터 조회 성능 개선
4. **비동기 이벤트 기반 처리**
    - `Kafka`를 활용하여 실시간 거래량 집계 및 비동기 데이터 처리
    - 실패한 거래 재처리를 위한 Dead Letter Queue(DLQ) 운영 경험
5. **트래픽 분산 및 모니터링**
    - `Kubernetes`를 활용하여 서비스 트래픽을 효율적으로 분산
    - `Grafana`를 사용하여 API 응답 시간 및 트래픽 모니터링

이러한 접근을 통해 API 응답 시간을 최적화하고, 안정적인 서비스를 제공할 수 있었습니다.