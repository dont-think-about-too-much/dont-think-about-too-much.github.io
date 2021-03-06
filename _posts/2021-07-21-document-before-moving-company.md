---
layout: post
title: "[Node] 백엔드 이직시 인수인계 목록"
tags: [Refactoring]
---

# 백엔드 인수인계할 때 고민할 목록

! 인수인계 글 적기 전에 리펙토링부터 간단한 것들은 합시다.

- 계층구조 잘지켰는지 확인.
- 선택한 디자인 패턴대로 코드가 흐르는지.
- 다음에 여유있을 때 하자고 넘긴 것들.
- 서버 인프라.
  - 개별 인스턴스의 이름 디테일하게 적어주기. 같은 종류의 서비스를 여러개 쓰면 헷갈릴 수 있다.
  - 서비스마다의 특이점 (ex. 이전 서비스에서는 오디오 관련이라 sox 라는 프로그램을 EC2에 직접 설치해야만 서버가 작동했다.)
- 관련 파일 (ssl, pem등)
- 모든 관련 계정
  - AWS
  - MongoDB Atlas
- 이용하는 외부 서비스들 설명
  - 쓰는 이유
  - 실행 방법
- API 문서화 최신화되어있는지 확인.
- 배포
  - 배포 방법 (디테일한 명령어와 함께)
  - 배포시 주의할 점
- DB 스키마 설명.
- 복잡한 메서드에 대해서는 디테일한 주석 필요.
- 지금까지의 서비스 장애 목록.

(계속 채워나갈 예정)
