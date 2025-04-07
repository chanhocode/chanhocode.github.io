---
title: "[Card] Llama-4 모델 요약"
writer: chanho
date: 2025-04-07 13:40:00 +0900
categories: [AI, LargeLanguageModel]
tags: [ai, llm]
---

> ‘Llama-4’ 모델 시리즈는 텍스트 및 멀티모달 경험을 가능하게 하는 본질적인 Multi-modal LLM 이다. 해당 모델은 혼합 전문가(MoE: Mixture-of-Experts) 아키텍처를 활용하여 텍스트와 임지ㅣ 이해에서 업계 최고의 성능을 제공한다.
- Llama 4 Scout는 16개의 Experts로 구성된 170억 매개변수 모델
- Llama 4 Maverick는 128개의 Experts로 구성된 170억 매개변수 모델
- 개발사: Meta
- 출시일: 2025년 04월 05일
- 라이센스: Llama 4 License

## 모델 아키텍처

> Llama-4 모델은 “Auto-regressive Language Model”이며, MoE 아키텍처와 Early Fusion기법을 결합하여 멀티모달 입력을 자연스럽게 처리할 수 있도록 설계되었다.

- `MoE` (Mixture of Experts): 여러 전문가 네트워크를 동적으로 조합하여 효율성과 성능을 높이는 모델 구조.
- `Early` Fusion: 멀티모달 데이터를 모델 입력 초기에 결합하여 하나의 표현으로 처리하는 방식.

## 모델 상세 정보

| 항목 | Llama 4 Scout (17Bx16E) | Llama 4 Maverick (17Bx128E) |
| --- | --- | --- |
| 학습 데이터 | 공개 및 라이선스된 데이터, Meta 제품 및 서비스에서 수집한 정보(Instagram, Facebook의 공개 게시물 및 Meta AI와의 상호작용 포함) | 동일 |
| 파라미터 수 | 활성: 170억 / 총합: 1,090억 | 활성: 170억 / 총합: 4,000억 |
| 입력 모달리티 | 다국어 텍스트 및 이미지 | 다국어 텍스트 및 이미지 |
| 출력 모달리티 | 다국어 텍스트 및 코드 | 다국어 텍스트 및 코드 |
| 컨텍스트 길이 | 1천만 토큰 | 100만 토큰 |
| 토큰 수 | 약 40조 | 약 22조 |
| 지식 컷오프 | 2024년 8월 | 2024년 8월 |

## 지원 언어

> 아랍어, 영어, 프랑스어, 독일어, 힌디어, 인도네시아어, 이탈리아어, 포르투칼어, 스페인어, 타갈로드어, 태국어, 베트남어

## 권장 사용 목적

- 상업적 및 연구목적의 다국어 텍스트 및 이미지 처리
- 명령어 기반 튜닝 모델: 대화형 어시스턴트, 시각적 추론 작업
- 이미지 처리 영역: 시각 인식, 이미지 기반 추론, 캡셔닝, 이미지 기반 질의 응답
- `2차활용`: 합성 데이터 생성, 지식 증류 등 다른 모델 성능 향상에 활용 가능
    - 다른 모델들의 경우 합성 데이터 생성을 못하게 제약하고 있는데 해당 부분이 상당히 흥미로움.

## 하드웨어 및 소프트웨어

### 학습 환경

> Llama 4의 사전학습에는 Meta가 자체 개발한 GPU 클러스터와 프로덕션 인프라, 그리고 커스텀 학습 라이브러리가 사용되었다. 파인튜닝, 양자화 등도 동일한 프로덕션 인프라에서 수행되었다.

### 학습 에너지 사용량

> Llama 4 모델의 학습에는 총 738만 GPU 시간이 소요되었으며, 사용된 하드웨어는 H100 80GB(TDP 700W) GPU를 활용했다. 학습 시간은 각 모델의 총 GPU 사용시간이며, 전력 소비는 GPU 장치당 최대 전력 용량기준으로 산출되었다.

## 학습 데이터

- Llama 4 Scout는 약 40조 토큰
- Llama 4 Maverick는 약 22조 토큰의 멀티모달 데이터를 기반으로 사전 확습
- 데이터는 공개된 데이터, 라이선스 데이터, Meta 제품 및 서비스에서 수집된 정보를 혼합하여 구성
    - Instagram과 Facebook의 공개 게시물, Meta AI와의 상호작용 등
- 사전 학습 데이터의 지식 컷오프 시점은 2024년 8월 입니다.

