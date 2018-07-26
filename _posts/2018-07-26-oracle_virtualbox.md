---
layout: post
title:  "[ORACLE] Virtualbox에서 ORACLE 64bit 나오지 않는 문제 해결"
date:   2018-07-26 17:20:00
author: garimoo
categories: Database
---

## 문제 사항

virtualbox에서 oracle 64bit을 다운로드받으려 하는데 계속 32bit만 나왔다.
oracle linux 포함 다른 linux들도 64bit는 나오지 않았다!
내 데탑은 64bit인데!

![Image](/assets/20180726/1.png)

찾아 보니 컴퓨터의 virtualization 속성이 enable 되어야 한다고 한다.
내 컴퓨터에서 현재 어떤 상태인지 보려면 `작업관리자` 에서 확인할 수 있다.
![Image](/assets/20180726/3.png)
여기 가상화가 (지금은 설정을 완료해서 사용이지만) 미사용이라고 되어 있을 것이다.

## 해결 방법
1. 부팅시 BIOS 접속
- Window10에서 BIOS 접속하려면 `shift + 전원 종료`로 종료하고, `f2 + 부팅버튼`으로 올려야 한다.
- virtualization 속성 enable한다.

2. 프로그램 및 기능 > Windows 기능 켜기/끄기
![Image](/assets/20180726/4.png)
- 만약 여기서 Hyper-V 가 켜져있다면 끄기!
- 재부팅 (자동)

------------

이렇게 하고 다시 virtualbox를 들어가면 oracle 64bit를 설정할 수 있다.
![Image](/assets/20180726/2.png)
