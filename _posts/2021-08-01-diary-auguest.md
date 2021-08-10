---
layout: post
title: "2021 8월 What I've done"
tags: [Diary]
---

## 1일 (일)

- 이직회사 코드 계속 보기..

# 첫번째 이직

## 2일 (월)

- 이직한 회사 첫출근
- 코드 설명 한번 더 듣고 TODO 목록 인수인계 받음.
- 비지니스와 DB 스키마 파악 중.g

## 3일 (화)

- 비지니스에 관해 세부적인 내용 파악.
- API 문서화위해 API 흐름 파악중.
- MySQL, Mongo를 ubuntu에 설치해서 이용하는 방식 공부중. (클라우드만 쓰다보니 쉽지않다.)<br>
mysql install on ubuntu : https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-20-04

## 4 ~ 6일 (수~금)

- kafka 이용 방식 파악
- 채팅 메시지 방식 파악.
- 이직 후 첫 임무가 주어짐/ 새로운 몽고 컬렉션 생성 필요.

## 8일 (일)

- 테스트 서버 구현 시작.
- TODO: 1. 기존 서버 구성과 동일하게 구현 실행 / 2. 실 서버 데이터중 일부 테스트 디비로 마이그레이션.

- ubuntu에 설치된 mysql를 외부에서 접속하기.<br>
`CREATE USER 'userName'@'yourPort' IDENTIFIED BY 'password';`<br>
`GRANT ALL PRIVILEGES ON * . * TO 'userName'@'yourPort';` <br><br>
포트에 대한 방화벽 설정은<br>
`sudo ufw allow out 3306/tcp`<br>  
`sudo ufw allow in 3306/tcp` <br><br>
방화벽 설정 확인은<br>
`sudo ufw status`

## 9일 (월)

- 로컬 환경 서버 모두 셋팅하기.
    - mongo: [Install Mongodb On MacOS](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/) (brew로 설치, 및 로컬에서 실행까지)/ 
    - mysql: [Start mysql on macos](https://osxdaily.com/2014/11/26/start-stop-mysql-commands-mac-os-x/)
    - redis: [Install redis by brew](https://gist.github.com/tomysmile/1b8a321e7c58499ef9f9441b2faa0aa8)

- linux symbolik link 알게됨.

- Error : "Client does not support authentication protocol requested by server: consider upgrading MySQL client"<br><br>
[해결 링크](https://stackoverflow.com/questions/50093144/mysql-8-0-client-does-not-support-authentication-protocol-requested-by-server) `WITH mysql_native_password`를 꼭 붙여야함.

## 10일 (화)

- HTTPS를 letsencrtpt랑 certbot 으로 편하게 무료로 이용할수 있구나~ [링크](https://jootc.com/p/201901062488)