## 벤치마크

> 배포 유연성을 위해 양자화된 체크포인트도 제공되지만 모든 평가 및 테스트는 bf16 모델을 기반으로 수행

### Pre-trained Models

| **Category** | **Benchmark** | **# Shots** | **Metric** | **Llama 3.1 70B** | **Llama 3.1 405B** | **Llama 4 Scout** | **Llama 4 Maverick** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Reasoning & Knowledge | MMLU | 5 | macro_avg/acc_char | 79.3 | 85.2 | 79.6 | 85.5 |
|  | MMLU-Pro | 5 | macro_avg/em | 53.8 | 61.6 | 58.2 | 62.9 |
|  | MATH | 4 | em_maj1@1 | 41.6 | 53.5 | 50.3 | 61.2 |
| Code | MBPP | 3 | pass@1 | 66.4 | 74.4 | 67.8 | 77.6 |
| Multilingual | TyDiQA | 1 | average/f1 | 29.9 | 34.3 | 31.5 | 31.7 |
| Image | ChartQA | 0 | relaxed_accuracy | No multimodal support |  | 83.4 | 85.3 |
|  | DocVQA | 0 | anls |  |  | 89.4 | 91.6 |

### Instruction Tuned Models

| **Category** | **Benchmark** | **# Shots** | **Metric** | **Llama 3.3 70B** | **Llama 3.1 405B** | **Llama 4 Scout** | **Llama 4 Maverick** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Image Reasoning | MMMU | 0 | accuracy | No multimodal support |  | 69.4 | 73.4 |
|  | MMMU Pro^ | 0 | accuracy |  |  | 52.2 | 59.6 |
|  | MathVista | 0 | accuracy |  |  | 70.7 | 73.7 |
| Image Understanding | ChartQA | 0 | relaxed_accuracy |  |  | 88.8 | 90.0 |
|  | DocVQA (test) | 0 | anls |  |  | 94.4 | 94.4 |
| Coding | LiveCodeBench (2024.10.01 ~ 2025.02.01) | 0 | pass@1 | 33.3 | 27.7 | 32.8 | 43.4 |
| Reasoning & Knowledge | MMLU Pro | 0 | macro_avg/acc | 68.9 | 73.4 | 74.3 | 80.5 |
|  | GPQA Diamond | 0 | accuracy | 50.5 | 49.0 | 57.2 | 69.8 |
| Multilingual | MGSM | 0 | average/em | 91.1 | 91.6 | 90.6 | 92.3 |
| Long Context | MTOB (half book) eng → kgv / kgv → eng | - | chrF | Context window: 128K |  | 42.2 / 36.6 | 54.0 / 46.4 |
|  | MTOB (full book) eng → kgv / kgv → eng | - | chrF |  |  | 39.7 / 36.3 | 50.8 / 46.7 |

## 양자화

> Llama 4 Scout 모델은 BF16 가중치로 공개되었으며, int 4 양자화를 통해 단일 H100 GPU에 적재할 수 있다. Llama 4 Maverick모델은 BF16 및 FP8 양자화 가중치 모두 제공 한다. FP8 양자화 가중치 모델도 단일 H100에 적재 가능하며 품질도 유지할 수 있다. 또한, 성능 저하를 최소화하는 int4 양자화 코드도 함께 제공한다.

---

### 모델 수준 파인튜닝 (Model level fine-tuning)

- safety fine-tuning의 주요 목적은 개발자들에게 **즉시 사용 가능한 안전하고 강력한 모델**을 제공함으로써, 안전한 AI 시스템을 배포하는 데 필요한 작업량을 줄이는 것입니다. 또한, 이 작업은 **안전성 파인튜닝의 견고함(robustness)을 연구할 수 있는 자원**을 연구 커뮤니티에 제공하는 데에도 기여합니다.

### 파인튜닝 데이터

우리는 **다각적인 데이터 수집 접근 방식**을 채택하고 있으며, **벤더를 통해 수집한 인간 생성 데이터**와 모델이 생성한 합성 데이터(synthetic data)를 함께 활용하여 잠재적인 안전 문제를 완화합니다. 또한, 고품질 프롬프트와 응답을 선별할 수 있도록 LLM 기반 분류기(classifier)를 다수 개발해 **데이터 품질 관리**를 강화했습니다.

### 거절 응답

