---
title: "[Paper] NASA/Galieo 논문 리뷰"
writer: chanho
date: 2025-08-12 10:40:00 +0900
categories: [AI, LargeLanguageModel]
tags: [ai, paper]
---

[Galileo: Learning Global & Local Features of Many Remote...](https://arxiv.org/abs/2502.09356)

[Gabriel Tseng](https://arxiv.org/search/cs?searchtype=author&query=Tseng,+G), [Anthony Fuller](https://arxiv.org/search/cs?searchtype=author&query=Fuller,+A), [Marlena Reil](https://arxiv.org/search/cs?searchtype=author&query=Reil,+M), [Henry Herzog](https://arxiv.org/search/cs?searchtype=author&query=Herzog,+H), [Patrick Beukema](https://arxiv.org/search/cs?searchtype=author&query=Beukema,+P), [Favyen Bastani](https://arxiv.org/search/cs?searchtype=author&query=Bastani,+F), [James R. Green](https://arxiv.org/search/cs?searchtype=author&query=Green,+J+R), [Evan Shelhamer](https://arxiv.org/search/cs?searchtype=author&query=Shelhamer,+E), [Hannah Kerner](https://arxiv.org/search/cs?searchtype=author&query=Kerner,+H), [David Rolnick](https://arxiv.org/search/cs?searchtype=author&query=Rolnick,+D)

We introduce a highly multimodal transformer to represent many remote sensing modalities - multispectral optical, synthetic aperture radar, elevation, weather, pseudo-labels, and more - across space and time. These inputs are useful for diverse remote sensing tasks, such as crop mapping and flood detection. However, learning shared representations of remote sensing data is challenging, given the diversity of relevant data modalities, and because objects of interest vary massively in scale, from small boats (1-2 pixels and fast) to glaciers (thousands of pixels and slow). We present a novel self-supervised learning algorithm that extracts multi-scale features across a flexible set of input modalities through masked modeling. Our dual global and local contrastive losses differ in their targets (deep representations vs. shallow input projections) and masking strategies (structured vs. not). Our Galileo is a single generalist model that outperforms SoTA specialist models for satellite images and pixel time series across eleven benchmarks and multiple tasks.

---

> 이 논문은 원격 감지(RS) 데이터의 다양성과 스케일 문제를 해결하기 위해 개발된 **Galileo**라는 고도로 다중 모달리티(Multimodal) 트랜스포머 모델을 소개합니다. RS 데이터는 위성 이미지(예: Sentinel-2 광학 데이터), 레이더(SAR), 고도, 날씨, 인구 밀도, 토지 피복 지도 등 다양한 모달리티로 구성되며, 이는 작물 매핑, 홍수 탐지, 해양 쓰레기 모니터링 등 사회적으로 중요한 태스크에 유용합니다. 그러나 기존 SSL 방법은 특정 모달리티나 입력 형태(이미지 vs. 픽셀 시계열)에 특화되어 있어 유연성이 부족합니다. 또한 RS 객체의 스케일이 다양합니다: 작은 보트(1-2 픽셀, 빠른 변화)부터 빙하(수천 픽셀, 느린 변화)까지 다중 스케일 특징(multi-scale features)을 학습합니다.
> 

Galileo는 마스크 데이터 모델링을 기반으로 한 새로운 SSL 알고리즘을 통해 이러한 도전을 해결합니다. 이 모델은 글로벌과 로컬 피처를 동시에 학습하며, 단일 모델로 이미지와 픽셀 시계열을 처리할 수 있습니다. 데이터셋은 지리적·의미적 다양성을 고려해 글로벌하게 샘플링 되어 있으며, 11개 벤치마크에서 SOTA(State-of-the-Art) 전문 모델을 능가합니다. 저자들은 모델의 유연성과 전이 학습 능력을 강조하며, 레이블이 부족한 실무 환경에서 유용함을 주장합니다.

<img width="100%" alt="image" src="https://github.com/user-attachments/assets/b970c772-4e14-4212-9991-d27d37dd14b6">
_Image Source: [galileo](https://arxiv.org/pdf/2502.09356)_

- 핵심 방법론
    - Galileo는 마스킹된 모델링을 통해 자체 지도 학습(self-supervised learning, SSL) 알고리즘을 사용합니다.
    - 이중 전역 및 지역 손실이 핵심입니다.
        - 전역 특징(Global Features) 학습: Deep representations을 목표로 하며, structured space-time masking을 사용합니다. 이는 주로 분류 작업에 적합하며, 입력 간의 특징 다양성을 증폭시킵니다.
        - 지역 특징(Local Features) 학습: Shallow input projections을 목표로 하며, Unstructured random masking을 사용합니다. 이는 특히 해양 잔해물과 같은 작은 객체를 포함하는 세분화 작업에 효과적이며, 입력 내 특징 다양성을 증폭시킵니다.
- 성능 및 유연성
    - Galileo는 단일 일반 모델로서 11가지 벤치마크와 여러 태스크에서 최고 수준의 전문 모델들을 능가하는 성능을 보입니다.
    - 이미지 및 픽셀 시계열 데이터를 모두 유연하게 처리할 수 있습니다.
    - 농작물 매핑 및 홍수 감지와 같은 다양한 원격 감지 작업에 적용 가능합니다.
- 훈련 데이터: Galileo는 지리적, 의미적 다양성을 위해 전 세계에서 수집된 방대한 멀티모달 원격 감지 데이터셋으로 사전 학습되었습니다. 이 데이터셋에는 Sentinel-1 & -2 위성 영상, 고도, 날씨, 인구 추정치, 야간 조명, 토지 피복 지도 등 9가지 원격 감지 데이터 양상이 포함됩니다.
- **공개 소스**: https://github.com/nasaharvest/galileo?tab=readme-ov-file
    - **Galileo ViT-Nano**: **0.8M**
    - **Galileo ViT-Tiny**: **5.3M**
    - **Galileo ViT-Base**: **85.0M**

## 핵심 기술 (Method: Global, Local, Multimodal Self-Supervision)

Galileo의 핵심은 듀얼 글로벌-로컬 SSL 알고리즘과 유연한 토큰화 아키텍처입니다. 이는 ViT(Vision Transformer)를 기반으로 하며, 마스크 모델링을 통해 레이블 없는 데이터를 활용합니다. 입력 x를 가시적 뷰(xv)와 마스크된 타겟(xt)으로 나누고, xv로부터 xt를 예측합니다. 예측은 잠재 공간(Latent Space)에서 이루어지며, 온라인 인코더(Gradient Descent로 업데이트)와 타겟 인코더(EMA로 업데이트)를 사용합니다.

<img width="100%" alt="image" src="https://github.com/user-attachments/assets/7ecce872-3f9d-40e2-898e-22e9655f1d9b">
_Image Source: [galileo](https://arxiv.org/pdf/2502.09356)_

### Input Tokenization

RS 데이터의 다양성을 처리하기 위해 입력을 공간(Height H, Width W), 시간(Timesteps T), 채널 그룹(Channel Groups G)으로 분할합니다. 즉, RS입력을 Transformer 모델이 처리할 수 있는 일련의 토큰으로 변환하는 과정을 의미합니다.

- 변환 과정
    - 입력 분할: Galileo의 인코더는 원격 감지 입력을 공간적 사각형(spatial squares), 타임스텝(timesteps), 그리고 채널 그룹(channel groups)으로 분할합니다.
        - 채널 그룹은 Sentinel-2 데이터의 10m 채널과 같이 특정 원격 감지 제품의 채널들 중 관련 있는 하위 집합을 의미합니다.
    - 선형 투영: 분할된 입력들은 인코더 차원 D(encoder dimension D)로 선형 투영(linear projection)됩니다. 데이터 유형에 따라 다른 변환이 적용됩니다:
        - 시공간 데이터(Space-time data): 높이(H), 너비(W), 타임스텝(T), 채널(C)을 패치 크기(P), 타임스텝(T), 채널 그룹(G), 인코더 차원(D)에 따라 변환합니다.
        - 공간 데이터(Space data): 높이(H), 너비(W), 채널(C)을 패치 크기(P), 채널 그룹(G), 인코더 차원(D)에 따라 변환합니다.
        - 시간 데이터(Time data): 타임스텝(T), 채널(C)을 타임스텝(T), 채널 그룹(G), 인코더 차원(D)에 따라 변환합니다.
        - 정적 데이터(Static data): 채널(C)을 채널 그룹(G), 인코더 차원(D)에 따라 변환합니다.
- 토큰 임베딩 추가
    - 선형 투영 후, 인코더는 토큰에 다음과 같은 임베딩을 추가합니다:
        - 공간 및 시간 사인파 위치 임베딩(spatial and temporal sinusoidal position embeddings)
        - 학습 가능한 채널 임베딩(learnable channel embeddings)
        - 월별 임베딩(month embeddings): 계절적 추론을 가능하게 합니다.
    - 이러한 임베딩들은 계산된 선형 투영에 추가되며, 모든 채널 그룹은 시퀀스 차원을 따라 연결되어 최종 입력 시퀀스를 형성합니다.
- 유연성
    - Galileo의 커스텀 토큰화 방식과 크기 조절 가능한 선형 투영 가중치(resizable linear projection weights) 덕분에, 트랜스포머 아키텍처는 입력 시퀀스 길이, 다양한 채널 그룹, 그리고 다양한 데이터 소스 간의 차이에 대해 높은 유연성을 가집니다. 이는 모델이 광범위한 원격 감지 데이터를 처리할 수 있도록 돕습니다.

### Masking & Dual Losses

**1. 마스킹 (Masking)**

마스킹은 Galileo가 표현을 학습하는 데 사용하는 마스킹된 데이터 모델링(masked data modeling) 프레임워크의 기본 요소입니다. 이 과정은 입력 x를 가려진("targets", xt) 부분과 보이는("visible", xv) 부분으로 나눈 다음, 보이는 부분을 사용하여 가려진 부분을 예측하도록 모델을 사전 학습하는 방식입니다. Galileo는 픽셀이 아닌 잠재 토큰(latent tokens)에 대해 마스킹 및 예측을 수행합니다.

(잠재 토큰: 토큰: RS 입력데이터가 모델 내부에서 처리되는 표현 또는 인코딩을 의미)

Galileo는 두 가지 주요 마스킹 전략을 사용합니다

**1) 구조화된 마스킹 (Structured Masking) - 전역 특징 학습용**

- 목적: 더 크고 전역적인 예측을 요구하여 전역(global) 특징을 학습하도록 설계되었습니다.
- 방식: "공간(space) 마스킹"은 시간 일관성을 유지하면서 공간에 걸쳐 토큰을 무작위로 샘플링하고, "시간(time) 마스킹"은 공간 일관성을 유지하면서 시간에 걸쳐 토큰을 무작위로 샘플링합니다.
- 효과: 보이는 토큰과 목표 토큰 사이의 거리를 증가시켜, 모델이 더 넓은 범위의 정보를 사용하여 예측하도록 강제합니다. 이는 분류 작업에 적합한 추상적이고 낮은 주파수 특징을 학습하는 데 도움이 됩니다.

**2) 비구조화된 마스킹 (Unstructured Masking) - 지역 특징 학습용**

- 목적: 더 작고 지역적인 예측을 요구하여 지역(local) 특징을 학습하도록 설계되었습니다.
- 방식: 공간, 시간 또는 채널 그룹 위치에 관계없이 동일한 확률로 토큰을 무작위로 샘플링합니다.
- 효과: 보이는 토큰과 목표 토큰 사이의 평균 거리를 최소화하여, 모델이 국소적인 정보를 사용하여 예측하도록 유도합니다. 이는 작은 객체 세분화와 같이 미세하고 높은 주파수 특징을 학습하는 데 유용합니다.

**2. 이중 손실 (Dual Losses)**

Galileo는 전역(global) 작업과 지역(local) 작업이라는 두 가지 자기 지도 학습 목표를 통해 표현을 학습합니다. 이 두 목표는 목표(targets)와 마스킹 전략(masking strategies)에서 차이를 보입니다. Galileo의 전체 손실은 이 두 손실의 평균으로 계산됩니다.

**1) 전역 손실 (LGlobal) - 심층 목표(Deep Targets) 및 구조화된 마스킹**

- 목표 표현: 모델의 깊은 잠재 공간(deep latent space)에서의 표현을 목표로 합니다. 이는 "고정된(frozen)" 인코더의 더 깊은 계층에서 나온 토큰 표현을 사용하여 구성됩니다. 직관적으로, 깊은 표현은 어텐션 메커니즘을 통해 비국소적인 처리가 더 많이 이루어지기 때문에 더 전역적입니다.
- 손실 함수: PatchDiscB 손실을 사용합니다. 이는 입력 내의 토큰 간 InfoNCE 손실을 적용하지만, 배치 내의 다른 샘플로부터의 부정적인 예시(negative examples)를 포함하여 글로벌 식별력을 향상시킵니다. 이 손실은 분류 작업에 적합한 전역 특징을 학습하도록 설계되었습니다.
- 마스킹: 위에서 설명한 구조화된 공간-시간 마스킹을 사용합니다.

**2) 지역 손실 (LLocal) - 얕은 목표(Shallow Targets) 및 비구조화된 마스킹:**

