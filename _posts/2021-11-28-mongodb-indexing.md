---
layout: post
title: "[MongoDB] 번역 Performance Best Practices: Indexing"
tags: [Database]
---

몽고DB 공식사이트에 올라와있는 퍼포먼스 향상 글 중 하나를 번역했다.<br>
This is Tlanslated Article: [Performance Best Practices: Indexing](https://www.mongodb.com/blog/post/performance-best-practices-indexing)

(몽고DB에서 제공하는 인덱싱 방법들과 몽고디비에서 직접 추천하는 방식을 대략적으로 알아볼수 있는 글이 될 것이다.)

----

이번 글에서는 **Indexing**에 대해 알아보자. 

15년 넘는 기간 동안 여러 종류의 Database 공급업체에서 일해보면서 확실하게 말할수 있는 것 중 하나가 바로 "적절한 인덱스를 설정하지 못하게 되면 퍼포먼스에 저하에 다른 요소들보다 더 큰 영향을 준다는 것"이다

그래서 우리는 지금 당장 제대로 바로 잡을 필요가 있다. 여기 당신을 도와줄 몇가지 좋은 예제 들이 있다.

# Indexes in MongoDB
어떠한 데이터베이스에서든지 인덱스는 쿼리의 효율적 실행을 도와준다. 인덱스없는 쿼리는 모든 document를 검색해야하는 상황에 처한다. 적절한 인덱스가 설정되어있다면, 데이터베이스는 인덱스를 이용하여 검색해야하는 document의 갯수를 줄일수 있게 된다.

MongoDB는 넓은 범위의 [인덱스 타입과 기능들](https://docs.mongodb.com/manual/indexes/)을 제공해준다. 몽고DB 인덱스는 언제든지 어플리케이션의 요구사항에 따라 생성되고 제거될수 있다. 그리고 어떠한 필드에든지 index를 설정할수 있는데, **nested 필드**에도 설정할 수 있다.

어떻게 최선으로 몽고DB 인덱스를 이용할수 있는지 알아본다.

## Use Compound Indexes
복합 인덱스는 여러개의 필드들이 섞인 인덱스를 의미한다. 예를 들어 "Last Name" 필드와 "First Name" 필드에 각각 인덱싱을 하는것보다 두 필드를 복합인덱스로 설정해놓는 것이 훨씬 효율적이게 된다. 이 복합쿼리는 "Last Name" 만을 조건으로 검색할 때에도 이용된다. 그러니 하나의 (복합) 인덱스 설정으로 더 많은 이점을 얻을 수 있다.

## Follow the ESR rule
복합 인덱스에 있어서, 이 규칙은 필드 인덱싱 순서를 정하는데에 많은 도움을 줄것이다.
- 첫째, equal 조건에 이용되는 필드.
- 둘째, Sort 조건에 필요한 필드.
- 셋째, 범위 조건 검색에 이용되는 필드.
이 3가지만 잘지키자.

## Use Covered Queries When Possible
> Covered Query: 설정된 인덱스를 충분히 활용할수 있는 쿼리. 불필요한 document를 검색하지 않는 쿼리.

[Covered queries](https://docs.mongodb.com/manual/core/query-optimization/#covered-query)는 불필요한 document를 검색하지 않고 값을 리턴하게 되므로 아주 효율적이다..

쿼리가 covered되기 위해서는, 모든 필드가 인덱스 안에서 필터링, 정렬되어 있어야한다. 쿼리가 covered query인지 확인하기 위해서 [explain](https://docs.mongodb.com/manual/reference/method/cursor.explain/) 메소드를 이용할수 있다. 만약 **explain()**의 결과값중 totalDocsExamined가 0이라면, 해당 쿼리는 인덱스안에서 검색되었다는 말이다. 더 자세한 내용은 [documentation for explain results](https://docs.mongodb.com/manual/reference/explain-results/#covered-queries).

Covered query를 수행할 때 일반적으로 발생하는 문제는 "_id"가 기본적으로 반환된다는 점입니다. 그러므로 "_id" 값을 제외하거나 index에 추가해야한다.

데이터가 많아지면 Sharded Cluster를 써야할텐데, Sharded Cluster에서는 shard key값이 무조건 필요하게된다. 그러므로 index에 "_id"를 넣는것이 결과적으론 좋다.

## Use Caution When Considering Indexes on Low-Cardinality Fields
> Cardinality: 농도 혹은 집합의 원소 수
복합 인덱스에서는 데이터 갯수가 적은 필드를 설정하는 것이 괜찮다만 다른 상황에서는 이용하지 않는 것이 좋다.

## Eliminate Unnecessary Indexes
인덱스는 자원 집약적이다. 필드가 업데이트되면, 관련된 인덱스들을 유지 관리해야하므로 추가 CPU, 디스크 I/O 오버헤드가 발생한다.

## Use text search to match words inside a field
보통의 인덱스는 필드의 모든 값이 맞을 때 유용하다. 반면에 당신이 특정 단어가 포함된 검색을 하고 싶다면 [text index](https://docs.mongodb.com/manual/text-search/)를 이용해보는 것도 좋다.

## Take Advantage of Multi-Key Indexes for Querying Arrays
당신의 쿼리 패턴들이 개별 배열 값들에 접근해야한다면 [multi-key index](https://docs.mongodb.com/manual/core/index-multikey/)를 이용하라. 몽고DB는 배열의 모든 값에 대해 인덱스 키를 생성해준다.