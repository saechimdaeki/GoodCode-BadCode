# 8. 코드를 모듈화하라.

## 8.1 의존성 주입의 사용을 고려하라

### 8.1.1 하드 코드화된 의존성은 문제가 될 수 있다

하드코드로 의존되어 있는 자동차 여행 플래너를 구현하는 클래스를 보자.

```kotlin
class RoutePlanner(
    private val roadMap = NorthAmericaRoadMap()
) {
    fun planRoute(startPoint : LatLong , endPoint: LatLong) : Rounte{
        ...
    }
}

interface RoadMap {
    fun getRoads(): List<Road>
    fun getJunctions() : List<Junction>
}

class NorthAmericaRoadMap : RoadMap {
    override fun getRoads(): List<Road> {...}
    override fun getJunctions() : List<Junction> {...}
}
```

현재 RoutePlanner 클래스는 생성자에서 NorthAmericaRoadMap을 생헝하는데, 이는 RoadMap의 특정 구현에 대한

의존성이 하드코드로 되어 있음을 의미한다. 따라서 RountePlanner클래스는 북미 여행 계획만 세울 수 있다.

### 8.1.2 해결책 : 의존성 주입을 사용하라.

```kotlin
class RoutePlanner(
    private val roadMap : RoadMap
) {
    fun planRoute(startPoint : LatLong , endPoint: LatLong) : Rounte{
        ...
    }
}
```

이제 원하는 로드맵을 사용하여 RoutePlanner를 생성할 수 있고 팩토리 함수로 다음과 같이도 가능하다

```kotlin 
class RoutePlannerFactory {

    companion object {
        fun createEuropeRoutePlanner() : RoutePlanner {
            return RoutePlanner(EuropeRoadMap())
        }

        fun createDefaultNorthAmericaRoutePlanner() : RoutePlannter{
            return RoutePlanner(NorthAmericaRoadMap())
        }
    }
}
```

이러한 팩토리 함수를 직접 작성하는 것의 대안으로 의존성 주입 프레임워크를 사용할 수도 있다.

## 8.2 인터페이스에 의존하라

### 8.2.1 구체적인 구현에 의존하면 적응성이 제한된다

다음은 의존성 주입을 사용하지만 로드맵 인터페이스가 아닌 북미 로드맵 클래스에 직접 의존할 경우 RoutePlanner 

클래스가 어떻게 되는지 보여준다

```kotlin
interface RoadMap {
    fun getRoads(): List<Road>
    fun getJunctions() : List<Junction>
}

class NorthAmericaRoadMap : RoadMap {
    ...
}

class RoutePlanner(
    private val roadMap : NorthAmericaRoadMap
) {
    fun planRoute(startPoint : LatLong , endPoint: LatLong) : Rounte{
        ...
    }
}
```

이 클래스는 이제 북미 외의 다른 어떤 지역에서도 작동하지 않는다

### 8.2.2 해결책 : 가능한 경우 인터페이스에 의존하라

```kotlin
class RoutePlanner(
    private val roadMap : RoadMap
) {
    fun planRoute(startPoint : LatLong , endPoint: LatLong) : Rounte{
        ...
    }
}
```

클래스가 인터페이스를 구현하고 이 인터페이스가 필요한 동작을 정의한다면 이것은 곧 다른 개발자가 해당 인터페이스에 대해 

다르게 구혀 ㄴ한 클래스를 작성할 수 있다는 것을 강하게 시사한다. 특정 클래스보다는 인터페이스에 의존한다고 해서 더 많은 

노력을 기울일 필요가 없으면서도 코드는 상당히 모듈화되고 적응성이 높아진다

`의존성 역전 원리` : 보다 구체적인 구현보다는 추상화에 의존하는 것이 낫다

## 8.3 클래스 상속을 주의하라

### 8.3.1 클래스 상속은 문제가 될 수 있다.

쉼표로 구분된 정수를 가지고 있는 파일을 열어 정수를 하나씩 읽어 들이는 클래스를 작성해야 한다고 가정해보자.

이 문제에 대해 생각해보면 다음과 같은 하위 문제를 파악할 수 있다

- 파일에서 데이터를 읽는다
- 쉼표로 구분된 파일 내용을 개별 문자열로 나눈다
- 각 문자열을 정수로 변환한다

