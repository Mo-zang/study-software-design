### 날짜와 시간 코드 구현하기
#### 개념을 일관성 있게 적용하기
* 특정 정보가 다양한 컨텍스트에서 사용될 때 일관성 유지가 어려울 수 있음.
  * 반품 정책의 유효 배송일을 중심으로 배송할 장소의 시간대와 맞물려 창고에서 출고 이벤트가 발생할 때를 기준으로 잡음. -> 일관성 유지가능.
  * 품목이 배송되는 시점을 다루는 코드가 있다면, 시점 정리를 명확하게 해야함.
* 날짜와 시간정보의 형태
  * 메모리에서, 코드가 실행되는 동안
    * 날짜와 시간 라이브러리에서 만들어진 객체 형태
  * 네트워크 요청에서, 정보가 컴퓨터 사이에서 교환되는 동안
    * 웹 애플리케이션인 경우에는 텍스트 형태.
    * 개발자는 발신자와 수신자가 동일 형식을 사용하고 기대하는지 확인 필요함.
      * 일반적으로 불투명한 바이너리 프로토콜인 경우가 있고, 그런경우 실제 바이트가 무엇인지 알 필요가 없기도 하다.
  * JSON, CSV, XML 파일이나 데이터베이스와 같은 저장소에서
    * 네트워크 요청과 마찬가지로, 정확한 형식을 통제할수도 통제하지 못할수도 있음.
      * SQL 필드나 표준 텍스트 형식의 표현법을 통해 데이터 타입을 선택할 수 있는 경우도 있음.
##### 임피던스 불일치 처리하기
* 코드에서는 LocalDate로 처리하지만 날짜와 시간 관련 데이터 타입만이 타임스탬프 형태로 데이터베이스에 저장되어야 한다고 할때
  * 데이터베이스가 제공하는 개념으로 변환하기
    * LocalDate를 **지정된 날짜가 시작할 때 자정**의 인스턴트(UTC 기준)으로 변환
  * 텍스트 기반의 필드를 사용하기
    * 실제 데이터는 2020-12-20같이 프론트에서 전송되는 방식과 동일하게 저장될 수 있음.
  * 숫자필드 사용
    * **1970년  1월 1일 이후 경과 날짜 수**로 저장하기
      * 저장과 질의 측면에서 효율적이지만 코드 측면에서 복잡성이 증가하고 SQL SSSM(SQL Server Management Studio)이라는 도구를 사용하여 데이터를 이해하기 힘들어짐.
* 참고사항
  * 입력받는 데이터를 빠르게 선호하는 메모리 타입으로 변환하려고 노력하고, 출력하는 데이터를 최대한 늦게 대상 데이터 타입으로 변환하려고 노력하기.
* 임피던스 불일치는 흔하지만 개념사이의 변환을 위해 노력이 요구되는 유일한 경우는 아님.

##### 애플리케이션에 고유한 개념
* 애플리케이션의 개념 중 N개가 표준 개념이나 라이브러리, 데이터베이스에 표현된 개념으로 매핑되지 않는 경우가 있음.
  * Ex) 회계 정책을 따르는 회계 분기
* 해당 문제를 처리하는데에도 일관성이 중요함.
* 사용중인 라이브러리에 관용으로 느껴지는 방식으로 개념을 캡슐화 하면 좋음.
  * 관련성이 있는 변환을 모두 포함시키기.
  * 적절한 텍스트 형태의 표현 설계
  * 저장 시스템 내에서 개념을 어떻게 표현할지 고민.
* 해당 문제 해결에 대한 노력을 얼마나 초기에 행동으로 옮기느냐는 관점에서 트레이드오프 존재함.
  * 일찍 작업을 시작할수록 설계의 유연성 증가
  * 개념 사용의 단일 사례에 기반해 의사결정을 내리게 되면 설계를 사례에 과도하게 끼워맞추게되어 사용 사례의 요구사항을 충족하지 못할 수 있음.
  * 첫번째 사례를 발견했을때 다른 사례가 있는지 검토필요.
