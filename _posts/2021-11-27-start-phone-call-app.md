---
layout: post
title: "[Project 1] 전화 봉사 어플 - 기능 파악하기"
tags: [project]
---

> 봉사 하는 곳에서 플랫폼 기획을 전달받음. 
"코로나로 밖을 돌아다니기 힘든 상황에 노인분들의 외로움을 덜어드릴수 있도록 전화 봉사자와 대화할수 있게 해주는 프로그램"을 만들길 원한다고 함.

"특정 번호에 전화를 걸고, 어떻게든 봉사자와 노인분이 전화연결이 된다. 그리고 대화를 할수 있다." 이것만 알려주셨다.

# 문제 해결해야할 3가지.
뭘해야하나 생각하보니 해결해야할 항목이 크게 3가지다.
1. 노인분들이 전화 요청을 하는 방식
2. 전화 가능한 봉사자 모집/ 관리하는 방식
3. 봉사자와 노인분들 전화연결하기

## 1. 노인분들이 전화 요청하는 방식 선택
"전화걸기" or "문자"

전화, 문자는 모두 **Twillo** 에서 진행할수 있어보인다.
어플이 금액도 제일 적게 들고 하겠지만, 모바일 개발자가 없으니 어쩔수 없다.
사용자(노인분들) 입장에서는 어플을 깔아야하는 장벽도 있을수 있으니 전화, 문자가 더 이용하기 편하다고 판단된다.

## 2. 전화 가능한 봉사자 모집 / 관리하기
### 봉사자 모집하기
프로토타입에서는 **Google Form**으로 진행할수 밖에 없다. 프로토타입을 넘어서면 어플안에서 신청할수 있도록 만들어야할듯하다.

전화번호와 전화 가능시간 목록을 받는다.

### 봉사자 관리하기
어드민 웹 페이지를 만들어야하는데 이건 일단 보류. (프로토타입에선 전화가 연결되는게 목적이니 어쩔수 없다.)

## 3. 봉사자와 노인분들을 전화연결한다.

노인분에게 요청이 오면 
1. 전화 가능시간대인 봉사자들 모두에게 문자 or 어떠한 방식으로든 알림이 간다.
2. 가장먼저 연락 온(전화, 문자, 어플등) 봉사자에게 노인분 연락처가 전달된다.
3. 봉사자가 직접 노인분에게 전화를 한다.
4. 전화가 끝난 후 결과를 저장한다. (추후 타임뱅크와 연동하기 위해 혹은 봉사시간 인정과 같은 처리를 위해 추가해야할 기능)

----
# 정리

### 프로그램의 전반적인 과정을 다시 정리하면 이렇다.
1. 노인분이 전화, 문자를 프로그램 전화번호로 전화 or 문자한다.
2. 서버가 현재 전화 가능한 봉사자들을 검색한다.
3. 검색된 봉사자들에게 알림, 노티를 보낸다. (현재는 문자로 진행할듯함)
4. 가장 먼저 전화 가능하다고 응답을 준 봉사자에게 노인분 연락처를 전달한다.
5. 봉사자가 노인분에게 전화한다.

### 해야할 일
1. 서버에서 전화, 문자를 받을수 있도록 해주는 Twillo를 찾아봐야 한다.
2. 구글 폼으로 오는 정보를 DB에 저장되도록 자동화해야한다.