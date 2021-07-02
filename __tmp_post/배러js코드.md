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

# 최대한 "const"와 불변 객체를 이용한다.
불변 객체를 선호해라. 잘바뀔수 있는 데이터를 이용하는 것은 전반적으로 프로그램의 불안정성을 가져온다.

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

# 항상 "==="를 이용한다.
"==="를 쓰는 습관을 들여라

# 전역 변수는 피한다.
library 혹은 framework를 만드는 것이 아니라면 전역변수는 피해야한다. 전역변수의 이름이 기본 도구들과 겹칠 확률도 잇고 디버그하기 힘들수 있다.

# 값이 없는 것은 "undefined"로 두지 않고 "null"로 둔다.
js에서는 값이 없을 때 "undefined"로 반환한다. 값이 없음을 나타내고 싶다면 "null"을 설정해야한다. (프론트 입장에서도 null을 줘야 편해한다.)

# 린트는 필수다. 그리고 당신만의 코드 스타일을 만들라.

# 타입스크립트는 필수다.

# 읽기 편한게 퍼포먼스보다 중요하다. 퍼포먼스가 중요해지기 전까진.

# for보다 map, filter, reduce를 선호한다.
```js
const dogs = [
    { name: "Sam", age: 2}
    { name: "Simon", age: 2}
]

// DO
dogs.map(dob => console.log(dob))

// DON'T
for (let dog of dogs) {
    console.log(dog)
}
```

# Object 메서드를 활용한다.
- Object.keys
- Object.values
- Object.entries

```ts
const dogs = {
    name: "Sam",
    age: 10,
}

// Looping over objects

for (let prop in dogs) {
    console.log(prop); // "name", "age"
}

Object.keys(dogs).forEach(key => {
    console.log(`${key} : ${dogs[key]}`);
    // "name : Sam"
    // "age : 10"
});

Object.values(dogs).forEach(value => console.log(value)); 
// "Sam", 10

Object.entries(dogs).forEach(([key, value]) => {
    console.log(`${key}:${value}`)
    // "name: Sam"
    // "age: 10"
})
```
values 혹은 keys들을 `for문` 돌릴때 이렇게 미리 추출해서 `map` 혹은 `forEach`를 돌리는게 더 보기 좋다.




## References
- https://javascript.plainenglish.io/50-javascript-best-practice-rules-to-write-better-code-86ce731311d7
- https://betterprogramming.pub/5-javascript-tips-to-make-you-a-better-coder-f5de38cf782b