* 효과적인 캡슐화의 한 측면
  * 시작부터 빌드 테스트 가능성을 설계에 반영
    * 전반적인 코드베이스에 걸쳐 테스트에 대해 생각해볼 가치가 있다.
#### 기본값을 피함으로써 테스트 가능성 개선하기
* 종종 요구사항 문서 내에 많은 예제를 제공하는 편이 유용함.
  * 예제를 단위 테스트로 전환하여 볼수도 있음.
    * 단 합리적인 방식으로 코드가 테스트 될 때에만 의미가 있다.
  * 몇 라이브러리에서는 쉽지 않지만 몇 가지 규율을 유지하는 방식으로 한결 쉽게 만들 수 있음.
  * 단순해 보이는 코드에 숨겨진 가정이 많은 사례
    * 시스템 시계를 사용 -> 특정 인스턴트에 어떤 일이 일어날지 테스트 불가.
    * 현재 인스턴트를 시스템 시간대로 변환 -> 다양한 시간대에서 코드가 어떻게 반응할지 테스트 불가.
    * 기본 로케일을 기준으로 기본 달력 시스템을 사용.
    * 기본 로케일의 날짜 형식을 사용.
  ```java
  String now = DateFormat.getDateInstance().format(new Date());
  ```
##### 기존 시계 추상화를 사용하기
* java.time 패키지에는 **시간 서비스에서 현재 인트턴트**와 시간대도 제공하는 Clock 추상 클래스가 존재함.
* 노다타임에는 단일 GetCurrentInstant() 메서드를 포함하는 IClock인터페이스 존재
* 두가지 클래스는 코드가 현재 인스턴트를 알고 싶을때 시스템 시계를 사용하는 방법보다는 의존성 주입을 사용해 시계를 사용 가능하게 만들고 이를 사용하는 방식 권장
* 테스트 불가능한 OneMinuteTarget 클래스스
```java
public final class OneMinuteTarget {
    private static final Duration ONE_MINUTE = Duration.ofMinutes(1);
    private final Instant minInclusive;
    private final Instant maxInclusive;

    public OneMinuteTarget(@Nonnull Instant target) {
        minInclusive = target.minus(ONE_MINUTE);
        maxInclusive = target.plus(ONE_MINUTE);
    }

    public boolean isWithinOneMinuteOfTarget() {
        Instant now = Instant.now(); // 코드를 테스트 하기 어렵게 만들 수 있음.
        return now.compareTo(minInclusive) >= 0 && now.compareTo(maxInclusive) <= 0;
    }
}
```
* 테스트코드 작성 시나리오
    1. 현재 인스턴트는 목표 인스턴트보다 1분 초과 전이다.
    2. 현재 인스턴트는 정확하게 목표 인스턴트보다 정확히 1분 전이다.
    3. 현재 인스턴트는 목표 인스턴트보다 1분 전과 1분 후를 넘어서지 않는다.
    4. 현재 인스턴트는 정확하게 목표 인스턴트보다 1분 후다.
    5. 현재 인스턴트는 목표 인스턴트보다 1분 초과 후다.
  * 해당 시나리오대로 깔끔하게 테스트는 불가함.
  * 테스트 1, 3, 5에 대한 코드를 합리적으로 작성할 수 있지만, 시스템 시계 자체가 **정확하게** 목표 인스턴트 전후 1분을 보증할 수는 없음.
* java.time.Clock을 사용해 위의 코드를 테스트 가능하게 만든 버전
```java
public final class OneMinuteTarget {
    private static final Duration ONE_MINUTE = Duration.ofMinutes(1);
    private final Clock clock; // 현재 인스턴트가 필요시 참조할 시계
    private final Instant minInclusive;
    private final Instant maxInclusive;

    public OneMinuteTarget(@Nonnull Instant target) {
        this.clock = clock; // 추후 호출을 위해 보관
        minInclusive = target.minus(ONE_MINUTE);
        maxInclusive = target.plus(ONE_MINUTE);
    }

    public boolean isWithinOneMinuteOfTarget() {
        Instant now = clock.now(); // 정적 메서드를 clock 메서드로 대체.
        return now.compareTo(minInclusive) >= 0 && now.compareTo(maxInclusive) <= 0;
    }
}
```

