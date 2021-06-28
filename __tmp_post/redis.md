# Redis as Cache

> API 캐싱만을 원한다면 [apicache](https://www.npmjs.com/package/apicache)도 좋다. 래디스보다 훨씬 깔끔하게 캐싱을 이용할 수 있다.

# 캐싱?

캐싱은 자주 사용되는 데이터의 복제본을 저장해둠으로써 어플리케이션 응답 속도를 향상시킨다. Im-memory 캐싱은 데이터를 속도가 느린 디스크에서 저장하지 않고 훨씬 빠른 메모리에 저장함으로써 응답이 빨라진다.

캐싱에는

- CDN
- Database Caching
- API Caching
  등 여러 종류가 있고 알게모르게 대부분의 소프트웨어에서는 캐싱이 쓰이고 있다.

# 왜 필요한가

같은 데이터를 여러명이 요청한다. 그숫자가 계속 늘어난다. 그런데도 그들이 요청하는 데이터는 계속 같다. 캐시가 없다면 같은 데이터를 반환하는데도 API의 코드들을 모두 거쳐야하고, 데이터베이스까지 들렀다오는 일을 **요청수만큼** 하게 된다. 반면에 반환값이 같고 자주 요청되는 데이터를 **캐싱해둔다면** API의 코드를 거치고 데이터베이스를 들렀다오는 일을 요청된 수만큼 하지 않아도 된다. 요청수가 기존에서 1/1000로도 줄어들수 있게 된다.

# 언제 캐싱하면 좋은가

-

# Use Cases

## 캐싱

전통적인 데이터베이스는 빠른 속도 대신 견고한 성능을 목적으로 설계되었습니다. 데이터베이스 캐시는 주로 데이터베이스에서 가져온 복사본을 저장해두고 빠르고 비용이 적게 응답하는데 이용됩니다.

## 세션 스토어/로그인

토큰에 유저 ID가 저장되어있고 권한이 필요한 API요청마다 그 ID로 권한을 검사하게 되는데 그럴 때마다 DB에 GET 검색을 하게 된다.

보통 권한 체크를 할때 아래 쿼리를 날리고 유저가 어떤 권한이 있는지 확인하게 된다.

```
SELECT * FROM users WHERE ID=token.userID
```

유저수가 적다면 상관없지만, DB에 인덱스가 잘되어있다면 크게 문제는 아니지만, 사용자가 늘어나면 문제가 된다.

캐시를 이용하지 않으면 매번 권한이 필요한 API 요청이 들어올 때마다 위 쿼리를 날려 유저 권한을 확인해야 한다. 이렇게 되면 DB에 접근하는 일이 쓸데없이 많아진다.

https://aws.amazon.com/ko/getting-started/hands-on/building-fast-session-caching-with-amazon-elasticache-for-redis/

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

## References

- [[입 개발] 왜 Cache를 사용하는가?](https://charsyam.wordpress.com/2016/07/27/%EC%9E%85-%EA%B0%9C%EB%B0%9C-%EC%99%9C-cache%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94%EA%B0%80/)
- [apicache](https://www.npmjs.com/package/apicache)

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
