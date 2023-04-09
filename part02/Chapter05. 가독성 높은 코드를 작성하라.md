# 5. 가독성 높은 코드를 작성하라.

## 5.1 서술형 명칭 사용

### 5.1.1 서술적이지 않은 이름은 코드를 읽기 어렵게 만든다.

다음은 이름을 지을 때 서술적인 이름을 짓기 위한 어떠한 노력도 기울이지 않으면, 코드가 어떻게 보일지 보여주는 예이다.

```kotlin
class T {
    val pns = mutableSetOf<String>()
    var s = 0
    ...

    fun f(n:String) : Boolean {
        return pns.contains(n)
    }

    fun getS() : Int {
        return s
    }
}

fun s(ts: List<T> , n : String) : Int? {
    for (t in ts) {
        if (t.f(n))
        return t.getS()
    }
    return null
}
```

이 코드는 무엇을 하는지, 코드에 있는 문자열, 정수, 클래스가 어떤 개념을 나타내는지 전혀 모를 것이다

### 5.1.2 주석문으로 서술적인 이름을 대체할 수 없다.

위의 코드를 개선하는 방법 중 하나는 주석문과 문서를 추가하는 것이다. 하지만 여전히 다음과 같은 문제가 있다

- 코드가 훨씬 복잡해 보인다. 작성자와 다른 개발자는 코드뿐만 아니라 주석문과 문서도 유지보수해야한다
- 개발자는 코드를 이해하기 위해 계속해서 위아래로 스크롤해야한다. 코드를 파악할 때 getS()함수를 보고 변수 s의 용도를 잊어버린 경우

    s가 무엇인지 설명하는 주석을 찾기 위해 파일 맨위로 스크롤해야한다
- 함수 s()의 내용을 확인할 때 클래스 T를 살펴보지 않는 한 t.f(n)와 같은 호출이 무엇을 하는지 또는 무엇이 반환되는지 알기 어렵다

```kotlin
/** 팀을 나타낸다 */
class T {
    val pns = mutableSetOf<String>() //팀에 속한 선수의 이름
    var s = 0 // 팀의 점수
    ...
    /**
    * @param n 플레이어 의 이름
    * @return true 플레이어가 팀에 속해있는 경우
    */
    fun f(n:String) : Boolean {
        return pns.contains(n)
    }

    /**
    * @return 팀의 점수
    */
    fun getS() : Int {
        return s
    }
}

fun s(ts: List<T> , n : String) : Int? {
    for (t in ts) {
        if (t.f(n))
        return t.getS()
    }
    return null
}
```

이처럼 매개변수 및 반환 유형을 주석문으로 설명하는 것은 다른 개발자가 코드 사용방법을 이해하는데 도움이 될 수 있다.

하지만 서술적인 이름을 붙이는 대신 주석문을 사용하면 안된다.

### 5.1.3 해결책 : 서술적인 이름 짓기

서술적인 이름을 사용하면 조금 전에 살펴본 이해하기 어려운 코드가 갑자기 이해하기 쉬운 코드로 바뀐다

```kotlin
class Team {
    val playersNames = mutableSetOf<String>()
    var score = 0
    ...

    fun containsPlayer(playerName : String) : Boolean {
        return playerNames.contains(playerName)
    }

    fun getScore() : Int {
        return score
    }
}

fun getTeamScoreForPlayer (teams : List<Team>, playerName : String ) : Int? {
    for (team in teams) {
        if (team.containsPlayer(playerName))
            return team.getScore()
    }
    return null
}
```

코드는 이제 훨씬 더 이해하기 쉽다

- 이제 변수, 함수 및 클래스로 별도로 설명할 필요가 없이 자명하다
- 코드를 따로 떼어내 보면 더 의미가 있다. Team클래스를 확인하지 않아도 분명하게 알 수 있다.

또한, 이 코드는 주석문을 사용한 경우보다 덜 지저분하고 개발자가 주석문까지 관리할 필요 없이 코드에만 집중할 수 있다

## 5.2 주석문의 적절한 사용

코드 내에서 주석문이나 문서화는 다음과 같은 다양한 목적을 수행할 수 있다

- 코드가 `무엇`을 하는지 설명
- 코드가 `왜` 그 일을 하는지 설명
- 사용지침 등 기타 정보 제공

### 5.2.1 중복된 주석문은 유해할 수 있다.

코드 자체로 설명이 되는 주석문은 쓸모가 없다