* Clock.fixed를 사용해 현재 시각에 민감한 클래스 테스트
```java
class OneMinuteTargetTest {
    @ParameterizedTest
    @ValueSource(ints = {-61, 61})
    void outsideTargetInterval(int secondsFromTargetToClock) {
        Instant target = Instant.ofEpochSecond(10000);
        Clock clock = Clock.fixed(
            target.plusSeconds(secondsFromTargetToClock),
            ZoneOffset.UTC);
        OneMinuteTarget subject = new OneMinuteTarget(clock, target);
        asserFalse(subject.isWithinOneMinuteOfTarget());
    }

    @ParameterizedTest
    @ValueSource(ints = {-60, -30, 60})
    void withinTargetInterval(int secondsFromTargetToClock) {
        Instant target = Instant.ofEpochSecond(10000);
        Clock clock = Clock.fixed(
            target.plusSeconds(secondsFromTargetToClock),
            ZoneOffset.UTC);
        OneMinuteTarget subject = new OneMinuteTarget(clock, target);
        asserTrue(subject.isWithinOneMinuteOfTarget());
    }
}
```
* 코드설명
  * 목표 간격을 벗어난 시간을 테스트하는 메서드와, 목표 간격에 들어오는 시간을 테스트하는 메서드 분리.
  * 메서드는 매개변수화된 값과 assertFale 또는 assertTrue 중 무엇을 호출하는 지만 다름 -> 예상되는 결과에 대해서도 매개변수화된 단일 메서드를 구현하는것도 가능.

##### 자신만의 시계 추상화 도입
* 동일한 날짜와 시간 라이브러리를 사용하는 다른 애플리케이션에도 재사용 가능한 범위내에서 진행.
  * 노다타임의 경우 **현재 인스턴트가 무엇인지**를 순수 추상화로 유지할지 말지
  * java.time의 경우 시간대를 포함할지 말지
* 세가지의 유형
  * 대다수 코드가 의존하는 추상 클래스 또는 인터페이스
  * 시스템 시계를 사용하는 싱글톤 구현
  * 생성자나 이후에 인스턴트를 호출자가 설정하게 허용하는 가짜 시계 구현
    * 양산 서비스 코드가 여기에 의존하기 못하게 명시적으로 방어하기 위해 전용 테스트 패키지에서 이 구현을 외부에 공개하는 방식을 선택할 수도 있음.
* 독자적인 인스턴트 중심의 시계 인터페이스 도입
    ```java
    pubic interface InstantClock {
        Instant getCurrentInstant();
    }
    ```
* 시스템 시계 싱글턴으로 InstantClock 구현
    ``` java
    public final class SystemInstantClock implements InstantClock {
        private static final SystemInstantClock instance = 
        new SystemInstantClock();

        private SystemInstantClock() {} // 다른곳에서 못만들게 방어

        public static SystemInstantClock getInstance() { // 공개메서드
            return instance;
        }

        public Instant getCurrentInstant() {
            return Intstant.now(); // 시스템 시계에 위임.
        }
    }
    ```

* 테스트 목적을 위해 가짜로 InstantClock구현
  ```java
  public final class FakeInstantClock implements InstantClock {
    private final Instant currentInstant;

    public FakeInstantCLock(@Nonnull Instant currentInstant) {
        this.currentInstant = currentInstant;
    }

    public Instant getCurrentInstant() {
        return currentInstant;
    }
  }
  ```

  * 테스트가 불가능한 양산 서비스 코드를 피하기 위해 시계를 사용하는 방법들
  * Mock을 사용하지 않은 이유
    * 상호작용 테스트에서는 목이 가치가 높지만 시계를위해서는 그다지 유용하지 않음.
    * 특정 목 라이브러리에서 비롯된 테스트 코드의 결합도를 떨어뜨릴 수도 있음.
