---
layout: post
title: "[Error] Cannot send secure cookie over unencrypted connection"
tags: [NodeJS, Backend, Koa]
---

> "message": "Cannot send secure cookie over unencrypted connection"

Koa에서만 나타나는 에러로 파악된다. 

내가 해결한 방법은 들어오는 모든 요청을 안전하다고 강제로 처리하는 것이다.

```js
app.use((ctx, next) => {
    ctx.cookies.secure = true;
    return next();
});
```
미들웨어를 하나 넣어서 모든 요청에 대해 `secure: true`설정을 해두었다.

그리 보편적인 에러가 아니라서 여러 방법을 시도해보다가 겨우 찾아서 해결했다.<br>
express에서는 잘 일어나지 않는 에러로 파악된다. 역시 koa를 이용하면 작지만 거슬리는 에러들을 꽤나 많이 만나게 되는게 현실이다.

<br>

----

## Reference
- [https://github.com/pillarjs/cookies/issues/51](https://github.com/pillarjs/cookies/issues/51)
- [https://github.com/koajs/koa/issues/974](https://github.com/koajs/koa/issues/974)