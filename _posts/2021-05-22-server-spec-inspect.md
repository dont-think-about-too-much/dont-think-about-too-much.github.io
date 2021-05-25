---
layout: post
title: Node 백엔드 spec 정하기 [ Node 백엔드 제작시 마주칠 것들 ]
tags: [Nodejs, renewal, spec]
---

> 스펙을 정립하고 가자.

<br>

## Nodejs 버전은?
>LTS(Long Term Support): 장기 지원 버전. 보통 짝수 버전들이 LTS이다.

LTS만 쓰면 괜찮다고 본다. 다만 너무 최신은 또 좀 그렇고.<br>`14`로 선택.

---

## Framework?

개인적인 경험으로 `koa`, `express`의 차이는 이용하는 몇몇 모듈이 달라지는 것뿐이라고 판단된다. 차이가 거의 없으니 지금까지 이용했던 koa를 이용한다. 성능도 더 좋기도 하다. <br>
개인적으로는 `nest`를 선호한다. 하지만 다음 사람이 코드를 인수인계받았을 때 비용이 덜드는 게 더 좋지않을까 싶다. 내 자리를 메꾸는 사람이 시니어는 분명히 아니다. 그렇다면 express만 해봤을 확률이 높다. 회사를 위해 express 혹은 koa를 이용하기로 !

---

## Nginx 왜써요?

**서버에서 너무 많은 일을 하는게 좋진않자나 !**<br>
이게 정답이다. 사실 process managing, https처리 다 할수 있다. 하지만 백엔드 서버는 요청들이 계속 들어오게 되고 우리가 만들어놓은 비지니스 관련 작업들을 처리하는 것이 더 중요하다. `https 처리`, `Load Balancing`까지 하면 서버에서 해야할 일이 너무 많아지고 요청에 대한 응답 속도도 당연히 느려진다.<br>
그리고 Node에서 다 할수 있다고 하지만, Nginx가 Load Balancing, https 처리하는데에 더 많은 노력을 쏟았을 것이고 더 성능이 좋을 것이니. 세무사에게 세무일을 맡기듣 전문가에게 맡기자.

지금의 서비스에서는 HTTPS, Load Balancing 이 두가지를 위해 이용한다.

---

## Pm2?

Nginx에서도 말했듯이 노드 에서도 clustering(여러개의 서버를 만들어주는 것)을 해준다. 하지만 그 전문가인 pm2에게 맡기자.

PM2는 이름부터가 **Process Manager**이다. 설정에 따라 process 갯수를 정할수 있고, **무엇보다 중요한 것은 서버가 어떠한 에러로 꺼지게 된다면 pm2가 알아서 다시 켜준다. 계속 서버를 살아있게 만들어준다.**

---

## INFRA
"최대한 AWS를 이용한다."

이용자 수에 따라 서버 적정 인프라를 설명해주는 글이다 ["천만 사용자를 위한 AWS 클라우드 아키텍처 진화하기"](https://www.slideshare.net/awskorea/aws-cloud-architecture-evolution-for-one-thousand-users-changsu-lee)

내 개인적인 주관보단 저 글이 훨씬 좋다.

---

## CICD
"Github Action"

내 주변에서 제일 많이 쓰는 것 같다. 퍼블릭 레포지토리면 공짜기도 하니깐 간단히 이걸로 정했다.