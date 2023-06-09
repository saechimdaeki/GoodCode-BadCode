# 단위 테스트의 원칙

## 10.1 단위 테스트의 원칙

단위 테스트와 관련하여 기억해야 할 몇가지 중요한 개념과 용어는 다음과 같다

- 테스트 중인 코드 : `실제 코드`라고도하고 테스트의 대상이 되는 코드를 의미한다
- 테스트 코드 : 단위 테스트를 구성하는 코드를 가르킨다.
- 테스트 케이스 : 테스트 코드의 각 파일에는 일반적으로 여러 테스트 케이스가 있고, 각 테스트 케이스는 특정 동작이나 시나리오를 테스트한다
  - 준비(arrange) : 테스트할 특정 동작을 호출하려면 먼저 몇가지 설정을 수행해야 하는 경우가 많다. 예를 들어 일부 테스트 값을 정의하거나, 의존성을 설정하거나, 

    테스트 대상이 되는 클래스의 인스턴스를 올바르게 설정하고 생성하는 것 등이 포함된다.
  - 실행(act) : 테스트 중인 동작을 실제로 호출하는 코드를 나타낸다. 이것은 일반적으로 테스트 대상이 되는 코드에 존재하는 함수를 호출하는 것을 수반
  - 단언(assert) : 테스트 중인 동작이 실행하고 나면 실제로 올바른 일이 발생했는지 확인해야 한다. 여기에는 일반적으로 반환값이 예상한 값과

    같거나 일부 결과 상태가 예상과 같은지 확인하는 작업이 포함된다.
- 테스트 러너 : 이름에서 알 수 있듯 테스트 러너는 실제로 테스트를 실행하는 도구다. 테스트 코드 파일이 주어지면 각 테스트 케이스를

    실행하고 통과 혹은 실패한 케이스에 대한 자세한 결과를 출력한다

## 10.2 좋은 단위 테스트는 어떻게 작성할 수 있는가?

단우 테스트에서 문제가 발생하면 유지 관리가 매우 어렵고, 버그가 테스트 코드에서 발견되지 못하고 배포한 뒤에 발생할 수도 있다.

그러므로 어떻게 해야 좋은 단위 테스트가 되는지 생각해보는 것이 중요하다. 이를 위해 좋은 단위테스트가 가져야할 5가지 주요 기능을 정의한다

- `훼손의 정확한 감지`: 코드가 훼손되면 테스트가 실패한다. 그리고 테스트는 코드가 실제로 훼솓된 경우에만 실패해야 한다
- `세부 구현 사항에 독립적` : 세부 구현 사항을 변경하더라도 테스트 코드는 변경하지 않는 것이 이상적이다
- `잘 설명되는 실패` : 코드가 잘못되면 테스트는 실패의 원인과 문제점을 명확하게 설명해야 한다
- `이해할 수 있는 테스트 코드` : 다른 개발자들이 테스트 코드가 정확히 무엇을 테스트하기 위한 것이고 테스트가 어떻게 수행되는지 이해할 수 있어야 한다
- `쉽고 빠르게 실행` : 개발자는 일상 작업 중에 단위 테스트를 자주 실행한다. 단위 테스트가 느리거나 실행이 어려우면 개발 시간이 낭비된다

### 10.2.1 훼손의 정확한 감지

단위 테스트의 가장 명확하고 주된 목표는 코드가 훼손되지 않았는지 확인하는 것이다. 이것은 매우 중요한 두가지 역할을 수행한다

- 코드에 대한 초기 신뢰를 준다 : 아무리 신중하게 코드를 작성해도 실수는 있기 마련이다. 새로운 코드나 코드 변경 사항과 함께

    철저한 테스트 코드를 작성하면 코드가 코드베이스로 병합되기 전에 이러한 실수를 발견하고 수정할 수 있다

