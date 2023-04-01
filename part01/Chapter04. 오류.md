# 4. 오류

## 4.1 복구 가능성

소프트웨어에 대해 생각할 때, 특정 오류가 발생한 경우 복구할 수 있는 현실적인 방법이 있는지 생각해야 하는 경우가 많다.

복구할 수 있는 오류와 복구할 수 없는 오류가 무슨 의미인지 보고 이러한 구별이 상황에 따라 어떻게 달라지는지 살펴보자.

### 4.1.1 복구 가능한 오류

많은 소프트웨어 오류는 치명적이지 않으며, 오류가 발생하더라도 사용자는 알아채지 못하도록 적절하게 처리한다면 작동을 계속할 수 있는 합리적 방법이 있다.

한가지 예로 사용자가 잘못된 입력을 하는 경우가 있다

잘못된 사용자 입력 외에도 복구 가능한 오류의 예는 다음과 같다

- `네트워크 오류` : 자신의 코드가 의존하는 서비스에 연결할 수 없는 경우 몇초동안 기다렸다가 다시 시도하거나, 그 코드가 사용자의 장치에서

    실행되는 경우라면 사용자에게 네트워크 연결을 확인하도록 요청하는 것이 최상이다

- `중요하지 않은 작업 오류` : 예를 들어 서비스 사용에 대한 어떤 통제를 기록하는 부분에서 오류가 발생한다면 실행을 계속해도 무방할것이다.

### 4.1.2 복구할 수 없는 오류

오류가 발생하고 시스템이 오류를 복구할 수 있는 합리적인 방법이 없을 때가 있다. 이러한 현상은 프로그래밍 오류 때문에 발생할 때가 많은데

이 경우는 개발자가 코드의 어느 부분에서 '뭔가를 망친 것이다' 예는 당므과 같다

- 코드와 함께 추가되어야 하는 리소스가 없다
- 다음 예와 같이 어떤 코드가 다른 코드를 잘못 사용한다
  - 잘못된 입력 인수로 호출
  - 일부 필요한 상태를 사전에 초기화하지 않음

오류를 복구할 수 있는 방법이 없다면 유일하게 코드가 할 수 있는 합리적인 방법은 피해를 최소화하고 개발자가 문제를 발견하고 해결할 가능성을 최대화하는것이다.

### 4.1.3 호출하는 쪽에서만 오류 복구 가능 여부를 알 때가 많다.

대부분의 오류는 한 코드가 다른 코드를 호출할 때 발생한다. 따라서 오류 상황을 처리할 때는 다른 코드가 자신이 작성한 코드를 호출하는 것과 관련해 

다음과 같은 사항을 신중하게 고려해야 한다

- 오류로부터 복구하기를 호출하는 쪽에서 원하는가?
- 만약 그렇다면 오류를 처리할 필요가 있다는 것을 호출하는 쪽에서 어떻게 알 수 있을까?

간결한 추상화 계층을 만들고자 한다면 일반적으로 코드의 잠재적 호출자에 대한 가정을 가능한 한 하지 않는 것이 좋다.

함수를 작성하거나 수정하는 시점에 오류로부터 복구할 수 있는지 혹은 복구해야 하는지 여부를 항상 알 수 있는 것은 아니기 때문이다.

```kotlin
class PhoneNumber {
    ...
    companion object {
        if (!isValidPhoneNumber(number)) {
            ... 에러를 처리하기 위한 코드
        }
    }
}
```

이 함수가 어떻게 사용되고 있으며 어디에서 호출되는지 알지 못한다면 프로글매이 이 오류를 극복할 수 있는지에 대한 질문에 답을 할 수 없다.

보다 일반적으로 다음 중 하나라도 해당하는 경우, 함수에 제공된 값으로 인해 발생하는 오류는 호출하는 쪽에서 복구하고자 하는 것으로 간주해야 한다

- 함수가 어디서 호출될지 그리고 호출 시 제공되는 값이 어디서 올지 정확한 지식이 없다
- 코드가 미래에 재사용될 가능성이 아주 희박하다. 재사용이 된다면 코드가 어디에서 호출되고 값이 어디서 오는지에 대한

    가정이 의미가 없어질 수 있음을 뜻한다

호출하는 쪽에서 오류로부터 복구하기를 원할 것이라고 판단하는 것은 좋은 일이지만, 오류가 발생할 수 있다는 것조차 인식하지 못한다면

그것을 제대로 처리하지 못할것이다.

### 4.1.4 호출하는 쪽에서 복구하고자 하는 오류에 대해 인지하도록 하라.

(위와 같은 내용 생략)

## 4.2 견고성 vs 실패

오류가 발생할 때, 다음 중 하나를 선택해야 한다

- 실패, 더 높은 코드 계층이 오류를 처리하게 하거나 전체 프로그램의 작동을 멈추게 한다
- 오류를 처리하고 계속 진행한다

