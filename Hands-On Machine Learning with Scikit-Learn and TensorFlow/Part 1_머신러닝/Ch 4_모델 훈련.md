# Ch 4. 모델 훈련

- 그 동안 내부 작동 원리는 모르고 많은 일을 처리
- 어떻게 작동하는지 이해하고 있으면 적절한 모델, 올바른 훈련 알고리즘, 좋은 하이퍼파라미터 빠르게 찾기 등이 가능.
- 디버깅이나 에러 분석에도 도움.
- 신경망 이해, 구축, 훈련에 필수



- 가장 간단한 모델 중 하나인 선현 회구부터 시작. 모델 훈련에 두가지 방법이 있음.
  - (1) 직접 계산할 수 있는 공식을 사용해서 훈련 세트에 가장 잘 맞는 모델 파라미터를 해석적으로 구함.
  - (2) **경사 하강법(GD)** 이라 불리는 반복적인 최적화 방식을 사용.
    - 모델 파라미터를 조금씩 바꾸면서 비용함수를 훈련 세트에 대해 최소화.
    - 경사하강법의 변종으로, **배치 경사하강법, 미니배치 경사하강법, 확률적 stochastic 경사 하강법(SGD)** 이 있음
- 비선형 데이터셋에 훈련시킬 수 있는 모델은 **다항 회귀**
  - **선형 회귀** 보다 파라미터가 많아서 과대적합되기 쉬운 문제가 있음.
    - 따라서, **학습 곡선 learning curve**를 사용해 모델이 과대적합되었는지 감지한 후, 과대적합을 감소시킬 수 있는 규제 기법을 사용해야 함.
- 분류 작업에 널리 사용하는 로지스틱 회귀와 소프트맥스 회귀도 살펴 볼 것.







## 4.1 선형 회귀

- 일반적으로 선형 모델은 입력 특성의 가중치 합 + **편향 Bias** 해서 예측을 만듦.
- 벡터 형태로 간단하게 표기
  - 종종 벡터를 **열 벡터 column vector**로 나타내는데, 이 때 **전치 transpose** 시켜 사용.
    - 예측 결과는 같지만 스칼라 값이 아닌 하나의 원소를 가진 행렬이 만들어짐.



- 모델을 훈련시킨다는 것은 모델이 훈련 세트에 가장 잘 맞도록 모델 파라미터를 설정한다는 것.
  - 모델이 훈련 데이터에 얼마나 잘 들어맞는지 측정 (회귀에서는 RMSE를 가장 널리 사용)
    - 실제로는 RMSE 보다 MSE를 최소화하는 것이 같은 결과를 내면서 더 간단함.







### 4.1.1 정규방정식

- **정규방정식 Normal equation** : 비용 함수를 최소화하는 theta 값을 찾기 위한 **해석적인 방법**



- **유사역행렬 Pseudoinverse** (정확히는, 무어-펜로즈 역행렬)
  - 유사역행렬 자체는 **특잇값 분해, Singular Value Decomposition (SVD)** 라 부르는 표준 행렬 분해기법을 사용해 계산.
    - 정규방정식을 계산하는 것보다 이 방식이 훨씬 효율적







### 4.1.2 계산 복잡도

- 정규방정식의 **계산 복잡도, Computational complexivity** 는 일반적으로 O(n^2.4)에서 O(n^3) 사이.
  - 특성 수가 2배로 늘어나면 계산 시간이 5.3에서 8배 증가.
    - 정규방정식과 SVD 모두 특성 수가 많아지면 매우 느려지지만, 훈련 세트의 샘플 수에 대해서는 선형적으로 O(m) 증가하므로 메모리 공간이 허락된다면 큰 훈련 세트도 효율적으로 처리할 수 있음.



- 다음 방법은 특성이 매우 많고 훈련 샘플이 너무 많에 메모리에 모두 담을 수 없을 때 적합.







## 4.2 경사 하강법

- 여러 종류의 문제에서 최적의 해법을 찾을 수 있는 일반적인 최적화 알고리즘
- 경사 하강법의 기본 아이디어는 비용 함수를 최소화하기 위해 반복해서 파라미터를 조정해가는 것.
- 그래이디언트가 0이 되면 최솟값에 도달한 것.



