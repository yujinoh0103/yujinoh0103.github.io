---
layout: post
title: "[스나이퍼팩토리] 한컴AI아카데미 18주차 - Transformer"
date: 2025-05-22
categories: [한컴ai]
author: "yujinoh0103"
---

# Transformer 구조 뜯어보기: Attention 그 이후

이전 글에서 Attention 메커니즘이 뭔지, 어떻게 작동하는지를 살펴봤다. 이제 본격적으로 이걸 어떻게 잘 감싸서 “Transformer”라는 구조가 만들어졌는지를 들여다볼 차례다. `MultiHeadAttention` 클래스 구현부터 `TransformerBlock`, 인코더 전체를 하나하나 쌓는 흐름을 따라가며 정리해봤다.

## 핵심 구성 요소 먼저 훑고 가자

Transformer는 크게 두 가지 파트로 나뉜다:

* **인코더 (Encoder)**
* **디코더 (Decoder)**

이번 노트북은 인코더 쪽만 다루고 있다. 인코더는 입력 시퀀스를 받아서 “문맥을 반영한 임베딩 벡터”로 바꿔주는 역할을 한다. 그리고 이 과정을 한 블록 안에서 반복적으로 수행한다.

## TransformerBlock: 이게 실질적 핵심이다

이 블록은 다음과 같이 구성되어 있다:

1. **Multi-head Self-Attention**
   → 앞서 살펴본 attention을 여러 번 (head 개수만큼) 수행해서 더 풍부한 관계를 포착한다.

2. **Residual Connection & Layer Normalization (1)**
   → attention 결과에 원래 입력을 더하고, 정규화를 거친다.
   → 안정적인 학습을 위한 장치.

3. **Feed Forward Network (FFN)**
   → 두 개의 Linear 레이어와 ReLU를 통과시켜 단순한 변환을 적용한다.
   → 각 단어별로 독립적으로 처리됨.

4. **Residual Connection & Layer Normalization (2)**
   → 마찬가지로 다시 더해주고 정규화함.

정리하자면, 하나의 Transformer Block은:

```
x → MultiHeadAttention → Add & Norm → FFN → Add & Norm → 출력
```

## 왜 이 구조가 강력한가?

이 구조는 RNN처럼 순차적으로 계산할 필요가 없기 때문에 병렬화가 가능하고, 동시에 모든 단어 간 관계를 attention을 통해 한 번에 살펴볼 수 있다. 특히 self-attention 덕분에 과거 정보뿐 아니라 미래 단어와의 관계도 쉽게 반영할 수 있다 (물론 디코더에선 마스킹을 걸어서 조정한다).

## 구현 중 흥미로웠던 부분들

노트북에서 직접 Multi-head Attention 클래스를 짠 부분이 꽤 인상 깊었다. `nn.Linear`를 통해 Q, K, V를 head 수만큼 나누고, attention을 head별로 계산한 뒤 `concat → projection` 과정을 통해 최종 출력을 얻는다. 실제 모델에서 이런 구조가 어떻게 구현되는지를 감 잡기 좋았다.

그리고 Positional Encoding에 대한 설명은 짧았지만, 중요하게 짚고 넘어가야 할 부분이다. Transformer는 순서를 고려하지 않기 때문에, 사인과 코사인 함수를 이용해 위치 정보를 인위적으로 더해줘야 한다.

## Encoder 전체를 쌓는 구조까지

마지막에는 여러 개의 Transformer Block을 반복해서 인코더 전체를 구성하고, 임베딩 레이어와 포지셔널 인코딩까지 포함한 `TransformerEncoder` 클래스를 정의했다. 이게 실제 BERT나 GPT 같은 모델에서 backbone 역할을 한다.

---

## 다음 글 예고: BERT와 GPT는 어떻게 다를까?

Transformer 구조만 봤을 때는 인코더와 디코더로만 나뉘는데, BERT와 GPT는 둘 다 Transformer 기반이라고 알려져 있다. 하지만 실제로는 꽤 다른 설계 철학을 가지고 있다.
다음 글에서는 이 둘이 어떻게 attention을 다르게 쓰는지, 어떤 방식으로 학습되는지 비교해보려고 한다.

<br/>
——————————————————————————<br/>
  본 후기는 [한글과컴퓨터x한국생산성본부x스나이퍼팩토리] 한컴 AI 아카데미 (B-log) 리뷰로 작성 되었습니다.

#한컴AI아카데미 #AI개발자 #AI개발자교육 #한글과컴퓨터 #한국생산성본부 #스나이퍼팩토리 #부트캠프 #AI전문가양성 #개발자교육 #개발자취업
