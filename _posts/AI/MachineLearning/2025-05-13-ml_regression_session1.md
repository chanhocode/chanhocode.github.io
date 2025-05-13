---
title: "[ML] 머신러닝에서의 회귀 분석: 단순, 다중, 다항 회귀"
writer: chanho
date: 2025-05-13 13:00:00 +0900
categories: [AI, MachineLearning]
tags: [ai, ml]
---

> 회귀(Regression) 분석은 머신러닝의 지도 학습(Supervised Learning) 패러다임에 속하는 핵심적인 기법 중 하나이다. 지도 학습은 시스템이 입력 데이터(독립 변수)와 그에 상응하는 정답(종속 변수) 쌍으로 구성된 데이터셋을 통해 학습하는 방식이며, 회귀 분석의 목표는 하나 이상의 독립 변수를 사용하여 연속적인 값의 결과(종속 변수)를 예측하는 것이다. 예시를 들자면,  집의 크기나 방 개수 등의 피처(독립 변수)로 집값(연속 값)을 예측하는 것이 회귀 문제에 해당한다. 분류 문제가 범주를 맞히는 것이라면, 회귀는 숫자 값을 예측하는 감독 학습의 한 종류이다. 회귀 분석에는 여러 유형이 있는데, 오늘은 머신러닝에서 가장 기본이 되는 단순 선형회귀, 다중 선형회귀, 다항식 회귀의 개념과 구현에 대해 정리하고자 한다.
> 

## Backgrounds

### **독립 변수와 종속 변수**

- **독립 변수 (X, Features, 설명 변수):** 모델의 입력 값으로 사용되며, 결과 값인 종속 변수에 영향을 미치는 원인으로 간주되는 변수들이다. 예를 들어, 특정 지역의 주택 가격을 예측하는 모델에서 '주택의 면적', '방의 개수', '건축 연도', '지하철 역 과의 거리' 등은 독립 변수가 될 수 있다.
- **종속 변수 (Y, Target, 반응 변수):** 모델을 통해 최종적으로 예측하고자 하는 목표가 되는 변수이다. 독립 변수들의 값 변화에 따라 그 값이 결정되는 결과로 해석된다. 앞선 주택 가격 예측 예시에서는 '주택 가격' 자체가 종속 변수에 해당한다.

### **필요한 최소 수학적 지식**

**1. 최소 제곱법 (Least Squares Method)**

> 선형 회귀 모델링의 핵심 과제는 주어진 데이터 포인트들을 가장 잘 대표하는 **회귀선(Regression Line)** 또는 고차원 공간에서의 초평면(Hyperplane)을 찾는 것이다. 이는 곧 모델의 파라미터인 **회귀 계수**, 즉 기울기(slope)와 절편(intercept)의 최적값을 결정하는 문제와 동일하다 . 최소 제곱법의 기본 원리는 실제 관측된 종속 변수 값(Y)과 회귀 모델이 예측한 값(ŷ) 사이의 차이, 즉 **오차(Error)** 또는 잔차(Residual)의 제곱합(SSR)을 **최소화**하는 회귀 계수를 찾는 것이다.
> 

수학적으로 표현하면, 단순 선형 회귀($\hat{y}=b0+b1x$)의 경우 최소 제곱법은 다음 식을 최소화하는 b0와 b1을 찾는다:

$$
\text{Minimize } \sum_{i=1}^{N} (y_i - \hat{y}i)^2 = \text{Minimize } \sum{i=1}^{N} (y_i - (b_0 + b_1x_i))^2 
$$

여기서 N은 데이터 포인트의 개수, yi는 i번째 실제 관측값, ŷi는 i번째 예측값이다. 잔차의 '제곱합'을 최소화하는 이유는 다음과 같다.

