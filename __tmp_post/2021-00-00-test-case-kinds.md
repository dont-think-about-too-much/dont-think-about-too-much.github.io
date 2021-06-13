> 테스트 코드는 구석에 숨어있는 실수들을 찾아서 예방해주는 것이 아니라 아주 멍청한 실수를 하지않도록 도와주는 것일 뿐이다. 그러니 노가다 같다고 느끼고 테스트 코드를 대충넘기지 말자. 대충 짜놓고 거기서 에러나면 나중에 회사에서 쪽팔린다.

# API 테스트 케이스 종류.

테스트를 입문하면 어떤 걸 테스트해야할지 잘모른다. 인터넷에서 예시들을 보면서 작성하지만, 작성하면서도 이런 노가다성 짙은 테스트 코드가 진정 안정성 향상에 도움이 될까? 하는 의구심까지 든다. 그런데 막상 개발이 끝나고 서비스 환경으로 넘어가는 순간, 서비스 진행중이 아니더라도 시간이 지나 코드가 어느새 내가 감당할 수 있는 양이 넘어가면 **테스트코드 없이는 새로 배포하는게 너무나 겁나는 본인을 마주칠 것이다.** 배포가 겁나는 순간 테스트 코드를 대충 작성하고 넘어간 본인이 미워질 것이다.

그러니 다시 한번 테스트 코드의 중요성을 머리속에 주입하고, 작성해야할 테스트 코드들의 종류를 파악하는 시간을 갖자.

# GET 요청에 대한 테스트 케이스

## 리스폰스의 형태가 맞는지.

### # 검색 결과가 있을 때.

아래가 미리 통일시켜둔 GET 요청에 대한 응답 형태라면 이 형태가 맞는지 체크한다.

```ts
200 OK
{
    pagination: {
        currentPage: 1,
        hasBefore: false
        hasNext: true,
        totalPage: 18
    },
    data: {
        posts: [
            {
                title: "title1",
                createdAt: "202112120101"
            },{
                title: "title1",
                createdAt: "202112120101"
            },{
                title: "title1",
                createdAt: "202112120101"
            },
        ]
    }
}
```

테스트 코드 예시.

```ts
expect(response.body).toHaveProperty("pagination");
expect(response.body).toHaveProperty("data");
expect(response.body.pagination).toHaveProperty("currentPage");
expect(response.body.pagination).toHaveProperty("hasBefore");
expect(response.body.pagination).toHaveProperty("hasNext");
expect(response.body.pagination).toHaveProperty("totalPage");
expect(response.body.data).toHaveProperty("posts");
expect(response.body.data.posts[0]).toHaveProperty("title");
```

### # 검색 결과가 없을 때

사람에 따라 검색 결과가 없을 때의 응답이 다르다. 어떤 이는 404 에러를 띄우기도 하고, 어떤 사람은 {}(빈 객체)를 전달해주기도 한다. 이 또한 통일되게 검사하면 된다.

404로 정했다면

```ts
expext(response.status).toEauql(404);
expect(response.body.message).toEqual("Not matched any data");
```

{ } (빈 객체)로 정했다면,

```ts
expect(response.body).toEqual({});
```

이걸 굳이 왜하나 싶을수도 있는데, 이런 간단한 response 형태도 나중에 바빠지거나, 아주~ 먼훗날에 코드를 수정할 때 형태 통일이 안되는 경우가 꽤 발생한다. **꽤나 클린코드에 도움이 된다.**

## Error 상태와 메시지 동일한지 확인.

방금 위의 예시에서 같이 했으니 패스.

## queryString 이 입력 가능한 값인지 확인.

`GET /api/v3/posts?page=some-number&category=some-category`이와 같은 API가 있다고 한다면 `sum-number`와 `some-category`가 입력가능한 값인지 확인한다. 입력 불가한 값으로 들어왔을 시 제대로 에러가 나는지 확인.

<br>

# POST

## Input 값에 대한 테스트

- 인풋 값 타입이 맞는지 틀린지. ex) Number 타입이 들어와야 하는데 String이 들어온 경우
- 인풋 값 갯수 제한이 다른 경우 ex) 하나의 값만을 받을 수 있는데 리스트로 들어오는 경우
- 필수값인 인풋이 들어오지 않을 때의 경우.
- null값이 들어왔을 시의 경우.
- undefined가 들어왓을 시의 경우.
- **[빼먹을 확률 높음]** 필수값이 아닌 인풋값이 입력되지 않을 때 디폴트로 설정해놓은 값으로 잘 설정되는지 확인. ex) 익숙하지않은 ORM을 이용할 때 updatedAt 관련 데코레이터가 있길래 잘작동하겠지 하고 붙여놨는데 나중에 보니 제대로 작동하고 있지 않더라.
- 공백("") 값이 들어올 경우.

# PATCH

# DELETE

#empty #errorstatus #errormessage #response형태 #response내용들 #input #공백입력시 #null입력시 #필수값안넣었을때 #디폴트값설정됐는지 #queryString

#
