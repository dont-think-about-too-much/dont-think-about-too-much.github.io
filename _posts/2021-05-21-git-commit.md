---
layout: post
title: git 커밋, 브렌치 방식 [ Node 서버 리뉴얼 기록 ]
tags: [Nodejs, renewal, API]
---

**커밋 방식**

```
<타입>(적용 범위/optional): <설명>

[본문(optional)]

[꼬리말(optional)]
```

<br>

## 제목 형식 <타입>

Feat : 새로운 기능을 추가했을 때.<br>
Fix : 버그를 수정했을 때.<br>
Build : 빌드 관련 파일 수정에 대한 커밋<br>
!Hotfix: 급하게 버그를 고쳐야할 때<br>
Chore : 빌드 테스트 업데이트, 패키지 매니저 설정하는 경우(프로덕션 코드 변경 X)<br>
Ci: CI 관련 설정 수정에 대한 커밋<br>
Docs : 도큐먼트에 수정에 대한 커밋<br>
Style : 코드 문법 또는 포맷에 대한 수정에 대한 커밋(세미콜론, 코드수정없는경우)<br>
Refactor : 프로덕션 코드 리팩토링<br>
Comment: 필요한 주석 추가 및 변경<br>
Test : 테스트 코드 수정에 대한 커밋(프로덕션 코드 변경 X)<br>
Rename: 파일 혹은 폴더명을 수정하거나 옮기는 작업<br>
Remove: 파일을 삭제하는 작업만 수행<br>

## 적용 범위

커밋된 내용의 적용범위를 나타냅니다.

## 본문

커밋의 첫줄로 표현하기 부족한 부분을 적습니다.

## 꼬리말

해당 커밋이 어떤 이슈에서 왔는지에 대한 참조 정보를 작성합니다.

### 꼬리말 타입

Fixes: 이슈 수정중<br>
Resolves: 이슈를 해결했다.<br>
Ref: 참고할 이슈가 있음<br>
Related to: 해당 커밋에 관련된 이슈 번호

<br><br>

## Reference

- [https://overcome-the-limits.tistory.com/entry/](https://overcome-the-limits.tistory.com/entry/%ED%98%91%EC%97%85-%ED%98%91%EC%97%85%EC%9D%84-%EC%9C%84%ED%95%9C-%EA%B8%B0%EB%B3%B8%EC%A0%81%EC%9D%B8-git-%EC%BB%A4%EB%B0%8B%EC%BB%A8%EB%B2%A4%EC%85%98-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0)
