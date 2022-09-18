---
layout: post
title: "[OS] Unix에서의 파일 종류들"
tags: [OS]
---

"유닉스에서 모든 것은 파일이다."

소켓이 파일이라길래 잘 와닿지 않아 파일을 정리해보려한다.

파일의 종류는
- 일반 파일
- 디렉토리
- symbolic link
- FIFO speical
- socket
- 장치 파일 (blocl special, character special)

이렇게 되겠다. + 파일의 종류는 `ls -l` 명령어로 확인할 수 있다. 

# 일반 파일
우리가 생각하는 그 파일이다. 데이터를 저장한다. 일반 파일은 택스트 파일이거나 바이너리 파일 둘중에 하나이다. 사람이 읽을수 있는 파일이 텍스트 파일이고 컴퓨터가 읽을 수 있는 것이 바이너리 파일이라고 생각하면 된다. 고로 이미지 파일, 실행 파일은 바이너리 파일이다

<br>

# 디렉토리 파일
이게 그 '폴더'다. 다른 파일들의 목록을 가진다. 그 목록의 형태는 파일의 이름과 파일의 인덱스(i-node number)로 구성되어있다. 디렉토리 파일은 다른 파일들처럼 파일 내용물을 가지고 있진 않다. 그러므로 다른 파일들에 비해 공간을 적게 필요로 한다.

<br>


# Symbolic link
다른 파일에 대한 참조다. `바로가기` 라고 생각하면 된다. 

<br>


# FIFO special
named pipe 라고도 한다.
> 여기서 pipe는 우리가 생활속에서 아는 pipe(무언가를 서로 연결해줄 때 이용하는 관)와 같은 뉘앙스를 가진다. 한 명령의 출력값(stdout)을 다른 명령의 입력값으로 연결해주는 것이다. ! pipe는 단일 방향성이다.

기본적으로 pipe는 같은 프로세스에서는 알아서 연결이된다. 그래서 unnamed pipe라고 한다. 같은 프로세스에서는 굳이 이름을 명명할 필요없고 알아서 연결되니깐.
반면에 다른 프로세스에서 연결하려면 각각의 pipe를 특정지어야만 한다. 다른 지역을 갈때 어떤 도로로 가야하는지 알아야하는 것처럼말이다. 그냥 "고속도로로 가면돼" 와 "경부고속도로로 가면 돼" 는 완전히 다르다.

FIFO 파일은 여러 프로세스에서 입출력 작업을 위해 열릴수 있다.(mkfifo() 함수를 통해서) 프로세스끼리 FIFO를 통해 데이터를 교환한다면 커널은 디스크작업 없이 커널 메모리를 거쳐 바로 데이터를 교환할수 있다.

어떻게 FIFO가 만들어지고 이용되어지는지 보자

1번째 터미널
```
$ mkfifo pipe1

$ file pipe1
pipe1: fifo (named pipe)
// 생성된걸 확인할 수 있다.

// pipe1에 입력
$ echo "hello world" > pipe1
```

2번째 터미널
```
$ cat pipe1
hello world
```

pipe가 서로 연결된걸 볼수 있다.

<br>


# socket
socket도 pipe와 비슷하다. 그러나 양방향성이라는게 큰 차이점이다. pipe는 한 명령의 출력값을 다른 명령의 입력값으로 넣는 것일뿐이니 당연히 단일방향이다.

여러개의 프로세스들이 커뮤니케이션 가능하게 해준다. 그 연결점이 socket 파일인 것이다.

소켓은 대략적으로만 알겠고 내가 직접 표현할 정도로 알지는 못하겠다. 그래서 발견한 설명들을 추가하겠다.

> What is special about unix domain sockets is that instead of having an IP address and port number, they have a file name as their address. This allows other applications that know nothing about networking to be told to open the file and read or write and the data is sent to the server instead of to the disk.

> Sockets don't contain anything. They are kind of like a "hole", that applications have access to for reading and writing. (링크)[https://stackoverflow.com/questions/71544605/how-socket-file-actually-works]

<br>

# 장치 파일
이 파일은 어플리케이션들이 장치 드라이버(ex, 프린터)와 상호작용할수 있게 해준다.


<br>
<br>

### 참고

- https://en.wikipedia.org/wiki/Unix_file_types
- https://www.ibm.com/docs/en/aix/7.2?topic=files-types
- https://www.geeksforgeeks.org/inter-process-communication-ipc/
- [fifo file description by manpages.ubuntu.com](https://manpages.ubuntu.com/manpages/bionic/man7/fifo.7.html)
- [What Are Unix Sockets and How Do They Work?](https://www.howtogeek.com/devops/what-are-unix-sockets-and-how-do-they-work/)
- [pipes vs sockets](https://www.baeldung.com/cs/pipes-vs-sockets)
- [unix socket vs tcp/ip socket](https://serverfault.com/questions/124517/what-is-the-difference-between-unix-sockets-and-tcp-ip-sockets)