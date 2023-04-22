# 7. 코드를 오용하기 어렵게 만들라

코드를 잘못 사용할 수 있는 몇가지 일반적인 경우는 다음과 같다

- 호출하는 쪽에서 잘못된 입력을 제공
- 다른 코드의 부수 효과(입력 매개변수 수정 등)
- 정확한 시간이나 순서에 따라 함수를 호출하지 않음
- 관련 코드에서 가정과 맞지 않게 수정이 이루어짐

## 7.1 불변 객체로 만드는 것을 고려하라

가변객체는 다음과 같은 오류를 일으킬 수 있다

- `가변 객체는 추론하기 어렵다`
- `가변 객체는 다중 스레드에서 문제가 발생할 수 있다`

### 7.1.1 가변 클래스는 오용하기 쉽다

```kotlin
class TextOptions (
    var font: Font,
    var fontSize: Double
) {
    // getter, setter
}

class MessageBox (
    val titleField: TextField,
    val messageField: TextField
) {
    fun renderTitle(title: String, baseStyle : TextOptions) {
        baseStyle.setFontSize(18.0) // TextOptions의 인스턴스는 폰트 크기를 수정함으로써 변경됨
        titleField.display(title, baseStyle)
    }

    fun renderMessage(message: String, style: TextOptions) {
        messageField.display(message, style)
    }
}

```

TextOptions 클래스는 가변적이기 때문에 해당 인스턴스를 전달받는 모든 코드는 이 객체를 변경할 수 있고 이로 인해 오용의

위험성이 있다. 

### 7.1.2 해결책: 객체를 생성할 때만 값을 할당하라

모든 값이 객체의 생성 시에 제공되고 그 이후로는 변경할 수 없도록 함으로써 클래스를 불변적으로 만들 수 있고 오용도 방지할 수 있다.

```kotlin
data class TextOptions (
    val font : Font,
    val fontSize : Double
)

class MessageBotx(
    val titleField : TextField
) {
    fun renderTitle(title: String, baseStyle: TextOptions) {
        titleField.display(tiel, baseStyle.withFontSize(18,0)) // baseStyle을 복사하고 폰트 크기 변경해 반환
    }
}
```

위에서 TextOptions에서는 모든 텍스트 옵션값이 필요하다. 그러나 모든 텍스트 옵션값이 반드시 필요한 것이 아니라면 빌더 패턴이나

쓰기 시 복사 패턴을 사용하는 것이 좋다.

### 7.1.3 해결책: 불변성에 대한 디자인 패턴을 사용하라

일부 값이 반드시 필요하지 않거나 불변적인 클래스의 가변적 버전을 만들어야 하는 경우, 클래스를 보다 다용도로 구현해야 할 필요가 있다.

이를 위한 두가지 유용한 패턴은 다음과 같다

- 빌더 패턴
- 쓰기 시 복사 패턴

## 7.2 객체를 깊은 수준까지 불변적으로 만드는 것을 고려하라

클래스가 무심코 가변적으로 될 수 있는 미묘한 경우를 간과하기 쉬운데. 이는 클래스가 실수로 가변적으로 될 수 있는 `깊은 가변성` 때문이다.

이 문제는 멤버 변수 자체가 가변적인 유형이고 다른 코드가 멤버 변수에 액세스할 수 있는 경우에 발생할 수 있다

### 7.2.1 깊은 가변성은 오용을 초래할 수 있다

```java
class TextOptions {
    private final List<Font> fontFamily;
    private final Double fontSize;
}
```

이는 생성된 후에 다음과 같은 문제가 있을 수 있다

```java
fontFamily.add(Font.COMIC_SANS)
```

또한 안의 내용도 수정이 가능하다

```java
List<Font> fontFamily = textOptions.getFontFamily()
fontFamily.clear();
fontFamily.add(Font.COMIC_SANS)
```

### 7.2.2 해결책: 방어적으로 복사하라

```java
class TextOptions {
    private final List<Font> fontFamily;
    private final Double fontSize;

    List<Font> getFontFamily() {
        return List.copyOf(fontFamily);
    }
}
```

