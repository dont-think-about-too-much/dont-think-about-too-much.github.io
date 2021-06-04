---
layout: post
title: 'AWS AccessKey를 간수잘하셔야해요 !'
tags: [IAM, AWS, AccessKey]
---

AWS 초보라면 특히 IAM을 잘 모른다면 해킹 한번쯤은 당할 확률이 꽤 높다.
여러 원인으로 해킹당할 수 있지만, 초보들이 흔히 해킹당하는 방식은 accessKey를 제대로 관리하지 못해 당하게 된다.

>**Access Key ?**<br>
액세스 키는 IAM 사용자 또는 AWS 계정 루트 사용자에 대한 장기 자격 증명입니다. 액세스 키를 사용하여 AWS CLI 또는 AWS API에 대한 프로그래밍 요청에 서명할 수 있습니다(직접 또는 AWS SDK를 사용하여). from AWS

`access_key` 와 `secret_access_key`가 노출된다면 해커는 바로 ec2를 새로 만들수 있게 된다. 아래는 AWS CLI로 계정에 접근하는 방법이다.

```
$ aws configure --profile produser
AWS Access Key ID [None]: AKIAI44QH8DHBEXAMPLE
AWS Secret Access Key [None]: je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY
Default region name [None]: us-east-1
Default output format [None]: text
```

위처럼 몇줄의 입력을 끝으로 계정에 로그인할수 있다. 그리고 해당 `access_key`가 가진 권한을 어디든지 실행할수 있게 된다.
`IAM` 유저를 안만들어둔 상태로 access key를 만들면 모든 AWS 명령을 실행할 수 있게 되고, `IAM` 유저일지라도 `admin상태`로 만들어 뒀다면 이 또한 모든 AWS 작업을 할 수 있게 된다. AWS 초보일때 IAM 설명을 제대로 읽지않고 대충 access key 만들고 s3를  연결시키거나 ec2를 연결시키면 root 권한 access key를 만들어놓은 것이기 때문에. 언제든 털릴 위험이 있게 되버린다.

그리고 IAM을 이용하지도 않는데 `MFA`도 당연히 안해놨을 확률이 농후하다.

<br>

## 그래서 어떻게 관리하느냐.
1. AWS 계정에 MFA부터 설정하자. 
2. IAM admin유저가 있다면 지우자. 정말 admin 사용자가 필요하다면 두겠지만, 되도록 쓰지말자. admin 유저가 있으면 한번 털리면 정말 끝이다.
3. 딱 필요한만큼만 권한을 부여한다. 예를 들자면, S3만을 위한 유저는 정말 딱 S3 access만 설정한다. 이렇게 설정해두면 access_key가 털려도 해커가 할수 있는게 S3 작업밖에 없으니 해킹당해도 피해가 크게 줄어든다.
4. IAM 유저마다 MFA를 설정해두자. 
5. Access Key를 정기적으로 교체한다.

> MFA가 해킹당하지않는다면 + 위 5개만 지키면 해킹당해도 거의 피해가 없다. 그러니 이정도는 꼭 하자.

그리고 IAM이 뭔지 알아야 구성에 빈틈이 없을 것이니 아래 Reference의 "IAM 작동 방식 이해"는 꼭 알아두자.

----
<br>

## Reference
[IAM 작동 방식 이해](https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/intro-structure.html)<br>
[Best practices for managing AWS access key](https://docs.aws.amazon.com/general/latest/gr/aws-access-keys-best-practices.html)