- 미래의 훼손을 막아준다 : 어느 시점에 다른 개발자가 코드를 변경하는 과정에서 실수로 코드를 훼손할 가능성이 크다. 이것에 대한 유일한

    효과적인 방어 방법은 코드가 컴파일을 중지하거나 테스트가 실패하는 것이다. 코드가 컴파일을 멈추도록 하는 것은 불가능 하므로

    모든 올바른 동작을 테스트를 통해 확인하는 것이 절대적으로 중요하다. 코드 변경으로 인해 잘 돌아가던 기능이 작동하지 않는 

    것을 `회긔(regression)`라고 한다. 이러한 회귀를 탐지할 목적으로 테스트를 실행하는 것을 `회귀 테스트`라 한다

테스트 대상 코드가 정상임에도 불구하고 때로는 통과하고 때로는 실패하는 테스트를 `플래키(flakey)`라고 한다.

이것은 무작위성, 타이밍 기반 레이스 조건, 외부 시스템에 의존하는 등의 테스트의 비결정적 동작에 기인한다. 단점은 원인을 찾느라 시간낭비한다는 것이다

코드에서 어떤 부분이 훼손될 때 그리고 오직 훼손된 경우에만 테스트가 실패하도록 하는 것이 중요하다

### 10.2.2 세부 구현 사항에 독립적

개발자가 코드베이스에 가할 수 있는 변경은 두가지 종류가 있다

- 기능적 변화: 이것은 코드가 외부로 보이는 동작을 수정한다. 새로운 기능 추가, 버그 수정, 에러 처리 등이 있다
- 리팩토링 : 이것은 큰 함수를 작은 함수로 분할하거나 재사용하기 쉽도록 일부 유틸리티 코드를 다른 파일로 옮기는 등의

    코드의 구조적 변화를 의미한다. 리팩터링이 올바르게 수행되더라도 이론적으로는 외부에서 보이는 동작이 변경되면 안된다

코드는 자주 리팩토링된다. 성숙한 코드베이스에서는 리팩토링의 양이 작성된 새 코드의 양을 초과할 수 있으므로 리팩토링 시 

코드가 훼손되지 않도록 하는 것이 가장 중요하다. 테스트가 구현 세부 정보에 의존하지 않으면 코드 리팩토링에 실수가 있었는지

확인해주는 테스트 결과를 신뢰할 수 잇따

### 10.2.3 잘 설명되는 실패

테스트의 주요 목적 중 하나는 미래의 훼손으로부터 코드를 보호하는 것이다. 일반적인 경우는 개발자가 변경한 코드로 인해 다른사람이

작성한 코드가 동작을 하지 않는 것이다. 테스트는 실패하기 시작하고, 테스트 결과는 개발자에게 그들이 무언가를 고장냈다는 것을 알린다

개발자는 테스트 실패에 대한 자세한 내용을 살펴보고 무엇이 문젠지 알아낸다. 개발자는 그들이 무심코 망가뜨린 코드를 잘 모를 수 있기

때문에 테스트 실패가 무엇이 잘못됐는지 알려주지 않는다면 그것을 알아내기 위해 많은 시간을 낭비해야 한다


`실패에 대한 자세한 내용을 보여주지 않는 테스트 실패의 예시`

```console
Test case testGetEvents failed :
Expected: [Event@ea4a92b, Event@3c5199da]
But was actually : [Event@3c5a99da, Event@ea4a92b]
```


`실패에 대한 자세한 내용을 잘 설명해주는 테스트 실패`

```console
Test case testGetEvents_inChronologicalOrder failed :
Conrtents match, but order differs
Expected :
    [ <Spaceflight, April 12, 1961>, <Moon Landing, July 20, 1969>]
But was actually : 
    [  <Moon Landing, July 20, 1969>, <Spaceflight, April 12, 1961>]
```

이렇게 하면 한번에 모든 것을 테스트하려고 하는 하나의 큰 테스트 케이스보다 각각의 특정 동작을 확인하기 위한 작은 테스트 케이스가

많이 만들어진다. 테스트가 실패할 때 실패한 케이스의 이름을 확인하면 어떤 동작이 작동하지 않는지 정확하게 알 수 있다

### 10.2.4 이해 가능한 테스트 코드

테스트 코드를 이해하기 쉽게 만들기 위해 노력해야 하는 이유는 일부 개발자들이 테스트를 코드에 대한 일종의 사용 설명서로 사용하기 때문이다.

