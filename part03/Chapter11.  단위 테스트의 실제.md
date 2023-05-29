# 11.단위 테스트의 실제

## 11.1 기능뿐만 아니라 동작을 시험하라

### 11.1.1 함수당 하나의 테스트 케이스만 있으면 적절하지 않을 때가 많다

주택 담보 대출 신청을 자동으로 평가하는 시스템이 있다고 가정하자.
- assess()함수는 프라이빗 헬퍼 함수를 호출해 고객이 주택담보대출을 받을 자격이 있는지 여부를 판단한다. 다음과 같은경우에 자격이 부여된다
  - 신용등급이 좋다
  - 기존 주택담보대출이 없다
  - 은행에 의해 금지된 고객이 아니다
  - 고객이 자격이 있을 경우 다른 프라이빗 헬퍼 함수를 호출해 해당고객에 대한 최대 대출 금액을 결정한다

```java
class MortgageAssessor {
    private const Double MORTAGE_MULTIPLIER = 10.0;

    MortgageDecision assess(Customer customer) {
        if (!isEligibleForMortage(customer)) {
            return MortgageDecision.rejected();
        }
        return MortgageDecision.approve(getMaxLoanAmount(customer));
    }

    private static Boolean isEligibleForMortgage(Customer customer) {
        return customer.hasGoodCreditRating() &&
            !customer.hasExistingMortgage() &&
            !customer.isBanned();
    }

    private static MonetaryAmount getMaxLoanAmount(Customer customer) {
        return cujstomer.getIncome()
            .minus(customer.getOutgoings())
            .multiplyBy(MORTGAGE_MULTIPLIER)
    }
}
```

테스트 코드는 다음과 같다

```java
testAssess() {
    Customer customer = new Customer(
        income : new MonetaryAmount(50000, Currency.USD),
        outgoings: new MonetaryAmount(20000, Currency.USD),
        hasGoodCreditRating: true,
        hasExistingMortgage: false,
        isBanned: false);
    
    MortgageAssessor mortgageAssessor = new MortgageAssessor();

    MortgageDecision decision = mortgageAssessor.assess(customer);

    assertThat(decision.isApporved()).isTrue();
    assertThat(decision.getMaxLoanAmount()).isEqualTo(
        new MonetaryAmount(300000, Currency.USD));
}
```

여기서 문제는 테스트를 작성하는 개발자가 행동이 아닌 기능 테스트에 집중했다는 점이다

assess()함수는 MortgageAssessor 클래스의 퍼블릭 API에 있는 유일한 함수이기 때문에 하나의 테스트 케이스만 작성했다.

안타깝게도 이 하나의 테스트 케이스는 MortgageAssessor.assess() 함수가 올바른 방식으로 동작하는지 확인하기에 충분치 않다

### 11.1.2 해결책: 각 동작을 테스트하는 데 집중하라

함수와 동작 사이에 일대일로 연결이 안되는 경우가 많다. 함수 자체를 테스트하는 데만 집중하면, 정작 실제로 신경 써야 할 중요한 

동작을 검증하지 않는 테스트 케이스를 작성하기가 매우 쉽다. 다음과 같은 동작이 관심의 대상이 된다

- 다음 중 적어도 하나에 해당하는 고객에 대해서는 담보 대출 신청이 기각된다
  - 신용 등급이 좋지 않다
  - 이미 대출이 있다
  - 대출이 금지된 고객이다 
- 주택담보대출 신청이 받아들여진다면 최대 대출 금액은 고객의 수입에서 지출을 뺀 금액에 10을 곱한 금액이다

`모든 동작이 테스트되었는지 거듭 확인하라`


코드가 제대로 테스트되는지 여부를 측정하기 위한 한 가지 좋은 방법은 수정된 코드에 버그나 오류가 있음에도 여전히 테스트를 통과할 수 

있는지에 대해 생각해보는 것이다. 다음과 같은질문을 코드를 검토하는 과정에서 던져보면 좋다

- 삭제해도 여전히 컴파일되거나 테스트가 통과하는 코드 라인이 있는가?
- if문의 참 거짓 논리를 반대로 해도 테스트가 통과하는가?
- 논리 연산자나 산술 연산자를 다른것으로 대체해도 테스트가 통과하는가?
- 상숫값이나 하드 코딩된 값을 변경해도 테스트가 통과하는가?

요점은 테스트 대상 코드의 각 줄, if문, 논리표현식, 값 등은 그것이 존재하는 이유가 있어야한다는 것이다. 불필요한 코드라면 그것은 제거되야한다

