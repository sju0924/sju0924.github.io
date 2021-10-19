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
`Map` 은 유저에 의해 쓰여지고, 입력 쌍을 받아  *intermediate* key/value 쌍을 만든다. 맵리듀스 라이브러리는 intermediate key I와 그 value 를 받아들인다. 그리고 같은 Intermediate key를 가진 value를 한데 묶고 이를 *Reduce* 함수로 넘긴다.<br>
`Reduce` 함수 또한 사용자에 의 쓰여졌고, intermediate key I 와 그 키들의 value 들의 집합을 받아드인다. 이는 더 작은 value 의 집합을 만들기 위해 합쳐진다. `Reduce` 함수에 전달되는 intermediate 데이터는 iterator 형태로 전달되어 메모리에 다 들어가지 않을 정도로 큰 데이터도 처리할 수 있다.
## 예시
어떤 문서에 들어간 단어의 개수를 세는 WordCount 함수를 생각해 보자. 그러면 `Map` 함수는 입력으로 <문서 이름, 문서 내용>의 key/value 쌍을 받는다. 그러면 맵 함수는 <단어, 1> 의 intermediate key/value 를 생성한다. 이는 <단어, list of counts> 의 형태로 리듀스 함수의 입력으로 들어가 <단어, 단어 개수>를 출력한다.

# Implementation
 `Map` 동작은 다수의 기계에 분산되어 있다. 자동으로 입력 데이터를 M개로 split 하여 서로 다른 장치에서 병렬로 처리된다. `Reduce` 동작은 intermediate key space를 R개로 분산시켜서 작업한다.

 ## Master Data Structures
 각 맵이나 리듀스 작업들은 idle, in-progress, complete 의 상태를 가지고 있다. idle을 제외하고 worker machine 의 정보까지 갖고 있다. 맵 작업이 끝나면 결과를 마스터가 받아서 리듀스 작업을 하는 worker에게 push한다.

 ## Fault Tolerance
 ### worker failure
 마스터는 worker들에 주기적으로 ping을 날려서 확인하고 응답이 없을 시 fail로 인식한다. 그 worker에 의해 처리된 맵 작업은 idle 상태로 돌아오고 다시 스케줄링된다. 또한 처리 중이었던 맵이나 리듀스 작업 또한 idle 상태로 돌아와서 다시 스케줄링된다.<br>
 완료된 맵 작업들까지 다시 실행하는 이유는, 맵의 결과들이 failed worker의 로컬에 있기 때문이다. 반면 완료된 리듀스 작업은 global file system 에 저장되기 때문에 다시 실행할 필요는 없다.

### Master Failure
마스터가 주기적으로 checkPoint를 만들어 마스터가 죽었을 때 마지막 체크포인트에서 다시 시작할 수 있다. 하지만 마스터가 하나라면 Master Failure 가 일어나고, 맵리듀스 연산을 다시 시작해야 한다.

 ## Locality
 맵리듀스 마스터가 입력 파일의 위치 정보를 받고, 해당 데이터의 복제본이 있는 머신에서 맵 작업을 스케줄링하여 입력 데이터가 지역적으로~locally~ 읽혀 네트워크 Bandwidth를 쓰지 않는다.

 ## Task Granulrity
 M과 R이 worker의 수보다 많을수록 이상적이다.

 ## Backup Tasks
 전체 시간을 늦추는 흔한 원인 중 하나는 마지막 몇 단계를 비정상적으로 길게 처리하는 머신 ~straggler~ 이다. 이들의 방해를 줄이기 위해서 마스터가 어떤 작업이 끝나갈 때 남은 in-progress 작업에 대해 backup execution을 스케줄링한다. 그래서 원래나 백업 작업 중 하나가 끝나면 complete 상태가 된다. 

 # Conclusion
 맵 리듀스의 성공의 이유에는
 1. 세부 사항은 라이브러리에 숨기고 모델을 사용하기 쉽게 만들어 분산이나 병렬 프로그래밍에 경험이 없는 사용자도 쉽게 사용할 수 있다.
 2. 다양한 종류의 문제들을 맵리듀스 연산으로 표현할 수 있다.
 3. 수 천대의 컴퓨터로 이루어진 클러스터에 적용될 수 있다.

 또한 이 연구로 배운 점은
 1. 프로그래밍 모델을 제한시키는 것이 병렬화나 분산화하기 쉽고 fault tolerant 한 연산을 만들 수 있다.
 2. 네트워크 bandwidth 는 한정된 자원이라 이를 최적화하는 것이 중요하다.
 3. 작업을 중복해서 수행하는 것은 느린 머신의 영향을 줄이거나 machine failure 을 해결하는 데 사용될 수 있다.