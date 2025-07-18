### 아파치 스파크를 사용한 조인 구현
* 아파치 스파크 : 메모리 기반의 분산 데이터 처리 플랫폼(빅데이터 프레임워크)
#### 브로드캐스트 없이 조인 구현하기
* 조인은 하부에 존재하는 상당한 복잡성을 숨김.
* 질의의 실제 물리적인 실행 계획을 추출하는 방식으로 추론이 가능함.
* 실행 계획은 어떤 조인 전략이 선택되었는지 보여줌.
* 데이터 정렬 후 해시 파티셔닝 알고리즘이 사용됨.
```scala
import spark.sqlContext.implicts._
val userData = 
    spark.sparkContext.makeRDD(List(
        UserData("a", "1"),
        UserData("b", "2"),
        UserData("d", "200"),
    )).toDS()

val clicks =
    spark.sparkContext.makeRDD(List(
        Click("a", "www.page1"),
        Click("b", "www.page2"),
        Click("c", "www.page3"),
    )).toDS()


val res: Dataset[(UserData, Click)]
 = userData.joinWith(clicks, userData("userId") === click("userId"), "inner)

res.show()
assert(res.count() == 2)

|_1 | _2|
---------
|[b, 2]|[b, www.page2]|
|[a, 1]|[a, www.page1]|


res.explain()
~~~~~
```
* 두 데이터 집합이 동일한 방식으로 처리됨.


#### 브로드캐스트로 조인 구현하기
* 데이터가 정렬되지 않음.
  * 데이터 집합 중 하나의 일부를 보낼 필요가 없어졌기 때문.
* 실제 환경에서는 브로드캐스트 사용사례와 미 사용사례를 모두 확인해보는 편이 권장됨.
