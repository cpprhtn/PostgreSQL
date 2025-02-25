# PostgreSQL DB 정보 확인

## Parameter 정보

PostgreSQL의 파라미터 정보는 데이터베이스의 설정을 확인하는 데 사용된다. `SHOW` 명령어로 현재 파라미터 값을 확인할 수 있다.

```sql
SHOW ALL;
```

특정 파라미터 값을 확인하려면:

```sql
SHOW <parameter_name>;
```

예: `SHOW work_mem;`

## 데이터베이스 구성 정보

데이터베이스의 구성 정보를 확인하려면, `pg_database` 시스템 카탈로그를 조회할 수 있다.

```sql
SELECT * FROM pg_database;
```

또는 현재 데이터베이스를 확인하려면:

```sql
SELECT current_database();
```

## 테이블과 컬럼의 코멘트 정보

PostgreSQL에서 테이블과 컬럼에 대한 설명을 확인하려면, `pg_description`과 `pg_class`를 사용할 수 있다.

- 테이블 코멘트 정보 조회:

```sql
SELECT obj_description(relfilenode, 'pg_class') AS table_comment
FROM pg_class
WHERE relname = '<table_name>';
```

- 컬럼 코멘트 정보 조회:

```sql
SELECT column_name, col_description(c.oid, col.attnum) AS column_comment
FROM pg_attribute col
JOIN pg_class c ON col.attrelid = c.oid
WHERE c.relname = '<table_name>';
```

## 테이블 구성 정보

테이블의 구조를 확인하려면, `pg_table`을 조회하거나 `\d` 명령어를 사용할 수 있다.

```sql
\d <table_name>;
```

또는 `information_schema`를 사용하여 테이블 정보를 조회할 수 있다.

```sql
SELECT * FROM information_schema.tables WHERE table_name = '<table_name>';
```

## 인덱스 구성 정보

인덱스 구성 정보를 확인하려면, `pg_indexes` 시스템 카탈로그를 사용한다.

```sql
SELECT * FROM pg_indexes WHERE tablename = '<table_name>';
```

또는 `\di` 명령어로 인덱스 목록을 조회할 수 있다.

```sql
\di
```

## 제약조건 정보

### 종류

| 제약조건        | 설명                                                                      | 예시                                                       |
| --------------- | ------------------------------------------------------------------------- | ---------------------------------------------------------- |
| **NOT NULL**    | 컬럼에 NULL 값이 저장되지 않도록 제한                                     | -                                                          |
| **UNIQUE**      | 특정 컬럼의 값이 테이블 내에서 중복되지 않도록 보장                       | 두 명 이상의 직원이 동일 e-mail 주소를 가질 수 없도록 설정 |
| **PRIMARY KEY** | UNIQUE와 NOT NULL 제약조건을 결합하여, 테이블 내에서 고유한 식별자를 설정 | -                                                          |
| **FOREIGN KEY** | 다른 테이블의 컬럼과 참조 관계를 설정하여 데이터 무결성을 보장            | -                                                          |
| **CHECK**       | 특정 조건이 참인 경우에만 데이터를 허용                                   | 급여 입력 시 0 이상 입력하도록 설정                        |
| **EXCLUSION**   | 특정 연산에 대해 중복을 방지                                              | -                                                          |

테이블의 제약조건 정보는 `pg_constraint`와 `information_schema`를 통해 확인할 수 있다.

```sql
SELECT * FROM information_schema.table_constraints WHERE table_name = '<table_name>';
```

또는 `pg_constraint`를 조회하여 제약조건에 대한 상세 정보를 확인할 수 있다.

```sql
SELECT conname, contype FROM pg_constraint WHERE conrelid = (SELECT oid FROM pg_class WHERE relname = '<table_name>');
```

## 테이블스페이스 구성 정보

PostgreSQL에서 테이블스페이스 관련 정보를 확인하려면 `pg_tablespace` 시스템 카탈로그를 조회한다.

```sql
SELECT * FROM pg_tablespace;
```

또는 테이블이 어떤 테이블스페이스에 저장되어 있는지 확인하려면:

```sql
SELECT tablename, tablespace FROM pg_tables WHERE schemaname = 'public';
```

## 성능 정보

PostgreSQL의 성능 정보를 확인하려면, `pg_stat_activity`, `pg_stat_database`, `pg_stat_user_tables` 등을 사용할 수 있다.

- 현재 세션 상태 확인:

```sql
SELECT * FROM pg_stat_activity;
```

- 데이터베이스 성능 통계:

```sql
SELECT * FROM pg_stat_database;
```

- 테이블 성능 통계:

```sql
SELECT * FROM pg_stat_user_tables;
```

성능 관련 인덱스나 쿼리 성능을 추적하기 위해 `pg_stat_all_indexes`와 `pg_stat_statements`를 사용할 수 있다.

```sql
SELECT * FROM pg_stat_all_indexes WHERE relname = '<table_name>';
```
