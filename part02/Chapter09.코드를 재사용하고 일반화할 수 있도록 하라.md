# 9. 코드를 재사용하고 일반화할 수 있도록 하라.

## 9.1가정을 주의하라

### 9.1.1 가정은 코드 재용 시 버그를 초래할 수 있다.

```java
class Article {
    private List<Section> sections;

    ...
    List<Image> getAllImages() {
        for (section : sections) {
            if(section.containsImage()) {
                // 기사 내에 이미지를 포함하는 섹션은
                // 최대 하나만 있다
                return section.getImages();
            }
        }
        return Collections.emptyList();
    }
}
```

이코드는 이미지가 포함된 섹션이 하나만 있을 것이라고 가정한다. 이 가정은 코드 내에서 주석문으로 설명되지만, 코드를 사용하는 쪽에서는 알 수 없다.

Article 클래스가 다른 용도로 재사용되거나 기사 내의 이미지 배치가 변경되는 경우 이 가정은 부정확한 것이 될 수 있다.

누구든 getAllImages()라는 함수를 보면 '모든' 이미지를 반환한다고 가정할 것이다. 불행하게도 이것은 숨겨진 가정이 사실일 때만 올바르다

### 9.1.2 해결책: 불필요한 가정을 피하라

이미지 섹션이 하나만 있다고 가정하는 것의 비용-이익 상충관계를 고려하면 그 가정이 별로 가치가 없을 수도 있다.

다른 한편으로는 약간의 성능 향상이 있지만, 다른 한편으로는 코드를 재사용하거나 요구 사항이 변경되면 버그가 발생할 가능성이 있다.

따라서 이 가정을 하지 않는 것이 더 나을수도 있다. 이 가정으로 인해 얻는 것은 거의 없고 위험만 늘어난다

```java
class Article {
    private List<Section> sections;

    ...
    List<Image> getAllImages() {
        List<Image> images = new ArrayList<>();
        for (section : sections) {
            images.addAll(section.getImages());
        }
        return images;
    }
}
```

### 9.1.3 해결책: 가정이 필요하면 강제적으로 하라

일반적으로 다음 두 가지 방법을 사용할 수 있다.

1. `가정이 깨지지 않게 만들라` : 가정이 깨지면 컴파일되지 않는 방식으로 코드를 작성할 수 있다면 가정이 항상 유지될 수 있다
2. `오류 전달 기술을 사용하라` : 가정을 깨는 것이 불가능하게 만들 수 없는 경우에는 오류를 감지하고 오류 신호 전달 기술을 사용해

    신속하게 실패하도록 코드를 작성할 수 있다

`문제의 소지가 있는, 강제되지 않은 가정`

가정을 포함하는 코드 

```java

class Article {
    private List<Section> sections;

    Optional<Section> getImageSection() {
        // 기사 내에 이미지를 퐇마하는 섹션은 
        // 최대 하나만 있다
        return sections
            .filter(section -> section.containsImages())
            .first();
    }
}
```

가정에 의존하는 호출자

```java
class ArticleReader {
    void render(Article article) {
        Optional<Section> imageSection = article.getImageSection();
        if (imageSection.isPresent()) {
            templateData.setImageSection(imageSection);
        }
    }
}
```

만약 누군가가 이미지가 들어 있는 섹션이 여러개 있는 문서를 작성한 후 이 코드를 사용한다면, 코드가 이상하고 예상치 못한 방식으로

동작하는 것을 알게 될 것이다. 오류나 경고가 발생하지 않기 때문에 모든것이 잘 작동하는 것처럼 보이지만, 실제로는 많은 이미지가

누락된 채 기사가 보일 것이다.

가정의 강제적 확인

```java
class Article{
    private List<Section> sections;

    Optional<Section> getImageSection() {
        List<Section> imageSections = sections
            .filter(section -> section.containsImages());
        
        assert(imageSection.size() <=1, "기사가 여러 개의 이미지 섹션을 갖는다");
        return imageSections.first();
    }
}
```

## 9.2 전역 상태를 주의하라 

### 9.2.1 전역 상태를 갖는 코드는 재사용하기에 안전하지 않을 수 있다

