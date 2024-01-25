---
layout: post
title: "Random Walk Approaches for Node"
date:   2024-01-21
excerpt: "Random Walk란?"
tag: 
 - Machine Learning with Graphs
 - Machine Learning
comments: true
---

## Notation

* Node u에 대한 embedding vector \\(z_u\\)
* u에서 시작한 random walk로 Node v에 도달할 확률 \\(P(v \vert z_u)\\)

* Softmax function : K개의 실수로 이루어진 vector를 총합이 1인 확률 K개로 변환시키기 위해 사용

\\(\sigma_i(z) = \frac{e^{z_i}}{\sum_{j=1}^{K}e^{z_j}}\\)

* Sigmoid function : 실수를 (0, 1)의 범위로 변환하기 위해 사용

\\(S(x) = \frac{1}{1+e^{-x}}\\)

<br>
<br>
<br>

## Random Walk

출발 Node에서 연결된 edge를 따라서 랜덤하게 아무 neighbor Node로 정해진 횟수만큼 이동하는 것을 Random Walk라고 한다. 이때, 이미 지나온 edge를 중복해서 지나갈 수 있다.

<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Random-Walk-Approaches-for-Node/random-walk.png" style="width: 60%"/>
</p>

<br>

Node u와 v가 Graph 내에서 Random Walk를 통해 마주칠 확률이 Graph 내에서의 두 Node의 유사도이다.

이러한 Random Walk를 사용하는 이유는, 지엽적이거나 광범위한 범위에서의 유사성을 모두 이야기할 수 있고, 학습과정에서 모든 node pair가 아닌 Random Walk에서 마주치는 pair들만 고려하면 되어 효율적이기 때문이다.

<br>
<br>
<br>

## Unsupervised Feature Learning

Unsupervised Feature Learning에서는 Graph내에서 유사도가 높은 Node들을 Embedding에서도 가깝게 만들려고 한다. 그렇다면 u라는 Node가 주어졌을 때, 가까운 Node들을 어떻게 정의할 수 있을까?

\\(N_R(u)\\)를 R이라는 Random Walk 방식으로 얻은 Node u의 neighborhood라고 했을 때, 우리의 목적은 \\(f:u \to \mathbb{R}^{d}:f(u) = z_{u}\\)의 mapping을 구하는 것이고, 이 과정에서 얻고자하는 것은 아래와 같다.

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Random-Walk-Approaches-for-Node/math.png" style="width: 40%"/>
</p>

<br>

위 식에서 \\(logP(N_{R}(u) \vert z_u)\\)를 최대로 하는 것이 학습의 목표이다.

이를 위해 먼저 Graph내의 모든 노드 u에서 짧은 고정 길이의 Random walk를 진행하고, 이를 이용해 \\(N_R(u)\\)를 구한다. 이후 위의 식으로 embedding을 최적화하는 것이다.

위 식을 다르게 나타내면 아래와 같다.

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Random-Walk-Approaches-for-Node/math2.png" style="width: 40%"/>
</p>

<br>

여기에 아래와 같이 softmax를 적용하면,

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Random-Walk-Approaches-for-Node/math3.png" style="width: 40%"/>
</p>

<br>

다음과 같은 최종 식이 나온다.

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Random-Walk-Approaches-for-Node/math4.png" style="width: 50%"/>
</p>

<br>

위 식에서 Random walk embedding을 최적화한다는 것은 곧 \\(L\\)을 최소로 만드는 \\(z_u\\)를 찾는 것이다. 하지만 위의 식에서 확인할 수 있듯, \\(V\\)에 속하는 모든 \\(u\\)와 \\(N_{R}(u)\\)에 속하는 모든 \\(v\\)를 확인하는 것은 \\(O(\vert V \vert ^2)\\)만큼의 복잡도를 가진다.

이에 대한 해결책이 Negative sampling이다.

<br>
<br>
<br>

## Negative Sampling

Negative Sampling을 수식으로 설명하면 다음과 같다.

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Random-Walk-Approaches-for-Node/math5.png" style="width: 70%"/>
</p>

<br>

풀어 말하자면, 모든 node에 대해 normalization을 진행하는 것이 아니라, k개의 랜덤한 negative sample(해당 Node와 무관한 Node)들을 추출해 이에 대해서면 normalization을 하는 것이다.

<br>
<br>
<br>

### 출처, 더 궁금하다면?
[Stanford CS224W: Machine Learning with Graphs | 2021 | Lecture 3.1 - Node Embeddings](https://youtu.be/rMq21iY61SE?si=mep2D6x6d0s-5z5R)