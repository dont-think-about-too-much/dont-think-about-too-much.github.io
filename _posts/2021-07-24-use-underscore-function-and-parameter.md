---
layout: post
title: "[번역] JS Tip of the Day: The UnderScore Convention"
tags: [JS]
---

> [JS Tip of the Day: The Underscore Convention](https://forum.kirupa.com/t/js-tip-of-the-day-the-underscore-convention/643076) 를 번역한 글이다.
> It is translated from [JS Tip of the Day: The Underscore Convention](https://forum.kirupa.com/t/js-tip-of-the-day-the-underscore-convention/643076).

JS에서는 \_(underscore)가 함수에 붙거나 인자에 붙은 코드를 거의 없었다. 그러다 이번에 본 코드에서 함수와 함수의 인자들에 꽤 쓰길래 찾아본 글이다. 다른 언어들에서 쓰는 용도와도 비슷한듯 하다.

# The Underscore Convention

Level: 초급

간혹가다 JS코드에서 변수들에 \_(underscore)가 붙어있는 것을 봤을 것이다.

```js
class Candy {
  constructor() {
    this._ingredient = "sugar";
  }

  isMadeOf(ingredient) {
    return this._ingredient === ingredient;
  }
}
```

언더스코어가 JS 런타임에서는 아무 의미없지만, 코드를 읽는 사람에게 코드의 목적을 제공해준다. 위의 예에서는 `_ingredient` 프로퍼티가 있다. 여기서는 이 프로퍼티가 private한 속성임을 나타내고 클래스 바깥에서는 이용되면 안된다는 것을 말해준다.

```js
let bar = new Candy();
console.log(bar.isMadeOf("sugar")); // true (의도된 접근)
console.log(bar._ingredient === "sugar"); // true (의도하지 않는 접근)
```

런타임에서는 프로퍼티에 \_(언더스코어)가 붙었는지 아닌지 아무 신경도 쓰지 않는다. 다만 관례적으로 프로그래머는 클래스 외부에서 접근하지않아야한다.

함수의 인자에도 언더스코어 붙는데 이는 함수에서 전달되긴하지만 이용되지 않을수도 있음을 나타낸다.

```js
document.addEventListener("click", (_event) => {
  console.log("We got a MouseEvent, but are ignoring it");
});
```

코드를 읽는 이에게 이 함수는 인자를 넣지 않아도 잘 작동한다는 것을 말해준다.
다만 ESLint에서 이용하지 않는 인자에 대해 에러를 발생시킬수 있다. 그러나 ESLint를 세팅해서 에러가 안생기도록 설정할 수 있다.

언더스코어가 prefix로 붙어있는 것이 아니라 떡하니 언더스코어만 인자로 있는 경우도 봤을 것이다. 이것도 가능하다. 이용되지않은 파라미터를 나타낸다. 그러나 종종 부분적 어플리케이션에서 값의 전달의 목적으로도 이용한다.

```js
let fiveUnlessSmaller = (_) => Math.min(_, 5);
console.log(fiveUnlessSmaller(1)); // 1
console.log(fiveUnlessSmaller(10)); // 5
```

위 예시를 보면 `_`는 무시되지 않는다. `Math.min()`의 인자로 전달되고, `fiveUnlessSmaller()`는 `Math.min()`의 여러 버전중의 하나가 된다.

언더스코어 컨벤션에 대한 내용이 아니지만 언급할만한게 하나 있다. 만약 언더스코어가 객체 혹은 함수로 이용된다면 그것은 underscore 라이브러리를 이용하는 것이다.

```js
let nums = [1, 2, 3];
let three = _.last(nums);
console.log(three);

let mixed = _(nums).shuffle();
console.log(mixed); // [2,3,1];
```

언더스코어 라이브러리는 유틸리티 라이브러리로 아주 많이 쓰인다. 자세한 정보는 링크 참고 [underscorejs.org](http://underscorejs.org/)

## Reference

- [JS Tip of the Day: The Underscore Convention](https://forum.kirupa.com/t/js-tip-of-the-day-the-underscore-convention/643076)