```java
class SHoppingBasket {
    private static List<Item> items = new ArrayList<>();

    static void addItem(Item item) {
        items.add(item);
    }

    static void List<Item> getItems() {
        return List.copyOf(items);
    }
}

class ViewItemWidget{
    private final Item item;
    
    void addItemToBasket() {
        ShoppingBasket.addItem(item); // 전역 상태를 수정함
    }
}

class ViewBasketWidget {
    void displayItems() {
        List<Item> items = ShoppingBasket.getItems();
    }
}
```

장바구니의 내용물을 읽고 수정하기가 너무 쉽기 때문에 이런 방식으로 전역변수를 사용하고 싶은 마음이 들 수 있다.

하지만 이렇게 전역 상태를 사용하는 코드는 재사용 시 작동이 안되고 이상한 일이 일어날 가능성이 있다

### 9.2.2 해결책: 공유 상태에 의존성 주입하라

수정된 ShoppingBasket 클래스는 다음과 같다

먼저 ShoppingBasket 클래스를 인스턴스화해야하는 클래스로 만들고 클래스의 각 인스턴스가 고유한 상태를 갖도록 한다.

```java
class SHoppingBasket {
    private final List<Item> items = new ArrayList<>();

    void addItem(Item item) {
        items.add(item);
    }

    void List<Item> getItems() {
        return List.copyOf(items);
    }
}
```

두 번째 단계는 ShoppingBasket의 인스턴스를 필요한 클래스에 주입하는 것이다

의존성 주입된 ShoppingBasket은 다음과 같다

```java
@RequiredArgsConstructor
class ViewItemWidget{
    private final Item item;
    private final ShoppingBasket basket;

    void addItemToBasket() {
        basket.addItem(item);
    }
}

@RequiredArgsConstructor
class ViewBasketWidget {
    private final ShoppingBasket basket;

    void displayItems() {
        List<Item> items = basket.getItems();
    }
}
```

프로그램의 여러 부분간에 정보를 공유하는 빠르고 쉬운 방법처럼 보이기 때문에 전역상태를 사용하고 싶은 마음이 들지도 모른다.

그러나 이것을 사용하면 코드 재사용이 전혀 안전하지 않을 수 있다. 전역 상태가 사용된다는 사실을 다른 개발자는 모를 수 있기 때문에 코드를 

재사용하려고 하면 이상한 동작과 버그가 발생할 수 있다. 프로그램의 서로 다른 부분 간에 상태를 공유해야 할 경우, 의존성 주입을

사용해 보다 통제된 방식으로 수행하는 것이 더 안전하다

## 9.3 기본 반환값을 적절하게 사용하라

### 9.3.1 낮은 층위의 코드의 기본 반환값은 재사용을 해칠 수 있다

```java
class UserDocumentSettings {
    private final Font font;

    Font getPreferredFont() {
        if (font != null) {
            return font;
        }
        // 사용자가 선호하는 폰트가 지정되어 있지 않은 경우
        // 기본값인 Font.ARIAL을 반환한다
        return Font.ARIAL; 
    }
}
```

이 방식은 적응성에도 해가 될 수 있는데, 기본값과 관련된 요구사항이 변경되는 경우 문제가 될 수 있다. 

예시를 들어 워드프로세서를 큰 회사에 판매하기 시작햇는데, 이 회사가 기본 글꼴을 전사적으로 지정할 수 있기를 원한다고 하자.

이 기능은 구현이 어려운데 UserDoccumentSettings 클래스에 대해 사용자가 제공한 환경설정이 없고, 따라서 조직 전체의 기본값이

적용되어야 하는 경우를 구분할 수 없기 때문이다.

### 9.3.2 해결책 : 상위 수준의 코드에서 기본값을 제공하라

기본값에 대한 결정을 UserDocumentSettings 클래스에서 하지 않도록 하기 위한 가장 간단한 방법은 사용자가 제공한 값이 없을 때

널값을 반환하는 것이다.

```java
class UserDocumentSettings {
    private final Font font;

    Font getPreferredFont() {
        return font;
    }
}
```

