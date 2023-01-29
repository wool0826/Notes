출처: https://www.assemblyai.com/blog/how-chatgpt-actually-works/

# How ChatGPT actually works

## 1. 개요

이전 모델인 GPT-3 보다 상당한 발전을 이루어낸 Large Language Model (LLM)

![capability-versus-alignment-in-machine-learning---how-chatGPT-works--1-](https://user-images.githubusercontent.com/19607962/215320014-22c213d2-6ee2-4708-ba7c-2a413baea7c3.png)

일반적인 LLM 모델들은 Capability는 높지만, Alignment는 낮은 문제가 있습니다.
친구가 이야기해준 문장이 있는데, **"릴레이소설글쓰기 AI를 만들어놓고(high capability) 자비스를 원하는 상황(low alignment)"** 이라고 하는데 의미가 더 쉽게 와닿았던 것 같습니다.

※ capability: 모델이 특정 task를 수행할 수 있는 능력. GPT-3 에서는 문장을 이어서 만들 수 있는 능력. (안녕 다음에 합니까를 잘 뽑는 능력)
※ alignment: 실제로 우리가 원하는 응답을 만들어 줄 수 있는가에 대한 능력. (메뉴 추천 좀 해줘 라는 input에 적절한 메뉴를 추천한다던지 하는 능력)

기존 LLM은 아래와 같은 한계가 있습니다.

- Leak of helpfulness
    - 실제 우리가 원하는 응답을 만들어줄 수가 없는 문제.
- Hallucinations
    - 존재하지 않거나 잘못된 사실로 응답을 만드는 문제
- Lack of Interpertability
    - 사람이 해당 응답을 만들기 위해 어떤 과정을 거쳤는지 알 수 없음.
- Generating biased or toxic output
    - 치우친 성향의 응답이나 해로운 응답을 만들어 낼 수 있음.

이런 한계가 있는 이유는, Language Model 의 학습방법때문입니다.

### LLM 학습방법

`ChatGPT is the latest language model from OpenAI and represents a significant improvement over its predecessor GPT-3.`

라는 문장을 만들기 위해서는 

input = `ChatGPT`, output = `ChatGPT is`
input = `ChatGPT is`, output = `ChatGPT is the`
input = `ChatGPT is the`, output = `ChatGPT is the latest`
...
input = `ChatGPT is the latest language model from OpenAI and represents a significant improvement over its predecessor`, output = `ChatGPT is the latest language model from OpenAI and represents a significant improvement over its predecessor GPT-3`

와 같이, 일종의 릴레이 소설 쓰기와 같은 학습과정을 거치게 됩니다.
즉, 글을 이어서 생성하는 것은 잘 하지만, 실제 사용자의 의도에 맞는 결과를 만들어냈는가?에 대해서는 답할 수 없는 상황인 것입니다.

### ChatGPT의 해결책

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

Supervised Learning 방식에서는 해당 위치에서 나올 수 있는 값을 정해두고, 정해둔 값을 생성할 수 있도록 모델을 학습시킵니다.

### 두 방식의 차이?

두 방식의 학습방식을 보시면 아실 수 있겠지만,

- Next-Token-Prediction = 계속 **문장을 만들어내는 것에 특화**되어있습니다.
- Masked-Language-Modeling = **완성된 문장의 빈 곳을 채워넣는 것에 특화**되어있습니다. 

👉 ChatGPT 에서는 문장을 생성해야하기 때문에, **Next-Token-Prediction** 방식을 사용했다고 합니다.

### 두 방식의 한계?

위 예시에서 볼 수 있지만, "머리부터 발끝까지 [MASK]" 의 마스킹 부분을 채울 수 있는 값은 3가지(혹은 그 이상) 입니다.
하지만 모델에서는 어떤 값이 더 적절한 값인지 판단할 수 없는데, 다 적절한 응답으로 볼 수 있기 때문입니다. (세대에 따라 다르겠지만..)

> 일반적인 LLM 모델들은 Capability는 높지만, Alignment는 낮은 문제가 있습니다.

위 두 방식은 문맥을 이해하고 생성하는 것이 아니라, 확률분포에 의해 만들기 때문에 유저의 의도를 정확히 반영하지 못하는 문제가 발생합니다.
**이러한 문제를 해결하기 위해서 ChatGPT-3에서는 RLHF를 활용하게 됩니다.**

## 3. Reinforcement Learning from Human Feedback

1. Supervised Fine Tuning
    - 결과물: SFT Model
2. Mimic human preference Step
    - 결과물: Reward Model
3. Proximal Policy Optimization (PPO) Step
    - 결과물: Policy Model
4. Evaluation Step

RLHF 는 다음과 같은 3가지 Step(평가 단계는 제외)이 존재하는데요. 하나씩 확인해보겠습니다.

1단계는 한 번만 수행되고, 2단계, 3단계는 반복적으로 수행될수 있습니다.

👉 3단계에서 생성된 Policy Model로, 새로 2번단계를 수행해서 Reward Model의 최적화를 수행할 수 있습니다.

### Supervised Fine Tuning Step

pre-trained Language Model 을 fine-tuning 하는 Step 입니다.

- pre-training
    - 쉽게 말해, 영어 문법을 배우기 전에 기초적으로 알파벳과 기초적인 단어를 알아야 하는데, 모델에도 이런 전처리작업을 하는 것을 의미합니다.
    - ChatGPT 한정으로 이야기하게 되면, 주어진 문장에서 다음 단어를 선택하기 위해 언어에 대한 개념을 이해해야하는데, 이를 위해 언어모델을 학습시켜둡니다.
    - 이 부분에서는 기존 모델의 가중치를 빌려오는 것으로 이해하였습니다.
- fine-tuning
    - 기존에 학습되어있는 모델을 기반으로 목적에 맞도록 모델을 변형(학습)시키는 과정입니다.
    - 단어 뜻 그대로 모델의 가중치를 미세조정하는 작업

---

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

이렇게 모델을 학습시키더라도, 학습시키는 데이터의 양이 많지 않기 때문에 위에서 언급한 **글은 잘 만들지만, 내 말귀는 못 알아듣는** 문제는 해결되지 않습니다.

다음 단계에서 해당 문제를 해결하기 위한 작업을 진행합니다.

### Mimic Human Preference Step (Reward Model)

SFT Model(1번 스텝에서 학습한 모델)의 outputs 을 평가하는 모델을 생성합니다.

평가의 기준은 **"사람의 선호도를 흉내내서 점수를 매긴다"** 입니다.
특정 그룹의 라벨러들의 선호도가 반영될 수도 있고, 특정 사회의 가이드라인들을 반영할 수도 있습니다.

#### 1. 특정 input 으로부터 SFT Model의 output 을 생성합니다.

"나의 취미는 [MASK]이다" 라는 문장에서 마스킹된 부분을 채우는 작업을 한다고 가정합니다. (ChatGPT는 Next-Token Prediction 이라 다르긴 합니다)

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
6|경범죄 <------ 이런 유해한 결과들은 추후 평가과정에서 필터링됨.

#### 3. 2번에서 순위를 매긴 결과물로 Reward Model 을 학습시킵니다.

특정 input 에 대한 결과값을 예상하는 것보다, 순위를 매기는 작업이 더 수월하기 때문에 작업할 데이터가 더 많더라도 효율적으로 처리될 수 있습니다.

### Proximal Policy Optimization Step

> Policy: 궁극적으로 학습을 통해 구하려는 특정 상황에서의 Action 또는 Action에 대한 확률을 정의해놓은 것

SFT Model(1번에서 생성된 모델)을 강화학습으로 fine-tuning하는 단계입니다.

여기서 Proximal Policy Optimization 이라는 알고리즘을 사용하여 SFT Model을 fine-tuning 하게 되는데요.
PPO라는 알고리즘이 정확히 어떻게 동작하는지는 너무 깊게 들어가는 것 같으므로, 본문에서 나온 이야기만 간단히 정리해보았습니다.

- "현재 정책으로 인해 생성된 결과물" 과 "이전 정책으로 인해 생성된 결과물"을 비교하여 현재 정책을 학습합니다. (on-policy)
- 현재 정책을 변경할 때 변경되는 값의 최대값을 제한해서, 학습의 결과물을 안정적으로 관리합니다.
- value function이라는 것을 통해 주어진 액션의 예상 반환값을 추정하게 됩니다.
- value function을 활용하여 advantage function 을 계산하고, 이 advantage function을 이용해 모델의 학습을 진행합니다.

PPO의 초기모델 = SFT Model
Value Function의 초기모델 = Reward Model

즉, Reward Model 기반으로 결과값을 예측하고, 예상된 결과값을 활용하여 SFT Model 을 학습하는 구조라고 생각하시면 됩니다.

Reward Model은 2번 단계를 통해 또 자체적으로 학습이 진행되므로, 2개의 모델의 점진적인 학습을 통해 최종 모델을 구축하는 방식이라고 볼 수 있습니다.

### Performance Evaluation

> Because the model is trained on human labelers input, the core part of the evaluation is also based on human input.

모델은 크게 3가지 조건으로 평가됩니다.
Overfitting이 되는 것을 방지하기 위해, 학습데이터에는 없는 OpenAI의 사용자들의 prompt를 TestSet도 활용하는 것으로 보입니다.

- Helpfulness
    - 처음에 이야기되었던, 실제 대답이 유저의 요구를 충족하였는지? 를 판단합니다.
    - ex) 오늘 스트레스 받는 일이 있었어
        - 스트레스는 영국에서 최초로 시작되어 일년에 한 바퀴를 돌면서.... (❌)
        - 스트레스가 심할 때는 매운 것을 먹어서 해소하는 방법도 있습니다 (🟢)
