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


- filter는 술어 함수를 인자로 받으므로 고차 함수다 : `list.filter { x > 0 }`

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
- 하지만 함수 타입에 인자 이름을 추가하면 코드 가독성이 좋아지고, IDE는 그 이름을 코드 완성에 사용할 수 있다.

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

fun 수신객체타입.filter(
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
- 해당 함수(joinToString)는 컬렉션의 원소 타입을 표현하는 T를 타입 파라미터로 받는 제네릭 함수다. transfrom 람다는 그 T 타입의 값을 인자로 받는다.
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
2. 인자 목록 뒤에 람다를 넣거나(여기서는 람다 밖에 전달할 인자가 없어서 () 없이 람다만 썻다)
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

## [인라인 함수: 람다의 부가 비용 없애기](https://velog.io/@dev-baik/inline-%ED%95%A8%EC%88%98)

- 어떤 함수를 inline으로 선언하면 그 함수의 본문이 인라인된다.
  - 함수를 호출하는 코드를 함수를 호출하는 바이트코드 대신에 함수 본문을 번역한 바이트 코드로 컴파일한다.
```kotlin
inline fun <T> synchronzied(lock: Lock, action: () -> T): T {
    lock.lock()
    try {
        return action()
    }
    finally {
        lock.unlock()
    }
}

val l = Lock()
synchronzied(l) {
    ...
}
```
```kotlin
fun foo(l: Lock) {
    println("Before sync")
    synchronized(l) {
        println("Action")
    }
    println("After sync")
} 
```
- foo 함수를 컴파일한 버전
```kotlin
fun __foo__(l: Lock) {
    // synchronized 함수를 호출하는 foo 함수의 코드 1
    println("Before sync")
    // synchronized 함수가 인라이닝된 코드 1
    l.lock()
    try {
        // 람다 코드의 본문이 인라이닝된 코드
        println("Action")
    // synchronized 함수가 인라이닝된 코드 2
    } finally {
        l.unlock()
    }
    // synchronized 함수를 호출하는 foo 함수의 코드 2
    println("After sync")
} 
```

> synchronized 함수의 본문뿐 아니라 synchronized에 전달된 람다의 본문도 함께 인라이닝된다는 점에 유의하라.

- 람다의 본문에 의해 만들어지는 바이트코드는 그 람다를 호출하는 코드(synchronized) 정의의 일부분으로 간주되기 때문에 코틀린 컴파일러는 그 람다를 함수 인터페이스를 구현하는 무명 클래스로 감싸지 않는다.


- <span style='background-color: #fff5b1'/>인라인 함수를 호출하면서 람다를 넘기는 대신에 함수 타입의 변수를 넘기는 경우 인라인 함수를 호출하는 코드 위치에서는 변수에 저장된 람다의 코드를 알 수 없다.
```kotlin
class LockOwner(val lock: Lock) {
    fun runUnderLock(body: () -> Unit) {
        // 람다 대신에 함수 타입인 변수를 인자로 넘긴다.
        synchronized(lock, body)
    }
} 
```
- <span style='background-color: #fff5b1'/>람다 본문은 인라이닝되지 않고 synchronized 함수의 본문만 인라이닝된다.
  - 람다는 다른 일반적인 경우와 마찬가지로 호출된다.
- runUnderLock을 컴파일한 바이트코드
```kotlin
class LockOwner(val lock: Lock) {
    fun __runUnderLock__(body: () -> Unit) {
        lock.lock()
        try {
            // synchronized를 호출하는 부분에서 람다를 알 수 없으므로
            // 본문(body())은 인라이닝되지 않는다.
            body()
        } finally {
            lock.unlock()
        }
    }
}
```
- 한 인라인 함수를 두 곳에서 다른 람다를 사용해 호출한다면 그 두 호출은 각각 따로 인라이닝된다.
- 인라인 함수의 본문 코드가 호출 지점에 복사되고 각 람다의 본문이 인라인 함수의 본문 코드에서 람다를 사용하는 위치에 복사된다.

### 인라인 함수의 한계
- <span style='background-color: #fff5b1'/>함수가 인라이닝될 때 그 함수에 인자로 전달된 람다 식의 본문은 결과 코드에 직접 들어갈 수 있다.
- 하지만 이렇게 람다가 본문에 직접 펼쳐지기 때문에 함수가 파라미터로 전달받은 람다를 본문에 사용하는 방식이 한정될 수밖에 없어 람다를 사용하는 모든 함수를 인라이닝할 수는 없다.


- 파라미터로 받은 람다를 다른 변수에 저장하고 나중에 그 변수를 사용한다면 람다를 표현하는 객체가 어딘가는 존재해야 하기 때문에 람다를 인라이닝할 수 없다.
- 일반적으로 인라인 함수의 본문에서 람다 식을 바로 호출하거나 람다 식을 인자로 전달받아 바로 호출하는 경우에는 그 람다를 인라이닝할 수 있다. 
- 그 외에는 "Illegal usage of inline-parameter" 오류와 함께 인라이닝을 금지시킨다.

```kotlin
fun <T, R> Sequence<T>.map(transform: (T) -> R): Sequence<R> {
    return TransformingSequence(this, transform)
}
```
- Transform 파라미터로 전달받은 함수 값을 호출하지 않는 대신, TransformingSequence라는 클래스의 생성자에게 그 함수 값을 넘긴다.
  - TransformingSequence 생성자는 전달받은 람다를 프로퍼티로 저장한다.
- 이런 기능을 지원하려면 map에 전달되는 transform 인자를 일반적인(인라이닝하지 않은) 함수 표현으로 만들 수밖에 없다.
  - 즉, 여기서는 transform을 함수 인터페이스를 구현하는 무명 클래스 인스턴스로 만들어야만 한다. 인라이닝하면 안되는 람다를 파라미터로 받아, nonline 변경자를 이름 앞에 붙여서 인라이닝을 금지할 수 있다.


- 둘 이상의 람다를 인자로 받는 함수에서 일부 람다만 인라이닝하고 싶은 경우
```kotlin
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) {
    // ...
}                                                                                        
```
- 코틀린에서는 어떤 모듈이나 서드파티 라이브러리 안에서 인라인 함수를 정의하고 그 모듈이나 라이브러리 밖에서 해당 인라인 함수를 사용할 수 있다.

### 컬렉션 연산 인라이닝
- 람다를 사용해 컬렉션 거르기
```kotlin
data class Person(val name: String, val age: Int)

val people = listOf(Person("Alice", 29), Person("Bob", 31))

println(people.filter { it.age < 30 }) // [Person(name=Alice, age=29)]
```
- 컬렉션 직접 거르기
```kotlin
val result = mutableListOf<Person>()
for (person in people) {
    if (person.age < 30) result.add(person)
}
println(result) // [Person(name=Alice, age=29)]
```
- 코틀린의 filter 함수는 인라인 함수다. 따라서 filter 함수의 바이트코드는 그 함수에 전달된 람다 본문의 바이트코드와 함께 filter를 호출한 위치에 들어간다.


- filter와 map을 연쇄해서 사용하기
```kotlin
println(people.filter { it.age > 30 }
              .map(Person::name)) // [Bob] 
```
- 이 예제는 람다 식과 멤버 참조를 사용한다. filter와 map은 인라인 함수다. 따라서 그 두 함수의 본문은 인라이닝되며, 추가 객체나 클래스 생성은 없다.
- **문제점** : filter 함수에서 만들어진 코드는 원소를 그 중간 리스트에 추가하고, map 함수에서 만들어진 코드는 그 중간 리스트를 읽어서 사용한다.
- **해결책** : asSequence를 통해 리스트 대신 시퀀스를 사용하면 중간 리스트로 인한 부가 비용은 줄어든다.


- 각 중간 시퀀스는 람다를 필드에 저장하는 객체로 표현되며, 최종 연산은 중간 시퀀스에 있는 여러 람다를 연쇄 호출한다.
- 따라서 시퀀스는 람다를 저장해야 하므로 람다를 인라인하지 않는다.

> 지연 계산을 통해 성능을 향상시키려는 이유로 모든 컬렉션 연산에 asSequence를 붙여서는 안 된다.

- 시퀀스 연산에서는 람다가 인라이닝되지 않기 때문에 크기가 작은 컬렉션은 오히려 일반 컬렉션 연산이 더 성능이 나을 수도 있다.
- <span style='background-color: #fff5b1'/>시퀀스를 통해 성능을 향상시킬 수 있는 경우는 컬렉션 크기가 큰 경우뿐이다.

### 함수를 인라인으로 선언해야 하는 경우

> inline 키워드는 람다를 인자로 받는 함수만 성능이 좋아질 가능성이 높다. 다른 경우에는 주의 깊게 성능을 측정하고 조사해봐야 한다.

- **WHY)** 일반 함수 호출의 경우 JVM은 이미 강력하게 인라이닝을 지원한다. JVM은 코드 실행을 분석해서 가장 이익이 되는 방향으로 호출을 인라이닝한다.
- 이런 과정은 바이트코드를 실제 기계어 코드로 번역하는 과정(JIT)에서 일어난다.
- 이런 JVM의 최적화를 활용한다면 바이트코드에서는 각 함수 구현이 정확히 한 번만 있으면 되고, 그 함수를 호출하는 부분에서 따로 함수 코드를 중복할 필요가 없다.
- <span style='background-color: #fff5b1'/>반면 코틀린 인라인 함수는 바이트코드에서 각 함수 호출 지점을 함수 본문으로 대치하기 때문에 코드 중복이 생긴다.
- 게다가 함수를 직접 호출하면 [스택 트레이스](https://ccomccomhan.tistory.com/190)가 더 깔끔해진다.

> ✅ **스택 트레이스** : 프로그램의 실행 과정에서 호출된 메서드들의 순서와 위치 정보를 나타내는 것

#### 반면 람다를 인자로 받는 함수를 인라이닝하면 이익이 더 많다.
1. 인라이닝을 통해 없앨 수 있는 부가 비용이 상당하다.
   - 함수 호출 비용을 줄일 수 있을 뿐 아니라 람다를 표현하는 클래스와 람다 인스턴스에 해당하는 객체를 만들 필요도 없어진다.
2. 현재의 JVM은 함수 호출과 람다를 인라이닝해 줄 정도로 똑똑하지는 못하다.
3. 인라이닝을 사용하면 일반 람다에서는 사용할 수 없는 몇 가지 기능을 사용할 수 있다. (non-local 반환)

#### 하지만 inline 변경자를 함수에 붙일 때는 코드 크기에 주의를 기울여야 한다.
- 인라이닝하는 함수가 큰 경우 함수의 본문에 해당하는 바이트코드를 모든 호출 지점에 복사해 넣고 나면 바이트코드가 전체적으로 아주 커질 수 있다.
  - **HOW)** 람다 인자와 무관한 코드를 별도의 비인라인 함수로 빼낼 수도 있다.
