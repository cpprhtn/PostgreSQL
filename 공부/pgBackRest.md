## pgBackRest를 이용한 물리적 백업과 복구

### pgBackRest란?

**pgBackRest**는 PostgreSQL 데이터베이스의 고급 백업 및 복구 솔루션입니다. 주로 대규모 환경에서 데이터베이스 백업과 복구를 효율적으로 관리하는 데 사용됩니다. **pgBackRest**는 **WAL(Write Ahead Log)** 아카이브 및 물리적 백업을 지원하며, 복구 시점으로 돌아가는 기능인 **PITR(Point-in-Time Recovery)**을 제공하고, 백업과 복구의 성능을 최적화합니다.

### pgBackRest의 장점

- **고성능 백업**: 병렬 백업을 지원하여 대규모 데이터베이스 환경에서 성능을 극대화할 수 있다.
- **효율적인 WAL 관리**: **WAL 아카이브** 및 **복원**을 효율적으로 관리할 수 있다.
- **자동화된 백업**: 스케줄링 및 자동화 기능을 제공하여 정기적인 백업을 쉽게 설정할 수 있다.
- **다양한 복구 옵션**: **Point-in-Time Recovery (PITR)**을 지원하여, 특정 시점으로 복원할 수 있다.
- **암호화 및 압축**: 백업 파일을 압축하거나 암호화하여 저장할 수 있다.

### pgBackRest 백업 방법

1. **설치 및 설정**

   - `pgBackRest`를 설치한 후, 설정 파일을 수정하여 백업을 사용할 수 있도록 구성해야 합니다. 기본 설정 파일은 `/etc/pgbackrest/pgbackrest.conf`입니다.

   ```bash
   [global]
   repo1-path=/var/lib/pgbackrest
   log-level-console=info
   log-level-file=debug

   [db]
   pg1-path=/var/lib/postgresql/data
   ```

   - `repo1-path`: 백업 파일을 저장할 디렉토리
   - `pg1-path`: PostgreSQL 데이터베이스의 데이터 디렉토리

2. **백업 실행**

   - **전체 백업**: 전체 백업을 수행하려면 다음 명령어를 사용합니다.

   ```bash
   pgbackrest --stanza=db --type=full backup
   ```

   - `--stanza`: 데이터베이스 설정을 정의한 스탠자 이름
   - `--type=full`: 전체 백업을 수행

   - **증분 백업**: 증분 백업은 이전 백업 이후 변경된 부분만 백업합니다.

   ```bash
   pgbackrest --stanza=db --type=incr backup
   ```

   - `--type=incr`: 증분 백업

3. **WAL 아카이브 활성화**

   `pgBackRest`는 WAL 파일을 자동으로 아카이브할 수 있습니다. 이를 위해서는 `archive_mode`와 `archive_command`를 설정해야 합니다.

   ```bash
   archive_mode = on
   archive_command = 'pgbackrest --stanza=db archive-push %p'
   ```

   - `archive_command`: WAL 파일을 아카이브 디렉토리에 보냅니다.

### pgBackRest 복구 방법

1. **복원 준비**

   - 복원을 시작하기 전에, 필요한 백업 파일과 WAL 파일들이 준비되어 있어야 합니다. 필요한 경우 `pgBackRest`의 **WAL 아카이브** 기능을 통해 원하는 시점까지의 로그 파일을 가져올 수 있습니다.

2. **복원 실행**

   - **전체 복원**: 전체 백업을 복원하려면 다음 명령어를 사용합니다.

   ```bash
   pgbackrest --stanza=db restore
   ```

   - **PITR(Point-in-Time Recovery)**: 특정 시점으로 복구하려면 `recovery.conf` 파일을 생성하고, 원하는 복구 시점에 해당하는 WAL 로그를 복원합니다.

   ```bash
   restore_command = 'pgbackrest --stanza=db archive-get %f %p'
   recovery_target_time = '2025-02-24 10:00:00'
   ```

   - `restore_command`: 아카이브된 WAL 파일을 복원하는 명령어
   - `recovery_target_time`: 복구하고자 하는 시점

3. **복구 완료 후 처리**

   - 복원 후, `recovery.conf` 파일을 삭제하거나 이름을 변경하여 **PostgreSQL**이 복구 모드에서 정상 모드로 전환되도록 해야 합니다.

   ```bash
   mv /var/lib/postgresql/data/recovery.conf /var/lib/postgresql/data/recovery.conf.done
   ```

### pgBackRest의 추가 기능

- **암호화**: `pgBackRest`는 백업 데이터를 암호화하여 저장할 수 있는 기능을 제공합니다. `encryption-passphrase`를 설정하여 데이터를 암호화할 수 있습니다.
- **압축**: 백업 파일을 압축하여 저장할 수 있습니다. 이를 통해 디스크 공간을 절약할 수 있습니다.
- **자동화 및 스케줄링**: `pgBackRest`는 자동화된 백업 스케줄을 설정할 수 있어, 백업을 정기적으로 수행할 수 있습니다.

### pgBackRest의 장점

- **병렬 처리**: 대규모 데이터베이스에서도 고속으로 백업 및 복구를 수행할 수 있습니다.
- **백업 효율성**: 전체 백업과 증분 백업을 효율적으로 관리할 수 있습니다.
- **운영 중 복구 가능**: 복구 작업이 백그라운드에서 실행되며, 데이터베이스 운영에 영향을 미치지 않습니다.
- **Point-in-Time 복구**: 특정 시점으로 복구할 수 있는 기능을 제공하여, 데이터 복구가 필요한 정확한 시점으로 되돌릴 수 있습니다.
