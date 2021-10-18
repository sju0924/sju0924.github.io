---
title: 'MapReduce: Simplified Data Processing on Large Clusters'
categories: [Paper Review]
comments: true
---

# Abstract
`맵리듀스`는 큰 규모의 데이터 셋을 처리하고 만들기 위한 프로그래밍 모델이자 구현체이다. 사용자는 key/value쌍을 처리하는 *map* 함수를 정의한다. 이는 intermediate key/value 쌍을 만들고, *reduce* 쌍을 만들어 같은 intermediate key를 가지고 있는 모든 intermediate value들을 합친다. 현실 세계의 많은 작업들을 이 모델로 나타낼 수 있다.

# Introduction
최근 구글은 방대한 양의 raw data를 처리하기 위한 방법들을 고안해 왔다. 그 중 대부분은 수백 수천 개의 분산되어 있는 컴퓨터 상에서 수행되어야 했다. 여기에서는 어떻게 계산을 병렬화할 것이고, 데이터를 분산시키고, failure들을 다루는 방법은 대량의 복잡한 코드로 이루어지는 간단한 계산을 모호하게 하는 데 공모한다. <br>
이런 것들 때문에, 간단한 계산을 나타낼 수 있지만 복잡한 병렬화, fault-tolerance, 데이터 분산, 로드 밸런싱에 대한 디테일을 숨기는 추상~abstraction~ 을 고안했다.  이는 `map` 과 `reduce` 에서 영감을 받았고 , key/value 쌍으로 이루어져 있는 집합으로 계산하기 위해 `map` 과정을 수행하고, 같은 키를 가진 모든 value 들을 묶기 위해 `reduce` 과정을 수행한다. 이 작업은 병렬 자동화와 대규모 연산의 분산을 가능하게 해 준다. 이는 대규모 PC 클러스터가 좋은 성능을 내게 해 준다. <br>
본 논문의 개요는 아래와 같다.
2. 기본 프로그래밍 모델과 예시
3. 클러스터 기반 컴퓨팅 환경에 맞춰진 맵리듀스 인터페이스의 구현
4. 유용한 프로그래밍 모델의 세부 사항
5. 여러 작업을 수행할 때 구현체의 성능 측정
6. 맵리듀스의 쓰임새

# Programming Model
계산 작업은 key/value 쌍의 *input* 집합을 받아, key/value 쌍의 *output* 의 집합을 만든다. <br>
`Map` 은 유저에 의해 쓰여지고, 입력 쌍을 받아  *intermediate* key/value 쌍을 만든다. 맵리듀스 라이브러리는 intermediate key I와 그 value 를 받아들인다. 그리고 value의 더 작은 집합