# Top 10 Best Open Source Projects on Github 2023

https://medium.com/@open-data-analytics/top-10-best-open-source-projects-on-github-2023-784bf4df2a94

## 1. RLHF + PaLM: Open Source ChatGPT Alternative

- 대체 프로그램: ChatGPT
- github link: https://github.com/lucidrains/PaLM-rlhf-pytorch
- PaLM2: https://korea.googleblog.com/2023/05/google-palm-2-ai-large-language-model.html

PaLM (Pathways Language Model)
Pathways: 특정 작업을 매우 잘 수행하도록 학습된 알고리즘을 만드는 일반적인 방법과는 달리, 문제를 해결하는 방법을 학습해서 모든 문제를 해결할 수 있는 단을 AI 모델을 만드는 접근방식

구글 Bard 가 PaLM 2를 기반으로 한다고 합니다.

RLHF와 PaLM 을 활용하여 ChatGPT 의 오픈소스 버전을 내는 것이 목표라고 하네요. 현재 [작업 중인](https://github.com/lucidrains/PaLM-rlhf-pytorch#todo) 프로젝트입니다.

## 2. RATH

- 대체 프로그램: Tableau
- github link: https://github.com/Kanaries/Rath

데이터 분석 및 시각화하는(BI, Business Intelligence) 오픈소스라고 합니다.

> RATH is a must-have tool for anyone looking to improve their data analysis and visualization skills.

## 3. Gogs

- 대체 프로그램: Github
- github link: https://github.com/gogs/gogs

github 와 달리 커스터마이징이 자유로운 장점이 있는 것 같습니다.
> The Gogs (/gɑgz/) project aims to build a simple, stable and extensible self-hosted Git service that can be set up in the most painless way.

## 4. NocoDB

- 대체 프로그램: AirTable
- github link: https://github.com/nocodb/nocodb/blob/develop/markdown/readme/languages/korean.md

DB를 좀 더 쉽게 사용할 수 있고, 시각화를 할 수 있게 도와주는 프로그램입니다.

![스크린샷 2023-06-11 19.26.13.png](/files/0a710ba9-87a1-1b45-8188-aa204d3607b1)
No-Code 데이터베이스 인터페이스를 오픈소스로 제공하는 것이 목표라고 하네요.

## 5. Rocket.Chat

- 대체 프로그램: Slack
- github link: https://github.com/RocketChat/Rocket.Chat

매터모스트와 유사하게 슬랙을 대체하는 프로그램입니다.
음성/영상 통화도 되고 화면공유도 되는 것을 보면 슬랙 + 디스코드 에 가까운 것 같아서 매터모스트보다 훨씬 나은 것 같기도..? 하네요.

## 6. Airbyte

- 대체 프로그램: Fivetran
- github link: https://github.com/airbytehq/airbyte

data integration 을 하는 간단하지만 강력한 인터페이스를 제공해준다고 합니다.
Fivetran 도 "Automated data movement" 라고 하고 Airbtye도 "open source solution to data movement" 이라고 하는 걸 보면 data movement 라고 뭔가 명칭이 따로 있는 것 같네요.

https://docs.airbyte.com/integrations/ 에서 지원하는 "Connector" 목록을 확인할 수 있는데요. 상당히 많네요. 
이 많은 Connect 들의 데이터를 Airbyte를 통해서 data warehouse, data lake, database, analytics tool 에 보낼 수 있는 것 같습니다.

## 7. Plausible Analytics

- 대체 프로그램: Google analytics
- github links: https://github.com/plausible/analytics

Google Analytics와 달리 개인정보를 수집하지 않는다는 점을 강조하는 것 같네요.
원한다면 Plausible Cloud를 사용하지 않고 Self-hosting 을 통해서 프로그램을 사용할 수도 있는 것 같습니다. 

## 8. Supabase

- 대체 프로그램: Firebase
- github link: https://github.com/supabase/supabase

각각의 오픈소스를 통합하여 하나로 제공하여 Firebase를 대체할 수 있도록 하는 것이 목표라고 하네요.
PostgreSQL(DB), Realtime(DB 변경사항 브로드캐스팅), PostgREST(RESTful API), pg_graphql, GoTrue(Token), Kong(Gateway) 와 같은 오픈소스를 사용합니다.

## 9. Kdenlive

- 대체 프로그램: Adobe Premiere
- github link: https://github.com/KDE/kdenlive, https://github.com/mltframework/mlt, https://invent.kde.org/frameworks

오,, 어떻게 이게 가능한지 조금 신기하긴 하네요.
영상편집툴의 오픈소스버전이 있다는 게 신기하게 느껴지는 것 같습니다.

영상편집 기능은 [MLT framework](https://en.wikipedia.org/wiki/Media_Lovin%27_Toolkit), UI 및 기타 기능들은 [KDE framework](https://ko.wikipedia.org/wiki/KDE_프레임워크) 를 사용한 것으로 보입니다.

## 10. mastodon

- 대체 프로그램: Twitter
- github link: https://github.com/mastodon/mastodon

https://www.chosun.com/economy/tech_it/2023/01/20/GNBGME5HQ5EDBI4RXTWFVF4GYE/?utm_source=naver&utm_medium=referral&utm_campaign=naver-news

예전에 보았던 뉴스에서 보았던 오픈소스였던 것 같은데, 아직도 핫한 것 같네요.
탈중앙화 SNS 로 각자 서버(여기서는 인스턴스라고 부름)를 구축하고, 각 인스턴스를 통해서도 메시지를 주고받거나 팔로잉을 할 수 있도록 구성되어 있다고 합니다.