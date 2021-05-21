---
layout: post
title: Package.json
tags: [frontpage, jekyll, blog]

---
<br>

>package.json을 제대로 아는 분들이 그리 많진않을 것 같다. 내가 몰라서 그렇게 생각하는 걸수도 있고..

package.json은 노드 어플리케이션에서 의존성을 관리하는 곳이다. CommonJS 형식을 강하게 따르고, json 형식이다.
`npm init` 으로 자동생성된다. 



# verseion number
## 숫자
무조건 그 버전으로 설치.
## 부등호
부등호에 맞는 버전 설치.
## x
x는 아무 버전이든 상관없다.
## latest
가장 최신 버전 설치.
## ~
패치버전까지 변경 허용
## ^
마이너 버전가지 허용 ex) 1.1.2 => 1.9.9 가능 / 1.1.2 => 2.0.0 불가능


## 주의사항
개발 도중일지라도 의존성 그냥 npm i 하지말고, dependencies, devDependencies 나눠서 install 해야한다.