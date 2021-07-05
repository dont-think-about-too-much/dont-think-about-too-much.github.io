---
layout: post
title: "[JS] / 번역"
tags: [Refactoring]
---

읽기 편한 자바스크립트 작성 방식.

>취향의 문제이므로 동의하지 않는 방식이 꽤 있을 수 있다.

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

# Object를 loop할 때는 Object의 메서드를 활용한다.



## References
- [5 Javascript tips to make you a beter coder](https://betterprogramming.pub/5-javascript-tips-to-make-you-a-better-coder-f5de38cf782b)