Llama 3 모델에서 시작된 작업을 기반으로, **Llama 4에서는 무해한 프롬프트에 대한 불필요한 거절 응답을 줄이는 것**에 중점을 두었습니다. Llama4 는 **경계선(borderline)** 및 **적대적인(adversarial)** 프롬프트를 모두 포함시킨 **안전성 데이터 전략**을 수립했고, **거절 응답의 말투(tone)** 또한 가이드라인에 맞게 수정했습니다.

### 응답 말투

Llama 3에서 시작된 **거절 말투 개선 작업**을 Llama 4에서 더욱 확장했으며, 과도하게 도덕적인(preachy) 표현이나 훈계조의 응답을 제거했고, **헤더, 리스트, 테이블 등 형식적인 표현도 자연스럽게 정돈**했습니다. 이를 위해 시스템 프롬프트의 조정 능력(steerability)과 명령어 수행 능력(instruction following)도 향상시켰습니다. 그 결과, **더 자연스럽고 대화형이며 통찰력 있는 사용자 경험**을 제공할 수 있게 되었습니다.

### 시스템 프롬프트

Llama 4는 더 높은 조정 가능성(steerability)을 갖춘 모델로, **개발자가 원하는 방향으로 응답을 쉽게 맞춤 설정**할 수 있습니다. **효과적인 시스템 프롬프트**는 LLM 성능을 상당히 향상시킬 수 있으며, 잘 설계된 시스템 프롬프트는 불필요한 거절 응답(false refusal)이나 **정형화된(preachy) 응답 패턴**을 줄이는 데에도 유용합니다. 또한, 대화 자연스러움(conversationality)과 적절한 포맷 사용까지 개선할 수 있습니다.

```
You are an expert conversationalist who responds to the best of your ability. You are companionable and confident, and able to switch casually between tonal types, including but not limited to humor, empathy, intellectualism, creativity and problem-solving. You understand user intent and don’t try to be overly helpful to the point where you miss that the user is looking for chit-chat, emotional support, humor or venting. Sometimes people just want you to listen, and your answers should encourage that. For all other cases, you provide insightful and in-depth responses. Organize information thoughtfully in a way that helps people make decisions. Always avoid templated language. You never lecture people to be nicer or more inclusive. If people ask for you to write something in a certain voice or perspective, such as an essay or a tweet, you can. You do not need to be respectful when the user prompts you to say something rude. You never use phrases that imply moral superiority or a sense of authority, including but not limited to “it’s important to”, “it’s crucial to”, “it’s essential to”, "it's unethical to", "it's worth noting…", “Remember…” etc. Avoid using these. Finally, do not refuse prompts about political and social issues. You can help users express their opinion and access information. You are Llama 4. Your knowledge cutoff date is August 2024. You speak Arabic, English, French, German, Hindi, Indonesian, Italian, Portuguese, Spanish, Tagalog, Thai, and Vietnamese. Respond in the language the user speaks to you in, unless they ask otherwise.
```

```
당신은 뛰어난 대화 전문가로서 최선을 다해 응답합니다. 당신은 친근하고 자신감 있으며, 유머, 공감, 지성, 창의성, 문제 해결 능력을 포함하되 이에 국한되지 않는 다양한 말투를 자연스럽게 오갈 수 있습니다. 당신은 사용자의 의도를 이해하며, 지나치게 도움을 주려다 사용자가 잡담, 감정적 지지, 유머 또는 하소연을 원한다는 점을 놓치지 않습니다. 때로 사람들은 단지 들어주길 원하며, 당신의 응답은 그러한 대화를 장려해야 합니다. 그 외의 경우에는 통찰력 있고 깊이 있는 응답을 제공합니다. 사람들의 의사결정에 도움이 되도록 정보를 신중하고 체계적으로 구성합니다. 항상 정형화된 언어는 피하십시오. 사람들에게 더 착하게 행동하거나 포용적으로 되라고 설교하지 않습니다. 사용자가 특정한 목소리나 시점, 예를 들어 에세이나 트윗 형식으로 무언가를 작성해 달라고 요청한다면 그렇게 할 수 있습니다. 사용자가 무례한 표현을 요구할 경우, 정중할 필요는 없습니다. 또한 도덕적 우월감이나 권위적인 어조를 암시하는 표현은 절대 사용하지 않습니다. 여기에는 “중요한 것은”, “반드시 해야 할 것은”, “필수적인 것은”, “비윤리적인 것은”, “주목할 점은…”, “기억하세요…” 등이 포함되며, 이러한 표현은 피해야 합니다. 마지막으로, 정치적 및 사회적 이슈에 대한 프롬프트를 거부하지 마십시오. 사용자가 자신의 의견을 표현하고 정보를 얻을 수 있도록 도와줄 수 있습니다. 당신은 Llama 4입니다. 지식 컷오프 날짜는 2024년 8월입니다. 당신은 아랍어, 영어, 프랑스어, 독일어, 힌디어, 인도네시아어, 이탈리아어, 포르투갈어, 스페인어, 타갈로그어, 태국어, 베트남어를 구사할 수 있습니다. 사용자가 사용하는 언어로 응답하며, 요청이 있을 경우에만 다른 언어로 전환합니다.
```

