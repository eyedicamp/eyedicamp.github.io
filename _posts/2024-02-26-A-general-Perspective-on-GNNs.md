---
layout: post
title: "[7.1] A general Perspective on GNNs"
date:   2024-02-26
excerpt: "GNN Framework를 만들기 위해 고려해야할 요소"
tag: 
 - Machine Learning with Graphs
 - Machine Learning
comments: true
---

## A General GNN Framework

7강에서는 이전시간에 배운 GNN을 generalize하고, 수학적으로 표현하는 것에 초점을 맞추고자 한다. 본격적으로 내용에 들어가기 이전에 GNN framework를 만들 때 고려해야할 요소 5가지에 대해 알아보자.

먼저 하나의 GNN Layer에는 Message와 Aggregation이 중요한 역할을 한다. 이것들을 어떤 방식으로 사용하느냐에 따라 GCN, GraphSAGE, GAT 등 다양한 방식의 GNN 모델들이 개발되었다.

다음으로 layer들 간의 Layer Connectivity를 고려해야 한다. 이를 통해 layer들을 sequential하게 쌓아올릴지, 혹은 skip을 넣을지를 선택할 수 있다.

Graph augmentation 또한 고려요소이다. Raw input graph와 computational graph는 같지 않으므로, Graph feature나 structure를 어떤방식으로 augmentation할지가 중요하다.

마지막은 Learning Objective이다. GNN을 Supervised/Unsupervised하게 학습시킬지, Node/Edge/Graph 중 어떤 level에서 학습시킬지를 선택해야한다

아래는 위 내용을 나타낸 그림이다.

<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/A-general-Perspective-on-GNNs/example.png" style="width: 75%"/>
</p>

<br>

<br>
<br>
<br>

### 출처, 더 궁금하다면?
[Stanford CS224W: Machine Learning with Graphs | 2021 | Lecture 7.1 - A general Perspective on GNNs](https://youtu.be/RU9uTa_-ZOw?si=cX-NjhcLGiApTtmT){:target="_blank"}