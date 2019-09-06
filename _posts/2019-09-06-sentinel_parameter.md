---
layout: post
title:  "[Redis Documentation #4] 센티널 관련 파라미터 "
date:   2019-09-06 11:03:00
author: garimoo
categories: Database
---
# 센티널  관련 파라미터
```
센티널은 기본 설정으로 bind와 protected-mode 파라미터가 비활성화되어 있음.
이렇게 하면 local 이외에 외부에서 센티널에 접속할 수 없음.
그러므로 bind에 ip를 지정하고 protected-mode를 yes로 설정하고 사용해야 함.
```
### DAEMONIZE
- 센티널은 기본적으로 데몬으로 실행되지 않음.
- 데몬으로 실행하려면 YES(Recommend)

### PIDFILE
- 데몬으로 실행될 때 process id를 pid 파일에 기록함.
- pid 파일의 경로를 지정
- `var/run/redis-sentinel.pid` 가 기본 경로

### LOGFILE
- 로그파일을 지정
- 데몬으로 실행할 경우 로그파일을 지정하지 않으면 `/dev/null`로 날아감.

### ANNOUNCE-IP / ANNOUNCE-PORT
- NAT으로 인해 센티널은 로컬이 아닌 주소를 통해 외부에서 접근할 수 있음.
- 해당 IP가 제공되면 일반적으로 로컬 주소를 감지하는 방법이 아니라, HELLO 메시지를 통해 지정된 IP 주소의 인스턴스를 감지함.
- 마찬가지로 announce 가 제공된 경우, 이 정보가 유효하고 0이 아니라면, 센티널은 지정된 TCP 포트를 이용함.
- ANNOUNCE-IP만 지정된 경우, 센티널은 port 옵션으로 지정된 대로 지정된 IP및 서버 포트를 사용하고, ANNOUNCE-PORT만 지정된 경우, 자동으로 감지된 로컬 IP와 지정된 포트를 사용함.

### DIR
- 작업 디렉토리 지정
- conf, log 파일이 위치

### SENTINEL MONITOR
- `sentinel monitor <master-name> <ip> <redis-port> <quorum>`
- 최소 \<quorum> 개의 센티널이 동의하는 경우에만 센티널이 이 마스터를 모니터링하고, ODOWN (Objectively Down) 상태로 간주
- ODOWN의 쿼럼이 무엇이든 센티널은 페일오버를 시작하기 위해 과반수 이상의 센티널에 의해 선출되어야 하므로, 소수의 페일오버는 수행되지 않음.
- 슬레이브는 자동으로 검색되므로, 어떠한 방식으로도 슬레이브를 지정할 필요가 없음.
- 복제본이 추가되면 센티널은 구성 파일을 다시 작성함. 복제본이 마스터로 승격되는 경우에도 구성파일은 다시 작성됨.
```
NOTE: 마스터 이름에는 특수 문자나 공백이 없어야 함.
유효한 셋은 `A-z 0-9` 와 문자 `.-_`
```


### SENTINEL AUTH-PATH
- `SENTINEL auth-path <master-name> <password>`
- 마스터와 슬레이브를 인증하는 데에 사용할 비밀번호를 설정
- 모니터링할 레디스 인스턴스에 비밀번호가 설정된 경우 유용함.
- 마스터 비밀번호는 슬레이브에도 사용되므로 슬레이브를 사용하여 인스턴스를 모니터링 하려면 마스터 및 슬레이브 인스턴스에서 다른 비밀번호를 설정할 수 없음.
- 인증이 필요한 레디스 인스턴스와, 필요하지 않은 인스턴스를 혼합하여 사용하는 경우 AUTH 명령이 꺼진 레디스 인스턴스에 영향을 미치지는 않음.

### SENTINEL DOWN-AFTER-MILLISECONDS
- `SENTINEL down-after-milliseconds <master-name> <milliseconds>`
- SDOWN 상태로 인식하기 위해 마스터(또는 연결된 슬레이브나 센티널)에 도달할 수 없는 시간
    - unreachable: 지정된 기간동안 PING에 적절한 응답을 하지 않음

- default는 30초

### SENTINEL DENY-SCRIPTS_RECONFIG

### SENTINEL MONITOR MASTER



### SENTINEL PARALLEL-SYNCS

