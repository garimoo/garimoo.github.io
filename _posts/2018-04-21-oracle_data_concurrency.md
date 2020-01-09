---
layout: post
title:  "[ORACLE] 데이터 동시성 관리"
date:   2018-04-19 20:31:00
author: garimoo
categories: Database
---
<br/>
## LOCK

* 다중 세션에서 동일한 데이터를 동시에 변경하는 것 방지
* 주어진 명령문에 대해 가장 낮은 레벨에서 자동으로 획득

### Lock 매커니즘

* 목적: 최대한의 데이터 동시성 제공
* 데이터 수정 트랜잭션은 `행 레벨 lock` 획득.
* 객체 수정 트랜잭션은 `객체 레벨 lock` 획득.
* Lock된 데이터를 읽을 때에는 Lock 이전 값이 표시.
* 다중 트랜잭션에서 같은 리소스를 Lock 하는 경우 첫번째 요청 트랜잭션이 획득
* 트랜잭션 완료(Commit, Rollback)이 적용 시에 모든 Lock이 해제.
* 실패시 롤백하는 프로세스에서 lock을 모두 해제.

### 데이터 동시성

* 하위 단계(fine-grain)의 행 레벨 Lock모드가 기본.
* 필요한 경우 더 높은 레벨에서 수동 Lock 가능.
* Lock 모드
    * **ROW SHARE**: lock된 테이블에 동시 액세스 가능하지만, 전체 테이블을 lock하는 것은 금지
    * **ROW EXCLUSIVE(RX)**: **ROW SHARE**와 동일하지만, share모드에서 lock하는 것 금지. 여러번 읽고 한번 쓸 수 있다.
    * **SHARE**: 동시 query 허용하지만 Lock된 테이블에 갱신 금지. 여러번 읽지만, 쓸 수 없다.
    * **SHARE ROW EXCLUSIVE**: 전체 테이블을 query하는 데에 사용. 다른 유저가 테이블의 query는 허용하지만, 해당 테이블을 lock하거나 행 갱신하는 것은 금지.
    * **EXCLUSIVE**: lock된 테이블에서의 query는 허용, 해당 테이블에서 다른 작업은 금지.

### DML LOCK

* 각 DML 트랜잭션은 다음의 Lock 획득
    * 갱신 중인 행에 대한 EXCLUSIVE 행 Lock
    * ROW EXCLUSIVE 모드에서 갱신 중인 테이블에 대한 테이블 lock(TM)
    변경 작업이 수행될 때 다른 세션에서 전체 테이블을 삭제하거나 truncate(전체 행 삭제)하기 위해 Lock 하는 것 방지.

### Enqueue 매커니즘

* Lock은 자동으로 큐에 저장. Lock을 보유하는 트랜잭션이 완료되면 다음 트랜잭션이 Lock 보유.
* Lock을 이미 보유한 세션은 해당 Lock의 변환 요청 가능.

## Lock 충돌

* 자주 발생하는 경우지만, 대개 Enqueue 매커니즘으로 해결.

### 충돌 원인

* commit되지 않은 변경 사항
* 장기 실행 트랜잭션
* 필요 이상으로 높은 Lock 레벨

### 충돌 해결

* lock을 보유하는 세션에서 lock 해제
* **kill session**: 비상시 사용. 현재 트랜잭션 내의 작업이 롤백됨.

#### SQL을 사용하여 충돌 해결

```
SQL> select SID, SERIAL#, USERNAME
from V$SESSION where SID in
(select BLOCKING_SESSION from V$SESSION)
```

에서 나온 결과값이 (SID: 144, SERIAL#: 8982) 라면

```
SQL> alter system kill session '144, 8982' immediate;
```

### Deadlock

* 둘 이상의 세션이 각각 Lock 된 데이터를 서로 대기하고 있을 때를 의미.
* 오라클 데이터베이스는 Deadlock을 자동으로 감지, 오류가 발생한 명령문 종료. 해당 오류에 대한 응답은 커밋 또는 롤백, lock 해제
