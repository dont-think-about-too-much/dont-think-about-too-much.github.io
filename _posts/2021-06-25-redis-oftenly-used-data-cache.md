---
layout: post
title: "[Redis] 자주 검색하는 데이터 캐싱"
tags: [Nodejs, Backend, scaling, cache]
---

캐싱의 대표적인 예가 홈 화면이지 않을까 싶다.
대부분의 홈화면은 자주 바뀌지않는다. 그렇기 때문에 캐싱을 하면 보다 빠르게 유저에게 응답할 수 있게 된다.

멜론 서비스와 같은 아티스트 정보 여럿을 보여주는 홈화면이라고 가정해보자.

우리가 테스트할 API는 `getArtists`.<br>
getArtists API는 6명의 아티스트 정보를 보내주는 API다. queryString에 따라 값이 달라진다. 하지만 홈페이지를 처음 들어왔을 때에는 매번 같은 정보를 보내준다.

아래 예시는 `getArtists`에 캐싱이 추가되기 전 코드다.

```ts
// 캐싱 전
@route('/test/redis/get-artists')
@GET()
@before([])
async getArtists(ctx: Koa.Context) {
    // Make DTO and pass to Service Layer
    let queryParams: IArtistQueryParams = ctx.request.query;

    // .getArtist() 는 6명의 아티스트 정보들(이름, 사진등 여러가지)을 가져온다.
    const results = await ArtistService.getArtists(queryParams);

    // Pass Response to client
    ctx.response.status = HttpStatus.OK;
    ctx.response.body = {
        pagination: results.pagination,
        data: {
            artists: results.artists
        }
    };
}
```

postman으로 요청을 해보면

postman 요청
![api-cache-result1](/images/posts/redis-api-cache1.png)
postman 요청
![api-cache-result2](/images/posts/redis-api-cache2.png)
postman 요청
![api-cache-result3](/images/posts/redis-api-cache3.png)

트레픽이 없는 테스트 상황에서는 응답 속도가 `160-170ms`나오고 있다.

<br>

## 이제 아주 간단한 캐시를 추가해보자.

### 1. 레디스 설정부터 한다

```ts
// loader/redis
import * as redis from "redis";

export class RedisLoader {
  constructor() {
    this.client = redis.createClient({
      host: Env.get().REDIS_HOST,
      port: Env.get().REDIS_PORT,
    });
    this.client.on("error", () => {});
  }

  client: redis.RedisClient;
}
```

`host`와 `port`를 같은 곳으로 하게 되면 여러 인스턴스에서, 여러 마이크로서비스에서 같은 캐시 정보를 이용할수 있다. (캐싱 공유에 대해선 다다음 글에서 작성할 예정)

<br>

### 2. 설정한 레디스 가져와서 이용하자.

```ts
import * as redis from 'redis';
import { promisify } from 'util';
import { RedisLoader } from '../loaders/redis.loader';

const redis = new RedisLoader();

const client = redis.client;

// 'redis' 모듈에서 제공하는 .get() 과 .set()은 콜백형태로 제공해준다.
// 프로미스화 시켜서 이용하는 것이 편하다.
const getAsync = promisify(this.client.get).bind(this.client);
const setAsync = promisify(this.client.set).bind(this.client);


@route('/test/redis/get-artists')
@GET()
@before([])
async getArtists(ctx: Koa.Context) {

    // Make DTO and pass to Service Layer
    let queryParams: IArtistQueryParams = ctx.request.query;

    // .getAsync() 의 인자가 뭔가 싶다면 아래로 내려가서 .setAsync의 인자를 보자.
    const checkCached = await getAsync(`api-getArtists-${queryParams.category}-${queryParams.type}`);

    if (checkCached) {
        const cacheToJSON = JSON.parse(checkCached);

        ctx.response.status = HttpStatus.OK;
        ctx.response.body = cacheToJSON;

        return;
    }

    // .getArtist() 는 6명의 아티스트 정보들(이름, 사진등 여러가지)을 가져온다.
    const results = await ArtistService.getArtists(queryParams);

    // Pass Response to client
    ctx.response.status = HttpStatus.OK;
    ctx.response.body = {
        pagination: results.pagination,
        data: {
            artists: results.artists
        }
    };

    // 캐시된 값이 없으면 캐시 저장.
    // #3 key를 검색 조건으로 설정해뒀다.
    await setAsync(`api-getArtists-${queryParams.category}-${queryParams.type}`, 'EX', 60 * 60)
}
```

<br>

### #3 redis를 이용할때 중요한 것중 하나가 key를 잘 정리하는 것이다.

정해진 규칙을 이용해야 관리가 편해진다. 이번 예제에서는 `key: value`형태의 자료구조를 이용했고 `key`를 **API이름과 검색조건**을 합쳐서 설정해뒀다. 검색 조건에 **category**와 **type**이 있어서 `-`로 연결해뒀다. 카테고리가 댄스이고 타입이 아이돌이라면 `api-getArtists-dance-idol`, 카테고리가 소울이고 타입이 입력되지않았다면 `api-getArtists-soul-null` 이렇게 저장된다.

### setAsync()에 있는 'EX'와 60\*60

`'EX'`는 expire의 약자로 데이터가 만료되는 시간을 설정할 때 이용한다.
`60*60`은 만료 시간으로 기본 단위가 초이므로 60\*60은 1시간이다.

<br>

## 캐싱 후 응답 속도 변화

아주 간단한 세팅이다. 실행시켜보면
첫 API요청에서는 기존과 같은 응답속도로 반응해주지만 그 다음부터는 'EX'시간이 되기전까지 아래의 응답 속도를 보인다. 기존에 160~170ms하던 응답 속도가 `10ms`근처로 줄어들었다.

**DB에 트레픽이 몰릴수록 api요청 응답속도는 160에서 더 오래걸리게 된다. 그런 상황에서 api캐싱이 되어있기만 하다면 DB를 들르는 일이 없기 때문에 아무리 DB에 몰려도 캐싱이 되어있는 API는 10ms근처로 응답을 할수 있다.**

![api-cache-result4](/images/posts/redis-cache-result7.png)
![api-cache-result5](/images/posts/redis-cache-result8.png)
![api-cache-result6](/images/posts/redis-cache-result6.png)

<br>

### 마무리

캐싱은 정말정말정말 중요하다. 아주 간단한 캐싱으로 첫 페이지를 보여주는 성능이 월등히 빨라진다.
서비스에 있어 홈 화면이 느리게 나온다면 서비스의 신뢰가 떨어질수 있다고본다. 그러므로 무조건 홈화면에는 캐싱을 하자.

## References

- [Caching strategies to speed up your API](https://blog.logrocket.com/caching-strategies-to-speed-up-your-api/)
