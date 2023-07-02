https://medium.com/apolitical-engineering/code-reviews-at-apolitical-37dad421ace4

코드리뷰 문화에 대한 글입니다.
뭔가 매 번 코드리뷰를 어떻게 할 것인지? 에 대한 고민이 있어서 읽어보았습니다.

## 개요

1. 어떻게 코드리뷰에서 최고의 가치를 창출할 수 있을까?
2. 어떻게 해야 `rubber stamp(자동적으로 approve 누르는 사람)` 이 되지 않을 수 있을까?
3. 코드리뷰가 업무속도에 영향을 주지 않는지 확인할 수 있을까?
4. 개발경험을 어떻게 향상시킬 수 있을까?

이런 의문을 가지고 몇 가지 원칙들을 세워보았다고 합니다.

## 원칙

### Reviewing code trumps writing code

코드를 작성하는 것은 쉽지만, 운영환경에서 문제없이 동작할 정도로 만드는 것은 어렵습니다.

실제 배포되어 동작하기 전까지의 코드는 큰 의미를 가지지 않습니다.
새로운 코드를 만드는 것보다 리뷰하는 것에서 더 많은 가치를 만들 수 있을 것이라고 생각하였습니다.

코드를 리뷰하는 것에 많은 시간을 사용하기로 하였습니다.
코드리뷰는 `회의가 끝나고 난 후` 또는 `점심을 먹고 난 직후`나 `업무의 시작 또는 끝 시간` 에 수행하는 것이 좋은 것으로 판단하였습니다.

그리고 코드리뷰를 받기 위해 기다리는 시간이 얼마가 적절할지 확인하는 작업이 필요합니다.
`저자의 팀에서는 24시간 이내에는 리뷰를 받을 수 있도록 권장하고 있다고 합니다.`

### But Reviewing code is writing code

리뷰하는 사람은 항상 "좀 더 개선된" 코드를 만들기 위해 리뷰를 해야합니다.

"같이 코드를 작성하는 사람"이라고 생각하고 리뷰를 진행해야합니다. 
단순히 "이 정도면 충분하네!"하고 승인을 누르는 일은 없어야 합니다.

> 주니어들은 시니어들의 코드를 리뷰하는 것이 쉽지 않다고 생각할 수 있는데,
> 배움의 기회라고 생각하면 좋은 시간이 될 수 있을 것이라고 이야기합니다.

### comment should indicate severity

우리는 아래 7가지 사항들을 확인하기 위해 코멘트를 작성하게 됩니다.

1. 코드가 잘 동작하는지
2. 코드가 효율적인지
3. 유지보수가 용이한지
4. 혹시라도 놓친 것은 없는지
5. 코드에 대해 질문을 하는 경우
6. 알고 있는 것에 대한 공유가 필요할 때
7. kudos (?)

어떤 코멘트들은 중요한 문제를 해결할 수 있게 해주고, 어떤 코멘트는 특별한 액션을 취하지 않아도 됩니다.
대부분의 코멘트들은 둘 사이에 존재합니다.

따라서 코멘트를 남길 때 아래의 우선순위를 가지고 작성하면 도움이 될 것 입니다.

1. blocking
    - 해당 수정사항을 반영하지 않으면 승인하지 않아야하는 경우
2. recommended
    - 지금 당장 수정하기 어렵다면 이슈로 작성해두고 나중에 처리하도록 권유할 수 있다.
3. not blocking
    - 단순히 제안을 하거나 질문을 하는 경우

시니어가 주니어에게 리뷰할 때는 이 점을 특히 중요하게 생각해보아야 합니다.
**더 많은 경험을 가진 사람의 리뷰는 판단할 수 있는 경험이 없는 주니어에게는 pressure가 될 수 있기 때문입니다.**

### ...but all comments are suggestions

코드의 책임은 작성자에게 있지 리뷰어에게 있지 않다는 점을 강조합니다.
리뷰어는 일종의 거부권을 가지고 있을 뿐이라고 설명하고 있습니다. (남용할 수 없는 거부권을..)

저자의 팀에서는 코드에 대한 진정한 의미에서의 논쟁이 있는 경우, 대부분 작성자의 의견을 들어주는 편인데
해당 코드에 대한 context를 더 잘 알고 있을 것이라고 생각하기 때문이라고 합니다.

### if it can be automated, it should be automated

자동화할 수 있는 것들은 자동화하여, 리뷰어가 신경써야하는 부분을 줄일 수 있다고 이야기하고 있습니다.

1. formatting / linting
2. static type checking
3. unit tests/test coverage

---

- 가능하다면 규격화된 제안(자동화한)들을 사용하여, 작성자가 더 쉽게 받아들이고 수정할 수 있도록 하는 것이 좋다.
- 동일한 내용의 리뷰가 반복되고 있다면 자동화해야하는 신호로 볼 수 있다.

코드에 누락되어있는 부분을 리뷰어가 알아차리는 것은 쉽지 않습니다.
자동화할 수 없는 부분이라면 체크리스트 등을 통해서 코드 작성자가 먼저 확인할 수 있도록 하면 좋을 것이라고 설명하고 있습니다.

### Simple sanity checks are essential

`코드 작성 -> 리뷰 -> QA -> 배포`

일반적인 코드의 흐름입니다.

QA 단계에서 코드가 거부되는 일은 드문데, 만약 발생한다면 절차 상의 문제일 수 있다고 이야기합니다.
각 단계에서 다시 이전 단계로 돌아가 업무를 진행하는 것은 상당히 비효율적이라서 가능한 조기에 문제를 발견해서 해결해야 합니다.

그래서 우리는 리뷰단계에서 가능한 더 시간을 사용하여 이후 단계에서 더 효율적으로 작업할 수 있도록 해야합니다.

> If we pay that cost, only to reject code for a reason that could have been caught at an earlier step by someone already working with it:
>
> - that is a poor experience for the person rejecting the code, distracting them unnecessarily and for little return on the investment of their time
> - code may be bounced back to the author where this might otherwise have been avoided entirely

코드 작성자는 자동화한 내용들이 pass 되었는지, checklist 확인결과 문제없는지 등을 확인하여 리뷰어가 단순히 문제상황을 파악하는데 집중할 수 있도록 해야합니다.
리뷰어는 프론트에서 적절히 렌더링되고 있는지, API는 사소한 에러를 던지지 않는지 확인하고 승인해야합니다.

### ...and, especially, value your reviewer's time

> This advice sounds obvious, but it’s common to see authors treat their reviewers like personal quality assurance technicians.

우리의 동료는 제한된 시간(집중력)을 가지고 있는데, 그 시간을 업무를 진행하는 데 쓰는 것이 아니라 리뷰에 사용하는 것임을 잊지 말아야 합니다.
그래서 우리는 그들의 시간을 "maximise" 해 줄 필요가 있습니다.

이를 위해 코드 작성자는 자신의 코드를 남의 코드라고 생각하고 읽고 리뷰하는 작업을 거쳐야한다고 이야기합니다.
단순히 실수를 찾는 수준이 아니라, 처음 코드를 읽는다는 생각으로 진행해야 합니다.

피드백을 진지하게 받아들이고 개선할 노력을 해야하지, 넘어서야 할 장애물로 생각해서는 안 됩니다.