하지만 이는 다음과 같은 단점도 존재한다

- 복사하는데 비용이 많이 들 수 있다
- 클래스 내부에서 발생하는 변경을 막아주지는 못한다

### 7.2.3 해결책: 불변적 자료구조를 사용하라

자바의 경우 Guava를 사용할 수 있다

```java
class TextOptions {
    private final ImmutableList<Font> fontFamily;
    private final Double fontSize;
}    
```

불변적 자료구조를 사용해 클래스가 깊은 불변성을 갖도록 보장하기 위한 좋은방법이 된다. 또한 방어적으로 복사해야 하는

단점도 피하고 실수로라도 클래스 내의 코드에서 변경되지 않도록 보장한다

## 7.3 지나치게 일반적인 데이터 유형을 피하라

### 7.3.1 지나치게 일반적인 유형은 오용될 수 있다.

지나치게 일반적인 데이터의 유형 예시는 다음과 같다

```java
class LocationDisplay {
    private final DrawableMap map;

    /**
    * 지도 위에 제공된 모든 좌표의 위치를 표시한다
    * 리스트의 리스트를 받아들이는데, 내부의 리스트는 정확히
    * 두 개의 값을 가지고 있다 첫번째 값은 위치의 위도이고
    * 두 번째 값은 경도를 나타낸다
    */
    void markLocationsOnMap(List<List<Double>> locations) {
        for (List<Double> location : locations) {
            map.markLocation(location.get(0), location.get(1)); 
        }
    }
}
```

이는 빠르고 쉬워보이지만 다음과 같이 오용하기 쉬운 단점이 있다

- List<List< Double>> 유형 자체로는 아무것도 설명해주지 않는다. 개발자가 markLocationsOnMap() 함수에 대한 주석문을 읽지않는다면 알지 못할것이다
- 개발자는 리스트에서 어떤 항목이 위도와 경도인지 혼동하기 쉽다
- 형식 안정성이 거의 없다. 컴파일러가 목록 내의 몇개의 요소가 있는지 보장할 수 없다.

### 패러다임은 퍼지기 쉽다.

```java
class MapFeature{ 
    private final Double latitude;
    private final Double longitude;

    List<Double> getLocation() {
        return [latitude, longitude];
    }
}
```

이 코드의 문제점은 다른개발자가 MapFeature 클래스를 다른 용도로 사용해야 할 경우도 어쩔 수 없이 `List<Double>` 방식을 사용해야한다는 것이다.

### 7.3.2 페어 유형은 오용하기 쉽다.

```kotlin
class LocationDisplay(
    val map : DrawableMap
) {
    fun markLocationsOnMap(locations: List<Pair<Double, Double>>) {
        for(location in locations) {
            map.markLocation(location.first, location.second)
        }
    }
}
```

- `List<Pair<Double,Double>>` 가 무슨 의미인지 여전히 이해하기 어렵다
- 개발자는 여전히 위도와 경도의 순서에 대해 혼동하기 쉽다.

### 7.3.3 해결책: 전용 유형 사용

```java

/**
* 위도와 경도를 각도로 나타낸다
*/

class LatLong {
    private final Double latitude;
    private fianl Double longitude;
}

class LocationDisplay {
    private final DrawableMap map;

    void markLocationsOnMap(List<LatLong> locations) {
        for (location : locations) {
            map.markLocation(location.getLatitude(), location.getLongitude());
        }
    }
}
```

## 7.4 시간 부분은 생략

## 7.5 데이터에 대해 진실의 원천을 하나만 가져야 한다

코드에서 숫자, 문자열, 바이트 스트림과 같은 종류의 데이터를 처리하는 경우가 많다. 데이터는 종종 두가지 형태로 제공된다

- 기본 데이터: 코드에 제공해야 할 데이터, 코드에 이 데이터를 알려주지 않고는 코드가 처리할 방법이 없다
- 파생 데이터: 주어진 기본 데이터에 기반해서 코드가 계산할 수 있는 데이터

### 7.5.1 또 다른 진실의 원천은 유효하지 않은 상태를 초래할 수 있다

