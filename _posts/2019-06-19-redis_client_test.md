---
layout: post
title:  "[Redis] 센티널 구성과 파이썬 패키지를 이용한 클라이언트 테스트"
date:   2019-05-19 18:57:00
author: garimoo
categories: Database
---
## Question
### 센티널을 이용한 레디스 환경에서 Application은 어떻게 MasterDB를 접속 하는가? ex) AP - 센티널 - 레디스 형태로의 통신?

### 이 상태에서 Master 서버의 NW문제로 통신이 불가할 경우, AP는 어떻게 Master를 찾아 연결 하는가?

# 레디스 구성
![Image](/assets/20190619/1.png)

### Sentinel info
```
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=10.162.29.33:6371,slaves=2,sentinels=3
```

# 테스트1: 마스터 프로세스 킬
## 마스터 킬 후 슬레이브 상황
- 슬레이브1: 마스터로 승격
```
10.162.29.36:6371> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=10.162.29.37,port=6371,state=online,offset=89041,lag=0
master_replid:fe476f74f3f980067686f0e53a1431b2259fb8bd
master_replid2:87869ad90965d8d2c319e96d3b6885705fe47038
master_repl_offset:89041
second_repl_offset:87337
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1821
repl_backlog_histlen:87221
```
- 슬레이브 2
```
10.162.29.37:6371> info replication
# Replication
role:slave
master_host:10.162.29.33
master_port:6371
master_link_status:down
master_last_io_seconds_ago:-1
master_sync_in_progress:0
.
.
.
10.162.29.37:6371> info replication
# Replication
role:slave
master_host:10.162.29.36
master_port:6371
master_link_status:up
```

## 페일오버 된 상태에서 새로운 데이터 입력
```
10.162.29.36:6371> set newkey newdata
OK
10.162.29.36:6371> set newkey2 newdata2
OK
```

## 죽은 마스터가 다시 올라오면?
- 기존 마스터가 새로운 마스터의 슬레이브로 연결
```
10.162.29.33:6371> info replication
# Replication
role:slave
master_host:10.162.29.36
master_port:6371
```
## 새로 입력했던 데이터 조회
```
10.162.29.33:6371> get newkey
"newdata"
10.162.29.33:6371> get newkey2
"newdata2"
```


# 테스트 2: 클라이언트 테스트
![Image](/assets/20190619/2.png)

## 테스트 시나리오
1. 마스터에 붙어있는 클라이언트
2. 마스터 Down(redis.io 문서에서는 partition 이라고 표현, 즉 shutdown, power down, network 단절 등)
3. 연결이 끊긴 상태에서 클라이언트가 인식하는 마스터 확인
4. 연결이 끊긴 상태에서 (구) 마스터에게 클라이언트가 데이터 입력
5. 재연결된 상태에서 클라이언트가 입력한 데이터 확인

## 클라이언트 연결
``` python
from redis import Redis, exceptions, RedisError
from redis.sentinel import (Sentinel, SentinelConnectionPool,ConnectionError, MasterNotFoundError, SlaveNotFoundError)


listSentinel = [('10.162.29.33', 27001), ('10.162.29.36', 27002), ('10.162.29.37', 27003)]
strServiceName = 'mymaster'
nMaxUser = 1000
sentinel = Sentinel(listSentinel, socket_timeout=0.1)

try:
    # No need for this
    #sentinel.discover_master(strServiceName)
    #sentinel.discover_slaves(strServiceName)
    master = sentinel.master_for(strServiceName, socket_timeout=0.1)
    slave = sentinel.slave_for(strServiceName, socket_timeout=0.1)
```
- 센티널들의 정보를 리스트로 입력
- `sentinel.discover_master(strServiceName)` 으로 마스터 정보 확인 가능


## 마스터에서 슬레이브 아이피 차단
- 네트워크 단절을 테스트하기 위해 iptables rule을 추가해 슬레이브 ip 차단
```
[irteamsu@data-redis-wa903 ~]$ sudo iptables -A INPUT -s 10.162.29.33 -j DROP
[irteamsu@data-redis-wa903 ~]$ sudo iptables -A INPUT -s 10.162.29.36 -j DROP
```

## 슬레이브 상태
- master는 변하지 않고, master_link_status는 down 상태 유지.
```
master_host:10.162.29.37
master_port:6371
master_link_status:down
```

## 센티널 상태
- 마스터에 연결된 slave를 0으로 인식
```
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=odown,address=10.162.29.37:6371,slaves=0,sentinels=2
```
**슬레이브에서 오는 연결은 차단됐지만, 센티널에서는 계속 정상으로 모니터링 되어서 `계속해서 마스터로 살아있음`**

## 센티널 모니터링 연결 차단
- 센티널이 떠있는 포트 차단
``` sh
[irteamsu@data-redis-wa903 ~]$ sudo iptables -A INPUT -p tcp --sport 27001 -j DROP
[irteamsu@data-redis-wa903 ~]$ sudo iptables -A INPUT -p tcp --sport 27002 -j DROP
[irteamsu@data-redis-wa903 ~]$ sudo iptables -A INPUT -p tcp --sport 27003 -j DROP
```
## 연결 상태
![Image](/assets/20190619/3.png)
## 센티널 상태
- 마스터 모니터링이 안되기 때문에 슬레이브를 마스터로 승격시킴
```
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=10.162.29.36:6371,slaves=2,sentinels=2
```

## 레디스 상태
- 네트워크 단절된 마스터
```
# Replication
role:master
connected_slaves:0
```
- (신) 마스터(36)
```
# Replication
role:master
connected_slaves:1
slave0:ip=10.162.29.33,port=6371,state=online,offset=9554376,lag=1
```

## 각 마스터에 데이터 입력
- 신 마스터에 iam master 입력
```
10.162.29.36:6371> set iam master
OK
```
- 구 마스터에 iwas master 입력

```
10.162.29.37:6371> set iwas master
OK
```

## 차단 복구

```
[irteamsu@data-redis-wa903 ~]$ sudo iptables -F
```
## 연결 상태
![Image](/assets/20190619/4.png)
## 마스터 상태
- 단절이 해결된 마스터가 신 마스터의 슬레이브로 연결됨.
```
10.162.29.36:6371> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=10.162.29.33,port=6371,state=online,offset=10210046,lag=1
slave1:ip=10.162.29.37,port=6371,state=online,offset=10210046,lag=1
```

## 입력된 데이터 확인
- 연결 단절된 마스터에 입력했던 데이터는 사라짐.
```
10.162.29.37:6371> get iam
"master"
10.162.29.37:6371> get iwas
(nil)
```
