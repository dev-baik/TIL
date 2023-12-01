# 고차 함수: 파라미터와 반환 값으로 람다 사용

> - 함수 타입
> - 고차 함수와 코드를 구조화할 때 고차 함수를 사용하는 방법
> - 인라인 함수
> - 비로컬 return과 레이블
> - 무명 함수

## 고차 함수 정의

> 💡 다른 함수를 인자로 받거나 함수를 반환하는 함수

- 코틀린에서는 람다나 함수 참조를 사용해 함수를 값으로 표현할 수 있다.
  - 고차 함수는 람다나 함수 참조를 인자로 넘길 수 있거나 람다나 함수 참조를 반환하는 함수다.
- 물론 함수를 인자로 받는 동시에 함수를 반환하는 함수도 고차 함수다.


- filter는 술어 함수를 인자로 받으므로 고차 함수다. : `list.filter { x > 0 }`

### 함수 타입
람다를 인자로 받는 함수를 정의하려면 먼저 람다 인자의 타입을 어떻게 선언할 수 있는 지 알아야 한다.

- 람다를 로컬 변수에 대입하는 경우
```kotlin
// 타입 추론으로 인해 변수 타입을 지정하지 않아도 된다.
val sum = { x: Int, y: Int -> x, y }
val action = { println(42) }

// 구체적인 타입 선언을 추가한다면 : 파라미터 타입 -> 반환 타입
val sum: (Int, Int) -> Int = { x, y -> x + y }
val action: () -> Unit = { println(42) }
```
- Unit 타입 : 의미 있는 값을 반환하지 않는 함수 반환 타입에 쓰는 특별한 타입


- 반환 타입을 널이 될 수 있는 타입으로 지정
```kotlin
var canReturnNull: (Int, Int) -> Int? = { x, y -> null } 
```
- 널이 될 수 있는 함수 타입 변수 정의
```kotlin
var funOrNull: ((Int, Int) -> Int)? = null 
```
- funOrNull의 타입을 지정하면서 괄호를 빼먹으면 널이 될 수 있는 함수 타입이 아니라 널이 될 수 있는 반환 타입을 갖는 함수 타입을 선언하게 된다.

> canReturnNull의 타입과 funOrNull의 타입 사이에는 큰 차이가 있다는 사실에 유의하라.

#### 파라미터 이름과 함수 타입
- 함수 타입에서 파라미터 이름을 지정할 수도 있다.
```kotlin
fun performRequest(
    url: String,
    callback: (code: Int, content: String) -> Unit
) { /*...*/ }

val url = "http://kotl.in"
performRequest(url) { code, content -> /*...*/ }
performRequest(url) { code, page -> /*...*/ }
```
- 함수 타입의 람다를 정의할 때 파라미터 이름이 꼭 함수 타입 선언의 파라미터 이름과 일치하지 않아도 된다.
- 하지만 함수 타입에 인자 일므을 추가하면 코드 가독성이 좋아지고, IDE는 그 이름을 코드 완성에 사용할 수 있다.

### 인자로 받은 함수 호출
- 간단한 고차 함수 정의하기
```kotlin
fun twoAndThree(operation: (Int, Int) -> Int) {
    val result = operation(2, 3)
    println("The result is $result")
}

twoAndThree { a, b -> a + b } // The result is 5
twoAndThree { a, b -> a * b } // The result is 6
```
- 술어 함수를 파라미터로 받는 filter 함수 정의하기
```kotlin
fun String.filter(
    predicate: (Char) -> Boolean
): String

fun 수신개체타입.filter(
    // 파라미터 이름: 파라미터 함수 타입
    파라미터 이름: (파라미터로 받는 함수의 파라미터 타입) -> 파라미터로 받는 함수의 반환 타입
): String
```
- predicate 파라미터는 문자(Char 타입)를 파라미터로 받고 불리언 결과 값을 반환한다.
- 술어는 인자로 받은 문자가 filter 함수가 돌려주는 결과 문자열에 남아 있기를 바라면 true를 반환하고 문자열에서 사리지기를 바라면 false를 반환하면 된다.

