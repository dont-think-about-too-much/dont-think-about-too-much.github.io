---
layout: post
title: "[MongoDB] $regex vs $text"
tags: [Database, Backend]
---

저번달에 새로 추가한 기능이 커뮤니티와 유사한 것이었다. 워낙 촉박하게 만들어야해서 + 다큐먼트가 많이 생기지않을거라고 예상해서 **$text** 인덱스, 쿼리를 생각하지 않고 **$regex**만을 이용해서 만들어뒀다.
근데 새 기능을 오픈하자마자 예상보다 훨씬 빠르게 데이터가 쌓이고 있고, 인기가 많아지고 있다. **지금은 괜찮다만 한달만 있어도 $regex 쿼리가 느려질 것이 뻔히 보여 다른 방안이 필요하다는 생각이 들었다.** 이 문제의 대응책으로 *$text*로 변경해야하나 고민하던 차에 둘의 차이를 정리해보려고 한다.

-----
<br>

몽고DB에서 String의 부분을 가지고 검색할때 이용하는 두가지가 **"$text"와 "$regex" 이다.** 간단한 용례를 이렇다.

### $regex 용례
```ts
// $regex

const searchResult = await Model.find({
    $text: { $search: "apple", $caseSensitive: false }
});
```

<br>

### $text 용례
```ts
// $text

const searchResult = await Model.find({
    $regex: name, $options: "i"
})
// name은 정규표현식으로 검색할 필드, $options에 들어간 i는 caseSensitive: false를 의미.
```

딱봐도 어렵지않다. (몽고는 역시나 aggregate가 좀 헷갈리고 설계가 제일 힘들다.)
바로 둘의 차이로 넘어가자.

<br>

# $regex 와 $text의 차이.

# 1. 초기 셋팅
용례에서 볼수 있듯이 *$regex*는 어떤 필드를 검색할지 쿼리문에 적은 후 검색한다. 그래서 딱히 셋팅해둘게 없다.

그런데 *$text*는 검색할 필드에 대해 미리 text 인덱스를 설정해놓아야만 한다. 이름과 이메일을 $text를 이용하여 검색하고 싶다면 아래처럼 인덱스 설정이 미리 되어있어야 한다.

```js
UserSchema.index({email: “text”,name: “text”}, {weights: {email: 1,name: 2}})
```
검색하는 테이블에 text 인덱스가 하나도 설정이 되어있지 않다면 
`text index required for $text query` 에러 메시지가 나타난다.

<br>

# 2. Full Text Search VS Partial Text Search
## $regex는 partial text search가 된다.
*$regex*는 당연하게도 정규표현식 검색이다보니 부분적으로 단어가 맞아도 검색 결과에 포함된다. `name: apple`이 저장되어있을 때 `app`만 입력해도 apple을 찾는다. `ppl`을 입력해도, `pp`만 입력해도 말이다.

## $text는 partial text search가 안된다.
검색하는 단어가 100%(Full)로 맞는 것이 있어야 한다.
반면 $text는 Full text search이다보니 검색하는 값이 단어(스페이스) 기준으로 100% 같아야 한다. 예를 들면, `content : "I have a apple"` 이 데이터가 있을 때 `{ $text: { $search: "app" } }`로 검색하면 결과값으로 아무것도 받을 수 없다. "I have a apple"를 스페이스 기준으로 분리된 체로 100% 매칭되는 것이 있어야만 찾을수 있다. 

> text index tokenizes and stems the terms in the indexed fields for the index entries. / [mongodb docs](https://docs.mongodb.com/manual/core/index-text/)

**$text 쿼리로는 "apple"을 "app"입력만으로 찾을수 없다.**

> 이렇게 따져보면 "$regex가 훨씬 디테일하게 값을 찾아서 좋은거 아냐?"라고 물을수 있다. 하지만 데이터가 많아진다면? Index를 이용해서 최대한 검색 범위를 줄이는 작업이 필요해진다. 근데 $regex는 index를 이용할 수 없다. 테이블내의 모든 데이터를 쿼리때마다 검색해야한다는 것이다. $regex는 데이터의 갯수가 어느정도 한계가 있을 때에만 이용할 수 있다. 혹은 다른 인덱스를 이용하여 값을 1차적으로 거른뒤에 걸러진 값내에서 regex를 이용하거나 말이다.

document가 많아지면 어쩔수 없이 $text 검색을 해야한다. $regex로는 너무 느려지니 아무것도 할수 없게 되버린다.

<br>

# $regex는 너무 느리고 $text는 디테일이 떨어진다. 대응책은?

## 1. $text 검색부터하고 검색값이 없으면 $regex 검색을 해서 보완.
[stacckoverflow](https://stackoverflow.com/questions/44833817/mongodb-full-and-partial-text-search) 이 링크에 답변을 보면 $text와 $regex를 같이 쓰는 것도 꽤 괜찮다고들 한다. $text 검색이 $regex보다 훠~얼씬 빠르니 **$text로 먼저 검색해보고 결과값이 시원치않으면 어쩔수 없이 $regex 검색을 하라는 것이다.**

아래 코드는 'Recardo Canelas'이라는 개발자가 남긴 답변인데 여럿이 이 답글에 동의하고 있다.
```ts
import mongoose from 'mongoose'

const PostSchema = new mongoose.Schema({
    title: { type: String, default: '', trim: true },
    body: { type: String, default: '', trim: true },
});

PostSchema.index({ title: "text", body: "text",},
    { weights: { title: 5, body: 3, } })

PostSchema.statics = {
    searchPartial: function(q, callback) {
        return this.find({
            $or: [
                { "title": new RegExp(q, "gi") },
                { "body": new RegExp(q, "gi") },
            ]
        }, callback);
    },

    searchFull: function (q, callback) {
        return this.find({
            $text: { $search: q, $caseSensitive: false }
        }, callback)
    },

    search: function(q, callback) {
        this.searchFull(q, (err, data) => {
            if (err) return callback(err, data);
            if (!err && data.length) return callback(err, data);
            if (!err && data.length === 0) return this.searchPartial(q, callback);
        });
    },
}

export default mongoose.models.Post || mongoose.model('Post', PostSchema)
```

<br>

## 2. Elasticsearch를 이용하자....
Elasticsearch는 Full, Partial 모두 검색된다. ㅎㅎ

<br>

---
---

### 참고 자료
- 제일 볼만한 내용 [MongoDB Full and Partial Text Search](https://stackoverflow.com/questions/44833817/mongodb-full-and-partial-text-search)
- [How to Speed-Up MongoDB Regex Queries by a Factor of up-to 10](https://medium.com/statuscode/how-to-speed-up-mongodb-regex-queries-by-a-factor-of-up-to-10-73995435c606)
- [Fuzzy search with MongoDB and Python](https://medium.com/xeneta/fuzzy-search-with-mongodb-and-python-57103928ee5d)
- [MongoDB text search partial words](https://sqlserverguides.com/mongodb-text-search-partial-words/)
