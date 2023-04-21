# 6. 예측 가능한 코드를 작성하라.

## 6.1 매직값을 반환하지 말아야 한다

`매직값`은 함수의 정상적인 반환 유형에 적합하지만 특별한 의미를 가지고 있다. 매직값의 일반적인 예는 값이 없거나 오류가 발생했음을 나타내기 위헤

-1을 반환하는 것이다. 함수의 정상적인 반환 유형에 들어맞기 때문에 이 값이 갖는 특별한 의미를 인지하지 못하고, 경계하지 않으면

정상적인 반환값으로 오인하기 쉽다

### 6.1.1 매직값은 버그를 유발할 수 있다. 

```kotlin
fun getMeanAge(users: List<User>) Double? {
    if (users.isEmpty())
        return null
    
    var sumOfAges = 0.0

    for (user in users) {
        sumOfAges += user.getAge().toDouble()
    }

    return sumOfAges / users.size().toDouble()
}

class User(
    val age: Int?
) {
    fun getAge() : Int {
        if (age == null)
            return -1
        return age
    }
}
```

이러한 경우 `user.getAge`가 호출자에게 명시적으로 알리지 않고 마이너스 1을 반환하기에 예측하지 못할 수 있다.

### 6.1.2 해결책: 널, 옵셔녈 또는 오류를 반환하라

```kotlin
class User(
    val age: Int?
) {
    fun getAge() : Int? {
        return age
    }
}
fun getMeanAge(users: List<User>) Double? {
    if (users.isEmpty())
        return null
    
    var sumOfAges = 0.0

    for (user in users) {
        sumOfAges += user.getAge().toDouble()
    }

    return sumOfAges / users.size().toDouble()
}
```

다음과 같이 수정하면 getMeanAge() 함수는 컴파일러 오류를 발생시키며 이 함수를 작성한 개발자는 코드에 잠재적 버그가 있다는 것을 깨닫는다

따라서 getMeanAge() 코드를 컴파일하기 위해 개발자는 User.getAge()가 널을 반환하는 경우를 처리해야 한다.

### 6.1.3 때때로 매직값이 우연히 발생할 수 있다.

예를 들어 최솟값 찾기에 대한 기능은 다음과 같다

```kotlin
fun minValue(values: List<Int>) : Int {
    var minValue = Int.MAX_VALUE
    for (value in values) {
        minValue = minof(value, maxValue)
    }
    return minValue
}
```

이에 대한 문제는 values가 비어있다면 문제가 있을 수 있기에 이를 수정하면 다음과 같다

```kotlin
fun minValue(values: List<Int>) : Int? {

    if (values.isEmpty())
        return null
    
    var minValue = Int.MAX_VALUE
    
    for (value in values) {
        minValue = minof(value, maxValue)
    }

    return minValue
}
```

매직값을 반환하는 것이 때로는 개발자가 의식적으로 내린 결정이지만, 어떤 겨우에는 우연히 일어날 수도 있다.

이유 여하를 막론하고 매직값은 예측을 벗어나는 결과를 초래할 수 있기 때문에 발생 가능한 상황에 대해 조심하는 것이 좋다.



## 6.2 널 객체 패턴을 적절히 사용하라.

### 6.2.1 빈 컬렉션을 반환하면 코드가 개선될 수 있다.

함수가 리스트, 집합, 배열과 같은 컬렉션을 반환할 때 컬렉션의 값을 얻을 수 없는 경우가 있다. 값이 지정이 안됐다거나

주어진 상황에서 컬렉션에 값이 없을 수 있다. 이 경우 한 가지 방법은 널값을 반환하는 것이다.

```kotlin
fun getClassNames(element : HtmlElement) : Set<String>? {
    val attribute = element.getAttribute("class")
    if (attribute == null)
        return null
    return SetOf(attribute.split(" "))
}

fun isElementHighlighted(element : HtmlElement) {
    val classNames = getClassNames(element)
    if (classNames == null)
        return false
    return classNames.contains("highlighted")
}

```

위와 같은 코드는 항상 호출하는 쪽에서 반환된 값이 널인지 여부를 확인한 후 사용해야 한다. 

이것은 별 이점없이 코드만 지저분하게 만드는데 널 값의 경우와 동일하게 'class' 속성이 설정되지 않은 것과 빈 문자열로 설정된

것의 차이를 구분할 일이 거의 없기 때문이다.

따라서 이런 상황에서는 다음과 같이 개선할 수 있다

```kotlin
fun getClassNames(element : HtmlElement) : Set<String> {
    val attribute = element.getAttribute("class")
    if (attribute == null)
        return SetOf()
    return SetOf(attribute.split(" "))
}

fun isElementHighlighted(element : HtmlElement) {
    return getClassNames(element).contains("highlighted")
}
```