```kotlin
fun geenerateId(firstName: String, lastName: String) : String {
    // "{이름}.{성}"의 형태로 ID를 생성한다
    return "$firstName . $lastName"
}
```

이런 불필요한 주석문은 다음과 같은 이유로 인해 쓸모없는 것 이상으로 더 나쁠 수 있다

- 개발자는 주석문을 유지보수해야한다. 코드를 변경하면 주석문 역시 수정해야 한다
- 코드를 지저분하게 만든다. 모든 코드 줄이 이와 같은 관련 주석을 가지고 있다면 100줄의 코드를 읽으려면 100줄의 코드와 
  
    100줄의 주석문을 읽어야한다

### 5.2.2 주석문으로 가독성 높은 코드를 대체할 수 없다.

```kotlin
fun generateId(data: Array<String>) : String {

    // data[0]는 유저의 이름이고 data[1]은 성이다
    // "{이름}.{성}"의 형태로 ID를 생성한다
    return "${data[0]} . ${data[1]}"
}
```

위 코드는 주석문이 있고 이해하기가 어려운 코드이다. 코드 자체가 가독성이 높지 않기 때문에 주석문이 필요하지만 더 나은 접근법은

가독성이 좋은 코드를 작성하는 것이다. 이럴 때에는 헬퍼 함수를 사용하면 가독성 높은 코드를 작성할 수 있다

```kotlin
fun generateId(data: Array<String>) :String {
    return "${firstName(data)} . ${lastName(data)} "
}

fun firstName(data :Array<String>) : String {
    return data[0]
}

fun lastName(data: Array<String>) : String {
    return data[1]
}
```


### 5.2.3 주석문은 코드의 이유를 설명하는 데 유용하다

특정 코드가 존재하는 이유나 어떤 일을 수행하는 목적은 다른 개발자가 알 수 없는 배경상황이나 지식과 관련있을 수 있다.

이러한 배경상황이나 지식이 코드를 이해하거나 안전하게 수정하기 위해 중요한 경우 주석문은 매우 유용하다

- 재품 또는 비즈니스 의사 결정
- 이상하고 명확하지 않은 버그에 대한 해결책
- 의존하는 코드의 예상을 벗어나는 동작에 대처


코드가 존재하는 이유를 설명하는 주석문의 예시는 다음과 같다

```kotlin

class User {
    private val username: Int
    private val firstName : String
    private val lastName : String
    private val signupVersion : Version
    ...

    fun getUserId() : String {
        if (signupVersion.isOlderThan("2.0")) {
            // v2.0 이전에 등록한 레거시 유저는 이름으로 ID가 부여된다
            // 자세한 내용은 #4218이슈를 보라
            return "${firstName.toLowerCase} . ${lastName.toLowerCase}"
        }

        // v2.0 이후로 등록한 새 유저는 username으로 ID가 부여된다
        return username
    }
}

```


### 5.2.4 주석문은 유용한 상위 수준의 요약 정보를 제공할 수 있다.

코드가 무슨 일을 하는지 설명하는 주석문과 문서는 마치 책을 읽을 떄 줄거리와 같다

코드 기능에 대한 상위 수준에서의 개략적인 문서는 다음과 같은 경우에 유용하다

- 클래스가 수행하는 작업 및 다른 개발자가 알고 있어야 할 중요한 세부 사항을 개괄적으로 설명하는 문서
- 함수에 대한 입력 매개변수 또는 기능을 설명하는 문서
- 함수의 반환값이 무엇을 나타내는지 설명하는 문서

예시는 다음과 같다

```kotlin
/**
* 스트리밍 서비스의 유저에 대한 자세한 사항을 갖는다
* 이 클래스는 데이터베이스에 직접 연결하지 않는다. 대신 메모리에 저장된 값으로
* 생성된다. 따라서 이 클래스가 생성된 이후에 데이터베이스에서 이뤄진 변경 사항을
* 반영하지 않을 수 있다
*/
class User {
    ...
}
```

## 5.3 코드 줄 수를 고정하지 말라

### 5.3.1 간결하지만 이해하기 어려운 코드는 피하라

코드의 줄 수가 얼마나 적으면 가독성이 떨어지는지 보자. 

```kotlin
fun isIdValid(id: UInt) : Boolean {
    return countSetBits(id and 0x7FFF) % 2 == ((id and 0x8000) shr 15)
}
```

이 코드는 간결하지만 다음과 같은 많은 가정과 복잡성을 가지고 있다

