---
layout: post
title: "[Backend] Push 서버에 대한 기본 구조"
tags: [Backend]
---

먼저 앱서비스에서 푸시를 구현하는 흐름은 이렇다. (지극히 백엔드 입장에서쓴 글이다.)

0. 클라이언트에선 어플을 Firebase에 등록한다.백엔드에선 Firebase에서 준 credit 정보들을 서버에 저장한다.
1. 첫째는 클라이언트측에서 해당 유저를 Firebase에 등록한다.
2. 등록이 완료되면 토큰 혹은 unique ID를 받을텐데 그걸 백엔드에 전달한다.
3. 백엔드에서 토큰 혹은 ID를 받아 저장한다. 그리고 저장한 값을 이용해서 나중에 push를 보낼 때 이용한다.
4. 백엔드에서 상황에 따른 푸시 메시지등을 정의해둔다.
5. 상황에 맞게 백엔드에서 push 메시지를 trigger한다. (유저의 행동에 따라 서버에서 push를 실행시킨다.)
6. 메시지가 잘보내졌는지 메시지 서비스의 응답을 확인한다. 
7. 푸시 메시지가 해당 유저의 device로 도착하면 그 메시지를 받아서 처리한다.

## 0. firebase에서는 "serviceAccountKey.json" 파일을 준다. 
이 파일을 서버에 저장해두고 push 관련 환경설정 때 번호를 이용해야 한다.

## 1. 첫째는 클라이언트측에서 해당 유저를 Firebase에 등록한다.
클라이언트의 일이라 pass
## 2. 등록이 완료되면 토큰 혹은 unique ID를 받을텐데 그걸 백엔드에 전달한다.
클라이언트의 일이라 pass

## 3. 백엔드에서 토큰 혹은 ID를 받아 저장한다. 그리고 저장한 값을 이용해서 나중에 push를 보낼 때 이용한다.
DB에 push Token을 저장할 API를 만들어 프론트에 전달해준다.

```js
router.post("/user/push-token", async (req, res) => {
    const pushToken = req.body.pushToken;

    const user = await userService.savePushToken(pushToken)

    res.success();
})
```
<br>

## 4,5. 백엔드에서 상황에 따른 푸시 메시지등을 정의해둔다. 그리고 상황에 맞게 trigger한다.
메시징 서비스마다 다르고, 개별 메시징 서비스에서 지원해주는 형식도 여러가지지만 보통은 JSON을 지원해준다.
메시지의 최소한의 형식은 이렇다.

```js
import * as admin from "firebase-admin";

router.post("/comment", async (req, res) => {
    const commentWriterId = req.currentUser.id;
    const commentWriterName = req.currentUser.name;
    const commentMessage = req.body.commentMessage;

    // 코멘트를 DB에 저장하는 commentService 함수.
    const comment = await CommentService.create(
        commentWriterId,
        commentMessage
    );

    // DB에 저장해둔 push token 가져오기.
    const pushToken = await UserService.getPushTokenByUserId(req.currentUser.id);

    const message = {
        data: {
            title: `${comment.writer.name}님의 새 댓글`,
            body: `${comment.message}`
        },
        token: pushToken,
        android: {
              // 안드로이드 설정
        },
        apns: {
            // iOS 설정
        }
    }
    
    // ! trigger한다. !
    await admin.messaging().send(message);
});
```

위 코드가 아주 기초적인 메시징 예시이다. message에 들어갈수 있는 내용들은 [여기(FMC 메시지 정보)](https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages)에서 더 확인할수 있다.

> 주의할 점은 위의 `POST /comment` API의 방식대로(**동기**) push를 보내는 것은 이용자가 아주 적은 서비스에서나 가능하다. push를 안정적으로 보내기위해 보통 **비동기방식**으로 메시지를 보내게 된다. 그 이유에 대해서는 다음글에서 알아볼 예정이다.  


## 6. 메시지가 잘보내졌는지 메시지 서비스의 응답을 확인한다.
FCM의 응답코드를 확인한다.
에러의 종류들은 [여기서](https://firebase.google.com/docs/reference/fcm/rest/v1/ErrorCode) 확인 가능하다.
인증서가 잘못되었을수도 있고, 메시지가 너무 클수도 있다. 서버에서 잘못된 방식으로 메시지를 짠게 대부분일것이니 초기에 메시지를 만들고 꼭 여러번 메시지가 잘전달되는지 응답값을 확인해야만 한다.

## 7. 푸시 메시지가 해당 유저의 device로 도착하면 그 메시지를 받아서 처리한다.
이것 또한 프론트의 일.

<br>

----
### 마무리

정말 백엔드 입장에서 글을 썼고 아주 기초적인 내용만 들어갔다. 이번글에서는 push 요청을 동기방식으로 작성했지만 대부분의 push서버는 비동기로 처리한다. 다음글에서는 왜 push를 비동기로 처리하는지 정리해보려고 한다.


## 참고 글
- [how push work](https://developers.google.com/web/fundamentals/push-notifications/how-push-works)
- [FCM message](https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages)
- [FCM errorCode](https://firebase.google.com/docs/reference/fcm/rest/v1/ErrorCode)