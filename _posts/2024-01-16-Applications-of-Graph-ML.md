---
layout: post
title: "Applications of Graph ML"
date:   2024-01-16
excerpt: "그래프 머신러닝 적용 사례"
tag: 
 - Machine Learning with Graphs
 - Machine Learning
comments: true
---

### Classic Graph ML Tasks

Graph ML은 세가지 Level로 구분할 수 있다. Node Level, Link Level, 그리고 Subgraph Level이다. 이번 글에서는 이러한 레벨에 따른 Graph ML의 사례들에 대해 이야기 해보도록 하겠다.

아래는 고전적인 Graph ML의 예시들이다.

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Applications-of-Graph-ML/graph-ml-types.png" style="width: 60%"/>
</p>


<br>
<br>
<br>

### Node-Level ML Tasks

Node Level의 Task는 node의 property를 예측하는 목적을 가지고 있다. 강의에서 그 예시로 사용한 Protein Folding Problem은 각 node(단백질읠 구성하는 아미노산)의 배열을 분석하여 node들의 거리관계(edge)에 대한 상호작용의 계산을 통해 protien의 3차원 구조를 예측한다. 
  
<p align="center">
  <img src="{{site.baseurl}}/assets/img/Applications-of-Graph-ML/protien-folding.png" style="width: 60%"/>
</p>

<br>
<br>
<br>

### Edge-Level ML Tasks

Edge Level의 Task는 주로 두 node 사이의 missing link가 존재하는지를 예상하고 이를 추가할지 말지 결정하는 목적을 가진다.

강의에서 사용한 첫번째 예시는 Recommendation System으로, 유저 및 아이템을 node로, 그 둘 사이의 관계를 edge로 설정한 Bipartite graph로, 관계가 깊은 노드일수록 임베딩을 더 가까이 한다. 이를 통해 내가 선호하는 아이템과 가까운 임베딩에 있는 아이템을 추천하거나, 나와 가까운 임베딩에 있는(나와 유사한) 다른 유저가 선호하는 아이템을 추천할 수 있다.

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Applications-of-Graph-ML/recommendation.png" style="width: 70%"/>
</p>

<br>

두번째 예시는 약을 동시 복용했을 때 나타나는 부작용에 관한 것이다. 이를 combination으로 계산하기에는 약 종류가 너무 많아 불가능하다. 따라서 이를 이전 예제와 마찬가지로 약과 단백질 2가지 종류의 node를 가진 graph로 표현하지만, 이 번에는 약-약 edge, 단백질-단백질 edge 또한 설정하여 약 node들 사이의 Edge를 동시 사용시의 부작용으로 지정한다.

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Applications-of-Graph-ML/biomedical.png" style="width: 70%"/>
</p>


<br>
<br>
<br>

### Community(Subgraph)-Level ML Tasks

Community(Subgraph) Level에서는 전체 Graph의 부분 Graph를 활용하기 위한 학습을 진행한다. 예를 들어 도로의 일부분을 Node로 설정하고, 각 Node들 사이의 연결부를 Edge로 설정하여, Node들의 교통상태를 통해 목적지까지의 예정 시간을 추정할 수 있다.

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Applications-of-Graph-ML/traffic.png" style="width: 70%"/>
</p>

<br>
<br>
<br>

### Graph-Level ML Tasks

Graph Level에서는 Graph 전체에 대한 학습을 진행한다. 이를 통해 신약개발에 쓰일 후보 물질의 분자구조를 만들어내거나, 이미 존재하는 분자를 어떻게 수정하면 원하는 작용을 추가할 수 있는지 찾아낸다. 

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Applications-of-Graph-ML/drug.png" style="width: 70%"/>
</p>

또는 입자를 node로, 입자 간 상호작용을 edge로 설정하여 물리엔진 시뮬레이션을 하는데에 활용하는 방법도 존재한다.

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Applications-of-Graph-ML/physics.png" style="width: 70%"/>
</p>


<br>
<br>
<br>

### 출처, 더 궁금하다면?
[Stanford CS224W: Machine Learning with Graphs | 2021 | Lecture 1.2 - Applications of Graph ML](https://www.youtube.com/watch?v=aBHC6xzx9YI&list=PLoROMvodv4rPLKxIpqhjhPgdQy7imNkDn&index=2&t=0s)