- 목표 표현: 모델의 가장 낮은 표현 수준, 즉 입력 공간(input space)의 얕은 선형 투영(shallow linear projections)을 목표로 합니다. 이는 깊은 처리(트랜스포머 블록)를 건너뛰고 타겟 인코더의 선형 투영 만을 사용하여 목표를 계산합니다. 직관적으로, 얕은 표현은 입력 처리량이 적기 때문에 더 지역적입니다.
- 손실 함수: PatchDisc 손실을 사용합니다. 이는 낮은 수준의 특징을 기반으로 패치 간의 구별을 하도록 모델에 지시합니다. 이 손실은 이미지 내에서 픽셀을 다른 픽셀과 구분하여 미세한 지역 특징을 증폭시키는 데 도움이 됩니다.
- 마스킹: 위에서 설명한 비구조화된 무작위 마스킹을 사용합니다.

**3) 이중 손실의 중요성 및 효과**

- 다중 스케일 특징 학습: Galileo는 이 두 가지 목표를 동시에 학습함으로써 다중 스케일(multi-scale) 특징을 학습합니다. 이는 작은 어선부터 수천 픽셀 크기의 빙하에 이르는 다양한 규모의 객체를 포함하는 원격 감지 데이터의 고유한 도전을 해결하는 데 필수적입니다.
- 유연성 및 일관성: 이 이중 목적 알고리즘은 단일 목적 알고리즘보다 분류 및 분할 작업 모두에서 더 뛰어난 성능을 보이며 더 일관된 학습을 달성합니다.
- 특징 다양성: 지역 작업은 입력 내(within-input) 특징 다양성을 증폭하는 반면, 전역 작업은 입력 간(between-input) 특징 다양성을 증폭합니다.