따라서 기본값 제공은 사용자 설정 처리와는 다른 별개의 하위 문제가 된다. 이는 호출하는 쪽에서 원하는 방식으로 이 하위 문제를 해결할 수 있다는

것을 의미하며, 코드의 재사용성을 높여준다. 다음 예제 코드에서 볼 수 있듯 상위 수준에서 기본값을 제공하는 하위 문제를 해결하기 위한

전용 클래스를 정의할 수 있다

```java
class DeafaultDocumentSettings {
    Font getDefaultFont() {
        return Font.ARIAL;
    }
}
```

그 다음 기본값과 사용자가 제공한 값중에서 선택하는 논리를 DocumentSettings 클래스안에서 정의할 수 있을 것이다. 상위 수준의 코드에서

사용할 설정을 결정할 때 DocumentSettings 클래스는 간결한 추상화 계층을 제공한다. 기본값과 사용자가 제공한 값에 대한 모든

구현 세부 정보를 숨기지만, 동시에 의존성 주입을 사용하여 이러한 구현 세부 정보를 재설정할 수도 있다. 이를 통해 코드의 재사용성과 적응성이 보장된다

```java
class DocumentSettings {
    private final UserDocumentSettings userSettings;
    private final DefaultDocumentSettings defaultSettings;

    Font getFont() {
        Font userFont = userSettings.getPreferredFont();
        if (userFont != null)
            return userFont;
        return defaultSettings.getFont();
    }
}
```

## 9.4 함수의 매개변수를 주목하라

```java
class TextOptions {
    private final Font font;
    private final Double fontSize;
    private final Double lineHeight;
    private final Color textColor;
}
```

함수가 데이터 객체나 클래스 내에 포함된 모든 정보가 있어야 하는 경우에는 해당 함수나 객체나 클래스의 인스턴스를 매개변수로 받는것이 타당하다

이렇게 하면 함수 매개변수의 수가 줄어들고 캡슐화된 데이터의 자세한 세부 사항을 처리해야 하는 코드가 필요 없다.

그러나 함수가 한두 가지 정보만 필요로 할 때는 객체나 클래스의 인스턴스를 매개변수로 사용하는 것은 코드의 재사용성을 해칠 수 있다

### 9.4.1 필요 이상으로 매개변수를 받는 함수는 재사용하기 어려울 수 있다

`필요 이상의 매개변수를 받는 함수`

```java
class TextBox {
    private final Element textContainer;

    void setTextStyle(TextOptions options) {
        setFont(...);
        setFontSize(...);
        setLineHeight(...);
        setTextColor(options);
    }

    void setTextColor(TextOptions options) {
        textContainer.setStyleProperty("color", options.getTextColor().asHexRgb());
    }
}
```

`너무 많은 매개변수를 받는 함수`
```java
void styleAsWarning(TextBox textBox) {
    TextOptions style = new TextOptions(
        Font.ARIAL,
        12.0,
        14.0,
        Color.RED
    );

    textBox.setTextColor(style);
}
```

TextBox.setTextColor() 함수의 전체 요지는 텍스트 색상만 설정한다는 것이다. 따라서 전체 TextOptions 인스턴스를 매개변수로

사용할 필요가 없다. 그렇게 하는 것은 불필요함을 넘어, 누군가가 그 기능을 조금 다른 상황에 재사용하고자 할 때 해로운 영향을 끼친다.

함수는 필요한 것만 매개변수로 받는 것이 더 바람직하다

### 9.4.2 해결책: 함수는 필요한 것만 매개변수로 받도록 하라.

TextBox.setTextColor() 함수가 TextOptions에서 유일하게 사용하는 것은 텍스트 색상이다. 따라서 함수는 전체 TextOptions

인스턴스를 사용하는 대신 Color 인스턴스를 매개변수로 사용할 수 있다

```java
class TextBox {
    private final Element textElement;

    void setTextStyle(TextOptions options) {
        setFont(...);
        setFontSize(...);
        setLineHeight(...);
        setTextColor(options.getTextColor());
    }
    
    void setTextColor(Color color) {
        textElement.setStyleProperty("color", color.asHexRgb());
    }
}
```

