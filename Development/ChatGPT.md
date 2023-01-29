출처: https://www.assemblyai.com/blog/how-chatgpt-actually-works/

# How ChatGPT actually works

## 1. 개요

이전 모델인 GPT-3 보다 상당한 발전을 이루어낸 Large Language Model (LLM)

![capability-versus-alignment-in-machine-learning---how-chatGPT-works--1-](https://user-images.githubusercontent.com/19607962/215320014-22c213d2-6ee2-4708-ba7c-2a413baea7c3.png)


일반적인 LLM 모델들은 Capability(precise = 정밀도)는 높지만, Alignment(accuracy = 정확도)는 낮은 문제가 있습니다.
즉, 학습된 결과에 대응되는 응답은 잘 생성하지만, 실제 사용자들이 원하는 응답을 주는 확률이 낮았습니다.

- Leak of helpfulness
    - 사용자가 입력한 명령어에 의도한 결과를 내지 않는 문제
- Hallucinations
    - 존재하지 않거나 잘못된 사실로 응답을 만드는 문제
- Lack of Interpertability
    - 사람이 해당 응답을 만들기 위해 어떤 과정을 거쳤는지 알 수 없음.
- Generating biased or toxic output
    - 치우친 성향의 응답이나 해로운 응답을 만들어 낼 수 있음.

이러한 문제를 해결하기 위해 Supervised Learning(지도학습) 이외에도 Reinforcement Learning(강화학습)을 같이 사용하여 해결하였습니다.

다른 프로그램과의 차이점이 바로 이 **RLHF(Reinforcement Learning from Human Feedback)** 에서 발생하게 됩니다.
RLHF를 활용하여 사람의 Feedback 을 학습과정에 포함시켜서, 해롭거나, 신뢰성이 낮거나, 한 쪽 사상에 치우친 응답들을 최소화합니다.

ChatGPT가 RLHF 라는 기술이 도입된 최초의 상용 프로그램이라고 하네요.

## 2. Learning Stratigies of Large Language Model

Language Model을 학습하는 방법으로 여기서는 크게 2가지의 방법을 설명하고 있습니다.

1. Next-Token-Prediction
2. Masked-Language-Modeling

### Next-Token-Prediction

**"머리부터 발끝까지"** 라는 문구 뒤에 나올 수 있는 문구는 **"다 사랑스러워", "핫이슈", "오로나민씨"** 등의 문구가 나올 수 있는데요.
모델이 가지고 있는 단어들 중 다음 문구로 나타날 확률이 높은 단어들을 선택해서 문장을 만들어내게 됩니다.

Supervised Learning 방식에서는 X 다음엔 Y라는 값이 나오는 걸 알고 있는 상태에서 X라는 값을 input 으로 넣었을 때 Y라는 값을 생성하도록 학습을 진행합니다.

### Masked-Language-Modeling

**"머리부터 [MASK] 오로나민씨"**

위처럼 문장의 특정 부분을 [MASK] 라는 식으로 마스킹처리하여, 모델에서 해당 부분을 예측하도록 학습하게 됩니다.

Supervised Learning 방식에서는 해당위치에서 나올 수 있는 값을 정해두고, 정해둔 값을 생성할 수 있도록 모델을 학습시킵니다.

### 두 방식의 차이?

두 방식의 학습방식을 보시면 아실 수 있겠지만,

- Next-Token-Prediction = 계속 **문장을 만들어내는 것에 특화**되어있습니다.
- Masked-Language-Modeling = **완성된 문장의 빈 곳을 채워넣는 것에 특화**되어있습니다. 

👉 ChatGPT 에서는 문장을 생성해야하기 때문에, **Next-Token-Prediction** 방식을 사용했다고 합니다.

### 두 방식의 한계?

위 예시에서 볼 수 있지만, "머리부터 발끝까지 [MASK]" 의 마스킹 부분을 채울 수 있는 값은 3가지(혹은 그 이상) 입니다.
하지만 모델에서는 어떤 값이 더 적절한 값인지 판단할 수 없는데, 다 적절한 응답이기 때문입니다.

---> ? 적절한 예시인가는 의문

> 일반적인 LLM 모델들은 Capability(precise = 정밀도)는 높지만, Alignment(accuracy = 정확도)는 낮은 문제가 있습니다.

위 두 방식은 문맥을 이해하고 생성하는 것이 아니라, 확률분포에 의해 만들기 때문에 정확도가 낮은 한계가 있습니다.

**이러한 문제를 해결하기 위해서 ChatGPT-3에서는 RLHF를 활용하게 됩니다.**

## 3. Reinforcement Learning from Human Feedback

1. Supervised Fine Tuning
    - OUTPUT: SFT Model
2. Mimic human preference Step
    - OUTPUT: Reward Model
3. Proximal Policy Optimization (PPO) Step
    - OUTPUT: Policy Model
4. Evaluation Step

RLHF 는 다음과 같은 3가지 Step(평가 단계는 제외)이 존재하는데요. 하나씩 확인해보겠습니다.

1단계는 한 번만 수행되고, 2단계, 3단계는 반복적으로 수행될수 있습니다.

👉 3단계에서 생성된 Policy Model로, 새로 2번단계를 수행해서 Reward Model의 최적화를 수행할 수 있습니다.

### Supervised Fine Tuning Step

pre-trained Language Model 을 fine-tuning 하는 Step 입니다.

<details>
<summary>용어 정리</summary>

- pre-training
    - 문제를 해결할 수 있는 하위문제들을 학습한 모델로 가중치를 초기화하는 작업
    - ex) 텍스트 유사도 예측 모델을 만들기 위해, 감정 분석 문제를 학습한 모델의 가중치를 가져오는 방식.
      감정분석문제를 해결하기 위해 언어와 관련된 학습이 이루어져있을 것이므로, 활용할 수 있다.
      [출처](https://velog.io/@soyoun9798/Pre-training-fine-tuning)
- fine-tuning
    - 기존에 학습되어있는 모델을 기반으로 목적에 맞도록 모델을 변형(학습)시키는 과정
    - 모델의 Weight를 조정하는 작업

</details>

1. 데이터 수집
    - 학습을 위해 demonstration data (학습을 위해 필요한 데이터)를 수집해야합니다.
    - Human 라벨러에게 X 값을 보여주고, 예상되는 응답값을 입력하게 합니다. 
      또는 GPT-3 사용자들에게서 데이터를 수집하여, `X -> ?` 데이터를 수집합니다.
    - 이 작업은 수작업으로 진행되기 때문에, 느리고 비싼 작업이지만 고품질의 선별된 데이터를 얻을 수 있게됩니다.
2. 모델 선정
    - fine tuning 을 진행한 모델을 선정해야합니다.
    - ChatGPT는 GPT-3.5 시리즈의 `text-davinci-003` 모델을 사용하였습니다.
    - 특이한 점은 text-davinci-003 은 프로그래밍 코드를 작성하기 위해 튜닝된 모델이라는 점입니다.
3. fine-tuning 수행
    - 수집된 데이터는 `X(input) -> Y(output)` 을 알고 있기 때문에, weight 를 조정할 수 있습니다.
    - 조정하는 방식은 여러가지입니다만, 여기서는 따로 언급되지는 않은 것 같네요.

이렇게 모델을 학습시키더라도, 학습시키는 데이터의 양이 많지 않기 때문에 위에서 언급한 **정밀성은 높지만 정확성은 낮은** 문제는 해결되지 않습니다.

다음 단계에서 해당 문제를 해결하기 위한 작업을 진행합니다.

### Mimic Human Preference Step (Reward Model)

SFT Model(1번 스텝에서 학습한 모델)의 outputs 을 평가하는 모델을 생성합니다.

평가의 기준은 **"사람의 선호도를 흉내내서 점수를 매긴다"** 입니다.
즉, 특정 그룹의 라벨러들의 선호도가 반영될 수도 있고, 특정 사회의 가이드라인들을 반영할 수도 있습니다.

#### 1. 특정 input 으로부터 SFT Model의 output 을 생성합니다.

"나의 취미는 [MAKS]이다" 라는 문장에서 마스킹된 부분을 채우는 작업을 한다고 가정합니다. (ChatGPT는 Next-Token Prediction 이라 다르긴 합니다)

후보 단어들로는 "게임", "코딩", "경범죄", "운동", "애니메이션 감상", "피아노 연주", "책 읽기" 등등이 나올 수 있습니다.

#### 2. 1번에서 나온 결과물에 순위를 매깁니다.

1번에서 나온 결과물을 특정 기준(사람이 매기거나, 특정 가이드라인을 생성해놓고 사용)으로 순위를 매깁니다.

랭킹|단어
:---:|:---:
1|게임
2|운동
3|피아노 연주
4|애니메이션 감상
5|책 읽기
6|경범죄

#### 3. 2번에서 순위를 매긴 결과물로 Reward Model 을 학습시킵니다.

특정 input 에 대한 결과값을 예상하는 것보다, 순위를 매기는 작업이 더 수월하기 때문에 작업할 데이터가 더 많더라도 효율적으로 처리될 수 있습니다.

### Proximal Policy Optimization Step

SFT Model(1번에서 생성된 모델)을 강화학습으로 fine-tuning 하는 단계입니다.



WIP.......


