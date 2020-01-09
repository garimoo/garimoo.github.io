---
layout: post
title:  "[ORACLE] Vagrant 환경에서 Oracle 설치하기"
date:   2018-05-11 11:26:00
author: garimoo
categories: Database
---

vagrant 환경에서 오라클을 설치하면서 겪었던 문제점 정리.
나와 같은 뻘짓을 하는 사람이 없길 바라며..<br/>
+) 내가 같은 실수를 반복하지 않길 바라며..

## virtualbox에 oracle 설치 과정 (osX 환경)
1. vagrant를 이용하여 virtualbox에 linuxoracle7을 다운로드 (https://yum.oracle.com/boxes/)
그 뒤 세팅도 해준다(add box, init, up 등)
2. https://docs.oracle.com/cd/E11882_01/install.112/e24326/toc.htm#CEGEGDBA 를 따라서 oracle 설치
3. oracle 11g 이미지 다운로드(http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html)받아서, virtual box 공유 폴더에 저장

## 문제점 1) virtualbox mounting 문제
인터넷에서 다운받은 oracle 11g 압축파일을 풀어서 database 폴더를 virtualbox로 옮기고 싶은데, 어디에 마운트되는지 몰라 한참을 해멨다.
- vagrant ssh를 하는 그 폴더에 압축을 푼 database 파일 저장
- virtualbox 환경설정에서 해당 vm 자동 마운트 설정하기
- linux에서 reboot 후 다시 접속
- `cd /media` 에서 `ll`로 확인해보면 마운트 된 폴더 명이 `sf_database` 라고 나온다.
- 저 설치 가이드를 따라가다 보면 `/u01/app/`밑에다가 db를 설치하라고 나오니까 app 폴더 밑에 sf를 떼고 database 파일을 옮겨준다.
- `/media` 에 접근 가능한 계정은 root 계정, 실제 runInstaller를 실행해야 하는 계정은 oracle 계정이므로 권한변경이 필요하다.
```
chown -R oracle:dba /database
chmod -R 775 /database
```
위의 명령어를 통해 계정 변경을 하면 끝!

## 문제점 2) vagrant 에서의 x forwarding 문제
설치를 따라서 진행하다 보면 linux에서 xforwarding을 이용하여 GUI 환경에서 runInstaller를 실행해야 하는데, 창이 뜨지 않는 에러가 발생한다.
runInstaller를 실행하기 전에 xforwarding이 잘 되는지 `xclock`이나 `xterm`을 이용하여 환경을 확인해 보라고 가이드가 나온다. 환경설정이 되어있지 않으면 다음과 같은 에러가 나오고 xclock이 실행되지 않음.
```
Error: Can't open display: localhost:10.0
```

- 맥에서는 **XQuartz** 를 우선 다운받아야 한다.
- vagrant를 띄우고 ssh를 통해 접속할 때 `vagrant ssh -- -Y`를 통해 접속하면 xforwarding이 가능한 환경이 만들어진다... 인터넷을 아무리 찾아도 나오지 않았다.. vagrant ssh -- -X를 하라는 답변은 많이 봤는데, 이제는 X가 아니라 Y를 써야 하나봄.
- EXPORT DISPLAY=localhost:0.0 을 통해 디스플레이 변수 설정!

이 과정을 지나고 xclock을 실행하면 잘 뜰것이다! runInstaller도 잘 돌아감.

## 문제점 3) runInstaller 실행중 에러
```
INFO: Exception thrown from action: make
Exception Name: MakefileException
Exception String: Error in invoking target 'install' of
makefile '/u01/app/product/11.2.0/db_1/ctx/lib/ins_ctx.mk'.
See '/u01/oraInventory/logs/installActions2016-11-14_01-52-13PM.log'
for details.
```
설치중 위와 같은 에러가 나오고 진행이 안되는 상황이 발생.
(http://silentcargo.tistory.com/41)를 보고 해결했다!

## 문제점 4) netca : command not found
오라클 엔진 설치 후 netca를 실행해줘야 하는데, 쉘에서 인식을 못할 때

```
$ export ORACLE_HOME=/u01/app/oracle/product/11.2.0/dbhome_1
$ export PATH=$ORACLE_HOME/bin:$PATH
$ netca
```

이렇게 변수 설정해주면 됨!