> ✅ **[술어(명제)](https://ko.wikipedia.org/wiki/%EB%AA%85%EC%A0%9C)** : '참' 혹은 '거짓'임을 검증할 수 있는 '객관적 사태'가 포함된 문장

- filter 함수를 단순하게 만든 버전 구현하기
```kotlin
fun String.filter(predicate: (Char) -> Boolean): String {
    val sb = StringBuilder()
    for (index in 0 until length) {
        val element = get(index)
        if (predicate(element)) sb.append(element)
    }
    return sb.toString()
}

println("ab1c".filter { it in 'a'..'z' }) // abc
```

### 자바에서 코틀린 함수 타입 사용
<span style='background-color: #fff5b1'/>함수 타입인 변수는 인자 개수에 따라 적당한 FunctionN 인터페이스를 구현하는 클래스의 인스턴스를 저장하며, 그 클래스의 invoke 메소드 본문에는 람다의 본문이 들어간다.
- 컴파일된 코드 안에서 함수 타입은 일반 인터페이스로 바뀐다. 즉 함수 타입의 변수는 FunctionN 인터페이스를 구현하는 객체를 저장한다.
- 코틀린 표준 라이브러리는 함수 인자의 개수에 따라 Function0<R>(인자가 없는 함수), Function1<P1, R>(인자가 하나인 함수) 등의 인터페이스를 제공한다.
- 각 인터페이스에는 invoke 메소드 정의가 하나 들어있다. invoke를 호출하면 함수를 실행할 수 있다.
```kotlin
// 코틀린
fun processTheAnswer(f: (Int) -> Int) {
    println(42)
}

// 자바 8 이후
processTheAnswer(number -> number + 1); // 43

// 자바 8 이전
processTheAnswer { 
    new Function1<Integer, Integer>() {
        @Override
        public Integer invoke(Integer number) {
            System.out.println(number);
            return number + 1;
        }
    }
}
```
- 확장 함수
```java
List<String> strings = new ArrayList();
strings.add("42");
CollectionsKt.forEach(strings, s -> {
    System.out.println(s);
    return Unit.INSTANCE;
});
```

### 디폴트 값을 지정한 함수 타입 파라미터나 널이 될 수 있는 함수 타입 파라미터
- 하드 코딩을 통해 toString 사용 관례를 따르는 joinToString

> ✅ **[하드 코딩](https://namu.wiki/w/%ED%95%98%EB%93%9C%EC%BD%94%EB%94%A9)** : 프로그래밍에서 특정 값이나 정보를 소스 코드 내에 직접 입력하여 사용하는 것

```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
): String {
    val result = StringBuilder(prefix)
    
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```
- 이 구현은 유연하지만 핵심 요소 하나를 제어할 수 없다는 단점이 있다.
  - 핵심 요소 : 컬렉션의 각 원소를 문자열로 변환하는 방법
- **WHY)** StringBuilder.append(o: Any?)를 사용하는 데, 이 함수는 항상 객체를 toString 메소드를 통해 문자열로 바꾼다.
- **해결책)** 원소를 문자열로 바꾸는 방법을 람다로 전달하면 된다.
```kotlin
fun main() {
    val stringBuilder = StringBuilder()

    val list = listOf(1, 2, 3, 4, 5)
    
    stringBuilder.append("[")
    list.joinTo(stringBuilder, transform = { element ->
        (element * element).toString()
    })
    stringBuilder.append("]")
    
    println(stringBuilder.toString()) // [1, 4, 9, 16, 25]
}
```
- **문제점)** joinToString를 호출할 때마다 매번 람다를 넘기게 만들면 기본 동작으로도 충분한 대부분의 경우 함수 호출을 오히려 더 불편하게 만든다.
- **해결책)** 함수 타입의 파라미터에 대한 디폴트 값 지정하기
```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
    // 함수 타입 파라미터를 선언하면서 람다를 디폴트 값으로 지정
    // transform: <T> -> String = { it.toString() }
    transform: (T) -> String = { it.toString() }
): String {
    val result = StringBuilder(prefix)

    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(transform(element))
    }
    result.append(postfix)
    return result.toString()
}

val letters = listOf("Alpha", "Beta")
println(letters.joinToString()) // Alpha, Beta
println(letters.joinToString { it.toLowerCase() }) // alpha, beta
println(letters.joinToString(separator = "!", postfix = "! ",
    transform = { it.toUpperCase() })) // ALPHA! BETA
```
- 해당 함수(joinToString)는 컬레션의 원소 타입을 표현하는 T를 타입 파라미터로 받는 제네릭 함수다. transfrom 람다는 그 T 타입의 값을 인자로 받는다.
- 다른 디폴트 파라미터 값과 마찬가지로 함수 타입에 대한 디폴트 값 선언도 `=`뒤에 람다를 넣으면 된다.
```kotlin
// buildString 함수 사용하기: stringBuilder를 직접 다룰 필요 없이 문자열 생성
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
    transform: (T) -> String = { it.toString() }
): String = buildString {
    append(prefix)
    for ((index, element) in this@joinToString.withIndex()) {
        if (index > 0) append(separator)
        append(transform(element))
    }
    append(postfix)
}

fun main() {
    val letters = listOf("Alpha", "Beta")
    println(letters.joinToString()) // Alpha, Beta
    println(letters.joinToString { it.toLowerCase() }) // alpha, beta
    println(letters.joinToString(separator = "!", postfix = "! ") { it.toUpperCase() }) // ALPHA! BETA
} 
```