빈 집합 객체를 반환해서 널 여부를 확인할 필요가 없게 만드는 방식이 있다.

### 6.2.2 빈 문자열을 반환하는 것도 때로는 문제가 될 수 있다

```kotlin
class UserFeedback(
    val additionalComments : String?
) {

    fun getAdditionalComments() : String {
        if (additionalComments == null) 
            return ""
        return additionalComments
    }
}
```

위와 같은 코드가 있다고 가정하자. 널이면 빈 문자열을 반환함으로써 널 객체 패턴을 사용한다.

개발자들은 이 필드는 항상 널이 되지 않는 것으로 생각하고 따라서 카드 트랜잭션이라고 생각할 수 있기 떄문에 문제의 소지가 있다

```kotlin
class Payment(
    val cardTransactionId : String?
) {

    fun getCardTransactionId() : String {
        if (cardTransactionId == null)
            return ""
        return cardTransactionId
    }
}
```

이러한 코드는 cardTransactionId가 널일 때 getCardTransactionId 함수가 널을 반환하는 것이 훨씬 더 낫다.

이로 인해 호출하는 쪽에서 결제가 카드 거래를 수반하지 않을 수도 있다는 점을 명확하게 알 수 있기 때문에 코드는 예측을 벗어나지 않는다

이렇게 하면 코드는 다음과 같이 된다

```kotlin
class Payment (
    val cardTransactionId : String?
) {
    fun getCardTransactionId(): String? {
        return cardTransactionId
    }
}
```

## 6.3 예상치 못한 부수 효과를 피하라

`부수 효과`는 어떤 함수의 호출이 함수 외부에 초래한 상태 변화를 의미한다. 함수가 반환하는 값 외에 다른 효과가 있다면 이는 부수효과가 있는 것이다

### 6.3.1 분명하고 의도적인 부수 효과는 괜찮다

```kotlin
class UserDisplay(
    val canvas : Canvas
) {
    fun displayErrorMessage(message: String) {
        canvas.drawText(message, Color.RED) // 캔버스가 업데이트 된다
    }
}
```

displayErrorMessage() 함수는 분명하고 의도적으로 부수 효과를 일으키는 예이다. 오류 메시지로 캔버스를 업데이트 하는 것은

호출하는 쪽에서 원하고 예상하는 바이다. 반면에, 호출하는 쪽에서 반드시 예상하거나 원하지 않는 부수효과는 문제가 될 수 있다

### 6.3.2 예기치 않은 부수 효과는 문제가 될 수 있다

```kotlin
class UserDisplay(
    val canvas : Canvas
) {
    
    fun getPixel(x: Int, y: Int) : Color { // 다시 그리기 이벤트를 발생하는 것은 부수효과다.
        canvas.redraw()
        val data = canvas.getPixel(x, y)
        return Color(data.getRed(), data.getGreen(), data.getBlue())
    }
}
```

## 6.4 입력 매개변수를 수정하는 것ㅇ ㅔ주의하라

### 6.4.1 입력 매개변수를 수정하면 버그를 초래할 수 있다.

```java
List<Invoice> getBillabaleInvoices(
    Map<User, Invocie> userInvoices,
    Set<User> usersWithFreeTrial) {
        userInvoices.removeAll(usersWithFreeTrial); // 무료 평가판을 사용할 수 있는 유저를 삭제함으로써 userInvocies를 변경함
        return userInvoices.values(); 
    }

void processOrders(OrderBatch orderBatch) {
    Map<User, Invocie> userInvoices = 
        orderBatch.getUserInvoices();
    Set<User> usersWithFreeTrial = 
        orderBatch.getFreeTrialUsers();
    
    sendInvocies(
        getBilliableInvoices(userInvocies, usersWithFreeTrial)); // getBillableInvoices()는 예상과 다르게 userInvocies를 변경
        enableOrderServices(userInvoices);
}
```

이 버그는 getBillableInvoices() 함수가 userInvoices 맵을 변경하기 때문에 발생한다.

### 6.4.2 해결책: 변경하기 전에 복사하라

입력 매개 변수 내의 값을 어쩔 수 없이 변경해야 하는 경우에는 변경 전에 새 자료구조에 복사하는 것이 최상의 방법이다.

이렇게 하면 원래의 객체가 변경되지 않는다.

```java
List<Invoice> getBillableInvoices(
    Map<User, Invoice> userInvoices,
    Set<User> usersWithFreeTrial) {
        return userInvoices
            .entries()
            .filter(entry -> 
                !usersWithFreeTrial.contains(entry.getKey()))
            .map(entry -> entry.getValue());
    }
```

## 6.5 오해를 일으키는 함수는 작성하지 말라

