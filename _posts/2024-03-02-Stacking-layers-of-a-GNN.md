---
layout: post
title: "[7.3] Stacking layers of a GNN"
date:   2024-03-02
excerpt: "Stacking Layers : Layer Connectivity"
tag: 
 - Machine Learning with Graphs
 - Machine Learning
comments: true
---

## Stacking GNN Layers

저번 강의에서는 Single GNN layer를 어떻게 구성해야하는지에 대해 이야기했다. 이번 시간에는 Single Layer를 stacking 하여 Multi-Layer로 만드는 방법에 대해 이야기하고자 한다.

Neural Network의 layer를 늘리는 가장 기본적인 방법은 순차적으로 stacking 하는 것이다. 

<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Stacking-layers-of-a-GNN/example.png" style="width: 30%"/>
</p>

<br>

하지만 GNN의 경우 이렇게 sequential한 stacking을 많이 하면 over-smoothing이 발생한다.

<br>
<br>
<br>

## Receptive Field of a GNN

Over-smoothing을 설명하기 위해 먼저 Receptive Field에 대해 알아보자.

Receptive Field란, 내가 관심이 있는 노드의 임베딩을 결정하는 노드들의 집합을 의미한다. 예를 들어, K-layer의 GNN의 경우, receptive field는 내가 관심있는 노드의 K-hop neighborhood가 되는 것이다.

<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Stacking-layers-of-a-GNN/example2.png" style="width: 60%"/>
</p>

<br>

내가 관심이 있는 노드의 수가 늘어날 수록 이러한 receptive field(shared neighbors)는 기하급수적으로 증가한다.

<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Stacking-layers-of-a-GNN/example3.png" style="width: 60%"/>
</p>

<br>
<br>
<br>

## Over-Smoothing

Over-smoothing이란, 모든 Node의 임베딩이 너무 유사해져서 두 Node의 차이를 학습할 수 없게 되는 것을 의미한다.

Node의 임베딩은 receptive field에 의해 결정된다. 그렇다면 두 Node가 highly-overlapped receptive field를 가진다면, 두 Node의 임베딩은 굉장히 유사해질 것이다.

따라서 GNN Layer를 많이 Stacking 하면, Node들이 highly-overlapped receptive field를 가지고, Node 임베딩이 유사해지며, over-smoothing 문제를 겪게 된다.

따라서 GNN의 Layer를 추가할때는 주의해야 한다. 그렇다면 GNN의 layer는 적지만 expressive power는 키우려면 어떻게 해야할까?

<br>
<br>
<br>

## Expressive Power for Shallow GNNs

#### Solution 1 : Increase the expressive power within each GNN Layer

이전에 배웠던 내용에서 한 layer 내의 aggregation/transformation 단계에서 사용하는 함수를 black box로 나타내었다. 이러한 aggregation/transformation 단계를 단순한 linear layer로 나타내는 것이 아닌, 예를들어 3-layer MLP로 만들면 한 layer 내에서의 복잡성을 증가시킬 수 있다.

<br>

#### Solution 2 : Add layers that do not pass messages

GNN Layer를 추가하는 것이 over-smoothing을 초래한다면, message passing을 하지 않는 layer를 추가하면 된다. 

아래의 그림과 같이 MLP Layer를 앞/뒤에 추가하여 Pre/Post-Processing을 하는 방식으로 GNN의 복잡성은 키우지만 over-smoothing은 피할 수 있다.

<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Stacking-layers-of-a-GNN/example4.png" style="width: 70%"/>
</p>

<br>
<br>
<br>

## Idea of Skip Connections

위 두 방법을 사용했음에도 여전히 추가적인 GNN Layer를 필요로한다면, skip connection을 추가하는 방법이 있다. 

Skip connection은 GNN에 일종의 shortcut을 만들어 초반부 layer의 impact를 강하게 만든다. Skip connection은 message를 transform한 후, transform 이전의 message와 transform된 message를 sum하거나 aggregate하는 방식이다.

<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Stacking-layers-of-a-GNN/example5.png" style="width: 70%"/>
</p>

<br>

Skip connection의 idea는 이해했지만, 정확히 어떤 원리에 의해 성능이 나오는걸까?

Skip connection은 여러 모델의 mixture를 만든다고 볼 수 있다. 아래의 그림 중 좌측을 보면, skip connection이 추가된 경우, 모듈을 포함하는 경로와 모듈을 skip하는 경로, 2가지의 갈래가 생기게 된다.

이러한 구조로 인해 가능한 모델은 path의 수가 \\(2^n\\)개가 되고, shallow GNN과 deep GNN의 mixture를 얻게 된다. 우측의 그림처럼 3개의 skip connection이 있다면 깊이가 1, 2, 3인 GNN 모델들의 mixture인 모델을 가지게 되는 것이다.

<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Stacking-layers-of-a-GNN/example6.png" style="width: 70%"/>
</p>

<br>

수식적으로 접근하자면 아래와 같이 표현할 수 있다. 위 수식은 이전에 배웠던 message transformation을 수행한 형태를 표현한 것이고, 아래의 수식은 skip connection이 추가된 형태로, raw message를 더해주는 것을 확인할 수 있다.

<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Stacking-layers-of-a-GNN/example7.png" style="width: 40%"/>
</p>

<br>

추가적으로 skip connection을 설계할 때는 1개만이 아닌, 여러 layer를 skip하게도 만들 수 있다.

<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Stacking-layers-of-a-GNN/example8.png" style="width: 30%"/>
</p>

<br>
<br>
<br>

### 출처, 더 궁금하다면?
[Stanford CS224W: Machine Learning with Graphs | 2021 | Lecture 7.3 - Stacking layers of a GNN](https://youtu.be/ew1cnUjRgl4?si=rpnBFyMnVWv7hkXr){:target="_blank"}