오류가 있더라도 처리하고 계속 진행하면 더 견고한 코드라고 볼 수 있지만, 오류가 감지되지 않고 이상한 일이 발생하기 시작한다는 의미도 될 수 있다

### 4.2.1 신속하게 실패하라.

`신속하게 실패하기(failing fast)`는 가능한 한 문제의 실제 발생 지점으로부터 가까운 곳에서 오류를 나타내는 것이다.

복구할 수 있는 오류의 경우 호출하는 쪽에서 오류로부터 훌륭하고 안전하게 복구할 수 있는 기회를 최대한으로 제공하고, 복구할 수 없는

오류의 경우 개발자가 문제를 신속하게 파악하고 해결할 수 있는 기회를 최대한 제공한다. 두 경우 모두 소프트웨어가 의도치 않게 잠재적으로 

위험한 상태가 되는것을 방지한다

이에 대한 일반적인 예는 잘못된 인수로 함수를 호출하는 경우다. 실패의 신속한 표시는 그 함수가 잘못된 입력과 함께 호출되는 즉시 오류를

발생시키는 것을 의미하며, 이는 잘못된 값임에도 불구하고 계속 실행해 코드의 다른 곳에서 문제를 일으키는 상황과는 반대된다.

실패가 신속하게 이뤄지면, 오류는 실제 위치 근처에서 나타나며 스택 트레이스는 종종 해당 오류의 위치에 대한 정확한 코드 위치를 제공한다

이렇게 발생하자마자 바로 실패나 오류를 보여주지 않으면 문제가 발생할 때 디버그하기 어려울 뿐만 아니라, 코드가 제대로 작동하지 않거나

잠재적으로 문제를 일으킬 수 있다. 예를 들어 손상된 데이터가 데이터베이스에 저장되고 있음에도 불구하고 버그는 몇 달 후에야 발견될 수

있으며, 이 버그로 인해 이미 많은 중요한 데이터가 영구히 파괴됐을 수도 있다.

### 4.2.2 요란하게 실패하라

프로그램이 복구할 수 없는 오류가 발생하면 프로그래밍 오류나 개발자의 실수로 인한 버그일 가능성이 크다. 

누구나 소프트웨어에서 이런 버그가 일어나지 않기를 원하고, 일어나면 곧바로 버그를 고치기를 바라지만, 버그가 있다는 사실을 알지 못하면

고칠 수 있는 방법이 없다.

`요란한 실패`는 오류가 발생하는데도 불구하고 아무도 모르는 상황을 막고자 하는 것이다. 이를 위한 가장 명백한 방법은 `예외` 를 발생해

프로그램이 중단되게 하는 것이다. 다른 방법은 오류 메시지를 기록하는 것인데 개발자가 얼마나 부지런하게 로그를 확인하는지, 혹은 

로그에 방해되는 다른 메시지가 얼마나 있는지에 따라 오류 메시지가 무시될 수 있다.

코드가 실패할 때 신속하고, 요란하게 오류를 나타내면 개발 도중이나 테스트하는 동안에 버그가 발견될 가능성이 크다.

그렇지 않더라도 배포된 후에 오류 보고를 보기 시작할 것이고 보고 내용으로부터 버그가 발생한 위치를 정확히 알 수 있는 이점이 있다

### 4.2.3 복구 가능성의 범위

복구할 수 있는 또는 복구할 수 없는 범위는 달라질 수 있다. 예를 들어 클라이언트의 요청을 처리하는 서버 내에서 실행되는 코드를 작성하는 경우

개별 요청을 처리하는 도중 버그가 있는 코드 경로를 실행해 오류가 일어날 수 있다. 그 요청을 처리하는 범위 내에서 복구할 수 있는

합리적인 방법은 없지만, 시스템 전체가 작동을 멈추는 것은 막을 수 있다. 이 경우에 오류는 해당 요청 범위 내에서 복구할 수 없지만,

서버 전체적으로는 복구할 수 있다.

가장 좋은 방법은 프로그래밍 오류가 발견되면 개발자가 이를 알아차릴 수 있도록 프로그래밍 오류를 기록하고 모니터링하는 것이다.

상세 오류 정보를 기록하여 개발자가 발생한 일을 디버그할 수 있게 해주고, 오류 발생률이 너무 높아지면 개발자에게 알림 메시지를 보내는 것이 해당한다.

### 4.2.4 오류를 숨기지 않음

#### `기본값 반환`

오류가 발생하고 함수가 원하는 값을 반환할 수 없는 경우 기본값을 반환하는 것이 간단하고 쉬운 해결책처럼 보일 때가 있다.

이것과 비교해보면 적절한 에러 전달과 처리를 위해 코드를 추가하는 것은 많은 노력이 드는 것처럼 보일 수 있다. 기본값의 문제점은

오류가 발생했다는 사실을 숨긴다는 것인데, 이는 코드를 호출하는 쪽에서 모든 것이 정상인 것처럼 계속 진행한다는 것을 의미한다.

