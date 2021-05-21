---
layout: post
title: package.json & package-lock.json
tags: [frontpage, jekyll, blog]

---
<br>
# Package.json

> package.json은 1. project에 대한 정보들을 명시되어있다. 그리고 2. project의 의존성들이 명시되어있다. [ 모든 데이터는 json형식으로 명시된다. ]

<br>

## 중요한 요소들을 집고 넘어가자.

### "main" , "version", "description"
main: 프로젝트의 시작점인 파일을 명시한다. 보통 서버의 시작은 "index.js".<br>
version: 프로젝트 버전<br>
description: 프로젝트 설명<br>
author: 글쓴이

### "scripts"
프로젝트에서 사용하는 명령어들을 정리해둔다.
대표적으로 로컬용 서버 실행 명령어, 프로덕션용 서버 실행 명령어, build 명령어 등등이 있겠다.
```json
"scripts": {
    "start:dev": "NODE_ENV=development node src/index.js",
    "start:prod": "NODE_ENV=production node src/index.js",
}
```
<br>
### "dependencies" 와 "devDependencies"
**"제대로 구분안해두면 협업하는 이에게 불편함을 줄수 있다."**<br>
이 둘에는 프로젝트에 설치된 의존성들이 명시된다. 예를 들어 `npm install sharp -P` 를 실행하면 sharp가 설치되고 dependencies에 적히게 된다.

**여기서 확실히 하고 넘어갈 것이 있다. 개발용이라면 확실히 devDpendencies에 명시되게하고, 프로덕션용이라면 dependencies에 확실히 명시되게 해야한다. `-P | -D`**

- -P<br>
`npm install <someDependency> -P`는 "dependencies"에 명시되고.
- -D<br>
`npm install <someDependency> -D`는 "devDependencies"에 명시된다.

대충 `npm i <someDependency>` 로만 하다보면 타인이 이 프로젝트의 의존성을 파악하는데 힘들어진다.

# Package-lock.json

> `node_modules` 가 수정되거나 `package.json`이 수정되었을 때 자동으로 생성되는 파일. 의존성 버전을 특정지어줌으로써 협업할 때  여러사람 간의 버전의 의존성 버전을 같도록 해준다.<br>
> => 버전의 차이로 발생하는 문제를 없애줌.

**대부분 `node_modules`를 제외시키고 저장소에 커밋하기때문에 `lock.json`도 `package.json` 파일과 같이 저장되어야 한다.**

<br><br>
## Reference
- [https://junwoo45.github.io/2019-10-02-package-lock/](https://junwoo45.github.io/2019-10-02-package-lock/)