- **부호 상쇄 방지:** 실제 값보다 예측값이 큰 경우 잔차는 음수, 작은 경우 양수가 된다. 단순히 잔차의 합을 최소화하려고 하면, 양수 잔차와 음수 잔차가 서로 상쇄되어 전체 오차가 작게 평가될 수 있다. 제곱을 함으로써 모든 잔차를 양수로 만들어 이러한 문제를 방지한다.
- **큰 오차에 대한 가중치 부여:** 제곱 연산은 절대값이 큰 잔차에 더 큰 값을 부여한다. 즉, 모델이 실제 값에서 크게 벗어나는 예측을 할 경우, 해당 오차에 더 큰 패널티를 주어 모델이 이러한 큰 오차를 줄이도록 유도한다.

**2. 결정 계수**

> 결정계수 $R^2$은 회귀 모델의 성능을 평가하는 지표 중 하나로, 모델이 종속변수의 분산을 얼마나 설명하는지를 나타내는 값이다. $R^2$ 값은 0에서 1 사이를 가지며, 1에 가까울수록 모델의 설명력이 높고 데이터에 잘 맞는다는 뜻이다. 예를 들어 $R^2 = 0.9$ 라면 해당 회귀식이 데이터 변동의 90%를 설명한다는 의미이다. 참고로, 독립 변수가 여러 개인 경우 과적합을 보정한 조정된 결정계수를 사용하기도 한다.
> 

**3. P값(P-value) 이해하기**

> 회귀 분석에서 산출되는 유의확률로, 추정된 회귀 계수가 통계적으로 유의한지 판단하기 위한 값이다. P값은 회귀 모델의 결과를 해석하고 특정 독립 변수가 실제로 종속 변수에 의미 있는 영향을 미치는지 판단하는 정보를 제공한다. 각 회귀 계수에 대해 귀무가설 $H_0: \text{계수}=0$을 세우고 t-검정을 수행한 결과가 p값이다. p값은 주어진 t-통계량에서 귀무가설을 기각할 확률이며, 일반적으로 0.05보다 작으면 해당 회귀계수가 유의미하다고 판단한다. 즉, p값이 0.03이라면 해당 변수의 영향력이 통계적으로 의미있어 95% 신뢰 수준에서 실제로 0이 아닐 가능성이 높다는 뜻이다. 다만 머신러닝에서는 주로 예측 성능에 초점을 맞추고 p값은 많이 다루지 않지만, 통계적 해석에서는 중요한 지표이다.
> 

## **단순 선형 회귀 (Simple Linear Regression)**

> 단순 선형회귀는 **독립 변수 X가 1개**인 가장 기본적인 회귀 모델이다. 하나의 입력 X와 출력 Y사이의 **선형 관계**를 가정한다. 즉 X 값이 변함에 따라 Y값도 일정한 기울기로 증가하거나 감소하는 직선 관계를 가정한다.
> 

$$
y=wx+by = w x + by=wx+b
$$

여기서 w는 회귀 계수(coefficient, 기울기)에 해당하고, b는 상수항(절편, bias)이다. 이 모델은 X와 Y의 관계를 2차원 평면상의 **직선 하나**로 나타낸 것이며, 최적의 직선을 찾기 위해 앞서 설명한 최소제곱법을 적용한다. 단순 선형회귀의 예로, 공부 시간(x)에 따른 시험 점수(y) 예측, 몸무게(y)를 키(x)로 예측하는 모델 등을 생각할 수 있다.

💡 **참조 공식**

- **모집단 회귀 함수(Population Regression Function, PRF)**

우리가 궁극적으로 알고 싶어하는, 모집단 전체에서의 X와 Y의 평균적인 관계를 나타내는 이상적인 선이다.

$$
E(Y∣X)=β0+β1X
$$

 여기서 E(Y∣X)는 주어진 X 값에 대한 Y의 조건부 기댓값(평균)을 의미하며, β0는 모집단의 Y절편, β1은 모집단의 기울기(회귀 계수)이다.

- **표본 회귀 함수(Sample Regression Function, SRF)**

실제로는 모집단 전체를 조사할 수 없으므로, 표본 데이터를 사용하여 추정한 회귀선이다.

$$
\hat{y}^=b0+b1x
$$