```kotlin
class AccountManager(
    private val accountStore : AccountStore
) {

    fun getAccountBalanceUsd(customerId: Int) : Double {
        val result = accountStore.lookup(customerId)
        if (!result.success()) {
            return 0.0 // 오류가 발생하면 기본값 0 이 반환
        }
        return result.account.balanceUsd
    }
}
```

코드에 기본값을 두는 것이 유용한 경우가 있을 수 있지만, 오류를 처리할 떄는 대부분의 경우 적합하지 않다.

잘못된 데이터로 시스템이 제대로 작동하지 못하게 만들고 오류가 나중에 이상한 방식으로 나타날 수 있기 때문에 신속한 실패와 요란한 실패의 원리를 위반한다.

#### `널 객체 패턴`

널 객체는 개념적으로 기본값과 유사하지만 이것을 더 확장해서 더 복잡한 객체를 다룬다. 널 객체는 실제 반환값처럼 보이지만

모든 멤버 함수는 아무것도 하지 않거나 의미 없는 기본값을 반환한다

```kotlin
class InvoiceManager(
    private val invoiceStore: InvoiceStore
) {

    fun getUnpaidInvoices(customerId: Int) : List<Invoice> {
        val result = invoiceStore.query(customerId)
        if (!result.success())
            return []
        return result.invoices.filter{ invoice -> !invoice.isPaid()}
    }
}
```

널 객체 패턴은 후에 자세히 다룰 예정이며 이것은 양날의 검이다. 널 객체 패턴을 사용하는 것이 꽤 유용한 경우가 있지만,

오류처리에 사용하는 것은 바람직하지 않다

#### `아무것도 하지 않음`

코드가 무언가를 반환하지 않고 단지 어떤 작업을 수행하는 경우, 문제가 발생할 때 가능한 한 가지 옵션은 오류가 발생했다는 신호를

보내지 않는 것이다. 호출하는 쪽에서는 코드에서 작업이 의도대로 완료되었다고 가정하기 때문에 일반적으로 이렇게 하는 것은 바람직하지 않다

이것은 코드가 하는 일에 대해 개발자가 가지고 있는 정신 모델과 코드가 실제로 수행하는 것 사이의 불일치를 일으킬 가능성이 매우 높다

이로 인해 소프트웨어에서 예상과 벗어나는 동작과 버그가 발생할 수 있다.

```kotlin
class MutableInvoice {
    fun addItem(item: InvoiceItem) {
        if (item.price.currency != this.currency) {
            return // 통화가 일치하지 않으면 함수는 되돌아감
        }
        this.items.add(item)
    }
}
```

바로 위의 코드는 오류신호를 보내지 않는 예이다. 또 다른 시나리오는 다른 코드가 전달하는 오류를 적극적으로 억제하는 코드다.



```kotlin
// 예외 억제
class InvoiceSender(
    private val emailService: EmailService
) {
    fun emailInvoice(emailAddress: String, invoice: Invoice) {
        try{
            emailService.sendPlainText(emailAddress,InvoiceFormat.plainText(invoice))
        }catch(e: EmailException){
            // EmailException이 발생하지만 무시된다
        }
    }
}
```

오류가 발생할 때 이 오류를 기록하면 약간 개선되긴 하지만 바로 위 코드와 비슷하게 바람직하지 않은 코드다

개발자가 로그를 확인하면 적어도 이러한 오류를 알아차릴 수 있어서 이 코드는 약간 개선된 코드라고 할 수 있다.

그러나 여전히 호출하는 쪽에 오류를 숨기고 있어서 이메일이 실제로 발송되지 않는 경우에도 발송되었다고 가정할 수 있다

```kotlin
// 예외 탐지 및 오류 기록
class InvoiceSender(
    private val emailService: EmailService
) {
    fun emailInvoice(emailAddress: String, invoice: Invoice) {
        try{
            emailService.sendPlainText(emailAddress,InvoiceFormat.plainText(invoice))
        }catch(e: EmailException){
            logger.logError(e) // EmailException이 기록된다
        }
    }
}
```

## 4.3 오류 전달 방법

오류가 발생하면 일반적으로 더 높은 계층으로 오류를 알려야 한다. 오류로부터 복구할 수 없는 경우 이는 일반적으로 프로그램의 훨씬 더 높은 계층에서 실행을

중지하고, 오류를 기록하거나 전체 프로그램의 실행을 종료하는 것을 의미한다.

오류로부터 복구가 잠재적으로 가능한 경우, 일반적으로 즉시 호출하는 쪽에 오류를 알려 정상적으로 처리할 수 있도록 해야한다.

오류를 알리는 방법은 크게 두 가지 종류로 나뉜다

- `명시적 방법` : 코드를 직접 호출한 쪽에서 오류가 발생할 수 있음을 인지할 수밖에 없도록 한다. 그것을 처리하든, 이전 호출자에게 전달하든,

    아니면 그냥 무시하든 간에 어떻게 처리할지는 호출하는 쪽에 달려있다. 하지만 무엇을 하든 그것은 적극적인 선택의 결과다.

    오류가 발생할 가능성이 코드 계약의 명확한 부분에 나타나 있기 때문에 오류를 모르고 넘어갈 수 있는 방법은 거의 없다