특정 코드를 어떻게 사용하는지, 혹은 어떤 기능을 제공하는지 궁금하다면 단위 테스트를 통해 알아보는 것도 좋은 방법이다

테스트 코드가 이해하기 어렵다면 사용 설명서로 유용하게 사용될 수 없을 것이다

### 10.2.5 쉽고 빠른 실행

단위 테스트의 중요한 기능 중 하나는 잘못된 코드가 코드베이스에 병합되는 것을 방지하는 것이다. 

따라서 많은 코드베이스에서 관련 테스트를 통과해야만 병합이 가능한 병합 전 검사를 수행한다.

테스트가 느리면 테스트가 힘든 작업이 되고, 테스트가 힘들면 하고 싶지 않은 마음이 든다. 

테스트가 가능한 쉽고 빠르게 실행할 수 있으면 개발자는 더 효율적으로 작업할 수 있고, 테스트 역시 더 광범위하고 철저해진다

## 10.3 퍼블릭 API에 집중하되 중요한 동작은 무시하지 말라

## 10.4 테스트 더블

단위 테스트는 `비교적 격리된 방식`으로 코드 단위를 테스트하는 것을 목표로 한다고 했다. 하지만 코드는 다른 것들에 의존하는 경향이 있고,

코드의 모든 동작을 완벽하게 테스트하기 위해 종종 입력을 설정하고 부수 효과를 검증해야 한다. 하지만 테스트에서 의존성을

실제로 사용하는 것이 항상 가능하거나 바람직한 것만은 아니다

의존성을 실제로 사용하는 것에 대한 대안으로 `테스트 더블(test double)`이 있다. 테스트 더블은 의존성을 시뮬레이션하는 객체지만

테스트에 더 적합하게 사용할 수 있도록 만들어진다. 

### 10.4.1 테스트 더블을 사용하는 이유

- `테스트 단순화` : 일부 의존성은 테스트에 사용하기 까다롭고 힘들다. 의존성은 많은 설정이 필요하거나 하위 의존성을 설정해야 할

    수도 있다. 이러면 테스트는 복잡하고 구현 세부 사항과 밀접하게 결합될 수 있다. 의존성을 실제로 사용하는 대신 테스트

    더블을 사용하면 작업이 단순해진다.

- `테스트로부터 외부 세계 보호` : 일부 의존성은 실제로 부수 효과를 발생한다. 코드의 종속성 중 하나가 실제 서버에 요청을 전송하거나

    실제 데이터베이스에 값을 쓰게 되면, 사용자나 비즈니스에 중요한 프로세스에 나쁜 결과를 초래할 수 있다. 이러한 상황에서

    테스트 더블을 사용하면 외부 세계에 있는 시스템을 테스트의 동작으로부터 보호할 수 있다.

- `외부로부터 테스트 보호` : 외부 세계는 비결정적일 수 있다. 다른 시스템이 데이터베이스에 쓴 값을 의존성 코드가 읽는다면

    이 값은 시간이 지남에 따라 변경될 수 있다. 이 경우 테스트 결과를 신뢰하기 어려울 수 있다. 반면에 테스트 더블은

    항상 동일하게 결정된 방식으로 작동하도록 설정할 수 있다

`테스트 단순화` 

어떤 의존성을 설정하는 데 많은 노력이 필요할 수 있다. 의존성 자체에서 많은 매개변수를 지정해야 할 수도 있고, 하위 의존성을

많이 설정해야 할 수도 있다. 설정 외에도 하위 의존성에서 원하는 부수 효과가 발생했는지 검증해야 할 수도 있다. 

![image](../image/테스트%20단순화1.png)


반대로 테스트 더블을 사용하면 실제 의존성을 설정하거나 하위 종속성에서 무언가를 검증할 필요가 없다.

테스트 코드는 테스트 더블과만 상호작용하면 설정과 부수효과 검증을 할 수 있는데 둘 다 비교적

간단하게 할 수 있다

![image](../image/테스트%20단순화2.png)

테스트 단순화에 대한 또 다른 동기는 테스트를 더 빠르게 실행하는 것이다.

의존성 코드에서 계산 비용이 많이 드는 알고리즘을 호출하거나 시간이 오래 걸리는 설정을 많이 한다면 이에 해당한다

