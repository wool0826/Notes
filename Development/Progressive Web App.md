## Progressive Web App?

최근에 애플이 iOS 16.4 베타에서 "웹 푸시"를 지원한다고해서 뭔가 싶어서 찾아보았습니다.

이름에서 드러나다싶이, 웹 브라우저를 기반으로 동작하는 어플리케이션을 의미합니다.

기존에는 Android, iOS Native 앱을 만들어서 마켓에 올리면 사용자는 다운받아서 사용하는 식으로 어플리케이션의 배포가 이루어졌는데요.
Web browser를 기반으로 동작하지만 Native App 같은 경험을 제공해줄 수 있습니다.

ex) https://app.starbucks.com/

## PWA 의 조건

1. HTTPS
2. Web App Manifest
3. Service Worker

### Service Worker ?

서비스 워커는 브라우저의 캐시를 뒤져서 클라이언트에게 제공하는 역할을 하는 worker 입니다.
=> 네트워크가 불안정하더라도 Service Worker를 통해 데이터를 제공해줄 수 있게 됩니다.

> 문서를 보다보면 흠,, 뭔가 잡다한 일을 해주는 백그라운드 스레드가 하나 더 생기는 느낌? 인 것 같네요.

## Advantages of PWA

### 1. Automatic Updates

Native App 이 아니므로, 유저가 따로 설치/업데이트할 필요가 없습니다.
항상 유저는 최신상태의 어플리케이션을 사용할 수 있게 됩니다.

> 유저의 action 없이도 최신의 기능을 제공해줄 수 있다는 점은 장점으로 보입니다.

### 2. Reduction in development costs

다양한 플랫폼에 대응하기 위해 다양한 개발 리소스를 가지지 않아도 됩니다. (개발자의 입장에서는 단점일수도...?)
여러 개의 codebase를 가질 필요없이 하나의 codebase를 가지고 개발을 진행할 수 있기 때문에 개발을 효율적으로 진행할 수 있습니다.

> PWA + Native App 을 제공할 때, 하나의 Codebase를 통해서 개발을 진행할 수 있다... 는 의미로 이해했습니다.

### 3. Adaptable to poor network conditions

PWA는 Service Worker 를 사용하는 것이 필수조건이다보니, Service Worker로부터 얻을 수 있는 이점을 그대로 누릴 수 있습니다.
네트워크 상황이 불안정하더라도 안정적으로 서비스의 제공이 가능해지고, 캐시를 사용하므로 성능부분에서도 이득을 볼 수 있습니다.

### 4. App Aggregators are not needed

마켓을 통해 어플리케이션을 배포하지 않아도 되므로, 수수료 등의 추가적인 비용을 지불하지 않아도 됩니다.

> 수수료가 꽤 쎈데 이 부분에서 많이 끌리지 않을까 싶네요.

### 5. Updates and loading times are faster

Service Worker API 를 통해 어플리케이션을 로딩하게 되는데, 표준 API 보다 짧은 시간에 로딩이 가능하다고 합니다. (..?)

> 캐시를 사용하므로 속도가 빠르다는 것을 의미하는 것 같네요.

### 6. Accessible

검색엔진으로 접근이 가능해집니다.
접근할 수 있는 경로가 늘어나게 되므로, 경제적인 측면에서 이득일 수 있습니다.

### 7. Fast, safe and offline work

PWA 는 Service Workers 기술을 사용하기 때문에 오프라인환경에서도 빠르고 안전하게 사용할 수 있습니다.

### 8. UI/UX Improvements

Native App 의 looksAndFeel 을 가지지만 속도, 반응형 등의 UI/UX 부분에서 Native App이 가질 수 없는 장점이 존재합니다.

## Disadvantages of PWA

Medium의 글은 장점밖에 없어서 단점은 따로 찾아보았는데요. medium 글에서는 장점으로 제시된 부분을 단점으로 제시하는 것도 있네요.

### 1. No Access to apps stores

App Stores 에서는 접근할 수 없으므로, 일반적인 유저의 접근이 그렇게 쉽지는 않습니다.
예시로 스타벅스의 PWA가 존재한다는 걸 이 글을 작성하면서 처음 알았습니다.

### 2. Fewer Functionalities

Native App 에서는 쉽게 가능한 기능들인 연락처에 접근하거나, 블루투스/NFC 등의 기능을 사용할 수 없습니다.
쉽게 말해, 하드웨어와 관련된 기능은 사용할 수 없습니다.

### 3. Still in the development phase

Apple 이 관련 기능을 지원하지 않는 부분을 예시로 들면서, 아직 PWA가 돌아갈 수 있는 플랫폼들이 개발 중이라는 것을 언급합니다.

### 4. Performance

Native App 보다는 성능적으로 한계가 있을 수밖에 없는 구조라고 이야기합니다.
=> PWA 를 wrapping 한 Native App을 만드는 것을 권장하는 것 같네요.

## 참고문서

https://medium.com/managing-digital-products/what-are-progressive-web-apps-benefits-types-and-cost-cb3c4bfca30e
https://bscnote.tistory.com/33
https://developer.mozilla.org/ko/docs/Web/Progressive_web_apps/Offline_Service_workers
https://www.d-tt.nl/en/articles/pwa-progressive-web-apps-pros-cons