- **무작위 초기화 random initialization**으로 시작, theta를 임의의 값으로 시작해서, 그래이디언트를 계산, 비용 함수가 감소하는 방향으로 진행하여, 알고지름이 최솟값에 수렴할 때 까지 점진적으로 향상.
  - 경사 하강법에서 중요한 파라미터는 스텝(반복적인 학습 알고리즘에서 학습의 각 단계)의 크기
    - **학습률 learning rate** 하이퍼 파라미터로 결정
      - 학습률이 너무 작으면 알고리즘이 수렴하기 위해 반복을 많이 진행해야 함.
      - 학습률이 너무 크면 골짜기를 가로질러 반대편으로 건너뛰어, 이전보다 더 높은 곳으로 올라가게 될지도 모름. 이러면 값이 발산하여 적절한 해법을 찾지 못하게 함.



- 모든 비용 함수가 매끈하지 않고, 최솟값으로 수렴하기 매우 어려운 경우가 있음. 이로 인해 두 가지 문제점이 생김.
  - (1) **전역 최솟값 global minimum** 보다 덜 좋은 **지역 최솟값 local minimum** 에 수렴
  - (2) 기울기가 0 이지만 최솟값이 아닌 구간, 평지에서 학습이 멈춰버림
- 선형 회귀에 사용하는 MSE 비용 함수는 볼록 함수, 지역 최솟값이 없고 하나의 전역 최솟값만 존재하며 연속된 함수이고 기울기가 갑자기 변하지 않으므로, 경사 하강법의 전역 최솟값 수렴이 보장됨.
  - **립시츠 연속 Lipschitz continuous** : 어떤 함수의 도함수가 일정한 범위 안에서 변할 때.
    - MSE는 x가 무한대일 떄 기울기가 무한대가 되므로 국부적인 립시츠 함수라고 함.



- 경사 하강법을 사용할 때는 **반드시 모든 특성이 같은 스케일을 갖도록** 만들어야 함.
  - 그러지 않으면 수렴하는데 훨씬 시간이 오래 걸림. (그래이디언트의 변화가 매우 줄어들어버리기 때문)



- 모델 훈련은 훈련 세트에서 비용 함수를 최소화하는 모델 파라미터의 조합을 찾는 일이며, 이를 모델의 **파라미터 공간 parameter space**에서 찾는다고 말함.
  - 파라미터가 많을 수록 공간의 차원이 커지고 검색이 더 어려워짐.







### 4.2.1 배치 경사 하강법

- 경사 하강법을 구현하려면 각 모델 파라미터에 대해 비용 함수의 그레이디언트를 계산해야 함.
  - 즉, theta가 조금 변경될 때 비용 함수가 얼마나 바뀌는지 계산해야 함.
  - **편도함수 partial derivative** 를 구해야함.

- 편도함수를 각각 계산하지말고 그레이디언트 벡터를 사용해서 한꺼번에 계산 함.
  - 비용함수의 그레이디언트 벡터를 구하는 공식은 매 경사 하강법 스텝에서 전체 훈련 세트 X에 대해 계산하며, 때문에 이 알고리즘을 **배치 경사 하강법 Batch gradient descent**라고 함.
    - 때문에 매우 큰 훈련 세트에서는 아주 느리지만, 특성 수에 민감하지 않은 장점이 있음.
    - 수십만 개 수준의 특성의 경우에 선형 회귀를 훈련시키려면, 정규방정식이나 SVD 분해 보다는 경사 하강법이 훨씬 빠름.
- 그레이디언트 벡터를 구한 뒤 내려가는 스텝의 크기를 곱해서 값을 결정함.



- 적절한 학습률을 찾기 위해서는 그리드 탐색을 사용하지만, 그리드 탐색에서 수렴하는 데 너무 오래 걸리는 모델을 막기 위해 반복 횟수를 제한해야 함.
  - 반복 횟수는 너무 작으면 최적점에 도달 전에 알고리즘이 멈추고 / 너무 크면 모델 파라미터가 더는 변하지 않는 동안 시간을 낭비하게 됨.
    - 해결법은 반복 횟수를 아주 크게 지정하고, 그레이디언트 벡터가 아주 작아지면 (= 벡터의 norm이 허용오차 tolerance) 보다 작아지면 경사 하강법이 최솟값에 충분히 근접했으므로 알고리즘을 중지하면 됨.
      - 보통 허용오차를 1/10로 줄이면 알고리즘의 반복은 10배 늘어남







### 4.2.2 확률적 경사 하강법

- 배치 경사 하강법의 가장 큰 문제는 매 스텝에서 전체 훈련 세트를 사용하므로, 훈련 세트가 커지면 매우 느려진다는 점.
  - 반면 **확률적 경사 하강법**은 매 스텝에서 한 개의 샘플을 무작위로 선택하고, 그 하나의 샘플에 대한 그레이디언트를 계산함.
    - 매 반복에서 다뤄야 할 데이터가 매우 적기 때문에 알고리즘이 훨씬 빠르고, 메모리 점유가 적기 때문에 매우 큰 훈련 세트도 훈련시킬 수 있음. (SGD는 외부 메모리 학습 알고리즘으로 구현할 수 있음.)