`테스트로부터 외부 세계 보호`

상대적으로 격리된 방식으로 코드를 테스트하고자 할때도 있지만, 어쩔 수 없이 격리된 상태로 테스트해야 하는 경우도 있다.

결제를 처리하는 시스템을 사용하고 고객의 은행계좌에서 돈을 인출하는 코드를 테스트한다고 가정하자

코드가 실제로 실행되면 이에 대한 부수 효과로 고객의 실제 계좌에서 돈이 실제로 인출될 것이다

테스트에 클래스의 인스턴스를 사용하면 테스트가 실행될 때마다 실제 계좌에서 돈이 실제로 인출된다. 이렇게 하면 사람들의 돈에 

영향을 미치거나 회사의 감사와 회계에 문제를 일으키기에 테스트는 절대 이렇게 수행하면 안된다

이제 실제 인스턴스 대신 테스트 더블을 사용하면 테스트로부터 외부 세계를 보호할 수 있다. 이렇게 하면 테스트는

실제 은행시스템으로부터 격리되고 테스트를 실행할 때 실제 은행계좌나 돈이 영향을 받지 않는다

`외부로부터 테스트 보호`


외부 세계로부터 테스트를 보호하기 위해서도 테스트 더블을 사용한다. 실제 의존성은 비결정적인 동작을 할 수 있다.

데이터에비스에서 정기적으로 변경되는 값을 읽거나 난수 생성기를 사용하여 ID와 같은 것을 생성하는 실제 의존성이 있을

수 있다. 이런 의존성을 테스트에 사용하다 보면 테스트 결과를 신뢰하기 어려울 때가 있다.

이럴 때에 실제 시스템과 분리해 이것은 테스트 더블을 통해 수행할 수 있다.

### 10.4.2 목

`목(mock)`은 클래스나 인터페이스를 시뮬레이션하는 데 멤버 함수에 대한 호출을 기록하는 것 외에는 어떠한

일도 수행하지 않는다. 함수가 호출될 때 인수에 제공되는 값을 기록한다. 테스트 대상 코드가 의존성을 통해

제공되는 함수를 호출하는지 검증하기 위해 목을 사용할 수 있다. 따라서 목은 테스트 대상 코드에서 부수 효과를

일으키는 의존성을 시뮬레이션 하는데 가장 유용하다

다음과 같은 코드가 있다고 하자.

```java
class PaymentManager {
    ..

    PaymentResult settleInvocie(
        BankAccount customerBankAccount,
        Invoice invoice) {
            customerBankAccount.debit(invoice.getBalance());
            return PaymentResult.paid(invocie.getId());
        }
}
```

여기에서 BankAccount는 인터페이스이고 구현하는 클래스는 BankAccountImpl이다. 

이는 실제 은행 시스템에 연결되기에 테스트에서 이 인스턴스를 사용할 수 없는 상황이다

목을 사용하는 테스트 케이스는 다음과 같다

```java
void testSettleInvocie_acoountDebited() {
    BankAccount mockAccount = createMock(BankAccount);
    MonetaryAmount invoiceBalance = 
        new MonetaryAmount(5.0, Currency.USD);
    Invoice invoice = new Invoice(invoiceBalance, "test-id");
    PaymentManager paymentManager = new PaymentManager();

    paymentManager.settleInvoice(mockAccount, invocie);

    verifyThat(mockAccount.debit)
        .wasCalledOnce()
        .withArguments(invoiceBalance);
}
```

이런식으로 목을 사용하면 실제 계좌에 가지 않고도 테스트 할 수 있다. 하지만 테스트로부터 외부 세계를 보호하는 데는

성공한 반면 테스트가 비현실적이고 중요한 버그를 잡지 못할 위험이 있다

### 10.4.3 스텁

`스텁(stub)`은 함수가 호출되면 미리 정해 놓은 값을 반환함으로써 함수를 시뮬레이션 한다.

이를 통해 테스트 대상 코드는 특정 멤버 함수를 호출하고 특정 값을 반환하도록 의존성을 시뮬레이션할 수 있다.

그러므로 스텁은 테스트 대상 코드가 의존하는 코드로부터 어떤 값을 받아야 하는 경우 그 의존성을 시뮬레이션하는데 유용하다