- `암시적 방법` : 코드를 호출하는 쪽에 오류를 알리지만, 호출하는 쪽에서 그 오류를 신경 쓰지 않아도 된다.

    오류가 발생할 수 있음을 알기 위해서는 문서나 코드를 읽는 등의 적극적인 노력이 필요하다.

    문서에 오류가 언급되어 있다면 코드 계약의 숨겨진 세부 조항이다. 가끔 오류가 여기서조차 언급되지 않을 때도 있는데,

    이 경우엔 오류가 계약 내용에 전혀 없는 것이 된다


||||
|:--|:--:|:-:|
||`명시적 오류 전달 기법`| `암시적 오류 전달 기법`|
|코드 계약에서의 위치|명확한 부분|세부 조항 혹은 아예 없음|
|호출하는 쪽에서 오류 발생 <br/> 가능성에 대해 아는가?|그렇다|알 수도 있고 모를 수도 있다|
|기법의 예|검사 예외 <br/> 널 반환 유형(널 안전성의 경우) <br/> 옵셔널 반환 유형 <br/> 리절트 반환 유형 <br/> 아웃컴 반환 유형(반환값 확인이 필수인 경우) <br/> 스위프트 오류|비검사 예외 <br/> 매직값 반환(피해야 한다) <br/> 프로미스 또는 퓨처 <br/> 어서션 <br/> 체크(구현에 따라 달라짐) <br/> 패닉|

### 4.3.1 요약 : 예외

예외는 일반적으로 충분한 기능을 가진 클래스로 구현된다. 보통 프로그래밍 언어는 즉시 사용할 수 있는 예외에 대한 클래스를 제공하지만

개발자들은 자신의 요구 사항에 따른 맞춤형 예외 처리를 위해 오류에 대한 정보를 자유롭게 정의하고 캡슐화할 수 있다

자바는 `검사 예외 (checked exception)`과 `비검사 예외(unchecked exception)`의 개념을 모두 가지고 있다.

### 4.3.2 명시적 방법: 검사 예외

컴파일러는 검사 예외에 대해 호출하는 쪽에서 예외를 인지하도록 강제적으로 조치하는데, 호출하는 쪽에서는 예외 처리를 위한 코드를 작성하거나

자신의 함수 시그니처에 해당 예외 발생을 선언해야 한다. 따라서 검사 예외를 사용하는 것은 오류를 전달하기 위한 명시적인 방법이다


#### `검사 예외를 사용한 오류 전달`


```java
class NegativeNumberException extends Exception { // 검사 예외의 구체적 유형임을 나타내는 클래스
    private final Double erroneousNumber; // 오류를 유발한 숫자를 캡슐화해서 추가 정보를 제공

    NegativeNumberException(DOuble erroneousNumber) {
        this.erroneousNumber = erroneousNumber;
    }

    Double getErroneousNumber() {
        return erroneousNumber;
    }
}

Double getSquareRoot(Double value) throws NegativeNumberException { // 함수는 검사 예외를 발생시킬 수 있음을 선언해야함

    if (value < 0.0 )
        throw new NegativeNumberException(value); //오류가 있을 시 검사 예외를 발생시킴
    return Math.sqrt(value);
}
```

#### `검사 예외 처리`

getSqaureRoot() 함수를 호출하는 코드는 NegativeNumberException 예외를 처리하거나 함수 시그니처에 이 예외를 발생시킬 수 있음을 표시해야한다

```java
void displaySquareRoot() {
    Double value = ui.getInputNumber();
    try{
        ui.setOutput("square root is : " + getSquareRoot(value));
    }catch(NegativeNumberException e){
        ui.setError("can't root of negative number: " + e.getErroneousNumber());
    }
}
```

displaySquareRoot() 함수가 NegativeNumberException을 포착하지 않는 경우에는 이 함수의 시그니처에 이 예외가 발생할 수 있음을

선언해야 한다. 이경우에는 이 예외가 발생하는 경우의 처리를 자신이 하지 않고 displaySquareRoot() 함수를 호출하는 코드에게 맡기는 것이 된다

```java
void displaySquareRoot() throws NegativeNumberException {
    Double value = ui.getInputNumber();
    ui.setOutput("square root is: " + displaySquareRoot(value));
}
```


displaySquareRoot() 함수가 NegfativeNumberException 예외를 포착하지도 않고, 자신의 함수 시그니처에 선언하지도 않으면

코드는 컴파일되지 않는다. 호출하는 쪽에서는 어떤 형태로든 해당 오류를 강제적인 방식으로 인지할 수밖에 없기 때문에 

검사 예외는 오류를 전달하는 명시적 방법이 된다


### 4.3.3 암시적 방법: 비검사 예외