- 빠른 대신, 확률적이므로 배치 경사 하강법보다 훨씬 불안정한 단점이 있음.
  - 비용 함수가 최솟값에 다다를 때까지 위아래로 요동치며, 평균적으로 수렴함.
  - 시간이 지나면 최솟값에 매우 근접하겠지만,  계속 요동치며 최솟값에 안착하지는 못할 것.
- 대신 비용 하수가 매우 불규칙할 때 알고리즘이 지역 최솟값을 건너뛰도록 도와주므로 전역 최솟값을 찾을 가능성은 배치 경사 하강법보다 높음.



- 정리하면, 지역 최솟값에서 탈출시켜주지만 / 전역 최솟값에는 다다르지 못하는 장단점.
  - 이를 해결하기 위해 학습률을 점진적으로 감소시키는 방법을 사용함.
    - 시작할 때는 학습률을 크게 해서 지역 최솟값에 빠지지 않게 하고,
    - 점차 작게 줄여서 알고리즘이 전역 최솟값에 도달하게 함.
    - 이러한 과정은 금속공학의 어닐링에서 영감을 얻어, **담금질 기법 Simulated annealing** 알고리즘과 유사함.
    - 매 반복에서 학습률을 결정하는 함수를 **학습 스케쥴 Learning schedule**
      - 학습률이 너무 빨리 줄어들면 지역 최솟값에 갇히거나, 최솟값까지 가는 중간에 멈춤
      - 학습률이 너무 천천히 줄어들면 오랫동안 최솟값 주변을 맴돌거나 훈련을 너무 일찍 중지해서 지역 최솟값에 머무를 수 있음.



- **에포크 Epoch** : 한 반복에서 훈련 세트에 있는 샘플 수 만큼 되풀이 됨. 각 반복을 에포크라고 함.
- 샘플이 무작위로 선택, 한 에포크에서 어떤 샘플은 여러 번 선택될 수도 있고 선택 안될 수도 있음.
  - 에포크마다 모든 샘플을 사용하게 하려면, 훈련 세트를 섞은 후 차례대로 하나씩 선택하게 하고, 다음 에포크에서 다시 섞는 방법을 사용.
    - 대신 이러면 보통 더 늦게 수렴하는 단점이 있음.







### 4.2.3 미니배치 경사 하강법

- **미니배치**라 부르는 임의의 작은 샘플 세트에 대해 그레이디언트를 계산하는 방법
  - 확률적 경사 하강법에 비해, 행렬 연산에 최적화된 하드웨어, 특히 GPU를 사용해서 성능 향상을 얻는 장점이 있음.
  - 미니배치를 적당히 크게 하면, 파라미터 공간에서 SGD보다 덜 불규칙하게 움직이고, 결국 SGD보다 최솟값에 더 가까이 도달할 수 있지만, 동시에 지역 최솟값에서 빠져나오기는 더 힘듦.



- 배치, 미니배치, 확률 모두 최솟값 근처에 도달했지만,
  - 배치는 실제로 최솟값에서 멈췄고,
  - 확률, 미니배치는 근처에서 맴돌고 있음.
- 하지만 배치는 매 스텝에서 많은 시간이 소요되고,
  - 확률과 미니배치는 적절한 학습 스케줄을 사용하면 최솟값에 도달하게 할 수 있음.







## 4.3 다항 회귀

- 비선형 데이터 학습에 선형 모델을 사용한 경우.
  - **다항 회귀 Polynomial regression** : 각 특성의 거듭제곱을 새로운 특성으로 추가하고, 이 확장된 특성을 포함한 데이터셋에 선형 모델을 훈련시키는 기법







## 4.4 학습 곡선

- 고차 다항 회귀는 보통 선형 회귀보다 훨씬 더 훈련 데이터에 잘 맞을 것.

  - 과대적합에 주의해야 함.

    - 과소적합일 수도 있으니, 이를 평가할 수 있어야 함.

    - 모델의 일반화 성능을 추정하기 위해 전에는 교차 검증을 사용했음.
      - 성능이 좋지만 교차 검증 점수가 나쁘면 과대적합
      - 둘 다 나쁘면 과소적합
    - 다른 방법으로 **학습 곡선** 을 살펴보는 방법이 있음.