먼저 CSV 파일을 읽는 클래스를 보자

```java
interface FileValueReader {
    String getNextvalue();
    void close();
}

interface FileValueWriter {
    void writeValue(String value);
    void close();
}

/**
 * 
 * 쉼표로 구분된 값을 가지고 있는 파일을 읽거나 쓰기 위한 
 * 유틸리티
 */

class CsvFileHandler implements FileValueReader, FileValueWriter {
    ...

    CsvFileReader(File file) {...}

    @Override
    String getNextValue() {...}

    @Override
    void writeValue(String value) {...}

    @Override
    void close() {...}
}
```

CsvFileHandler 클래스를 사용하여 상위 수준의 문제를 해결하려면 이 클래스를 작성할 코드에 통합해야한다.

```java
/**
 * 파일로부터 숫자를 하나씩 읽어 들이는 유틸리티 
 * 파일은 쉼표로 구분된 값을 가지고 있어야 한다
 */
class IntFileReader extends CsvFileHandler {
    ...

    IntFileReader(File file){
        super(file);
    }

    Integer getNextInt() {
        String nextValue = getNextValue();
        if (nextValue == null)
            return null;
        return Integer.parseInt(nextValue, Radix.BASE_10);
    }
}
```

상속의 주요 특징 중 하나는 서브클래스가 슈퍼클래스에 의해 제공되는 모든 기능을 상속한다는 점인데, 따라서 IntFileReader

클래스의 인스턴스는 close() 함수와 같이 CsvFileHandler에 의해 제공된 함수 중 어느것이라도 호출 할 수 있다.

하지만 이는 문제가 될 수 있다

`상속은 추상화 계층에 방해가 될 수 있다`

한 클래스가 다른 클래스를 확장하면 슈퍼클래스의 모든 기능을 상속한다. 이 기능은 close()함수의 경우처럼 유용할 때가 있지만,

원하는 것보다 더 많은 기능을 노출할 수 있다. 이로 인해 추상화 계층이 복잡해지고 구현 세부 정보가 드러날 수 있다.

```java
class IntFileReader extends CsvFileHandler {
    ...

    Integer getNextInt(){...}

    String getNextValue(){...}

    void writeValue(String value) {...}

    void close(){...}
}
```

IntFileReader입장에서 getNextValue() 및 writeValue() 함수를 호출할 수 있음을 알 수 있다. 파이로부터 정수를 읽는 기능만

갖는 클래스로 개발한 것인데 이 함수들을 제공한다면 이는 이상한일이다.

`상속은 적응성 높은 코드의 작성을 어렵게 만들 수 있다`

IntFileReader 클래스를 통해 해결하려는 문제는 쉼표로 구분된 값을 가진 파일로부터 정수를 읽어들이는 것이다.

하지만 요구사항이 변경되어 쉼표뿐만 아니라 세미콜론으로 구분된 값도 읽을 수 있어야 한다고 가정하자.

그런데 다른 개발자가 이미 SemicolonFileHandler 클래스를 구현해놓았다. 

```java
/**
 * 세미콜론으로부터 구분된 값을 가지고 있는 파일을 읽거나 쓰기 위한
 * 유틸리티
 */
class SeimicolonFileHandler implements FileValueReader, FileValueWriter {

    SemicolonFileHandler(File file) {...}

    @Override
    String getNextValue() {...}

    @Override
    void writeValue(String value) {...}

    @Override
    void close() {...}
}

```

해결해야 하는 문제는 CsvFileHandler 대신 SemicolonFileHandler를 사용해야 할 때가 있다는 점이다.

이렇게 살짝 바뀐 요구사항을 반영해야 할 때 코드를 약간만 수정하면 될 것 같지만, 상속을 사용하는 경우에는 코드 변경이 간단치 않을 수 있다

유일한 방법은 IntFileReader 클래스의 새 버전을 작성하고 이 클래스가 SemicolonFileHandler를 상속하도록 하는것이다.

