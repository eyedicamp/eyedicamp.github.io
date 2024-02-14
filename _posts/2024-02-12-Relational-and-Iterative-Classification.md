---
layout: post
title: "[5.2] Relational and Iterative Classification"
date:   2024-02-12
excerpt: "Relational Classification vs Iterative Classification"
tag: 
 - Machine Learning with Graphs
 - Machine Learning
comments: true
---

## Relational Classification

기본적인 아이디어는 이웃 Node들의 Class 확률의 weighted average를 이용하여 Node \\(v\\)의 Class 확률 \\(Y_v\\)를 구하는 것이다.

레이블이 있는 Node는 레이블의 값(\\(Y_{v}^*))으로, 레이블이 없는 Node는 \\(Y_{v} = 0.5\\)로 초기화 한다.

converge가 일어나거나, 미리 설정해둔 iteration 최대값이 될때까지 업데이트한다. 이 때, Node Label과 Network의 구조만 활용하고, Node attribute는 사용하지 않는다.

<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Relational-and-Iterative-Classification/math1.png" style="width: 60%"/>
</p>

<br>

위는 업데이트에 대한 공식으로, Node \\(v\\)가 레이블 \\(c\\)를 가질 확률(\\(P(Y_{u} = c)\\))은 이웃 Node들이 레이블 \\(c\\)를 가질 확률의 합을 degree로 나눈 값, 즉 평균이라고 정한다. 이때, 경우에 따라 \\(A_{v,u})를 곱해 weight를 부여한다.

이러한 Relational Classification은 Convergence가 보장되어있지 않고, Node feature 정보를 사용하지 않는다는 단점이 있다.

아래는 Relational Classification을 초기화하고, Node ID 순서대로 업데이트 하는 예시를 나타내고 있다. 이때, 이미 레이블링 된 Node는 고정으로 두고, 회색 Node만 업데이트를 진행한다.

<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Relational-and-Iterative-Classification/example1.png" style="width: 60%"/>
</p>

<br>

하지만 이러한 Relational Classification은 위에서 언급했듯 Node attribute를 전혀 활용하지 않는다. Node attribute를 활용하려면 어떻게 해야할까?

<br>
<br>
<br>

## Iterative Classification

Iterative Classification은 Node \\(v\\)에 대하여 \\(f_v\\)의 feature vector를 만들고, 이웃 노드의 집합 \\(N_v\\)의 \\(f_v\\)에 대한 summary인 \\(z_v\\)를 활용하여 레이블 \\(Y_v\\)를 예측한다. 이 과정에서 2개의 Classifier를 사용하는데, 이는 아래와 같다.

* \\(\phi_{1}(f_{v}) = \\) feature vector \\(f_v\\)만을 사용하여 Node 레이블을 예측한다.

* \\(\phi_{2}(f_{v}, z_{v}) = \\) feature vector \\(f_v\\)와 Node \\(v\\)의 이웃의 레이블 summary \\(z_v\\)를 활용하여 레이블을 예측한다.

<br>

이 때, \\(z_v\\)란 정확하게 무엇을 의미하는 것일까?

\\(z_v\\)는 기본적으로 vector 형태를 가지며, \\(N_v\\)에 존재하는 레이블의 수를 기록하거나, 가장 흔한 레이블, 혹은 레이블의 종류의 수를 기록하는 등의 방식으로 summary 역할을 한다.

<br>

Iterative Classifier들은 2가지 Phase를 통해 사용된다.

#### Phase 1: Node attribute만을 사용하여 Classify

파이1을 이용하여 모든 노드에 임의의 레이블을 부여해줌
파이2 또한 train할거임. 얘는 f랑 z둘다 활용해서 예측함

페이즈2
iterate진행함 파이1으로 레이블 없는 노드에 레이블 부여, z를 계산하고 파이2를 이용해서 레이블 예측

주변 Y를 토대로 z를 업데이트, z를 활용하여 파이2로 Y예측 이걸 반복. 클래스 레이블이 converge되면 끝



<br>
<br>
<br>

## Correlation