- **학습 곡선** : 훈련 세트와 검증 세트의 모델 성능을 훈련 세트 크기의 함수로 나타낸 것.
  - 과소적합 : 전반적으로 RMSE 값이 높고, 두 곡선이 수평이고 오차가 거의 없이 근접해 있는 경우
  - 과대적합 : 전반적으로 RMSE 값이 낮고, 두 곡선 사이의 차이가 크며, 훈련세트가 검증세트보다 성능이 훨씬 나은 경우



- **편향 / 분산 트레이드 오프**

  - 통계학과 머신러닝에서 나온 중요한 이론 : 모델의 일반화 오차는 세 가지 다른 종류의 오차의 합으로 표현할 수 있다.
    - 편향 bias
      - 잘못된 가정으로 생김. 예를 들어 데이터가 실제로는 2차인데 선형이라 가정한 경우 발생.
      - 편향이 큰 모델은 과소적합되기 쉬움
    - 분산 variance
      - 훈련 데이터에 있는 작은 변동에 모델이 과도하게 민감해서 생김
      - 자유도가 높은 모델(예를 들면 고차 다항 회귀 모델)이 높은 분산을 가지기 쉬움.
      - 분산이 높으면 과대적합되기 쉬움
    - 줄일 수 없는 오차 Irreducible error
      - 데이터 자체에 잡음이 있어서 발생.
      - 이 오차를 줄일 수 있는 유일한 방법은 데이터에서 잡음을 제거하는 것.

  

  - 모델의 복잡도가 커지면 통상적으로 분산이 늘어나고 편향은 줄어듦.
  - 반대로 모델의 복잡도가 줄어들면 편향이 커지고 분산이 작아짐. 그래서 트레이드오프라 함.







## 4.5 규제가 있는 선형 모델

- 과대적합을 감소시키는 좋은 방법은 모델을 규제하는 것 (자유도를 감소)
- 다항 회귀 모델을 규제하는 간단한 방법은 다항식의 차수를 감소시키는 것.
- 릿지 회귀, 라쏘 회귀, 엘라스틱넷







### 4.5.1 릿지 회귀

- **릿지 Ridge 회귀 (티호노프 Tikhonov 규제)** : 선형 회귀, 규제항을 비용 함수에 추가
  - 규제항은 훈련하는 동안에만 비용 함수에 추가
  - 모델의 훈련이 끝나면 규제가 없는 성능 지표로 평가



- 편향 theta 0 는 규제하지 않음
- 미분시 계수를 깔끔하게 만들기 위해 1/2 계수를 시그마 앞에 붙임
- 가중치 벡터의 L2 norm
- 하이퍼파라미터 alpha를 시그마 앞에 붙여서 모델을 얼마나 많이 규제할지 조절
  - 0 이면 선형회귀와 같고
  - 아주 크면 모든 가중치가 0에 가까워지고, 결국 데이터의 평균을 지나는 수평선이 됨







### 4.5.2 라쏘 회귀

- **라쏘 Least Absolute Shrinkage and Selection Operator, Lasso** 회귀

  - 선형 회귀의 또 다른 규제된 버전

  - 릿지 회귀처럼 비용 함수에 규제항을 더하지만, L1 norm을 사용



- 라쏘 회귀의 중요한 특징은 덜 중요한 특성의 가중치를 제거하려고 한다는 점 (즉, 가중치가 0이 됨)
  - 자동으로 특성 선택을 하고 **희소 모델 Sparse model**을 만듬(즉, 0이 아닌 가중치가 희소함)



- 라쏘 회귀와 다른 점 2가지
  - 파라미터가 전역 최적점에 가까워질수록 그레이디언트가 작아짐
    - 경사 하강법이 자동으로 느려지고, 수렴에 도움 됨, 진동 없음
  - alpha 를 증가시킬수록 최적의 파라미터가 원점에 더 가까워지지만, 완전히 0이 되지는 않음



- 라쏘를 사용할 때는 경사 하강법이 최적점 근처에서 진동하는 것을 막기위해 훈련하는 동안 점진적으로 학습률을 감소시켜야 함. 그래야 수렴.



- 라쏘의 비용함수는 theta = 0 일 때 미분 가능하지 않은 문제가 있음
  - 이를 해결하기 위해 **서브그레이디언트 벡터 Subgradient vector, g** 를 사용해서 경사 하강법을 적용함.







### 4.5.3 엘라스틱넷

- **엘라스틱넷 Elastic net** : 릿지 회귀와 라쏘 회귀를 절충한 모델
  - 규제항을 릿지와 라쏘를 더해서 사용하며, 그 비율은 혼합 비율 상수 r을 사용해서 조절
    - r = 0 이면, 릿지 회귀
    - r = 1 이면, 라쏘 회귀



