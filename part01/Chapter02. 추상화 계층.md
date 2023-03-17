# 2. 추상화 계층

> 코드를 구성하는 방법은 코드 품질의 기본적인 측면 중 하나이며, 코드를 잘 구성한다는 것은 간결한 
> `추상화 계층`을 만드는 것으로 귀결될 떄가 많다.

## 2.1 널값 및 의사코드 규약

사용 중인 언어가 널 안정성을 지원하지 않는다면, 널값을 사용하는 대신 옵셔널 타입을 사용하는 것이 좋다.

표준 기능으로 지원하지 않는 언어에서도 타사 유틸리티를 통해 지원되는 경우가 많다

```kotlin
fun getFifthElement(elements:List<Element>) : Elements? {
    if (elements.size() < 5) {
        return null
    }
    return elements[4]
}
```

위 코드를 자바로 표현한다면 다음과 같다

```java
Optional<Element> getFifthElement(List<Element> elements) {
    if (elements.size() <5) {
        return Optional.empty();
    }
    return Optional.of(elements[4]);
}
```

## 2.2 왜 추상화 계층을 만드는가?

코드 작성은 복잡한 문제를 계속해서 더 작은 하위 문제로 세분화하는 작업이다.

사용자의 어떤 장치에서 실행되면서 서버에 메시지를 보내는 코드를 작성한다 가정하자. 다음과 같은 개념만 다루면 된다.

- 서버의 URL
- 연결
- 메시지 문자열 보내기
- 연결 닫기

```java
HttpConnection connection = HttpConnection.connect("http://example.com/server");
connection.send("Hello server");
connection.close();
```

높은 층위에서는 이것이 간단한 문제처럼 보이고 해결책도 간단해 보인다. 그러나 이것이 간단한 문제가 아니라는 것은 분명하다.

클라이언트 장치에서 서버로 'Hello server'라는 문자열을 보내는데는 다음 같은 복잡한 일이 일어난다

- 전송할 수 있는 형식으로 문자열 직렬화
- HTTP 프로토콜의 모든 복잡한 동작
- TCP 연결
- 사용자의 장치가 와이파이 혹은 셀룰러 네트워크에 연결되어 있는지 여부 확인
- 데이터를 라디오신호로 변조
- 데이터 전송 오류 및 수정

이렇게 복잡하지만 다행스럽게 다른 개발자들이 이 모든 하위 문제를 이미 해결했을 뿐더러 그것들을 인식할 필요도 없게 만들었다.

하나의 문제가 있을 때 이 문제와 하위 문제에 대한 해결책이 일련의 층을 형성하고 있는 것으로 생각할 수 있다. 최상위 계층에서는 HTTP프로토콜이

어떻게 구현되는지 알 필요도 없이 서버에 메세지를 보내는 것에만 신경을 쓰면서 코드를 작성할 수 있다. 이와 비슷하게 HTTP프로토콜을 구현하기 위한

코드를 작성한 엔지니어는 데이터가 무선 신호에 변조되는 방법에 대해 아무것도 몰라도 문제가 없었을 것이다.

HttpConnection 코드를 구현한 개발자는 물리적인 데이터 전송을 추상적인 개념으로 생각할 수 있었고, 우리 역시 HTTP 연결을 추상적인 개념으로 생각할 수 있다

이것을 `추상화 계층`이라고 한다.

