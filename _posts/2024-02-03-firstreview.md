---

layout: single

title: "[논문 리뷰] Learning the Graphical Structure of Electronic Health Records with Graph Convolutional Transformer (AAAI, 2019)"

decription: "Medical AI Paper review"

comments: true

published: true

categories: Medical AI

typora-copy-images-to: ../assets/images/2024-02-03-firstreview

---



## Overview

이번에 리뷰할 논문은 2019년에 AAAI에 출판된 Learning the Graphical Structure of Electronic Health Records with Graph Convolutional Transformer이라는 논문이다. 이 논문은 의료 분야의 전자 건강 기록이라 불리는 Electronic Health Records (EHR)이라 불리는 데이터를 Graph Convolution Network (GCN)과 Transformer를 이용하여 EHR의 representation을 학습한 연구이다.



## Introduction

기존의 연구에서는 EHR 시스템에서 수집된 대규모 의료 기록은 진단 예측, 의료 개념 표현 학습, 해석 가능한 예측 과 같은 다양한 task들에서 높은 성능을 보였다. 이를 통해 EHR은 진단 코드, 검사 결과, 진료 등 심지어 환자 자체를 효과적으로 표현하는 방법을 배우는 것이 EHR 관련 작업을 수행하는 데에 필수적이라고 볼 수 있다.

EHR 데이터의 경우 보통 계층적 그래프로 표현될 수 있는 관계형 데이터 베이스에 저장되어진다. 그러나 이 연구의 전까지는 일반적으로 데이터를 bag-of-features의 방식으로 처리하였다. 이는 의사의 결정 과정을 반영하는 Graph적인 형태를 무시하게 된다.

아래의 그림을 통해 예시를 들 수 있다.

![image-20240203142959322](https://github.com/mmistakes/minimal-mistakes/assets/126770258/7c3682a9-8040-40d4-a9a6-b23e479309c6)

그림의 Graph Structure를 bag-of-features로 처리시 Benzonatate라는 치료가 Cough에 의해 내려진 구조이지만, 이를 무시하게 될 수도 있다는 것이다. 즉, 기존의 방법을 사용할 시 정보 손실을 일으킬 수 있다.

이런 문제점을 통해 해당 연구는 아래의 3가지 Contribution을 가지고 있다.

---

- 본 연구는 EHR 데이터의 숨겨진 구조와 지도학습의 예측 작업의 공동 학습을 성공적으로 수행한 최초의 연구이다.

- Attention Mask와 Conditional probability 기반 규제 형태의 prior knowledge를 활용하여 Self-attention을 Guide하고, EHR의 숨겨진 구조를 학습하도록 Transformer에 독창적인 구조적 수정을 제안한다.

- GCT는 합성 데이터셋과 공개된 EHR 데이터셋에서의 모든 예측 작업에서 모든 Baseline 모델들을 능가하여, EHR 데이터에 대한 효과적인 일반적인 표현 학습 알고리즘 으로서의 잠재력을 보여준다.

---





## Method

### *Electronic Health Records as a Graph*

![image-20240203142959322](https://github.com/mmistakes/minimal-mistakes/assets/126770258/7c3682a9-8040-40d4-a9a6-b23e479309c6)

Figure 1에 나타난 대로, $t$번째 방문 $V(t)$는 맨위의 방문 노드 $v(t)$으로 부터 내려오기 시작한다.

그 아래에는 진단(Diagnosis) Node $d_{1}^{(t)},...,d_{\vert d^{t}\vert}^{(t)}$가 있으며, 이는 일련의 치료(Treatment) Node $m_{1}^{(t)},...,m_{\vert m^{t}\vert}^{(t)}$를 만들게 되는데 이때 아래 첨자로 있는 $\vert d^{(t)}\vert, \vert r^{(t)}\vert$는 각각 $V(t)$에서 진단 및 치료의 코드 수를 나타낸다. 또한 일부 치료는 연속 값(혈압) 또는 binary 값 (양/음성 알레르기 반응)과 연관된 검사 결과 (Lab Result) Node $r_{1}^{(t)},...,r_{\vert r^{t}\vert}^{(t)}$이 생성되며, 이후에 설명에선 방문 횟수를 나타내는 $t$를 생략하고 진행하도록 하겠다.



이 논문의 큰 맥락은 다음과 같다.
  - 만약 모든 노드들이 특정 잠재 공간에서 표현이 가능하다면 $\vert d\vert + \vert m\vert + \vert r\vert$개의 노드로 이루어진 encounter로 볼 수 있다.
  - 후에 나오는 인접행렬 $A$는 EHR의 구조를 표현하는 행렬이다. 또한 $d_{i}, m_{i}, r_{i}$를 의미하는 통합 용어로 $c_{i}$를 사용한다. 
  - 만약 $c_{i}$와 $A$가 함께 있을 경우 Graph Network를 사용하여 방문 표현 $v$를 도출해낸다.
  - 구조정보 $A$가 없을 경우 Transformer의 feed-forward network를 사용하여 $v$를 도출해낸다.
  - 위의 과정들은 기본적으로 모든 노드 표현 $c_{i}$를 합산하고 어떤 잠재공간으로 투영하는 과정이다.

