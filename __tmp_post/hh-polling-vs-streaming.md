# ~~ing

HTTP에서는 서버가 클라이언트에게 데이터를 보내줄 방법이 없다. (push server 가 없다.)
이를 해결하기 위한 방법이 Polling, Long Polling, Streaming 방식들이다.

# Polling
주기적으로 요청하여 결과를 확인.

# Long Polling
요청에 대한 응답을 서버 이벤트 발생시점에 받는 방식.
누군가 이벤트를 발생시키면 그 이벤트를 받은 모든 클라이언트에서 다시 서버로 요청을 보내버림.
새 이벤트마다 모든 클라이언트에서 다같이 반응하다보니 한번에 트래픽이 몰림.


# Streaming
요청에 대한 응답을 완료하지 않은 상태에서 데이터를 계속 내려받는 방식이다.


## References
- [polloing, long polling, streaming/ 민이 님 블로그](https://m.blog.naver.com/PostView.nhn?isHttpsRedirect=true&blogId=youreme&logNo=110162110369)
- [naver d2](https://d2.naver.com/helloworld/1052)
- [Long-Polling 방식과 Polling 방식 선택하기](https://kuimoani.tistory.com/74)
- [polling and streaming concept scenarios](https://www.geeksforgeeks.org/polling-and-streaming-concept-scenarios/)