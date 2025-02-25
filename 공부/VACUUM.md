# VACUUM 개요

## 읽기 일관성(MVCC)

- **MVCC(Multi-Version Concurrency Control)**는 트랜잭션에서 변경된 데이터가 조회될 때 일관된 데이터를 보장하는 메커니즘이다.
- PostgreSQL에서는 데이터가 변경되면, 변경 전과 후의 데이터를 각각 별도의 **Tuple**로 기록한다. 변경된 데이터의 이전 버전과 최신 버전이 동시에 존재할 수 있으며, 이를 통해 트랜잭션 간의 일관성을 유지한다.
- 각 **Tuple**은 생성 시점과 삭제 또는 업데이트 시점을 나타내는 **XMIN**과 **XMAX** 값을 가지고, 이를 통해 해당 **Tuple**의 상태를 추적한다. **XMIN**은 트랜잭션 ID가 해당 데이터의 생성 시점을, **XMAX**는 해당 데이터가 삭제되거나 업데이트된 시점을 기록한다.

## VACUUM 개념

- PostgreSQL에서는 데이터가 삭제되거나 업데이트되더라도 해당 **Tuple**은 실제 테이블에서 물리적으로 제거되지 않는다. 이러한 **Dead Tuple**은 **VACUUM** 작업이 실행될 때까지 테이블에 남아 있으며, **VACUUM**은 이들을 정리하는 역할을 한다.
- **VACUUM**은 **Dead Tuple**을 제거하고, 테이블의 공간을 회수하여 디스크 공간을 효율적으로 관리한다. 또한, **VACUUM**은 테이블의 성능을 유지하고, 디스크 I/O를 최적화하는 데 중요한 역할을 한다.

## FSM과 VM

- **FSM (Free Space Map)**

  - **FSM**은 테이블과 인덱스의 여유 공간을 추적하는 맵이다. 이 정보는 데이터를 **INSERT**하거나 **UPDATE**할 때 여유 공간이 있는 페이지를 빠르게 찾아내어 디스크 I/O를 줄이는 데 도움이 된다.
  - **VACUUM** 작업 시에도 **FSM**은 빈 공간에 대한 정보를 제공하여 **VACUUM**이 효율적으로 작업을 수행할 수 있도록 돕는다.

- **VM (Visibility Map)**
  - **VM**은 각 테이블 페이지가 **Dead Tuple**을 포함하고 있는지 여부를 기록하는 맵이다. **VACUUM** 작업 시 **VM**을 사용하면 **Dead Tuple**이 존재하는 페이지만을 선택적으로 처리하여 효율성을 높일 수 있다.

# VACUUM과 데이터베이스 성능

## VACUUM이 필요한 이유

- **Dead Tuple 정리**  
  PostgreSQL에서는 데이터를 **DELETE**하거나 **UPDATE**하면, 실제로는 물리적으로 데이터를 삭제하지 않는다. 대신, 새로운 **Tuple**을 삽입하고 이전 버전은 "Dead Tuple"로 남겨둔다. 이러한 **Dead Tuple**이 계속 쌓이면 디스크 공간이 낭비되고, 테이블이 커질수록 쿼리 성능이 저하된다. **VACUUM**은 이러한 불필요한 **Dead Tuple**을 정리하여 테이블의 크기를 줄이고, 성능을 최적화한다.

- **디스크 공간 회수**  
  **VACUUM**은 **Dead Tuple**을 제거하고, 테이블의 공간을 회수하여 디스크 공간을 효율적으로 사용할 수 있도록 돕는다. 이를 통해 테이블이 비대해지지 않도록 관리할 수 있다.

## Wraparound 방지

