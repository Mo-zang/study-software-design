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
* 새로운 구성 설정으로 클라우드 클라이언트를 생성하길 원한다면 모든 클라이언트에 해당 절을 추가해야함.
  * 추가하지 않는다면 배치 서비스 도구 생성에 실패함.
* UX는 크게 변하지 않아, 유지보수 비용이 낮음
  * 변경이 자주 발생하고, 내용이 추가된다면 해당 방법이 상당히 효율적임.

#### 스트리밍 도구에 새로운 설정 추가하기
* 스트리밍 도구는 프로그래밍 방식으로 클라우드 클라이언트를 생성하므로 이를 책임진 코드를 변경할 필요가 있음.
  * 구성이 변경될때마다 코드를 변경해주어야함.
  * 유지보수 비용이 증가.

### UX 친화성과 유지보수성 측면에서 두 해법 비교
* 배치 서비스는 변경 사항을 최종 사용자에게 전파.
* 스트리밍 서비스는 클라우드 서비스를 사용하는 사실을 추상화하려고 시도.
---

|도구|유지보수 비용|UX|
|---|---|---|
|배치도구|비용 없음|사용자가 새로운 설정 추가할 필요 있음.|
|스트리밍 도구|비용 증가|사용자가 새로운 설정을 추가 할 필요 있음.|