- **개별 관측치**

실제 개별 관측치 yi는 예측값 ŷi와 오차항(error term) ei의 합으로 표현된다. 오차항은 모델이 설명하지 못하는 무작위적인 변동을 나타낸다.

$$
yi=β0+β1xi+ei
$$

여기서 ei는 i번째 관측치의 잔차이다.

💡 **주요 가정**

단순 선형 회귀 모델의 추정 결과(회귀 계수, P값 등)가 통계적으로 의미 있고 신뢰성을 가지려면 다음과 같은 가정들이 충족되어야 한다.

1. **선형성(Linearity):** 독립 변수 X와 종속 변수 Y의 조건부 기댓값 사이에는 선형 관계가 존재한다 (E(Y∣X)=β0+β1X). 즉, 모델이 파라미터(β0,β1)에 대해 선형이어야 한다.
2. **오차항의 평균은 0 (Zero Conditional Mean of Error):** 주어진 X 값에 대해 오차항 e의 기댓값(평균)은 0이다 (E(e∣X)=0). 이는 모델이 평균적으로는 Y 값을 정확히 예측하며, 오차가 특정 방향으로 치우치지 않음을 의미한다.
3. **등분산성(Homoscedasticity):** 모든 X 값에 대해 오차항 e의 분산은 σ2으로 일정하다. 즉, X 값의 크기에 관계없이 데이터 포인트들이 회귀선 주위에 흩어진 정도가 비슷해야 한다. 만약 X 값에 따라 오차의 분산이 달라진다면 이분산성 문제가 발생한다.
4. **오차항의 독립성(Independence of Errors):** 서로 다른 관측치에 대한 오차항들은 서로 독립적이다. 즉, 한 관측치의 오차가 다른 관측치의 오차에 영향을 주지 않아야 한다. 시계열 데이터 등에서는 이 가정이 위배되기 쉽다.
5. **독립 변수의 변동성 및 비확률성:** 독립 변수 X의 값들은 모두 동일하지 않아야 하며(즉, X의 분산이 0보다 커야 함), 고정된 값으로 간주하거나, 확률 변수이더라도 오차항 e와는 독립적이어야 한다.
6. **오차항의 정규성(Normality of Errors):** 오차항 e는 평균이 0이고 분산이 σ2인 정규분포를 따른다 . 주로 회귀 계수의 가설 검정(P값 계산)이나 신뢰 구간 추정에 필요하다. 표본 크기가 충분히 크면 중심 극한 정리에 의해 이 가정이 다소 완화될 수 있다.

### **구현**

**Scikit-learn 구현**

```python
import numpy as np
from sklearn.linear_model import LinearRegression
import matplotlib.pyplot as plt

# 1차원 샘플 데이터 생성 (X: 입력, y: 출력)
X = np.array([1, 2, 3, 4, 5, 6, 7, 8]).reshape(-1, 1)
y = 2 * X.flatten() + 1 + np.random.normal(0, 1, X.shape[0])

# 단순 선형회귀 모델 학습
model = LinearRegression()
model.fit(X, y)

# 학습된 회귀 계수와 절편 출력
print("학습된 계수:", model.coef_[0], "절편:", model.intercept_)

# 실제 데이터 점과 예측 직선 시각화
y_pred = model.predict(X)
plt.scatter(X, y, color='blue', marker='x', label='데이터 점')      # 산점도
plt.plot(X, y_pred, color='orange', label='회귀 직선')             # 회귀 직선
plt.xlabel('X')
plt.ylabel('y')
plt.legend()
plt.title('단순 선형회귀 예제')
plt.show()
```

**PyTorch 구현**