- **트랜잭션 ID Wraparound**  
  PostgreSQL은 **XMIN**과 **XMAX**를 통해 **Tuple**의 생성 및 삭제 시점을 관리하는데, 이 값들은 트랜잭션 ID와 연관된다. 트랜잭션 ID는 **32비트**로 표현되며, 시간이 지나면서 값이 증가하다가 결국 **Wraparound**가 발생한다. 이때 트랜잭션 ID가 0으로 다시 돌아가게 되며, 이전의 트랜잭션 ID 값과 충돌할 수 있다.
- **Wraparound 문제 해결**  
  **VACUUM**은 트랜잭션 ID의 **Wraparound**를 방지하기 위해 일정 간격으로 실행되어야 한다. **VACUUM**을 주기적으로 실행하면, 이전 트랜잭션 ID를 처리하고, **Wraparound** 문제를 예방할 수 있다. 그렇지 않으면 **Tuple**을 잘못된 트랜잭션 ID로 인식할 수 있어 데이터 무결성 문제가 발생할 수 있다.

## DISK I/O 성능 개선

- **디스크 공간 최적화**  
  **VACUUM**은 사용되지 않는 **Dead Tuple**을 제거하고, 빈 공간을 회수하여 디스크 공간을 재사용 가능하게 만든다. 이로 인해 데이터베이스는 디스크에서 데이터를 읽을 때 불필요한 공간을 스캔하는 일을 줄여 **I/O 성능**을 향상시킨다.
- **디스크 페이지 재사용**  
  **VACUUM**은 빈 페이지를 식별하고 이를 재사용 가능하게 만들어, 향후 **INSERT**나 **UPDATE**가 발생할 때 더 적은 디스크 I/O가 발생하도록 돕는다. 빈 페이지를 재사용함으로써 새로운 페이지를 할당할 필요가 없어져 I/O를 최소화할 수 있다.

- **읽기 성능 향상**  
  **VACUUM** 작업 후, 불필요한 데이터가 제거되며, 실제로 디스크에 저장된 데이터는 유효한 데이터만 포함하게 된다. 이로 인해 쿼리 시 불필요한 데이터를 읽는 일을 방지하고, 디스크 I/O 효율성이 높아져 쿼리 성능이 향상된다.

# Auto VACUUM과 VACUUM FULL

## Auto VACUUM

- **Auto VACUUM**은 PostgreSQL의 자동 관리 기능으로, 데이터베이스 내에서 **Dead Tuple**을 정리하고 디스크 공간을 회수하기 위한 작업을 자동으로 수행한다. 이 기능은 기본적으로 활성화되어 있으며, **VACUUM** 작업을 수동으로 실행하지 않아도 주기적으로 테이블을 최적화할 수 있다.
- **동작 방식**
  - **Auto VACUUM**은 **autovacuum** 데몬이 데이터베이스를 모니터링하면서, 지정된 기준에 따라 **VACUUM**을 자동으로 실행한다.
  - **autovacuum**은 **Dead Tuple**이 일정 수치 이상 쌓이거나, 테이블 크기가 일정 크기 이상이 되면 자동으로 실행된다. 이때, 특정 테이블에 대한 **VACUUM**을 수행하고, 공간을 회수하며, **Wraparound** 문제를 방지한다.
  - Auto VACUUM은 **VACUUM** 작업을 여러 테이블에 대해 병렬로 수행하므로, 전체 데이터베이스 성능에 미치는 영향을 최소화하도록 설계되어 있다.
- **Auto VACUUM의 구성 요소**
  - **autovacuum_vacuum_threshold**: 테이블에 대해 **VACUUM**을 실행할 때, Dead Tuple의 최소 수치가 이 값 이상이어야 한다.
  - **autovacuum_vacuum_scale_factor**: 테이블에 대해 **VACUUM**을 실행할 때, 테이블의 크기 대비 Dead Tuple의 비율이 이 값 이상이어야 한다.
  - **autovacuum_freeze_max_age**: 트랜잭션 ID의 **Wraparound**를 방지하기 위한 기준으로, 이 값에 도달하면 **VACUUM**이 강제로 실행된다.
  - **autovacuum_naptime**: **Auto VACUUM**이 실행될 최소 간격을 설정하는 값이다. 이 시간 간격마다 **Auto VACUUM**이 확인된다.