- ID의 하위 15비트에 값이 포함되어 있다
- ID의 최상위 비트는 패리티 비트다
- 15비트로 표현된 갑이 짝수인 경우 패리티 비트는 0이다
- 15비트로 표현된 값이 홀수인 경우 패리티 비트는 1이다
- 0x7FF는 하위 15비트를 위한 비트 마스크다
- 0x8000은 최상위 비트에 대한 비트마스크다.

위 코드는 간결하지만 거의 이해할 수 없다. 여러 개발자는 이 코드가 무엇을 하는지 이해하려고 많은 시간을 낭비할 가능성이 크다.

또한 이 코드가 전제하고 있는 명확하지 않고 문서화되지 않은 많은 가정으로 인해 코드 수정에 상당히 취약하고, 수정된 코드가 제대로 동작하지 않기 쉽다

### 5.3.2 해결책 : 더 많은 줄이 필요하더라도 가독성 높은 코드를 작성하라

```kotlin
fun isValid(id : UInt) : Boolean {
    return extractEncodedParity(id) ==
        calculateParity(getIdValue(id))
}

private const val PARITY_BIT_INDEX = 15
private const val PARITY_VIT_MASK = (1 shl PARITY_BIT_INDEX)
private const val VALUE_BIT_MASK = nor PARITY_BIT_MASK

fun getIdValue(id : UInt) : UInt {
    return id and VALUE_BIT_MASK
}

fun extractEncodedParity(id : UInt) : UInt {
    return (id and PARITY_BIT_MASK) shr PARITY_BIT_INDEX
}

// 패리티 비트는 1인 비트의 수가 짝수이면 0이고
// 홀수이면 1이다
fun calculateParity(value : UInt) : UInt {
    return countSetBits(value) % 2
}
```

## 5.4 일관된 코딩 스타일을 고수하라,

### 5.4.1 일관적이지 않은 코딩 스타일은 혼동을 일으킨다

일관성 없는 명명스타일의 예시는 다음과 같다

```java
class GroupChat {
    ...

    end() {
        connectionManager.terminateAll(); 
    }
}

class connectionManager {
    static terminateAll() {
        ...
    }
}
```

이 코드는 작동하지 않는다. connectionManager는 인스턴스 변수가 아니라 클래스 이름이고 terminateAll()은 그 클래스의

정적함수인 상황이였던 것이다. 표준 명명 규칙을 따라 ConnectionManager라는 이름으로 작성됐어야 한다.

## 5.5 깊이 중첩된 코드를 피하라

### 5.5.1 깊이 중첩된 코드는 읽기 어려울 수 있다

다음은 차량 소유자의 주소를 찾아주는 함수의 예시다

```kotlin
fun getOwnersAddress (vehicle : Vehicle) : Address? {

    if (vehicle.hasBeenScraped()) {
        return SCRAPYARD_ADDESS
    } else {
        val mostRecentPurchase = vehicle.getMostRecentPurchase()
        if (mostRecentPurchase == null ){
            return SHOWROOM_ADDRESS
        } else {
            val buyer = mostRecentPurchase.getBuyer()
            if (buyer != null) {
                return buyer.getAddress()
            }
        }
    }
    return null
}
```

인간의 눈은 각 코드 라인의 중첩 수준이 정확히 어느 정도인지 추적하는 데 능숙하지 않다. 중첩이 깊어지면 가독성이 떨어지기 때문에

중첩을 최소화하도록 코드를 구성하는 것이 바람직하다

### 5.5.2 해결책 : 중첩을 최소화하기 위한 구조 변경

앞선 코드를 눈이 따라가기 쉽게하고 논리는 더 여유있게 표현한 코드는 다음과 같다

```kotlin
fun getOwnerAddress(vehicle : Vehicle) : Address ? {
    if (vehicle.hasBeenScraped())
        return SCRAPYARD_ADDRESS

    val mostRecentPurchase = vehicle.getMostRecentPurchase()

    if (mostRecentPurchase == null)
        return SHOWROOM_ADDRESS
    
    val buyer = mostRecentPurchase.getBuyer()

    if (buyer != null)
        return buyer.getAddress()

    return null
}
```

### 5.5.3 중첩은 너무 많은 일을 한 결과물이다

다음은 차량 소유자의 주소를 찾고, 그 주소를 이용해 편지를 보내는 두 가지 논리를 수행한다.

이때문에 앞에서 살펴본 해결책을 적용하기 간단하지 않은데 함수에서 일찍 반환되어 돌아오게 되면 편지 발송하는 일을 수행하지못하기 때문이다