불필요한 것이 아니라면 그것은 어떻게든 그것에 의존하는 어떤 중요한 행동이 있다는 것을 의미한다. 코드가 나타내는 중요한 동작이

있는 경우 그 동작을 테스트하는 테스트 케이스가 있어야 하므로, 기능을 변경한 경우 적어도 하나의 테스트케이스가 실패해야 한다.

`오류 시나리오를 잊지 마라`

간과하기 쉬운 또다른 중요한 동작은 오류 시나리오가 발생할 때 코드가 어떻게 동작하는가다. 오류가 자주 발생할 것으로 예상하지 않기

때문에 이러한 경우는 다소 경계조건처럼 보일 수 있다.

코드로 예시를 보자

```java
class BankAccount {
    void debit(MonetaryAmount amount) {
        if (amount.isNegative()) {
            throw new ArgumentException("액수는 0보다 적을 수 없음");
        }
    }
}
```

테스트 코드는 다음과 같다

```java
void testDebit_negativeAmount_throwsArgumentException {
    MonetaryAmount negativeAmount = new MonetaryAmount(-0.01, Currency.USD);
    BankAccount bankAccount = new BankAccount();

    ArgumentException exception = assertThrows(
        ArgumentException,
        () -> bankAccount.debit(negativeAmount));
    assertThat(exception.getMessage())
        .isEqualTo("액수는 0보다 적을 수 없음");
}
```

하나의 코드는 많은 동작을 나타내는 경향이 있으며, 하나의 함수라도 호출되는 값이나 시스템의 상태에 따라 다양한 동작을 나타내는

경우가 꽤 많다. 함수당 하나의 테스트 케이스만 작성하는 것으로 테스트가 충분히 되는 경우는 거의 없다. 함수에 집중하기 보다는

궁극적으로 중요한 모든 행동을 파악하고 각각에 대한 테스트 케이스가 있는지 확인하는 것이 더 효과적이다

## 11.2 테스트만을 위해 퍼블릭으로 만들지 말라

### 11.2.1 프라이빗 함수를 테스트하는 것은 바람직하지 않을 때가 많다

```java
class MortgageAssessor {
    MortgageDecision assess(Customer customer){...}

    private static Boolean isEligibleForMortgage(Customer customer){...}

    private static MonetaryAmount getMaxLoanAmount(Customer customer){...}
}
```


여기서 private 함수를 테스트를 위해 퍼블릭으로 변경하고 테스트할 떄의 문제는 다음과 같다

- 이 테스트는 실제로 우라가 신경쓰는 행동을 테스트하는 것이 아니다.
- 이렇게 되면 테스트가 구현 세부 사항에 독립적이지 못하게 된다
- 테스트를 위해서만 공개와 같은 주석문은 간과되기 쉽기 때문에 다른 개발자가 이 함수를 호출하여 기능을 의존할 수 있다

### 11.2.2 해결책: 퍼블릭 API를 통해 테스트하라

테스트를 위해 프라이빗 함수를 퍼블릭으로 만들 필요가 없다

```java
testAssess_badCreditRating_mortgageRejected() {
    Customer customer = new Customer(
        income : new MonetaryAmount(50000, Currency.USD),
        outgoings: new MonetaryAmount(20000, Currency.USD),
        hasGoodCreditRating: true,
        hasExistingMortgage: false,
        isBanned: false);
    MortgageAssessor mortgageAssessor = new MortgageAssessor();

    MortgageDecision decision = mortgageAssessor.assess(customer);

    assertThat(decision.isApproved()).isFalse();
}
```


비교적 간단한 클래스의 경우 퍼블릭 API만을 사용하여 모든 동작을 테스트하기가 매우 쉽다. 이렇게 하면 코드의 문제점을

보다 정확하게 감지하고 구현 세부 사항에 얽메이지 않는 더 나은 테스트를 수행할 수 있다. 그러나 클래스가 더 복잡하거나 많은

논리를 포함하면 퍼블릭 API를 통해 모든 동작을 테스트하는 것이 까다로울 수 있다. 이 경우는 코드의 추상화 계층이 너무 크다는 것을

의미하기 때문에 코드를 더 작은 단위로 분할하는 것이 유익하다

### 11.2.3 해결책: 코드를 더 작은 단위로 분할하라


실제로 프라이빗 함수가 복잡한 논리를 갖게되면 이 함수를 퍼블릭으로 만들어 테스트하고자 하는 마음이 들 수 있다.

다음과 같이 복잡한 논리가 있다고 보자

