# 파티션 테이블

## 파티셔닝 개념 및 종류

- **파티셔닝(Partitioning)**은 대용량 데이터를 여러 개의 작은 단위로 분할하여 관리하는 기법이다.
- **파티션 테이블**은 하나의 테이블을 여러 파티션으로 나누어 저장하고, 데이터를 조회할 때 필요한 파티션만을 효율적으로 선택하여 성능을 향상시킨다.

### 파티션 종류

1. **RANGE 파티셔닝**: 범위에 따라 데이터를 분할한다. 예를 들어, 날짜를 기준으로 분할할 수 있다.
2. **LIST 파티셔닝**: 지정된 값을 기준으로 데이터를 분할한다. 예를 들어, 특정 카테고리나 지역별로 데이터를 분할할 수 있다.
3. **HASH 파티셔닝**: 해시 함수에 의해 데이터를 분할한다. 특정 컬럼 값의 해시 값을 기준으로 데이터를 나눈다.
4. **SUB-PARTITION**: 파티션 테이블 내에서 또 다른 파티셔닝을 적용하여 복잡한 데이터 구조를 관리한다.

## 파티션 테이블 실습(RANGE)

- **RANGE 파티셔닝**은 날짜나 범위를 기준으로 데이터를 나누는 방식이다.

  ```sql
  CREATE TABLE sales (
      id serial PRIMARY KEY,
      amount decimal,
      sale_date date
  ) PARTITION BY RANGE (sale_date);

  CREATE TABLE sales_2021 PARTITION OF sales
      FOR VALUES FROM ('2021-01-01') TO ('2022-01-01');

  CREATE TABLE sales_2022 PARTITION OF sales
      FOR VALUES FROM ('2022-01-01') TO ('2023-01-01');
  ```

## 파티션 테이블 실습(LIST)

- **LIST 파티셔닝**은 특정 값을 기준으로 데이터를 나누는 방식이다.

  ```sql
  CREATE TABLE employees (
      id serial PRIMARY KEY,
      name text,
      department text
  ) PARTITION BY LIST (department);

  CREATE TABLE employees_sales PARTITION OF employees
      FOR VALUES IN ('Sales');

  CREATE TABLE employees_hr PARTITION OF employees
      FOR VALUES IN ('HR');
  ```

## 파티션 테이블 실습(HASH)

- **HASH 파티셔닝**은 해시 함수로 데이터를 나누는 방식이다.

  ```sql
  CREATE TABLE customer_data (
      id serial PRIMARY KEY,
      name text,
      region text
  ) PARTITION BY HASH (id);

  CREATE TABLE customer_data_part1 PARTITION OF customer_data
      FOR VALUES WITH (MODULUS 4, REMAINDER 0);

  CREATE TABLE customer_data_part2 PARTITION OF customer_data
      FOR VALUES WITH (MODULUS 4, REMAINDER 1);
  ```

## 파티션 테이블 실습(SUB-PARTITION)

- **SUB-PARTITION**은 파티션 테이블 내에서 다시 파티셔닝을 적용하는 방식이다.

  ```sql
  CREATE TABLE orders (
      order_id serial PRIMARY KEY,
      order_date date,
      region text
  ) PARTITION BY RANGE (order_date);

  CREATE TABLE orders_2021 PARTITION OF orders
      FOR VALUES FROM ('2021-01-01') TO ('2022-01-01')
      PARTITION BY LIST (region);

  CREATE TABLE orders_2021_europe PARTITION OF orders_2021
      FOR VALUES IN ('Europe');

  CREATE TABLE orders_2021_usa PARTITION OF orders_2021
      FOR VALUES IN ('USA');
  ```

## 파티션 테이블 사용 시 주의사항

- **파티셔닝 설계**: 파티셔닝을 설정할 때는 데이터의 특성에 맞게 파티셔닝 방법을 선택해야 한다.
- **파티션 키 선택**: 파티션 키가 성능에 큰 영향을 미친다. 자주 조회되는 컬럼을 파티션 키로 선택해야 효율적인 쿼리가 가능하다.
- **파티션 관리**: 파티션을 추가하거나 제거하는 작업을 잘 관리해야 한다. 예를 들어, 일정 시간이 지나면 새로운 파티션을 추가하고, 오래된 파티션은 삭제해야 한다.
- **쿼리 최적화**: 파티션을 활용한 쿼리는 파티션 프루닝(Partition Pruning)을 자동으로 활용하여 성능을 향상시킬 수 있다. 하지만 잘못된 파티션 설계는 오히려 성능 저하를 유발할 수 있다.