- 코틀린 표준 라이브러리가 제공하는 inline 함수를 보면 모두 크기가 아주 작다는 사실을 알 수 있다.

### 자원 관리를 위해 인라인된 람다 사용

> 💡 **자원 관리** : 람다로 중복을 없앨 수 있는 패턴으로, 어떤 작업을 하기 전에 자원을 획득하고 작업을 마친 후 자원을 해제하는 패턴

> ✅ **자원** : 컴퓨팅 환경에서 사용되는 파일, 락, 데이터베이스 트랙잭션 등 모든 유한한 시스템 요소들

- 자원 관리 패턴을 만들 때 보통 try/finally문을 사용하되 try 블록을 시작하기 직전에 자원을 획득하고 finally 블록에서 자원을 해제하는 방법을 사용한다.
- 락 객체를 인자로 취하는 자바의 synchronized 함수와 같은 기능을 제공하는 코틀린 라이브러리 withLock 함수가 있다.
```kotlin
val l: Lock = ...
// 락을 잠근 다음에 주어진 동작을 수행한다.
l.withLock {
    // 락에 의해 보호되는 자원을 사용한다.
}
```
- withLock 함수 정의
```kotlin
// 락을 획득한 후 작업하는 과정을 별도의 함수로 분리한다.
fun<T> Lock.withLock(action: () -> T): T {
    lock()
    try {
        return action()
    } finally {
        unlock()
    }
} 
```
- try-with-resource : 파일의 각 줄을 읽는 자바 메소드
```java
static String readFirstLineFromFile(String path) throws IOException {
    try (BufferedReader br =
            new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```