앞서 목 테스트는 목 객체를 만들어야 하지만 stub은 실제로 목 기능을 사용하는 것이 아닌 함수를 스텁한다

예시를 보자

```java
void testSettleInvoice_insufficientFundsCorrectResultReturend() {
    MonetaryAmount invoiceBalance = 
        new MonetartyAmount(10.0, Currency.USD)
    Invoice invocie = new Invoice(invoiceBalance, "test-id");
    BankAccount mockAccount = createMock(BankAccount); // 스텁만 필요하지만 목객체 생성
    when(mockAccount.getBalance())
        .thenReturn(new MonetaryAmount(9.99, Currency.USD));
    PaymentManager paymentManager = new PaymentManager();

    PaymentResult result = paymentManager.settleInvoice(mockAccount, invoice);

    assertThat(result.getStatus()).isEqualTo(INSUFFICIENT_FUNDS);
}
```

스텁을 사용하면 테스트를 외부로부터 보호하고 결과를 신뢰할 수 있따. 하지만 목과 스텁도 단점이 있다

### 10.4.4 목과 스텁은 문제가 될 수 있다

- 목이나 스텁이 실제 의존성과 다른 방식으로 동작하도록 설정되면 테스트는 실제적이지 않다
- 구현 세부 사항과 테스트가 밀접하게 결합하여 리팩토링이 어려워질 수 있다

`목과 스텁은 실제적이지 않은 테스트를 만들 수 있다`

클래스나 함수에 대해 목 객체를 만들거나 스텁할 때 테스트 코드를 작성하는 개발자는 목이나 스텁이 어떻게 동작할지

결정해야 한다. 클래스나 함수가 실제와 다르게 동작하도록 하는 것은 아주 위험하다. 이렇게 하면 테스트는

통과하고 모든것이 잘 작동한다고 착각하지만 코드가 실제로 실행되면 부정확하게 동적하거나 버그가 발생할 수 있다

마이너스 송장 잔액 테스트를 보자

```java
void testSettleInvoice_negativeInvoiceBalance() {
    BankAccount mockAccount = createMock(BankAccount);
    MonetaryAmount invoiceBalance = 
        new MonetaryAmount(-5.0, Currency.USD);
    Invoice invoice = new Invoice(invoiceBalance, "test-id");
    
    PaymentManager paymentManager = new PaymentManager();

    verifyThat(mockAccount.debit)
        .wasCalledOnce()
        .withArguments(invoiceBalance);
}
```

이 코드는 음숫값으로 테스트하는건 실제 세계에서 일어나리라고 기대하기 어렵다. 이것은 이 가정의 타당성이 실제로

테스트된 적이 없다는 것을 의미한다. 또한 코드가 실제 세계에서 작동하는 것과는 상관없이 테스트가 통과한다는 점에서

5달러로 설정하고 테스트한 코드를 동어 반복한 것에 지나지 않는다

```java
interface BankAccount {

    void debit(MonetaryAmount amount); // ArgumentException 0보다 적은 금액 호출하는 경우

    void credit(MonetaryAmount amount); // ArgumentException 0보다 적은 금액 
}
```

paymentManager.settleInvoice() 함수는 버그가 있다는 것이 명확하지만 테스트에서 목을 사용하기 때문에 

이 버그가 드러나지 않는다. 이것이 목의 주요 단점 중 하나다. 테스트 코드를 작성하는 개발자는 목이 어떻게 동작할지

결정해야 하는데, 실제 의존성이 어떻게 동작하는지 이해하지 못하면 목을 설정할 때 실수를 할 가능성이 크다.

`목과 스텁을 사용하면 테스트가 구현 세부 정보에 유착될 수 있다`

결국 앞서 말한 내용에서 버그가 발견되면 아래 코드에서와 같이 settleInvoice()함수에 if문을 도입해 해결할 것이다

```java
PaymentResult settleInvoice(...) {
    ..
    MonetaryAmount balance = invoice.getBalance();
    if (balance.isPositive()) {
        customerBankAccount.debit(balance);
    } else {
        customerBankAccount.debit(balance.absoluteAmount());
    }
}
```