```python
import torch
import torch.nn as nn
import torch.optim as optim

# PyTorch용 텐서로 데이터 준비
X_tensor = torch.tensor(X, dtype=torch.float32)
y_tensor = torch.tensor(y, dtype=torch.float32).unsqueeze(1)  # shape 맞추기 (n,1)

# 단순 선형 회귀 모델 정의 (초기 w,b는 무작위 초기화)
model_torch = nn.Linear(1, 1)  
optimizer = optim.SGD(model_torch.parameters(), lr=0.01)
loss_fn = nn.MSELoss()

# 모델 학습 (경사하강법으로 손실 최적화)
for epoch in range(1000):
    optimizer.zero_grad()
    y_pred_t = model_torch(X_tensor)        # 예측
    loss = loss_fn(y_pred_t, y_tensor)     # MSE 손실 계산
    loss.backward()                       
    optimizer.step()                       # 파라미터 업데이트

# 학습된 파라미터 확인
w_learned = model_torch.weight.item()
b_learned = model_torch.bias.item()
print("학습된 계수:", w_learned, "절편:", b_learned)
```

## **다중 선형 회귀 (Multiple Linear Regression)**

> 다중 선형 회귀(Multiple Linear Regression)는 두 개 이상의 독립 변수(X₁, X₂,..., Xₚ)를 사용하여 하나의 종속 변수(Y)를 예측하는 회귀 모델이다. 현실의 많은 문제는 단일 요인이 아닌 여러 요인이 복합적으로 결과에 영향을 미치므로, 단순 선형 회귀보다 더 현실적인 상황을 모델링하는 데 적합하다. 이 모델은 각 독립 변수가 종속 변수에 미치는 개별적인 영향력을 다른 변수들의 영향을 통제한 상태에서 평가하려고 시도한다.
> 

$$
y = w_1 x_1 + w_2 x_2 + b
$$

더 일반적으로 피처가 n개라면, $y = w_1 x_1 + w_2 x_2 + \cdots + w_n x_n + b$ 형태로 모든 입력의 선형 조합으로 표현된다. 단순 회귀와 마찬가지로 최소제곱법으로 오차 제곱합을 최소화하는 w_i와 b를 찾는다. 다만 독립 변수 개수가 늘어나므로, 데이터의 차원과 필요한 데이터 양도 증가한다. 피처들 간에 강한 상관관계가 있는 경우 다중공선성 문제로 회귀계수 추정이 불안정해질 수 있고, 불필요한 피처는 과적합을 일으킬 수 있다. 따라서 다중 회귀에서는 “feature selection”이나 “regularization” 기법을 활용하여 모델 복잡도를 관리하기도 한다.

다중 선형 회귀 역시 단순 선형 회귀와 동일한 기본 가정들(선형성, 오차항 평균 0, 등분산성, 오차항 독립성, 오차항 정규성)을 따른다. 그러나 여러 개의 독립 변수를 동시에 고려하면서 새로운 문제, 즉 **다중공선성(Multicollinearity)** 문제가 발생할 수 있다.

**다중공선성 (Multicollinearity)**

: 회귀 모델에 포함된 독립 변수들 사이에 강한 선형 관계가 존재하는 현상을 말한다. 예를 들어, '주택 크기(제곱미터)'와 '주택 크기(평)'는 거의 완벽한 선형 관계를 가지므로 함께 모델에 포함하면 다중공선성 문제가 발생한다. 또는, 한 독립 변수가 다른 여러 독립 변수들의 선형 조합으로 거의 완벽하게 표현될 때도 발생한다.

- **문제점**
    - **회귀 계수 추정치의 불안정성:** 개별 회귀 계수(bj)의 표준 오차(Standard Error)가 매우 커진다. 이는 추정된 계수 값의 신뢰도를 떨어뜨린다.
    - **해석의 어려움:** 계수의 크기나 부호가 예상과 다르게 나타나거나, 데이터를 조금만 변경해도 계수 값이 크게 변동될 수 있다. 특정 독립 변수가 종속 변수에 미치는 순수한 영향력을 분리하여 해석하기 어려워진다.
    - **통계적 유의성 오판:** 실제로는 종속 변수와 관련 있는 변수임에도 불구하고, 표준 오차가 커져 P값이 유의 수준보다 크게 나올 수 있다 (즉, 유의하지 않다고 잘못 판단할 수 있다).