- MorbageAssessor 클래스는 CreditScoreService에 의존한다
- 고객의 신용 점수를 조회하기 위해 고객 ID로 CreditScoreService에 질의한다
- CreditScoreService에 대한 호출이 실패할 수 있으므로 오류 시나리오를 처리해야 한다
- 호출이 성공하면 반환된 점수를 기준값과 비교하여 고객의 신용 등급이 양호한지 결정한다

퍼블릭 API를 통해 이러한 모든 복잡성과 오류시나리오를 테스트하는 것이 상당히 어려워 보인다.

여기에는 근본적 문제가 있는데 MortgageAssessor 클래스가 하는 일이 너무 많다는 점이다

```java
class MortgageAssessor {
    private const Double MORTGAGE_MULTIPLIER = 10.0;
    private const Double GOOD_CREDIT_SCORE_THRESHOLD = 880.0;

    private final CreditScoreService creditScoreService;

    MortgageDecision assess(Custoemr custoer) {
        ...
    }

    private Result<Boolean, Error> isEligibleForMortgage(
        Customer customer) {
            if(customer.hasExistingMortgage() || customer.isBanned()) {
                return Result.ofValue(false);
            }
            return isCreditRatingGood(customer.getId());
        }
    /** 테스트를 위해서만 공개**/
    Result<Boolean, Error> isCreditRatingGood(Int customerId) {
        CreditScoreResponse response = creditScoreService
            .query(customerId);
        if (response.errorOccurred()) {
            return Result.ofError(response.getError());
        }
        return Result.ofValue(
            response.getCreditScore() >= GOOD_CREDIT_SCORE_THRESHOLD);
    }    
}
```

퍼블릭 API만으로 모든것을 테스트하기 어려워보이는 이유는 MortgageAssessor 클래스가 너무 비대하기 때문이다

여기서 해결책은 코드를 더 작은 계층으로 나누는 것이다. 고객의 신용등급이 좋은지 판단하는 논리를 별도의 클래스로 옮기자.

```java
class CreditRatingChecker {

    private const Double GOOD_CREDIT_SCORE_THRESHOLD = 880.0;

    private final CreditScoreService creditScoreService;

    Result<Boolean, Error> isCreditRatingGood(Int customerId) {
        CreditScoreResponse response = creditScoreService
            .query(customerId);
        if (response.errorOccurred()) {
            return Result.ofError(response.getError());
        }
        return Result.ofValue(
            response.getCreditScore() >= GOOD_CREDIT_SCORE_THRESHOLD);
    }   

}

class MortgageAssessor {
    private const Double MORTGAGE_MULTIPLIER = 10.0;

    private final CreditRatingChecker creditRatingChecker;

    MortgageDecision assess(Custoemr custoer) {
        ...
    }

    private Result<Boolean, Error> isEligibleForMortgage(Customer customer) {
        if(customer.hasExistingMortgage() || customer.isBanned()){
            return Result.ofValue(false);
        }
        return creditRatingChecker
            .isCreditRatingGood(customer.getId());
    }
}
```

이로 인해 퍼블릭 API를 사용하여 둘 다 쉽게 테스트할 수 있다

코드를 테스트하기 위해 프라이빗 함수를 퍼블릭으로 만든다면, 이것은 실제로 신경써야 하는 행동을 테스트하지 않는다는 경고 신호로

받아들여야 한다. 이미 공개된 함수를 사용해 코드를 테스트하는 것이 대부분은 더 바람직하다. 이것이 어렵다면 클래스가 너무 크기 때문에

하위 문제를 해결하는 더 작은 클래스로의 분할을 고려해봐야 하는 시점에 이르렀음을 의미한다

## 11.3 한번에 하나의 동작만 테스트하라

### 11.3.1 여러 동작을 한꺼번에 테스트하면 테스트가 제대로 안 될 수 있다

유효한 쿠폰을 추려내는 코드는 다음과 같다

```java
List<Coupon> getValidCoupons(List<Coupon> coupons, Customer customer) {
    return cupons
        .filter(coupon -> !coupon.alreadyRedeemed())
        .filter(coupon -> !coupon.hasExpired())
        .filter(coupon -> coupon.issuedTo() == customer)
        .sortBy(coupon -> coupon.getValue(), SortOrder.DESCENDING);
}
```