개발자가 이 코드를 테스트하기 위해 목을 사용할 경우 다음과 같이 작성할 것이다

```java
void testSettleInvoice_positiveInvoiceBalance() {
    ...
    verifyThat(mockAccount.debit)
        .wasCalledOnce()
        .withArguments(invoiceBalance);
}

void testSettleInvoice_negativeInvoiceBalance() {
    ...
    verifyThat(mockAccount.credit)
        .wasCalledOnce()
        .withArguments(invoiceBalance.absoluteAmount());
}
```

이 테스트는 예상되는 함수가 호출되는지 테스트하지만 이 클래스를 사용할 때 실제로 관심을 갖는 동작을 직접 테스트하지는 않는다

이 if/else 를 리팩토링 하는 코드가 있었다고 가정하자

```java
interface BankAccount {
    /**
     * 지정된 금액을 계좌로 송금한다. 금액이 0보다 적으면
     * 계좌로부터 인출하는 효과를 갖는다
     */ 
    void transfer(MonetaryAmount amount);
}

PaymentResult settleInvoice(..) {
    ...
    MonetaryAmount balance = invoice.getBalance();
    customerBankAccount.transfer(balance.negate());
    ...
}
```

이 리팩토링은 구현 세부 사항만 변경했을 뿐 동작은 변경하지 않았다. 하지만 테스트에서 목을 사용하고 있었고,

이 함수들이 호출되지 않기 때문에 테스트는 실패한다. 리팩터링을 수행한 개발자는 테스트 통과를 위해

많은 테스트 케이스를 수정해야 하ㅣ므로 리팩터링이 의도치 않게 동작을 변경하지 않았다는 확신을 하기 어렵다

### 10.4.5 페이크

`페이크(fake)`는 클래스의 대체 구현체로 테스트에서 안전하게 사용할 수 있다. 페이크는 실제 의존성의 공개 API를

정확하게 시뮬레이션하지만 구현은 일반적으로 단순한데, 외부 시스템과 통신하는 대신 페이크 내의 멤버 변수에 상태를 

저장한다

페이크의 요점은 코드 계약이 실제 의존성과 동일하기 때문에 실제 클래스가 특정 입력을 받아들이지 않는다면 

페이크도 마찬가지라는 것이다. 따라서 실제 의존성에 대한 코드를 유지보수하는 팀이 일반적으로 페이크 코드도

유지보수해야 하는데, 실제 의존성에 대한 코드 계약이 변경되면 페이크의 코드 계약도 동일하게 변경되어야 하기 때문이다

BankAccount에 대한 페이크 코들르 구현하면 다음과 같을 것이다

```Java
class FakeBankAccount implements BankAccount {
    private MonetaryAmount balance;

    override void debit(MonetaryAmount amount) {
        if (amount.isNegative) {
            // 예외 처리
        }
        balance = balnce.subtract(amount);
    }

    override void credit(MonetaryAmount amount) {
        if (amount.isNegative()) {
            // 예외 처리
        }
        bvalance = balance.add(amount);
    }

    override void transfer(MonetaryAmount amount) {
        balance.add(amount);
    }

    override MonetaryAmount getBalance() {
        return roundDownToNearest10(balance);
    }
}
```

목이나 스텁 대신 페이크를 사용하면 이전 하위 절에서 논의한 문제점을 피할 수 있는데 살펴보자

`페이크로 인해 보다 실질적인 테스트가 이뤄질 수 있다`

이전 테스트 케이스는 목을 사용해 BankAccount.debit() 함수가 마이너스 금액으로 호출되었는지 확인했다

실제로 debit() 함수는 마이너스 금액을 허용하지 않기 때문에 코드에 버그가 있음에도 불구하고 테스트는 토오가했다

목대신 페이크를 테스트 케이스에 사용했다면 이 버그가 발견되었을 것이다.

