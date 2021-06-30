---
layout: post
title: "[Logging] 초기 스타트업의 로깅 필수요소 with Nodejs"
tags: [Refactoring]
---

# 어떤 로그를 남겨야할까?
1. 예상치 못한 에러 (500 에러)
2. API 요청 데이터(body, queryString 등)과 응답 데이터.  (morgan이 좋다.)
3. 나중에 유저 분석을 위해 필요한 데이터 (기획이 먼저 나와야 하므로 개발자 혼자 할수 없는 부분이다.)

제일 중요한 **1번!** 초기 스타트업이라면 서비스의 예상치 못한 에러들이 산재해있을 것이다. 이 에러들을 이용자보다 먼저 알아채서 업데이트해야한다. 이용자보다 먼저 알아채지 못하더라도 이용자에게 에러가 발생한 순간 바로 대응할 수 있어야 한다. 

**"예상하지 못한 에러가 발생하면 바로 문자든 슬렉이든 여러 사람에게 알람이 뜨도록 설정해둬야한다."**

2번은 기본적으로 모든 요청에 대해 로그를 남겨두면 서비스가 어떤 흐름으로 돌아가는지 알수 있고 여러모로 활용하기가 좋다. **API 요청, 응답 로깅은 기본이다.**

3번은 기획이 있어야 가능한 부분이므로 개발자 혼자하기 힘들다. 그러므로 패스.
  
<br>

# 로그를 남길때 필수 요소들
1. Source of log: 어디서 이 로그가 발생한건지
2. Timestap: 언제 발생했는지
3. Level and Context: 로그 레벨과 디테일한 맥락/상황.

> ! Tip 로그 레벨을 잘 활용하지 못할 것 같다면 Error, Info, Debug만 구분하여 작성하는게 나을수 있다.

개인적으로도 `warn`, `fatal` 레벨이 무슨 소용인가 싶다. 에러 관련 로그는 `.catch(error)`문안에 적어놓을 텐데 `Logger.error()`만 적어놓지 직접 `Logger.warn();` 코드를 적어놓는 상황이 거의 없을텐데말이다.

> Exception이 발생하는 경우 무의식적으로 ERROR 레벨을 사용하는 경우가 있는데 `500`이 에러지 미리 예상해둔 Exception들에 Error 로그를 쓰지말자. 예상하지 못한 결과가 발생했을때 알림을 받기를 원하는 것이다. 예상하는 것들을 에러 로그로 받는 것은 쓸데없는 일을 하는 게 아닌가. 
초기 스타트업에서는 예상하지 못한 상황을 최대한 빨리 알아채야한다. 그런데 Exception들을 에러 로그로 받아버리면 **다른 예상 못한 에러들이 묻힐수 있다.**

