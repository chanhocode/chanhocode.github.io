---
title: "[Paper] LLM 프롬프트 전략: Chain of Draft, CoD 리뷰"
writer: chanho
date: 2025-05-29 14:30:00 +0900
categories: [AI, LargeLanguageModel]
tags: [ai, llm, paper]
---
[Chain of Draft: Thinking Faster by Writing Less](https://arxiv.org/abs/2502.18600)

Large Language Models (LLMs) have demonstrated remarkable performance in solving complex reasoning tasks through mechanisms like
Chain-of-Thought (CoT) prompting, which emphasizes verbose, step-by-step reasoning. However, humans typically employ a more efficient strategy: drafting concise intermediate
thoughts that capture only essential information. In this work, we propose Chain of Draft
(CoD), a novel paradigm inspired by human
cognitive processes, where LLMs generate minimalistic yet informative intermediate reasoning outputs while solving tasks. By reducing verbosity and focusing on critical insights,
CoD matches or surpasses CoT in accuracy
while using as little as only 7.6% of the tokens, significantly reducing cost and latency
across various reasoning tasks. Our code and
data are available at https://github.com/
sileix/chain-of-draft.

[Silei Xu](https://arxiv.org/search/cs?searchtype=author&query=Xu,+S), [Wenhao Xie](https://arxiv.org/search/cs?searchtype=author&query=Xie,+W), [Lingxiao Zhao](https://arxiv.org/search/cs?searchtype=author&query=Zhao,+L), [Pengcheng He](https://arxiv.org/search/cs?searchtype=author&query=He,+P)

------

> "Chain of Draft (CoD)"는 대규모 언어 모델(LLM)이 복잡한 추론 작업을 수행할 때, 장황한 단계별 과정을 거치는 기존의 "Chain-of-Thought (CoT)" 방식 대신, 인간처럼 간결하고 핵심적인 정보만 담은 초안(draft)을 생성하며 문제를 해결하는 새로운 패러다임을 제시합니다. 이를 통해 CoD는 CoT와 동등하거나 더 나은 정확도를 유지하면서도 토큰 사용량을 현저히 줄여(최소 7.6% 수준) 비용과 지연 시간을 크게 단축시킨다고 논문에 나와있습니다. 해당 논문에서의 목표는  LLM의 추론 과정에서 불필요한 장황함을 줄이고, 인간의 효율적인 사고방식을 모방하여 최소한의 정보로 정확한 결과를 도출함으로써, LLM의 실제 적용에서의 비용 효율성과 응답 속도를 개선하는 것을 목표로 합니다.
> 
- **Chain-of-Thought (CoT)**: LLM이 문제 해결을 위해 상세하고 단계적인 탐색 과정을 생성하도록 하는 기존의 프롬프팅 기법입니다. 정확하지만, 많은 계산 자원과 긴 응답 시간을 요구합니다.
- **Chain of Draft (CoD)**: 인간이 문제 해결 시 간결한 메모나 초안을 활용하는 방식에서 영감을 받아, LLM이 각 추론 단계에서 최소한의 정보만 담은 간결한 초안을 생성하도록 하는 해당 논문에서 제시하는 프롬프팅 전략입니다.

### 핵심 문제 (기존 방식의 문제점)

기존의 CoT와 같은 구조화된 추론 방식은 LLM의 문제 해결 능력을 크게 향상시켰지만, 최종 답변에 도달하기까지 상당한 양의 토큰을 사용하게 만듭니다. 이로 인해 비용에 민감하거나 빠른 응답이 필수적인 시나리오에서는 CoT 방식의 적용이 어렵습니다. 또한, 모델이 문제의 복잡도를 제대로 인지하지 못해 간단한 문제에도 과도하게 많은 생각을 하거나(overthinking) 불필요한 자원을 소모하는 경향이 있습니다. CoT의 장황함은 인간이 일반적으로 문제를 해결할 때 간결한 메모나 핵심 아이디어를 활용하는 효율적인 방식과 대조됩니다.

### 연구 목적 (CoD를 통한 해결)

1. 인간의 인지 과정에서 영감을 받은 간결한 추론 프롬프팅 전략인 "Chain of Draft (CoD)"를 제안
2. CoD가 기존의 CoT 방식에 비해 정확도를 희생하지 않으면서도 지연 시간과 비용을 현저히 줄일 수 있음을 실험적으로 검증
3. CoD가 LLM의 설계, 배포, 그리고 실제 사용성에 미치는 긍정적인 영향과 시사점을 논의

---

## 논문 주요 내용 요약

### Chain of Draft (CoD)의 작동 원리 및 특징

- **인간의 사고방식 모방**: CoD는 인간이 수학 문제를 풀거나, 글을 쓰거나, 코딩을 할 때 불필요한 설명을 생략하고 진행에 도움이 되는 핵심 정보만 간략히 적어두는 방식에서 착안했습니다.
- **간결한 중간 단계**: LLM은 CoT처럼 장황한 단계별 설명을 생성하는 대신, 각 단계에서 최소한의 정보만을 담은 간결하고 정보 밀도가 높은 초안(draft)을 생성합니다.
- **효율성 우선**: 이 접근 방식은 불필요한 장황함을 줄여 응답 지연 시간과 계산 비용을 낮추면서도, 정확도는 유지하거나 향상시키는 것을 목표로 합니다.
- **핵심 통찰력 집중**: LLM이 장황한 추론 과정의 부담 없이 해결책을 향해 나아가는 데 집중할 수 있도록 합니다.

### CoT 및 CoD 프롬프트 예시

**질문(Q):** Jason had 20 lollipops. He gave Denny some lollipops. Now Jason has 12 lollipops. How many lollipops did Jason give to Denny?

1. **Standard 프롬프팅 (예상 답변)**:
    - A: 8
2. **Chain-of-Thought (CoT) 프롬프팅 (예상 답변)**:
    - A: Let's think through this step by step:
    
    8 lollipops
        1. Initially, Jason had 20 lollipops.
        2. After giving some to Denny, Jason now has 12 lollipops.
        3. To find out how many lollipops Jason gave to Denny, we need to calculate the difference between the initial number of lollipops and the remaining number.
        4. We can set up a simple subtraction problem: Initial number of lollipops - Remaining number of lollipops = Lollipops given to Denny
        5. Putting in the numbers: 20 - 12 = Lollipops given to Denny
        6. Solving the subtraction: 20 - 12 = 8
        Therefore, Jason gave 8 lollipops to Denny.
3. **Chain-of-Draft (CoD) 프롬프팅 (예상 답변)**:
    - A: 20 - x = 12; x = 20 - 12 = 8. #### 8

**참고: 시스템 프롬프트 예시 (고객님 제공 내용 기반)**
실제 구현 시, 위와 같은 답변 스타일을 유도하기 위해 다음과 같은 시스템 프롬프트를 사용할 수 있습니다.

- **Standard 시스템 프롬프트:** "Answer the question directly. Do not return any preamble, explanation, or reasoning."
- **CoT 시스템 프롬프트:** "Think step by step to answer the following question. Return the answer at the end of the response after a separator ####."
- **CoD 시스템 프롬프트:** "Think step by step, but only keep a minimum draft for each thinking step, with 5 words at most. Return the answer at the end of the response after a separator ####."

### CoD의 효과 검증 및 결과

- **다양한 벤치마크 테스트**: CoD의 효과를 검증하기 위해 산술 추론(GSM8K), 상식 추론, 기호 추론 등 다양한 다단계 추론 능력을 요구하는 벤치마크에서 실험을 수행했습니다.
- **정확도 유지 및 향상**: 실험 결과, CoD는 표준적인 CoT 방식과 비교했을 때 정확도를 유지하거나 오히려 향상시키는 것으로 나타났습니다.
- **토큰 사용량 및 지연 시간 대폭 감소**: 가장 주목할 만한 결과는 CoD가 CoT에 비해 현저히 적은 토큰(최소 7.6% 수준)을 사용하며, 이는 직접적으로 비용 절감 및 지연 시간 감소로 이어집니다. (논문의 Figure 1은 Claude 3.5 Sonnet 모델에서 Standard, CoT, CoD 전략 간의 정확도와 토큰 사용량을 비교하여 CoD의 효율성을 시각적으로 보여줍니다.)

### CoD의 주요 이점 및 시사점

- **지연 시간 감소**: CoD는 추론 단계를 간결하게 만들어 응답 생성 속도를 크게 향상시키므로, 실시간 상호작용이 중요한 애플리케이션에 매우 유리합니다.
- **비용 절감**: CoD는 few-shot 프롬프팅에 필요한 입력 토큰 수를 줄이고 출력 토큰 길이를 단축시켜 계산 비용을 직접적으로 낮춥니다. 이는 대규모 LLM 배포나 예산 제약이 있는 애플리케이션에 특히 매력적입니다.
- **실용성 증대**: LLM 추론에 있어 효과적인 결과가 반드시 장황한 출력을 요구하지 않는다는 것을 보여주며, 최소한의 표현으로도 추론의 깊이를 유지할 수 있는 대안적 접근법을 제시합니다.
- **향후 LLM 개발 방향 제시**: CoD의 간결한 추론 원리는 해석 가능성과 효율성을 유지하면서 추론 모델을 개선하기 위한 새로운 전략(예: 간결한 추론 데이터로 모델 훈련)에 영감을 줄 수 있습니다.

## 🤔 Q&A 형태의 이해 및 마무리

- **의문 1:** CoT와 CoD가 **프롬프트 자체를 대체하는 것**인지, 아니면 기존 프롬프트에 **어떤 추가적인 지시를 더하는 것**인지?
    - **해소 답변:** CoT와 CoD는 사용자의 질문 자체를 대체하는 것이 아니라, LLM이 답변을 생성하는 과정을 특정한 방식으로 유도하는 '프롬프트 전략' 또는 '기법'입니다. 이는 일반적으로 프롬프트 내에 **추가적인 지침을 포함**시키는 형태로 작동합니다.
- **의문 2:** CoT와 CoD를 LLM 서비스에 적용할 때, 실제로 **어떤 방식**으로 프롬프트를 구성해야 하는지 구체적인 예시가 필요한지? 특히 **시스템 프롬프트**와 **사용자 질문** 외에 다른 요소가 필요한지?
    - **해소 답변:** CoT와 CoD를 효과적으로 적용하는 주된 방법은 **Few-shot 프롬프팅**입니다. 이는 다음과 같은 요소로 구성됩니다:
        1. 모델의 전반적인 동작 방식을 설정하는 **시스템 프롬프트** (예: 단계별 생각 지시).
        2. CoT 또는 CoD 스타일의 **추론 과정과 최종 답변을 보여주는 문제-해결 예시들** (Few-shot examples).
        3. 사용자의 **실제 질문**.
    - 이 구조를 통해 모델은 예시에서 제시된 추론 스타일을 모방하여 사용자의 질문에 대한 답변을 생성합니다 .
- **의문 3:** Few-shot 방식 외에 CoT나 CoD를 적용하는 다른 방식(예: Zero-shot)도 있는지, 그리고 그 방식의 효과는 어떠한지?
    - **해소 답변:** Few-shot 예시 없이 **Zero-shot 방식**으로도 CoT 또는 CoD 시스템 프롬프트만 사용하여 모델의 추론을 유도하는 것이 기술적으로 가능합니다. 그러나  Few-shot 예시가 없는 Zero-shot 설정에서는 특히 CoD의 효과(정확도 및 토큰 절감)가 크게 감소하는 한계가 있습니다. 이는 모델 학습 데이터에 CoD와 같은 간결한 추론 패턴이 부족하기 때문일 수 있으며, Few-shot 예시가 모델이 CoD 스타일을 생성하도록 안내하는 데 중요함을 시사합니다. CoT는 Zero-shot에서도 효과를 보일 수 있지만, Few-shot이 일반적인 접근 방식입니다.
- **의문 4:** CoT와 CoD의 **추론 과정 표현 방식**이 구체적으로 어떻게 다른지, 그리고 CoD의 '최소한의 초안'이 정확히 무엇을 의미하는지?
    - **해소 답변:**
        - **Standard 프롬프팅:** 추론 과정 없이 최종 답변만 직접 반환합니다.
        - **CoT:** 문제를 해결하기 위한 **상세하고 장황한 단계별 추론 과정**을 생성합니다. 투명하지만 토큰 사용량이 많고 지연 시간이 증가합니다.
        - **CoD:** CoT처럼 단계별로 생각하지만, 각 단계의 추론 과정을 **핵심적인 정보만을 담은 '최소한의 초안(minimum draft)' 형태로 간결하게** 표현합니다. 논문에서는 이를 "minimalistic yet informative intermediate reasoning outputs" 또는 "concise, dense-information outputs"로 설명하며, 간결성을 유도하기 위해 **최대 5단어**라는 가이드라인을 사용할 수 있습니다. CoD는 인간이 문제를 풀 때 핵심 내용을 짧게 메모하는 방식에서 영감을 받았습니다.