```kotlin
fun sendOwnerALetter(vehicle: Vehicle, letter: Letter) : SentConfirmation? {
    var ownerAddress = null

    if (vehicle.hasBeenScraped()) {
        ownerAddress = SCRAPYARD_ADDRESS
    } else {
        val mostRecentPurchase = vehicle.getMostRecentPurchase()

        if (mostRecehtPurchase == null)
            ownerAddress = SHOWROOM_ADDRESS
        else {
            val buyer = mostRecentPurchase.getBuyer()

            if (buyer != null)
                ownerAddress = buyer.getAddress()
        }
    }
    if (owenrAddress == null)
        return null
    
    return sendLetter(ownersAddress, letter)
}
```

이 코드의 문제점은 함수가 너무 많은 일을 한다는 것이다. 주소를 찾기 위한 자세한 로직과 편지를 보내는 로직이 하나의 함수에 다 포함되어 있다

### 5.5.4 해결책: 더 작은 함수로 분리

차량 소유자의 주소를 찾는 일을 다른 함수를 통해 수행하면 코드를 개선할 수 있다.

```kotlin
fun sendOwnerALetter(vehicle: Vehicle, letter: Letter) : SentConfirmation? {
    val ownerAddress = getOwnersAddress(vehicle)
    if (ownerAddress != null)
        return sendLetter(ownerAddress, letter)
    return null
}

fun getOwerAddress(vehicle: Vehicle) : Address ? {
    if (vehicle.hasBeenScraped()) 
        return SCRAPYARD_ADDRESS
    
    val mostRecentPurchase = vehicle.getMostRecehtPurchase()

    if (mostRecentPurchase == null) 
        return SHOWROOM_ADDRESS
    
    val buyer = mostRecentPurchase.getBuyer()

    if (buyer == null)
        return null
    
    return buyer.getAddress()
}
```

## 5.6 함수 호출도 가독성이 있어야 한다

### 5.6.1 매개변수는 이해하기 어려울 수 있다.

다음은 메시지를 보내는 함수를 호출하는 코드다. 함수의 인수가 무엇을 나타내는지 분명하지 않다

1이나 true는 무엇을 의미하는지 알기 어렵다

```kotlin
sendMessage("hello", 1, true)
```

1과 true가 무엇을 의미하는지 알려면 함수 정의를 살펴봐야 한다. 함수 정의를 확인하면, 1은 우선순위를 나타내고 true는

메시지의 전송이 실패한 경우 다시 전송을 한다는 의미라는 것을 알 수 있다

```kotlin
fun sendMessage(message: String, priority: Int, allowRetry: Boolean) {
    ...
}

```


함수 호출 시 각 인수의 값이 무엇을 의미하는지 알려면 함수 정의를 확인해봐야 한다. 함수 정의가 완전히 다른 파일에 있거나, 수백 줄 떨어져 있다면

이것은 힘든 작업일 수 있다. 주어진 코드가 무엇을 하는지 알아내기 위해 다른 파일이나 많은 줄을 확인해야 한다면, 그 코드는 가독성이 떨어진다

### 5.6.2 해결책: 명명된 매개변수 사용

명명된 인수를 사용한다면, 함수 정의를 확인하지 않고도 sendMessage() 함수에 대한 호출은 쉽게 이해할 수 있다

```kotlin
sendMessage(message: "hello", priority: 1, allowRetry: true)
```

이 방법은 모든 언어가 지원하는 것은 아니므로, 이 방법은 이 기능을 지원하는 프로그래밍 언어에만 해당된다.

### 5.6.3 해겶책 : 서술적 유형 사용

```java
class MessagePriority {
    ...
    MessagePriority(int priority) {...}
    ...
}

enum RetryPolicy {
    ALLOW_RETRY,
    DISALLOW_RETRY
}

void sendMessage(
    String message,
    MessagePriority priority,
    RetryPolicy retryPolicy
) {
    ...
}


sendMessage("hello", new MessagePriority(1), RetryPolicy.ALLOW_RETRY);
```

이 방법은 함수에 대한 호출이 함수 정의를 알지 못해도 이해하기 쉽다

## 5.7 설명되지 않은 값을 사용하지 말라


하드 코드로 작성된 값이 필요한 경우가 많이 있는데, 몇 가지 일반적인 예는 다음과 같다

