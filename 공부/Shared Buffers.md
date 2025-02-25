### PostgreSQL Memory Architecture

PostgreSQL의 메모리 구조는 크게 **공유 메모리(Shared Memory)** 와 **로컬 메모리(Local Memory)** 로 나눌 수 있습니다. 각각의 메모리는 데이터 처리 속도를 최적화하고 시스템 성능을 향상시키는 역할을 합니다.

## **1. Shared Memory (공유 메모리)**

공유 메모리는 PostgreSQL 프로세스 간에 공유되며, 주요 데이터 및 트랜잭션 로그를 효율적으로 관리하는 역할을 합니다.

### **1.1 Shared Buffers**

- 디스크 I/O를 줄이기 위한 캐시 역할을 수행.
- PostgreSQL에서 테이블 및 인덱스 페이지를 메모리에 저장하여 성능을 향상.
- `shared_buffers` 설정을 통해 크기 조절 가능 (`postgresql.conf`에서 설정).
- OS의 파일 시스템 캐시와 별도로 동작.

### **1.2 WAL Buffers**

- Write-Ahead Logging(WAL) 데이터가 기록되기 전에 임시로 저장되는 버퍼.
- WAL은 장애 발생 시 복구를 위해 필요한 로그 데이터이며, 성능 최적화를 위해 일정 크기만큼 메모리에 유지됨.
- `wal_buffers` 값이 크면 WAL 기록이 효율적으로 이루어질 수 있음.

### **1.3 CLOG Buffers (Commit Log Buffers)**

- 트랜잭션 커밋 상태를 저장하는 공간.
- 트랜잭션이 성공적으로 수행되었는지 확인할 때 사용.
- `pg_xact` 디렉터리에 저장되는 커밋 로그와 연계되어 동작.

### **1.4 Lock Space**

- PostgreSQL의 동시성 제어를 위해 사용되는 락 정보를 저장하는 공간.
- 트랜잭션 간 충돌을 방지하고, 데이터 무결성을 유지하는 역할.
- 다중 트랜잭션 환경에서 효율적인 데이터 접근을 지원.

## **2. Local Memory (로컬 메모리)**

로컬 메모리는 개별 백엔드 프로세스마다 할당되는 메모리 공간으로, 특정 쿼리 실행이나 세션 정보 저장에 사용됩니다.

### **2.1 Maintenance Work Memory**

- `VACUUM`, `CREATE INDEX`, `ALTER TABLE` 등 유지보수 작업에 사용되는 메모리.
- `maintenance_work_mem` 설정을 통해 크기 조정 가능.
- 인덱스 생성 및 Autovacuum의 효율성을 높이기 위해 적절한 크기로 설정 필요.

### **2.2 Temporary Buffers**

- 세션별로 생성되는 임시 테이블(Temporary Table)을 저장하는 버퍼.
- 일반적인 `work_mem` 과 달리 개별 세션에서만 사용됨.
- `temp_buffers` 설정을 통해 크기 조정 가능.

### **2.3 Work Memory**

- 정렬(Sort), 해시 조인(Hash Join), 집계 연산(Aggregation) 등에서 사용되는 작업 메모리.
- `work_mem`이 작으면 디스크 스필(Spill)이 발생하여 성능 저하가 발생할 수 있음.
- 설정 예시:
  ```sql
  SET work_mem = '64MB';
  ```

### **2.4 Catalog Cache**

- 시스템 카탈로그 정보를 저장하는 캐시.
- 테이블, 인덱스, 사용자 정보 등의 메타데이터를 빠르게 조회하기 위해 사용됨.
- PostgreSQL의 내부 동작 최적화를 위해 중요한 요소.

### **2.5 Optimizer / Executor Memory**

- 쿼리 실행 계획을 생성하고 실행하는 데 필요한 메모리.
- 최적화된 쿼리 실행을 위해 내부적으로 사용됨.
- 쿼리 계획(Cache Plan) 유지로 재사용 가능한 계획을 빠르게 실행 가능.

## **3. PostgreSQL 메모리 설정 예시**

```ini
# 공유 메모리 설정
shared_buffers = 2GB      # Shared Buffers 크기 조정
wal_buffers = 16MB        # WAL Buffers 크기 조정

# 로컬 메모리 설정
maintenance_work_mem = 512MB
work_mem = 64MB
temp_buffers = 8MB
```
