---
layout: post
title: git 커밋, 브렌치 방식 [ Node 백엔드 제작시 마주칠 모든 것 ]
tags: [Nodejs, renewal, API]
---
> **목차**<br>1. 커밋 방식<br>2. 개발자간의 브렌치 이용 방식 

<br>

# **커밋 방식**

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
<br>
<br>

-----

# 개발자간의 브렌치 이용 방식

## 1. 태초에 마스터 브렌치가 있다.
마스터 브랜치는 현재 서비스되고 있는 코드를 의미한다. 
아직 서비스가 되고 있지 않다면, 지금까지 확정된 코드를 의미한다.

## 2. branch 생성
새로운 코드를 작성할때는 새로운 branch를 생성하여 작성한다.

```console
$ git checkout -b newBranch
```
## 3. push
새로운 코드를 작성 후 [1] git add [2] git commit ~~ [3] push 한다.<br>
push할때는 branch의 이름을 정확히 명시해주어야 한다.
```console
$ git add . # 모든 수정 내용을 추가
$ git commit -m "feat(POST): 글 작성 기능 추가"
$ git branch 
main
newBranch
$ git push main newBranch # newBranch의 내용을 main에 추가한다.
```
## 4. create pull request
push가 성공적으로 진행되면, github웹 해당 repository에 새로운 push 가 있다는 메시지기 나오게 된다.
이 push내용에 어떤 것들을 담고 있는지 협업자들에게 설명하는 글을 작성한 후 `create pull request`를 누르면 다른 개발자들에게 `pull request`가 전달된다.

## 5. Merge
pull request를 처리하는 사람이라면 코드를 본 후 최종 코드에 합칠지 말지 결정한다.
코드 리뷰를 디테일하게 적어주는 것이 서로의 성장에 도움이 될터이니 열심히 적어주도록 하자.

## 6. 작업한 branch 삭제
```console
$ git branch -d newBranch
```
Merge된 브렌치를 지우자. 간혹 branch를 계속 남겨두어 한사람이 여러개의 branch를 생성해두는 사람이 있다. 특정 목적이 있으면 상관이 없으나. 그냥 안지웠을 뿐일 때가 꽤 많다. repo를 깔끔히 유지하기 위해, 그리고 서로의 업무 현황을 쉽게 볼수 있도록 merge한 branch는 지워버리자.

> 지금까지의 브랜치 이용 순서대로 계속 진행하면 OK다.

<br><br>

## Reference

- [https://overcome-the-limits.tistory.com/entry/](https://overcome-the-limits.tistory.com/entry/%ED%98%91%EC%97%85-%ED%98%91%EC%97%85%EC%9D%84-%EC%9C%84%ED%95%9C-%EA%B8%B0%EB%B3%B8%EC%A0%81%EC%9D%B8-git-%EC%BB%A4%EB%B0%8B%EC%BB%A8%EB%B2%A4%EC%85%98-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0)
- [https://wayhome25.github.io/git/2017/07/08/git-first-pull-request-story/](https://wayhome25.github.io/git/2017/07/08/git-first-pull-request-story/)