다음 클래스는 대변, 차변 및 계정 잔액값으로 구성되어 있다. 계좌 잔액은 대변과 차변에서 파생될 수 있기 때문에 중복 정보이므로

이 클래스는 논리적으로 잘못된 상태로 인스턴스를 생성할 수 있다

```java
class UserAccount {
    private final Double credit;
    private final Double debit;
    private final Double balance;
}
```

아래 코드는 UserAccount 클래스가 잘못된 상태의 인스턴스를 생성하는 예를 보여준다. 개발자가 잔액을 계산할 때 대변에서 차변을 빼지 않고

반대로 차변에서 대변을 빼는 잘못을 범하고 있다

```java
UserAccount account = new UserAccount(credit, debit, debit - credit); // 잘못된 방법 계산
```

### 7.5.2 해결책: 기본 데이터를 진실의 원천으로 사용하라

```java
class UserAccount {
    private final Double credit;
    private final Double debit;

    Double getBalnace() {
        return credit - debit;
    }
}
```

## 7.6 논리에 대한 진실의 원천을 하나만 가져야 한다.

### 7.6.1 논리에 대한 진실의 원천이 여러개 있으면 버그를 유발할 수 있다


```java
class DataLogger {
    private final List<Int> loggedValues;
    
    saveValues(FileHandler file) {
        String serializedValues = loggedValues
            .map(value -> value.toString(Radix.BASE_10))
            .join(",");
        file.write(serializedValues);
    }
}

class DataLoader {

    List<Int> loadValues(FileHandler file) {
        return file.readAsString()
            .split(",")
            .map(str -> Int.parseInt(str,Radix.BASE_10);
    }
}

```

이 형식이 무엇인지에 대해 진실의 원천이 두 개 존재한다.

이 형식을 지정하는 논리가 DataLogger, DataLoader 클래스에 독립적으로 포함되어 있다. 클래스가 모두 동일한 논리를 포함하면

모든 것이 잘 동작하지만 한 클래스가 수정되고 다른 클래스가 수정되지 않으면 문제가 발생한다.

### 7.6.2 해결책 : 진실의 원천은 단 하나만 있어야 한다


```java
class IntListFormat {
    private const String DELIMITER = ",";
    private const Radix RADIX = Radix.BASE_10;

    String serialize(List<Int> values ){
        return values
            .map(value -> value.toString(RADIX))
            .join(DELIMITER);
    }

    List<Int> deserialize(String serialized) {
        return serialized
            .split(DELIMITER)
            .map(str -> Int.parseInt(str, RADIX));
    }
}

class DataLogger {
    private final List<Int> loggedValues;
    private final IntListFormat intListFormat;

    saveValues(FileHandler file) {
        file.write(intListFormat.serialize(loggedValues));
    }
}

class DataLoader {
    private final IntListFormat intListFormat;

    List<Int> loadValues(FileHandler file) {
        return intListFormat.deserialize(file.readAsString());
    }
}
```

이처럼 IntListFormat 클래스가 직렬화된 정수의 저장 형식에 대한 유일한 진실의 원천임을 알 수 있다. 이렇게 하면 앞서

DataLogger클래스에서 사용하는 형식을 변경하지만 실수로 DataLogger 클래스는 변경하지 않는 것과 같은 위험은 제거된다

# 요약
- 코드가 오용되기 쉽게 작성되고 나면 어느 시점에선가는 오용될 가능성이 크고 이것은 버그로 이어질 수 있다
- 코드가 오용되는 몇가지 일반적 사례는 다음과 같다
  - 호출하는 쪽에서 잘못된 입력을 제공
  - 다른 코드에서 일어나는 부수 효과
  - 함수 호출 시점이 잘못되거나 올바른 순서로 호출되지 않은 경우
  - 원래의 코드에 연관된 코드를 수정할 때 원래의 코드가 내포한 가정과 어긋나게 수정하는 경우
- 오용이 어렵거나 불가능하도록 코드를 설계하고 구조화하는 것이 종종 가능하다.

    이를 통해 버그 발생 가능성이 크게 줄어들고 중장기적으로 개발자의 시간을 많이 절약할 수 있다.