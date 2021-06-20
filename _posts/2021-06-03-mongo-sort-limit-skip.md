---
layout: post
title: "[Error] mongoDB&mongoose sort, skip, limit 중복 에러"
tags: [Nodejs, Backend, JWT]
---

> 문제: 정렬은 잘된다. 그런데 페이지네이션을 하게 되면 나왔던 값이 또 나온다.

예시 mongoose 쿼리는 아래와 같다.

```ts
const users = await this.userModel
  .find({ _id: userId })
  .select("_id name")
  .sort("year") // 연도로
  .skip(skip)
  .limit(3)
  .lean();
```

정렬 기준은 `year`이다. 여기서 문제가 발생했는데, **"첫째 페이지에서 나왔던 데이터가 두번째 페이지에서도 나왔다."** 몇몇 분은 코드를 보자마자 문제의 원인을 벌써 파악했을 수도 있다.

<br>

# 원인

**바로 `year`에 문제가 있는 것이었다.** year는 다큐먼트간의 값이 겹칠 확률이 크다. 유저1의 year이 2021년이고 다른 유저2의 year도 2021이고 수많은 유저들의 year값이 같을 수 있다. 같은 값만을 기준으로 정렬을 해버리니 나왔던 값이 또 나와버린다. **고유값이 아닌 값을 정렬할 때는 두번째 정렬 기준이 꼭 필요하다.**

```ts
const users = await this.userModel
  .find({ _id: userId })
  .select("_id name")
  .sort("year _id") // _id 값을 두번째 정렬 기준으로 설정.
  .skip(skip)
  .limit(3)
  .lean();
```

혹시 다큐먼트간의 첫째 정렬 기준과 둘째 정렬 기준이 모두 같은 값이라면 같은 문제가 발생할 수 있다 그러니 마지막 정렬 기준으로 고유값인 것을 넣으면 좋다.