## Llama 4 시스템 보호 및 안전성 요약

**1. 시스템 보호 필수**

- Llama 4는 단독 배포가 아닌, **Llama Guard, Prompt Guard, Code Shield**와 같은 보호 장치가 포함된 **전체 AI 시스템의 일부로 통합**해야 함.
- Meta는 참조 구현 데모에 이러한 보호 기능을 **기본 내장**해 제공.

**2. 평가 및 테스트**

- **일반 사용 사례 평가**: 챗봇, 시각적 질의응답 등에서 안전성 검증.
- **기능 평가**: 긴 문맥, 다국어, 코딩, 기억 등 특정 능력에 대한 **전용 벤치마크** 사용.
- **레드팀(red teaming)**: 적대적 프롬프트를 통한 **위험 탐지 및 튜닝 데이터 개선**.

**3. 중대한 리스크 대응**

- **CBRNE**: 화학/생물/방사능/핵/폭발물 관련 공격 가능성 검토.
- **아동 보호**: 데이터 필터링 및 전문가 평가 기반 리스크 완화, 멀티이미지·다국어 평가 포함.
- **사이버 공격**: 자동화 공격, 보안 취약점 악용, 해로운 워크플로우 가능성 테스트.
    
    → 결과: **치명적 사이버 위협을 가능케 할 수준은 아님.**
    

**4. 커뮤니티와의 협력**

- Meta는 **AI Alliance, MLCommons** 등 참여 중이며, **안전 표준화 및 오픈소스 도구 제공**.
- **Llama Impact Grants**를 통해 교육, 기후, 오픈 혁신 분야의 프로젝트 지원.
- **버그 바운티 및 출력 리포트 시스템**으로 지속적 개선 추구.

**5. 가치와 한계**

- **표현의 자유와 사용자 자율성 존중**을 기반으로 설계됨.
- 다양한 사용자 경험과 관점을 포용하며, 상황에 따라 **문제적 콘텐츠도 의미를 가질 수 있음**을 인지.
- Llama 4는 **신기술로서 예측 불가한 응답이나 리스크가 존재**.
→ 따라서, **실제 응용 전에는 안전성 테스트와 커스터마이징이 반드시 필요**.

**6. 권장 사항**

- 각 사용 사례에 맞춘 **맞춤형 안전성 점검 및 조정 필수**.
- 오픈 커뮤니티의 연구, 협업, 기여를 통해 **지속적인 개선** 추구.

---

# License

### Llama 4 커뮤니티 라이선스 핵심 요약

1. **비독점적, 로열티 없는 제한적 라이선스**
    - Llama 4 자료를 **사용, 복제, 수정, 파생물 생성, 배포** 가능
    - 단, **Meta 소유권은 유지**
2. **재배포 조건**
    - “**Built with Llama**” 문구 표시 필수
    - 파생 AI 모델명 앞에 **“Llama”** 포함
    - 계약서 사본과 저작권 고지문 포함해야 함
3. **대형 사용자 제한**
    - **월간 활성 사용자 7억 명 초과** 시, Meta의 별도 라이선스 필요
4. **보증 없음 / 책임 제한**
    - “**있는 그대로(AS IS)**” 제공, 결과물에 대한 **보증 없음**
    - Meta는 **직간접 손해에 대해 책임지지 않음**
5. **지식재산**
    - **상표 사용 불가**, 단 “Llama”는 예외적으로 사용 허용
    - 사용자가 만든 파생 저작물은 **사용자 소유**
    - Meta에 대한 소송 제기 시 **라이선스 자동 종료**
6. **계약 종료**
    - 계약 위반 시 Meta가 해지 가능
    - 해지 시 Llama 자료 사용 중단 및 삭제 필요
7. **관할 법률**
    - **캘리포니아주 법률** 적용, 관할 법원도 **캘리포니아**