- 한 수량을 다른 수량으로 변환할 때 사용하는 계수
- 작업이 실패할 경우 재시도의 최대 횟수와 같이 조정 가능한 파라미터 값
- 어떤 값이 채워질 수 있는 템플릿을 나타내는 문자열

하드 코드로 작성된 모든 값에는 두 가지 중요한 정보가 있다

- 값이 무엇인지: 컴퓨터가 코드를 실행할 때 이 값을 알아야 한다
- 값이 무엇을 의미하는지: 개발자가 코드를 이해하려면 값의 의미를 알아야 한다. 이 정보가 없으면 코드를 이해할 수 없다

### 5.7.1 설명되지 않은 값은 혼란스러울 수 있다.

```kotlin
class Vehicle {
    ...

    fun getMassUsTon(): Double{
        ...
    }

    fun getSpeedMph() : Double {
        ...
    }

    // 차량의 현재 운동에너지를 줄 단위로 반환한다
    fun getKineticEnergyJ() : Double {
        return 0.5 *
            getMassUsTon() * 907.1847 * // 미국톤/킬로그램 변환 계수에 대한 설명이 없다
            pow(getSpeedMph() * 0.44704, 2) // 시간당 마일/초당 미터 변환 계수에 대한 설명이 없다
    }
}
```

코드에 설명되지 않은 값이 있으면 혼란을 초래하고 이로 인해 버그가 발생할 수 있다. `그 값이 무엇을 의미하는지`를 다른 개발자들에게

명확하게 해주는 것이 중요하다.

### 5.7.2 해결책 : 잘 명명된 상수를 사용하라

값을 설명하기 위해 할 수 있는 한 가지 간단한 방법은 상수를 정의하고 상수 이름을 통해 값을 설명하는 것이다.

```kotlin
class Vehicle {
    private val KILOGRAMS_PER_US_TON = 907.1847
    private val METERS_PER_SECOND_PER_MPH = 0.44704

    // 차량의 현재 운동에너지를 줄 단위로 반환한다
    fun getKineticEnergyJ() : Double {
        return 0.5 *
            getMassUsTon() * KILOGRAM_PER_Us_TON *
            pow(getSpeedMph() * METERS_PER_SECOND_PER_MPH, 2) 
    }
}
```

### 5.7.3 해결책: 잘 명명된 함수를 사용하라

잘 명명된 상수를 사용하는 것의 대안으로 잘 명명된 함수를 사용할 수 있다. 코드의 가독성을 높이기 위해 함수를 사용할 수 있는 방법은 두가지있다

- 상수를 반환하는 공급자 함수
- 변환을 수행하는 헬퍼 함수

`공급자 함수`

이것은 개념적으로 상수를 사용하는 것과 거의 동일하며, 단지 야각ㄴ 다른 방식으로 이루어진다

```kotlin
class Vehicle {
        // 차량의 현재 운동에너지를 줄 단위로 반환한다
    fun getKineticEnergyJ() : Double {
        return 0.5 *
            getMassUsTon() * kilogramsPerUsTon() * 
            pow(getSpeedMph() * metersPerSecondPerMph(), 2) 
    }

    companion object {
        fun kilogramsPerUsTon() : Dobule {
            return 907.1847
        }

        fun metersPerSecondPerMph() : Double {
            return 0.44704
        }
    }
}
```

`헬퍼 함수`

```kotlin
class Vehicle {
        // 차량의 현재 운동에너지를 줄 단위로 반환한다
    fun getKineticEnergyJ() : Double {
        return 0.5 *
            kilogramsPerUsTon(getMassUsTon())  * 
            pow(metersPerSecondPerMph(getSpeedMph()), 2) 
    }

    companion object {
        fun kilogramsPerUsTon(usTons: Double) : Dobule {
            return usTons * 907.1847
        }

        fun metersPerSecondPerMph(mph : Double) : Double {
            return mph * 0.44704
        }
    }
}
```

## 5.8 익명 함수를 적절하게 사용하라

`익명 함수`는 이름이 없는 함수이며, 일반적으로 코드 내의 필요한 지점에서 인라인으로 정의된다.

```java
class List<T> {
    ...
    List<T> filter(Function<T, Boolean> retainIf) {
        ...
    }
}

List<Feedback> getUsefullFeedback(List<Feedback> allFeedback) {
    return allFeedback
        .filter(feedback-> !feedback.getComment().isEmpty());
}
```

