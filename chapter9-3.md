### 테스트 가능성
#### 테스트 라이브러리
* 복잡한 기능을 제공하는 라이브러리를 사용할 때 코드를 쉽게 테스트 할 수 있어야하며, 테스트는 간단해야한다.
  * 리액티브 처리
    ```java
    public static Flux<Integer> sumElementsWithinTimeWindow(Flux<Integer>) {
        return flux.window(Duration.ofSeconds(10)).flatMap(window -> window.reduce(Integer::sum));
    }
    ```
  * 리액티브 처리 과정 테스트: 단순한 방식
    ```java
    // given
    Flux<Integer> data = Flux.fromIterable(Arrays.asList(1, 2, 3));
    Thread.sleep(10_000);

    // when
    Flux<Integer> result = sumElementsWithinTimeWindow(data);

    // then
    assertThat(result.blockFirst()).isEqualTo(6);
    ```
  * 실제 개발할때에는 더 많은 처리 과정과 시나리오를 테스트 해야하는 경우가 많은데, 예시에서는 `Thread.sleep()`라는 테스트 시간을 늘리는 방법을 사용하고 있음.
    * 받을 수 있는 임계값 까지 모든 단위 테스트에 필요한 시간이 늘어남.
    * 잘 작성된 라이브러리를 사용하더라도 테스트가 쉽지 않을 수 있음.
  * 테스트 라이브러리를 사용한 리액티브 처리 과정 테스트
    ```java 
    final TestPublisher<Integer> testPublisher = TestPublisher.create();

    Flux<Integer> result = sumElementsWithinTimeWindow(testPublisher.flux());

    StepVerifier.create(result)
        .then(() -> testPublisher.emit(1, 2, 3))
        .thenAwait(Duration.ofSeconds(10))
        .then(() -> testPublisher(4))
        .expectNext(6)
        .verifyComplete();
    ```
   * 테스트 라이브러리를 사용하면 사용하지 않는것보다 더욱 빠르게 테스트를 완료 할 수 있다.

#### 가짜 객체(테스트 더블)와 목 객체로 테스트하기
* 라이브러리는 종종 사용자의 잠재적인 오용을 방지하기 위해 많은 내부 사항을 호출자에게 숨김. -> 테스트를 어렵게 만들 수 있다.
* 테스트를 더욱 쉽게 하기 위해서는 테스트 프레임워크의 사용도 고려하면 좋음.

#### 테스트 툴킷 통합
* 임포트한 라이브러리가 다른 컴포넌트에서 격리될 수 있다면 단위테스트만 수행 후 통합테스트로 넘어가도 괜찮을 수 있다.
  * 통합 테스트는 저수준 세부 사항을 고려하지 않고, 고수준 컴포넌트만 테스트 해도 되기 때문.
* 프레임워크에 기반한 어플리케이션을 구축할때, 프레임워크가 통합 테스트에서 어플리케이션을 쉽게 구동할 수 있어야 한다.
  * 스프링은 **@SpringBootTest**와 **스프링러너**러 통합 테스트에서 어플리케이션을 시작하도록 함. -> ***따로 찾아볼것***.
* 스프링 통합테스트
    ```java
    @RunWith(SpringRunner.class)
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
    @ActiveProfiles("integration")
    public class PaymentServiceIntegrationTest {
        @Value("${local.server.port}")
        private int port;

        private String createTestUrl() {
            return "http://localhost:" + port + suffix;
        }
        // ...
    }
    ```