individual한 특징도 네트워크 내에서 연관되어 있다. Graph 내에서 이야기하자면, 가까운 Node들은 같은 색(= label, class)를 갖는 경향성이 있다.

correlation을 갖게하는 2가지 의존성이 존재한다.

<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Message-passing-and-Node-Classification/example2.png" style="width: 50%"/>
</p>

<br>

#### 1. Homophily

Homophily를 한마디로 표현하면 "유유상종"이다. 즉, 유사한 관심사/취미를 갖거나 유사한 특성들을 가진 Node들끼리 관계를 맺게 되고, 가까워진다는 것이다.

#### 2. Influence

Influence는 그와 반대로 "친하면 닮는다"이다. 이미 관계를 맺고 있는 Node들은 서로 영향을 주고받으며 유사한 관심사/취미를 갖게되거나 유사한 특성을 갖게 되는 것이다.

<br>

이러한 영향력을 주고받는 현상들을 어떻게 사용할 수 있을까? 한가지 방법은 "연좌제"이다.

label X와 연결되어있는 Node라면 label X를 가지고 있을 확률이 높다. 따라서 유해한 웹페이지와 같은 경우, 유해한 페이지와 link를 주고받는 페이지의 경우, 해당 페이지도 유해한 페이지일 것으로 추측할 수 있다.

Collective Classification 이 외에도 아래와 같은 다양한 분야에서 활용 가능하다.

<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Message-passing-and-Node-Classification/example3.png" style="width: 60%"/>
</p>

<br>
<br>
<br>

## Collective Classification Overview

correlation을 이용하여 연결된 Node들을 순차적으로 classification을 진행하고자 한다. 이를 위해 Markov Assumption을 사용할 것이다. 이는 Node\\(v\\)의 label\\(Y_v\\)가 Node\\(v\\)의 직속?(거리가 1인) 이웃들 \\(N_v\\)에 의해서만 결정되도록 하는 방식이다. 이를 식으로 나타내면 아래와 같다.

\\[P(Y_{v}) = P(Y_{v} \vert N_{v})\\]

Collective Classification은 다음 3가지 단계를 가진다.

<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Message-passing-and-Node-Classification/steps.png" style="width: 75%"/>
</p>

<br>

#### 1. Local Classifier

Local Classifier는 색이 없던 Gray Node들에게 initial한 레이블을 부여해준다. 이에 Node의 attribute와 feature정보를 이용한 기본적인 Classification 방식이 사용되고, 네트워크에 대한 정보는 사용되지 않는다.

#### 2. Relational Classifier

Relational Classifier는 Node들 간의 correlation을 찾아서 classifier가 이웃의 label/attribute을 이용해 Node 레이블링을  하는 과정이다. 이 과정에서 네트워크에 대한 정보가 사용된다.

#### 3. Collective Inference

Collective Inference 과정에서는 네트워크가 converge 되거나, 사전에 정해둔 수의 iteration이 채워질 때까지 네트워크 전체의 correlation을 반복적으로 전파하는 과정이다. 이 과정에서 네트워크 구조가 최종 결과에 영향을 미친다.

<br>
<br>
<br>

## Problem Setting

앞으로 해결하고자 하는 문제는 다음과 같다.

Unlabeled Node \\(v\\)의 label \\(Y_v\\)를 어떻게 예측하느냐?

각 Node \\(v\\)는 feature vector \\(f_v\\)를 갖는다.

몇몇 Node들은 \\(\color{green}0\\)또는 \\(\color{red}1\\)의 label을 가진다.

모든 feature와 Network가 주어졌을 때, \\(P(Y_{v})\\)를 구하라.

<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Message-passing-and-Node-Classification/problem.png" style="width: 50%"/>
</p>

<br>
<br>
<br>

## Overview

<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/Message-passing-and-Node-Classification/overview.png" style="width: 70%"/>
</p>

<br>


### 출처, 더 궁금하다면?
[Stanford CS224W: ML with Graphs | 2021 | Lecture 5.2 - Relational and Iterative Classification](https://youtu.be/QUO-HQ44EDc?si=mWHllS1wkocR_Xt6){:target="_blank"}