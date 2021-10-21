---
title: 'Improving MapReduce Performance in Heterogeneous Environments'
categories: [Paper Review]
comments: true
---

# Abstract
하둡의 성능은 클러스터 노드가 모두 같고 작업이 선형으로 진행되는 것을 상정한 task scheduler 와 밀접하게 연관되어 있다. 하지만 언제나 노드가 모두 같지는 않다. 늑히 EC2 같은 클라우드 환경에서 그렇다.  이 논문에서는 노드의 성능이 다른 환경에서 하둡의 스케줄러가 심각한 성능 저하를 일으킴을 보여주고, Lost Approxymate Time to End(LATE) 라는 새로운 스케줄링 알고리즘을 고안하였다. 이는  EC2 의 200개의 가상 머신 클러스터에서 성능 향상을 보인다.

# Introduction
 오늘날 인기 있는 컴퓨터 어플리케이션은 수 백만 사용자들이 사용하는 인터넷 서비스이다. 이들은 방대한 양의 데이터를 생성하고 처리한다. 이를 처리하는데 하둡이 사용된다. 맵리듀스의 주 장점은 straggler 라고 불리는, 사용 가능하지만 성능이 좋지 않은 노드를 다른 노드로 대체하는 것이다. 이 논문에서는 이 과정이 straggler 가 모호한, 노드의 성능이 다양한 환경에서도 잘 동작하지만, 이것이 성능 저하를 일으킴을 보여주고 성능을 최대화 하기 위한 방법을 다룬다. 

# Background
하둡에서는 Response time 을 줄이기 위해 추측에 의거한 실행을 수행한다. 하지만 이기종 환경에서는 제대로 수행하지 못 한다. 하둡의 Default scheduler 는 추측에 의거한 실행을 수행할 때 노드의 특성을 고려하지 않기 때문이다. 무조건 먼저 요구한 노드가 빠르다고 판단한다. 또한 작업 진행 상황을 Progress Percent로 나타내는데, 현재 위치만을 나타내고 실제로 일을 얼마나 했는지는 나타내지 못한다. 즉, Task들은 항상 시간당 동일한 일을 하지 않는다.
## Hadoop Default Scheduler
Idling node 가 있는 경우 스케줄러는 
1. Failed Tasks
2. Non running tasks
3. Speculative tasks

 순서로 작업을 할당한다. <br>
 순서를 정할 때 Progress Score 항목을 사용한다. 이는 각 작업을 Phase 로 쪼갠 후 진행상황을 반영하였다. 모든 Phase 에 동일한 가중치가 주어져 시간이 더 걸리는 Phase를 Progress Score을 통해 알 수 없다.<br>
 straggler 인지 판단하는 기준이 Progress Score의 평균에서 0.2를 뺀 값이고, 이값보다 작고 1분보다 더 많이 남아 있는 것이다. straggler 는 `Speculative Execution` 의 대상이 된다.

 ## Speculative Execution
작업의 응답속도를 줄이는 것이 목적이지만, 느린 작업을 복제하는 것만으로 끝나지 않는다.
1. Speculative Execution 에는 비용이 든다.
    - 기준값 이하인 모든 Task를 느린 작업으로 인식하여 추가 작업이 할단되는데, 대규모일 수 있고 다른 작업에 영향을 끼칠 수 있다.
2. 노드를 잘 선택해야 한다.
    - 성능 차이가 나는 노드를 제대로 파악하지 않고 요청 순서대로 작업을 할당한다.
3. 느린 작업이 가급적 빨리 발견되어야 한다.
    - 느린 노드에서 작업이 오래 진행되면 다른 노드에 배치할 수 없을 때까지 진행될 수 있고, 돌이킬 수 없다.

# The LATE Scheduler 
LATE 알고리즘은 아래와 같은 관점으로 만들어졌다.

* 어떤 작업을 Speculatively 하게 싱행할 지 빠르게 결정한다.
* 작업 종료 시간을 사용하낟.
* 노드들의 성능은 동일하지 않다.
* 리소스는 소중하다.

또한 LATE 의 특징에는
1. Speculative Execution 할 수 있는 작업 수를 제한한다.
    - 모든 작업에는 비용이 소모되어서, 제한을 통해 네트워크와 I/O 에 드는 부하를 최소화하여 기존 작업에 끼치는 영향을 줄인다.
2. 작업 종료까지 남은 시간을 기준으로 느린 작업을 찾는다.
    -  작업 추가 배치 시의 기준을 Progress Score이 아닌 남은 시간을 기준으로 한다.
3. 노드의 성능을 분석해 빠른 노드에만 추가 작업을 제공한다.

### 동작 매커니즘
1. 현재 작업들이 추가되면 Spectaculer Execution 수의 상한선을 넘었는지 확인한다.
2. Non running node 중 성능이 상위 25% 안에 있는 노드가 있는지 조사한다.
    - 25% 안에 들지 않더라도 추가 Speculative task 를 받을 수 있음
    - 선정 기준은 지금까지 성공시킨 작업의 Progress Score 의 합으로 결정
3. 작업 완료까지 남은 시간을 보고, 가장 늦게 끝날 작업을 최우선으로 배치한다.

> Progress Rate = Progress/T 
> Estimated Time Left = (1-ProgressScore)/ProgressRate
# Analyze

default 스케줄러나 no speculation 과 비교해서, 모든 상황에서 빠른 속도를 보였지만 Straggler 가 있을 때 훨씬 빠른 속도를 보였다.