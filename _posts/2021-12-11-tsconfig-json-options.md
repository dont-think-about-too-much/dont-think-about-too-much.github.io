---
layout: post
title: "[JS] tsconfig.json 옵션을 짚고 넘어가자."
tags: [JS]
---

tsconfig 옵션 설정으로 typescript가 어떻게 작동해야하는지 정의해준다. 한번 읽어보면 타입스크립트에 대해 좀더 깊히 이해할수 있지 않나 싶다. ([원본](https://www.typescriptlang.org/tsconfig)에서 Recommended 표시되어있는 것들 위주로 정리했다.)

# Type Checking

### #allowUnreachableCode
어떠한 상황에서도 접근되지 않을 코드가 있다면 에러가 발생한다.

```ts
function fn(n: number) {
  if (n > 5) {
    return true;
  } else {
    return false;
  }
  return true; // Unreachable code detected. 위의 if else 에서 코드 진행이 끝난다.
}
```
<br>

### #allowUnusedLabels
변수를 선언만하고 값을 주지않았을 때 에러 발생

```ts
function verifyAge(age: number) {
  // Forgot 'return' statement
  if (age > 18) {
    verified: true; //Unused label. 선언만 했다.
  }
}
```
<br>

### #alwaysStrict *Recommended
ECMAScript strict mode를 이용할지 설정한다.
<br>

### #exactOptionalPropertyTypes *Recommended
`?` prefix를 가진 변수의 타입에 대해 강제적인 조건을 적용한다.
예를 들어 `colorThemeOverride?: "dark" | "light";` 이렇게 정의되어있다면 `colorThemeOverride`는 **무조건** 둘중에 하나의 타입을 가져야한다. 만약 false로 셋팅한다면 `undefined`로도 정의될 수 있다.

```ts
const settings = getUserSettings();
settings.colorThemeOverride = "dark";
settings.colorThemeOverride = "light";
 
// But not:
settings.colorThemeOverride = undefined;
// Type 'undefined' is not assignable to type '"dark" | "light"' with 'exactOptionalPropertyTypes: true'. Consider adding 'undefined' to the type of the target.
```

<br>

### #noFallthroughCasesInSwitch
switch문을 이용할 것이라면 모든 경우가 포함되어야한다.
```ts
const a: number = 6;

switch (a) {
    case 0:
        console.log("even");
    case 1:
        console.log("odd");
        break;
}
```

<br>

### #noImplicitAny *Recommended
`any` 타입으로 선언되었다면 에러 발생.
```ts
function fn(s) {
    console.log(s.subtr(3));
}
```

<br>

### #noImplicitReturns
모든 함수에 대해 리턴값이 있는지 확인.

<br>

### #noImplicitThis *Recommended
`this`의 대상이 `any`로 타입 선언되어있다면 발생하는 에러

```ts
class Rectangle {
  width: number;
  height: number;
 
  constructor(width: number, height: number) {
    this.width = width;
    this.height = height;
  }
 
  getAreaFunction() { // 이 함수가 any로 선언돼서 발생.
    return function () {
      return this.width * this.height;
    //'this' implicitly has type 'any' because it does not have a type annotation.
    //'this' implicitly has type 'any' because it does not have a type annotation.
    };
  }
}
```

<br>

### #noPropertyAccessFromIndexSignature
정의되지 않은 변수에 대해서 dot(.)으로 접근하지 못하도록 한다. true로 선언되었다면 가능하다.
```ts
interface GameSettings {
    speed: "fase" | "slow";
    quality: "high" | "low";

    //  타입 선언되지 않은 것들은 모두 string으로 타입을 선언하도록 설정.
    [key: string] : string;
};

const settings = getSettings();
settings.speed; // dot(.)으로 접근 가능하다.
settings.quality;

settings.username; // Property 'username' comes from an index signature, so it must be accessed with ['username'].
// ! 지금의 옵션으로는 접근 불가하다.
```

<br>

### #noUnusedLocals
사용되지 않는 로컬 변수에 대해 에러를 발생시킨다.
```ts
const createKeyboard = (modelID: number) => {
    const defaultModelID = 23;

    // 'defaultModelID' is declared but its value is never read.
    return { type: "keyboard", modelID };
}
```

<br>

### #noUnusedParameters
사용되지 않는 함수의 파라미터에 대해서 에러 발생

<br>

### #strict *Recommended
모든 strict 옵션들을 킬때 이용한다.

<br>

### #strictBindCallApply *Recommended
`call`, `bind`, `apply`이 호출될때 올바른 인자들과 함께 이용되는지 확인한다.

```ts
// With strictBindCallApply on
function fn(x: string) {
    return parseInt(x);
}

const n1 = fn.call(undefined, "10");

const n2 = fn.call(undefined, false); // Argument of type 'boolean' is not assignable to parameter of type 'string'
```

<br>

### #strictFunctionTypes *Recommended
함수의 파라미터들을 좀더 디테일하게 체크한다.
```ts
function fn(x: string) {
    console.log("Hello, " + x.toLowerCase());
}

type StringOrNumberFunc = (ns: string | number) => void;

// Unsafe assignment
let func: StringOrNumberFunc = fn;
// Unsafe call - will crash
func(10);
```

`true`로 설정하면,
```ts
function fn(x: string) {
  console.log("Hello, " + x.toLowerCase());
}
 
type StringOrNumberFunc = (ns: string | number) => void;
 
// Unsafe assignment is prevented
let func: StringOrNumberFunc = fn; 
// fn() 은 string을 받는데 stringOrNumberFunc은 string 혹은 number를 받는다.
// 서로 상충되서 에러 발생

// Type '(x: string) => void' is not assignable to type 'StringOrNumberFunc'.
//   Types of parameters 'x' and 'ns' are incompatible.
//     Type 'string | number' is not assignable to type 'string'.
//       Type 'number' is not assignable to type 'string'.
```

<br>

### #strictNullChecks *Recommended
`null`이거나 `undefined`인 변수를 이용하려하면 에러 발생
자바스크립트를 쓰다가 타입스크립트를 쓰는 제일 첫번째 이유가 이거지 않나 싶다.

<br>

### #strictPropertyInitialzation *Recommended
생성자에 설정되어있지 않은 프로퍼티를 선언하면 에러가 발생한다.
```ts
class UserAccount {
    name: string;
    accountType: "user";

    email: string; // Property 'email' has no initializer and is not definitely assigned in the constructor.
    address: string | undefined;

    constructor(name: string) {
        this.name = name;
        // this.email이 설정되어있지 않았다는 사실을 확인 !
    }
}
```

<br>

# Modules

<br>

### #allowUmdGlobalAccess
> UMD: AMD와 CommonJS를 모두 쓸수 있게 해주는 장치라고 생각하면 된다.
모듈안에서 UMD 를 접근할수 있게 해준다.