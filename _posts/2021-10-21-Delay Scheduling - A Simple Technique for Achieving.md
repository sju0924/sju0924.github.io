---
title: 'Delay Scheduling A Simple Technique for Achieving
Locality and Fairness in Cluster Scheduling'
categories: [Paper Review]
comments: true
---

# Abstract
유저 간 클러스터를 공유할 필요가 생겨나고 있다. 하지만, *Scheduling* 과 *Data locality* 사이에는 충돌이 발생한다.이 논문에서는 이 문제를 보여준다. 이 논문에서는 locality와 fairness 사이의 충돌을 다루기 위해 *Delay Scheduling* 이라 부르는 간단한 알고리즘을 제안한다. 공평성에 의거해 다음으로 스케줄링 되어야 하는 작업이 로컬 작업에 설치되지 못할 때, 다른 작업들이 설치되도록 잠깐 기다린다. 이 논문에서는 delay scheduling 이 다양한 비후가 있을 때 거의 최적의 데이터 지역성을 얻게 되고 , 공정성을 지키며 스루풋을 2배 이상 증가시킬 수 있다는 것을 알아내었다. 

# Introduction
 
맵리듀스같은 클러스터 컴퓨팅 시스템은 웹 인덱싱같은 batch job 들에 최적화 있지만, 다른 사용 케이스가 나타났다. batch job 을 길게 하고 공통 데이터 셋에 대한 짧은 interactive query 들을 수행하는 다수의 유저들과 공유할 때이다. 이 논문에서는 맵리듀스와 같은 시스템의 효율성 -특히 계산 위치가 입력 데이터의 근처에 있는 data locality- 을 유지하면서 클러스터를 공유할 때의 문제들을 찾는다. <br>
공유 클러스터를사용할 때, 공유 클러스터에서 데이터 통합은 유용했다. 하지만, 많은 집단이 하둡을 사용하기 시작하면서, 하둡의 FIFO 스케줄러 때문에 응답 시간이 느려지기 시작했다. 이 문제를 해결하기 위해, Hadoop Fair Scheduler 를 설계하였고, 이 논문에서 HFS라고 명칭하였다. 이는 두 가지 목표가 있다.
* Fair Scheduling: max-min fair sharing 을 사
용하여 리소스를 공평하게 나눈다. 
* Data locality: 스루풋을 극대화하기 위해 계산을 입력 근처에 위치시킨다.

첫 목표를 달성하려면 스케줄러가 리소스를 작업 수가 바뀔 때마다 재할당해주어야 한다. 새 작업을 할당하기 위해 기존 실행되고 있던 작업에 두 가지 접근을 할 수 있는데
1. 작업을 끝낸다.
2. 작업이 끝날 때까지 기다린다.
가 있다. 전자는 낭비가 생기고 후자는 공정성에 악영향을 끼칠 수 있다.
Delay Scheduling 으로 이런 문제를 해결할 수 있다. 작업이 데이터를 가지고 있는 노드에 스케줄링 기회를 기다리는 시간을 제한하는 것이다.

# Background
하둡은 HDFS 라는 분산 파일 시스템 상에서 작동한다. 
## Delay Scheduling
공정성에 영향을 최소화하고 데이터 로컬리티를 높게 유지하면서 작업량의 균형을 맞추기 위해, 두 질문에 답이 되는 간단한 공평한 공유 알고리즘을 분석하였다.
1. 어떻게 리소스가 새로운 작업에 재할당될 수 있을까?
2. 어떻게 데이터 로컬리티를 달성할 수 있을까?

첫 질문에 답하기 위해 리소스 제할당에 대한 두 접근을 고려하였다. 

1. 기존 작업을 종료한다.
2. 기존 작업이 끝날때까지 기다린다.

전자는 빠르지만 작업량 낭비가 생긴다. 여기서는 작업이 평균 작업 길이보다 길고 클러스터가 다수의 유저에 의해 공유될 때  대기하는 것이 작업 응답 시간에 적은 영향을 주는 것을 보일 것이다. 대기 시간을 사용하는 것을 선택하면서, 로컬리티로 넘어가 보자. fair sharing 이 엄격하게 지켜질 때 두 가지 지역적 문제가 생긴다. `head-of-line scheduling` 과 `sticky slots` 이다. 두 경우에, 스케줄러는 공정성을 지키기 위해 노드에 local data를 사용하지 않고 작업을 실행한다. 여기서 대기량이 로컬리티와 작업 응답 시간에 미치는 영향이 얼마인지 분석할 것이다.