```java
/**
 * 파일로부터 정수를 하나씩 읽어 들이기 위한 유틸리티
 * 파일은 세미콜론으로 구분된 값을 가지고 있다
 */ 
class SemicolonIntFileReader extends SemicolonFileHandler {
    ...

    SemicolonFileReader(File file){
        super(file);
    }

    Integer getNextInt() {
        String nextValue = getNextValue();
        if (nextValue == null)
            return null;
        return Integer.parseInt(nextValue, Redix.BASE_10);
    }
}
```

CsvFileHandler와 SemicolonFileHandler 클래스 모두 FileValueReader 인터페이스를 구현한다는 점을 고려할 때,

이토록 많은 코드가 중복된다는 것은 당혹스러운 일이다. 이 인터페이스는 파일 형식을 모르더라도 값을 읽을 수 있는 추상화 계층을 제공한다

하지만 상속을 사용했기 떄문에 이러한 추상화 계층을 활용할 수 없게 되었다

### 8.3.2 해결책 : 구성을 사용하라.

클래스를 확장하는 상속을 사용하기 보다 해당 클래스의 인스턴스를 가지고 있음으로써 하나의 클래스를 다른 클래스로부터 구성(compose)하자

```java
@AllArgsConstructor
class IntFIleReader {
    private final FileValueReader valueReader;

    Integer getNextInt(){
        String nextValue = valueReader.getNextValue();
        if (nextValue == null)
            return null;
        return Integer.parseInt(nextValue, Radix.BASE_10);
    }
    
    void close() {
        valueReader.close();
    }
}
```

`더 간결한 추상화 계층`

상속을 사용할 때 서브클래스는 슈퍼클래스의 모든 기능을 상속하고 외부로 제공한다. 상속대신 구성을 사용하면

클래스가 전달이나 위임을 사용하여 명시적으로 노출되지 않는한 다른 클래스의 기능이 노출되지 않는다

`적응성이 높은 코드`

팩토리 함수로 클래스의 인스턴스를 생성할 수도 있다

```java
class IntFileReaderFactory {

    IntFileReader createCsvIntReader(File file) {
        return new IntFileReader(new CsvFileHandler(file));
    }

    IntFileReader createSemicolonIntReader(File file) {
        return new IntFileReader(new SemicolonFileHandler(file);)
    }
}
```

## 8.4 클래스는 자신의 기능에만 집중해야 한다.

### 8.4.1 다른 클래스와 지나치게 연관되어 있으면 문제가 될 수 있다

다음 코드애는 두 개의 개별 클래스에 대한 코드의 일부가 나와 있다. 첫 번째 클래스는 책을 나타내고 두 번째 클래스에는

책의 한 장을 나타낸다. 

```java
class Book {
    private final Lsit<Chapter> chapters;

    Int wordCount() {
        return chapters
            .map(it -> getChapterWordCount(it))
            .sum();
    }

    private static int getChapterWordCount(Chapter chapter) {
        return chapter.getPrelude().wordCount + 
            chapter.getSections()
                .map(section -> secion.wordCount())
                .sum();
    }
}

class Chapter {
    ...
    TextBlock getPrelude() {...}

    List<TextBlock> getSections() {...}
}
```

getChapterWordCount() 함수를 Book 클래스에 두면 코드가 모듈화되지 않는다. 요구사항이 변경되어 장의 끝에 요약을 포함해야 한다면,

getChapterWordCount() 기능도 수정해서 요약에 들어있는 단어들도 셀 수 있도록 해야한다. 개발자가 요약에 대한 변경 사항을

Chapter 클래스에만 반영하고 Book.getChapterWordCount() 함수는 잊어버린다면, 이 함수는 올바르게 작동하지 않을 것이다.

### 8.4.2 해결책 : 자신의 기능에만 충실한 클래스를 만들라

코드 모듈화를 유지하고 한 가지 사항에 대한 변경 사항이 코드의 한 부분만 영향을 미치도록 하기위해, Book과 Chapter 클래스는 가능한

한 자신의 기능에만 충실해야 한다.

수정한 클래스는 다음과 같다


```java
class Book {
    private final List<Chapter> chapters;

    Integer wordCount() {
        return chapters
            .map(chapter -> chpater.wordCount())
            .sum();
    }
}

class Chapter {
    ...

    TextBlock getPrelude(){...}

    List<TextVlock> getSections() {...}

    Integer wordCount() {
        return getPrelude().wordCount() +
            getSections()
            .map(section -> section.wordCount())
            .sum();
    }
}
```