- **Auto VACUUM의 장점**
  - 사용자가 직접 **VACUUM** 작업을 관리할 필요 없이, **Dead Tuple**을 자동으로 정리할 수 있다.
  - 데이터베이스의 성능 저하를 예방하고, **Wraparound** 문제를 미리 방지한다.
  - 성능을 유지하면서 데이터베이스 관리를 자동화한다.

## VACUUM FULL

- **VACUUM FULL**은 **VACUUM**과 동일한 역할을 하지만, **Full Table Lock**을 걸고 실행되며, 테이블의 빈 공간을 완전히 압축하여 디스크 공간을 회수한다. **VACUUM FULL**은 **Dead Tuple**뿐만 아니라, 테이블 내의 실제 물리적 빈 공간까지 제거하므로 더 많은 공간을 확보할 수 있다.
- **차이점**
  - **VACUUM**은 테이블에서 **Dead Tuple**만 제거하고, 물리적 공간은 그대로 남겨둔다. 그러나 **VACUUM FULL**은 빈 공간을 압축하여 실제 물리적 공간을 회수한다.
  - **VACUUM FULL**은 테이블에 **Lock**을 걸기 때문에 **VACUUM**에 비해 실행 시간이 길고, 다른 작업이 테이블에 접근할 수 없게 된다.
- **사용 시 주의사항**
  - **VACUUM FULL**은 디스크 공간을 회수하고 테이블의 크기를 줄이는 데 매우 유용하지만, 실행 중에 **Exclusive Lock**을 걸기 때문에 다른 트랜잭션이 해당 테이블에 접근할 수 없게 된다. 따라서 데이터베이스의 가용성을 낮출 수 있다.
  - 대규모 테이블에서 **VACUUM FULL**을 실행하면 성능 저하가 발생할 수 있으며, 특히 운영 중인 시스템에서 실행하기 어려울 수 있다.

## Auto VACUUM과 VACUUM FULL 실습

### Auto VACUUM 설정 확인

```sql
SHOW autovacuum;
```

- `autovacuum` 설정 값이 `on`으로 되어 있는지 확인한다.

### Auto VACUUM의 동작

1. **Auto VACUUM 작업이 자동으로 실행되는 예시**:

   - 대개 PostgreSQL은 주기적으로 **Auto VACUUM**을 실행하므로, 특별히 직접 실행할 필요는 없다.
   - 하지만 **Auto VACUUM**의 로그를 확인하여 실행되는 시점을 볼 수 있다.

   ```sql
   SELECT * FROM pg_stat_all_tables WHERE last_autovacuum > now() - interval '1 day';
   ```

### VACUUM FULL 실행

1. **VACUUM FULL**을 실행하여 테이블의 공간을 압축하고 회수하기:

   ```sql
   VACUUM FULL my_table;
   ```

2. **VACUUM FULL**은 실행 중에 **Exclusive Lock**을 걸기 때문에, 실행 전 해당 테이블에 대한 잠금 상태를 확인:
   ```sql
   SELECT * FROM pg_locks WHERE relation = 'my_table'::regclass;
   ```

### Auto VACUUM의 효율성

- Auto VACUUM은 **Dead Tuple**을 자동으로 정리하고, **Wraparound** 문제를 예방하는 데 중요한 역할을 한다. 예를 들어, **autovacuum_vacuum_scale_factor**가 낮게 설정된 경우, 적은 **Dead Tuple**이라도 **VACUUM**이 자주 실행될 수 있다.
- **Auto VACUUM**의 성능을 최적화하려면 **autovacuum_vacuum_threshold**와 **autovacuum_vacuum_scale_factor**를 적절하게 설정하고, 필요한 경우 **VACUUM FULL**을 수동으로 실행해 테이블의 크기를 최적화할 수 있다.