- 코틀린에서는 함수 타입의 값을 파라미터로 받는 함수(람다를 인자로 받는 함수)를 통해 매끄럽게 이를 처리할 수 있으므로, try-with-resource와 같은 기능을 언어 구성 요소로 제공하지 않는다.
- 대신 자바 try-with-resource와 같은 기능을 제공하는 `use 함수`를 자원 관리에 활용할 수 있다.
```kotlin
fun readFirstLineFromFile(path: String): String {
    // BufferedReader 객체를 만들고 "use" 함수를 호출하면서 파일에 대한 연산을 실행할 람다를 넘긴다.
    BufferedReader(FileReader(path)).use { br -> 
        // 자원(파일)에서 맨 처음 가져온 한 줄을 람다가 아닌 readFirstLineFromFile에서 반환한다.
        return br.readLine()
    }
}
```
- use 함수는 닫을 수 있는(closeable) 자원에 대한 확장 함수며, 람다를 인자로 받는다.

> ✅ **닫을 수 있는 자원?** : 시스템 리소스를 사용한 후에 반드시 해제해야 하는 객체로, 주로 I/O 관련 작업에 사용되는 파일, 네트워크 연결, 데이터베이스 연결 등이 있다.

- use는 람다를 호출한 다음에 자원을 닫아준다. 이때 람다가 정상 종료한 경우는 물론 람다 안에서 예외가 발생한 경우에도 자원을 확실히 닫는다. 물론 use 함수도 인라인 함수다.