### Pretraining Dataset

- 127,155 인스턴스, 글로벌 샘플링(H3 그리드 + k-means 클러스터링으로 지리적·의미적 다양성 확보)
- 9 모달리티: Sentinel-1(SAR), Sentinel-2(광학, NDVI), SRTM(고도), ERA5(날씨), TerraClimate(기후), VIIRS(야간 조명), Dynamic World/World Cereal(토지 피복), LandScan(인구)
- 각 인스턴스: 96×96 픽셀(10m 해상도), 24개월 타임스텝. 공간-시간 변동, 공간만, 시간만, 정적 데이터로 그룹화

## Input & Output

### Model Input

- **다중 양식 (Multimodal)**
    - 다중 스펙트럼 광학 데이터 (Multispectral Optical, MS): Sentinel-2 위성 데이터의 모든 밴드(B1, B9, B10 제외) 및 NDVI 포함
    - 합성 개구 레이더 (Synthetic Aperture Radar, SAR): Sentinel-1 위성 데이터의 VV 및 VH 편파 포함
    - 고도 (Elevation): Shuttle Radar Topography Mission (SRTM)에서 캡처된 고도 및 경사 데이터
    - 날씨 (Weather): ERA5 데이터셋의 강수량 및 온도, TerraClimate의 기후 수분 부족, 토양 수분, 실제 증발산량
    - 유사 라벨 (Pseudo-labels): Dynamic World 토지 피복 지도 확률 (시간 평균) 및 World Cereal 농업 토지 피복 지도
    - 야간 조명 (Nightlights): VIIRS 야간 조명
    - 인구 추정치 (Population Estimates): LandScan 데이터셋
    - 공간 위치 (Spatial Location): 인스턴스의 중심 위도 및 경도
