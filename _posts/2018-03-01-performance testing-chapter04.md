﻿---
layout: post
title: "Performance Testing Guidance for Web Applications Chapter 4"
---
<br/>

이 글은 Microsoft의 Performance Testing Guidance for Web Applications 에 대한 글로, **직역과 오역이 다분히 많고, 저의 개인적인 생각을 추가하여 정리한 글** 임을 서두에 밝힙니다.

## Part 4 - Identify Performance Acceptance Criteria

### Chapter 9 - Determining Performance Testing Objectives
- 성능 테스트의 목표는 변화, 잠재적 위험 및 개선 기회를 파악하는 것이다.
- 프로젝트 라이프사이클 초기에 성능 테스트의 목표를 수집하는 것이 가장 중요하지만, 정기적으로 이러한 목표를 다시 설정하고 새로운 목표를 추가할 것인지 묻는 것이 중요하다

#### 성능 테스트의 목적 결정
- 전체 목표 결정
- 프로젝트 계획 검토
- 아키텍처 검토
- 팀원들에게 성과 관련 관심사 물어보기

#### 리소스 사용 목표 및 임계 값 예상
- 테스터가 목표 및 임계 값을 결정하는 것은 적절하지 않다.
- 관련 정보를 팀 구성원에게 제공해야 한다.
- 성과 측정 기준을 명확하게 하고, 이해 관계자와 협력해야 한다.

#### 리소스 예산 예측

#### 지표 식별
- 수집된 목표가 충족되는지를 나타내는 지표로 매핑하는 스프레드 시트를 작성해야 한다.
- 테스트를 왜곡시키지 않고 지표를 수집하는 방법을 생각해야 한다.
- 동시에 수집하고자 하는 다른 데이터를 수집하는 방법이 명확하지 않은 경우 다른 팀과 협력해서 최적의 접근 방식을 결정해야 한다.

#### 결과 전달
- 객관적인 결과는 어플리케이션의 릴리스 적합성을 결정하는 것 보다 더 중요한 정보로 사용될 수 있다.
- 따라서 결과를 자유롭게 공유하는 것이 유익하다.
- 성능 저하의 원인을 잠재적으로 판단해서 보고하지 말고, 증상과 상태를 보고해야 한다. 잘못된 원인을 보고하면 신뢰가 손상될 수 있다.

### Chapter 10 - Quantifying End-User Response Time Goals
- 어플리케이션 사용자는 어플리케이션의 속도가 느려지는지의 여부를 확인하지만, '너무 오래'에 대한 값을 알지 못하거나 신경 쓰지 않는다.
- 따라서 사용자의 인식을 숫자로 변환하는 방법에 대해 알아야 한다.

#### 어플리케이션의 기능 및 사용량 결정
- 성능을 특성화 해야 할 시나리오를 먼저 식별해야 한다.
 - 자주 사용하는 시나리오
 - 성능 집약적인 시나리오
 - 비즈니스에 중요한 시나리오
 - 가시성이 좋은 시나리오

#### 성능의 요구 사항 및 목표
- 프로젝트 문서 및 관련 계약 검토
- 라이브한 결정에 영향을 줄 수 있는 인터뷰 이해 관계자와 상의해야 한다
- 어플리케이션과 관련된 표준 사항이 있는지 검증해야 한다.

#### 성과 요구 사항 및 목표의 정량화
- 요구사항을 목표와 분리해야 한다.
- 파악된 성능 목표의 수를 정량화 해야 한다.

#### 성과 요구 사항과 목표를 기록해야 한다.
- 목표와 요구 사항을 기록하는 데에 선호되는 방법은 도구에 따라 다르다.
- 팀은 요구 사항과 목표를 관리하지만, 숫자로 정리하는 것도 중요하다.

### Chapter 11 - Consolidating Various Types of Performance Acceptance Criteria
- 시스템의 사용자, 시스템의 비즈니스 소유자 및 프로젝트 팀의  관점, 규정 준수 기대치 및 관련 기술에 기반하여 다양한 유형의 성능 허용 기준을 통합하는 방법을 이해해야 한다.

#### 최종 사용자의 요구 사항 조사
- 가장 중요한 성능 특성은 어플리케이션의 사용자가 성능 저하로 실망하지 않아야 한다는 것이다.
- 어플리케이션이 얼마나 많은 사용자를 지원하는지, 얼마나 많은 데이터를 처리할 수 있는지, 리소스를 얼마나 효율적으로 사용하는지는 문제가 되지 않는다.
- 사용자는 성능 테스트의 결과를 알지 못하거나 신경 쓰지 않아도 된다. 프로그램이 느린지 아닌지만 판단한다.
- 최종 사용자의 감정을 정량화하는 것은 어려운 일이지만 불가능한 것은 아니다.

#### 비즈니스의 요구 사항 수집
- 배포 날짜가 언제인지, 제한된 수의 사용자에게 초기 릴리스가 허용되는지 여부 등을 알아야 한다.
-  중요한 기능을 제거하여 시간과 예산에 따라 원하는 성능을 얻을 수 있는 상황이 있다면 고려해야 한다.
- 내부 요구 사항(처리량, 응답성, 용량 등)뿐 아니라 외부 비즈니스의 요구 사항도 영향받을 수 있다.

#### 기술 요구 사항 결정
- 기술의 요구 사항은 특정 자원과 관련된 특정 메트릭 대상 및 임계 값으로 구성된다.

#### __성능 테스트의 목적 수립__
- 튜닝이 필요한 병목 현상을 감지한다
- 확장성과 용량 계획을 위한 입력 데이터를 제공한다

#### 성능 특성 비교, 통합
- 최종 사용자 규정 준수 요구사항을 달성하는 데에 필요한 기술적 특성을 결정한다.
- 수정된 기술 요구 사항을 달성하기 위한 일정, 자원의 비용을 예측한다.
- 성능 테스트를 수행하는 전반적인 목표를 검토하여 테스트 목적이 기술적 요구 사항과 비즈니스 요구 사항을 지원하는지 확인한다.

#### 성능 계획 검토 및 업데이트