1. 람다를 아예 생략하거나(람다를 생략하면 디폴트 람다에 있는 대로 toString()을 써서 원소를 변환한다)
2. 인자 목록 뒤에 람다를 넣거나(~~여기서는 람다 밖에 전달할 인자가 없어서 () 없이 람다만 썻다~~)
3. 이름 붙인 인자로 람다를 전달할 수 있다.
4. 널이 될 수 있는 함수 타입 사용할 수도 있다.
- 널이 될 수 있는 함수 타입으로 함수를 받으면 그 함수를 직접 호출할 수 없다는 점에 유의하라.
- **WHY)** 코틀린은 NPE가 발생할 가능성이 있으므로 그런 코드의 컴파일을 거부할 것이다.
- null 여부를 명시적으로 검사하는 것도 한 가지 해결 방법이다.
```kotlin
fun foo(callback: (() -> Unit)?) {
    // ...
    if (callback != null) {
        callback()
    }
} 
```
- 함수 타입이 invoke 메소드를 구현하는 인터페이스라는 사실을 활용하면 더 짧게 만들 수 있다.
```kotlin
// 널이 될 수 있는 함수 타입 파라미터를 사용하기
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
    transform: ((T) -> String)? = null
): String {
    val result = StringBuilder(prefix)

    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        val str = transform?.invoke(element)
            ?: element.toString()
        result.append(str)
    }
    result.append(postfix)
    return result.toString()
}
```

### 함수를 함수에서 반환
```kotlin
// 함수를 반환하는 함수 정의하기
enum class Delivery { STANDARD, EXPEDITED }

class Order(val itemCount: Int)

fun getShippingCostCalculator(delivery: Delivery): (Order) -> Double {
    if (delivery == Delivery.EXPEDITED) {
        return { order -> 6 + 2.1 * order.itemCount }
    }
    return { order -> 1.2 * order.itemCount }
}

val calculator = getShippingCostCalculator(Delivery.EXPEDITED)
println("Shipping costs ${calculator(Order(3))}") // Shipping costs 12.3
```
- 다른 함수를 반환하는 함수를 정의하려면 함수의 반환 타입으로 함수 타입을 지정해야 한다.
- 함수를 반환하려면 return 식에 람다나 멤버 참조나 함수 타입의 값을 계산하는 식 등을 넣으면 된다.


