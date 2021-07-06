---
layout: post
title: "[JS] Better Javascript Code 모음"
tags: [Refactoring]
---

> 여러 글들에서 보게 되는 JS 작성 팁들을 모아두는 글이다.

# 함수 선언식보다 함수 표현식을 이용한다.
함수 선언식은 호이스팅이 된다. 도움이 될때도 많지만. 코드가 이상한 흐름으로 돌아갈 수도 있다. 어디에 있는 함수를 이용하는지 정확히 안뒤 이용하는 습관을 들이자.

```js
doSomething(); // 작동한다 하지만 이상하다.
doAnothor(); // 참조에러가 발생한다.

function doSomething() {
    // some code
}

const doAnothor = function() {
    // some code
}
```

<br>

# 최대한 "const"와 불변 객체를 이용한다.
불변 객체를 선호해라. 잘바뀔수 있는 데이터를 이용하는 것은 전반적으로 프로그램의 불안정성을 가져온다.

<br>


# 순수 함수를 선호한다.
함수가 인자로 들어오는 값을 최대한 바꾸지 않게 설계하자. 새로운 값을 리턴하도록 해야한다.
```js
// 이함수는 인자로 들어오는 array를 변형시킨다.
const doubleArrayValuesImpure = (array) => {
    for(const index in array) {
        array[index] *= 2;
    }
}

// 인자로 들어오는 array를 변환시키지 않고 새로운 값을 반환한다.
const doubleArrayValuesPure = (array) => array.map(number => number * 2)
```

<br>


# 화살표함수를 바로 쓰지않고 함수 표현식으로 변수를 정의한 뒤 이용한다.
`map`, `filter` 등과 같은 메서드에 넣을 화살표함수를 함수표현식으로 미리 정의한뒤 함수 이름만 넣어주자.

```ts
const names = ["Kim", "Lee", "Park"];

// 화살표 함수를 .map() 에 바로 쓴다.
const lowerCaseNames = names.map(name => {
    return name.toLowerCase();
});
```

```ts
const names = ["Kim", "Lee", "Park"];

// 화살표 함수를 바로 넣지 않고 따로 정의해준뒤 전달해준다.
const namesToLowerCase = name => name.toLowerCase();

const lowerCaseNames = names.map(namesToLowerCase);
```

<br>


# if문이 중첩되는 것과 if 조건이 길어지는 것을 피한다.
백엔드 초보때는 한번쯤 아래와 같은 코드를 작성했을 것이다.
```ts
if(request.body) {
    if (request.body.someField) {
        // some code
    }
}
```

어느 순간부터 조금 성장하면 아래처럼 `&&` 연산자를 쓰게 된다.
```ts
if(request.body && request.body.someField) {
    // some code
}
```

그리고 조금더 깔끔하게 코드를 작성하고 싶은 욕심이 생기면 아래와 같은 코드까지도 간다.

```ts
const hasBodyAndSomeField = (request) => {
    if (request.body && request.body.someField) {
        return true;
    }

    return false;
}

if(hasBodyAndSomeField()) {
    // some code
}
```

<br>


# for문 대신 .map과 .forEach을 쓴다.
for문보다 `.map`과 `.forEach`가 훨씬 보기 쉽다.
```ts
const teammates = [
    { name: "Kim", age: 24 },
    { name: "Lee", age: 30 },
    { name: "Park", age: 50 },
]

// map 코드와 for of 를 비교해보자.

// map 코드
teammates.map(person => console.log(person.age))

// for of 코드
for (let person of teammates) {
    console.log(person.age);
};
```
`map`이 훨~ 씬 보기 좋다.

<br>


# 항상 "==="를 이용한다.
"==="를 쓰는 습관을 들여라

<br>


# 전역 변수는 피한다.
library 혹은 framework를 만드는 것이 아니라면 전역변수는 피해야한다. 전역변수의 이름이 기본 도구들과 겹칠 확률도 잇고 디버그하기 힘들수 있다.

<br>


# 값이 없는 것은 "undefined"로 두지 않고 "null"로 둔다.
js에서는 값이 없을 때 "undefined"로 반환한다. 값이 없음을 나타내고 싶다면 "null"을 설정해야한다. (프론트 입장에서도 null을 줘야 편해한다.)

<br>


# 린트는 필수다. 그리고 당신만의 코드 스타일을 만들라.

<br>


# 타입스크립트는 필수다.

<br>


# 읽기 편한게 퍼포먼스보다 중요하다. 퍼포먼스가 중요해지기 전까진.

<br>

# Object 메서드를 활용한다.
- Object.keys
- Object.values
- Object.entries

```ts
const teammates = { 
    name: "Kim", 
    age: 24 
}

// Looping over objects
for (let prop in teammates) {
    console.log(prop); // "name", "age"
}

Object.keys(teammates).forEach(key => {
    console.log(`${key} : ${teammates[key]}`);
    // name : "Kim"
    // age : 24
});

Object.values(teammates).forEach(value => console.log(value)); 
// "Kim", 24

Object.entries(teammates).forEach(([key, value]) => {
    console.log(`${key}:${value}`)
    // "name: Kim"
    // "age: 24"
})
```
values 혹은 keys들을 `for문` 돌릴때 이렇게 미리 추출해서 `map` 혹은 `forEach`를 돌리는게 더 보기 좋다.



<br>



## References
- https://javascript.plainenglish.io/50-javascript-best-practice-rules-to-write-better-code-86ce731311d7
- https://betterprogramming.pub/5-javascript-tips-to-make-you-a-better-coder-f5de38cf782b