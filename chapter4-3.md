### 4.3 훅을 통한 API의 확장성
* 모든 프레임워크와 시스템마다 여러 단계에 걸친 생명주기가 존재함.
  * 클라이언트의 모든 가능한 사용사례를 예측하려고 애쓰는 대신, 클라이언트가 자신만의 동작을 제공해 이를 우리가 만든 컴포넌트에 주입할 수 있게 허용할 수 있다.
    * 새로운 기능 요청에 따라 API와 코드를 변경할 필요 X 
    * 클라이언트는 자신의 로직을 제공하고, 우리는 그것에 대해 알 필요없음.
* 코드에 최대한의 유연성을 부여 하고  싶다면 훅 메커니즘을 이용하면됨.
  * **특정 지점에서 사용자가 개입하거나 커스터마이징할 수 있도록 시스템이 제공하는 확장 지점**
    * 프로그램의 실행 흐름을 중단하지 않고, 특정 시점에 사용자 코드를 삽입할 수 있게 해줍니다.
    * Laravel의 경우 **Eloquent**모델의 생명주기에서 동작하는 이벤트 훅 이라는게 있음.
    ```PHP 
    public static function boot()
    {
        parent::boot();

        static::creating(function ($model) {
            $model->uuid = Str::uuid();
        });
    }
    ```

#### 4.3.1 훅 API의 예기치 못한 사용 방어하기
* 이상적으로는 사용자를 위한 API를 공개할때 문서화가 이루어져야함.
  * Ex) 모든 훅 구현은 예외를 던질 수 없다고 명시.

##### 훅에서 일어나는 예상하지 못한 문제 테스트
* `코드4.12`
```java
HttpClientExcecution httpClientExecution = 
    new HttpClientExecution(
        metricRegistry,
        3,
        client,
        Collections.singletonList(
            httpRequest -> {
                throw new RuntimeException("Unpredictable problem!"); // 우리가 제어하지 않는 코드에서 예상치 못한 문제 발생.
            }
        )
    );
```
* 위의 경우 우리의 컴포넌트 생명주기가 영향을 받음.
##### 실패 방어
* `코드4.13` 
```java
for (HttpRequestHook httpRequesHook : httpRequestHooks) {
    try {
        httpRequestHook.excuteOnRequest(httpPost);
    } catch (Exception ex) {
        logger.error("HttpRequestHook throws an exception. Please validate your hook logic", ex); // 클라이언트가 예외를 던지더라도 우리 컴포넌트에는 영향없음.
    }
}
```

#### 4.3.2 훅 API의 성능 영향
* 정확성 관점에서 우리 코드는 클라이언트 코드가 제공한 코드의 예상치 못한 실패를 처리해야함.
  * 클라이언트가 제공한 로직이 차단될 가능성에도 주의필요.
    * 훅 API 호출의 대기시간이 1000ms라 가정할 때, 생명주기 1단계는 100ms 2단계는 200ms라는 시간이 걸린다 하면 대기시간을 포함 총 1300ms라는 시간이 소요됨.
      * 고로 성능에 엄청난 악영향을 미친다.


* 훅의 동기식 호출 내부에는 우리 컴포넌트가 사용하는 동일 스레드를 사용함.
  * 때문에 몇몇 상황에서는 **데드락**이 생길 수 있음을 주의.


* 모든 훅 API가 항상 차단될 수 있다고 가정하고 설계
  * 모든 훅 API를 호출을 독립적인 스레드로 수행
    * 필요한 스레드 수, 큐 크기, 동적으로 스레드를 추가할지 여부에 대한 결정도 필요함.
      * 복잡도 상승 및 큐를 감시하는것에 대한 설계도 필요.
* 훅 API의 선후관계 (hppens-before)
  * Laravel : 비즈니스 로직의 흐름상 선/후 처리 타이밍과 유사함
    * **"요청/응답 전후 시점에 후킹하는 방식"**
  * Spring : 빈 생성 -> 의존성 주입 -> 초기화 -> 이벤트 처리 흐름과 유사
* 훅 API를 이용한 설계의 유연성은 대기시간을 증가 시켜 성능에 저하가 있을 수 있음.

##### 병렬성 개선
* `코드`
```java
private final ExecutorService executorService = 
    Executors.newFixedThreadPool(8);
private void executeWithErrorHandlingAndParallel(String path) throws
    Exception {
        HttpPost httpPost = new HttpPost(path);
        List<Callable<Object>> tasks = new ArrayList<>(httpRequestHooks.size()); // task n개 호출
        for (HttpRequestHook httpRequestHook : httpRequestHooks) {
            tasks.add(
                Executors.callable( // 훅 마다 callable 생성.
                    () -> {
                        try {
                            httpRequestHook.executeOnRequest(httpPost);
                        } catch (Exception ex) {
                            logger.error("HttpRequestHook throws an exception. Please validate your hook logic", ex);
                        }
                    }
                )
            );
        }
        List<Future<Object>> responses = executorService.invokeAll(tasks); // 모든 task 호출
        for (Future<Object> response : responses) { // 중단된 task List 순회
            response.get(); // 비동기식 행위 대기
        }
        CloseableHttpResponse execute = client.execute(httpPost); // 모든 훅이 완료된 후에 다음 단계 실행.
        if (execute.getStatusLine().getStatusCode() == HttpStatus.SC_OK) {
            successMeter.mark();
        }
    }
```

* 훅 API를 사용하여 얻는 유연성은 여러 비용(트레이드오프)이 소모됨
  * 전용 스레드 풀을 위한 더 많은 유지보수 비용과 해법의 복잡도 라는 트레이드오프 발생.
 