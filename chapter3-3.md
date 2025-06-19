## 예외 처리에서 주의할 안티 패턴
#### 예외 삼키기
  * `코드3.11`
```java
try {
    check(); // 직전 장에서 만든 메서드 사용.
} catch(Exception e) {
    // 예외가 일어나지 않는다는 가정은 매우 위험하다.
}
```
* **정보를 잃어버리기 때문**에 매우 위험
#### 스택 추적 출력
* `코드3.12`
```java
try {
    check();
} catch (Exception e) {
    e.printStackTrace();
}
```
* **스택 추적이 기본적으로 표준 출력으로 예외 내용을 출력하기 때문**에 위험.

#### 오류를 잡기 위한 로거 사용
* `코드3.13`
```java
try {
    check();
} catch (Exception e) {
    logger.error("Problem when check ", e); // logger.error 메서드는 필요한 정보만 추출해서 보여준다.
}
```

* 로거는 예외의 스택 추적을 얻어 호출자에게 전파.
  * 해당 내용은 로그 파일 끝에 추가되어 호출자가 더 효율적으로 문제를 디버깅 할 수 있게 만든다.
  
##### 제일 좋은 방법은 메서드 계약에서 예외를 명시적으로 선언 하여 클라이언트(사용자)가 메서드 호출 직후 무엇을 기대할지 알려주는 것. 

### 3.3.1 오류가 발생할 경우 자원 닫기
* 처리과정이 문제없이 진행되고, 모든 작업이 예상대로 동작하는 경우 작업 종료 후 클라이언트(자원)를 닫을 필요가 있다.
#### HTTP 클라이언트 닫기
* `코드3.14`
```java
CloseableHttpClient client = HttpClients.createDefault(); // 클라이언트를 사용해 처리.
try {
    processRequests(client); // 시스템 자원을 할당하는 새로운 클라이언트 생성
    client.close(); // 처리 완료 시 클라이언트 종료
} catch (IOException e) {
    logger.error("Problem when closing the client or processing requests", e); // close()에 예외가 발생해 실패할 경우 로그남김.
}
```

* 처리과정에 네트워크가 몇몇 패킷을 누락할 경우 실패할 수 있는 로직이 개입됨.
* processRequests()는 때때로 IOException을 던질 수도 있음. 그러게 되면 close()메서드는 호출되지 않는다. -> **자원낭비 발생**
  * 해당 상황 방지를 위해 실패할 경우에도 클라이언트를 종료 시켜주는 코드 작성 필요함.
#### 처리 요청에 문제가 발생할 경우 HTTP 클라이언트 종료
* `코드3.15`
```java
CloseableHttpClient client = HttpClients.createDefault();
try {
    processRequests(client);
} catch (IOException e) { // 처리 관련 예외
    logger.error("Problem when processing requests", e);
}

try {
    close();
} catch (IOException e) { // 종료 관련 예외
    logger.error("Problem when closing client", e);
}
```

* 해당 코드는 장황하고 오류 발생 위험도 높음.
  * 동일한 문제인 IOException 두번 다룸.
  * 처리 과정에서 실패하는경우에 close() 메서드 호출되지 않음.
* **try-with-resource** 문 사용으로 해당 위험 방지.
#### try-with-resource 문을 사용해 HTTP 클라이언트 종료
* `코드3.16`
```java
try (CloseableHttpClient client = HttpClients.createDefault()) { // try-with-resource 내에서 client 생성
    processRequests(client);
} catch (IOException e) {
    logger.error("Problem when processing requests", e); // processRequests()가 던진 예외 처리.
}
```
* 코드가 수행될 필요가 있는 로직에만 집중하게됨.
  * Closeable을 구현한 객체의 생명주기는 자동으로 처리됨.

* **언어가 try-with-resource 지원하지 않는 경우도 존재함**
  * 몇몇의 언어는 예외가 던져졌는지와 무관하게 프로그래머가 코드를 수행하게 허용함.
* Java의 경우 자원을 닫을 책임이 있는 로직을 구현하기 위해 **finally** 블록을 사용할 수 있음.
#### finally 블록을 사용해 자원 닫기
* `코드3.17`
```java
CloseableHttpClient client = HttpClients.createDefault();
try {
    processRequests(client);
} finally {
    System.out.println("closing");
    client.close();
}
```

### 3.3.2 애플리케이션 흐름을 제어하기 위해 예외를 사용하는 안티 패턴
* 어플리케이션 흐름을 제어하기 위해 예외를 사용하는 경우도 존재함.
  * 해당상황에 필요한 경우일 수 있으나 로직이 너무 복잡해지는 문제가 발생.
* 