- **진단**
    - **상관 계수 행렬(Correlation Matrix):** 독립 변수들 간의 pairwise 상관 계수를 확인한다. 높은 상관 계수(예: 절대값 0.8 이상)는 다중공선성의 가능성을 시사하지만, 이것만으로는 충분하지 않다. 여러 변수가 복합적으로 선형 관계를 형성하는 경우는 pairwise 상관 계수만으로는 탐지하기 어렵다.
    - **분산 팽창 요인(Variance Inflation Factor, VIF):** 각 독립 변수 Xj에 대해 다른 모든 독립 변수들을 이용하여 Xj를 예측하는 회귀 모델을 만든다. 이 모델의 결정 계수(Rj2)를 이용하여 VIF를 계산한다:
    VIFj=1−Rj21Rj2가 1에 가까울수록 (즉, Xj가 다른 독립 변수들로 잘 설명될수록) VIF 값은 무한히 커진다. 일반적으로 **VIF 값이 5 또는 10을 초과**하면 해당 변수에 다중공선성 문제가 심각하다고 판단한다.
- **해결**
    - **변수 제거:** VIF가 높은 변수 중 이론적으로 덜 중요하거나 다른 변수와 의미가 중복되는 변수를 제거한다. 단, 중요한 변수를 제거하면 모델의 편향(bias)이 커질 수 있으므로 신중해야 한다.
    - **변수 결합:** 강한 상관 관계를 보이는 변수들을 합쳐 새로운 변수를 만들거나, 주성분 분석(PCA) 등을 이용해 변수들을 변환하여 사용한다.
    - **Ridge 또는 Lasso 회귀 사용:** 이들 규제 선형 모델은 다중공선성 문제에 비교적 덜 민감하며, 계수 추정치를 안정화하는 데 도움이 된다.

### **구현**

**Scikit-learn 구현**

```python
import numpy as np
from sklearn.linear_model import LinearRegression

# 2차원 샘플 데이터 생성 (X1, X2 두 피처)
X = np.array([[8, 8],
              [6, 2],
              [8, 7],
              [2, 1],
              [5, 4],
              [4, 5],
              [7, 3],
              [6, 4]])
y = 3 * X[:, 0] - 2 * X[:, 1] + 5 + np.random.normal(0, 1, X.shape[0])  # y = 3x1 - 2x2 + 5 + 노이즈

# 다중 선형회귀 모델 학습
model = LinearRegression()
model.fit(X, y)

# 학습된 회귀 계수 및 절편 출력
print("학습된 계수:", model.coef_, "절편:", model.intercept_)

# 새로운 입력에 대한 예측
X_new = np.array([[4, 3]])
y_pred = model.predict(X_new)
print("새로운 입력", X_new.tolist(), "예측된 출력:", float(y_pred))

```

**PyTorch 구현**

```python
import torch
import torch.nn as nn
import torch.optim as optim

# 데이터 텐서화
X_tensor = torch.tensor(X, dtype=torch.float32)
y_tensor = torch.tensor(y, dtype=torch.float32).unsqueeze(1)

# 다중 선형회귀 모델 정의 (입력 차원 2 -> 출력 차원 1)
model_torch = nn.Linear(2, 1)
optimizer = optim.SGD(model_torch.parameters(), lr=0.01)
loss_fn = nn.MSELoss()

# 모델 학습
for epoch in range(2000):
    optimizer.zero_grad()
    pred = model_torch(X_tensor)
    loss = loss_fn(pred, y_tensor)
    loss.backward()
    optimizer.step()

# 학습된 파라미터 출력
w1, w2 = model_torch.weight.data[0]
b = model_torch.bias.item()
print("학습된 계수:", float(w1), float(w2), "절편:", b)

```

## **다항식 회귀 (Polynomial Regression)**

> 다항식 회귀는 비선형 곡선 형태의 관계를 모델링하기 위해 입력 변수의 다항(Term)을 특징으로 추가하는 회귀 기법 이다. 해당 기법은 비선형 관계를 모델링하지만 그 기반은 선형 회귀 모델에 두고 있다.
> 

