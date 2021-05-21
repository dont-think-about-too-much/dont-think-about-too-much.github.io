---
layout: post
title: Access main program. Access main security. Access main program grid
tags: [frontpage, jekyll, blog]
image: '/images/posts/1.jpg'
---

드디어 블로그의 기초 공사를 거의 다 했다.
테마를 적용한 후 카테고리, 폰트, 커스텀도메인, 썸네일 등등을 손봤다.
그 과정을 요약하자면, 검색하고 고치고의 반복이었다.
지금은 거의 다 완료한 후여서 그 과정도 아름답게 미화됐지만 정말 쉽지 않은 과정이었다.

1.커스텀 도메인
깃허브 블로그에 테마를 적용하고 나서 가장 먼저 한 것은 커스텀 도메인을 적용하는 것이었다. .me랑 .dev 중 고민하다, 역시 개발자로서의 자아는 .dev라는 생각에 바로 구글 도메인에서 결제를 했다.
그 후 DNS를 적용하는 과정은 한마디로 삽질 그 자체였다.

첫번째로 마주한 것은 클라우드 플레어에 레코드를 등록하는 거였다.
깜빡하고 레코드 설정을 하지 않고 github repository의 settings에 있는 커스텀 도메인을 바꾸기만 하고 왜 바뀌지 않는 건지 한참을 고민하며 구글링을 계속 했다. 도대체 왜 되지 않는거냐며…_config.yml의 url 부분을 몇 번씩이나 고치며 삽질을 반복하다 다시 참고했던 블로그를 보다가 내가 레코드 설정을 하지 않았다는 걸 깨달았다. 다행이 레코드 설정을 하고 나서 커스텀 도메인이 잘 적용되어 https://riyo.dev를 주소로 쓸 수 있게 되었다. 레코드 설정은 아래 이미지 참조.

dnsrecord.JPG

이 후에도 url 관련 설정으로 애를 먹었는데 결과적으로는 잘 해결됐다.

2. 구글 웹 폰트
다음은 폰트를 적용했다. 처음에는 폰트에 대한 생각이 없었는데, 하나를 하면 또 다른 것도 하고 싶어져서 이것 저것 찾아보다 결국 폰트도 바꾸고 싶어져서 강행했다.
Pri probo alterum aliquando an. Duo appetere laboramus intellegat ea, ex suas diam exerci vix. Mel simul debitis id, est nusquam fuisset mentitum in. Te mei iudico iisque.
목록에 css가 적용되어 있지 않아서 그 부분을 수정했다. css는 거의 건드려본 적이 없기 때문에 조심스러운 마음으로 접근했다. html태그 안에 css 속성을 추가해본 적은 있지만 <style>태그 안에서 건드려본 적은 없어서 혹시나 뭐가 잘못되면 어떡하나 걱정이 많이 됐지만 정말 간단하게 수정할 수 있었다. 나는 list의 bullet을 제거했다.

```html
<style>
    .posts-list {
        list-style-type: none;
    }
</style>
```

5. 트위터카드와 오픈그래프
4까지 완료하고 나서 기쁜 마음에 트