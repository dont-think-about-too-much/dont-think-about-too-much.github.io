---
layout: post
title: Redis as Cache [ Node 백엔드 제작시 마주칠 것들 ]
tags: [Nodejs, infra, cache, scaling]
---

# Redis as Cache

> API 캐싱만을 원한다면 [apicache](https://www.npmjs.com/package/apicache)도 좋다. 래디스보다 훨씬 깔끔하게 캐싱을 이용할 수 있다.

# 캐싱?

캐싱은 자주 사용되는 데이터의 복제본을 저장해둠으로써 어플리케이션 응답 속도를 향상시킨다. Im-memory 캐싱은 데이터를 속도가 느린 디스크에 저장하지 않고 훨씬 빠른 메모리에 저장함으로써 응답이 훠~~씬 빨라진다.

캐싱에는

- CDN
- Database Caching
- API Caching
  등 여러 종류가 있고 알게모르게 대부분의 소프트웨어에서는 캐싱이 쓰이고 있다는 점은 알고하고 넘어가자.

# 왜 필요한가

같은 데이터를 여러명이 요청한다. 그숫자가 계속 늘어난다. 그런데도 그들이 요청하는 데이터는 계속 같다. 캐시가 없다면 같은 데이터를 반환하는데도 API의 코드들을 모두 거쳐야하고, 데이터베이스까지 들렀다오는 일을 **요청수만큼** 하게 된다. 반면에 반환값이 같고 자주 요청되는 데이터를 **캐싱해둔다면** API의 코드를 거치고 데이터베이스를 들렀다오는 일을 요청된 수만큼 하지 않아도 된다. 요청수가 기존에서 1/1000로도 줄어들수 있게 된다.

# 언제 캐싱하면 좋은가
### 데이터페이스 속도 향상 
데이터베이스는 어차피 느리긴하지만 row가 많아질수록 더더욱 느려진다. 유저들이 검색하는 데이터는 비슷한 것이 대부분의 트레픽을 차지한다. 그 비슷한 것들의 DB쿼리 결과를 캐시해둔다면 서비스의 전반적인 데이터 전달 속도가 빨라진다. 

### 트레픽이 몰리는 곳들에 캐싱하여 성능 향상: 
갑작스런 트레픽 증가가 발생하는 곳에 여러방법으로 캐시를 둘수 있다. 많은 이용자가 조회하는 데이터를 캐싱하는 것은 당연하고, 어떨때는 크게 중요하지 않은 팔로우수, 조회수와 같은 동적 데이터들을 매번 조회하거나, 매번 +,- 실행하지 않고 write caching으로 모아서 한번에 DB를 수정하는 것도 꽤 큰 도움이 될수 있다.

### 세션 스토어
로그인한 유저에 대한 정보를 매번 검색으로 가져오거나, 로그인한 유저 데이터가 필요한 API들에 대한 요청에 대해 모두 DB에 쿼리하게 된다면 당연히 많은 부하를 주게 된다. 새션에 저장하게 되면 memory만을 들렀다오게 되므로 속도도 훨씬 빠르고 DB에 가는 부하도 줄어든다.

더 깊은 설명은 `redislabs`에서 제공해준다. [Session Management by redislab](https://redislabs.com/solutions/use-cases/session-management/)

### 게임
게임에선 데이터가 실시간으로 관리되어야하기 때문에 당연히 수많은 데이터를 메모리에 저장하게 된다.
게임뿐만 아니라 게임의 스코어보드 또한 캐싱되어있을 것이다. 예로 네이버의 스코어 보드가 그렇다.

![score board](/images/posts/naverscoreboard.png)


### 웹 페이지 캐싱
웹페이지에서 수많은 사용자가 접근하는 페이지에대해서는 화면 전체를 캐싱해둘수도 있고, 부분적으로라도 캐싱하여 조금이라도 더 빨리 제공할수 있게된다. 

![baskcetball](/images/posts/naverbascketball.png)


### 빠른 접근이 필요한 데이터
캐시가 자주 이용되는 데이터에 주로 이용되는게 맞고 그것이 대부분의 이용하는 이유이지만 종종 상황에 따라 빠른 접근이 필요한 데이터를 미리 캐싱해두오 사용하기도 한다.

---

## Q. 세션을 굳이 레디스로 써야하나? 그냥 로컬 메모리로 직접 이용하는 것보다 뭐가 좋지?
1. (제일 중요) 서버가 한대만 돌아간다면 로컬메모리를 직접 써도 무방하다. 하지만 실서비스의 서버를 하나의 인스턴스로 돌리는건 불가능하다.(토이플젝이라면 몰라도) 당연히 여러대의 서버가 로드밸런서에 의해 요청이 분산되어진다. 로그인했던 유저의 정보가 매번 같은 인스턴스로 요청이 갈수 없기때문에 인스턴스간의 새션 정보를 공유할 수 있는 Redis와 같은 것들이 필요하게 된다.

2. 레디스의 메모리 관리가 JS의 메모리 관리보다 훨씬 더 정교하고 효율적이다.

3. 여러 종류의 자료구조를 제공하고 여러 편의기능들을 제공해준다.(pub/sub, atomicity 등)

4. master/slave 구조로 복제본을 만들기 쉽다.


## Write Back (모아서 쓰기)

영어라서 뭔가 싶을텐데 즉각 반영이 필요없는 것들을 캐시에 모아뒀다가 어느정도 쌓이면 한번에 저장/실행하는 것이다.
한번에 저장할 데이터를 임시로 모아두는 곳이 Redis이다. 대표적인 예로 `log`를 예로 들수 있다. 로그는 즉시 반영될 필요가 적은 요소이기 때문에 로그 하나하나 바로 쓰기작업을 할 필요가 없다. 한번에 모아서 저장하는게 비용적으로 큰 이득을 준다. 1000개의 로그를 모아서 저장하는 것과 1000개의 로그 하나하나를 저장하는 것의 차이는 어마어마하게 크다.

## 인증번호 체크

# 어떤걸 캐시로 저장하지?

페이스북이라면 로그인을 위한 유저정보
첫 화면에 보여줄 피드

> 서비스마다 다르다. 정답은 없다. 자주쓰는 것을 파악하자.

## Cache aside (lazy loading)

관리가 힘들다.

## Read through cache

Read through cache is similar to cache aside, the only difference being that it always sends the result from the cache.

## Write through cache

## Write behind cache

# How to choose a caching key

paginated data example: trweets:${pageNumber}:${limit}

# 캐시 서버 확장은 어떻게?

# 캐시는 어디에 어떤걸로 두지? "Resis on EC2" VS "AWS Elasticache"
토이 프로젝트
## References

- [[입 개발] 왜 Cache를 사용하는가?](https://charsyam.wordpress.com/2016/07/27/%EC%9E%85-%EA%B0%9C%EB%B0%9C-%EC%99%9C-cache%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94%EA%B0%80/)
- [apicache](https://www.npmjs.com/package/apicache)
- [Elasticache Or Self Hosted Redis On EC2?](https://www.totalcloud.io/blog/elasticache-or-self-hosted-redis-on-ec2-which-is-the-one-for-you)
<!-- -->
<!-- -->
<!-- -->
<!-- -->

## Write Back

레디스에 몰아두고 한번에 저장.
극단적으로 헤비한 라이트가 있으면 이용

## Race condition 문제가 덜 발생함. 잘못짜면 발생할수는 있음.

# Redis 사용처

## Remote Data Store

- 인증 토큰 등을 저장
- Ranking 보드
- 유저 API limit

# 키를 어떻게 설정할지가 중요하다

- 프리픽스를 붙이는 방식.

# JS에서 long 값은 스트링으로 해야한다. JS에서 숫자의 범위가 크지않아 근사값으로 저장됨.

# 하나의 컬렉션에 몇천개 수준으로 유지하는게 좋다. 너무 많으면 안좋다.

메모리 관리를 잘하자
ON관련 명령어는 주의하자
레플리케이션
권장 설정 팁

# 메모리 관리를 잘하자

- 이름부터 in-memory 데이터스토어다.

- 메모리 꽉차서 에러난적이 있을 것이다.
  피지컬 메모리 이상을 사용하면 문제가 발생 !

- 멕스메모리를 설정하더라도 이보다 더 사용할 가능성이 큼.

# 메모리관리

작은거 여러개 쓰는게 좋다.
큰 메모리를 사용하는 instance 하나보다는 적은 메모리를 사용하는 instance 여러개가 안전함

# On 관련 명령어는 주의하자.

## 싱글스레드라서 긴 시간이 걸리는 명령어 쓰면 안됨.

## Keys는 모든 값을 다가져오는데 한번에 다처리하니 너무 무겁다. scan으로 대체하면 scan중간중간에 다른 요청들이 처리 될수 있다.

# Redis Replication

##

전체 장애의 90% 이상이 KEYS와 SAVE 설정을 사용해서 발생

# 레디스 데이터 분산

Redis as Cache

Cache aside is very easy to implement but very difficult to manage. Cache invalidation is difficult. Whenever the data in the source is updated,
