### 클라우드 클라이언트 라이브러리에서 설정을 더 이상 사용하지 않기로 결정하거나 제거하기

#### 배치 도구에서 설정 제거하기
* 특정 설정값이 사용되지 않거나 제거된다면, 모든 클라이언트는 새로운 타입으로 이주해야함.
  * 타사 라이브러리의 내부를 외부에 공개해서 해당 구성에 대한 모든 변경사항을 클라이언트 코드에서도 조정해야함.
  * 변경되거나 제거된 설정을 따라가지 않은 클라이언트는, 새로운 배치 서비스를 사용할 때 실행 시점에서 예외가 발생함.
    * 하위 호환성을 유지하고 UX문제를 줄이려면 우회 방식을 제공해야함.
      * 해당 방식은 심각한 문제로 이어질 수 있음.
        * 디버깅에 어려움.
        * 클라이언트가 생성될 때마다 임시 파일을 생성해야함.
        * 원본파일을 조작해서 사용자가 모르게 구성 값을 변경.
        * 코드에 강결합 문제 초래함.
#### 스트리밍 도구에서 설정 제거하기
* 책의 예제에서 설명하는 바로는 (인증전략에 관한내용.) 내부 인증 전략이 사용자로부터 추상화 되어있기에, 호환성에 손상없이 변경이 가능.
  * 구성 설정이 변하지 않음.
  * 최종 사용자는 변경없이 새로운 버전을 사용가능.
  * 매핑 계층만 새로운 구성 설정에 맞춰 조정.
  * 새로운 방법으로 이주하는 단계가 구현되어야 하지만, 캡슐화를 사용해 간소하게 처리됨.

#### UX 친화성과 유지보수성 측면에서 두 해법 비교
|도구|UX|유지보수 비용|
|---|---|---|
|배치도구|낮음: 사용자는 크게 영향을 받음|높음/현실성 없음|
|스트리밍 도구|높음: 사용자는 영향을 받지 않음|아주 낮음|

### 결론
* 기술적인 결정이 UX에 영향을 미칠 수 있다.
* 다운스트림 라이브러리에 새로운 설정을 추가하는 작업은 유지보수 비용 없이 처리할 수 있다.
* 추가적인 추상화는 호환성을 깨뜨리지 않고서 도구를 진화하게 만들 수 있다. 하지만 추가적인 유지보수 비용이 들어간다.
* 추상화를 추가하면 기반 컴포넌트가 변경될 때마다 이를 사용하는 코드에도 추가 작업이 필요하다.
* 제품이 외부에 공개되고 사용자의 UX가 중요하다면 코드를 사용하는 라이브러리의 내부 세부 사항을 전파하지 않는 방식이 현명하다.