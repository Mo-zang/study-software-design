### 잠재적인 핫 코드 경로가 존재하는 단어 서비스
* 기능정의
  * 오늘의 단어 얻기
  * 단어의 유효성 검사
#### 오늘의 단어 얻기
* 주어진 날짜에 맞춰 색인을 계산하는 것.
* getWordOfTheDay() 메서드 삽입
```java
@Override
public String getWordOfTheDay() {
    int index = indexProvider.getAsInt(); // 현재 날짜에 대한 색인 얻기.
    try (Scanner scanner = new Scanner(filePath.toFile())) { // 스캐너에게 사전 파일의 위치 제공
        int i = 0;

        while (scanner.hasNextLine()) {
            String line = scanner.nextLine(); // 다음 행 검색
            if (index == i) {
                return line;
            }
            i++;
        }
    } catch (FileNotFoundException e) {
        throw new RuntimeException ("Problem in getWordOfTheDay for index: " + filePath, e);
    }
    return "No word today.";
}
```
* 처리가 끝날때 한 가지 극단적인 경우를 고려함.
  * 오늘의 단어를 위한 색인이 너무 크면 오늘을 위한 어떤 단어도 반환되지 않음.

#### 단어가 존재하는지 검증하기
* wordExists() 메서드 삽입
```java
@Override
public boolean wordExists(String word) {
    try (Scanner scanner = new Scanner(filePath.toFile())) {
        while (scanner.hasNextLine()) {
            String line = scanner.nextLine();
            if (word.equals(line)) {
                return true;
            }
        }
    } catch (FileNotFoundException e) {
        throw new RuntimeException("Problem in wordExists for word: " + word, e);
    }
    return false;
}
```

* wordExists()를 위한 최적화가 되지않음.
  * SLA를 정의하지 않았기 때문.

#### HTTP 서비스를 사용해 WordsService를 외부에 공개
* 엔드포인트
  * /word-of-the-day
  * /word-exists
