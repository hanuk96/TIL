# 커넥션 풀
데이터베이스 커넥션 과정을 살펴보자
1. 애플리케이션 로직에서 DB 드라이버를 통해 커넥션을 조회한다.
2. DB 드라이버는 DB와 `TCP/IP` 커넥션을 연결한다. 3 way handshake 같은 `TCP/IP` 연결을 위한 네트워크 동작이 발생한다.
3. DB 드라이버는 `TCP/IP` 커넥션이 연결되면 ID, PW와 기타 부가정보를 DB에 전달한다.
4. DB는 ID, PW를 통해 내부 인증을 완료하고, 내부에 DB 세션을 생성한다.
5. DB는 커넥션 생성이 완료되었다는 응답을 보낸다.
6. DB 드라이버는 커넥션 객체를 생성해서 클라이언트에 반환한다.

<img width="800" alt="image" src="https://github.com/hanuk96/TIL/assets/12428689/5d7ca582-f6a4-4f64-a15d-cc8afbe9f4a8">

커넥션을 새로 만드는 것은 과정도 복잡하고 시간도 많이 소요됨

이런 문제를 해결하기 위해 커넥션을 미리 생성해두고 사용하는 커넥션 풀이라는 방법을 사용한다.

### DataSource
커넥션을 얻기 위해서는 DriverManger를 통해 신규 Connection을 직접 맺거나, 커넥션 풀을 이용할 수 있다.

<img width="800" alt="image" src="https://github.com/hanuk96/TIL/assets/12428689/e068144c-31e9-47c5-ba1e-2dd5a1e56b7e">

DriverManger를 사용하다가 커넥션 풀을 이용하려고 할때 어플리케이션 로직을 수정해야한다.

이를 간편하게 하기 위해 커넥션을 획득하는 방법을 DataSource로 추상화

대부분의 커넥션 풀은 `DataSource` 인터페이스를 이미 구현해두었다. 

따라서 개발자는 `DBCP2 커넥션 풀` ,`HikariCP 커넥션 풀` 의 코드를 직접 의존하는 것이 아니라 `DataSource` 인터페이스에만 의존하도록 어플리케이션 로직을 작성하면된다.

<img width="800" alt="image" src="https://github.com/hanuk96/TIL/assets/12428689/4d684ce4-0708-4dba-8e31-41f1f7b66a7a">

