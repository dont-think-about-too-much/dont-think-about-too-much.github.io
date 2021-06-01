---
layout: post
title: 리뉴얼 시작 [ Node 백엔드 제작시 마주칠 것들 ]
tags: [NodeJS, Backend]
---

<br>

> 백엔드 API 리뉴얼하며 복습하는 시리즈.

1. [협업 컨벤션 정하기: github 커밋, 브렌치 이용 방식.](https://dont-think-about-too-much.github.io/2021/05/21/0git-commit/)
2. [백엔드 스펙 정하기](https://dont-think-about-too-much.github.io/2021/05/22/server-spec-inspect/)
3. [의존성 관리/package.json](https://dont-think-about-too-much.github.io/2021/05/23/package-json/)
4. Hello World 앱 만들기.
5. Hello World 앱 CICD 설정
6. TDD 진행 방식 정립
   <br><br>

---

# 개발 시작전: 셋팅

- 언어, 프레임워크
  - 언어의 특징을 알고 가기. [js,node 다운 코드?]
  - 프레임워크의 특징을 알고 가기. [express or nest 의 코드 ?]
- DB - SQL or NoSQL
  - 모델링
  - Query
    - 쿼리 최적화
    - MongoDB 쿼리 이전 => 최적화 후 코드적으로, 성능적으로 향상 방식, 결과
  - Transaction
- ORM
- [의존성 관리: package.json (+ lock.json)을 제대로 알고 넘어값시다.](https://dont-think-about-too-much.github.io/2021/05/20/package-json/)
- 서비스에 필요한 도구들(third party libarary)
- DevOps 서버는 어디에 올릴건지? AWS?
- 디자인 패턴( 코드의 일관성과 개발자들끼리 협업을 위해 )
- CICD는 어차피 회사들어가면 해야해요. 미리 배워둡시다.
- 문서화(다른 이들과의 협업)
- 의존성은? 의존성의 필요, 의존성의 장점
- API 설계
  - API 구조, 과정 통일화
  - 리턴값 통일
  - 인풋/아웃풋 통일
- 개발 프로세스 정립
  - 이슈
  - 협업 진행 방식
  - nvm, lint, prettier
- Docker

# 개발 중:

- 개발의 효율을 위해서 최대한 모든 걸 자동화한다.
- TDD 해야해요. 다만들고 테스트 코드 짜지말구요.
- 인풋 체크 [ required/ optional / enum / type]
- 계층의 중요성 / 계층의 독립성 유지의 중요성
- 코드를 깔끔히 짜는 방법.
- github는 어떻게 해야 깔끔하게 쓰는건지.
- 사용자 인증 방식
- CORS
- 파일 처리
  - multer?
  - s3
- 이미지 최적화
  - 용량 줄여서 저장하는건 당연한 일
  - S3를 깔끔히 쓰자.
  - AWS 권한을 제대로 알고 간다.
- cron jobs
- Redis

# 개발이 어느정도 됐다.

- CORS 서버를 띄워주고 전달했는데 프론트측에서 안된다고 할 것이 분명하다.
- README.md을 깔끔히
- Load Balancing (nginx)
- Process Manage (pm2?)
- Cache
- 로그 시스템과 사투하기 (city7310님 참고)
  - New Relic같은 별도의 SaaS APM을 쓸 지, 로그를 CloudWatch에 남겨서 ElasticSearch로 넘길지, Elastic APM을 써볼지 등등에 대해 의사결정한다.
  - CloudWatch도 써 보고, Elastic APM도 써 보고, New Relic도 써 보고, ELK Stack도 써 보고, TICK Stack도 써 본다.
  - 이것저것 써보는 동안 Grafana같은 걸로 로그 시각화도 한 번씩 해 본다.
  - StatsD같은 로그 집계 프록시를 로그 파이프라인 사이에 끼워넣어 본다.
  - 결론적으로 우리에게 가장 쓸만한 솔루션은 무엇일지 검토한다.
    [ 로깅 데이터 종류/ 로깅 방식/ 로깅 linux => s3 / 로깅 이용]
- 리펙토링을 한다.(너무 크므로 따로 뺄 예정)
- DB를 안전하게 유지.
  - 안정성을 지킬 방도
  - 백업
  - 퍼포먼스 향상/ 쿼리 최적화
  - 멀티 오너
  - 레플리카
- 부하에 대한 대처 /오토스케일링
- 부하 테스트
- 인프라를 다시 한번 체크
- 무중단 배포
- 모니터링

<br><br>

## Reference

- [https://velog.io/@city7310/](https://velog.io/@city7310/%EB%B0%B1%EC%97%94%EB%93%9C%EA%B0%80-%EC%9D%B4%EC%A0%95%EB%8F%84%EB%8A%94-%ED%95%B4%EC%A4%98%EC%95%BC-%ED%95%A8-%EC%BB%A4%EB%A6%AC%ED%81%98%EB%9F%BC-%ED%84%B8%EC%96%B4%EB%86%93%EA%B8%B0)
