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

\\(S(x) = \frac{1}{1+e^{-x}})

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



<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Node-Embeddings/embedding.png" style="width: 70%"/>
</p>

<br>
<br>
<br>



<br>
<br>
<br>

### 출처, 더 궁금하다면?
[Stanford CS224W: Machine Learning with Graphs | 2021 | Lecture 3.1 - Node Embeddings](https://youtu.be/rMq21iY61SE?si=mep2D6x6d0s-5z5R)