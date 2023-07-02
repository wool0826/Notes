https://medium.com/swlh/rate-limiting-fdf15bfe84ab
https://medium.com/@ajin.sunny/system-design-concept-rate-limiting-f4da72371533
https://www.mimul.com/blog/about-rate-limit-algorithm/

### Types of Rate Limiter

- User rate limiter
    - 특정 유저가 API에 제한적으로 접근할 수 있도록 하는 limiter
    - 요청의 수나 지속시간 등이 식별자(userId, api keys, ip address) 등에 의해 제한되는 방식
- Concurrent/server rate limiter
    - 특정 유저의 parallel session이나 conection 의 제한을 두는 limiter
    - DDos attack 을 완화시킬 수 있다고 합니다.
- Location rate limiter
    - 특정 지역이나 특정 시간대에 제한을 거는 limiter

### Different ways to rate limit

#### 1. token bucket algorithm

![스크린샷 2022-12-18 22.37.57.png](/files/0a710dbd-83f9-1069-8185-2574b1113988?1671370868319)
예를 들면 1분의 5개의 요청을 받을 수 있도록 정해놓았다고 가정했을 때,

새로운 유저가 요청을 하게 되면, `요청시간/남은 요청 횟수` 를 가지는 데이터를 생성합니다. (이 데이터는 유저 식별자로 접근할 수 있게 됩니다.)
유저가 요청을 할 때마다 요청시간과 남은 요청횟수를 갱신합니다. 1분이 지나지 않았는데 남은 요청횟수가 0이 되게 된다면, 이후 해당 유저의 요청을 거절하는 식으로 동작합니다.

새로운 요청이 들어왔을 때, 요청시간이 저희가 정해둔 "1분" 기준을 넘었다면 다시 남은 요청 횟수를 5로 생성한 뒤 동일한 작업을 반복합니다.

---

![스크린샷 2022-12-18 22.38.28.png](/files/0a710dbd-83f9-1069-8185-25752585398c?1671370881080)
다른 문서에서는 다르게 표현되는데, bucket은 가질 수 있는 최대 토큰 수가 있고, 요청을 시작할 때 토큰을 만들고 요청이 마무리되면 토큰을 삭제합니다.
bucket이 꽉 차서 토큰을 만들 수 없는 경우에만 요청을 거절합니다.


#### 2. leaky bucket algorithm

![스크린샷 2022-12-18 22.39.07.png](/files/0a710dbd-83f9-1069-8185-2575ba8e3992?1671370894231)
token bucket algorithm 과 유사한데, 요청을 관리하는 Queue를 만들고 유저는 해당 큐의 버퍼 사이즈만큼 메시지를 계속 보낼 수는 있습니다.
해당 큐에서는 일정 속도로 요청을 처리하면서 queue를 비우게 됩니다.

요청을 처리하는 속도보다 요청이 들어오는 속도가 빠르게 되면(= 큐가 꽉 차게 되면) 넘치는 요청들은 버리는 방식입니다.


#### 3. fixed window counter algorithm

![스크린샷 2022-12-18 22.39.29.png](/files/0a710dbd-83f9-1069-8185-25761c273997?1671370905448)
정해진 시간 단위로 window가 만들어지고 요청 건수가 기록되어서 해당 window의 요청 건수가 정해진 건수보다 크면 해당 요청은 처리가 거부되는 방식입니다.
예를 들어 1분에 4개의 요청이 들어온 경우, 실제 1분에서는 3개의 요청을 처리하고 나머지 하나가 2분 time frame 에서 처리될 수 있도록 하는 식입니다.

~~~
time frame / threshold / request
00:00:00 / 3 / 0
00:01:00 / 3 / ** 3 **
00:02:00 / 3 / ** 1 ** (재시도 했다면?)
00:03:00 / 3 / 0
~~~


> https://dev.to/satrobit/rate-limiting-using-the-fixed-window-algorithm-2hgm

이 방식의 단점은 1분마다 10건의 요청이 들어오는 것을 가정하지만, 00:00:59 에 10건의 요청, 00:01:00 에 10건의 요청이 들어오면
2초만에 20건의 요청이 들어올 수 있는 문제가 있다는 것입니다.

예시로 든 20건은 적긴 하지만 만약 크기가 커지고 bot-net 이 커서 요청이 늘어난다면 문제가 될 수 있습니다.

#### 4. sliding log algorithm

![스크린샷 2022-12-18 22.40.26.png](/files/0a710dbd-83f9-1069-8185-2576f775399f?1671370920056)
fixed window counter algorithm 의 double request count issue를 해결할 수 있는 알고리즘입니다.

특정 유저가 "과거에 언제 요청했는지"를 Sliding Log 방식으로 관리하면서 요청을 처리할지 말지 결정하게 됩니다.
최근 X분 동안의 요청을 하나하나 기록해두고, 지정된 요청건수를 넘어가면 요청을 거부한다. 라고 이해하시면 편할 것 같습니다.

단점으로는 요청기록들을 계속 관리해야되다보니 관리비용이 크다는 점이 있습니다.


#### 5. sliding window algorithm


![스크린샷 2022-12-18 22.40.46.png](/files/0a710dbd-83f9-1069-8185-25773fc439a4?1671370929014)
sliding log와 fixed window counter 알고리즘을 합쳐서 만든 알고리즘입니다.
X분 동안 들어온 요청의 "수"를 저장하고, 저장된 요청 수를 기반으로 요청을 거절할지 말지 결정하게 됩니다.

X분이라는 경계가 계속 이동해서 sliding window 라는 이름이 붙은 것 같습니다.

1분당 10건의 요청으로 제한했다고 가정

00:01:00 window 에 9건의 요청이 들어온 상황
00:01:10 이전까지 4건의 요청이 이미 들어온 상태라면

00:01:15 에 들어온 요청은

9 * 0.75 + 5 * 0.25 = 14.75 > 10 

이므로 처리되지 않습니다.

### 저자의 의견

어떤 방식이 좋을지는 각자의 시스템의 구성에 따라 다르지만, system이 많은 양의 요청들을 처리하고 있는 경우라면 sliding window algorithm 을 추천한다고 합니다.

leaky bucket algorithm은 특정 요청이 처리되지 않는 상태(starved)를 만들 수 있고
fixed window algorithm은 많은 요청들을 거부할 수 있게 되는데

이 둘의 단점을 어느정도 커버할 수 있는 sliding window technic 을 추천한다고 합니다.

> Unlike the leaky bucket algorithm where requests are starved and fixed window counter algorithm where denial of bursts of requests is avoided when you use the hybrid approach of the sliding window technique.