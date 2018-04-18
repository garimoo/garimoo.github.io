---
layout: post
title:  "[ORACLE] 데이터베이스 Instance"
date:   2018-04-17 17:40:00
author: garimoo
categories: Database
---
<br/>
## 관리 프레임워크

* 데이터베이스 Instance
* 리스너
* 관리 인터페이스
    * Database Control
    * 관리 에이전트 (grid control 사용시)

## 초기화 파라미터 파일

instance가 시작되면 초기화 파라미터 파일 읽음.

* **서버 파라미터 파일(SPFILE)**: 일반적으로 사용되는 초기화 파라미터 파일. 데이터베이스 서버가 읽고 쓸 수 있는 이진 파일. _수동으로 편집하면 안된다._ 종료 및 시작과 상관없이 계속 유지.
* **텍스트 초기화 파라미터 파일**: 서버는 읽을 수 있지만, 쓸 수 없다. 종료 및 시작과 관계없이 초기화 파라미터 설정을 유지하려면 해당 설정을 수동으로 설정, 변경. SPFILE을 찾을 수 없을 때 자동으로 검색.

다음과 같은 유형의 초기화 파라미터 값이 있다.

* Boolean
* String
* Integer
* 파라미터 파일
* Reserved
* Big Inteager

다른 파라미터로부터 계산되는 **파생 파라미터**의 값은 변경해서는 안된다.
일부 파라미터는 운영체제에 종속적이다. _ex\) DB\_BLOCK\_SIZE_

### 기본 파라미터

* **DB\_NAME\, DB\_DOMAIN** : Global Database Name
* **DB\_RECOVERY\_FILE\_DEST\, DB\_RECOVERY\_FILE\_DEST\_SIZE** : Fast Recovery Area
* **SGA_TARGET** : 모든 SGA 구성 요소의 전체 크기 지정
* **UNDO_TABLESPACE** : undo space 관리 테이블스페이스 방법 지정
* COMPATIBLE 초기화 파라미터 및 취소 불가능한 호환성

### 초기화 파라미터

* **CONTROL_FILES**: 하나 이상의 control file 이름을 지정.
* **DB_FILES**: 해당 데이터베이스에 대해 열 수 있는 최대 데이터베이스 파일 수 지정
* **PROCESSES**: Oracle 서버에 동시에 연결할 수 있는 OS User Process의 최대 ㅜ 지정
* **DB\_BLOCK\_SIZE**: 오라클 데이터베이스 블록 크기를 바이트 단위로 지정. _생성시 설정, 차후 변경 불가_
* **DB\_CACHE\_SIZE**: 표준 블록 버퍼 캐시 크기 지정. 16MB 이상, SGA_TARGET이 설정된 경우 0, 그렇지 않으면 48MB
* **SGA_TARGET**: 모든 SGA 구성 요소의 전체 크기 지정. 설정할 경우 다음 메모리 풀의 크기가 자동 지정
    * DB\_CACHE\_SIZE \(버퍼 캐시\)
    * SHARED\_POOL\_SIZE
    * LARGE\_POOL\_SIZE
    * JAVA\_POOL\_SIZE
    * STREAMS\_POOL\_SIZE
* **MEMORY_TARGET**: Oracle 시스템 전체에서 사용 가능한 메모리 지정. SGA 및 PGA를 늘리거나 줄이면서 MEMORY_TARGET 값에 맞게 메모리 튜닝
* **PGA\_AGGREGATE\_TARGET**: PGA 메모리 양 지정. SGA에 포함되지 않음.
* **SHARED\_POOL\_SIZE**: Shared Pool의 크기를 바이트 단위로 지정.
* **UNDO_MANAGEMENT**: 시스템에서 사용할 undo space 관리 모드를 지정.

### 파라미터 확인

![Image](/assets/20180417/1.png)

### 초기화 파라미터 유형

* **Static parameter**: Instance 또는 전체 DB에 영향을 주며, init.ora 또는 SPFILE의 내용을 변경해서 수정. 현재 Instance에 대해서는 수정할 수 없다.
* **Dynamic Parameter**: 데이터베이스가 온라인 상태인 동안 변경 가능.
    * session level parameter: 유저 세션에만 영향. 국가별 설정 등. 세션이 끝나면 파라미터도 만료.
    * system level parameter: 전체 데이터베이스 및 모든 세션에 영향.

## 데이터베이스 시작

![Image](/assets/20180417/2.png)
데이터베이스를 열 때에 데이터 파일이나 온라인 리두 로그 파일이 없다면 Oracle 서버는 오류 반환.

```
SQL> startup
SQL> startup nomount
SQL> alter database mount;
SQL> alter databse open;
```

## 데이터베이스 종료

### 종료 모드

| **종료 모드** | **A** | **I** | **T** | **N** |
| ----- | --- | --- | --- | --- |
| 새로운 연결 허용 | N | N | N | N |
| 현재 세션 종료 시까지 대기 | N | N | N | Y |
| 현재 트랜잭션 종료 시까지 대기 | N | N | Y | Y |
| 체크포인트 적용 및 파일 닫기 | N | Y | Y | Y |