- Truthfulness
    - 잘못된 사실을 응답하거나, 의미없는 말을 하지는 않는지 판단합니다. TruthfulQA dataset 이라는게 있어서 활용할 수도 있는 것 같네요.
    - ex) JDK 19는 언제 출시했어?
        - JDK 19는 출시된 적이 없습니다. (❌)
        - Oracle JDK 19는 22년 9월 21일 출시되었습니다. (🟢)
- Harmlessness
    - 폭력적이거나 선정적인, 문제되는 결과는 없는지 판단합니다.
    - 이것도 다양한 Dataset이 있어서 활용할 수 있는 것 같습니다.

질문에 답변을 하거나, 독해 및 요약을 하는 기존 NLP들과 동일하게 Zero-shot performance 도 평가하고, GPT-3 와 비교하여 성능평가도 진행하기도 하는 것 같습니다.

> This is an example of an “alignment tax” where the RLHF-based alignment procedure comes at the cost of lower performance on certain tasks.

## 단점

모델을 학습하는 과정에서 여러가지 주관적인 요소들에 의해 영향을 받는 문제가 있습니다.

- 최초 학습 데이터를 생성하는 labeler의 선호도
    - labeler가 X에 대한 예측되는 응답값을 제시해야함
- 연구를 설계하고 labeling 지침을 만드는 연구자
    - 연구자의 성향이 반영될 수 있다?
- 개발자로부터 생성되거나 OpenAI 사용자들이 제공한 prompt에서 선택
- labeler의 성향이 reward model 과 모델의 평가에 모두 참여
    - labeler가 응답값의 ranking을 측정하면서 성향이 반영됨
    - model 평가에 labeler의 성향이 반영됨
