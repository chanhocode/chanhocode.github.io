---
title: "[Paper] ConceptAttention: 이미지 생성 모델의 내부"
writer: chanho
date: 2025-05-30 13:30:00 +0900
categories: [AI, LargeLanguageModel]
tags: [ai, llm, paper]
---

[ConceptAttention: Diffusion Transformers Learn Highly...](https://arxiv.org/abs/2502.04320)

[Alec Helbling](https://arxiv.org/search/cs?searchtype=author&query=Helbling,+A), [Tuna Han Salih Meral](https://arxiv.org/search/cs?searchtype=author&query=Meral,+T+H+S), [Ben Hoover](https://arxiv.org/search/cs?searchtype=author&query=Hoover,+B), [Pinar Yanardag](https://arxiv.org/search/cs?searchtype=author&query=Yanardag,+P), [Duen Horng Chau](https://arxiv.org/search/cs?searchtype=author&query=Chau,+D+H)

> Do the rich representations of multi-modal diffusion transformers (DiTs) exhibit unique properties that enhance their interpretability? We introduce ConceptAttention, a novel method that leverages the expressive power of DiT attention layers to generate high-quality saliency maps that precisely locate textual concepts within images. Without requiring additional training, ConceptAttention repurposes the parameters of DiT attention layers to produce highly contextualized concept embeddings, contributing the major discovery that performing linear projections in the output space of DiT attention layers yields significantly sharper saliency maps compared to commonly used cross-attention mechanisms. Remarkably, ConceptAttention even achieves state-of-the-art performance on zero-shot image segmentation benchmarks, outperforming 11 other zero-shot interpretability methods on the ImageNet-Segmentation dataset and on a single-class subset of PascalVOC. Our work contributes the first evidence that the representations of multi-modal DiT models like Flux are highly transferable to vision tasks like segmentation, even outperforming multi-modal foundation models like CLIP.
> 

---

## Diffusion Transformer의 속살을 보다: ConceptAttention으로 이해하는 이미지 생성 모델의 내부

## 논문의 핵심 요약 및 목표

최근 텍스트-이미지 생성 분야에서 Flux, SD3와 같은 Diffusion Transformer (DiT) 모델들이 뛰어난 성능을 보여주며 주목받고 있습니다. 하지만 이 모델들은 마치 "블랙 박스"처럼 내부 작동 방식을 이해하기 어렵다는 한계가 있었습니다. 오늘 소개할 논문, "ConceptAttention: Diffusion Transformers Learn Highly Interpretable Features"는 바로 이러한 DiT 모델의 내부 표현을 해석하고 이해하기 위한 혁신적인 방법론인 CONCEPTATTENTION을 제안합니다.

CONCEPTATTENTION은 DiT 모델의 주의(Attention) 레이어를 활용하여, 이미지 내 특정 텍스트 개념이 어디에 위치하는지 정확하게 보여주는 고품질의 **saliency 맵(현저성 맵)**을 생성합니다. 놀랍게도 이는 추가적인 학습 없이 기존 DiT 아키텍처를 재활용하여 가능하며, 심지어 제로샷 이미지 분할 벤치마크에서 최신 기술(state-of-the-art) 성능을 달성했습니다. 이 논문의 목표는 DiT 모델 내부에서 개념이 어떻게 표현되고 처리되는지 시각적으로 이해하고, 그 표현의 우수성을 입증하는 것입니다.

### Background

- **DiT (Diffusion Transformer):** 노이즈로부터 시작하여 텍스트 설명에 따라 이미지를 생성하는 확산 모델의 한 종류로, Transformer 아키텍처를 핵심으로 사용합니다. Flux, SD3 등이 대표적입니다.
- **ViT (Vision Transformer):** 이미지 인식을 위해 설계된 Transformer 아키텍처로, CLIP, DINO와 같은 모델의 기반이 됩니다. 이 논문에서는 ConceptAttention의 성능 비교 대상으로 등장합니다.
- **교차 어텐션 (Cross-attention):** 두 개의 다른 시퀀스(예: 텍스트와 이미지) 간의 관계를 파악하는 어텐션 메커니즘입니다. 기존에는 이 교차 어텐션 맵이 개념 위치 파악에 사용되곤 했습니다.
- **선형 투영 (Linear Projection):** 두 벡터 간의 유사성을 측정하기 위한 내적(dot product) 연산을 의미합니다. ConceptAttention은 어텐션 출력 공간에서의 선형 투영을 핵심 아이디어로 사용합니다.
- **Saliency Map (현저성 맵):** 이미지 내에서 특정 개념이나 객체가 얼마나 두드러지는지, 즉 어디에 위치하는지를 시각적으로 나타내는 맵입니다.
- **MMATTN (Multi-Modal Attention Layer):** DiT에서 이미지 패치와 텍스트 프롬프트 토큰을 함께 처리하는 멀티모달 어텐션 레이어입니다.

### 핵심 문제

DiT 모델은 이미지 생성 능력은 뛰어나지만, 그 내부 의사 결정 과정은 인간에게 숨겨져 있어 "블랙 박스"로 여겨졌습니다. 모델이 어떻게 텍스트 프롬프트를 이해하고 이미지로 변환하는지, 특정 개념을 이미지의 어느 부분에 위치시키는지 명확히 알기 어려웠습니다. 기존의 해석 방법론들은 주로 UNet 기반 확산 모델에 초점을 맞추거나, DiT 아키텍처의 고유한 특성을 충분히 활용하지 못하는 한계가 있었습니다.

### 연구 목적 (ConceptAttention을 통한 해결)

본 논문은 다음과 같은 목적을 가지고 ConceptAttention 방법론을 제안합니다.

1. DiT 모델의 내부 표현을 해석하고 이해하기 위한 새로운 방법론인 CONCEPTATTENTION을 제안합니다.
2. 추가적인 학습 없이 DiT의 멀티모달 주의 레이어(MMATTN) 파라미터를 재활용하여, 이미지 내 특정 텍스트 개념의 위치를 정확히 보여주는 고품질 saliency 맵을 생성합니다.
3. DiT 모델의 표현이 이미지 분할과 같은 하위 비전 작업으로 높은 전이성을 가짐을 처음으로 입증하고, 제로샷 이미지 분할 벤치마크에서 SOTA 성능을 달성합니다.

## 논문 주요 내용 상세 요약

### ConceptAttention 방법론

<img width="100%" alt="image" src="https://github.com/user-attachments/assets/345d81c4-fd22-4f61-b81f-cf0e37fed848">
 _Image Source: [arxiv](https://arxiv.org/pdf/2502.04320)_

- **개념 시각화 (Saliency Map 생성):** ConceptAttention은 사용자가 지정한 텍스트 개념(예: "고양이", "하늘")이 이미지 내 어디에 나타나는지를 보여주는 정교한 saliency 맵을 생성합니다. 이는 모델이 특정 개념을 "보는" 방식을 시각화하는 것입니다.
- **기존 아키텍처 재활용 (No Extra Training):** 가장 큰 특징은 **추가적인 학습이 전혀 필요 없다**는 점입니다. 대신, Flux 및 SD3와 같은 멀티모달 DiT가 사용하는 기존 멀티모달 주의 레이어(MMATTN)의 파라미터를 그대로 재활용합니다.
- **개념 임베딩 생성 및 업데이트:** 사용자가 지정한 개념에 대한 개념 임베딩을 생성하고, 이 임베딩을 이미지 패치 표현과 함께 MMATTN 레이어를 통과시키며 업데이트합니다.
- **단방향 주의 (One-directional Attention):** 중요한 구현 특징은 개념 토큰이 이미지 토큰이나 다른 개념 토큰으로부터 정보를 얻을 수 있지만, 이미지 토큰이나 텍스트 프롬프트 토큰은 개념 토큰을 무시하도록 설계되었다는 것입니다. 이는 개념 토큰의 처리가 이미지 생성 결과 자체에 영향을 주지 않도록 하기 위함입니다.
- **주의 출력 공간 활용 (Key Discovery):** 논문에서 발견한 핵심 내용 중 하나는, 기존의 교차 주의(cross-attention) 맵보다 **주의 연산의 출력 공간(attention output space)**에서 이미지 출력 벡터(`ox`)와 개념 출력 벡터(`oc`) 간의 선형 투영(내적)을 통해 생성된 saliency 맵(`softmax(ox * o^T_c)`)이 훨씬 더 선명하고 품질이 높다는 것입니다. 이는 멀티모달 DiT 아키텍처에서만 가능한 독특한 접근 방식이라고 논문은 설명합니다.
- **여러 레이어 정보 통합:** 여러 MMATTN 레이어(특히 후반부 레이어)에서 얻은 saliency 맵 정보를 평균화하여 최종 결과를 구성합니다. 실험에서는 마지막 10개 레이어의 정보를 활용했습니다.

### **작동 방식**

- 추가적인 훈련 없이 DiT 어텐션 레이어의 기존 매개변수를 재사용합니다.
- 개념 임베딩(concept embeddings) 세트를 생성하고 업데이트합니다. 이 개념 임베딩은 이미지 외관에 영향을 주지 않으면서 텍스트 및 이미지 임베딩과 함께 순차적으로 업데이트됩니다.
- 이미지 패치 표현(image patch representations)을 이러한 개념 임베딩에 선형 투영하여 시각화 맵을 생성합니다.
- 특히, CONCEPTATTENTION은 이미지 패치에서 개념으로의 교차 어텐션(cross attention)과 개념들 간의 자체 어텐션(self attention)을 모두 활용합니다. 논문의 결과에 따르면 이 두 가지 어텐션 연산을 모두 사용하는 것이 성능을 향상시키는 것으로 나타났습니다.
- 이 방법은 여러 개념에 대한 시각화 맵을 동시에 생성할 수 있으며, 프롬프트 단어에 국한되지 않고 사용자가 지정한 임의의 텍스트 개념(예: “dragon”, “sky”, “cat”, “sun” 등)에 대해 작동합니다.

### 주요 성과

- 제로샷 이미지 분할 SOTA: ConceptAttention은 ImageNet-Segmentation 및 PascalVOC와 같은 제로샷 이미지 분할 벤치마크에서 최신 기술(state-of-the-art) 성능을 달성했습니다.
- 다른 해석 방법론 대비 우수성: CLIP, DINO, UNet 기반 확산 모델 등 11가지 다른 제로샷 해석 방법론들보다 뛰어난 성능을 보였습니다.
- DiT 표현의 높은 전이성 입증: DiT 모델의 표현이 이미지 분할과 같은 하위 비전 작업으로 높은 전이성을 가짐을 처음으로 입증했으며, 이는 멀티모달 파운데이션 모델인 CLIP보다도 우수할 수 있음을 시사합니다.
- 임의 개념 시각화: 프롬프트에 명시적으로 포함되지 않은 임의의 개념에 대해서도 saliency 맵 생성이 가능합니다.

### **핵심 발견**

CONCEPTATTENTION을 통해 DiT 어텐션 레이어의 출력 공간에서 선형 투영을 수행하는 것이 일반적으로 사용되는 교차 어텐션 메커니즘보다 훨씬 선명한 시각화 맵을 생성한다는 것을 발견했습니다. 이 출력 공간 맵은 멀티모달 DiT 모델에 고유하며, UNet 기반 모델로는 기본적으로 생성될 수 없습니다.

### **아키텍처**

이 방법은 Flux및 SD3와 같은 멀티모달 확산 트랜스포머(DiT)에 적용됩니다. 이 모델들은 텍스트 설명에 따라 노이즈를 사실적인 이미지로 변환하는 확산 모델의 최신 기술입니다. DiT는 이미지 패치 표현과 프롬프트 토큰 임베딩을 모두 처리하는 멀티모달 어텐션 레이어(MMATTN)를 사용합니다.

### 가용성

ConceptAttention의 코드는 공개되어 있으며, 논문에 GitHub 링크가 제공되어 직접 확인하고 활용해볼 수 있습니다.

https://github.com/helblazer811/ConceptAttention

## Q&A 형태의 이해 및 마무리

**Q1: ConceptAttention은 단순 시각화 도구인가, 아니면 의미 분류 모델인가?**

→ ConceptAttention은 단순 시각화 도구 이상이다. Diffusion Transformer(DiT) 모델의 내부 표현을 활용하여 텍스트로 지정한 임의의 의미적 개념이 이미지에서 정확히 어디에 위치하는지 고품질로 시각화(saliency maps)하고, 이를 제로샷 이미지 분할 태스크에서 정량적으로 검증한 방법론이다. 즉, 이미지에서 개념의 존재 여부와 위치를 해석하고 활용할 수 있는 고성능, 고해상도의 해석 가능한 시각화 방법이다.

**Q2: 멀티모달 확산 트랜스포머(DiTs, Diffusion Transformers)는 무엇이며 어떤 작업에 쓰이나?**

→ 멀티모달 DiT 모델(예: Flux, SD3)은 최근 이미지 생성(text-to-image synthesis)을 위해 등장한 고성능 모델로, 텍스트 프롬프트를 입력받아 현실적이고 정교한 이미지를 생성한다. 주요 특징은 노이즈에서부터 실제 이미지로의 생성 과정을 어텐션 메커니즘(Multi-modal Attention)을 통해 학습하며, 내부적으로 이미지 패치 표현과 텍스트 임베딩을 함께 다룬다는 점이다. 비록 이미지 생성 모델로 탄생했으나, ConceptAttention을 적용하면 이미지 내 의미적 개념의 위치 파악과 같은 비전 인코더 형태로도 유용하게 쓰일 수 있다는 점을 이 논문이 제시했다.

**Q3: Flux/SD3와 같은 DiT 모델은 생성 모델 아닌가? 나는 이미지의 의미를 찾는 비전 인코더로 활용하고자 하는데, 가능한가?**

→ 원래 Flux와 SD3는 생성 모델로 설계됐지만, ConceptAttention을 활용하면 내부 어텐션 표현을 기반으로 이미지 내 특정 개념의 위치를 정확하게 파악하고 시각화할 수 있다. 논문에 따르면, 이렇게 얻은 표현은 CLIP이나 DINO와 같은 기존 ViT 기반 모델보다 더 뛰어난 정확도로 개념의 '존재 및 위치 정보'를 제공할 수 있어, 특히 제로샷 이미지 분할 벤치마크에서도 더 높은 성능을 보였다. 즉, DiT 모델을 ConceptAttention과 결합하면 이미지의 의미파악과 위치특정을 위한 강력하면서도 해석 가능한 비전 인코더 형태로 활용 가능하다.

**Q6: Flux/SD3 베이스의 ConceptAttention 접근법을 멀티모달 대형언어모델(LLM)의 비전인코더로서 활용할 수 있는가?**

→ 충분히 가능하다. 특히 멀티모달 LLM 구현 시, 이미지 내 특정 개념의 위치를 정밀히 파악해야 하는 과제(예를 들어 "이미지 왼쪽 아래 비행기가 있는가?", "도로 위 자동차 위치는 어디인가?") 등에 ConceptAttention이 유리하다. 이는 CLIP이나 DINO와 같은 전역적인 특징 추출 모델과 결합하면 더욱 시너지 있는 조합을 이루어 풍부한 시각 이해 표현을 LLM에 제공할 수 있다.

---

> "ConceptAttention: Diffusion Transformers Learn Highly Interpretable Features" 논문은 DiT 모델의 '블랙 박스'를 열어 그 내부를 들여다볼 수 있는 강력하고 새로운 도구를 제공합니다. 추가 학습 없이 기존 아키텍처를 활용하여 개념의 위치를 정확히 시각화하고, 제로샷 이미지 분할에서 SOTA 성능을 달성한 것은 매우 인상적입니다. ConceptAttention은 DiT 모델이 단순히 이미지를 생성하는 것을 넘어, 내부적으로 매우 풍부하고 해석 가능한 특징을 학습하고 있음을 보여줍니다. 이는 향후 DiT 모델의 이해를 깊게 하고, 더 발전된 이미지 생성 및 편집 기술, 그리고 다양한 비전 작업으로의 응용 가능성을 열어줄 것으로 기대됩니다.
>
