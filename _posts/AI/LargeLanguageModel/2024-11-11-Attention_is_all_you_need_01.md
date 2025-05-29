---
title: "[Paper] Attention is All You Need를 읽어보자(1)"
writer: chanho
date: 2024-11-11 23:00:00 +0900
categories: [AI, LargeLanguageModel]
tags: [ai, llm, paper]
---

> 회사에 들어와 약 반년이 안되는 시간동안 `Large Vision Language Model`을 연구 중 이다. 프로젝트로 인해 달리기만 하다보니 기초가 탄탄하지 못하다는 생각이 들어 처음 읽어 보는 건 아니지만, 근본 논문인 'Attention is All You Need' 부터 자세히 읽어보고자 한다.

`Attention is All You Need`는 2017년에 구글(Google)에서 발표한 논문이다. 해당 논문에서는 `Transformer` 아키텍처를 제안하고 있다.

`Seq2Seq의 구조인 Endcoder, Decoder를 따르면서도, RNN을 사용하지 않고 Attention 만으로 구현한 모델이다.`

## 0. Abstract
- The domainate Sequence transduction model은 인코더와 디코더를 포함하는 Complex recurrent 또는 컨볼루션 신경망을 기반으로 한다.
> 해당 논문에서는 `Attention mechanism`만을 기반으로 하는 새로운 간단한 네트워크 아키텍처인 Transfore을 제안한다.

## 1. Introduce
'Recurrent neural network(RNN: 순환 신경망)은 언어모델링 및 기계번역과 같은 시퀀스 모델링 및 변환 문제에서 확고히 자리 잡았지만, 필연적으로 `이전 결과를 입력으로 받는 순차적인 특성때문에 병렬처리를 배제하고 메모리 제약으로 인해 예제간 일괄처리가 제한되므로 시퀀스 길이가 길어지면 문제가 심각해진다.` factorization tricks과 conditional computation을 통해 연산 효율의 대폭적인 향상을 달성했지만 순차적 계산의 근본적인 제약은 여전히 남아있다.

`Attention mechanism`은 다양한 작업에서 compelling sequence modeling과 transduction models의 필수적인 부분이 되어 입력또는 출력시퀀스의 거리에 관계없이 종속성을 모델링 할 수 있게 해준다. 해당 연구에서는 재귀를 피하고 대신 `Attention mechanism만으로` 입력과 출력간의 Global dependencies를 도출하는 모델 아키텍처인 `Transformer`을 제안한다.

## 2. Background
> 순차연산을 줄이려는 목표는 모든 입력 및 출력위치에 대해 'hidden representations'을 병렬로 계산하는 컨볼루션 신경망(CNN)의 기반이 되기도 한다. 이러한 모델에서는 두 임의의 입력 또는 출력 위치에서 신호를 연관시키는데 필요한 연산 횟수가 위치 사이의 거리에 따라 'ConvS2S'의 경우 선형적으로 'ByteNet'의 경우 대수적 중가한다. 즉, `멀리 떨어진 위치의 종속성을 학습하기가 어렵다.`

`Transformer`에서는 'Attention-weighted'가 적용된 위치의 평균화로 인해 유효해상도가 감소하는 대신 연산횟수가 일정하게 줄어들지만, `Multi-Head Attention`을 통해 상쇄할 수 있다.

- Self-Attention은 시퀀스의 표현을 계산하기 위해 단일 시퀀스의 서로 다른 위치를 연관시키는데 Attention mechanism 이다. (독해, 추상적 요약, 문장 표현 등 다양한 과제에서 성공적으로 사용됨)
- End-to-End momory network는 시퀀스 정렬된 반복대신 반복주의 메커니즘을 기반으로 하며 간단한 언어 질문답변과 언어 모델링 작업에서 우수한 성능을 보인다.

`Transformer은 RNN이나 CNN은 사용하지 않고 입출력을 계산하기 위해 전적으로 Self-Attention에 의존하는 최초의 트랜지션 모델이다.`

------
> 해당 1, 2 Page의 부연을 위해 다음 게시글에는 특정 부분에 대한 간단한 설명을 작성하고자 한다.
- Seq2Seq 모델 및 한계
- Encoder & Decoder
- RNN & CNN
- Attention Mechanism