##### 시스템 시간대의 명시적인 사용을 회피하기
* 시간대를 직접 사용하는 메서드 내에서는 이미 시간대가 정해져 있으므로 어떤 시간대를 사용하는지 명확하다.
* 해당 메서드를 호출할 필요가 있는 모든 곳에서 시간대를 **제공해야** 하므로 우연히 시스템 시간대를 사용하게 될 가능성은 희박.

##### 명시적인 로케일과 문화적인 가정을 회피하기
* 시스템 로케일은 시스템 시간대와 유사함.
* 몇 사용자가 특정 달력 시스템을 원하더라도 대다수 비즈니스 소프트웨어는 그레고리 달력 시스템의 사용을 원함.
  * 우리의 코드는 텍스트 형식과는 무관하게 동작해야함.??


#### 텍스트에서 날짜와 시간 값 표현하기
* 로깅과 디버깅, 데이터 전송, 사용자에게 표기해줄때 등 텍스트 표현 방식이 좋을때가 있음.

##### 텍스트와 진실 사이에서 혼동 피하기
* `System.out.println(new Data());`
  * 책의 내용엔 Sun Dec 27 14:21:05 GMT 2020 출력
    * 출력값은 표준 시간대 약어가 아닌 문자열인 GMT를 포함. -> 값 자체가 시간대를 인식하고 있음.
    * 출력값은 요일, 축약된 월 이름, 연도를 포함 -> 값이 달력 시스템을 인식하고 있음.
    * 출력값은 초 단위 까지만 표기함. -> 1초가 지날 때 정확하게 Date 생성자를 호출 했는지 또는 정보 손실을 의미하는지 알 수 없음.
* java.util.Date는 연관된 시간대나 달력 시스템이 존재하지 않음.
* toString()은 그레고리 달력과 기본 시간대를 사용하지만 값 자체의 일부가 아님. -> 달과 일 이름은 지역화되지 않음.
* ISO-8691은 사전에 어떤 값인지 알지 못하면 의미가 없음.
* 텍스트 표현방식을 사용하기 전에 어떤 형태로 쓸지에 대한 충분한 고민이 필요하다는 얘기.

##### 불필요한 텍스트 변환 피하기
* 텍스트 형식에서 변환하는 방식은 피하는 편이 좋음.
* 날짜와 시간값을 문자열로 변환하는 경우
  * SQL을 직접 사용하거나 매개변수로 사용하면서 데이터베이스 질의에 값을 포함시키는 경우
  * 동일 라이브러리나 다른 라이브러리의 타입 사이에서 다양한 표현으로 변환하는 경우(Ex: LocalDateTime에서 LocalDate를 얻는 경우)
  * 1초 단위의 나머지를 포함하지 않고서 LocalDateTime 값을 포매팅하고 초 단위를 절삭하는 방식으로 다시 파싱하는 경우처럼 고의로 정보를 누락하는 경우
* 위 경우의 단점
  * 우회적인 방법으로 변환작업을 수행하면 달성하려는 목표가 모호해진다.
  * 실수로 정밀도가 떨어지거나 다른 버그가 발생하는 위험이 생긴다.
  * 더 직접적인 접근 방식에 비해 거의 항상 더 느리기 마련이다.

##### 효과적인 텍스트 표현 설계하기
* 해당 데이터를 보는 주체에 맞게 텍스트 표현 설계하는것이 중요함.
  * 사용자
  * 개발자
  * 컴퓨터

##### 라이브러리에 기대기
* 위 설명되어있는 것들을 라이브러리로 처리가 가능하다면 개발자보다 더 정확하게 처리가 가능하다.
  * 사용하는 라이브러리에 따라 처리가 불가능한 부분이 있을 수 있음.
  * 라이브러리의 포매팅방식이 모든 플랫폼에서 동일하다고 믿으면 안됨.
* 날짜와 시간 텍스트 처리 부분을 일부라도 중앙집중화 하는 편이 좋음.

##### 텍스트 형식에서 개념 파싱하기
  
#### 주석으로 코드 설명하기
* 책에서는 코드가 왜 존재하는지에 대한 주석보다는 무엇을 수행하는지에 대한 주석을 달아두는건 좋다고 함