---
layout: post
title:  "[Redis Documentation #2] 리플리케이션 관련 파라미터 "
date:   2019-05-19 21:05:00
author: garimoo
categories: Database
---

# 리플리케이션 관련 파라미터
### SLAVEOF
> 레디스 5부터 더이상 슬레이브라는 단어를 사용하지 않는다. 대신 REPLICAOF 명령어의 사용을 권장한다. SLAVEOF 명령어는 오직 역호환성을 위해 계속 작동할 것이다.

### REPLICAOF
- replica의 복제 설정을 즉시 변경
- `REPLICAOF hostname port`
    - 해당 서버를 지정된 서버의 복제본으로 만듬.
    - 해당 서버가 이미 다른 마스터의 복제본인 경우, 이전 서버에 대한 복제를 중지하고, 데이터셋을 삭제한 뒤 새 서버에 대한 동기화를 시작함.
- `REPLICAOF NO ONE`
    - 해당 서버가 이미 replica로 작동 중인 경우 복제가 해제되어 해당 서버가 마스터로 변환됨.
    - 복제된 내용을 삭제하지는 않음. 따라서 다시 복제본으로 작동할 수 있도록 재구성할 수 있음.

### REPLICA-SERVE-STALE-DATA
- 슬레이브가 마스터와의 연결이 끊겼거나, 복제가 진행중인 상황에서 해당 파라미터에 의해 두 가지 방법으로 작동됨.
- **YES** : 오래된 데이터거나, 데이터 셋이 비어있는 경우(첫번째 동기화 진행중) 에도 클라이언트 요청에 응답함.
- **NO**: 아래 명령어 이외에는 `SYNC with master in progress` 오류로 응답.
    - INFO, replicaOF, AUTH, PING, SHUTDOWN, REPLCONF, ROLE, CONFIG, SUBSCRIBE, UNSUBSCRIBE, PSUBSCRIBE, PUNSUBSCRIBE, PUBLISH, PUBSUB, COMMAND, POST, HOST, LATENCY

### REPLICA-READ-ONLY
- 기본 설정은 read-only **YES**
- 쓰기를 허용하도록 슬레이브를 구성할 수 있다. 복제본 인스턴스에 대해 쓰기를 허용하는 것은 사용 후에 삭제할 데이터를 저장하는 데 유용할 수 있지만 (마스터와 재동기화 후 삭제되기 때문에), 잘못된 구성의 적용으로 클라이언트가 write를 했을 때 문제를 발생할 가능성이 존재함.


```
NOTE: 읽기 전용 슬레이브는 인터넷에서 신뢰할 수 없는 클라이언트에 노출되도록 설계되지 않았다. 단지 인스턴스의 misuse에 대한 protection layer일 뿐이다. 여전히 읽기 전용 슬레이브는 기본적으로 CONFIG, DEBUG 등과 같은 모든 관리 명령을 내보낼 수 있다. 제한적으로 어드민 또는 위험한 명령어에 대해 `rename-command`를 사용해서 읽기 전용 슬레이브의 보안을 개선할 수 있다.
```

### REPL-DISKNESS_SYNC
***WARNING: 이 기능은 현재 EXPERIMENTAL한 기능임.***

- 부분 동기화를 진행할 수 없거나, 새로운 연결을 시도할 경우 full synchronization을 수행해야 함. RDB 파일은 마스터에서 복제본으로 전송됨.

- **DISK-Backed**: 마스터는 디스크에 RDB 파일을 쓰는 새로운 프로세스 생성. 나중에 파일은 부모 프로세스에 의해 슬레이브로 증분 전달됨.
    - RDB 파일이 생성되는 동안, 자식 프로세스에 의해 더 많은 복제본들이 RDB 파일에 추가되어 함께 전송될 수 있음.
- **Diskless**: 마스터는 디스크를 사용하지 않고 RDB 파일을 복제 소켓에 직접 쓰는 새로운 프로세스 생성
    - 전송이 시작되면 새로운 복제본이 대기열에 오르고, 해당 복제가 끝난 다음에 새로운 복제본이 전달됨.
    - 마스터는 여러 복제본이 도착하고 전송을 병렬화 하게 진행하기 희망하기 때문에 몇 초(configurable amount of time)을 대기함.
    - 느린 디스크와 빠른 네트워크인 경우, diskless 복제가 더 효과적

### REPL-DISKLESS-SYNC-DELAY
- 소켓을 통해 복제본으로 RDB를 전송하는 자식 프로세스의 대기 시간을 구성할 수 있음.
- 전송이 시작되면, 다음 RDB 전송을 위해 대기할 새로운 복제본을 제공할 수 없으므로, 서버는 한번에 더 많은 복제본이 도착하도록 지연시간을 갖고 기다림.
- 지연은 초 단위로 지정되며, 기본값은 5초.
- 0초로 설정하여 대기시간 없이 바로 전송하게 설정 가능.

### REPL-PING-REPLICA-PERIOD
- 슬레이브에서 마스터로 ping을 전송하는 간격
- 기본값은 10초