- 규제가 약간 있는 것이 대부분의 경우에 좋음. 평범한 선형 회귀는 피하는게 낫다.
  - 릿지가 기본이지만, 쓰이는 특성이 몇 개뿐이라고 의심되면 라쏘나 엘라스틱넷이 나음.
    - 필요없는 특성의 가중치를 0으로 만들기 때문
  - 특성 수가 훈련 샘플 수보다 많거나 특성 몇 개가 강하게 연관되어 있는 경우는 보통 라쏘가 문제를 일으켜서 엘라스틱넷을 선호.







### 4.5.4 조기 종료

- **조기 종류 Early stopping** : 규제하는 아주 색다른 방식으로, 검증 에러가 최솟값에 도달하면 바로 훈련을 중지시키는 것.
  - 에포크가 진행되면서 검증 에러가 감소하다 다시 상승하는 경우가 있음. 이는 과대적합의 시작을 의미.
  - 조기 종료는 과대적합을 피하기 위해 즉시 훈련을 멈추는 것.



- 확률적 경사 하강법이나 미니배치 경사 하강법에서는 곡선이 매끄럽지 않아 최솟값에 도달했는지 확신이 어려움.

  - 해결책 : 검증 에러가 일정 시간 동안 최솟값보다 클 때 학습을 멈추고 검증 에러가 최소였을 때의 모델 파라미터로 되돌림.

  

  

  

## 4.6 로지스틱 회귀

- **로지스틱 회귀 (로짓 회귀 Logit regression)** 은 샘플이 특정 클래스에 속할 확률을 추청하는데 널리 사용
  - 50%가 넘으면 모델은 그 샘플이 해당 클래스에 속한다고 예측
    - 레이블이 '1' 인 **양성 클래스** / 레이블이 '0'인 **음성 클래스**
- 0과 1사이의 값을 출력하는 **시그모이드 함수 Sigmoid function** 사용

- **로짓** 은 확률 p와 아닐 확률 1-p의 분수에 로그를 취한 것







### 4.6.2 훈련과 비용 함수

- 로지스틱 회귀의 비용 함수(로그 손실)의 최솟값을 계산하는 알려진 해, 정규방정식은 없음. 하지만 볼록 함수 이므로 경사 하강법(이나 다른 알고리즘)을 사용해서 전역 최솟값을 찾을 수 있음.
  - 따라서, 편미분을 해서 배치, 확률적, 미니배치 경사 하강법을 적용







### 4.6.3 결정 경계

- 다른 선형 모델처럼 로지스틱 선형 모델도 L1, L2 페널티를 사용하여 규제할 수 있음.
  - 사이킷런은 L2 페널티가 디폴트.







### 4.6.4 소프트맥스 회귀

- 로지스틱 회귀 모델은 여러 개의 이진 분류기를 연결하지 않고, 직접 다중 클래스를 지원하도록 일반화할 수 있음. 이를 **소프트맥스 회귀 Softmax regression** 또는 **다항 로지스틱 회귀 Multinomial logistic regression** 이라 함
- 샘플 x 에 대해 소프트맥스 회귀 모델이 각 클래스 k에 대한 점수 s를 계산하고, 그 점수에 **소프트맥스 함수** 또는 **정규화된 지수 함수 Normalized exponential function**을 적용하여 각 클래스의 확률을 추정.
  - 소프트맥스 회귀 분류기는 한 번에 하나의 클래스만 예측함. 다중 클래스지 다중 출력은 아님.  따라서, 종류가 다른 붓꽃 같이 상호 배타적인 클래스에서만 사용해야함. 여러 사람의 얼굴을 인식하는 같은 경우는 사용할 수 없음.



- 모델이 타깃 클래스에 대해서는 높은 확률을, 다른 클래스에 대해서는 낮은 확률을 추정하도록 만들어야 함.
  - **크로스 엔트로피 Cross entropy** 비용 함수를 사용. 추정된 클래스의 확률이 타깃 클래스에 얼마나 잘 맞는지 측정하는 용도로 종종 사용.
    - 두 개의 클래스만 있는 K=2 의 경우, 로지스틱 회귀 비용 함수와 같음
      - **쿨백-라이블러 발산 Kullback-Leibler divergence** : 크로스 엔트로피를 사용할 때의 전제 가정이 틀린 경우, 크로스 엔트로피의 값이 커져버림.







## 4.7 연습문제

생략.