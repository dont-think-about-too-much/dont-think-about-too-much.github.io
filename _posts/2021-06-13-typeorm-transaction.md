---
layout: post
title: Transaction with TypeORM [ Node 백엔드 제작시 마주칠 것들 ]
tags: [Node, Backend, Transaction]
---

쿼리가 여러개이고 그 쿼리들의 값이 서로에게 영향을 준다면 Transaction은 필수다. 특히나 수정기능의 API들에서 그렇다.

TypeORM에서 Transaction을 어떻게 만드고 이용하는지 알아보자.

**이번 글에서 이용할 것의 큰 틀은 1. EntityManager 와 2. QueryRunner 이 두개다.**

> 1. Entity Manager
>    :EntityManager is similar to Repository and used to manage database operations such as insert, update, delete and load data. While Repository handles single entity, EntityManager is common to all entities and able to do operations on all entities. - [tutorialspoint](https://www.tutorialspoint.com/typeorm/typeorm_working_with_entity_manager.htm)

> 2. Query Runner
>    :Each instance of QueryRunner is a separate isolated database connection. Using query runners you can control your queries to execute using single database connection and manually control your database transaction. - [orkhan.gitbook.io](https://orkhan.gitbook.io/typeorm/docs/query-runner)

<br>

# Transaction 없는 코드

```ts
// without Transaction
// post.service.ts

import { getManager } from "typeorm"; // EntityManager로 모든 것을 다 한다.


async createWithoutTransaction(token: JWTToken, createPostInput: CreatePostInput): Promise<Post> {
    const entityManager = getManager();

    const getUserData = await entityManager
        .createQueryBuilder(User, "user")
        .where("user.id = :id", { id: token.userId })
        .getOne();

    const newPost = await entityManager
        .create(Post, {
            userName: getUserData.userName,
            ...createPostInput
        });

    const savedPost = await entityManager.save(newPost);

    return savedPost;
  }
```

<br>

# Transaction이 추가된 코드

```ts
// with Transaction
// post.service.ts

import { getConnection, QueryRunner } from 'typeorm';

async createWithTransaction(token: JWTToken, createPostInput: CreatePostInput): Promise<Post> {

    // connection pool을 연결
    const connection: Connection = getConnection();

    // 새 queryRunner를 생성한다.
    const queryRunner: QueryRunner = connection.createQueryRunner();

    // EntityManager도 편의를 위해 변수로 저장해서 씀.
    const queryRunnerManager = queryRunner.manager;

    // 실제 DB 연결 생성
    await queryRunner.connect();

    // !! transaction 오픈.
    await queryRunner.startTransaction();

    try {

        const getUserData = await queryRunnerManager
            .createQueryBuilder(User, "user")
            .where("user.id = :id", { id: token.userId })
            .getOne();

        const newPost = await queryRunnerManager
            .create(Post, {
                userName: getUserData.userName,
                ...createPostInput
            });

        const savedPost = await queryRunnerManager.save(newPost);

        return savedPost;
    } catch (error) {
        // 에러가 발생하면 모든 쿼리를 되돌린다. 트랜젝션을 이용하는 이유.
        await queryRunner.rollbackTransaction();
    } finally {
        // 모든 작업이 성공하면 Transaction을 끝낸다.
        await queryRunner.release();
    }
  }
```

**딱히 어렵지 않다. queryRunner 만들어주고 Transaction이 필요한 작업들을 모두 Transaction안에서 작업한다. 기존에 manager만을 이용하던 방식에서 queryRunner만 더 이용하면 된다.**

# QueryRunner 추가 설명

`TypeORM`의 `Connection`은 단지 Connection pool만을 구성하기 때문에 데이터베이스에 직접 접근은 불가하다. 직접 접근하기 위해서는 `QueryRunner`가 필요하다.

`QueryRunner`는 단일의 독립적인 데이터베이스 connection이고. 쿼리를 컨트롤할수 있고, 트랜잭션을 직접 조절할수 있게 해준다.
(트렌잭션을 이용하는데에 사람 차이가 있겠지만, 결국엔 `queryRunner`를 많이 이용하는 듯하다. 많이 이용하는 것부터 제대로 알고 나머지는 나중에 찾아보자.)

**[주의] 각각의 연결을 고립된 환경에서 진행하기 위해 `QueryRunner`를 이용하는 것이기 때문에 더이상 필요하지 않을 때는 바로바로 연결을 끝내야한다.**