- ContactListFilters 클래스를 사용해 선택 사항의 상태를 저장한다.
```kotlin
class ContactListFilters {
    var prefix: String = ""
    var onlyWithPhoneNumber: Boolean = false
}
```
- 함수를 반환하는 함수를 UI 코드에서 사용하기
```kotlin
data class Person(
    val firstName: String,
    val lastName: String,
    val phoneNumbeer: String?
)

class ContactListFilters {
    var prefix: String = ""
    var onlyWithPhoneNumber: Boolean = false
    
    fun getPredicate(): (Person) -> Boolean {
        val startsWithPrefix = { p: Person ->
            p.firstName.startsWith(prefix) || p.lastName.startsWith(prefix)
        }
        if (!onlyWithPhoneNumber) {
            return startsWithPrefix
        }
        return { startsWithPrefix(it) && it.phoneNumbeer != null }
    }
}

val contacts = listOf(Person("Dmitry", "Jemerov", "123-4567"),
            Person("Svetlana", "Isakova", null))
val contactListFilters = ContactListFilters()
with(contactListFilters) {
    prefix = "Dm"
    onlyWithPhoneNumber = true
}

// 함수 타입을 사용하면 함수에서 함수를 쉽게 반환할 수 있다.
println(contacts.filter(
    contactListFilters.getPredicate()
)) // [Person(firstName=Dmitry, lastName=Jemerov, phoneNumber=123-4567)]
```
- getPredicate 메소드는 filter 함수에게 인자로 넘길 수 있는 함수를 반환한다.

### 람다를 활용한 중복 제거
```kotlin
// 사이트 방문 데이터 정의
data class SiteVisit(
    val path: String,
    val duration: Double,
    val os: OS
)

enum class OS { WINDOWS, LINUX, MAC, IOS, ANDROID }

val log = listOf(
    SiteVisit("/", 34.0, OS.WINDOWS),
    SiteVisit("/", 22.0, OS.MAC),
    SiteVisit("/login", 12.0, OS.WINDOWS),
    SiteVisit("/signup", 8.0, OS.IOS),
    SiteVisit("/", 16.3.0, OS.ANDROID),
)

// 사이트 방문 데이터를 하드 코딩한 필터를 사용해 분석하기
val averageWindowsDuration = log
    .filter { it.os == OS.WINDOWS }
    .map(SiteVisit::duration)
    .average()

// 윈도우 사용자의 평균 방문 시간 출력
println(averageWindowsDuration) // 23.0
```
- 일반 함수를 통해 중복 제거하기
```kotlin
fun List<SiteVisit>.averageDurationFor(os: OS) =
    filter { it.os == os }.map(SiteVisit::duration).average()

println(log.averageDurationFor(OS.WINDOWS)) // 23.0
println(log.averageDurationFor(OS.MAC)) // 22.0 
```
- 복잡하게 하드코딩한 필터를 사용해 방문 데이터 분석하기
```kotlin
val averageMobileDuration = log
    .filter { it.os in setOf(OS.IOS, OS.ANDROID) }
    .map(SiteVisit::duration)
    .average()

println(averageMobileDuration) // 12.15
```
- 고차 함수를 사용해 중복 제거하기
```kotlin
fun List<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean) =
    filter(predicate).map(SiteVisit::duration).average()

println(log.averageDurationFor {
    it.os in setOf(OS.ANDROID, OS.IOS)
}) // 12.15


// ios 사용자의 /signup 페이지 평균 방문 시간
println(log.averageDurationFor {
    it.os == OS.IOS && it.path == "/signup"
}) // 8.0
```