```java
void testGetValidCoupons_allBehaviors() {
    Customer customer1 = new Customer("test customer 1");
    Customer customer2 = new Customer("test customer 2");
    Coupon redeemed = new Coupon(
        alreadyRedeemed: true, hasExpired: false,
        issuedTo: customer1, value: 100);
    Coupon expired = new Coupon(
        alreadyRedeemed: false, hasExpired: true,
        issuedTo: customer1, value: 100);
    Coupon issuedToSomeoneElse = new Coupon(
        alreadyRedeemed: false, hasExpired: false,
        issuedTo: customer2, value: 100);
    Coupon valid1 = new Coupon(
        alreadyRedeemed: false, hasExpired: false,
        issuedTo: customer1, value: 100);
    Coupon valid2 = new Coupon(
        alreadyRedeemed: false, hasExpired: false,
        issuedTo: customer1, value: 150);

    List<Coupon> validCoupons = getValidCoupons(
        [redeemed, expired, issuedToSomeoneElse, valid1, valid2], customer1);
    assertThat(validCoupons)
        .containsExactly(valid2, valid1)
        .inOrder();
}
``` 
위처럼 한번에 여러 동작을 테스트하면 테스트가 실패하더라도 실패에 대한 설명이 자세히 제공되지 않을 수 있다.

테스트 코드가 이해하기 어렵고 통과하지 못한 경우 이유를 제대로 설명하지 않으면, 다른 개발자의 시간을 낭비할 뿐만 아니라

버그가 발생할 가능성도 커진다.

모든 것을 한꺼번에 테스트하는 테스트 케이스는 정확히 무엇이 변경됐는지 알려주는 대신, 무언가 변경됐다는 것만 알려준다.

따라서 코드를 의도적으로 변경할 때 그 변경으로 인해 어떤 동작이 영향을 받았고 어떤 동작이 영향을 받지 않았는지 정확히 알기 어렵다


### 11.3.2 해결책 : 각 동작은 자체 테스트 케이스에서 테스트하라

훨씬 더 나은 접근법은 잘 명명된 테스트 케이스를 사용하여 각 동작을 개별적으로 테스트하는 것이다.

이렇게 하면 각 테스트 케이스 안의 코드가 훨씬 간단하고 이해하기 쉽다는 것을 알 수 있다. 각 테스트 케이스 이름에서 어떤 동작을

테스트 하고 있는지 정확하게 식별할 수 있으며, 테스트의 작동 방식을 이해하기 위해 코드를 파악하는 것이 비교적 쉽다.

단위 테스트 코드는 이해하기 쉬워야 한다는 관점에서 보면 테스트 코드가 크게 개선 됐다

```java
void testGetValidCoupons_validCoupon_include() {
    Customer customer = new Customer("test customer");
    Coupon valid = new Coupon(
        alreadyRedeemed: false, hasExpired: false,
        issuedTo: customer, value: 100);
    List<Coupon> validCoupons = getValidCoupons([valid], customer);

    assertThat(validCoupons).containsExactly(valid);
}

void testGetValidCoupons_alreadyRedeemed_excluded() {
    Customer customer = new Customer("test customer");
    Coupon redeemed = new Coupon(
        alreadyRedeemed: true, hasExpired: false,
        issuedTo: customer, value: 100);
    List<Coupon> validCoupons = getValidCoupons([redeemed], customer);

    assertThat(validCoupons).isEmpty();   
}

void testGetValidCoupons_expired_excluded(){...}

void testGetValidCoupons_issuedToDifferentCustomer_excluded(){...}

void testGetValidCoupons_returnedInDescendingValueOrder(){...}
```

이처럼 각 동작을 하나의 테스트 케이스로 테스트하면 장점이 있지만, 코드 중복이 많아지는 단점도 있다.

각 테스트 케이스에 사용된 값과 설정이 일부 사소한 차이를 제외하고 거의 동일한 경우 이렇게 별도의 테스트 케이스를 작성하는 것이

번거로워 보일 수 있다. 이런 코드 중복을 줄이는 한 가지 방법은 매개변수화된 테스트를 사용하는 것이다

### 11.3.3 매개변수를 사용한 테스트



매개변수를 사용한 테스트의 예시는 다음과 같다 (수도코드)

```java
[testCase(true, false, TestName= "이미 사용함")]
[testCase(false, true, TestName = "유효기간 만료")]
void testGetValidCoupons_excludesInvalidCoupons(
    Boolean alreadyRedeemed, Boolean hasExpired) {
        Customer customer = new Customer("test customer");
        Coupon valid = new Coupon(
            alreadyRedeemed: alreadyRedeemed, hasExpired: hasExpired,
            issuedTo: customer, value: 100);

        List<Coupon> validCoupons = 
            getValidCoupons([coupon], customer);
        assertThat(validCoupons).isEmpty();
    }
```
## 11.4 공유 설정을 적절하게 사용하라

### 11.4.1 상태 공유는 문제가 될 수 있다.