간단하고 자명한 것에 익명 함수를 사용하면 코드의 가독성을 높여주지만, 복잡하거나 자명하지 않은 것 혹은 재사용해야 하는 것에

사용하면 문제가 될 수 있다

### 5.8.1 익명 함수는 간단한 로직에 좋다

```java
List<Feedback> getUsefullFeedback(List<Feedback> allFeedback) {
    return allFeedback
        .filter(feedback-> !feedback.getComment().isEmpty());
}
```

이 코드는 하나의 문장이면 충분하고, 해결하려는 문제는 간단하기 때문에 이 코드는 매우 이해하기 쉽고 단순 명료하다

이런 경우에는 익명 함수로 표현하려는 논리는 단순하고 자명하기 때문에 익명 함수를 사용하는 것이 괜찮다.

### 5.8.2 익명 함수는 가독성이 떨어질 수 있다.

익명 함수는 정의상 이름이 없기 때문에 그 익명 함수 코드를 읽는 사람에게 어떠한 것도 제공하지 않는다. 그것이 얼마나 간단한 것이든

간에 익명 함수의 내용이 자명하지 않다면 코드의 가독성은 떨어지기 마련이다


```java
List<UInt16> getValidIds(List<UInt16> ids) {
    return ids
        .filter(id -> id != 0)
        .fileter(id -> countSetBits(id * 0x7FFF)%2 ==
            ((id & 0x8000) >> 15)); //패리티 비트를 확인하는 익명핫무
}
```

이처럼 간단하지만 이해하기 어려운 코드가 있다. 이런 논리는 대부분의 개발자들이 이해하기 어렵기 때문에 설명이 필요하다.

그리고 익명함수는 그안에 있는 코드 이상의 어떤 설명도 제공하지 않으므로 이런 경우는 익명함수를 쓰는 것이 반드시 좋은방법이 아닐 수 있다


### 5.8.3 해결책: 대신 명명 함수를 사용하라

```java
List<UInt16> getValidIds(List<UInt16> ids) {
    return ids
        .filter(id -> id != 0)
        .filter(isParityBitCorrect); 
}

private Boolean isParityBitCorrect(UInt16 id) {
    ...
}
```

## 5.9 프로그래밍 언어의 새로운 기능을 적절하게 상요하라

### 5.9.1 새 기능은 코드를 개선할 수 있다

전통적인 자바코드를 우선 보자

```java
List<String> getNonEmptyStrings(List<String> strings) {
    List<String> nonEmptyStrings = new ArrayList<>();
    for (String str : strings) {
        if (!str.isEmpty()) {
            nonEmptyStrings.add(str);
        }
    }
    return nonEmptyStrings;
}
```

이 코드를 스트림을 사용해 작성하면 더 간결하고 이해하기 쉬운 코드가 된다

```java
List<String> getNonEmptyStrings(List<String> strings) {
    return strings
        .stream()
        .filter(str -> !str.isEmpty())
        .collect(toList());
}
```

코드가 읽기 쉽고 간결해졌기 때문에 스트림을 잘 사용한 것처럼 보인다. 무언가를 직접 만드는 대신 언어에서 제공하는 기능을

사용하면 코드가 최적화되어 효율적이고 버그가 없을 가능성이 커진다

### 5.9.2 불분명한 기능은 혼동을 일으킬 수 있다.

일반적으로 코드 품질을 개선한다면 언어가 제공하는 기능을 사용하는 것이 바람직하다. 그러나 개선 사항이 적거나 다른 개발자가

그 기능에 익숙하지 않다면 차라리 사용하지 않는 것이 좋을 때도 있다

# 요약

- 코드의 가독성이 떨어져서 이해하기 어려울 때 다음과 같은 문제가 발생할 수 있다
  - 다른 개발자가 코드를 이해하느라 시간을 허비함
  - 버그를 유발하는 오해
  - 잘 작동하던 코드가 다른 개발자가 수정한 후에 작동하지 않음
- 코드의 가독성을 높이다 보면 때로는 코드가 더 장황하게 되고 더 많은 줄을 작성해야 할 수도 있다. 이것은 종종 가치있는 절충이다
- 코드의 가독성을 높이려면 다른 개발자의 입장을 공감하고, 그들이 코드를 읽을 때 어떻게 혼란스러워할지를 상상해보는 것이 필요하다
- 실제 시나리오는 다양하며 보통 그 상황에 해당하는 어려움이 있다. 가독성이 좋은 코드를 작성하려면 언제나 상식을 적용하고 판단력을 사용해야한다

