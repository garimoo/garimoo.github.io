---
layout: post
title:  "[Redis] 레디스 파이프라인을 이용해 더미데이터 생성"
date:   2019-05-29 11:57:00
author: garimoo
categories: Database
---

# 레디스에 파이프라인을 이용해 더미데이터 생성하기
## 레디스 연결 제대로 되는지 확인
```
$ (printf "PING\r\nPING\r\nPING\r\n"; sleep 1)|nc <server IP> <port>
+PONG
+PONG
+PONG
```
## 더미데이터 생성
- 다음 파일 [다운로드](/assets/20190529/redis_dummy.txt)
```
$  echo -e "$(cat redis_dummy.txt)" | /home1/cubrid1/redis-5.0.3/src/redis-cli -h <server IP> -p <port> --pipe
```
- 파일은 레디스 프로토콜을 통해 작성되어있음.
```
127.0.0.1:6379> DBSIZE
(integer) 364
127.0.0.1:6379> DBSIZE
(integer) 364
```
- DBSIZE 명령어를 통해 데이터가 잘 들어갔는지 확인할 수 있음.
## 출처
- http://intro2libsys.info/introduction-to-redis/pipelining-and-mass-insertions