일반적으로 테스트 케이스는 서로 격리되어야 하므로 한 테스트 케이스가 수행하는 모든 조치는 다른 테스트 케이스의 결과에 

영향을 미치지 않아야 한다. 테스트 케이스 간에 상태를 공유하고 이 상태가 가변적이면 이 규칙을 위반하기 쉽다.

테스트 케이스 간 상태를 공유하는 예시 코드를 보자

```java
class OrderManagerTest {
    private Database database;

    @BeforeAll
    void oneTimeSetUp(){
        database = Database.createInstance();
        database.waitForReady();
        //동일한 데이터베이스 인스턴스가 모든 테스트 케이스 간에 공유된다
    }

    void testProcessOrder_outOfStockItem_orderDelayed() {
        Int orderId = 12345;
        Order order = new Order (
            orderId: orderId,
            containsOutOfStockItem: true,
            isPaymentComplete: true);
        OrderManager orderManager = new OrderManager(database);

        orderManager.processOrder(order);

        assertThat(database.getOrderStatus(orderId))
            .isEqualTo(OrderStatus.DELAYED);
    }

    void testProcessOrder_paymentNotComplete_orderDelayed() {
        Int orderId = 12345;
        Order order = new Order (
            orderId: orderId,
            containsOutOfStockItem: false,
            isPaymentComplete: true);
        OrderManager orderManager = new OrderManager(database);

        orderManager.processOrder(order);

        assertThat(database.getOrderStatus(orderId))
            .isEqualTo(OrderStatus.DELAYED);
    }

}
```

서로 다른 테스트 케이스 간에 가변적인 상태를 공유하면 문제가 발생하기 매우 쉽다. 가능하다면 상태를 공유하지 않는 것이 최선이다.

하지만 상태 공유가 꼭 필요하다면 한 테스트 케이스에 의해 변경된 상태가 다른 테스트 케이스에 영향을 미치지 않도록 조심해야 한다

### 11.4.2 해결책: 상태를 공유하지 않거나 초기화하라

가변적인 상태를 공유하는 데서 오는 문제점을 해결하기 위한 가장 분명한 방법은 애초에 공유하지 않는 것이다.

이럴때는 AfterEach블록을 사용하여 테스트 케이스 간에 데이터 베이스를 초기화하자

```java
class OrderManagerTest{
    private Database database;

    @BeforeAll
    void oneTimeSetUp(){
        database = Database.createInstance();
        database.waitForReady();
        //동일한 데이터베이스 인스턴스가 모든 테스트 케이스 간에 공유된다
    }

    @AfterEach
    void tearDown(){
        database.reset();
    }
    ...
}
```

테스트 케이스 간에 가변적인 상태를 공유하는 것은 이상적이지 않다. 피할 수 있다면 일반적으로 공유하지 않는 것이 바람직하다

피할 수 없다면 각 테스트 케이스 간에 상태를 초기화해야 한다. 이를 통해 한 테스트 케이스가 다른 테스트 케이스에 악영향을 미치지 않도록 해야한다

### 11.4.3 설정 공유는 문제가 될 수 있다

테스트 케이스 간 설정을 공유하는 것은 상태를 공유하는 것만큼 위험해 보이지는 않지만 설정을 공유하면 테스트가 효과적이지

못할 때가 있다. 주문 처리 예에서 소포에 대한 우편 요금 라벨을 생성하는 시스템이 또 다른 인프라 시스템으로 존재한다고 가정해보자.

```java
class OrderPostageManager{
    PostageLabel getPostageLabel(Order order) {
        return new PostageLabel(
            address: order.getCustomer().getAddress(),
            isLargePackage: order.getItems().size() > 2
        );
    }
}
```

테스트 설정을 공유하는 코드는 다음과 같다

```java
class OrderPostageManagerTest {
    private Order testOrder;

    @BeforeEach
    void setUp() {
        testOrder = new Order(
            customer: new Customer(
                address: new Address("Test address"),
            ),
            items: [
                new Item(name: "Test item 1"),
                new Item(name: "Test item 2"),
                new Item(name: "Test item 3")
            ]);
    }
    ...

    void testGetPostageLabel_threeItems_largePackage() {
        PostageManager postageManager = new PostageManager();

        PostageLabel label = postageManager.getPostageLabel(testOrder);

        assertThat(label.isLargePackage()).isTrue();
    }
}

```

이 테스트 코드는 신경 써야 할 행동 중 하나를 테스트하고 모든 테스트 케이스마다 주문을 생성하기 위한 번거로운 코드의

