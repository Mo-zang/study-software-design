## 타사 라이브러리에서 오는 예외
#### 예외가 선언된 API
* `코드3.18`
```java
import java.io.IOException;
import org.apache.commons.io.FileExistsException; // 외부 클래스 import

public interface PersonCatalog {
    PersonInfo getPersonInfo(String personName) throws IOException; // 표준 IOException
    boolean createPersonInfo(String personName, int amount) throws FileExistsException; // 외부 라이브러리 구현을 바깥으로 노출하는 예외
}
```

* 두 API 메서드 선언 모두 예외를 던짐.

#### 도메인 전용 예외 생성
* `코드3.19`
```java
public class PersonCatalogException extends Exception {
    private PersonCatalogException(String message, Throwable cause) {
        super(message, cause);
    }
    public static PersonCatalogException getPersonException(String personName, Throwable t) {
        return new PersonCatalogException("Problem when getting person file for: " + personName, t);
    }
    public static PersonCatalogException createPersonException(String personName, Throwable t) {
        return new PersonCatalogException("Problem when creating person file for: " + personName, t);
    }
}
```
* **PersonCatalogException**은 실제 **Throwable**과 오류를 캡슐화한 private로 선언된 생성자를 받음.
  * getPersonInfo() 메서드를 위한 예외 getPersonException() 팩토리 제공
  * createPersonInfo() 메서드는 Throwable을 새로운 PersonCatalogException으로 감싸는 과정에서 유사한 상황에 대한 대응방안 제공

#### 타사 라이브러리의 예외를 누출하지 않는 PersonCatalog
* `코드3.20`
```java
public interface PersonCatalog {
    PersonInfo getPersonInfo(String personName) throws PersonCatalogException;
    boolean createPersonInfo(String personName, int amount) throws PersonCatalogException;
}
```
* 저렇게 변경 후 **PersonCatalogException**을 던진다고 선언
  * 타사 라이브러리의 예외 누출 X
  * 클라이언크 코드와 강하게 결합되지 않더라도 API사용 가능함.