비검사 예외를 사용하면 다른 개발자들은 코드가 이 예외를 발생시킬 수 있다는 사실을 전혀 모를 수 있다. 이 경우에는 함수에서 어떤 예외를 발생

시키는지 문서화하는 것이 바람직하지만 개발자가 문서화하는 것을 잊어버릴 때가 있다. 설사 문서화를 하더라도 이것은 코드 계약의 세부 조항이다

비검사 예외는 오류가 발생할 수 있다는 것을 호출하는 쪽에서 인지하리라는 보장이 없기 때문에 오류를 암시적으로 알리는 방법이다

#### `비검사 예외를 사용한 오류 전달`

대부분의 언어에서 예외는 비검사 예외지만 자바에서는 RuntimeException 클래스를 확장하는 예외 클래스는 비검사 예외이다.

```java
class NegativeNumberException extends RuntimeException { 
    private final Double erroneousNumber; 

    NegativeNumberException(DOuble erroneousNumber) {
        this.erroneousNumber = erroneousNumber;
    }

    Double getErroneousNumber() {
        return erroneousNumber;
    }
}

Double getSquareRoot(Double value) { 

    if (value < 0.0 )
        throw new NegativeNumberException(value); //오류가 있을 시 비검사 예외를 발생시킴
    return Math.sqrt(value);
}
```

비검사 예외를 발생시키는 함수를 호출하는 쪽에서는 예외가 발생할 수 있다는 사실을 전혀 몰라도 된다.

이로 인해 비검사 예외는 오류를 암시적으로 전달하는 방법이다

### 4.3.4 명시적 방법: 널값이 가능한 반환 유형

함수에서 널값을 반환하는 것은 특정한 값을 계산하거나 얻는 것이 붉다능함을 나타내기 위한 효과적이고 간단한 방법이다.

#### `널 값을 이용한 오류 전달`

널 값을 반환할 때의 한 가지 문제점은 오류가 발생한 이유에 대한 정보를 제공하지 않기 때문에 널값이 의미하는 바를

설명하기 위해 주석문이나 문서를 추가해야 한다

```kotlin
// 제공되는 값이 음수이면 널을 반환한다

fun getSquareRoot(value : Double) : Double? {
    if (value < 0.0 )
        return null
    return sqrt(value)
}
```

#### `널값 처리`

```kotlin
fun displaySquareRoot() {
    val squareRoot: DOuble? = getSquareRoot(ui.inputNumber)
    if (squareRoot == null) 
        ui.error = "can't get square root of a negative number"
    else
        ui.output = "Square root is: $squareRoot"
}
```

호출하는 쪽에서 널값 여부를 강제로 확인해야 한다는 것은 엄밀히 말해서 사살이 아니다. 반환되는 값을 널이 아닌 값으로 

타입 변환할 수 있으며, 이것은 여전히 적극적인 결정이다. 타입 변환 시 값이 널인 경우를 처리할 수 밖에 없다.

### 4.3.5 명시적 방법: 리절트 반환 유형

널값이나 옵셔널 타입을 반환할 때의 문제 중 하나는 오류 정보를 전달할 수 없다는 것이다. 호출자에게 값을 얻을 수 없음을 알릴 뿐만 아니라

값을 얻을 수 없는 이유까지 알려주면 유용하다. 이러한 경우에는 리절트 유형을 사용하는 것이 적잘할 수 있다

```java
@AllArgsConstructor
class Result<V,E> {
    private final Optional<V> value;
    private final Optional<E> error;

    static Result<V,E> ofValue(V value) {
        return new Result(Optiional.of(value), Optional.empty());
    }

    static Result<V,E> ofError(E error) {
        return new Result(Optional.empty(), Optional.of(error));
    }

    Boolean hasError() {
        return error.isPresent();
    }

    V getValue(){
        return value.get();
    }

    E getError() {
        return error.get();
    }
}
```

#### `리절트 유형을 이용한 전달`

```java
class NegativeNumberException extends Error { 
    private final Double erroneousNumber; 

    NegativeNumberException(DOuble erroneousNumber) {
        this.erroneousNumber = erroneousNumber;
    }

    Double getErroneousNumber() {
        return erroneousNumber;
    }
}

Result<Double, NegativeNumberException> getSquareRoot(Double value) { 

    if (value < 0.0 )
        return Result.ofError(new NegativeNumberError(value));
    return Result.ofValue(Math.sqrt(value));
}
```

### 4.3.6 명시적 방법: 아웃컴 반환 유형

어떤 함수들은 값을 반환하기보다는 단지 무언가를 수행하고 값은 반환하지는 않는다. 어떤 일을 하는 동안 오류가 발생할 수 있고 그것을

호출한 쪽에 알리고자 한다면, 함수가 수행한 동작의 결과를 나타내는 값을 반환하도록 함수를 수정하는 것이 한 가지 방법이 될 수 있다.