반복을 피한다. 하지만 안타깝게도 다른 개발자가 테스트를 수정해야 한다면 일이 잘못될 수도 있다.

다른 개발자가 getPostageLabel() 함수에 새로운 기능을 추가한다고 가정하면 다음과 같이 될것이다

```Java
class PostrageManager {
    PostageLabel getPostageLabel(Order order) {
        return new PostageLabel(
            address: order.getCustomer().getAddress(),
            isLargePackage: order.getItems().size() > 2,
            isHazardous: containsHazardousItem(order.getItems()));
    }

    private static Boolean containsHazardousItem(List<Item> items) {
        return items.anyMatch(item -> item.isHazardous());
    }
}
```

예를 들어 위와 같은 기능이 추가되었을 떄 이제는 네 개의 항목이 있을 때 발생하는 결과를 테스트하면 문제가 발생하기에 더 이상 완벽하게 보호하지 못한다

```java
class OrderPostageManagerTest {
    private Order testOrder;

    @BeforeEach
    void setUp() {
        testOrder = new Order(
            customer: new Customer(
                address: new Address("Test address"),
            ),
            items: [
                new Item(name: "Test item 1"),
                new Item(name: "Test item 2"),
                new Item(name: "Test item 3"),
                new Item(name: "Hazardous item", isHazardous: true)
            ]);
    }
    ...

    void testGetPostageLabel_threeItems_largePackage() {
        PostageManager postageManager = new PostageManager();

        PostageLabel label = postageManager.getPostageLabel(testOrder);

        assertThat(label.isLargePackage()).isTrue();
    }
}

```

설정을 공유하는 것은 코드의 반복을 피하는 데는 유용하지만 일반적으로 테스트 케이스에 중요한 값이나 상태는 고융하지 않는 것이 최선이다

설정을 공유하면 어떤 테스트 케이스가 어떤 특정 항목에 의존하는지 정확하게 추적하는 것은 매우 어려우며, 향후 변경 사항이

발생하면 테스트 케이스가 원래 목적했던 동작을 더 이상 테스트하지 않게 될 수 있다


### 11.4.4 해결책: 중요한 설정은 테스트 케이스 내에서 정의하라

모든 테스트 케이스에 대해 반복해서 설정을 하는 것이 어려워 보일 수 있지만 테스트 케이스가 특정 값이나 설정 상태에 의존한다면

그렇게 하는 것이 더 안전한 경우가 많다. 보통 헬퍼 함수를 사용해 이 작업을 좀 더 쉽게 할 수 있기 때문에 코드를 반복하지 않아도 된다

테스트 케이스 내에서 이루어지는 중요 설정 코드의 예시는 다음과 같다

```java
class OrderPostageManagerTest {
    ...

    void testGetPostageLabel_threeItems_largePackage() {
        
        Order order = createOrderWithItems([
            new Item(name: "Test item 1"),
            new Item(name: "Test item 2"),
            new Item(name: "Test item 3")
        ])
        
        PostageManager postageManager = new PostageManager();

        PostageLabel label = postageManager.getPostageLabel(testOrder);

        assertThat(label.isLargePackage()).isTrue();
    }

    void testGetPsotageLabel_hazardousItem_isHazardous() {
        Order order = createOrderWithItems([
            new Item(name: "Hazardous item", isHazardous: true)
        ]);

        PostageManager postageManager = new PostageManager();

        PostageLabel label = postageManager.getPostageLabel(order);

        assertThat(label.isHazardous()).isTrue()
    }
    ...

    private static Order crateOrderWithItems(List<Item> items) {
        return new Order(
            customer: new customer(
                address: new Address("Test address")
            ),
            items: items);
    }
}
```

테스트 케이스의 결과가 설정값에 직접 영향을 받는 경우 해당 테스트 케이스 내에서 설정하는 것이 가장 좋다.

이렇게 하면 향후의 코드 변경으로 인해 의도치 않게 테스트 코드에 문제가 발생하는 것을 막을 수 있다.

그뿐만 아니라 테스트 케이스에 의미 있는 방식으로 영향을 미치는 설정이 모두 테스트케이스 내에 있기 떄문에

각 테스트 케이스에서 원인과 결과가 명확해진다. 

### 11.4.5 설정 공유가 적절한 경우

테스트 설정 공유를 주의해야 하지만 테스트 설정을 절대 공유해서는 안된다는 의미는 아니다.

필요하면서도 테스트 케이스의 결과에 직접적인 영향을 미치지는 않는 설정이 있을 수 있다.

