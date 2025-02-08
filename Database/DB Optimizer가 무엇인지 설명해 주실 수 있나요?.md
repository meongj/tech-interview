
# DB Optimizer란?

>데이터베이스에서 SQL 쿼리를 가장 효율적으로 실행할 수 있도록 최적화 역할을 하는 엔진 또는 시스템 <br>
>쿼리 실행 계획을 분석하고, 최적의 방식으로 데이터를 조회 도와주는 핵심 컴포넌트

# 역할

1. 쿼리 실행 계획 생성
    - 사용자가 SQL 을 실행하면, 옵티마이저가 내부적으로 어떤 방식으로 데이터를 가져올지 결정함
    - 인덱스 사용할지, 풀테이블 스캔 할지, 조인을 어떻게 수행할지
2. 비용 기반 최적화  (CBO)
    - 옵티마이저는 비용을 계산해서 가장 적은 비용으로 데이터 조회할 방법 선택
    - where 절 조건이 많을 경우, 가장효율적인 필터링 방법 선택
    - 요즘 대부분 RBO 보다 CBO 사용
3. 규칙 기반 최적화 (RBO)
    - 미리 정해진 규칙(예. 인덱스 항상 먼저사용 ) 기반으로 실행 계획 결정
4. 인덱스 및 조인 방식 최적화
    - 옵티마이서는 인덱스 사용할지, 테이블 스캔 할지 결정
    - 여러 개의 테이블을 조인할 때, **Nested Loop Join, Hash Join, Merge Join** 중 어떤 방식을 사용할지 선택함.

# DB Optimizer 작동과정

1. 사용자가 SQL 쿼리 실행
2. 옵티마이저가 가능한 실행 계획 여러개 생성
3. 각 실행 계획 비용 계산 (CBO)
4. 가장 효율적인 실행계획 선택해 쿼리 실행

# 옵티마이저의 최적화 방법 (쿼리 튜닝 기법)

> 옵티마이저가 자동으로 최적화하지만, 개발자가 직접 개선할 수도 있다
>
1. 인덱스 활용
    - `where` 조건에 자주 사용되는 컬럼에 인덱스를 추가하면 성능개선
        - 풀 스캔 대신 인덱스 스캔 사용

        ```java
        CREATE INDEX idx_users_age ON users (age);
        ```

2. 실행계획 확인
    - `Explain` 실행계획을 보고 옵티마이저가 어떤 방식으로 데이터 조회하는지 분석 가능
        - Index 스캔 사용되는지 확인

        ```java
        EXPLAIN ANALYZE SELECT * FROM users WHERE age > 30;
        ```

3. 조인 최적화
    - 옵티마이저는 여러 가지 조인 방식 중 가장 적절한 방법을 선택
    - `Join` 시 인덱스를 걸어두면 속도 향상 가능

    ```java
    CREATE INDEX idx_orders_user_id ON orders (user_id);
    ```

4. 서브쿼리 대신 Join 사용
    - 서브쿼리는 성능이 낮을 수 있으므로, Join 을 활용하는 것이 효율적

    ```java
    -- ❌ 비효율적인 서브쿼리
    SELECT name FROM users WHERE id IN (SELECT user_id FROM orders);
    
    -- ✅ JOIN 사용하여 최적화
    SELECT users.name FROM users JOIN orders ON users.id = orders.user_id;
    ```


# 주요 데이터베이스의 옵티마이저

- **MySQL** → **MySQL Query Optimizer**
- **PostgreSQL** → **PostgreSQL Query Planner & Optimizer**
- **Oracle** → **Oracle Cost-Based Optimizer (CBO)**
- **SQL Server** → **SQL Server Query Optimizer**

# DB Optimizer가 중요한 이유?

- **SQL을 빠르게 실행할 수 있도록 자동으로 최적화함**
- **인덱스, 조인 방식, 실행 계획 등을 분석해서 최적의 방법을 선택함**
- **개발자가 직접 튜닝하면 성능을 더욱 향상시킬 수 있음**

👉 **즉, DB Optimizer는 SQL 쿼리를 "최소한의 비용"으로 실행할 수 있도록 도와주는 엔진**