$$
y = w_0 + w_1 x + w_2 x^2 + \dots + w_d x^d
$$

비선형 관계 모델링의 핵심 아이디어는 기존 독립 변수 X를 거듭제곱하여 만든 새로운 항들(X2,X3,...)을 추가적인 독립 변수로 간주하고, 이들을 포함하여 다중 선형 회귀를 수행하는 것이다. 

**Degree 선택 및 Overfitting**

다항식 회귀에서 가장 중요한 결정 중 하나는 다항식의 최고 차수(degree)를 얼마로 설정할 것인가이다. 차수 선택은 모델의 성능과 일반화 능력에 직접적인 영향을 미친다.

- **낮은 차수 (예: 1차 - 단순 선형 회귀):** 모델이 너무 단순하여 데이터의 복잡한 비선형 패턴을 충분히 잡아내지 못할 수 있다. 이 경우 과소적합(Underfitting)이 발생하여 학습 데이터와 테스트 데이터 모두에서 성능이 낮게 나타난다.
- **높은 차수 (예: 10차):** 모델이 매우 유연해져서 학습 데이터의 거의 모든 점을 통과하는 복잡한 곡선을 만들 수 있다. 하지만 이는 데이터의 실제 기본 패턴뿐만 아니라 우연한 노이즈까지 과도하게 학습하는 과적합(Overfitting)으로 이어질 가능성이 높다. 과적합된 모델은 학습 데이터에서는 오차가 매우 작지만, 이전에 보지 못한 새로운 데이터(테스트 데이터)에 대해서는 예측 성능이 현저히 떨어지는 일반화 실패 문제를 보인다.

**과적합을 방지하고 적절한 차수를 선택하기 위한 방법은 아래와 같다.**

- **교차 검증(Cross-Validation):** 데이터를 여러 부분으로 나누어 일부는 학습에, 일부는 검증에 사용하는 과정을 반복하여 다양한 차수에 대한 모델의 일반화 성능을 평가하고 최적의 차수를 선택한다.
- **학습 곡선(Learning Curves):** 학습 데이터 크기를 늘려가면서 학습 데이터와 검증 데이터에 대한 모델 성능(오차) 변화를 관찰한다. 과적합된 모델은 두 곡선 사이에 큰 간격이 나타난다.
- **정보 기준(Information Criteria):** AIC(Akaike Information Criterion), BIC(Bayesian Information Criterion) 등 모델 복잡도에 패널티를 부여하는 지표를 사용하여 차수를 선택한다.

다항식 회귀는 선형 회귀의 제약을 벗어나 비선형 데이터 패턴을 모델링할 수 있는 유용한 도구이다. 그러나 모델의 유연성(복잡도)을 높이는 대가로 과적합의 위험이 커진다. 따라서 적절한 차수를 신중하게 선택하거나 규제 기법을 활용하여 모델의 일반화 성능을 확보하는 것이 매우 중요하다. 이는 머신러닝 모델 개발 전반에서 나타나는 근본적인 딜레마인 편향-분산 트레이드오프(Bias-Variance Tradeoff)를 잘 보여주는 사례이다. 차수를 높이면 편향(Bias, 모델이 실제 관계를 얼마나 잘 표현하는지)은 줄어들지만 분산(Variance, 데이터의 노이즈에 모델이 얼마나 민감하게 반응하는지)이 커지고, 차수를 낮추면 그 반대가 된다. 좋은 모델은 이 둘 사이의 적절한 균형점을 찾는 것이다.

### **구현**

**Scikit-learn 구현**

