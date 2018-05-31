---
layout: post
title:  "[ORACLE] sqlplus 에서 방향키 사용"
date:   2018-05-31 10:50:00
author: garimoo
categories: Database
---

oracle 설치 후 sqlplus를 그냥 실행하면 방향키가 안먹어서, ↑ 버튼이 안눌리는 문제가 발생한다. 했던 명령어 치려면 계속 다시 쳐야 함..
그럴 때
```
$ yum install rlwrap
$ alias sqlplus = 'rlwrap sqlplus'
```
해서 들어가면 해결된다. 하지만 rlwrap 다운로드가 안될 수 도 있는데..! `no package rlwrap available` 이란 에러가 발생하면

```
$ yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm
$ yum install rlwrap
```
이렇게 하면 해결됨.
