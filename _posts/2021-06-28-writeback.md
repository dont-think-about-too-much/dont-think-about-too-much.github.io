---
layout: post
title: "[Redis] 조회수 +1을 매번 DB 요청할순 없자나/ Write Back Cache"
tags: [Nodejs, infra, cache, scaling, Redis]
---

"viewCount를 최적화하는 전반적인 흐름일뿐 Best Practice가 아님을 주의!"

### 상황
메인 페이지에 노출되고 있는 Post들을 캐싱하여 제공하고 있다. 그런데 Post들을 캐싱해버리면 개별 Post들의 조회수를 증가시킬수가 없게 된다. post의 데이터들을 가져오는 API요청에서 조회수를 증가시켜왔는데 API가 캐싱되어버리니 조회수를 증가시키는 과정이 생략된다

### 해결 방법
개별 데이터들(조회수를 측정하는 데이터들)이 캐싱되어 이용된다면 조회수 증가 작업 또한 redis에서 따로 모아둔 뒤 **`Write-Back`**해주어야 한다.

<br>

# Write Back Cache
쓰기작업에 있어 데이터를 먼저 캐시(Redis)에 저장한 뒤 일정 시간이 지난 후 한번에 캐싱된 데이터를 실제 DB에 저장하는 방식.

데이터의 정확성이 크게 중요하지 않은 부분에서 이용할수 있다. Youtube 조회수도 `Write Back`으로 구현되어있는 듯하다. 조회수가 한번에 바로 올라가지 않는다. 사용자가 그렇게 많은 서비스에서 조회수 하나하나마다 DB를 접근하는 것은 당연히 비용적으로 손해다. DB에 요처이 줄어드니 그동안에 다른 중요한 일을 처리하도록할 수 있다.

단점은 Redis가 예기치 않게 꺼지면 임시적으로 저장하고 있던 데이터들(ex. 조회수)가 날라갈수 있다.(그러니 중요한 데이터는 넣지말자.)

---

<br>

>아주 간단한 조회수 Write Back Cache 구현 과정을 보자.

# HASH를 이용한다.
Redis에서 viewCount를 저장하려면 `hash`를 써야한다. 기본으로 쓰던 `key: value`는 스트링만 저장되므로 `+1` 하는 게 안된다.

`viewCounts`라는 헤시 테이블이 있다. 그리고 그 해시 테이블에는 `field: value`로 데이터 ID와 조회수를 저장하려고 한다. field를 조회하는 데이터의 ID로 넣을 것이고, value로 조회수를 넣는다.

## 0. redis 메서드부터 보고가자
- `.hset()`: 셋팅안하고 바로 `.hincrby()`로 증가시키면 알아서 셋팅이 되어서 오늘 쓸일은 없다. ex) .hset("newHash", "field1", 0);
- `.hincrby()`: 입력하는 숫자만큼 증가한다. ex) .hincrby("newHash", "field1", 1);
- `.hdel()`: 필드를 지운다. ex) .hdel("newHash", "field1");
- `.hget()`: 필드 값을 가져온다. ex) .hget("newHash", "field1");

<br>

## 1. 조회수 +1 을 캐시에 쓴다. (DB에 쓰지않는다.)
아래는 특정 글을 읽어오는 API이다. (조회수 캐싱 추가되기 전)

```typescript
@route('/post/:id')
@GET()
@before([])
async getPostDetail(ctx: Koa.Context) {
    const client = redis.client;

    const postID = ctx.params.id;

    // 캐싱 되었는지 확인.
    const checkCached = await client.get(`api-getPost-${queryParams.category}-${queryParams.type}`);

    // 캐싱되어있다면 캐싱된 데이터를 바로 반환.
    if (checkCached) {
        const cacheToJSON = JSON.parse(checkCached);

        ctx.response.status = HttpStatus.OK;
        ctx.response.body = cacheToJSON;
        
        return;
    }

    const results = await PostService.getPostAndIncreaseViewCount(postID);

    ctx.response.status = HttpStatus.OK;
    ctx.response.body = {
        pagination: results.pagination,
        data: result
    };

    await client.set(`api-getPost-${queryParams.category}-${queryParams.type}`, 'EX', 60 * 60)
}
```

<br>

### 이제는 찾는 데이터가 캐싱되어있다면 `viewCount +1`을 캐시에 실행시킨다.(조회수 캐싱 추가된 후)

```typescript
@route('/post/:id')
@GET()
@before([])
async getPostDetail(ctx: Koa.Context) {
    const client = redis.client;

    const postID = ctx.params.id;

    const checkCached = await client.get(`api-getPost-${queryParams.category}-${queryParams.type}`);

    if (checkCached) {
        // post 가 캐시되어있다면 viewCount도 캐시에서 +1 한다.
        // 미리 hash table에 데이터를 세팅해두지 않아도 처음부터 1을 증가시키면 알아서 생성부터 한다.
        await this.hincrby("viewCounts", `${postID}`, 1)

        const cacheToJSON = JSON.parse(checkCached);

        ctx.response.status = HttpStatus.OK;
        ctx.response.body = cacheToJSON;
        
        return;
    }

    const results = await PostService.getPostAndIncreaseViewCount(postID);

    ctx.response.status = HttpStatus.OK;
    ctx.response.body = {
        pagination: results.pagination,
        data: result
    };

    await client.set(`api-getPost-${queryParams.category}-${queryParams.type}`, 'EX', 60 * 60)
}
```

이렇게 매번 읽어올 때마다 DB에 조회수 `+1`을 요청하지 않고 `cache`에 저장해둔다. 사용자가 많아질수록 조회수가 많아지므로 당연히 성능적으로 큰 이득이다.

<br>

## 3. cache에 모인 조회수를 일정시간마다 DB에 쓴다.
1. OS에서 **`cron`**으로 정기적으로 실행시켜도 된다. 혹은 API서버 안에서 **`scheduler`**로 정기적으로 실행시켜도 된다.
2. 조회수 업데이트가 완료됬다면 캐시에 저장되어있는 viewCount를 제거한다. **`.hdel()` 이용**
```typescript
await client.del('viewCounts', 'postID1');
```

<br>

## 마무리
다시 한번 강조하면, `Write Back Cache`를 이용할수 있는 경우는 어느정도 범위의 오차가 있어도 되는 데이터에 한해서다. 만약 조회수가 100% 맞아야한다면, 이 방식을 이용하면 안된다.

<br><br>

## References
- [stackoverflow /redis as write-back view count cache for mysql](https://stackoverflow.com/questions/16761898/redis-as-write-back-view-count-cache-for-mysql)
- [Redis를 사용한 View Count, 방문자 수 관리하는 효과적인 방법은?](https://webisfree.com/2017-11-13/redis%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%9C-view-count-%EB%B0%A9%EB%AC%B8%EC%9E%90-%EC%88%98-%EA%B4%80%EB%A6%AC%ED%95%98%EB%8A%94-%ED%9A%A8%EA%B3%BC%EC%A0%81%EC%9D%B8-%EB%B0%A9%EB%B2%95%EC%9D%80)
- [HINCRBY key field increment](https://redis.io/commands/HINCRBY)