- **다중 시간 스텝 및 형태 (Multi-Timestep & Shapes)**
    - Galileo는 단일 시간 스텝 이미지(single-timestep imagery), 다중 시간 스텝 이미지(multi-timestep imagery), 그리고 픽셀 시계열(pixel time series)과 같은 다양한 입력 형태를 유연하게 처리할 수 있습니다.
    - 단일 인스턴스는 일반적으로 10m/픽셀 해상도로 96x96 픽셀 크기의 24개 월별 시간 스텝으로 구성됩니다.

### Model Output

Galileo 모델은 사전 학습을 통해 다중 스케일의 유용한 표현을 학습하며, 이 표현들은 다양한 다운스트림 작업에 활용됩니다.

- **학습된 표현**
    - Galileo는 마스킹된 데이터 모델링 프레임워크를 통해 학습된 잠재 토큰(latent tokens)의 표현을 출력합니다. 이러한 표현들은 사전 학습 후 파인튜닝 또는 특징 재사용(reuse as features)을 통해 실제 작업으로 전이될 수 있습니다.
- 이중 손실 목표
    - Galileo의 핵심은 "전역" 및 "지역"이라는 두 가지 자기 지도 학습 목표를 동시에 학습하는 것입니다. 전체 손실은 이 두 손실의 평균으로 계산됩니다.
    - 전역 특징 학습
        - 목표 표현: 모델의 깊은 잠재 공간에서의 표현을 목표로 합니다. 이는 "고정된" 인코더의 더 깊은 계층에서 나온 토큰 표현을 사용하여 구성됩니다.
        - 마스킹 전략: structured space-time masking을 사용하며, 이는 보이는 토큰과 목표 토큰 사이의 거리를 증가시켜 더 넓은 범위의 정보를 사용하여 예측하도록 강제합니다.
        - 손실 함수: PatchDiscB 손실을 사용합니다. 이는 배치 내의 다른 샘플로부터의 부정적인 예시를 포함하여 글로벌 식별력을 향상시키며, 분류 작업에 적합한 전역 특징을 학습하도록 설계되었습니다.
    - 지역 특징 학습
        - 목표 표현: 모델의 가장 낮은 표현 수준, 즉 입력 공간의 얕은 shallow linear projections을 목표로 합니다. 이는 깊은 트랜스포머 블록의 처리를 건너뛰어 지역적이고 미세한 특징을 학습하는 데 유용합니다.
        - 마스킹 전략: unstructured random masking을 사용하며, 이는 보이는 토큰과 목표 토큰 사이의 평균 거리를 최소화하여 국소적인 정보를 사용하여 예측하도록 유도합니다.
        - 손실 함수: PatchDisc 손실을 사용하여 낮은 수준의 특징을 기반으로 패치 간의 구별을 하도록 지시합니다. 이는 미세한 지역 특징을 증폭하는 데 도움이 됩니다.
