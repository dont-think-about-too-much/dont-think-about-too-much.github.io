---
layout: post
title: '[문제해결] 이미지 로딩이 너무 느리다_ Node에서 할수 있는 일'
tags: [IAM, AWS, AccessKey]
---

> 온전히 Node 서버에서 할 수 있는 것들만 정리했다. (프론트측에서 가능한 것들은 제외)

## 문제: “서비스의 이미지 로딩이 너무 느리다.”

1. 이미지 로딩 시간이 오래걸린다.
2. 로딩하는 기간동안 하얀 화면이 나와버린다.
3. 1–2초동안 화면이 버벅거리는게 불편할 수 있다.

## 원인
1. 이용자가 올리는 이미지를 통째로 저장하고 있다.
2. 이미지의 용량 제한을 걸어두지 않았다.
3. 이용자가 올리는 이미지가 프로그램 만들 때 생각했던 것보다 고화질이다.(용량이 크다)

## 해결책
1. 입력 가능한 이미지 크기를 제한한다.
2. 이미지의 용량을 줄여서 S3에 업로드한다.
3. 원본 크기대로 보일 필요가 없는 이미지들은 유저에게 보일 때 괜찮을 정도로 최대한 사이즈를 줄인다.(리사이징이 용량을 엄~청 줄여준다.)

---

<br>

## 해결책 1. 이미지 크기를 제한한다. _‘Node File System’
> Node 기본 모듈인 fs를 이용하여 입력되는 파일의 용량을 확인할 수 있다.

fs 모듈은 파일의 정보를 읽을 수 있는 메서드를 제공한다. 동기 메서드는 fs.statSync() / 비동기 메서드는 fs.stat()
그리고 size 프로퍼티로 파일의 용량을 읽어올 수 있다. MB로 계산하려면 1024 * 1024 를 나눠주면 된다.
‘fs’ 모듈을 이용하여 파일의 용량을 확인하고 그에 따라 작업을 계속 수행할지 말지 분기되는 미들웨어를 만든다.

```ts
export function checkImageFileSize(): Koa.Middleware {
    return async (ctx, next) => {
        if(ctx.request.files && ctx.request.files.image) {
            let imageFileStats = fs.statSync(ctx.request.files.image.path); // 동기로 파일의 정보를 읽어온다.
            let imageFileSizeToMB = imageFileStats.size / (1024 * 1024); // MB로 단위를 수정한다.
            if (imageFileSizeToMB > 5) {
                throw new ExError(ErrorCodes.BadRequest, HttpStatusCode.BAD_REQUEST, 'Image file bigger than 5MB is not able to upload')
            }
        }
        await next();
    }
}
```

이 미들웨어를 이미지 파일 업로드 하는 API에 끼워넣으면 된다.

---

<br>

## 해결책 2. 이미지의 용량을 줄여서 S3에 업로드한다._ ‘JIMP’
> 추가 정보: PNG 파일은 이미 퀄리티가 손실되지 않는(비손실) 상태에서 최대로 사이즈가 작아진 파일이다. 그러므로 PNG 상태에서는 사이즈를 더 줄일 수 없다. JPEG로 바꾼다면 가능하다. 다만 손실이 발생한다.

Jimp library를 이용하여 이미지의 용량을 줄일 수 있다. [JIMP 링크](https://www.npmjs.com/package/jimp)

```ts
var Jimp = require('jimp');
 
// open a file called "lenna.png"
Jimp.read('image1.png', (err, image1) => {
  if (err) throw err;
  image1
    .quality(60) // set JPEG quality
    .write('image1_compressed.jpeg'); // save
});
```

위 용례를 봐보면, input은 ‘image1.png’이지만 output(write)는 ‘image1_compressed.jpeg’ 로 JPEG 형식이다.
변환 결과를 자신이 원하는 확장자로 선택할 수 있다. png가 인풋일지라도 jpeg로 확장자를 바꾸어서 저장하면 파일크기를 줄일 수 있게 된다. 다만 손실은 감수해야한다. 그 손실의 정도를 여러 기기에 걸쳐 직접 테스트해봐야한다.

---

<br>

## 해결책 3. 원본 크기대로 보여줄 일이 없다면 리사이징 하여 저장한다._ ’SHARP’
리사이징하면 이미지 용량이 정말 많이 줄어들게 된다. 이미지 딜레이가 거의 다 사라질 정도로.
리사이징의 대표적인 것은 아마 유튜브 썸네일이지 않을까 싶다. 유튜브에서도 썸네일은 정말 이미지 퀄리티 손상이 있는 상태로 저장해둔다. 어차피 원본 크기로 보일 일이 없으니 픽셀이 견고할 필요가 없다.

### Sharp
Sharp 라이브러리는 매우 잘만들어져있어서 정말 편하게 이용할 수 있다. Jimp는 stream에 piping이 안되지만, sharp는 아주 간단히 piping이 된다.
piping 예시.

```ts
const transformer = sharp()
  .resize({
    width: 200,
    height: 200,
    fit: sharp.fit.cover,
    position: sharp.strategy.entropy
  });
// Read image data from readableStream
// Write 200px square auto-cropped image data to writableStream
readableStream
  .pipe(transformer)
  .pipe(writableStream);
```

---

<br>

### 서비스에 적용한 결과
현재 문제가 발생한 서비스에서는 세가지를 모두 이용하고 있다. 3가지 모두 당연히 필요한 것들이지만 너무 급하게 스프린트로 만드느라 하나하나 챙기지 못했다. 막상 사이트를 오픈하니 이미지들이 너무 커 서비스가 무거워졌다. 그래서 바로 바로 모든 이미지들에 적용했고, 이미지 가져오는 도중에 나오던 하얀 화면도 이젠 볼수 없다. 이미지 딜레이가 90%는 사라졌다.