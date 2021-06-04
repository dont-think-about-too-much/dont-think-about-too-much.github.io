---
layout: post
title: "입력되지 않는 값을 undefined로 그냥 둘까, null로 저장할까?"
tags: [Node, Backend, JS]
---

>**문득 고민**<br>
`optional` 조건이고, `default` 값이 설정되지 않은 데이터가 입력되지않을 때는 어떤 상태로 DB에 저장해놓는게 최적이지?
그냥 undefined로 두나? 아니면 null로 저장해야하나? 

DB에 **값이 없음**을 저장하는 방식에는 여러가지가 있는듯 하다.
- null
- undefined
- ''(빈 문자열)

이 세가지가 대표적이다. 뭐가 최적일까 고민을 해봤다.

<br>

## 나만의 결론은 **"입력되지않은 데이터는 무조건 `null`로써 저장하는 것이다."**<br>
**이유 1**. JS에서의 `undefined`와 `null`의 정의 차이<br>
**이유 2**. 쿼리의 결과값의 명확함<br>
**이유 3**. 백엔드 개발자의 편의

<br>

## 이유 1. JS에서의 `undefined`와 `null`의 정의 차이

### `undefined`는 단어의 뜻대로 **"정의되지않았다"**를 나타낸다. 
값을 할당하지 않으면 `undefined`이다.

```js
let a; // 할당하지 않음
>> undefined
console.log(a)
>> undefined
```

<br>

### `null`은 **"의도적으로 비어있음"**을 나타낸다.

둘의 **정의** 차이만으로도 뭐가 나은지 개인적인 판단이 섰다. **웹서비스에서 대부분의 입력되지않은 값들은 의도적으로 비워둔 값들이다.** 그러니 `null`로 정의하는게 조금이라도 더 옳다고 판단된다.

---

<br>

## 이유 2. 쿼리 결과값의 명확함
쿼리면에서도 검색하는 값이 `undefined`면 필드의 키, 값이 나오지 않는다. 반면 null로 할당해놓으면 쿼리시에 필드 키와 null 값이 나온다. **그게 훨씰 더 데이터를 받아서 쓰는 사람 입장에서 명확하다.** 


### missing 데이터를 **`undefined`** 상태라면
```json
{
    "success": true,
    "data": [{
        "title": "What is null?",
        "author": "kim",
        "age": 21,
        "createdAt": "20210101"
    },{
        "title": "What is number?",
        "author": "kim", 
        // 이 데이터에는 age가 안나온다.
        "createdAt": "20210101",
    },{
        "title": "What is undefined?",
        "author": "kim",
        // undefined라면 age 숨겨저 버린다.
        "createdAt": "20210101",
    }]
}
```
<br>

### missing 데이터를 **`null`**로 저장해두었을 때.
```json
{
    "success": true,
    "data": [{
        "title": "What is null?",
        "author": "kim",
        "age": 21,
        "createdAt": "20210101"
    },{
        "title": "What is number?",
        "author": "kim",
        "age": null, // null로 저장해두면 분명하게 null로 나온다.
        "createdAt": "20210101",
    },{
        "title": "What is undefined?",
        "author": "kim",
        "age": null, // null로 저장해두면 분명하게 null로 나온다.
        "createdAt": "20210101",
    }]
}
```

내가 보기엔 null 값이 반환되는게 훨~씬 명확하다. undefined일때는 age는 왜 안나오지? 고민을 할수도 있다.

---
<br>

## 이유 3. 백엔드 개발자의 편의
백엔드에서 값의 존재 여부를 체크를 잘체크해야한다. 하지만 사람이 완벽하진 않다. 그러다보니 **`Cannot read property '' of undefined`**이 에러를 아주 자주 만났을 것이다. 입력되지 않은 값들을 null로 정의해둔다면 이 에러를 덜 마주치게된다.

---

<br>

## Reference

- [mongoose가 'undefined'를 처리하는 방식에 대해](https://blog.ull.im/engineering/2019/03/22/mongooses-undefined-handling.html)<br>
- [When storing missing data, should I store them as NULL?](https://discuss.codecademy.com/t/when-storing-missing-data-should-i-store-them-as-null/349730)