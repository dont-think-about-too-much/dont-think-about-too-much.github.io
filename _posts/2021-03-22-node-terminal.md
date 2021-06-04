---
layout: post
title: 'node에서 터미널 실행'
tags: [NodeJS, Backend]
---

### node 백엔드 API에서 터미널 명령을 실행시키는 방법

node의 BuiltIn 모듈인 `child_process`를 이용하여 가능합니다.

`child_process`를 간단히 정리하고 넘어가자면,
child_process는 부피가 큰 연산을 node가 실행되는 컴퓨터의 OS 또는 커널에서 실행할수 있게 해줍니다. window의 cmd, mac의 terminal을 node가 실행시켜주는 것이라고 생각하면 됩니다. child_process를 이용하면 node에서는 할 수 없는 연산들까지도 가능하게 되는 것이죠.

`child_process`를 이용하는 방법은 크게 두가지가 있습니다. `spawn()`과 `exec()`입니다. 둘은 같은 일을 하지만 방식의 차이가 있어 상황에 맞춰 쓰시면 됩니다.

제일 큰 차이점은`spawn`은 **stream**으로, `exec`는 **buffer**로 작동한다는 점입니다. (이번글은 spawn만을 이용합니다.) ![](https://images.velog.io/images/moongq/post/b1f1a1eb-3804-4df8-9d4f-38a825dc37bf/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-02-26%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%205.55.16.png)[둘의 차이점는 stackoverflow에 잘 정리되어 있습니다.(링크)](https://stackoverflow.com/questions/48698234/node-js-spawn-vs-execute)

`spawn`을 이용해봅시다. (spawn를 번역하면 "알을 낳다"라고 나옵니다. 처음엔 의아하지만 새로운 프로세스를 만들어 실행시키는 것이기 때문에 적당한 단어인듯 하네요.)

spawn에서 이용하는 것은 `stdin`, `stdout`, `stderr`입니다.

- `stdin`을 이용하여 부모프로세스로부터 자식프로세스에게 데이터를 보냅니다.
- `stdout`을 이용하여 자식 프로세스는 데이터를 출력합니다.
- `stderr`는 명령에서 발생한 에러를 출력할 때 이용합니다.

```js
// 1. spawn을 불러오고.
import { spawn } from 'child_process';

function spawnTest() {
  return new Promise(function(resolve, reject) {
    // 2. spawn을 이용하여 새 프로세스를 만듭니다.
    let process = spawn('bash');

    // 3. 실행할 명령을 작성합니다.
    // '\n' 은 엔터입니다. terminal 이기 때문에 엔터로 명령어를 입력해야 실행되겠죠?
    const command = 'ls -al \n'; // a: 숨긴 파일까지 , l: 자세한 내용까지 검색

    try {
      // 4. 부모 프로세서에서 자식프로세서로 명령을 보냅니다.
      process.stdin.write(command);
      
      // stdin을 이용할때는 end()로 반드시 입력을 끝내야합니다.
      process.stdin.end(); 

      // 5. 명령이 모두 실행됐다면 'close' 이벤트가 발생합니다.
      process.on('close', function (code) {
        console.log('end')
        resolve(code);
      });
    } catch (err) {
      console.log('error')
      reject(err);
    }
  })
 }
```
현재 위치의 파일 목록에 대한 정보를 보여주는 `spawnTest()` 함수입니다. child_process를 이용할 때는 대부분 비동기상황에서 이용하기 때문에 promise를 반환하도록 짰습니다.

> spawn과 exec의 차이에서 말했듯이 spawn은 **Stream**으로 구성되어있기 때문에 이벤트를 감지할수 있습니다. 그렇기 때문에 'close'이벤트를 감지합니다.

코드에 적힌 숫자대로 하시면 됩니다.
1. spawn을 불러오고, 
2. 새 프로세스를 생선합니다.
3. 실행할 명령문을 작성하고.
4. 부모프로세서에서 자식 프로세서로 작성한 명령문을 전달합니다.
5. 명령문이 완료됐음을 알아야한다면 'close' 이벤트를 감지하여 이용합니다.

역시나 잘만들어져 있는 언어라서 쉽게 이용할 수 있답니다 