다른분 블로거분의 의견: "시나리오 상 의도된 Exception이라면 ERROR 레벨로 작성할 이유가 전혀 없습니다. Exception을 오류라고 생각하면 그럴 수도 있지만 Exception은 ‘예외’이기 때문에 의도한 경우에는 INFO로 적는 것이 더 좋다고 생각합니다." [참고 [무엇을 로그로 작성할까? 로그 목적, 방법 그리고 로그 레벨](https://blog.lulab.net/programmer/what-should-i-log-with-an-intention-method-and-level/)]

<br>

# 그래서 로그 어떤 형태로 남길건데? 예시좀.
개인적으로 디폴트 설정되있는 것들을 만족하는 편이라서 HTTP 요청에 대해서는 morgan library 디폴트 메시지대로 이용한다.
```
// morgan 'combined' format
::1 - - [26/Apr/2020:16:58:09 +0000] "GET / HTTP/1.1" 200 13 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.122 Safari/537.36"
```

<br>

**하지만 !** 500 에러/예상하지 못한 에러에 대한 로그는 더 많은 내용이 필요하게 된다.
로그 형태는 당신이 어떤 정보를 로그 메시지로 받고싶은지에 따라 다르다. 당신만의 형식으로 커스텀해서 logger.error() 인자로 넣어주면 되긴한다. 만들기 귀찮다면 **winston `infoObj`** 옵션에 아래 객체만 추가해주자.

```ts
const errObj = {
    req: {
        url: request.url,
        headers: request.header,
        query: request.querystring,
        body: request.body,
    },
    error: {
        message: err.message,
        stack: err.stack,
        status: err.status
    },
    user: request.currentUser
}
```

위 데이터를 `infoObj` 옵션에 넣어준다.
```
Logger.error({ message: "Unexpected Error Ocurred", infoObj: errObj })
```
<br>
**이 정도만 추가해줘도 개발 상태에서 받는 `error stack`과 비슷해진다. 에러 원인을 찾을때 훨씬 편해질 것이다.** (아래는 로그 예시)

![emessage1](/images/posts/emessage1.png)
![emessage2](/images/posts/emessage2.png)

<br>

# 로그는 뭘로 남기지?
Nodejs 환경에서 Logger library로 제일 유명한 것은 [winston](https://www.npmjs.com/package/winston)이다. (되도록 대세 라이브러리를 이용하자.)

## 로그를 남기자 with "Winston"
[winston](https://www.npmjs.com/package/winston) 사이트를 한번 읽고 오자. 그리고 시작하자.

## winston 설정.
```ts
// logger.loader.ts
import * as winston from 'winston';
import yenv from 'yenv';
import * as fs from 'fs';
import { getFrommatedTime } from '../dateMethods';

const logDir = yenv().LOG_DIR;

// 에러용 로그 저장 파일
const ErrorLogFileName = `${logDir}/[${getFormattedTime()}]error.log`;
// 모든 로그 저장 파일
const AllLogFileName = `${logDir}/[${getFormattedTime()}]all.log`;

const saveLogInLocal = [ new winston.transports.File({ 
        filename: ErrorLogFileName, 
        level: 'error',
        maxFiles: 1
    }),
    new winston.transports.File({ 
        filename: AllLogFileName,
        maxFiles: 1
    }) 
]

// 로그를 저장할 파일이 없으면 미리 생성.
if (!fs.existsSync(logDir)) {
    fs.mkdirSync(logDir);
}

// "Logger"를 불러와 실행시키면 끝이다.
export const Logger = winston.createLogger({
    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.prettyPrint(),
        winston.format.splat(),
    ),
    transports: saveLogInLocal // ! transports 설정으로 로그를 저장할 곳을 설정한다.
});
```

<br>
주석들 읽으며 내려오면 후루룩 읽힐 것이다. 

### 로그 사용 예시
사용하는 방법은 아주 간단하다. 미리 만들어놓은 `winston logger` 클래스를 가져와서 이용하면 된다. <br>
`ex. Logger.error();`

```ts
// post.route.ts
import { Logger } from '../logger.loader';

@route('/posts')
@GET()
async getPosts(ctx: Koa.Context) {
    try {
        const result = await PostService.getPosts();
    } catch (error) {
        // ! 에러 로그를 남긴다
        Logger.error({ message: "Unexpected Error Occured", infoObject: error }) //
    }
}
```
<br>

지금까지의 코드로는 서버가 돌아가는 컴퓨터 안에 로그가 저장된다. **우리가 바라는건 예상치 못한 에러가 발생했을 시 최대한 팀원들이 이를 알아채는 것이다. 문자 혹은 Slack으로 알면 좋겠다.** 이를 위해 에러 관리툴의 대세인 [Sentry](https://sentry.io/)를 이용하자.

<br>

## 특정 에러는 알림을 받자 with "Sentry"
Sentry는 서비스 관리툴로 많이들 쓴다. 이번 글에서는 로그 관리용으로만 이용하려한다. **기존 Winston 설정에 Sentry `DSN(Data Source Name)`만 붙이면 거의 끝이다.**

> 설정관련해서 설명은 꽤 부족할수 있으므로 [winston-transport-sentry-node](https://www.npmjs.com/package/winston-transport-sentry-node)를 보는게 더 나을 수 있다. + Sentry 자체적으로 설명을 잘해주고 인터렉션이 쉬워 금방할 수 있을 것이다.

1. Sentry에 가입한다.
2. 프로젝트를 만든다.
3. `DSN`(Data Source Name)을 받는다. 프로젝트를 만들면 받을 수 있다.
4. !! `DSN`을 기존 winston의 `transports`에 연결시킨다.

> `Winston`과 `Sentry`를 함께 쓸때는 [winston-transport-sentry-node](https://www.npmjs.com/package/winston-transport-sentry-node)을 이용하는게 편하다.

## winston 설정에 Sentry DSN 설정
```ts
// logger.loader.ts

// Sentry를 연결하기 쉽게해주는 모듈을 이용한다.
import Sentry from 'winston-transport-sentry-node';

// Sentry 옵션을 설정한다.
const options = {
    sentry: {
        dsn: "https://10dbae2760914f1eaef3aa566afc8fe6@o883226.ingest.sentry.io/5836820"
    }
};

// Sentry 경로
const toSentry = [ new Sentry(options) ];

// 로그를 저장할 파일이 없으면 미리 생성.
if (!fs.existsSync(logDir)) {
    fs.mkdirSync(logDir);
}

// "Logger"를 불러와 실행시키면 끝이다.
export const Logger = winston.createLogger({
    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.prettyPrint(),
        winston.format.splat(),
    ),

    //
    transports: toSentry // ! 기존 local에서 Sentry로 바꿧다
});
```

기존 winston 에러 로그의 목적지를 local에서 Sentry로 바꿨을 뿐이다. **이제 로그가 생성되면 Sentry 사이트에서 모두 확인할 수 있게 된다.**

### 여기에 Sentry 사이트를 들어가서 알람을 추가하자.
![sentry_setting](/images/posts/createAlert.png)
"Create Alert Rule" 클릭

![sentry_alert](/images/posts/sentry_alert.png)
위 알람 규칙은 로그 레벨이 **"error"**이면 **sentry**(내 슬렉 워크스페이스 이름이다) slack workspace에 **"#로깅-확인"** 채널에 에러 메시지가 전달된다.

**이제 이메일과 Slack으로 알림이 오도록 설정했다. 아래처럼 에러가 발생할 때마다 메시지가 오게 된다.**

![sentry_alert](/images/posts/sentry_slack1.png)

이제는 에러가 발생할때마다 로그를 보고 업데이트를 잘하면 된다. ㅎㅎ

## References
- [무엇을 로그로 작성할까? 로그 목적, 방법 그리고 로그 레벨](https://blog.lulab.net/programmer/what-should-i-log-with-an-intention-method-and-level/)
- [Logging: Best Practices for Node.JS Applications
Learn how to get the most out of logging.](https://blog.bitsrc.io/logging-best-practices-for-node-js-applications-8a0a5969b94c)