```java
void testSettleInvoice_negativeInvoiceBalance() {
    FakeBankAccount fakeAccount = new FakeBankAccount(
        new MonetaryAmount(100.0, Currency.USD));
    MonetaryAmount invoiceBalance = 
        new MonetaryAmount(-5.0, Currency.USD);
    Invoice invoice = new Invoice(invoiceBalance, "test-id");
    PaymentManager paymentManager = new PaymentManager();

    paymentManager.settleInvoice(fakeAccunt, invoice);

    assertThat(fakeAccount.getActualBValance())
        .isEqualTo(new MonetaryAmount(105.0, Currency.USD));
    
}
```

테스트를 하는 이유는 버그가 있으면 테스트가 실패하고 코드 작성자는 이를 통해 자신의 코드에 버그가 있다는 것을

인지하는 것이다. 따라서 위 예제 코드의 테스트 케이스는 정확히 이일을 해주기 때문에 유용하다


`페이크를 사용하면 구현 세부 정보로부터 테스트를 분리할 수 있다`

목이나 스텁 대신 페이크를 사용할 때의 또 다른 이점은 테스트가 구현 세부 사항에 밀접하게 결합하는 정도가

덜하다는 것이다. 앞서 개발자가 테스트 대상이 되는 코드를 리팩터링한 경우 목을 사용한 테스트가 실패하는 것을 봤다

이와는 대조적으로 테스트가 페이크를 사용하는 경우 구현 세부 사항 대신 최종 계정 잔애깅 정확한지 확인한다


## 10.5 테스트 철학으로부터 신중하게 선택하라

테스트 철학과 방법론의 예는 다음과 같다

- `테스트 주도 개발(TDD)` : TDD는 실제 코드를 작성하기 전에 테스트케이스를 먼저 작성하는 것을 지지한다

    실제 코드는 테스트만 통과하도록 최소한으로 작성하고 이후에 구조를 개선하고 중복을 없애기 위해 리팩터링을 한다

    TDD지지자들은 일반적으로 테스트 케이스를 격리하고 한 테스트 케이스는 하나의 동작만 테스트하도록

    집중하며 구현 세부사항을 테스트하지 않는 등의 모범사례를 지지한다
- `행동 주도 개발(BDD)` : 이 철학의 핵심은 사용자, 고객,비즈니스 과넞ㅁ에서 소프트웨어가 보여야할 행동을

    식별하는 데 집중하는 것이다. 이런 동작은 소프트웨어가 개발될 수 있는 형식으로 포착되고 기록된다

    테스트는 소프트웨어 자체의 속성보다는 이러한 원하는 동작을 반영한다

- `수용 테스트 주도 개발(ATDD)` : ATDD는 고객의 관점에서 소프트웨어가 보여줘야 하는 동작을 식별하고

    소프트웨어가 보여줘야 하는 동작을 식별하고 소프트웨어가 전체적으로 필요에 따라 작동하는지 검증하기 위해

    자동화된 `수락테스트(acceptance test)`를 만드는 것을 수반한다


# 10장 요약
- 코드베이스에 제출된 거의 모든 `실제 코드`는 그에 해당하는 단위 테스트가 동반되어야 한다
- `실제 코드`가 보여주는 모든 동작에 대해 이를 실행해보고 결과를 확인하는 테스트 케이스가 작성되어야 한다

    아주 간단한 테스트 케이스가 아니라면 각 테스트 케이스 코드는 준비, 실행 및 단언의 세가지 부분으로

    나누는 것이 일반적이다
- 바람직한 단위 테스트의 주요 특징은 다음과 같다
  - 문제가 생긴 코드의 정확한 탐지
  - 구현 세부 정보에 구애받지 않음
  - 실패가 잘 설명됨
  - 이해하기 쉬운 테스트 코드
  - 쉽고 빠르게 실행
- 테스트 더블은 의존성을 실제로 사용하는 것이 불가능하거나 현실적으로 어려울 때 단위 테스트에 사용할 수 있다
  - 목
  - 스텁
  - 페이크
- 목 및 스텁을 사용한 테스트 코드는 비현실적이고 구현 세부 정보에 밀접하게 연결될 수 있다
- 목과 스텁의 사용에 대한 여러 의견이 있다. 가능한 한 실제 의존성이 테스트에 사용되어야 한다

    이렇게 할 수 없다면 페이크가 차선책이고, 목과 스텁은 최후의 수단으로만 사용되어야 한다

    
