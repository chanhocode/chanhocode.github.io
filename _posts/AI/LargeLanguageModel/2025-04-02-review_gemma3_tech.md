---
title: "[Paper] Gemma 3 Technical Report 핵심 요약"
writer: chanho
date: 2025-04-02 12:11:00 +0900
categories: [AI, LargeLanguageModel]
tags: [ai, llm, paper]
---

# Gemma3 Paper
> We introduce Gemma 3, a multimodal addition to the Gemma family of lightweight open models, ranging in scale from 1 to 27 billion parameters. This version introduces vision understanding abilities, a wider coverage of languages and longer context - at least 128K tokens. We also change the architecture of the model to reduce the KV-cache memory that tends to explode with long context. This is achieved by increasing the ratio of local to global attention layers, and keeping the span on local attention short. The Gemma 3 models are trained with distillation and achieve superior performance to Gemma 2 for both pre-trained and instruction finetuned versions. In particular, our novel post-training recipe significantly improves the math, chat, instruction-following and multilingual abilities, making Gemma3-4B-IT competitive with Gemma2-27B-IT and Gemma3-27B-IT comparable to Gemini-1.5-Pro across benchmarks. We release all our models to the community.

[Gemma 3 Technical Report](https://arxiv.org/abs/2503.19786)

### 모델 개요

<table style="width: 100%">
  <thead>
    <tr>
      <th>항목</th>
      <th>내용</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>모델 유형</strong></td>
      <td>오픈소스 <strong>멀티모달 모델</strong> (텍스트+이미지 입력, 텍스트 출력)</td>
    </tr>
    <tr>
      <td><strong>컨텍스트 길이</strong></td>
      <td>최대 <strong>128K 토큰</strong> (1B 모델은 32K)</td>
    </tr>
    <tr>
      <td><strong>지원 언어</strong></td>
      <td><strong>140개 이상 다국어</strong></td>
    </tr>
    <tr>
      <td><strong>모델 크기</strong></td>
      <td>1B, 4B, 12B, 27B</td>
    </tr>
    <tr>
      <td><strong>활용 예시</strong></td>
      <td>질의응답, 요약, 추론, 이미지 분석 등</td>
    </tr>
    <tr>
      <td><strong>출력 제한</strong></td>
      <td>최대 <strong>8K 토큰</strong></td>
    </tr>
  </tbody>
</table>

### 1. Introduction

> **Gemma 3**는 **경량(open lightweight) 모델** 시리즈의 최신 버전으로, **1B~27B 파라미터** 모델을 포함합니다. 기존 **Gemma 2**보다 다양한 능력(수학, 대화, 다국어, 추론 등)에서 더 우수하며 **‘Gemini-1.5-Pro’에 필적하는 벤치마크 성능**을 보입니다.

**핵심 특징:**

- **다국어 성능** 우수
- **긴 문맥 처리** 능력 (최대 128K 토큰)
- **멀티모달 지원**: SigLIP 비전 인코더를 통해 이미지 이해 가능
- **Pan and Scan(P&S)** 방식 지원
  - 고해상도 이미지를 다양한 위치로 나눠 **패치 단위로 입력 및 분석**
- **KV 캐시 메모리 최적화**

### 2. Model Architecture

<div style="display: flex; flex-wrap: wrap; gap: 20px; align-items: flex-start;">

<div style="flex: 1; min-width: 300px;">

<h4>주요 개선</h4>

- <b>Grouped-Query Attention (GQA)</b> 도입  
- <b>QK-norm</b> 채택으로 안정성 향상  
- <b>5:1 비율의 Local:Global attention layer 구성</b>으로 메모리 절감  
  - <i>Local Attention</i>: 근처 토큰만 참조 → 메모리 효율적  
  - <i>Global Attention</i>: 전체 문맥 참조 → 계산량 많음  
  - <i>5:1 interleaving</i> 구조: 대부분 Local, 가끔 Global로 전체 이해 유지  
- <b>128K 긴 문맥 처리</b> 지원

</div>
<div style="flex: 1; min-width: 200px;">

<h4>구조 개요</h4>
: <b>Transformer 기반 decoder-only 구조</b> 유지  

<img width="100%" alt="image" src="https://github.com/user-attachments/assets/8edeed13-509c-44d3-bae1-2080a3f51fdf">
<small><i>_Image Source: [Google](https://arxiv.org/abs/2503.19786)_</i></small>

</div>

</div>

### 2.1 Vision Modality

- **SigLIP 기반 비전 인코더 (400M)** 사용
- 고정 해상도(896x896) 처리 → **Pan & Scan (P&S)** 기법으로 다양한 이미지 크기 대응
- 이미지 토큰 수: **256개로 고정**


### 2.2 Pre-training

- **지식 증류 (Knowledge Distillation)** 기반 학습
- 훈련 데이터 토큰 수 증가 (예: 27B 모델은 14T tokens)
- **다국어 데이터 비중 확대** + **이미지 포함**
- **SentencePiece tokenizer (Gemini 2.0 동일)** 사용

### 2.3

<div style="display: flex; flex-wrap: wrap; gap: 20px; align-items: flex-start;">

<div style="flex: 1; min-width: 200px;">

<img width="100%" alt="image" src="https://github.com/user-attachments/assets/90a60782-9662-40e3-a0d0-4ebdeca7801a">
<small><i>_Image Source: [Google](https://arxiv.org/abs/2503.19786)_</i></small>

</div>
<div style="flex: 1; min-width: 300px;">

<ul>
  <li><strong>int4, SFP8 등 다양한 양자화 포맷</strong> 지원</li>
  <li><strong>양자화 인식 학습(QAT)</strong> 수행 (5,000 스텝 fine-tuning)</li>
</ul>

</div>

</div>

### 3. Instruction Tuning

- **대형 IT 모델 기반 지식 증류** + **강화학습(RLHF)**
    - **지식 증류 (Knowledge Distillation)**: 대형 IT 모델로부터 학습하여 핵심 능력(수학, 코딩, 다국어 등) 전수.
    - **강화학습 (RLHF)**: BOND, WARM, WARP 기법으로 보상 기반 튜닝 → 도움성↑, 유해성↓
- **BOND, WARM, WARP** 등 최신 기술 활용
- 다양한 **보상 함수 (수학 정확도, 코드 실행 등)** 사용
- **안전 필터링 및 거절 전략 학습**으로 허위 생성 방지
    - 개인 정보, 유해 콘텐츠 제거 + 출처 인용/거절 표현 포함 → 환각(hallucination) 방지


<div style="display: flex; flex-wrap: wrap; gap: 20px; align-items: flex-start;">

<div style="flex: 1; min-width: 300px;">

<img width="100%" alt="image" src="https://github.com/user-attachments/assets/d9bdf330-1f14-498a-8fec-2e3f4e7cef8c">
<small><i>_Image Source: [Google](https://arxiv.org/abs/2503.19786)_</i></small>

</div>
<div style="flex: 1; min-width: 300px;">

<ul>
  <li><strong>포맷 규칙</strong>:
    <ul>
      <li>문장 시작에 <code>[BOS]</code> 필수 (<code>add_bos=True</code>)</li>
      <li>PT 모델: <code>&lt;eos&gt;</code> / IT 모델: <code>&lt;end_of_turn&gt;</code> 사용</li>
    </ul>
  </li>
</ul>

</div>

</div>


### 4. Evaluation

- **LMSYS Chatbot Arena** 기준 상위권 모델 진입 (Elo 1338, 27B 기준)
- **Gemma 2 27B (Elo 1220)** 대비 큰 폭 향상
- **MMLU, LiveCodeBench, MATH** 등 다양한 벤치마크에서 성능 개선

<div style="display: flex; flex-wrap: wrap; gap: 20px; align-items: flex-start;">

<div style="flex: 1; min-width: 300px;">

<img width="100%" alt="image" src="https://github.com/user-attachments/assets/fd00c320-5996-4213-beaa-605c80a80f73">
<small><i>_Image Source: [Google](https://arxiv.org/abs/2503.19786)_</i></small>

</div>
<div style="flex: 1; min-width: 300px;">

<img width="100%" alt="image" src="https://github.com/user-attachments/assets/223e027f-fe90-4a85-a903-4f89dbed6174">
<small><i>_Image Source: [Google](https://arxiv.org/abs/2503.19786)_</i></small>

</div>

</div>

### 5. Ablation Studies
> Local:Global 비율 조정 실험에서는 5:1 비율이 최적으로 나타났으며, Sliding Window 크기 축소는 퍼플렉시티에 미치는 영향이 적었다. KV 캐시 메모리 최적화를 통해 성능 개선 효과가 확인되었고, 128K 문맥 확장 실험에서는 RoPE 주파수 재조정과 스케일링 기법이 활용되었다.

### 6. Memorization & Privacy
> Local:Global 비율 조정 실험 결과, 최적의 성능은 5:1 비율에서 확인되었으며, Sliding Window 크기 축소는 퍼플렉시티에 큰 영향을 미치지 않는 것으로 나타났다. 또한, KV 캐시 메모리 최적화를 통해 성능 개선 효과가 확인되었고, 128K 문맥 확장 실험에서는 RoPE 주파수 재조정과 스케일링 기법이 효과적으로 활용되었다.

### 7. Safety & Responsible AI
> 훈련 시 데이터 필터링과 RLHF를 통해 모델의 안전성을 확보했으며, CBRN 지식 테스트와 같은 다양한 위험 평가도 수행되었다. 또한, Gemma 3 4B를 기반으로 한 이미지 안전 모델인 ShieldGemma 2가 공개되었다.

### 8. Conclusion
> Gemma 3는 가볍지만 강력한 멀티모달 LLM으로, 긴 문맥 처리, 이미지 이해, 다국어 지원 등 다양한 영역에서 성능이 향상되었다. 오픈 소스로 제공되며, 경량화와 실용성을 동시에 지향한다.

<br/>

## **Gemma License (Summary)**

### 1. 정의

- **Gemma**: Google이 제공하는 LLM 모델 세트 (가중치, 파라미터 포함).
- **Model Derivatives**: Gemma를 수정하거나, distillation 등으로 유사하게 만든 모델.
- **Output**: Gemma 또는 파생 모델로 생성된 결과물. → *Output은 파생 모델이 아님!*


### 2. 사용 조건

- 개인/조직은 본 약관에 동의해야 사용 가능.
- Gemma 서비스 사용은 본 약관에 **엄격히 따라야 함**.


### 3. 배포 및 제한

- Gemma 또는 파생 모델을 배포하려면:
    - *제한사항(Section 3.2)**을 포함한 약관 고지 필요
    - 이 약관 사본 제공 필수
    - 수정 내용은 명시해야 함
    - 함께 "Gemma는 ai.google.dev/gemma/terms 약관에 따릅니다" 문구 포함
- **금지된 사용** (Prohibited Use Policy에 따라):
    - 예: 불법, 유해, 차별적, 개인정보 침해 등의 목적
    - 위반 시 Google은 사용 제한 권한 보유


### 4. 추가 조항

- **Output 권리**: 사용자가 만든 Output에 대해 Google은 권리 없음 → 사용자 책임
- **업데이트**: Google은 언제든지 Gemma를 업데이트할 수 있음
- **상표**: Google 로고나 상표는 사용 불가
- **보증 없음**: "있는 그대로" 제공되며, Google은 사용 결과에 대해 책임지지 않음
- **책임 제한**: 직접적/간접적 손해에 대해 Google은 법적 책임 없음
- **약관 종료**: 위반 시 Google이 약관 종료 가능, 종료 시 모든 사본 삭제 의무
- **관할법**: 캘리포니아 법 적용, 분쟁은 산타클라라 카운티 법원 관할