`디미터의 법칙` : 한 객체가 다른 객체의 내용이나 구조에 대한 가능 한 한 최대한으로 가정하지 않아야 한다는 소프트웨어 공학의 원칙

## 8.5 관련 있는 데이터는 함께 캡슐화하라

### 8.5.1 캡슐화되지 않은 데이터는 취급하기 어려울 수 있다.

다음 TextBox 클래스는 사용자 인터페이스 내의 한 요소를 나타내고 renderText()함수는 이 요소내의 텍스트를 표시한다

```java
class TextBox {
    ...

    void renderText(String text, Font font, Double fontSize, Double lineHeight, Color textColor) {
        ...     
    }
}
```

TextBox 클래스는 상대적으로 하위 수준의 코드일 가능성이 크기에 renderText() 함수는 그 보다 상위 수준의 다른 함수에 의해

호출되고, 그 함수는 또다시 더 높은 층위의 다른 함수에 의해 호출될 것이다.

### 8.5.2 해결책 : 관련된 데이터는 객체 또는 클래스로 그룹화하라.

```java
@AllArgsConstructor
@Getter
class TextOptions {
    private final Font font;
    private final Double fontSize;
    private final Double lineHeight;
    private final Color textColor;
}
```

이제 TextOptions 클래스를 사용하여 텍스트 스타일 정보를 함께 캡슐화해서 TextOptions 인스턴스를 전달할 수 있다

```java
class UiSettings {
    ...

    TextOptions getTextStyle() {...}
}

class UserInterface {
    private final TextBox messageBox;
    private final UiSettings uiSettings;

    void displayMessage(String message) {
        messageBox.renderText(
            message, uiSettings.getTextStyle());
    }
}

class TextBox {
    ...

    void renderText(String text, TextOptions textStyle){
        ...
    }
}
```

## 8.6 반환 유형에 구현 세부 정보가 유출되지 않도록 주의하라

### 8.6.1 반환 형식에 구현 세부 사항이 유출될 경우 문제가 될 수 있다

아래의 코드는 요청이 성공했는지를 나타내는 정보와 프로필 사진의 이미지 데이터를 보관하는 정보가 유출된다

```java
class ProfilePictureService {
    private final HttpFetcher httpFetcher;
    ...

    ProfilePictureResult getProfilePicture(Int64 userId){...}
}

class ProfilePictureResult {
    ...
    /**
     *  프로필 사진에 대한 요청이 성공인지 여부를 나타낸다
     */
    HttpResponse.Status getStatus() {...}

    /**
     * 프로필 사진이 발견된 경우 그 사진의 이미지 데이터
     */ 
    HttpResponse.Payload getImageData() {...}
}
```

### 8.6.2 해결책: 추상화 계층에 적합한 유형을 반환하라

ProfilePictureService 클랙스가 해결하는 문제는 사용자의 프로필 사진을 가져오는 것이다. 따라서 이 클래스를 통해

제공하고자 하는 이상적인 추상화 계층과 모든 반환 형식은 이점을 반영해야 한다. 이 클래스를 사용하는 다른 개발자에게

노출되는 개념이 최소가 되도록 노력해야 한다

```java
class ProfilePictureService {
    private final HttpFetcher httpFetcher;
    ...

    ProfilePictureResult getProfilePicture(Int64 userId){...}
}

class ProfilePictureResult {
    ...

    enum Status {
        SUCCESS,
        USER_DOES_NOT_EXIST,
        OTHER_ERROR
    }

    Status getStatus(){...}

    List<Byte> getImageData(){...}
}
```

## 8.7 예외 처리 시 구현 세부 사항이 유출되지 않도록 주의하라

### 8.7.1 예외 처리 시 구현 세부 사항이 유출되면 문제가 될 수 있다

비검사 예외의 핵심 기능 중 하나는 예외가 발생하는 위치나 시기, 코드가 어디에서 그 예외를 처리하는지 등에 대해

그 어떠한 것도 컴파일러에 의해 강제되지 않는다는 것이다. 비검사 예외에 대한 지식은 코드 계약의 세부 조항을 통해

