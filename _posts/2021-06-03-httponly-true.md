---
layout: post
title: 'httpOnly: true & 로그아웃'
tags: [Node, Backend, JWT]
---

> jwt & httpOnly: true 를 이용하면,

php버전의 클라이언트에서 react 버전으로 리뉴얼하는 도중 발생한 이슈.
프론트분께서 "로그아웃해도 토큰이 안지워지네요?"라고 하신다. php버전에서는 프론트에서 로그아웃하면 알아서 토큰 잘지우던데 왜그런가 생각해보니.

1. 프론트분의 요청으로 `httpOnly: true`를 설정해뒀었다. 그 이유로 프론트단에서 토큰을 지울수 없게 되어버렸다.
```ts
// jwt 토큰 세팅 설정
ctx.cookies.set(
    'access_token', 
    token, 
    { 
        httpOnly: true, 
        maxAge: 1000 * 60 * 60 * 24 * 7, 
        sameSite: "none", 
        secure: true});
    }
```

**`httpOnly: true`를 설정해두면 프론트에서 토큰을 건드릴수 없게 된다.** 보통의 로그아웃 기능은 클라이언트에서 토큰을 지워버리는 방법으로 진행된다. 그러나 `httpOnly: true`로 설정되어있다면 서버단에서 직접 지워주어야 한다.

# 해결 

기존의 cookie 값인 `access_token`을 `null`값으로 수정하는 logout API를 만든다.

```ts
// auth.route.ts
@route('/logout')
@GET()
async logout(ctx: Koa.Context) {
    ctx.cookies.set('access_token', null, { // set null
        maxAge: 0,
        httpOnly: true
    });
    ctx.response.status = HttpStatus.NO_CONTENT;
}
```