아웃컴 반환 유형을 반환할 때 호출하는 쪽에서 반환값을 강제적으로 확인해야 한다면 이것은 오류를 알리는 명백한 방법이다

#### `아웃컴을 이용한 오류 전달`

```java
Boolean sendMessage(Channel channel, String message) {
    if (channel.isOpen()) {
        channel.send(message);
        return true;
    }
    return false;
}

void sayHello(Channel channel) {
    if (sendMessage(channel, "hello"))
        ui.setOutput("Hello sent");
    else
        ui.setError("Unable to send hello");
}
```

아웃컴 반환 유형에 대한 문제점 중 하나는 호출하는 쪽에서 반환값을 무시하거나 함수가 값을 반환한다는 사실조차 인식 못할 수 있다는 점이다.

이로 인해 아웃컴 반환 유형은 오류를 알리는 명시적 방법으로서 한계가 있다. 예를들어 다음곽 같다

```java
void sayHello(Channel channel) {
    sendMessage(channel, "hello"); // 아웃컴이 무시된다.
    ui.setOutput("Hello sent");
}
```

이럴 때에는 `@CheckReturnValue` 애노테이션을 사용하면 된다

```java
@CheckReturnValue
Boolean sendMessage(Channel channel, String message) {
    ...
}
```
이는 개발자가 알아차릴 수 있도록 컴파일러가 경고 메시지를 보여준다.

### 4.3.7 암시적 방법: 프로미스 또는 퓨처

비동기적으로 실행하는 코드를 작성할 때 `프로미스(promise)`나 `퓨처(future)`를 반환하는 함수를 작성하는 것이 일반적이다.

#### `프로미스를 이용한 전달`

```java
class NegativeNumberError extends Error {
    ...
}

// 이는 수도코드임
Future<Double> getSqaureRoot(Double value) async {
    await Timer.wait(Duration.ofSeconds(1)); // 실행 전 1초 기다림

    if (value < 0.0) 
        throw new NegativeNumberError(value); //함수 내에서 오류를 발생하고 프로미스는 거부
    return Math.sqrt(value); //값을 반환하면 프로미스는 이행됨
}

void displaySquareRoot() {
    getSquareRoot(ui.getrInputNumber())
        .then(squareRoot -> // then 콜백은 프로미스가 이행되면 호출
            ui.setOutput("Square root is : " + squareRoot));
        .catch(error ->  // catch 콜백은 프로미스가 거부되면 호출
            ui.setError("An error occurred: " + error.toString()));
}
```

#### `왜 프로미스는 암묵적인 오류 전달 기법인가`

오류가 발생하고 프로미스가 거부될 수 있음을 알려면 프로미스를 생성하는 함수의 세부 조항이나 구현 세부 사항을 확인해야 한다.

이 내용을 모르면 프로미스의 사용자는 잠재적인 오류 상태를 쉽게 알 수 없으며, then() 함수를 통해서만 콜백을 제공할 것이다.

catch() 함수를 통해 콜백이 제공되지 않으면, 오류는 일부 상위 수준의 오류 처리 코드에 의해 포착되거나 완전히 눈에 띄지 않을 수 있다

프로미스와 퓨처는 비동기 함수로부터 값을 반환하는 훌륭한 방법이다. 그러나 호출하는 쪽에서는 잠재적인 오류 시나리오를 

완전히 알지 못하기 때문에 프로미스나 퓨처를 사용하는 것은 오류를 알리는 암시적 방법이 된다

#### `프로미스를 명시적으로 만들기`

프로미스나 퓨처를 반환할 때 명시적 오류 전달 기법으로 사용하려면, 리절트 유형의 프로미스를 반환하는 것이 한 가지 방법일 수 있다.

이것은 유용한 기술이지만, 코드가 복잡해지기 때문에 모든 사람이 이렇게 사용하지는 않을 것이다

```java
FUture<Result<Double, NegativeNumberError>> getSqaureRoot (Double value) async {
    await Timer.wait(Duration.ofSeconds(1));

    if (value < 0.0) 
        return Result.ofError(new NegativeNumberError(value));
    return Result.ofValue(Math.sqrt(value));
}
```

### 4.3.8 암시적 방법: 매직값 반환

`매직값`은 함수의 정상적인 반환 유형에 적합하지만 특별한 의미를 부여하는 값이다. 매직값이 반환될 수 있다는 것을 알려면

문서나 코드를 읽어야 한다. 따라서 이것은 암시적 오류 전달 기법이다

```java
// 음숫값이 입력으로 제공되면 -1을 반환한다
Double getSquareRoot(Double value) {
    if (value < 0.0) 
        return -1.0;
    return Math.sqrt(value);
}
```

매직값은 오류를 알리는 좋은 방법이 아니다. 이에 대한 자세한 내용은 6장에서 다룰 예정

## 4.4 복구할 수 없는 오류의 전달

현실적으로 복구할 가능성이 없는 오류가 발생하면 신속하게 실패하고, 요란하게 실패하는 것이 최상의 방법이다.