### REPL-TIMEOUT
- 마스터와 슬레이브간에 연결이 끊겼다고 인식하는 시간.

##### 마스터 관점
- 1초마다 replconf 명령을 슬레이브로 보내고, ack가 timeout 시간동안 오지 않으면 슬레이브에 대한 연결을 해제하고 정보를 지움
    - 평상시에는 마지막 항목 lag이 0이거나 1
    - 복제 서버로부터 ack가 오지 않으면 그 시간만큼 lag 숫자가 증가
    - lag이 timeout을 초과하면 마스터 서버는 복제서버 정보 삭제

##### 슬레이브 관점
- 슬레이브는 `repl-ping-replica-period` 간격으로 ping을 보내는데, timeout 시간동안 응답이 없으면 마스터와의 연결이 다운된 것으로 인식
-  info replication으로 `master_last_io_seconds_ago` 확인
    -  마스터로부터 응답을 받지 못한 시간
    -  연결이 다운되었을 때는 -1

### REPL-DISABLE-TCP-NODELAY
![Image](/assets/20190519/1.png)
- **YES : NODELAY FALSE**
    - 데이터를 모아서 큰 패킷으로 전송
    - 최대 40ms의 지연 발생 가능
    - 지연시간이 짧고 트래픽이 많을 경우에 사용 권장

- **NO**
    - 기본 설정
    - yes일 경우보다 대역폭을 더 많이 사용


### REPL-BACKLOG-SIZE
- 부분적 재동기화(partial resynchronization)에 사용되는 버퍼 사이즈 단위를 지정
- 기본값은 1mb
    - kb, mb, gb 단위 사용 가능
- 0으로 설정시 아래와 같은 에러 발생
    ```
    *** FATAL CONFIG FILE ERROR ***
    Reading the configuration file, at line 392
    >>> 'repl-backlog-size 0'
    repl-backlog-size must be 1 or greater.
    ```
- 해당 서버에 슬레이브가 연결되는 순간 해당 파라미터만큼 backlog-buffer 할당됨
- 백로그 크기가 클 수록 재동기화를 수행하는 시간이 길어짐
```
Resynchronization 방식

슬레이브와 연결이 끊어졌을 때 입력되는 데이터를 backlog-buffer에 저장했다가, 다시 연결되면 전체 데이터를 다시 보낼 필요 없이 해당 버퍼의 데이터를 슬레이브에 보내서 동기화한다. 이렇게 동작하는 것을 부분 동기화 (partial resynchronization)이라 한다. 입력되는 데이터가 이 버퍼 사이즈를 초과하면 전체 동기화(full synchronization)을 시도한다.
```

### REPL-BACKLOG-TTL
- 마스터에서 한동안 슬레이브가 연결되지 않을 때 백로그 버퍼를 해제하기 위해 경과해야 하는 시간
- 연결이 해제된 슬레이브가 부분적으로 재동기화 할 가능성이 있기 때문에 항상 백로그를 누적해야 하기 때문에 해당 시간이 초과하기 전까지는 백로그를 항상 유지함.
- 값이 0이면 백로그를 절대 해제하지 않음을 의미
- 기본 설정은 3600초


### REPLICA-PRIORITY
- 레디스 센티넬에서 마스터가 올바르게 작동하지 않을 때 마스터로 승격할 슬레이브을 선택하기 위해 사용.
- 기본값은 100, 숫자가 작을수록 우선순위가 높음
- 0은 마스터가 되지 않도록 하는 값
    - 다른 슬레이브가 있을 경우에만 해당됨
    - 다운된 마스터에 오직 하나의 슬레이브만 있다면 이 값이 0이어도 마스터로 승격됨

### MIN-REPLICAS-TO-WRITE
- 복제가 성공적으로 수행되어야 하는 최소한의 슬레이브 수
- 슬레이브가 설정한 수 보다 적으면 마스터는 쓰기 명령을 수행하지 못하고 에러 리턴
- 기본 값은 0 (기능 비활성화)

### MIN-REPLICAS-MAX-LAG
- 복제가 성공적으로 수행되어야 하는 시간을 설정
- 마스터가 슬레이브에에 1초마다 보내는 replconf의 ack로 확인.
- 설정한 시간동안 슬레이브로부터 ack가 없으면 해당 값이 줄어듬.
- 기본 값은 10

### REPLICA-ANNOUNCE-IP / REPLICA-ANNOUNCE-PORT
- 해당 옵션을 사용해서 마스터는 연결된 슬레이브의 주소와 포트를 다른 방식으로 나열할 수 있음.
- 포트 포워딩 또는 NAT을 사용할 경우 슬레이브는 실제로 서로 다른 IP나 port를 통해 연결되는 경우 필요.
- 예를 들어 `info replication` 명령어 또는 `role` 명령어로 해당 마스터에 연결된 슬레이브의 정보를 지정한 값으로 노출시킬 수 있음.

# 참고
- https://redis.io/commands
- redis.conf 파일 (5.0.3 버전)