이제 styleAsWarning() 함수는 훨씬 더 간단해지고 혼란도 줄어든다. 관련 없는 값을 일부러 만들어 TextOptions 인스턴스를 생성할 필요가 없다

```java
void styleAsWarning(TextBox textBox){
    textBox.setTextColor(Color.RED);
}
```

## 9.5 제네릭의 사용을 고려하라

### 9.5.1 특정 유형에 의존하면 일반화를 재현한다

단어 맞히기 게임을 개발한다고 가정하자. 게임 참여자들이 각각 단어를 제출한 다음 돌아가면서 한 단어씩 동작으로 설명하면

다른 선수들이 어떤 단어인지 맞혀야 한다. 해결해야 할 하위 문제 중 하나는 단어 모음을 저장하는 것이다. 또한 한 단어씩 무작위로 선택

할 수 있어야 하고, 제한 시간 내에 맞히지 못한다면 그 단어는 다시 단어 모음에 들어가야 한다

문자열 유형을 하드코딩해서 사용하는 예시는 다음과 같다

```java
class RandomizedQueue {
    private final List<String> values = new ArrayList<>();

    void add(String value) {
        values.add(value);
    } 
    /**
     * 큐로부터 무작위로 한 항목을 삭제하고 그 항목을 반환한다
     */
    String getNext() {
        if (values.isEmpty()) {
            return null;
        }

        int randomIndex = Math.randomInt(0, values.size());
        values.swap(randomIndex, values.size() - 1);
        return values.removeLast();
    }
}
```

위 코드는 한 유형의 문제는 해결하지만 다른 유형의 동일한 하위 문제를 해결할 수 있을 만큼 일반화되어 있진 않다. 

### 9.5.2 해결책: 제네릭을 사용하라 

```java
class RandomizedQueue<T> {
    private final List<T> values = new ArrayList<>();
    
    void add(T value) {
        values.add(value);
    }

    /**
     *  큐에서 무작위로 한 항목을 삭제한 후에 그 항목을 반환한다
     */ 
    T getNext() {
        if (values.isEmpty()) {
            return null;
        }
        int randomIndex = Math.randomInt(0, values.size());
        values.swap(randomIndex, values.size() - 1);
        return values.removeLast();
    }
}
```

높은 수준의 문제를 하위 문제로 세분화하다 보면 다양한 사례에 적용할 수 있는 매우 근본적인 문제를 접한다.

하위 문제에 대한 해결책이 모든 데이터 유형에 쉽게 적용될 수 있을때 특정 유형에 의존하는 대신 제네릭을 

사용하더라도 추가적인 노력이 거의 들어가지 않는다. 이렇게 하면 코드는 보다 더 일반화되고 재사용이 가능하다는 측면에서

쉽게 효과를 볼 수 있다

# 9장 요약
- 동일한 하위 문제가 자주 발생하므로 코드를 재사용하면 미래의 자신과 팀 동료의 시간과 노력을 절약할 수 있다
- 다른 개발자가 여러분이 해결하려는 문제와는 다른 상위 수준의 문제를 해결하더라도 특정 하위 문제에 대해서는 여러분이

    작성한 해결책을 재사용할 수 있도록 근본적인 하위 문제를 식별하고 코드를 구성하도록 노력해야 한다
- 간결한 추상화 계층을 만들고 코드를 모듈식으로 만들면 코드를 재사용하고 일반화하기가 훨씬 쉽고 안전해진다
- 가정을 하게 되면 코드는 종종 더 취약해지고 재사용하기 어렵다는 측면에서 비용이 발생한다
  - 가정을 하는 경우의 이점이 비용보다 큰지 확인하라
  - 가정을 해야할 경우 그 가정이 코드의 적절한 계층에 대해 이루어지는 것인지 확인하고 간으하다면 가정을 강제적으로 적용하라
- 전역 상태를 사용하면 특히 비용이 많이 발생하는 가정을 하는 것이 되고 재사용하기에 전혀 안전하지 않은 코드가 된다

    대부분의 경우 전역 상태를 피하는 것이 가장 바람직하다

    