## 고차 함수 안에서 흐름 제어
### 람다 안의 return문: 람다를 둘러싼 함수로부터 반환
- 일반 루프 안에서 return 사용하기
```kotlin
data class Person(val name: String, val age: Int)

val people = listOf(Person("Alice", 29), Person("Bob", 31))

fun lookForAlice(people: List<Person>) {
    for (person in people) {
        if (person.name == "Alice") {
            println("Found!")
            return
        }
    }
    // "people" 안에 엘리스가 없다면 이 줄이 출력된다.
    println("Alice is not found")
}

lookForAlice(people) // Found!
```
- forEach에 전달된 람다에서 return 사용하기
```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name == "Alice") {
            println("Found!")
            return
        }
    }
    println("Alice is not found")
} 
```
- 람다 안에서 return을 사용하면 람다로부터만 반환되는게 아니라 그 람다를 호출하는 함수가 실행을 끝내고 반환한다.
- 이렇게 자신을 둘러싸고 있는 블록보다 더 바깥에 있는 다른 블록을 반환하게 만드는 return문을 `넌로컬(non-local) return`이라 부른다.
- <span style='background-color: #fff5b1'/>return이 바깥쪽 함수를 반환시킬 수 있는 때는 람다를 인자로 받는 함수가 인라인 함수인 경우뿐이다.
- forEach는 인라인 함수이므로 람다 본문과 함께 인라이닝된다. 따라서 return 식이 바깥쪽 함수(lookForAlice)를 반환시키도록 쉽게 컴파일할 수 있다.
- 인라이닝되지 않는 함수는 람다를 변수에 저장할 수 있고, 바깥쪽 함수로부터 반환된 뒤에 저장해 둔 람다가 호출될 수도 있다.