- 예측 과정
    - 예측자 트랜스포머(predictor transformer) P는 목표 토큰에 대한 위치, 시간, 월, 채널 그룹 임베딩 et를 입력으로 받고, 가시 토큰 인코딩 zv에 교차-어텐딩하여 패치 인코딩 pt를 예측합니다.
    - 이 예측 pt와 목표 zt를 비교하여 온라인 인코더를 업데이트하는 손실 L(pt, zt)를 계산합니다.

## Transformer를 이용하면서 얻은 이점 및 강점

Galileo는 ViT(Vision Transformer) 기반으로 설계되어 RS 데이터의 복잡성을 효과적으로 모델링합니다. Transformer의 Attention 메커니즘과 토큰화(Tokenization)가 핵심으로, 웹 검색 결과에서도 RS에서 Transformer가 "higher accuracy, scalability, generalization"을 제공한다고 강조됩니다.  아래는 주요 이점입니다.

### **1. 유연한 입력 처리**

Transformer의 토큰화로 공간(H×W), 시간(T), 채널 그룹(G)을 독립적으로 처리. 예: 공간-시간 데이터 → 패치로 분할 후 위치/채널/월 임베딩 추가

- 이점: 다양한 모달리티와 형태(이미지 ~ 시계열)를 단일 아키텍처로 통합. 기존 CNN 기반(MMEarth 등)보다 RS 데이터의 불규칙성을 잘 다룸.
- 강점: 패치 크기 조정으로 MACs(Multiply-Accumulate Operations) 줄여 컴퓨트 제한 환경에서 유용

### **Attention 메커니즘의 공간-시간 관계 모델링**

Self-Attention으로 토큰 간 장거리 의존성(Long-Range Dependencies) 학습. RS에서 공간(픽셀 간)과 시간(타임스텝 간) 관계를 동시에 캡처