이를 달성하기 위한 몇 가지 일반적인 방법은 다음과 같다

- 비검사 예외를 발생
- 프로그램이 패닉이 되도록
- 체크나 어서션의 사용

암시적인 기술을 사용하면 오류 시나리오를 확인하거나 처리하기 위한 코드를 호출 체인의 상위에 있는 모든 호출자가 다 작성할

필요는 없다. 오류를 복구할 방법이 없을 때는 이것이 합리적이다. 왜냐하면 이 경우 자신을 호출한 쪽에 오류를 전달하는 것 외에는

할 수 있는 방법이 없기 때문이다.

## 4.5 호출하는 쪽에서 복구하기를 원할 수도 있는 오류의 전달

### 4.5.1 비검사 예외를 사용해야 한다는 주장

잠재적으로 복구할 수 있는 오류에도 불구하고 비검사 예외를 사용하는 것이 더 나은 이유에 대한 일반적 주장은 다음과 같다

#### `코드 구조 개선`

대부분의 오류 처리가 코드의 상위 계층에서 이루어질 수 있기 때문에 비검사 예외를 발생시키면 코드 구조를 개선할 수 있다는 주장이다.

오류가 높은 계층까지 거슬러 올라오면서 전달되고, 그 사이에 있는 코드는 오류 처리를 할 필요가 없다

중간 계층은 원한다면 예외 중 일부를 처리할 수 있지만, 그렇지 않으면 오류가 최상위 오류 처리 계층으로 전달된다.

이 접근법의 핵심 장점은 오류를 처리하는 로직이 코드 전체에 퍼지지 않고 별도로 몇 개의 계층에만 있다는 점이다

#### `개발자들이 무엇을 할 것인지에 대해서 실용적이어야 함`

개발자들이 너무 많은 명시적 오류 전달 (반환 유형 및 검사 예외)를 접하면 결국 잘못된 일을 한다고 주장한다.

예를 들어 예외를 포착하고도 무시한다거나, 널이 가능한 유형을 확인도 하지 않고 널이 불가능한 유형으로 변환을 하는 것이다.

다음과 같은 코드가 있다 하자. 지금은 오류가 없는 초기 코드이다.

```java
class TemperatureLogger {
    private final Thermometer thermometer;
    private final DataLogger dataLogger;

    void logCurrentTemperature() {
        dataLogger.logDataPoint(
            Instant.now(),
            thermometer.getTemperature());
    }
}

class DataLogger {
    private final InMemoryDataStore dataStore;

    void logDataPoint(Instant time, Double value) {
        dataStore.store(new DataPoint(time.toMillis(), value));
    }
}

```

추후 이 코드는 메모리에 값을 저장하는 대신 디스크에 저장하는 식으로 바꾸리 수도 있다.

따라서 IOException을 발생시킬 수 있는데 함수 시그니처에 이 에러를 추가하면 호출하는 모든 코드를 수정하고 

그보다 더 위 계층의 코드까지도 수정해야 할 수도 있다. 이렇게 할 경우 작업의 양이 너무 많기 때문에 개발자는

여기에서 오류를 숨기고 아래 처럼 작성할지도 모른다

```java
class DataLogger{
    private final DiskDataStore dataStore;

    void logDataPoint(Instant time, Double value) {
        try{
            dataStore.store(new DataPoint(time.toMillis(), value))
        }catch (IOException e){} // IOException 오류가 호출자에게 숨겨진다
    }
}
```

이렇게 오류를 숨기는 것은 결코 좋은 생각이 아니다. 이 함수를 호출할 때 데이터가 저장되지 않을 수도 있지만, 호출하는 쪽에서는

이 사실을 알지 못한다. 명시적인 오류 전달 방식을 사용하면 코드의 계층을 올라가면서 오류를 반복적으로 전달하고 이를 처리하는

일련의 작업이 필요한데, 이런 번거로운 작업 대신 개발자는 편의를 도모하고 잘못된 작업을 하고 싶은 마음이 들 수 있다.

### 4.5.2 명시적 기법을 사용해야 한다는 주장

잠재적으로 복구할 수 있는 오류에 대해 명시적인 오류 전달 기술을 사용하는 것이 더 나은 이유에 대한 주장은 다음과 같다

#### `매끄러운 오류 처리`

비검사 예외를 사용한다면 모든 오류를 매끄럽게 처리할 수 있는 단일 계층을 갖기가 어렵다. 호출하는 쪽에 잠재적 오류를

강제적으로 인식하도록 하면 이러한 오류를 좀 더 매끄럽게 처리할 가능성이 커진다

#### `실수로 오류를 무시할 수 없다`

어떤 호출자의 경우에는 실제로 오류를 처리해야 하는 경우가 있을 수 있다. 비검사 예외가 사용되면 적극적인 의사 결정이 들어갈 여지는 줄어들고

대신 기본적으로 잘못된일이 일어나기 쉽다. 이는 개발자가 특정 오류가 발생할 수 있다는 사실을 완전히 알지 못하기 때문이다.

