### 클라우드 클라이언트 라이브러리를 위해 새로운 설정 추가하기
* timeout 설정 추가
```yaml
auth:
    strategy: username-password
    username: user
    password: pass


timeouts:
    connection: 1000
```

* CloudServiceConfiguration을 위한 새로운 타임아웃 설정
```java
public class CloudServiceConfiguration {
    private final AuthStrategy authStrategy;
    private final Integer connectionTimeout;
    // 생성자, hashCode, equals, getter, setter 생략
}
```

* CloudServiceClientBuilder에서 타임아웃 추출
```java
Map<String, Object> timeouts = config.get("timeouts");
// ...
return new DefaultCloudServiceClient(
    new CloudServiceConfiguration(authStrategy, (Integer) timeouts.get("connection"));
)
```

* 스트리밍과 배치 라는 두 도구의 관점에서 볼때, 해당 변화는 하위 호환성이 없음.
* 반대로 추가 항목이 없는 경우 기본설정을 제공한다면, 하위 호환을 유지함.
* 기본값이 명시적으로 설정되지 않는다면, 클라우드 클라이언트의 새로운 버전을 구성하지 못함.

#### 배치 도구에 새로운 설정 추가하기
* 배치 도구는 클라우드 호출자에서 클라우드 빌더로 설정을 직접 전달함.
  * 새로운 timeouts 절을 추가로 제공할 필요가 있다는 의미.