이같은 경우에는 설정 공유를 통해 불필요한 코드 반복을 피할 수 있고 테스트는 좀 더 뚜렷한 목적을 갖고 이해하기 쉬워진다.

반드시 필요하지만 테스트와 그다지 관련 없는 데이터를 테스트 케이스에서 반복적으로 설정할 필요 없을 떄 이럴때 사용하기가 적합하다


## 11.5 적절한 어서션 확인자를사용하라

`어서션 확인자(assertion matcher)`는 보통 테스트 통과 여부를 최종적으로 결정하기 위한 테스트 케이스 내의 코드다

```java
assertThat(someValue).isEqualTo("expected value");
assertThat(someList).contains("expected value");
```

테스트 케이스가 실패하면 어서션 확인자는 실패 이유를 설명하는 메시지를 생성한다. 각각의 어서션 확인자는 자신들의 목적에 따라

각자 다른 실패 메시지를 생성한다. 

### 11.5.1 부적합한 확인자는 테스트 실패를 잘 설명하지 못할 수 있다

```java
class TextWidget {
    private const ImmutableList<String> STANDARD_CLASS_NAMES = 
        ["test-widget", "selectable"];
    private final ImmutableList<String> customClassNames;

    TextWidget(List<String> customClassNames) {
        this.customClassNames = ImmutableList.copyOf(customClassNames);
    }

    /**
     * 구성 요소에 대한 클래스 이름을 반환한다. 반환된 리스트에서 클래스 이름은
     * 특정한 순서를 갖지 않는다
     */
     ImmutableList<String> getClassNames() {
        return STANDARD_CLASS_NAMES.concat(customClassNames);
     } 
}
```

```java
void testGetClassNames_containsCustomClassNames() {
    TextWidget textWidget = new TextWidget(
        ["custom_class_1", "custom_class_2"]);
    assertThat(textWidget.getClassNames()).isEqualTo([
        "text-widget",
        "selectable",
        "custom_class_1",
        "custom_class_2"
    ]);
}
```

위의 테스트는 클래스 이름이 반환되는 순서가 변경되면 테스트가 실패한다. 이런 테스트 코드는 잘못된 경고나 취약한 테스트로 이어질 수 있다

```java
void testGetClassNames_containsCustomClassNames() {
    TextWidget textWidget = new TextWidget(
        ["custom_class_1", "custom_class_2"]);
    
    ImmutableList<String> result = textWidget.getClassNames();

    assertThat(result.contains("custom_class_1")).isTrue()
    assertThat(result.contains("custom_class_2")).isTrue()

}
```

위의 테스트는 사용자 정의 클래스 중 하나가 없어 테스트 케이스가 실패해도 실제 결과가 예상 결과와 어떻게 다른지 설명하지 않는다.

### 11.5.2 해결책 : 적절한 확인자를 사용하라

적절한 어서션 확인자로 코드를 변경하면 다음과 같다

```java
void testGetClassNames_containsCustomClassNames() {
    TextWidget textWidget = new TextWidget(
        ["custom_class_1", "custom_class_2"]);
    
    assertThat(textWidget.getClassNames())
        .containsAtLeast("custom_class_1", "custom_class_2");
}
```

이 경우 테스트가 실패할 경우 실패의 이유도 잘 설명한다. 

코드에 문제가 있을 때 테스트가 반드시 실패해야 한다는 점 외에도 테스트가 어떻게 실패할지에 대해 생각해보는 것이 중요하다.

적절한 어서션 확인자를 사용하면 테스트 실패 시 실패의 이유에 대해 잘 알 수 있지만, 그렇지 않으면 실패의 이유를

명확히 파악하기 어렵기 때문에 테스트 코드를 실행할 때 다른 개발자가 어려움을 겪을 수 있다

## 11.6 테스트 용이성을 위해 의존성 주입을 사용하라

### 11.6.1 하드 코딩된 의존성은 테스트를 불가능하게 할 수 있다.

의존성 주입을 하지 않는 클래스는 다음과 같다

```java
class InvoiceReminder {
    private final Address addressBook;
    private final EmailSender emailSender;

    InvoiceReminder() {
        this.addressBook = DataStore.getAddressBook();
        this.emailSender = new EmailSenderImpl();
    }

    @CheckReturnValue
    Boolean sendReminder(Invoice invoice) {
        EmailAddress address = 
            addressBook.lookupEmailAddress(invoice.getCustomerId());
        
        if (address == null)
            return false;

        return emailSender.send(
            address,
            InvoiceReminderTemplate.generate(invoice));
    }
}
```