- 이점: 멀티스케일 객체 처리 우수 (작은 객체: 로컬 Attention, 큰 객체: 글로벌 Attention). 기존 방법(MAE 스타일)보다 "stable performance in segmentation" 제공
- 강점: 계절성(월 임베딩)과 위치 임베딩으로 RS 특성(예: 작물 변화) 반영. RS Transformer 서베이에서 "LULC classification and fusion"에서 centimeter-level accuracy 달성

### **Scalability와 Generalization**

Scalability는 모델이 다양한 컴퓨팅환경과 자원 제약 조건에 맞춰 성능과 비용의 균형을 조절할 수 있는 능력을 의미합니다. Generalization는 모델이 특정 작업이나 데이터 유형에만 특화되지 않고, 다양한 종류의 새로운 작업, 데이터 양식, 입력 형태에 대해 우수한 성능을 보이는 능력을 의미합니다.

- Transformer의 모듈러 구조로 모델 크기(ViT-Nano ~ Base) 쉽게 조정. 프리트레이닝 후 파인튜닝 용이
- 이점: 대규모 데이터셋(127k 인스턴스)에서 학습, 다양한 태스크(분류, 세그멘테이션)로 전이. RS에서 Transformer가 "enhanced accuracy and generalisation when processing satellite data"라고 웹 결과
- 강점: EMA 타겟 인코더와 Predictor Transformer로 SSL 안정성 향상

### **RS 특화 이점**

RS 데이터의 고차원성(멀티스펙트럴 채널)을 Attention으로 효율 처리. 기존 CNN보다 "time series observations" 분석에 강합니다.

**1. 다양한 RS 양식 및 고차원 데이터의 통합 처리**

> Galileo는 다중 스펙트럼 광학, 합성 개구 레이더(SAR), 고도, 날씨, 유사 라벨, 야간 조명, 인구 추정치 등 다양한 RS 양식을 통합적으로 표현하는 "고도로 다중 양식적인 Transformer"입니다. 이러한 다양한 입력 데이터는 Transformer의 토큰화(tokenization) 과정을 통해 잠재 토큰(latent tokens) 시퀀스로 변환되어 통합적으로 처리됩니다. 이는 각 데이터 양식의 고유한 특성(예: 다양한 채널 그룹)을 유연하게 받아들이면서도, 이를 단일 모델 내에서 효율적으로 처리할 수 있게 합니다. 기존 CNN 기반 모델들은 특정 양식이나 데이터 형태에 특화되어 있었지만, Transformer는 이러한 다양한 양식을 유연하게 결합하여 학습함으로써 RS 데이터의 복잡한 특성을 포괄적으로 이해할 수 있습니다.
> 

**2. Attention 메커니즘을 통한 비국소적 및 다중 스케일 특징 학습**

> RS 데이터는 1~2픽셀 크기의 작은 보트부터 수천 픽셀 크기의 빙하에 이르기까지 객체의 스케일이 매우 다양합니다. Transformer의 self-attention 메커니즘은 입력 시퀀스 내의 모든 토큰 간의 관계를 동시에 고려하여 비국소적 처리를 수행합니다. 이는 전통적인 CNN이 가진 제한된 수용장으로 인한 국소적 특징 추출의 한계를 넘어섭니다. 이러한 비국소적 처리 능력 덕분에 Galileo는 전역적 표현과 지역적 표현을 동시에 학습하는 이중 목표 자기 지도 학습(SSL) 알고리즘을 구현할 수 있었으며, 이는 다양한 스케일의 현상과 복잡한 공간적 패턴을 효과적으로 모델링하는 데 핵심적인 역할을 합니다.
> 

**3. 유연한 시계열 및 입력 형태 처리 능력**

> Galileo의 Transformer 아키텍처는 단일 시간 스텝 이미지, 다중 시간 스텝 이미지, 픽셀 시계열 등 다양한 입력 형태를 유연하게 처리할 수 있도록 설계되었습니다. 이는 특히 "time series observations" 분석에 큰 강점입니다. 전통적인 시계열 분석 모델들은 특정 시계열 형태에 맞춰 설계되는 경우가 많았지만, Transformer는 시간적 차원을 토큰 시퀀스의 일부로 간주하고, 시간적 사인파 위치 임베딩과 월별 임베딩을 추가하여 계절적 변화 및 시간적 맥락을 이해합니다. 이를 통해 모델은 짧은 기간 동안 빠르게 변화하는 현상과 수십 년에 걸쳐 추적해야 하는 현상을 모두 효과적으로 처리할 수 있습니다.
>