![image](https://user-images.githubusercontent.com/40031858/224547233-d7767bed-dffa-4209-a540-caf596db8b70.png)

### 2.2.1 추상화 계층 및 코드 품질의 핵심 요소

깨끗하고 뚜렷한 추상화 계층을 구축하면 코드 품질의 네 가지 핵심요소를 달성할 수 있다.

`가독성`

    개발자들이 코드베이스에 있는 코드의 모든 세부사항을 이해하는 것은 불가능하지만 몇 가지 높은 계층의 추상화를 이해하고

    사용하기는 쉽다. 깨끗하고 뚜렷한 추상화 계층을 만드는 것은 개발자가 한 번에 한두 개 정도의 계층과 몇 개의 개념만 다루면 된다는 것을 의미한다

    따라서 코드의 가독성이 향상된다

`모듈화`

    추상화 계층이 하위 문제에 대한 해결책을 깔끔하게 나누고 구현 세부 사항이 외부로 노출되지 않도록 보장할 때, 다른 계층이나 코드의 일부에

    영향을 미치지 않고 계층 내에서만 구현을 변경하기가 쉬워진다.

`재사용성 및 일반화성`

    하위 문제에 대한 해결책이 간결한 추상화 계층으로 제시디면 해당 하위 문제에 대한 해결책을 재사용하기가 쉬워진다.

    문제가 적절하게 추상적인 하위 문제로 세분화된다면, 해결책은 여러가지 다른 상황에서 유용하게 일반화될 가능성이 크다

`테스트 용이성`

    신뢰할 수 있는 코드를 작성하고자 한다면, 각 하위 문제에 대한 해결책이 견고하고 제대로 작동하는지 확인해야 한다.

    코드가 추상화 계층으로 깨끗하게 분할되면 각 하위 문제에 대한 해결책을 완벽하게 테스트하는 것이 훨씬 쉬워진다.

## 2.3 코드의 계층

대부분의 프로그래밍 언어는 코드를 다른 단위로 나누기 위해 몇가지 언어 요소를 자유롭게 사용할 수 있다. 대체적으로 그 요소는 다음과 같다

- 함수
- 클래스 (및 구조체나 믹스인과 같이 클래스와 비슷한 요소도 가능)
- 인터페이스(또는 이와 동일한 요소)
- 패키지, 네임스페이스, 모듈

### 2.3.1 API 및 구현 세부 사항

코드를 작성할 떄 고려해야 할 측면이 두 가지 있다

- 코드를 호출할 때 볼 수 있는 내용:
  - 퍼블릭 클래스, 인터페이스 및 함수
  - 이름, 입력 매개변수 및 반환 유형이 표현하고자 하는 개념 
  - 코드 호출 시 코드를 올바르게 사용하기 위해 알아야 하는 추가 정보
- 코드를 호출할 때 볼 수 없는 내용 : 구현 세부 사항

API는 서비스를 사용할 때 알아야 할 것들에 대한 개념을 형식화 하고, 서비스의 모든 구현 세부사항은 이 API뒤에 감춘다

API는 호출하는 쪽에 공개할 개념만 정의하면 되고 그 이외의 모든 것은 구현 세부사항이기 때문에 코드를 API의 관점에서 생각하면 추상화

계층을 명확하게 만드는데 도움이 된다. 코드의 일부를 작성하거나 수정할 때, API에 이 수정사항에 대한 구현 세부 정보가 새어 나간다면

추상화 계층이 명확하게 구분되어 이루어진 것이 아니다

### 2.3.2 함수

다음과 같은 함수가 있다고 하자

```kotlin
fun sendOwnerALetter(vehicle : Vehicle, letter : Letter): SentConfirmation? {
    
    // 소유자의 주소를 찾기 위한 자세한 로직
    var ownersAddress : Address? = null
    if (vehicle.hasBeenScraped())
        ownersAddress = SCRAPYARD_ADDRESS
    else {
        var mostRecentPurchase : Purchase? = vehicle.getMostRecentPurchase()
        if (mostRecentPurchase == null) 
            ownersAddress = SHOWROOM_ADDRESS
        else
            ownersAddress = mostRecentPurchase.getBuyerAddress()
    }

    // 조건부로 편지를 보내는 로직
    if (ownerAddress == null) {
        return null
    }
    return sendLetter(ownersAddress, letter)
}
```

위의 함수는 두 가지 작업, 즉 차량 소유자의 주소를 찾고 편지 보내기를 요청하는 작업을 수행한다. 그러나 다른 함수를 사용해

이 기능을 구현하지 않고, 차량 소유자의 주소를 찾기 위한 자세한 로직을 함수 자신이 직접 구현하고 있다.

더 나은 접근법은 소유자의 주솔르 찾는 로직을 다른 함수로 구현하는 것이다. 이렇게 하면 sendOwnerALetter()함수는 보다 이상적인 문장으로 표현할 수 있다

```kotlin
fun sendOwnerALetter(vehicle : Vehicle, letter : Letter): SentConfirmation? {

    // 소유자의 주소를 찾는다
    val ownerAddress : Address? = getOwnersAddress(vehicle) 

    if (ownerAddress == null) 
        return null
    return sendLetter(ownersAddress, letter)
}

// 소유자의 주솔르 찾기 위한 함수 (재사용이 쉽다)
private fun getOwnerAddress(vehicle: Vehicle): Address? {
    if (vehicle.hasBeenScraped())
        return SCRAPYARD_ADDRESS

    val mostRecentPurchase : Purchase? = vehicle.getMostRecentPurchase()

    if (mostRecentPurchase == null)
        return SHOWROOM_ADDRESS
    return mostRecentPurchase.getBuyersAddress()
}
```

함수를 작게 만들고 수행하는 작업을 명확하게 하면 코드의 가독성과 재사용성이 높아진다. 코드를 마구 작성하다 보면 너무 길어서 읽을 수 없는 함수가 되기 쉽다.

따라서 코드 작성을 일단 마치고 코드 검토를 요청하기 전에 자신이 작성한 코드를 비판적으로 다시 한번 살펴보는 것이 좋다.

### 2.3.3 클래스

다음과 같이 텍스트를 요약하는 클래스가 있다고 하자. 텍스트 요약을 위해 이 코드는 먼저 텍스트를 단락으로 나누고, 중요도가 낮다고 판단되는 단락을 누락한다

텍스트 요약이라는 문제를 풀 때 이 클래스의 작성자는 하위 문제를 풀어야한다. 여러 함수로 나누어 추상화 계층을 만들었지만 여전히 하나의 클래스가

모든 것을 다 가지고 있다. 이것은 추상화 계층들 사이의 분리가 그다지 뚜렷하지 않다는 것을 의미한다

```kotlin
class TextSummarizer {
    ...
    fun summarizeText(text: String): String {
        return splitIntoParagraphs(text)
            .filter(paragraph -> calculateImportance(paragraph) >=
                IMPORTANCE_THRESHHOLD)
                .join("\n\n")
    }

    private calculateImportance(paragraph : String) : Double{
        val nouns : List<String> = extractImportanceNouns(paragraph)
        val verbs : List<String> = extractImportantVerbs(paragraph)
        val adjectives : List<String> = extractImportantAdjectives(paragraph)
        ...
        return importanceScore
    }

    private fun extractImportanceNouns(text: String): List<String> {...}
    private fun extractImportantVerbs(text: String): List<String> {...}
    private fun extractImportantAdjectives(text: String): List<String> {...}

    private fun splitIntoParagraphs(text: String) : List<String> {
        val paragraphs = mutableListOf<String>()
        var start : Int? = detectParagraphStartOffset(text,0)
        while (start != null) {
            val end : Int? = detectParagraphEndOffset(text, start)
            if (end == null)
                break
            paragrphs.add(text.subString(start,end))
            start = detectParagraphStartOffset(text, end)
        }
        return paragraphs
    }
    private fun detectParagraphStartOffset(text: String, fromOffset : Int): Int? {
        ...
    }
    private fun detectParagraphEndOffset(text: String, fromOffset : Int) : Int? {
        ...
    }
}
```

이 클래스는 단지 한 가지 일, 즉 텍스트를 요약하는 것에만 관련이 있고 높은 층위에서 보면 어느정도 맞는 말이다.

하지만 이 클래스는 여러 하위 문제를 해결하는 코드를 가지고 있다

- 텍스트를 단락으로 분할
- 텍스트 문자열의 중요도 점수 계산
  - 이 하위 문제는 또 다시 중요한 명사, 동사, 형용사를 갖는 하위 문제로 나뉜다.

이 클래스가 분리되어야 할지 판단하기 위해서는 이 클래스가 어떻게 네 가지 핵심 요소에 반해서 작성되어 있는지 살펴보는 것이 더 나을 수 있다.

- `코드를 읽을 수 없다`
- `코드가 특별히 모듈화되어 있지 않다`
- `코드를 재사용할 수 없다`
- `코드를 일반화할 수 없다`
- `코드를 제대로 테스트하기 어렵다`

즉 위의 TextSummarizer 클래스는 너무 크고 너무 많은 개념을 처리하므로 코드 품질이 낮다.

### `코드 개선 방법`

앞선 예제 코드는 각 하위 문제에 대한 해결책을 자체 클래스로 분할하여 개선할 수 있다. 하위 클래스는 생성자의 매개 변수를 통해 TextSummarizer 클래스에 제공된다

```kotlin
class TextSummarizer (
    private val paragraphFinder : ParagraphFinder,
    private val importanceScorer : TextImportanceScorer
) {
    companion object {
        fun createDefault() : TextSummarizer {
            return TextSummarizer(
                ParagraphFinder(),
                TextImportanceScorer()
            )
        }
    }
    
    fun summarizeText(text: String) : String {
        return paragraphFinder.find(text)
            .filter {it -> importanceScorer.isImportant(paragraph) }
            .joinTo("\n\n")
    }
}

class ParagraphFinder {
    fun find(text: String) : List<String> {
        val paragraphs = mutableListOf<String>()
        var start = detectParagraphStartOffset(text, 0)
        while (start != null ){
            val end = detectParagraphEndOffset(text, start)
            if (end == null)
                break
            paragraphs.add(text.subString(start, end))
            start = detectParagraphStartOffset(text, end)
        }
        return paragraphs
    }

    fun detectParagraphEndOffset(text: String, fromOffset : Int)  : Int? {
        ...
    }

    fun detectParagraphEndOffset(text: String, fromOffset : Int) : Int? {
        ...
    }
}

class TextImportanceScorer {
    ...

    fun isImportant(text: String) : Boolean {
        return calculateImportance(text) >= IMPORTANCE_THRESHOLD
    }
}

    private calculateImportance(paragraph : String) : Double{
        val nouns : List<String> = extractImportanceNouns(paragraph)
        val verbs : List<String> = extractImportantVerbs(paragraph)
        val adjectives : List<String> = extractImportantAdjectives(paragraph)
        ...
        return importanceScore
    }

    private fun extractImportanceNouns(text: String): List<String> {...}
    private fun extractImportantVerbs(text: String): List<String> {...}
    private fun extractImportantAdjectives(text: String): List<String> {...}
```

이 코드는 각 클래스마다 몇 개의 개념만 파악하면 되므로 코드의 가독성이 훨씬 좋아졌다고 할 수 있다.

TextSummarizer 클래스를 살펴보면 몇 초 만에 높은 층위의 알고리즘을 구성하는 모든 개념과 단계를 알 수 있다

- 단락을 찾는다
- 중요하지 않은 것은 걸러낸다
- 남아 있는 단락을 연결한다

앞선 코드보다 다음과 같은 이점이 있다

- `코드가 좀 더 모듈화되고 재구성할 수 있게 됐다`
- `코드의 재사용성이 좀 더 높아졌다`
- `코드의 테스트 용이성이 좀 더 높아졌다`

### 2.3.4 인터페이스 

계층 사이를 뚜렷이 구분하고 구현 세부 사항이 계층 사이에 유출되지 않도록 하기 위해 사용할ㅇ 수 있는 한 가지 접근법은 어떤 함수를 외부로 노출할 것인지를

인터페이스를 통해 결정하는 것이다. 그 다음 이 인터페이스에 정의된 대로 클래스가 해당 계층에 대한 코드를 구현한다. 이보다 위에 있는 계층은

인터페이스에 의존할 뿐 구현하는 구체적인 클래스에 의존하지 않는다.

```kotlin
interface TextImportanceScorer {
    fun isImportant(text: String) : Boolean
}

class WordBasedScorer : TextImportanceScorer {
    ...

    override fun isImportant(text: String) : Boolean {
        return calculateImportance(text) >= IMPORTANCE_THRESHOLD
    }

    private calculateImportance(paragraph : String) : Double{
        val nouns : List<String> = extractImportanceNouns(paragraph)
        val verbs : List<String> = extractImportantVerbs(paragraph)
        val adjectives : List<String> = extractImportantAdjectives(paragraph)
        ...
        return importanceScore
    }
    private fun extractImportanceNouns(text: String): List<String> {...}
    private fun extractImportantVerbs(text: String): List<String> {...}
    private fun extractImportantAdjectives(text: String): List<String> {...}  
}

class ModelBasedScorer(
    private val model : TextPredictionModel) : TextImportanceScorer {

    companion object {
        return ModelBasedScorer(TextPrecitionModel.load(MODEL_FILE))
    }
    override fun isImportant(text : String) : Boolean {
        return model.predict(text) >= MODEL_THRESHHOLD
    }
}
```

이제 TextSummarizer가 두 가지 팩토리 함수 중 하나를 사용해 WordBasedScorer 또는 ModelBasedScorer를 사용하도록 구성할 수 있다.

다음은 두 가지 팩토리 함수를 보여준다

```kotlin
fun createWordBasedSummarizer() : TextSummarizer {
    return TextSummarizer(ParagraphFinder(), WordBasedScorer())
}

fun createModelBasedSummarizer() : TextSummarizer {
    return TextSummarizer(ParagraphFinder(), ModelBasedScorer.create())
}
```

추상화 계층을 깔끔하게 구현하는 코들르 만드는 데 있어 인터페이스는 매우 유용한 도구다. 주어진 하위 문제에 대한 둘 이상의 서로 다른 구체적인 구현이 가능하고

이들 구현 클래스 사이에 전환이 필요할 때는 추상화 계층을 나타내는 인터페이스를 정의하는 것이 가장 좋다

일르 통해 코드를 더욱 모듈화할 수 있고 재설정도 쉽게 할 수 있다.

### `모든 것을 위한 인터페이스?`

주어진 추상화 계층에 대해 한 가지 구현만 있고 향후에 다른 구현을 추가할 계획이 없더라도 여전히 인터페이스를 통해 추상화 계층을

표현해야 하는가는 고려할 사안이다. 몇 소프트웨어 철학은 이 상황에서도 인터페이스 사용을 권고한다.

이 권고를 따라 인터페이스 뒤로 TextSummarizer 클래스를 숨긴다면 다음과 같은 코드가 될것이다.

```kotlin
interface TextSummarizer {
    fun summarizeText(text: String): String
}

class TextSummarizerImpl : TextSummarizer {

    override fun summarizeText(text: String) : String {
        return paragraphFinder.find(text)
            .filter{it -> importanceScorer.isImportant(it)}
            .joinTo("\n\n")
    }
}
```

향후 다르게 구현할 필요가 있을지 현재는 알 수 없더라도 이 접근 방식은 몇가지 장점이 있다

- `퍼블릭 API를 매우 명확하게 보여준다` : 이 계층에서 사용해야 하는 기능과 사용하지 말아야 하는 기능에 대해 혼동할 일이 없다
- `테스트를 쉽게 할 수 있다` : 구현 클래스가 복잡하거나 네트워크 I/O에 의존하는 작업을 수행한다면 mock으로 대체할 수 있다
- `같은 클래스로 두 가지 하위 문제를 해결할 수 있다` : 한 클래스가 두 개 이상의 서로 다른 추상화 계층에 구현을 제공할 수도 있다

### 2.3.5 층이 너무 얇아질 때

코드를 별개의 계층으로 세분화하면 장점이 많지만 다음과 같은 추가 비용이 발생한다

- 클래스를 정의하거나 의존성을 새 파일로 임포트하려고 반복적으로 사용하는 코드로 인해 코드의 양이 늘어난다
- 로직의 이해를 위해 파일이나 클래스를 따라갈 때 더 많은 노력이 필요하다
- 인터페이스 뒤에 계층을 숨기게 되면 어떤 상황에서 어떤 구현이 사용되는지 파악하는데 더 많은 노력이 필요하다

불필요하게 계층을 잘게 쪼갠 코드를 보자

```kotlin
class ParagraphFinder (
    private val startDetector : OffsetDetector,
    private val endDetector : OffsetDetector
) {
    ...
    fun find(text: String) : List<String> {
        val paragraphs = mutableListOf<String>()
        var start = detectParagraphStartOffset(text, 0)
        while (start != null ){
            val end = detectParagraphEndOffset(text, start)
            if (end == null)
                break
            paragraphs.add(text.subString(start, end))
            start = detectParagraphStartOffset(text, end)
        }
        return paragraphs
    }
}

interface OffsetDetector {
    fun detectOffset(text: String, fromOffset: Int): Int?
}

class ParagraphStartOffsetDetector : OffsetDetector {
    override fun detectOffset(text: String, fromOffset : Int): Int? {
        ...
    }
}

class ParagraphEndOffsetDetector : OffsetDetector {
    override fun detectOffset(text: String, fromOffset: Int) : Int? {
        ...
    }
}
```

ParagraphFinder 클래스를 다른 곳에서 사용할 가능성이 있다 하더라도 ParagraphStartOffsetDetector는 사용하면서

ParagraphEndOffsetDetector는 사용하지 않는 경우를 상상하기는 어렵다. 왜냐하면 이 구현 클래스들은 단락의 시작과 끝을 감지하는 방법에

대해 서로 밀접하게 연관되어 있기 때문이다.

코드 계층의 규모를 올바르게 결정하는 것은 중요하다. 코드베이스에 의미 있는 추상화 계층이 없으면 전혀 관리할 수 없는 코드가 된다.

계층이 있더라도 각 계층이 너무 크면 쪼개져야 할 여러 추상화가 한 계층으로 병합되어 결국은 모듈화되지 않고, 재사용할 수 없으며,

가독성이 낮은 코드가 된다. 반면 계층을 너무 얇게 만들면 단일 계층으로 만들어도 될 것을 둘로 분해한 것이고, 이것은 불필요한 복잡성을 초래할 수 있다.

# 요약

- 코드를 깨끗하고 뚜렷한 추상화 계층으로 세분화하면 가독성, 모듈화, 재사용, 일반화 및 테스트 용이성이 향상된다
- 특정 언어에 국한된 기능뿐만 아니라 함수, 클래스 및 인터페이스를 사용하여 코드를 추상화 계층으로 나눌 수 있다
- 코드를 추상화 계층으로 분류하는 방법을 결정하려면 해결 중인 문제에 대한 판단과 지식을 사용해야 한다
- 너무 비대한 계층 때문에 발생하는 문제는 너무 얇은 계층 때문에 발생하는 문제보다 더 심각하다.

    확실하지 않은 경우에는 남용의 위험에도 불구하고 계층을 얇게 만드는 것이 좋다.