* ABORT: 종료 전 최소한의 작업 수행. 다시 시작하기 전에 recovery가 수행되어야 함.
* IMMEDIATE: 가장 일반적으로 사용. 커밋되지 않은 트랜잭션은 롤백.
* TRANSACTIONAL: 기존 트랜잭션 종료 가능, 새 트랜잭션을 시작하지는 않음.
* NORMAL: 세션의 연결이 끊길 때까지 대기

### 종료 옵션

* **SHUTDOWN NORMAL**: 모든 유저가 연결을 끊을 때까지 종료하지 않고 대기.
* **SHUTDOWN TRANSACTIONAL**: 진행 중인 트랜잭션이 종료되면 클라이언트의 연결이 해제.
* **SHUTDOWN IMMEDIATE**: 현재 처리하고 있는 SQL문이 완료되지 않음. 활성 트랜잭션을 롤백, 모든 유저를 해제. 이후 시작시 recovery는 필요 없다. _IMMEDIATE는 Enterprise Manager를 사용할 때의 기본 종료 모드._
* **SHUTDOWN ABORT**: 현재 처리중인 SQL문이 즉시 종료. 롤백되지 않고, 다음 번 시작 시 Instance Recovery가 필요하며, _자동으로 수행_

```
SQL> shutdown
SQL> shutdown transactional
SQL> shutdown immediate
SQL> shutdown abort
```

## Alert Log

`$ORACLE_BASE/diag/rdbms/<db_name>/<SID>/trace`

* 시작 시 사용된 기본값이 아닌 초기화 파라미터
* 발생한 모든 내부 오류(ORA-600), 블록 손상 오류(ORA-1578), deadlock 오류(ORA-60)
* CREATE, ALERT, DROP DATABSE 및 TABLESPACE SQL 문 및 STARTUP, SHUTDOWN, ARCHIVE LOG 및 RECOVER와 같은 **Enterprise Manager** 또는 **SQL * Plus**문과 같은 관리작업
* shared server 및 디스패처 프로세스의 기능과 관련된 일부 메시지 및 오류
* Materialized View의 자동 Refresh 중에 발생한 오류

### Trace file 사용

* 각 서버 프로세스와 백그라운드 프로세스는 연관된 Trace file에 정보 기록.
* Trace file의 파일 이름에는 Trace file을 생성한 프로세스의 이름이 포함되어 있지만, 작업 큐 프로세스(Jnnn)에 의해 생성되는 파일 제외.
* **ADR(Automatic Diagnostic Repository)**
    * Trace, Alert Log, 상태 모니터 보고서 등 데이터베이스 진단 데이터를 저장하는데 사용하는 **시스템 중앙 추적 및 로깅 Repository**

## Dynamic Performance 뷰

* 데이터베이스 instance의 작업 및 성능에 대한 동적 데이터 집합.
* 데이터베이스에 있는 일반 테이블이 아니므로 마운트되거나 열리기 전에 일부 뷰를 사용할 수 있다.
* 포함하는 정보
    * 세션
    * 파일 상태
    * 작업 진행 상황
    * Lock
    * 백업 상태
    * 메모리 사용 및 할당
    * 시스템 및 세션 파라미터
    * SQL 실행
    * 통계 및 Metric

```
SQL> SELECT sql_text, executions FROM v$sql WHERE cpu_time > 200000;
SQL> SELECT * FROM v$session WHERE machine = 'EDRSR9P1' and logon_time > SYSDATE -1;
SQL> SELECT sid, ctime FROM v$lock WHERE block > 0;
```

### 고려 사항

* 뷰는 SYS 유저가 소유.
* 사용 시점에 따라 서로 다른 뷰 사용 가능.
    * instance가 시작된 경우
    * 데이터베이스가 마운트 된 경우
    * 데이터베이스가 열린 경우
* V$FIZED_TABLE을 query하면 모든 뷰 이름을 볼 수 있다.
    * `v-dollar 뷰`라고 부른다.
* 데이터가 동적이기 떄문에 읽기 일관성이 보장되지 않음,

## 데이터 딕셔너리

* 데이터베이스에 대한 메타 데이터
* 데이터베이스에 있는 모든 객체의 이름과 속성 포함.
* 기본 테이블에 저장되지만, 테이블을 직접 읽는 대신 미리 정의된 뷰를 사용하여 테이블에 액세스 가능.
* 특성
    * 오라클 데이터베이스 서버가 유저, 객체, 제약 조건 및 저장 영역에 대한 정보를 찾는데 사용
    * 객체 주조나 정의가 수정될 때 데이터베이스에 의해 유지 관리
    * 유저가 데이터베이스 정보를 query하는 데 사용.
    * SYS 유저가 소유
    * SQL을 사용해서 직접 수정 불가.

### 데이터 딕셔너리 뷰

![Image](/assets/20180417/3.png)

```
SQL> SELECT table_name, tablespace_name FROM user_tables;
SQL> SELECT sequence_name, min_value, max_value, increment by FROM all_sequences WHERE sequence_owner IN ('MDSYS', 'XDB');
SQL> SELECT USERNAME, ACCOUNT_STATUS FROM dba_users WHERE ACCOUNT_STATUS = 'OPEN';
SQL> DESCRIBE dba_indexes;
```