```python
import numpy as np
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import PolynomialFeatures
import matplotlib.pyplot as plt

# 1차원 샘플 데이터 생성 (비선형 관계: 2차 함수 형태)
X = np.array([-3, -2, -1, 0, 1, 2, 3]).reshape(-1, 1)
y = 0.5 * X.flatten()**2 + 1 + np.random.normal(0, 0.5, X.shape[0])  # y = 0.5*x^2 + 1 + 약간의 노이즈

# 다항 특성 변환 (2차항 추가)
poly = PolynomialFeatures(degree=2, include_bias=True)
X_poly = poly.fit_transform(X)  # X_poly는 [1, x, x^2] 컬럼으로 구성됨

# 선형회귀 모델 학습 (다항 회귀)
model = LinearRegression()
model.fit(X_poly, y)

# 학습된 회귀계수 출력
print("회귀 계수:", model.coef_, "절편:", model.intercept_)

# 예측 곡선 그리기 위해 연속된 x 값에 대한 예측 계산
X_range = np.linspace(X.min(), X.max(), 100).reshape(-1, 1)
X_range_poly = poly.transform(X_range)
y_range_pred = model.predict(X_range_poly)

# 실제 데이터와 회귀 곡선 시각화
plt.scatter(X, y, color='blue', label='데이터 점')
plt.plot(X_range, y_range_pred, color='orange', label='회귀 곡선')
plt.xlabel('X')
plt.ylabel('y')
plt.legend()
plt.title('다항 회귀 예제')
plt.show()

```

**PyTorch 구현**

```python
import torch
import torch.nn as nn
import torch.optim as optim

# 데이터 텐서화 (및 2차항 추가)
X_tensor = torch.tensor(X, dtype=torch.float32)
X_poly_tensor = torch.cat([X_tensor, X_tensor**2], dim=1)  # (n,2) 모양 -> [x, x^2]
y_tensor = torch.tensor(y, dtype=torch.float32).unsqueeze(1)

# 다항 회귀를 위한 선형 모델 정의 (입력 차원 2 -> 출력 차원 1)
model_torch = nn.Linear(2, 1)
optimizer = optim.SGD(model_torch.parameters(), lr=0.01)
loss_fn = nn.MSELoss()

# 모델 학습
for epoch in range(3000):
    optimizer.zero_grad()
    pred = model_torch(X_poly_tensor)
    loss = loss_fn(pred, y_tensor)
    loss.backward()
    optimizer.step()

# 학습된 파라미터 확인
w1, w2 = model_torch.weight.data[0]
b = model_torch.bias.item()
print("학습된 계수:", float(w1), float(w2), "절편:", b)
```

---

## **마무리**

### **회귀 모델 비교 요약**

| **특징** | **단순 선형 회귀** | **다중 선형 회귀** | **다항식 회귀** |
| --- | --- | --- | --- |
| **독립 변수 개수** | 1개 | 2개 이상 | 1개 이상 (주로 1개에서 파생) |
| **모델링 관계** | 선형 | 선형 | 비선형 (독립 변수의 다항식 사용) |
| **주요 가정/고려사항** | 선형성, 독립성, 등분산성, 정규성 | 단순 선형 회귀 가정 + 다중공선성 확인 필요 | 선형 회귀 가정 기반 + 차수 선택, 과적합 주의 |
| **장점** | 구현/해석 용이, 계산 빠름 | 여러 변수 동시 고려, 현실 설명력 높을 수 있음 | 비선형 관계 모델링 가능 |
| **단점** | 비선형 관계 모델링 불가, 과소적합 가능성 | 다중공선성 문제, 해석 복잡성 증가 | 과적합 위험 높음, 차수 선택 어려움, 해석 어려움 |

각 모델은 저마다의 강점과 약점을 지니고 있으며, 특정 문제 상황과 데이터의 특성에 더 적합할 수 있다. 따라서 성공적인 회귀 분석을 위해서는 해결하려는 문제의 본질, 사용 가능한 데이터의 특성, 그리고 최종 분석 목표(예측 정확성 vs. 모델 해석 가능성)를 종합적으로 고려하여 가장 적합한 기법을 선택하고, 선택된 모델의 가정을 검토하며 성능을 체계적으로 평가하고 개선해 나가는 반복적인 과정이 필요하다.
