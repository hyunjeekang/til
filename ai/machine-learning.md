# machine learing

## 정의
데이터를 기반으로 **최적 모델**을 계산하여 완성 <br/>
이를 활용해 새로운 데이터 **예측** & **분류** <br/>
LLM에서 사용되는 신경망도 머신러닝의 일종

### 모델
입력 데이터 - 출력 데이터 관계를 수식으로 표현한 함수 <br/>

<img src = "https://miro.medium.com/v2/resize:fit:1312/1*RyMB9kkhDbP66_mwYNJcpA.png">

`feature` : 모델에 입력되는 변수 <br/>
`output`: 모델 예측 결과값

### 학습 & 추론
**학습** : 모델을 만드는 과정<br/>
**추론** : 만든 모델을 가지고 예측하는 것

## Linear Regression
선형 회귀 모델 : `feature`의 `추세선` (feature 1개) or `추세면`(feature 2개)<br/>

### 추세선
목표 : **점들과 떨어져있는 거리의 합이 최소인 선** 찾기 <br>
`y = ax + b`에서 `a`, `b` 찾기 -> 추세선

오차 : 실제 `y`값과 예측된 `y`값의 오차<br/>
각 점별로 **오차의 합**이 최소가 되는 직선 -> 추세선!<br/>

<img src = "https://statistics.laerd.com/spss-tutorials/img/lr/linear-nonlinear-relationships.png"> <br>

`a`, `b` 를 어떻게 찾을까? <br/>

`Grid Search` 방식 또는 `Normal Equation (정규방정식)` 방식으로 <br/>
`MSE` 값이 작게 나오는 `a`, `b` 를 계산한다! <br/>

**MAE : Mean Absolute Error**<br>
오차의 합은.. 데이터가 많아지면 값이 아주 커진다 <br/>
그렇다면 오차의 평균을 써보자! : `MAE`<br>
데이터 수에 관계 없이 예측 모델과 실제 값들의 차이를 수치로 설명할 수 있다.<br/>
`np.mean()`을 사용하면 구할 수 있다.<br><br>

**MSE : Mean Sequre Error**<br>
`MAE`는 오차 값이 적절히 반영된다.<br/>
큰 오차는 큰 에러가 발생했다!며 크게 반영하고 싶다면?<br/>
오차에 제곱 -> 오차 갑셍 크게 반응하도록 만든 것이 `MSE`<br>

**Grid Search** <br/>
`MSE`값이 최소가 되는 a, b를 *2중 for문으로* 구하는 방법

**정규방정식**<br>
`MSE`값이 최소가 되는 a, b를 2중 for문 없이 *미분 방정식*으로 한 번에 구하는 방법!

<img src = "https://miro.medium.com/v2/resize:fit:1120/1*7ZiWm6xAF4oWiYfWklUMEw.jpeg"><br/>

정규 방정식의 단점
- 역행렬 계산하는 데 걸리는 시간
- 역행렬을 구할 수 없을 때도 있다
- feature가 늘어날 수록 연산이 불가능할정도로 연산량과 메모리 사용량이 많다
- 이래서 `Gradient Descent(경사 하강법)`을 사용한다!
<br/>
<br/>

**Gradient Descent : 경사 하강법**<br/>
전체 데이터를 말고 *최소한의 데이터*만 살펴보며 `MSE`가 최소인 곳을 찾기(`확률적 경사하강법`, `미니 배치 경사하강법`)<br/>

경사하강법 과정
- 랜덤한 지점 선택
- 지점의 기울기 구하기(미분) -> 최소 방향 찾기
- 해당 방향의 랜덤 지점 으로 점프!
- 반복

점프 이동 크기를 `학습률`, 전체 확인 수를 `에포크`라고 한다

적은 횟수로 `MSE`가 작은 좋은 `a`값을 찾을 수 있다<br/>
참고) 무조건 `MSE`가 최소인 값일 필요는 없음. 적당히 작으면 된다
<br/>

<img src = "https://media.geeksforgeeks.org/wp-content/uploads/20250609112949505173/Batch-Gradient-Descent.webp"><br/><br/>

**Adam Optimizer 아담**<br/>
<img src = "https://miro.medium.com/1*_9-WcjGQ-yXEOEGW-wQXWw.png"><br>
만약 이 그림처럼 극소값이 여러 개인 경우 `LM (local minimum)`에 갇힐 수 있다.<br>
더 좋은 `a` `b`를 찾지 못하고 학습이 안 될 수 있음<br/>

더 똑똑한 방식으로 점프하는 방식이 `Adam`!
- 이전 점프 방향과 동일한 방향으로 몇번 더 점프해보기 (관성)
- 이전 점프와 동일한 방향이었따면 조금 더 크게 점프해보기
- 왔다갔다 하는 부분은 학습률을 줄여본다 (RMSProp)

## Logistic Regression

<img src = "https://miro.medium.com/1*3FgpptTWzpd2RLgKbV-HvA.jpeg">

실제로는 분류 기법<br/>
입력 값이 **두 범주 중 한 범주에 속할 확률**을 예측하는 기법<br/>
-> 이진 분류 문제에 사용<br/>


<img src = "https://dezyre.gumlet.io/images/blog/example-on-how-to-do-logistic-regression-in-r/image_60871686071641293463932.png?w=376&dpr=2.6"><br/>

`y = sigmoid(ax+b)`에서 `a`, `b` 찾기 -> 확률 곡선 만들기<br/>

<img src = "https://miro.medium.com/0*D5do3xhv5ulF50w2.png"><br/>

**Sigmoid function**
어떤 값이 들어와도 0과 1의 사이 값으로 바꿔주는 함수<br/>
-> 분류 가능

로지스틱 회귀 역시<br/>
`Grid Search` 방식 또는 `Gradient Descent` 방식으로 <br/>
`Cross Entropy` 값이 작게 나오는 `a`, `b` 를 찾는다. <br/>

선형회귀에서의 `MSE`는 오차의 제곱을 사용한다. <br/>
근데 생각해보면? sigmoid이기 때문에 0과 1사이의 값이 되는데, <br/>
이를 제곱하면 더 작아진다..(0.5*0.5 = 0.25) <br/>

**Cross Entropy** <br/>
위의 문제를 해결하기 위해 `Cross Entropy`를 사용한다. <br/>
기능은 선형회귀에서의 `MSE`와 같다<br/>
작은 오차는 적게, 큰 오차는 아주 크게 확대해주는 것!<br/>


## Summary

<img src = "https://miro.medium.com/1*lnWfrrvR8qkANHombhQMTQ.png">