### 6.5.1 중요한 입력이 누락되었을 때 아무것도 하지 않으면 놀랄 수 있다.

매개변수가 없더라도 호출할 수 있고 해당 매개변수가 없으면 아무 작업도 수행하지 않는 함수가 있다면, 이 함수가 수행하는 작업에 대해

오해의 소지가 있을 수 있다. 호출하는 쪽에서는 해당 매개변수의 값을 제공하지 않고 함수를 호출하는 것의 심각성을 모를 수 있으며,

코드를 읽는 사람은 함수 호출 시 항상 무언가 작업이 이루어진다고 잘못 생각할 수 있다

```kotlin
class UserDisplay(
    val final messages : LocalizedMessages
) {
    fun displayLegalDisclaimer(legalText: String?) {
        if (legalText == null)
            return
        displayOverlay(
            title = messages.getLegalDisclaimerTitle()
            message = legalText
            textColor = Color.RED
        )
    }
}
```

### 6.5.2 해결책: 중요한 입력은 필수 항목으로 만들라

```kotlin
class UserDisplay(
    val final messages : LocalizedMessages
) {
    fun displayLegalDisclaimer(legalText: String) {
        displayOverlay(
            title = messages.getLegalDisclaimerTitle()
            message = legalText
            textColor = Color.RED
        )
    }
}
```

## 6.6 미래를 대비한 열거형 처리

### 6.6.1 미래에 추가될 수 있는 열거값을 암묵적으로 처리하는 것은 문제가 될 수 있다.

```kotlin
enum PredicatedOutcome {
    COMPANY_WILL_GO_BUST,
    COMPANY_WILL_MAKE_A_PROFIT
}

...

fun isOutcomeSafe(prediction: PredictedOutcome) : Boolean {
    if (predication == PredicatedOutcome.COMPANY_WILL_GO_BUST)
        return false
    return true
}
```

위 예제 코드들은 열거값이 2개인 동안에는 작동한다. 그러나 만약 누군가 새로운 열거값을 추가한다면 심각하게 잘못될 수 있다.

```kotlin
enum PredicatedOutcome {
    COMPANY_WILL_GO_BUST,
    COMPANY_WILL_MAKE_A_PROFIT,
    WOLRD_WILL_END // 추가된 새 열거값
}
```

이제 isOutComeSafe() 함수가 수정되지 않으면 WORLD_WILL_END 예측에 대해 참이 반환되어 안전한 결과임을 나타낸다

분명히 WORLD_WILL_END는 안전한 결과가 아니며 비즈니스 전략을 시작한다면 재앙이 일어날 수 있다.

```kotlin
fun isOutcomeSafe(prediction: PredictedOutcome) : Boolean {
    if (predication == PredicatedOutcome.COMPANY_WILL_GO_BUST)
        return false
    return true
}
```

### 6.6.2 해결책 : 모든 경우를 처리하는 스위치 문을 사용하라


```kotlin
enum PredicatedOutcome {
    COMPANY_WILL_GO_BUST,
    COMPANY_WILL_MAKE_A_PROFIT
}


fun isOutcomeSafe(prediction : PredicatedOutcome) : Boolean {
    switch(prediction) {
        case COMPANY_WILL_GO_BUST :
            return false
        case COMPANY_WILL_MAKE_A_PROFIT:
            return true
    }
    throw UncheckedException()
}

```

이와 같이 switch문을 사용하였을 경우

임의의 값에 대해 예외가 발생하면 테스트가 실패하고 PredicatedOutCome에 새 값을 추가한 개발자는

isOutcomeSafe() 함수도 변경해야 함을 알게된다.


# 6장 요약

- 다른개발자가 작성하는 코드는 종종 우리가 작성하는 코드에 의존한다
  - 다른 개발자가 우리 코드의 기능을 잘못 해석하거나 처리해야 하는 특수한 경우를 발견하지 못하면, 우리가 작성한 코드에

    기반한 그 코드에서 버그가 발생할 가능성이 크다

  - 코드를 호출하는 쪽에서 예상한대로 동작하기 위한 좋은 방법 중 하나는 중요한 세부 사항이 코드 계약의 명백한 부분에 포함되도록 하는것이다
- 우리가 사용하는 코드에 대해 허술하게 가정을 하면 예상을 벗어나는 또 다른 결과를 볼 수 있다
  - 예를 들어 열거형에 추가되는 새 값을 예상하지 못한 경우
  - 의존해서 사용 중인 코드가 가정을 벗어날 경우, 코드 컴파일을 중지하거나 테스트가 실패하도록 하는 것이 중요하다
- 테스트만으로는 예측을 벗어나는 코드의 문제를 해결할 수 없다. 다른 개발자가 코드를 잘못 해석하면 테스트해야 할 시나리오도 잘못이해할 수 있다