## Naive Fair Sharing Algorithm
작업들 간 공평하게 클러스터를 나누는 간단한 방법은 작업에 거의 하지 않는 여유 슬롯을 언제나 할당해 놓는 것이다. 슬롯이 빠르게 비워질수록 할당 결과는 max-min fairness 를 만족한다. locality 를 달성하기 위해, 하둡 FIFO 스케줄러처럼 local task를 탐욕적으로 탐색할 수 있다.

## Scheduling Responsiveness

## Locality Problems with Naive Fair Sharing
데이터를 포함한 노드에서 실행하는 것이 제일 효율적이지만 그렇지 못할 경우 같은 rack 에서 실행시키는 것이 그렇지 않은 경우보다 효율적이다. 지금은 node locality 만 살펴볼 것이다. 두 가지 문제가 있다.

## Head-of-line Scheduling
규모가 작은 작업에서 일어난다. 작업이 정렬된 리스트의 head 에 도달했을 때, 작업 중 하나가 슬롯이 어느 노드에 있던 간에  비어 있는 다음 슬롯에서 실행하는 것이다.

## Sticky Slots 
규모가 큰 작업에서 일어난다. 같은 슬롯에 반복해서 할당되는 경향이 문제가 된다.

# Hadoop Fair Scheduler Design
HFS 에서는 아래와 같은 결점들을 다루고자 한다.
1. 어떤 사용자는 다른 사용자보다 많은 작업을 실행한다. 사용자가 아니라, 작업의 공평한 공유를 원한다.
2. 사용자들은 작업 스케줄링을 컨트롤하길 원한다.
3. 클러스터에 다수의 긴 작업이 대기하고 있어도 예측할 수 있게 수행되어야 한다.

이는 2단계 scheduling 계층을 사용한다. fair sharing 을 사용하여 pool 전체에 작업 슬롯을 할당하는 것이고, 2단계는 각 pool 이 우선순위 FIFO 를 사용하거나 fair sharing 을 사용하여 작업을 슬롯에 할당하는 것이다. 그러면 pool 끼리 정렬해서 fair sharing 을 달성핼 수 있다. 또한 그 안에서도 내부 정책에 따라 구현한다.

<br>
<br>

이를 `Delay scheduling` 내에서 구현해 보자.
1. 작업이 로컬에 대기하는 시간을 정하기 위해 최대 대기 시간을 설정한다.
2. 작업이 node-local task 를 시작할 수 없을 때, rack locality 달성을 위해 두 가지 수준의 delay scheduling 을 사용한다.
    - 각 작업은 node-local task 만 실행할 수 있는 locality level 이 0일 때 시작
    - 1초 이상 기다리면 locality level 1로 이동하여 rack-local task 를 시작할 수 있음
    - W2 초를 더 기다리면 level 2로 이동하여 off-rack task 를 시작할 수 있음
    - 현재보다 더 로컬작업을 시작하는경우 이전 수준으로 다시 내려감

# Analyze


## IO-heavy load 에서
* Delay scheduling 에서 모든 빈에 대해 로컬리티 99~100% 달성
* FIFO, Naive fair 에서는 job 이 커질수록 로컬리티 증가.

## CPU-heavy workload 에서
* Delay Scheduling 이 small job 에서 FIFO 보다 큰 성능 개선. FIFO 에서 큰 job 을 더 오래 키다려야 하기 때문. Naive 와는 큰 차이 없음

## Mixed-workload 에서
* fair sharing 에서는 small job 에서 빠름.

## small jobs 에서
* 로컬리티 증가, 스루풋 증가

## sticky slot 에서
* 딜레이 스케줄링을 적용하지 않으면 job 수가 많을 수록 로컬리티 감소. 적용시 로컬리티 99~100% 달성