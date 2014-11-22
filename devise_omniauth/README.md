# Devise OmniAuth 이용하기

이 글을 작성할 시점에서 `Devise` 젬의 최신 버전은 `3.2.4`이다.
`1.5` 버전부터 `OmniAuth 1.0`을 지원하기 시작했다. `OmniAuth` 젬은 웹어플리케이션에서 다양한 인증 제공서비스(`Twitter`, `Facebook` 등)를 사용할 수 있게 해 주는 라이브러리로 생각하면 된다. 최근 `Twitter` 또는 `Facebook`으로 로그인하는 웹서비스를 흔히 볼 수 있는데, 바로 이런 기능을 구현할 수 있게 해 주는 젬 정도로 이해하면 된다.

이러한 `OmniAuth` 젬을 이용하면 다양한 인증 서비스에 연결할 수 있도록 직접 구현할 수도 있는데, 이렇게 구현한 로직을 `strategy`라고 말한다. 이미 다양한 인증 서비스에 쉽게 연결할 수 있도록 업체별 전용 `strategy`가 젬 형태로 제공되고 있다. 예를 들면, `Facebook`에 연결을 하고자할 때는 `omniauth-facebook` 젬을, `Twitter`에 연결하고 싶을 때는 `omniauth-twitter` 젬을 사용하면 되는 것이다.

그렇다면 바로 사용할 수 있는 `strategy` 목록이 궁금할 수 있는데,  [`List of Strategies`](https://github.com/intridea/omniauth/wiki/List-of-Strategies)를 참고하면 이 목록을 볼 수 있다.

이제 본격적으로 각각의 `strategy` 젬을 이용한 실제 구현방법을 알아 보자.

---

_**References:**_

1. [Github : intridea/omniauth](https://github.com/intridea/omniauth)
2. [OmniAuth: Overview](https://github.com/plataformatec/devise/wiki/OmniAuth:-Overview)
3. [List of Strategies](https://github.com/intridea/omniauth/wiki/List-of-Strategies)
