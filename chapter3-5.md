## 멀티스레드 환경에서 주의할 예외
#### 제출 후 대기
* `코드3-21`
```java
ExecutorService executorService = Executors.newSingleThreadExecutor(); // 싱글스레드 사용
Runnable r = 
    ()-> {
        throw new RuntimeException("problem"); // 확인되지 않은 예외 발생.
    };
Futrue<?> submit = executorService.submit(r); // submit은 Future를 반환
assertThatThrownBy(submit::get) // get은 main스레드 차단
    .gasRootCauseExactlyInstanceOf(RuntimeException.class)
    .hasMessageContaining("problem");
```
* 행위 제출 후 결과를 어디서든 사용하지 않는다면 예외는 전파되지 않으며 무언의 실패에 대한 위험을 감수해야함.
  * Executors 서비스가 Future를 반환할 경우 정확성을 직접 검증해야함.

#### 수행 후 망각
* `코드3.22`
```java
Runnable r = 
    () -> {
        throw new RuntimeException("problem");
    };;
executorService.execute(r)
```
* Executors가 결과를 반환하지 않는다는 사실은 분리된 스레드에서 수행된 비동기 작업의 실패가 조용히 넘어갈 수 있음을 의미함.
  * 갯수가 고정된 스레드풀을 사용한다면 문제가 되는 상황이 생김.
    * 스레드가 실패했을 경우 재생성되지 않으며, 특정 시점에 모든 스레드가 실패하게 되어 스레드풀이 텅비는 오류 생길 수 있음.
  * 다이나믹한 스레드풀을 사용하는 경우에는 자원누수가 심각하게 발생할 수 있음
    * 메모리 부족문제 등
* 해당 문제를 해결하기위해 **전역 예외 처리기** 등록필요.

#### UncaughtExceptionHandler 등록
* `코드3.23`
```java
// given
AtomicBoolean uncaughtExceptionHandlerCalled = new AtomicBoolean(); // 처리기가 실행되면 true Set
ThreadFactory factory = 
    r -> {
        final Thread thread = new Thread(r);
        thread.setUncaughtExceptionHandler( // 전역 예외 처리기 설정
            (t, e) -> {
                uncaughtExceptionHandlerCalled.set(true); // 예외 발생시 uncaughtExceptionHandlerCalled가 true로 설정됨.
                logger.error("Exception in thread: " + t, e);
        });
    return thread;
};

Runnable task = 
    () -> {
        throw new RuntimeException("problem");
    };
ExecutorService pool = Executors.newSingleThreadExecutor(factory);
// when
pool.execute(task); // execute() 는 독립적인 워커 스레드에서 동작
await().atMost(5,
    TimeUint.SECONDS).untill(uncaughtExceptionHandlerCalled::get); // 처리기가 호출될 때까지 기다릴 시간을 설정 (비동기동작)
```

### 3.5.1 프라미스 API를 사용한 비동기식 작업 흐름의 예외 처리

* 확인된 예외를 확인되지 않은 예외로 감싸 처리하는 방법
##### 비동기식 API에서 예외 감싸기
* `코드3.24`
```java
public int externalCall() throw IOException {
    throw new IOException("Problem when calling an external service");
}

public CompletableFuture<Integer> asyncExternalCall() {
    return CompletableFuture.supplyAsync(
        () -> {
            try {
                return externalCall();
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        });
}
```
##### 적절히 처리되지 않은 예외가 담긴 스택 추적
* `코드3.25` ***생략***
* 언어별 차이나 라이브러리에 따라 이런 예외가 전파되지 못하기에 해당 스택을 못보고 넘기거나, 스레드를 죽이는 바람에 자원누수가 발생할 수 있음.

##### 결과나 예외와 함께 프라미스 이행
* `코드3.26`
```java
CompletableFuture<Integer> result = new CompletableFuture<>(); // 새로운 CompletableFuture
CompletableFuture.runAsync(
    () -> {
        try {
            result.complete(externalCall()); // 외부 호출 성공 및 프라미스 완료.
        } catch (IOException e) {
            result.completeExceptionally(e); // 예외가 발생하면 프리마스로 감쌈.
        }
    });
return result; // 값이나 예외로 이행된 결과값 반환.
```