### 람다로부터 반환: 레이블을 사용한 return
- 로컬 return은 람다의 실행을 끝내고 람다를 호출했던 코드의 실행을 계속 이어간다.
- 로컬 return과 넌로컬 return을 구분하기 위해 레이블을 사용해야 한다. return으로 실행을 끝내고 싶은 람다 식 앞에 레이블을 붙이고, return 키워드 뒤에 그 레이블을 추가하면 된다.
```kotlin
// 레이블을 통해 로컬 리턴 사용하기
fun lookForAlice(people: List<Person>) {
    // 람다에 레이블을 붙이거나 return 뒤에 레이블을 붙이기 위해 @ 사용하기
    people.forEach label@{
        // return@label은 앞에서 정의한 레이블을 참조한다.
        if (it.name == "Alice") return@label
    }
    println("Alice might be somewhere")
}

lookForAlice(people) // Alice might be somewhere
```
- 람다에 레이블을 붙여서 사용하는 대신 람다를 인자로 받는 인라인 함수의 이름을 return 뒤에 레이블로 사용해도 된다.
```kotlin
// 함수 이름을 return 레이블로 사용하기
fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name == "Alice") return@forEach
    }
    println("Alice might be somewhere")
} 
```

> 람다 식의 레이블을 명시하면 함수 이름을 레이블로 사용할 수 없다는점에 유의하라. 람다식에는 레이블이 2개 이상 붙을 수 없다.

#### 레이블이 붙은 this 식
- 수신 객체 지정 람다의 본문에서는 this 참조를 사용해 묵시적인 컨텍스트 객체를 가리킬 수 있다.
- 수신 객체 지정 람다 앞에 레이블을 붙인 경우 this 뒤에 그 레이블을 붙여서 묵시적인 컨텍스트 객체를 지정할 수 있다.
```kotlin
println(StringBuilder().apply sb@ {
    listOf(1, 2, 3).apply {
        this@sb.append(this.toString())
    }
}) // [1, 2, 3]
```

- 하지만 넌로컬 반환문은 장황하고, 람다 안의 여러 위치에 return 식이 들어가야 하는 경우 사용하기 불편하다.

### 무명 함수: 기본적으로 로컬 return
- 무명 함수 안에서 return 사용하기
```kotlin
fun lookForAlice(people: List<Person>) {
    // 람다 식 대신 무명 함수를 사용한다.
    people.forEach(fun(person) {
        // "return"은 가장 가까운 함수를 가리키는데 이 위치에서 가장 가까운 함수는 무명 함수다.
        if (person.name == "Alice") return
        println("${person.name} is not Alice")
    })
}

lookForAlice(people) // Bob is not Alice
```
- 무명 함수도 일반 함수와 같은 반환 타입 지정 규칙을 따른다.
```kotlin
// filter에 무명 함수 넘기기
people.filter(fun (person) : Boolean) {
    return person.age < 30
}
```
- 블록이 본문인 무명 함수는 반환 타입을 명시해야 하지만, 식을 본문으로 하는 무명 함수의 반환 타입은 생략할 수 있다.
```kotlin
people.filter(fun (person) = person.age < 30) 
```
- 무명 함수 안에서 레이블이 붙지 않은 return 식은 무명 함수 자체를 반환시킬 뿐 무명 함수를 둘러싼 다른 함수를 반환시키지 않는다.

#### 정리
- return은 fun 키워드를 사용해 정의된 가장 안쪽 함수를 반환시킨다.
- 람다식은 fun을 사용해 정의되지 않으므로 람다 본문의 return은 람다 밖에 함수를 반환시킨다.
- 무명 함수는 fun을 사용해 정의되므로 그 함수 자신이 바로 가장 안쪽에 있는 fun으로 정의된 함수다. 따라서 무명 함수 본문의 return은 그 무명 함수를 반환시키고, 무명 함수 밖의 다른 함수를 반환시키지 못한다.