- 페일오버 도중에 새 마스터를 바라보도록 재구성 할 수 있는 슬레이브의 수.
- 동기화를 수행하는 동안 쿼리가 다른 슬레이브에 도달할 수 없도록 피하기 위해서는 작은 수로 설정해야 함.
- 1일 경우 한번에 한 슬레이브씩 처리함.
-  만약 이 기능이 없다면 여러 슬레이브가 동시에 새 마스터에 데이터 전체 동기(Full resync)를 요청함.
### SENTINEL FAILOVER-TIMEOUT
- 페일오버가 설정한 시간이 지나도 완료되지 않으면 취소하는 시간
- 기본은 3분
- 지정된 센티널이 동일한 마스터에 대해 이전의 페일오버를 이미 시도한 뒤, 페일오버를 다시 시작하는 데에 필요한 시간은 페일오버 타임아웃 시간의 **두 배** 가 필요함.
- 센티널의 현재 구성에 따라 잘못된 마스터에 대해 복제본을 연결하여 복제하는데 필요한 시간은, 그래서 올바른 마스터에 연결되어 복제를 진행해야 하는 시간은 페일오버 타임아웃 시간과 같다. (센티널에서 잘못된 구성임을 감지한 시간부터)

- 페일오버가 이미 진행중이지만, 어떠한 구성적으로 변화가 없었을 때 페일오버를 취소하는 데에 필요한 시간 (SLAVE OF NO ONE을 승격된 마스터에서 승인하지 않았을 경우)

- 진행중인 페일오버가 모든 슬레이브가 새로운 마스터의 복제본으로 재구성될 때까지 대기하는 최대 시간을 의미. 그러나 이 시간이 지나도 복제본은 센티널에 의해 재구성되지만, 지정된 병렬 동기화 진행으로는 재구성되지 않음.



### SENTINEL CONFIG-EPOCH



### SENTINEL NOTIFICATION-SCRIPT
- `SENTINEL notification script <master-name> <script-path>`

- 센티널 notification-script나 sentinel-reconfig-script는 관리자에게 알리거나, 페일오버 이후 클라이언트를 다시 구성하기 위해 호출되는 스크립트를 구성하는데에 사용됨. 스크립트는 오류 처리를 위해 다음 규칙에 의해 실행됨.
- 스크립트가 1로 종료되면 나중에 재시도됨. (최대 10회까지 재시도)
- 스크립트가 2 또는 더 높은 값으로 종료되면 스크립트는 재시도되지 않음.
- 스크립트의 최대 실행 시간은 60초, 이 타임아웃에 도달하면 스크립트는 SIGKILL로 종료되고, 실행이 다시 시도됨.
- 예를 들어 WARNING 레벨에서 생성된 센티널 이벤트 (예: -sdonw, odown 등) 에 대해 지정된 알림 스크립트를 호출할 수 있다.
- 이 스크립트는 이메일, SMS 또는 기타 메시징 시스템을 통해 시스템 관리자에게 모니터링되는 레디스 시스템에 문제가 있음을 알릴 수 있음.
- 스크립트는 단지 두개의 인수만으로 호출됨. 첫번째는 이벤트 유형, 두번째는 이벤트 설명
- 이 옵션이 제공되면 센티널을 시작하려면 스크립트가 존재해야 하며, 실행 가능해야 한다.





### SENTINEL CLIENT-RECONFIG-SCRIPT
- `sentinel client-reconfig-script <master-name> <script-path>`
- 페일오버로 인해 마스터가 변경된 경우 어플리케이션별 작업을 수행하여 구성이 변경되었으며, 마스터가 다른 주소에 있음을 클라이언트에 알리기 위해 스크립트를 호출할 수 있음.

다음 인수가 스크립트에 전달됨
```
<master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
```
- \<state> 는 항상 failover
- \<role>은 leader 혹은 observer 임.
- from-ip, from-port, to-ip, to-port 인수는 마스터의 이전 주소와 선택한 복제본 (현재 마스터)의 새 주소를 전달하는 데 사용됨.

- 기본적으로 SENTINEL SET은 런타임중에 알림 스크립트나 클라이언트 재구성 스크립트를 변경할 수 없음.
- 클라이언트가 스크립트를 설정하고 장애조치를 트리거하여 프로그램을 실행시킬 수 있는 간단한 보안 문제를 피하기 위함.


### SENTINEL LEADER-EPOCH



### SENTINEL RENAME-COMMAND
- 센티널이 모니터하는 레디스 서버가 명령을 rename했을 경우 여기에 바뀐 명령을 설정함.


### SENTINEL KNOWN-REPLICA

### SENTINEL KNOWN-SENTINEL

# 참고
- https://redis.io/commands
- sentinel.conf 파일 (5.0.3 버전)
