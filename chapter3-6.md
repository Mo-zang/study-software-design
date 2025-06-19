## Try로 오류를 처리하는 함수형 접근 방식
* 함수형 프로그래밍은 부작용(예외) 없는 코드가 중요함.
* 확인되지 않는 예외를 던지는것은 함수형 프로그래밍에서 부정적으로 보고있음.
##### Try 모나드(성공의 경우)
* `코드3.27`
```java
// given
String defaultResult = "default";
Supplier<Integer> clientAction = () -> 100;

// when
Try<Integer> response = Try.ofSupplier(clientAction);
String result = response.map(Object::toString).getOrElse(default);

// then
assertTrue(response.isSuccess());
response.onSuccess(r -> assertThat(r).isEqualTo(100));
assertThat(result).isEqualTo("100");
```
* 실패할 경우 반환할 수 있는 기본값을 제공할 필요가 있음.
* Try모나드의 장점은 try-catch블럭에서 예외처리할 필요가 없다.
##### Try 모나드(실패하는 경우)
* `코드3.28` ***생략***
* 성공하는 경우에는 isSuccess()를 통해 성공여부 체크 실패하는 경우에는 isFailure()를 통해 실패여부 체크

#### 3.6.1 실제 양산 서비스 코드에서 Try 사용하기
* `코드3.29` ***생략***
* 처리 과정의 정의만 보고도 실패 단계와 실패하지 않을 단계 추론가능.

##### Exception API를 사용한 HTTP 서비스 호출
* `코드3.30`
```java
public String getIdExceptions() {
    CloseableHttpClient client = HttpClients.createDefault();
    HttpGet httpGet = new HttpGet("http://external-service/resource");

    try {
        CloseableHttpResponse response = client.execute(httpGet);
        String body = extractStringBody(response);
        EntityObject entityObject = toEntity(body);
        return extractUserId(entityObject);
    } catch (IOException ex) {
        logger.error("The getId() failed", ex);
        return "DEFAULT_ID";
    }
}
```
* try-catch 와 Try모나드 방식의 주요 차이점

| 항목 | `try-catch` 방식 | Try 모나드 방식 |
|------|------------------|------------------|
| 예외 처리 | 명령어 기반 (`catch`) | 값 기반 (`Success`, `Failure`) |
| 프로그래밍 스타일 | 명령형 | 함수형 |
| 오류 처리 흐름 | 흐름 변경 (비정상 흐름) | 정상적인 값의 흐름 |
| 재사용성 | 낮음 | 높음 (`map`, `flatMap`, `recover` 등) |
| 병렬성 / 조합성 | 어렵고 번거로움 | 조합하기 쉬움 |
| 테스트/디버깅 | 흐름이 갑자기 바뀌므로 어려울 수 있음 | 모든 흐름이 명시적이라 추적 용이 |
| 예시 언어 | Java, C#, Python 등 | Scala, Kotlin (`Result`), Java (Vavr) 등 |

#### 3.6.2 예외를 던지는 코드와 Try를 섞어서 사용하기
* 확인되지 않은 예외는 어떤 메서드에서든 나올 수 있음.
* 서로의 장단점을 보완하면서 안정성과 가독성, 그리고 외부 라이브러리와의 연동성까지 챙길 수 있음.
  * try-catch → Try로 감싸기
    * 불규칙적인 예외 흐름을 Try로 정규화시켜서 map, flatMap, recover 같은 함수형 메서드 체인을 사용
  * Try 사용 후 최종 처리에 try-catch (예외 로그/변환 등)
    * 중간은 Try, 최종은 try-catch로 다뤄서 외부 시스템과의 경계에서만 예외를 핸들링