전달되지만 개발자가 문서화하는 것을 잊어버리면 코드 계약을 통해 전혀 전달되지 않는다.

```java
class TextSummarizer {
    private final TextImportanceScorer importanceScorer;
    ...

    String summeraizeText(String text) {
        return paragraphFinder.find(text)
            .filter(paragrph -> 
                importanceScorer.isImportant(paragraph))
                .join("\n\n");
    }
}

interface TextImportanceScorer {
    Boolean isImportant(String text);
}

class ModelBasedScorer implements TextImportanceScorer {

    @Override 
    Boolean isImportant(String text) {
        return model.predict(text) >= MODEL_THRESHOLD;
    }
}
```

이와 같은 경우 추상화 계층 개념을 위반할 뿐만 아니라 신뢰할 수 없고 오류를 일으키기 쉽다.

TextSummarizer 클래스는 TextImportanceScorer 인터페이스에 의존하므로 이 인터페이스를 구현하는 어떤 클래스로도

설정할 수 있다.

특정 구현에 종속된 예외 처리의 예시는 다음과 같다

```java
void updateTextSummary(UserInterface ui) {
    String userText = ui.getUserText();
    try {
        String summary = textSummarizer.summarizeText(userText);
        ui.getSumaryFiled().setValue(summary);
    } catch (PredictionModelException e) {
        ui.getSummaryFiled().setError("Unable to summarize text");
    }
}
```

구현 세부 정보가 유출될 위험이 비검사 예외에만 있는 것은 아니지만, 이 예에서 비검사 예외로 인해 문제가 더욱 악화되어

발생한다. 비검사 예외를 발생할 수 있다는 점을 개발자가 문서화하지 않을 가능성이 크고, 인터페이스를 구현하는 클래스가 반드시

인터페이스가 규정하는 오류만 발생시켜야만 하는 것은 아니다

### 8.7,2 해결책: 추상화 계층에 적절한 예욀르 만들라

구현 세부사항의 유출을 방지하기 위해 코드의 각 계층은 주어진 추상화 계층을 반영하는 오류 유형만을 드러내는 것이 이상적이다

이것은 하위 계층의 오류를 현재 계층에 적합한 오류 유형으로 감사면 가능하다

이렇게 하면 호출하는 쪽에 적절한 추상화 계층이 제시되면서 동시에 원래의 오류 정보가 손실되지 않는다는 것을 의미한다

```java
class TextSummarizerException extends Exception {
    ...
    TextSummarizerException(Throwable cause){...}
    ...
}

class TextSummairzer {
    private final TextImportanceScorer importanceScorer;

    String summarizeText(String text){
        try {
            return paragraphFinder.find(text)
                .filter(paragraph ->
                    importanceScorer.isImportant(paragraph))
                .join("\n\n");
        } catch (TextImportanceScorerException e){
            throw new TextSummarizerException(e);
        }
    }
}

class TextImportanceScorerException extends Exception {
    ...
    TextImportanceScorerException(Throwable cause){...}
    ...
}

interface TextImportanceScorer {
    Boolean isImportant(String text) throws TextImportanceScorerException;
}

class ModelBasedScorer implements TextImportanceScorer {
    ...
    Boolean isImportant(String text) throws TextImportanceScorerException {
        try{
            return model.predict(text) >= MODEL_THRESHOLD;
        }catch (PredictionModelException e){
            throw new TextImportanceScorerException(e);
        }
    }
}
```

# 8장 요약

- 코드가 모듈화되어 있으면 변경된 요구 사항을 적용하기 위한 코드를 작성하기가 쉽다
- 모듈화의 주요 목표 중 하나의 요구 사항의 변경이 해당 요구 사항과 직접 관련된 코드에만 영향을 미치도록 하는 것이다
- 코드를 모듈식으로 만드는 것은 간결한 추상화 계층을 만드는 것과 관련이 있다
- 다음의 기술을 사용하여 코드를 모듈화 할 수 있다
  - 의존성 주입
  - 구체적인 클래스가 아닌 인터페이스에 의존
  - 클래스 상속 대신 인터페이스 및 구성의 활용
  - 클래스는 자신의 기능만 처리
  - 관련된 데이터의 캡슐화
  - 반환 유형 및 예외 처리 시 구현 세부 정보 유출 방지

