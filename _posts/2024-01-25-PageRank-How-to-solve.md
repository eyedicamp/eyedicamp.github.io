---
layout: post
title: "PageRank: How to solve?"
date:   2024-01-25
excerpt: "PageRank를 풀어내는 수학적 방법"
tag: 
 - Machine Learning with Graphs
 - Machine Learning
comments: true
---

## Power Iteration

이전 강의에 이어서 설명하자면, r을 구하기 위해 iterative한(반복적인) 절차를 거쳐 r을 업데이트 한다. 각 Node에 initial한 PageRank를 부여하고, converge 될 때까지 반복 한다. 이 때, converge는 이전 시간의 값과 비교했을 때 변화량이 \\(\varepsilon\\)보다 작으면 converge한 것으로 본다.

일반적으로 50step 정도에 converge가 되므로 굉장히 scalable하다.

<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/PageRank-How-to-solve/power.png" style="width: 80%"/>
</p>

<br>

아래는 위의 내용에 대한 예시이다.

<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/PageRank-How-to-solve/example1.png" style="width: 60%"/>
</p>

<br>

<br>
<br>
<br>

## PageRank : Questions and Problems

이렇게 PageRank를 구하는 과정에서 우리는 3가지 궁금증이 생긴다.

* 과연 converge 하는가?

* 우리가 원하는대로 converge 하는가?

* 결과가 합리적인가?

<br>

그리고 위 3가지 조건에 모두 부합하는 PageRank를 설계하기 위해서 우리는 2가지의 문제를 해결해야한다.

* Dead Ends : out-link가 없는 Node
 - 중요도가 새어나가 Graph 전체의 중요도를 0으로 만듦

* Spider Traps : out-link가 특정 Node 그룹 내에서만 돌도록 형성되어 있음
 - 모든 중요도를 빨아들여 중요도를 독식함





<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/PageRank/math1.png" style="width: 15%"/>
</p>

<br>

이러한 행렬 연산 과정을 예시로 보면 아래와 같다.

<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/PageRank/example2.png" style="width: 50%"/>
</p>

<br>

## Connection to Random Walk

수식적으로 풀어보았으니 이제 직관적으로도 이해해보자. 이제는 PageRank를 이전에 배운 Random Walk와 연결지어 생각해보려고 한다.

uniformly(모든 확률이 동등하게) random하게 out-link를 클릭하며 웹서핑을 하는 사람이 있다고 가정해보자. 이러한 과정이 무한하게 지속되면 웹서퍼는 어디에 존재하게 될까?

서퍼가 \\(t\\)라는 시점에 \\(i\\)라는 페이지에 존재할 가능성을 \\(p(t)\\)라는 벡터에서 i번째에 들어가는 값이라고 하자. 그러면 \\(p(t)\\)는 페이지들에 대한 확률분포가 된다.

그러면 \\(t+1\\) 시점에 웹서퍼는 행렬 \\(M\\)에 \\(p(t)\\)를 곱한 확률 분포를 가지게 된다.

그렇다면 어느 시점에 웹서퍼가 \\(p(t+1) = M \cdot p(t) = p(t)\\)인 상태에 도달했다고 가정해보자. 이 지점에서는 stationary distribution(정적분포) 상태가 된다. 그리고 이는 이전에 살펴본 \\(r = M \cdot r\\)과 같으므로, \\(r\\) 자체가 random walk에서의 stationary distribution이라는 의미이다.

이러한 flow 기반의 수식이 flow로 해석될 수도 있고, 방금과 같이 무한히 random walk하는 사람이 시작지점과 무관하게 stationary distribution한 곳으로 회귀하는 것으로 해석할 수도 있다. 이를 계산하는 것이 곧 위에서 보았던 재귀적인 수식을 계산하는 것과 동일한 것이다.

여기서 한가지의 관점을 더 추가해보자.

위에서 본 \\(r = M \cdot r\\) 식을 \\(1 \cdot r = M \cdot r\\)이라고 본다면, 이는 eigenvector/eigenvalue 공식과도 구조가 같아진다. 즉, \\(M\\)이 1이라는 eigenvalue를 가질 때, eigenvector가 \\(r\\)인 것이다.

그리고 이때의 r은 \\(M(M(M(...Mu)))\\)처럼 웹서퍼가 \\(u\\)에서 시작하여 무한히 많은 random walk후의 위치이다.

다음 포스트에서는 이러한 \\(r\\)을 효율적으로 구하는 방법인 Power Iteration에 대해 배워보도록 하겠다.


<br>
<br>
<br>

## summary

<br>

<p align="center">
  <img src="{{site.baseurl}}/assets/img/PageRank/summary.png" style="width: 65%"/>
</p>

<br>

### 출처, 더 궁금하다면?
[Stanford CS224W: Machine Learning with Graphs | 2021 | Lecture 4.2 - PageRank: How to Solve?](https://youtu.be/rK2ZBmQHVVs?si=05J4VewpcLELSUjZ)