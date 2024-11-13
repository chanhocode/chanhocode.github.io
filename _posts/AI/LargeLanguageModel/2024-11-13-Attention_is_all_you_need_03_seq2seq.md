---
title: "[LMM] Attention is All You Need를 읽어보자(3)_Seq2Seq"
writer: chanho
date: 2024-11-13 22:08:00 +0900
categories: [AI, LargeLanguageModel]
tags: [ai, llm]
---

> `Transformer` 와 `Seq2Seq`는 모두 순차적 데이터의 입력과 출력을 처리하는 데 사용되지만, 구조와 작동에서 몇 가지 중요한 차이점이 있다. 오늘은 Seq2Seq는 무엇이고 Transformer과의 차이는 무엇인지에 대해 간단하게 알아보고자 한다.

# Seq2Seq Model

> Seq2Seq 모델은 입력 시퀀스(문장)를 받아 출력 시퀀스(문장)로 변환하는 모델로, 주로 자연어 처리(NLP) 작업에 사용됩니다. 기계 번역, 텍스트 요약, 질문 생성 등 입력과 출력이 모두 시퀀스인 작업에 적합합니다. Seq2Seq 모델은 `Encoder` 와 `Decoder`라는 두가지 주요 부분으로 구성됩니다.

`"Hello" -> [Seq2Seq] -> "안녕" | 문장(Sequence)를 Input으로 Seq2Seq 모델에 입력시 Output으로 문장(Sequence)를 준다.`

## Seq2Seq의 구조

1. 인코더(Encoder): 입력 시퀀스를 받아 시퀀스의 의미를 요약한 고정 길이의 벡터로 변환합니다. 보통 RNN 기반이며, 인코더의 마지막 은닉상태가 디코더에 전달됩니다.
   <img width="709" alt="image" src="https://github.com/user-attachments/assets/ff254878-022c-429c-9de2-f697d570ccf1">
   _Image Source: [wekipedia_seq2seq](https://en.wikipedia.org/wiki/Seq2seq)_

   > Encoder는 입력 시퀀스를 처리하고 필수 정보를 캡처하여 네트워크의 숨겨진 상태와 Attention mechanism이 있는 모델에서는 컨텍스트 벡터로 저장합니다. 컨텍스트 벡터는 input hidden states 의 합이며 출력 시퀀스의 모든 시간 인스턴스에 대해 생성된다.

2. 디코더(Decoder): 인코더의 출력을 받아 하나씩 출력을 생성하며, 각 단계에서 이전 출력과 인코더로부터 전달받은 벡터를 바탕으로 다음 출력을 예측합니다.
   <img width="709" alt="image" src="https://github.com/user-attachments/assets/462c1499-bcf0-4c7d-bf60-31dc47c4e210">
   _Image Source: [wekipedia_seq2seq](https://en.wikipedia.org/wiki/Seq2seq)_
   > Decoder는 인코더에서 컨텍스트 벡터와 숨겨진 상태를 가져와 최종 출력 시퀀스를 생성합니다. 디코더는 자동 회귀 방식으로 작동하여 한 번에 출력 시퀀스의 요소를 하나씩 생성합니다. 각 단계에서는 이전에 생성된 요소, 컨텍스트 벡터, 입력 시퀀스 정보를 고려하여 출력 시퀀스의 다음 요소를 예측합니다.

## Seq2Seq의 특징

- 연속적인 정보처리: 입력 시퀀스를 순서대로 처리하여 문장의 순서와 문맥을 반영할 수 있습니다.
- Attention 추가 가능: Attention 메커니즘을 추가하여 긴 문장에서도 성능을 향상시키고, 중요한 부분에 가중치를 두어 의미를 잘 반영하게 할 수 있습니다.

## Transformer 와 Seq2Seq의 주요 차이점

| **특징**               | **Seq2Seq (RNN 기반)**                                    | **Transformer**                                    |
| ---------------------- | --------------------------------------------------------- | -------------------------------------------------- |
| **구조**               | 인코더-디코더 구조를 사용하며 RNN(LSTM 또는 GRU) 기반     | 인코더-디코더 구조이지만 Attention 메커니즘을 기반 |
| **데이터 처리 방식**   | 순차적으로 처리, 입력을 한 단계씩 처리                    | 병렬 처리 가능, 모든 입력을 동시에 처리            |
| **Attention 메커니즘** | 주로 Global Attention이나 Bahdanau/Luong Attention 사용   | Self-Attention (Scaled Dot-Product Attention) 사용 |
| **병렬화**             | 병렬화가 어려움 (시퀀스를 순서대로 처리해야 함)           | Self-Attention으로 모든 입력을 동시에 처리 가능    |
| **학습 효율성**        | 긴 시퀀스에서 기울기 소실 문제에 취약                     | 긴 시퀀스에서도 학습 효율성이 뛰어남               |
| **성능**               | 짧은 시퀀스에서는 효과적이나, 긴 시퀀스에서는 한계가 있음 | 긴 문장 및 복잡한 문맥에서 뛰어난 성능을 보임      |
| **계산 복잡도**        | 시퀀스 길이에 따라 선형적으로 증가                        | Self-Attention으로 인해 \(O(n^2)\)의 복잡도를 가짐 |
| **주요 활용**          | 기계 번역, 요약, 음성 인식 등                             | 자연어 처리(NLP) 전반, 기계 번역, 텍스트 생성 등   |

- Seq2Seq 모델은 RNN을 기반으로 하여 순차적으로 입력을 처리하므로 긴 시퀀스에 약점이 있으며, 병렬 처리가 어려운 구조입니다. Attention 메커니즘이 추가되었지만 여전히 RNN 기반의 한계를 가집니다.
- Transformer는 Self-Attention 메커니즘을 사용하여 모든 입력을 동시에 처리할 수 있어 병렬화가 가능하며, 긴 시퀀스에서도 성능이 뛰어납니다. 이 덕분에 학습 속도와 성능이 크게 향상되었습니다.
