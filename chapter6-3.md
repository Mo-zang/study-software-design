### 의존성 라이브러리의 설정을 추상화하는 도구
* 스트리밍 도구 구성

#### 스트리밍 도구 구성하기
* 배치 도구 구성과의 두드러진 차이점은 **streaming 절에서 모든 설정을 공개**한다.
  * 전용 streaming절 에서 모든 설정을 정의
    * 클라이언트는 기반 클라우드 서비스 클라이언트에 대해 아무것도 모름. -> 추상화 되어있음을 뜻함.
    * UX 관점에서는 단순해 보이지만, 몇 가지 유지보수 작업을 요함.
* 스트리밍 도구는 **사용자 이름 / 암호 인증방식만 지원**
* 예시
```java
public StreamingService create(Path configFilePath) {
    try {
        Map<String, Map<String, Object>> config = 
            mapper.readValue(configFilePath.toFile(), yamlConfigType);
        Map<String, Object> streamingConfig = config.get("streaming"); // 해당 부분은 추출된 다음에 스트리밍 서비스가 소유함.

        StreamingServiceConfiguration streamingServiceConfiguration = 
            new StreamingServiceConfiguration((Integer) streamingConfig.get("maxTimeMs")); // 구성을 생성하기 위해 maxTimeMs를 사용.

        CloudServiceConfiguration cloudServiceConfiguration = 
            new CloudServiceConfiguration(
                new UsernamePasswordAuthStrategy(
                    (String) streamingConfig.get("username"),
                    (String) streamingConfig.get("password"), // 구성을 생성하기 위해 username과 password를 사용한다.
                )
            );
        return new StreamingService(
            streamingServiceConfiguration,
            new CloudServiceClientBuilder().create(cloudServiceConfiguration)
        ); // 빌더는 프로그래밍 방식의 API로 클라우드 클라이언트를 생성
    } catch (IOException e) {
        throw new UncheckedIOException("Problem when loading file from: " + configFilePath, e);
    }
}
```

* 스트리밍 서비스가 외부에 공개한 설정을 클라우드 구성을 매핑할 필요가 있음.
  * 시스템 릴리즈 후에 매핑 내역을 유지보수 해야함.
* UX는 호출자가 전용 구성 절에만 초점을 맞추면 되어 훨씬 간단해짐.


##### 결론
* 각각의 방법을 선택했을때 드는 유지보수 비용이 무시하지 못할 정도임을 생각하고 결정을 신중히 해야한다.