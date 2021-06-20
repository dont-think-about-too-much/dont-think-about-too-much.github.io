---
layout: post
title: "[MongoDB] MongoDB&Mongoose 퍼포먼스 향상시키기"
tags: [Nodejs, Backend]
---

"mongodb/mongoose 쿼리를 더 빠르게 하기 해봅시다!"

> 불필요한 일을 최대한 안하면 됩니다.

### 일단 "lean()"부터 이용하세요.

아마도 "lean"은 쿼리의 성능향상을 위한 최고의 방법일 것입니다. mongoose는 쿼리에 `.lean()` 붙일 수 있게 해줍니다. lean 쿼리를 이용하면 쿼리 객체가 리턴되는 것이 아니라 순수 JSON object가 반환됩니다. 그리고 퍼포먼스 성능이 정말정말 좋아집니다.

+간단히 덫붙이자면, `.lean()`을 붙이지 않으면 검색 결과가 리턴되긴 하나 그 결과값에 쿼리를 더 할수 있는 상태로 리턴됩니다. 리턴값으로 받은 값에 새로운 검색 조건을 붙여 또 다시 쿼리를 진행할수 있으려면, 쿼리가 가능한 상태를 유지하고 있어야 합니다.(단순히 json object라면 몽고db 쿼리가 불가능하죠) 그 상태를 유지하기 위해 여러가지 부가적인 값들, 상태들을 가지고 있어야합니다. 그러니 데이터의 크기가 커지고 성능이 느려지는 것입니다.

From <U>Mongoose docs on lean</U>

> “By default, Mongoose queries return an instance of the Mongoose Document class. Documents are much heavier than vanilla JavaScript objects, because they have a lot of internal state for change tracking. Enabling the leanoption tells Mongoose to skip instantiating a full Mongoose document and just give you the POJO.
> The lean option tells Mongoose to skip hydrating the result documents. This makes queries faster and less memory intensive, but the result documents are plain old JavaScript objects (POJOs), not Mongoose documents.”

다만 `lean()`을 쓰면 mongodb, mongoose에서 지원해주는 것들의 제한이 생깁니다. 아래의 나열된 것들이 lean()과 함께 이용하지 못하는 것들입니다.

- Change tracking
- Casting and validation
- Getters and setters
- Virtuals (including "id")
- `save()` function

위의 나열된 한계점들이 있기 때문에 `lean()`은 **GET 엔드포인트 그리고 `.save()`나 virtuals를 이용하지 않는 `.find()` 작업에 최적의 선택이 됩니다.**

[Faster Mongoose Queries With Lean](https://mongoosejs.com/docs/tutorials/lean.html)

### 커스텀 index를 생성하세요.

MongoDB는 '\_id' 인덱스 말고도 다른 필드들에 인덱스를 생성할 수 있게 해줍니다. 당신이 원하는 필드에 인덱스를 설정해주어 성능을 향상시켜줄수 있습니다.

혼합된 인덱스(Compound indexes)를 생성할 수도 있습니다. Compound indexes는 복수개의 필드로 쿼리를 진행할 때 아주 유용합니다.

`UserModel.find({name: "Kim", job: "developer"})`
위의 쿼리를 진행한다면, 몽고DB는 위의 검색 조건과 맞는 데이터를 찾기 위해 모든 docouments를 둘러보게 됩니다. 이 상황에서 최적화를 하고 싶다면, "type"과 "status"에 index를 추가하면 됩니다.

`UserSchema.index({ name: 1, job: 1 })`
위와 같은 인덱스를 생성해둔다면 기존 쿼리는 관련된 documents만 을 둘러보게 되어 불필요한 탐색은 줄어들게 됩니다.

디테일한 index 정보는 아래를 참고해주세요
https://mongoosejs.com/docs/guide.html#indexes https://docs.mongodb.com/manual/indexes/

### DB request를 최소화합시다. (.populate() 를 최대한 피합니다.)

request가 많아질수록 앱의 반응은 당연히 느려집니다. `redis`를 이용하는 것도 좋은 해결책입니다.

NoSQL 데이터베이스 구조를 최대한 `.populate()`를 이용하지 않게, 그리고 양방향 관계가 되지않게 짜는 것이 좋습니다.(저또한 populate를 최대한 이용하지 않기 위해 복수개의 값을 가진 객체의 배열 값들을 배열이 엄청 커지지않는 한에서 값 통째로 저장해두는 편입니다.)
`populate()`는 RDB의 `join`과 같은 기능이 아니고(db단에서 해주는것이 아닙니다.), 여분의 query들을 생성한뒤 그 값들을 js단에서 합쳐주기 때문에 reference하고 있는 배열의 값이 커질수록 엄청난 성능 저하를 유발합니다.**[배열의 한개의 값마다 한개의 query를 생성하는 것과 같습니다.]** `.populate()`가 정말 필요하다면 `.aggregate()`를 이용하시길 추천합니다.

### .limit()을 이용하여 쿼리 결과값을 제한하세요.

limit을 설정하지 않으면 조건에 맞는 document를 모두 가져오게 됩니다. 그러니 `.limit()`메서드를 이용하여 갯수를 특정하시기 바랍니다.

```js
// 10개만 받기
UserModel.find().limit(10);
```

### .select()를 이용하여 필요한 데이터만 받아오세요.

`.select()`를 지정해주지 않는다면 기본적으로 documents의 모든 값을 가져오게됩니다. document의 모든 값들을 원하는게 아니라면 불필요한 데이터까지 가져오게 되는 겁니다. 이또한 불필요한 작업을 추가적으로 하게 되는겁니다. 그러니 정말 필요한 데이터만 선택해서 받아오시길 추천드립니다.

```js
UserModel.find({ job: "developer" }).select({ name: 1 }); // get only name field
```

### 병렬로 작업을 실행시키셔요~ [정말 강추]

Nodejs에서의 흔한 실수중 하나가 언제든지 async/await를 이용하여 한개의 작업이 끝나고 다른 작업을 실행시키는 것입니다. 아래의 코드가 그런 실수의 예시입니다.

```js
const movie = new Movie({ title: "metrics" });
const car = new Car({ type: "SUV" });

await movie.save();
await car.save();
```

위의 예시를 보시면 movie와 car를 저장시키고 있습니다. 그런데 둘의 저장 작업은 서로 연관된 것이 하나도 없기 때문에 순서대로 작업할 필요가 없습니다. 두 작업이 순서가 필요한 상황이 아닐때라면 병렬적으로 실행하여 성능을 높여주어야 합니다. 아래 코드 처럼 말이죠.

```js
const [movie, car] = await Promise.all([movie.save(), car.save()]);
```

추가적으로 위의 코드는 api 레벨에서만 성능을 향상시켜줬습니다.여전히 2개의 request를 실행하죠. 만약 Database 레벨에서도 성능을 높여주려면 `insertMany()` 혹은 **`bulkWrite()`를** 이용하면 두개의 request보다는 성능이 향상될 것입니다.

### 2개가 이상의 Connection을 만드세요.

단 1개의 connection만을 이용한다면, 그리고 만약 오래걸리는 쿼리가 있다면(ex. 10초이상이 걸리는 쿼리), 다른 쿼리들은 앞의 쿼리가 끝날때까지 기다려야만 합니다. 아무것도 못하게 되는 것이죠. 그러니 1개가 넘는 connection을 만들어두어야만 합니다. 그래야만 다른 요청들을 지연시키지 않고 계속 반응할수 있는 상태로 둘수 있습니다.