#### `개발자들이 무엇을 할 것인지에 대해서 실용적이어야 함`

개발자들이 오류 처릴르 너무 많이 해야 되서 잘못되게 처리한다는 주장은 비검사 예외의 사용을 반박하는 것에도 적용할 수 있다.

비검사 예외가 코드베이스 전반에 걸쳐 제대로 문서화된다는 보장이 없고, 개인적인 경험에 비추어 보면 문서화되지 않는 경우가 많다.


```java
Boolean isDataFileValid(byte[] fileContents) {
    try{
        DataFile.parse(fileContents);
        return true;
    }catch (InvalidEncodingException || ParseException || UnrecognizedDataKeyException e) {
        return false;
    }
}
```

이렇게 하나하나 예외를 할때에 모든 종류의 예외를 묶을 수도 있다
```java
Boolean isDataFileValid(byte[] fileContents) {
    try{
        DataFile.parse(fileContents);
        return true;
    }catch (Exception e) {
        return false;
    }
}
```

이 코드에서 같이 모든 예외를 다 아우르는 예외를 처리하는 것은 바람직하지 않다. 이렇게 하면 프로그램이 현실적으로

복구할 수 없는 많은 오류를 포함하여 거의 모든 유형의 오류가 숨겨진다. 어쩌면 심각한 프로그래밍 오류가 숨겨질 수도 있다.

DataFile.parse() 함수의 버그일 수도, ClassNotFoundException과 같은 심각한 소프트웨어 구성 오류일 수도 있다.

이러한 코드는 상당히 노골적이므로 코드 검토 중에 발견되고 수정되어야 한다.

## 4.6 컴파일러 경고를 무시하지 말라

```java
@AllArgsConstructor
class UserInfo {
    private final String realName;
    private final String displayName;

    String getRealName(){
        return realName;
    }

    String getDisplayName(){
        return realName; // 사용자의 실제 이름이 잘못 반환된다
    }
}
```

이 코드는 컴파일 되지만 컴파일러는 `UserInfo.displayName`은 할당된 값을 읽는 경우가 전혀 없기 때문에 없어도 된다 라는 경고를 보여준다.

이 경고를 무시하면 이 버그에 대해 모를 수 있다

대부분의 컴파일러는 경고를 오류로 여기고 코드가 컴파일되지 않도록 설정할 수 있다. 이렇게 하는 것이 다소 과장되고 엄격해 보일 수 있지만,

개발자들이 경고를 알아차리고 그에 따라 행동하도록 강제하기 때문에 실제로는 매우 유용하다

경고가 실제로 걱정할 것이 아닌 경우에는 일반적으로 특정 경고만 억제할 수 있는 방법이 있다. 

```java
@AllArgsConstructor
class UserInfo {
    private final String realName;


    // 실제 이름을 사용하지 않도록 마이그레이션 작업중이라 displayName이 지금은
    // 사용되지 않는다. 이것은 곧 사용할 것을 대비해 만들어 놓은 변수다
    // 마이그레이션에 대한 상세한 내용은 이슈 #31231 을 참고하라
    @Suppress("unsed") // 이필드에 대한 경고 메시지가 있더라도 컴파일러는 출력하지 않는다
    private final String displayName;

        String getRealName(){
        return realName;
    }

    String getDisplayName(){
        return realName; 
    }
}
```

# 요약

- 오류에는 크게 두 가지 종류가 있다
  - 시스템이 복구할 수 있는 오류
  - 시스템이 복구할 수 없는 오류
- 해당 코드에 의해 생성된 오류로부터 복구할 수 있는지 여부를 해당 코드를 호출하는 쪽에서만 알 수 있는 경우가 많다
- 에러가 발생하면 신속하게 실패하는 것이 좋고, 에러를 복구할 수 없는 경우에는 요란하게 실패하는 것이 바람직하다
- 오류 전달 기법은 두 가지 범주로 나눌 수 있다
  - 명시적 방법: 코드 계약의 명확한 부분, 호출하는 쪽에서는 오류가 발생할 수 있음을 인지한다
  - 암시적 방법: 코드 계약의 세부 조항을 통해 오류에 대한 설명이 제공되거나 전혀 설명이 없을 수도 있다

    오류가 발생할 수 있다는 것을 호출하는 쪽에서 반드시 인지하는 것은 아니다

- 복구할 수 없는 오류에 대해서는 암시적 오류 전달 기법을 사용해야 한다
- 잠재적으로 복구할 수 있는 오류에 대해서는
  - 명시적 혹은 암시적 기법 중 어느 것을 사용할지에 대해서는 개발자들 사이에서도 일치되는 의견이 없다
  - 이 책 저자의 의견으로는 명시적인 기법이 사용되어야 한다
- 컴파일러 경고는 종종 코드에 문제가 있을 때 이에 대해 표시해준다. 이 경고에 주의를 기울이는 것이 바람직하다