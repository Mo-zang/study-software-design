### 당신이 소유한 코드에서 예외를 처리하기 위한 우수 사례
#### 3.2.1 공개 API에서 확인된 예외 처리하기
* 공개 메서드에는 특정한 경우에 실패한다고 예상되는 경우 해당 사항을 명시해야함.
  * 공개 API시그니처에 예외 선언
##### 두 가지 예외를 던지는 API 메서드
* `코드3.8` 
```java
void check() throws IOException, InterruptException;
```
###### 확인되지 않은 예외로 전파
* `코드3.9`
```java
public void wrapIntoUnchecked() {
    try {
        check();
    } catch (RuntimeException e) {
        throw e;
    } catch (Exception e) { // 공개 메서드 호출에서 모든예외 잡음
        throw new RuntimeException(e); // 잡은 예외를 확인되지 않은 예외로 감싼다.
    }
}
```
* 다른 **RuntimeException**을 불필요하게 감싸지 않기위해 먼저 잡아준다는것에 유의.
* 해당 방식은 전에 기술되어있는 코드의 내결함성을 떨어뜨릴 수 있어 좋지 않음.

###### 계약을 명시적으로 선언시
* 호출자는 메서드 구현부를 확인하지 않고도 결과에 대해 추론 가능
* 확인되지 않은 예외로 인한 문제 발생위험 감소 및 오류처리 코드 작성에 용이


#### 3.2.3 공개 API에서 확인되지 않은 예외 처리하기
##### API에서 확인되지 않은 예외를 던지기
* `코드3.10`
```java
boolean running;

public void setupService(int numberOfThreads)
    throws IllegalStateException,
            IllegalArgumentException { // 메서드가 확인되지 않은 예외를 던질 수 있도록 선언
                if (numberOfThreads < 0) {
                    throw new IllegalArgumentException(
                        "Number of threads cannot be lower than 0."
                    ); // 인수가 올바르지 않다면 IllegalArgumentException
                }
                
                if (running) {
                    throw new IllegalStateException(
                        "The service is already running."
                    ); // 이미 동작중 인 경우 IllegalStateException
                }
            }
```

* 해당 코드의 내용은 알고있으면 유용하지만 해당 코드의 예외를 잡을 필요는 없음.
* **메서드 시그니처를 통한 문서화**를 한다면 사용자(개발자)가 해당 **내용을 파악할 확률**이 높다.
* API가 확인된 예외를 던져야 하는지, 확인되지 않은 예외를 던져야 하는지에 대한 고민 필요.
  * 호출자 코드와 구현 코드를 동시에 작성하는 경우에는 전체를 알기 때문에 확인된 예외로 처리하는 것이 불 필요 할 수 있음.
  * 반대의 경우에는 사용자가 어떻게 코드를 사용할지 모르기 때문에 필요.