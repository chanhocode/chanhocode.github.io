---
title: "[RAG] Self-Routing RAG Paper 리뷰"
writer: chanho
date: 2025-04-21 16:30:00 +0900
categories: [AI, LargeLanguageModel]
tags: [ai, rag, paper]
---
[💡 Self-Routing RAG: Binding Selective Retrieval with Knowledge Verbalization](https://arxiv.org/abs/2504.01018)

> Selective retrieval improves retrieval-augmented generation (RAG) by reducing distractions from low-quality retrievals and improving efficiency. However, existing approaches under-utilize the inherent knowledge of large language models (LLMs), leading to suboptimal retrieval decisions and degraded generation performance. To bridge this gap, we propose Self-Routing RAG (SR-RAG), a novel framework that binds selective retrieval with knowledge verbalization. SR-RAG enables an LLM to dynamically decide between external retrieval and verbalizing its own parametric knowledge. To this end, we design a multi-task objective that jointly optimizes an LLM on knowledge source selection, knowledge verbalization, and response generation. We further introduce dynamic knowledge source inference via nearest neighbor search to improve the accuracy of knowledge source decision under domain shifts. Fine-tuning three LLMs with SR-RAG significantly improves both their response accuracy and inference latency. Compared to the strongest selective retrieval baseline, SR-RAG reduces retrievals by 29% while improving the performance by 5.1%.

[Di Wu](https://arxiv.org/search/cs?searchtype=author&query=Wu,+D), [Jia-Chen Gu](https://arxiv.org/search/cs?searchtype=author&query=Gu,+J), [Kai-Wei Chang](https://arxiv.org/search/cs?searchtype=author&query=Chang,+K), [Nanyun Peng](https://arxiv.org/search/cs?searchtype=author&query=Peng,+N)

- https://github.com/xiaowu0162/self-routing-rag

---

> 해당 논문에서는 “SR-RAG”라는 새로운 검색 증강 생성 프레임워크를 제안한다. SR-RAG는 기존 Selective Retrieval 방식의 한계를 극복하고 LLM의 성능과 효율성을 향상시키는 것을 목표로 한다.

## **핵심 개념**

- **RAG (Retrieval-Augmented Generation)**
    
    대규모 언어모델(LLM)에 외부 지식을 결합하여 더 정확하고 시의적절한 응답을 생성하는 기법
    
- **선택적 검색 (Selective Retrieval)**
    
    모든 쿼리에 대해 외부 지식을 검색하는 대신, 필요한 경우에만 검색을 수행하여 효율성을 높이고 불필요한 잡음을 줄이는 기법
    
- **지식 언어화 (Knowledge Verbalization)**
    
    LLM이 스스로 알고 있는 내재된 지식(파라메트릭 지식)을 명시적으로 설명해내는 과정이다. 검색 없이도 고품질 응답을 유도할 수 있다.
    

## **핵심 문제**

: 기존의 선택적 검색 RAG는 외부 검색이 필요 없을 때 단순히 LLM이 직접 답변을 생성하도록 한다. 이는 LLM 자체에 내재된 방대한 지식(파라미터 지식)을 명시적으로 활용하지 못하게 하여, 검색 결정이 최적이 아닐 수 있고 최종 답변의 질이 저하될 수 있다.

## **연구 목적**

기존 선택적 검색 기법은 LLM이 이미 알고 있는 정보를 충분히 활용하지 못한다는 한계를 LLM이 검색 여부를 스스로 판단하고, 필요 시 내부 지식을 언어화하여 활용할 수 있도록 Self-Routing RAG (SR-RAG)을 제안한다.

## **Self-Routing RAG (SR-RAG)**

SR-RAG는 기존 RAG의 한계를 넘어 LLM의 내부 지식 활용을 극대화하는 새로운 프레임워크이며 핵심 아이디어는 다음과 같다.
<img width="100%" alt="image" src="https://github.com/user-attachments/assets/0d90e59c-05cf-4544-a311-d2911f32434a">

> 사용자 질의가 주어지면, 시스템은 특수 토큰 예측과 최근접 이웃 검색을 결합하여 가장 적절한 지식 소스를 먼저 선택한다. 그런 다음, 지식은 외부 소스로부터 검색되거나 LLM이 자체적으로 언어화하여 생성된다. 이 모든 과정은 하나의 외쪽부터 오른쪽으로 진행되는 생성 패스로 간소화되어 수행된다.

### **지식 소스 선택과 내부 지식 언어화의 통합**

<img width="100%" alt="image" src="https://github.com/user-attachments/assets/aebcd5bd-f827-4231-9ba2-7312edea26a6">

- LLM이 단순히 외부 검색 여부만 결정하는 것이 아니라, **외부 데이터베이스 검색** 또는 **자신의 내부(파라미터) 지식을 직접 '언어화(Verbalization)'하여 사용**할지 동적으로 스스로 라우팅(Self-Routing)하도록 한다. 즉, LLM 자체를 중요한 지식 소스로 취급.
- **구현:** 이 선택 과정은 학습 및 추론 시 special token(**`<EOQ>`**, **`<Self>`**, **`<Wiki>`**, **`<EOK>`** 등)을 활용하여, 질문 입력부터 지식 선택, 지식 생성/검색, 최종 답변 생성까지 **하나의 순차적인 생성 과정(single left-to-right generation pass)**으로 효율적으로 처리된다.
- **언어화:** 내부 지식을 활용할 경우, `GenRead`와 같은 기법을 사용하여 LLM의 파라미터 지식을 명시적인 텍스트 형태로 다양하게 생성(verbalize)한다.

### **다중 작업 학습 및 지식 품질 향상**

- LLM을 파인튜닝할 때, 다음 세 가지를 **동시에 최적화**하는 다중 작업(Multi-task) 목표를 사용.
    - (1) 최적의 지식 소스(외부 vs. 내부) 선택 능력
    - (2) 고품질의 내부 지식 언어화 능력
    - (3) 선택/생성된 지식을 바탕으로 정확한 최종 답변 생성 능력
- **학습 데이터 구축:** 학습 시, 언어화된 내부 지식과 외부 검색 결과를 각각 사용하여 정답을 생성했을 때의 로그 우도(log-likelihood) 등을 비교하여 어떤 소스가 더 유용했는지 판단하고, 이를 "선호 소스" 레이블로 사용한다.
- **품질 향상:** 특히 DPO(Direct Preference Optimization)와 같은 기법을 추가로 활용하여, LLM이 더 유용하고 고품질의 내부 지식을 스스로 언어화하도록 학습시킨다.

`SR‑RAG 학습은 데이터 구축 → 1단계 Behavior Cloning → 2단계 DPO 미세조정”으로 구성된다.`

| 단계 | 핵심 아이디어 | 세부 절차 |
| --- | --- | --- |
| **① 데이터 구축** | _LLM 스스로 생성한 지식_과 _위키 등 외부 검색 지식_을 모두 수집 후, <br> 어느 쪽이 답변에 더 기여하는지 로그 우도($l_j$)로 자동 판단 | <ul><li>**내부 지식 언어화**: `GenRead`로 n개의 다양화된 문맥 $c_{i1}, ..., c_{in}$ 생성</li><li>**외부 검색**: 밀집 검색기로 n개의 문맥 $c_{e1}, ..., c_{en}$ 수집</li><li>각 문맥 $c_j$에 대해 답변 생성 로그 우도 $l_j = p_M(a|q, c_j)$ 계산</li><li>로그 우도 순위 기반으로 더 나은 지식 소스(내부 $S_i$ 또는 외부 $S_e$) 결정</li></ul> |
| **② 1-Stage BC (Behavior Cloning)** | 지식 소스 선택, 지식 언어화, 답변 생성을 **동시에** 학습 | <ul><li>선호 소스 선택 손실($L_{src}$), (내부 지식 선택 시) 언어화 손실($L_{verb}$), 답변 생성 손실($L_{ans}$)을 합산하여 학습</li><li>최종 손실: $L_{stage1} = L_{src} + L_{verb} + L_{ans}$</li></ul> |
| **③ 2-Stage DPO (Direct Preference Optimization)** | 내부 지식 품질을 더 높이기 위해 <br> **선호 지식($c_i^+$)과 비선호 지식($c_i^-$) 쌍**에 대해 DPO로 추가 파인튜닝 | <ul><li>선호 소스 선택 손실($L_{src}$)과 답변 생성 손실($L_{ans}$)은 유지</li><li>내부 지식 언어화 손실을 DPO 손실($L^{DPO}*{verb}$)로 대체하여 학습</li><li>최종 손실: $L*{stage2} = L_{src} + L^{DPO}*{verb} + L*{ans}$</li></ul> |

### SR-RAG 학습 손실 함수 요약

| 수식 (Formula) | 의미 (Meaning) | 직관적 해설 (Intuitive Explanation) |
| --- | --- | --- |
| `$L_{src} = -\\log p_M(\\langle s \\rangle \| q)$` | 지식 소스 선택 손실 | 학습 데이터에서 결정된 선호 지식 소스 토큰 `<s>`를 예측하도록 학습한다. |
| `$L_{verb}$` | 지식 언어화 손실 | 내부 소스($S_i$)가 선택된 경우에만, 가장 도움이 되었던 **최적 언어화 문맥 $c_i^+$** 시퀀스를 생성하도록 학습한다. (외부 소스 $S_e$가 선택된 경우는 손실값 0) |
| `$L_{ans}$` | 답변 생성 손실 | 선택된 지식 소스로부터 얻은 최적 문맥(내부면 $c_i^+$, 외부면 $c_e^+$)을 조건으로 **정답 $a$**를 생성하도록 학습한다. |
| `$L_{stage1} = L_{src} + L_{verb} + L_{ans}$` | 1단계 총 손실 (BC) | 위의 세 가지 손실을 단순 합산하여, 기본적인 지식 소스 선택, 언어화, 답변 생성 행동을 모방(Behavior Cloning)하도록 학습한다. |
| `$L_{stage2} = L_{src} + L^{DPO}_{verb} + L_{ans}$` | 2단계 총 손실 (DPO) | 소스 선택($L_{src}$) 및 답변 생성($L_{ans}$) 손실은 유지하면서, 언어화 손실을 DPO 기반($L^{DPO}_{verb}$)으로 변경하여 **내부 지식 언어화 품질을 더욱 정교하게** 만든다. |
| `$L^{DPO}_{verb}$` | DPO 언어화 손실 | 내부 소스($S_i$)가 선택된 경우에만 작동한다. <br> $\sigma$는 시그모이드 함수, $\beta$는 스케일링 파라미터(온도)이다. <br> **선호 문맥 $c_i^+$**가 비선호 문맥 $c_i^-$보다 더 높은 확률을 갖도록, 즉 모델이 더 좋은 내부 지식을 생성하도록 직접적으로 선호도를 학습한다. |
- **완전 자동 라벨링**: 외부 모델·휴먼 수작업 불필요 → 확장성 우수
- **다목적 통합 학습**: 세 손실을 묶어 **한 차례 좌→우 생성 패스**로 추론 가능
- **DPO 단계**: 고비용(“System 2”) 지식을 저비용(“System 1”) 추론으로 압축해 효율 ↑

### **동적 k-NN 기반 소스 선택 (추론 시):**

- 추론 시 LLM의 단순 확률 예측에만 의존하지 않고, **k-최근접 이웃(kNN) 검색**을 함께 활용하여 소스 선택의 정확성과 견고성을 향상한다.
- **작동 방식:** 파인튜닝된 LLM의 특정 레이어 숨겨진 상태(hidden state)를 사용하여, 미리 구축된 '정책 데이터 저장소(policy datastore)'에서 현재 질문과 유사한 과거 질문들의 성공적인 소스 선택 사례(어떤 소스를 썼을 때 결과가 좋았는지)를 k개 찾는다.
- **이점:** 이 kNN 정보를 LLM의 자체 소스 예측 확률과 결합하여 최종 결정을 내린다. 이는 파인튜닝으로 인한 LLM의 능력 변화나 특정 데이터셋(도메인) 변화에 더 잘 적응하도록 돕는다.

---

## **실험 구성**

### **사용한 LLM들**

- LLaMA-2 7B Chat
- Phi-3.5-Mini-Instruct
- Qwen2.5-7B-Instruct

### **평가 데이터셋**

- **PopQA**: 롱테일 엔티티 QA
- **TriviaQA**: 일반 오픈도메인 QA
- **PubHealth**: 공중보건 팩트체크
- **ARC Challenge**: 초등 과학 문제 (다지선다형)

## **주요 실험 결과**

| 항목 | 성능 향상 | 검색 사용량 감소 |
| --- | --- | --- |
| SR-RAG vs 항상 검색 | +5.1% (정확도) | -29% (retrieval 횟수) |
| SR-RAG vs 선택적 검색 | +8.5%~4.7% | -26%~40% |
- SR-RAG은 **불필요한 검색을 줄이면서도 성능을 향상**
- 특히 내부 지식이 충분한 경우, **검색 없이 더 빠르고 정확한 응답 가능**

## **Ablation Study (구성요소 제거 실험)**

| 제거 구성 | 결과 |
| --- | --- |
| kNN 제거 | 정확도 감소 및 검색량 증가 |
| 지식 언어화 미사용 | 모델이 검색에 과도하게 의존하게 됨 |
| DPO 미사용 | verbalization 품질 저하 |

=> SR-RAG의 3가지 핵심 요소(지식 언어화, DPO, kNN 기반 선택)가 모두 성능에 필수적이다.