- 이 클래스는 생성자 내에서 DataSource.getAddressBook()을 호출해 자체적으로 인스턴스를 생성한다

    이점은 시간이 지남에 따라 데이터베이스에 데이터가 변경될 수 있기 때문에 실제 고객 데이터를 테스트에 사용하는 것은 적합하지 않다.

    또 다른 근본적인 문제는 테스트 환경에서는 실제 데이터베이스에 액세스할 수 있는 권한이 없기 때문에 반환된 주소록이 작동하지 않을 수 있다

- 또 이 클래스는 자체적으로 EmailSenderImpl을 생성한다. 따라서 이메일을 실제로 보내는 겨로가를 초래한다. 

    이것은 테스트에서 일어나면 안되는 부수효과다

일반적으로 이 두문제에 대한 손쉬운 해결책은 테스트 더블을 사용하는 것이다. 하지만 이 경우 실제 의존성 대신

테스트 더블을 사용해 InvoiceReminder 클래스의 인스턴스를 생성할 수 없기 떄문에 이 방법을 사용할 수 없다.

InvoiceReminder 클래스는 테스트 용이성이 낮고, 이로 인해 일부 동작이 제대로 테스트되지 않아 코드에 버그가 발생할 가능성이 커진다

### 11.6.2 해결책: 의존성 주입을 사용하라

의존성 주입을 사용하면 InvoiceReminder 클래스는 테스트하기 쉬워지고 이 문제를 해결할 수있다.

```java
@RequiredArgsConstructor
class InvoiceReminder {
    private final AddressBook addressBook;
    private final EmailSender emailSender;

    static InvoiceReminder create() {
        return new InvoiceReminder(
            DataSource.getAddressBook(),
            new EmailSenderImpl());
    }

    @CheckReturnValue
    Boolean sendReminder(Invoice invoice) {
        EmailAddress address = 
            addressBook.lookupEmailAddress(invoice.getCustomerId());
        
        if (address == null)
            return false;

        return emailSender.send(
            address,
            InvoiceReminderTemplate.generate(invoice));
    }
}
```

이제 테스트 더블을 사용하ㅕ 테스트 코드에서 InvoiceReminder 클래스를 생성하는 것이 아주 쉬워졌다

```java
...
FakeAddressBook addressBook = new FakeAddressBook();

fakeAddressBook.addEntry(
    customerId: 123456,
    emailAddress: "test@example.com");
FakeEmailSender emailSender = new FakeEmailSender();

InvoiceReminder invoiceReminder =
    new InvoiceReminder(addressBook, emailSender);

...
```
- `통합 테스트(Integration test)` : 한 시스템은 일반적으로 여러 구성 요소, 모듈, 하위 스스템으로 구성 된다.

    이러한 구성 요소와 하위 시스템을 서로 연결하는 프로세스를 통합이라고 한다. 통합테스트는 이러한 통합이 제대로 작동하는지 

    확인하기 위한 테스트다
- `종단 간 테스트 (end-to-end test)` : 이 테스트는 처음부터 끝까지 전체 소프트웨어 시스템을 통과하는 여정을 테스트한다.

    테스트하려는 소프트웨어가 온라인 쇼핑몰이라면, E2E 테스트의 예로는 웹 브라우저를 자동으로 구동하고 사용자가 구매를 

    완료하는 과정까지 거치면서 구매가 잘 이루어지는지 확인하는 것이다


# 11장 요약
- 각 함수를 테스트 하는 것에 집중하다 보면 테스트가 충분치 되지 못하기 쉽다. 보통은 모든 중요한 행동을 파악하고 각각의

    테스트 케이스를 작성하는 것이 더 효과적이다
- 결과적으로 중요한 동작을 테스트해야 한다. 프라이빗 함수를 테스트하는 것은 거의 대부분 결과적으로 중요한 사항을 테스트하는 것이 아니다
- 한번에 한가지씩만 테스트하면 테스트 실패의 이유를 더 잘 알 수 있고 테스트 코드를 이해하기가 더 쉽다
- 테스트 설정 공유는 양날의 검이 될 수 있다. 코드 반복이나 비용이 큰 설정을 피할 수 있지만 부적절하게 사용할 경우

    효과적이지 못하거나 신뢰할 수 없는 결과를 초래할 수 있다
- 의존성 주입을 사용하면 코드의 테스트 용이성이 상당히 향상될 수 있다
- 단위 테스트는 개발자들이 가장 자주 다루는 테스트 수준이지만 이것이 유일한 테스트는 아니다

    높은 품질의 소프트웨어를 작성하고 유지하려면 여러 가지 테스트 기술을 함께 사용해야 할 때가 많다

    