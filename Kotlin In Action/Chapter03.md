# 함수 정의와 호출
> - 컬렉션, 문자열, 정규식을 다루기 위한 함수
> - 이름 붙인 인자, 디폴트 파라미터 값, 중위 호출 문법 사용
> - 확장 함수와 확장 프로퍼티를 사용해 자바 라이브러리 적용
> - 최상위 및 로컬 함수와 프로퍼티를 사용해 코드 구조화

- - -

## 코틀린에서 컬렉션 만들기
```kotlin
val set = hashSetOf(1, 7, 53)

val list = arrayListOf(1, 7, 53)

val map = hashMapOf(1 to "one", 7 to "seven", 53 to "fifty-threee")

println(set.javaClass) // class java.util.HashSet
println(list.javaClass) // class java.util.ArrayList
println(map.javaClass) // class java.util.HashMap
```
- 코틀린 컬렉션은 자바 컬렉션과 똑같은 클래스다. 이는 코틀린이 자신만의 컬렉션 기능을 제공하지 않는다는 뜻이다. 표준 자바 컬렉션을 활용하면 자바 코드와 상호작용하기가 훨씬 더 쉽다.
- 하지만 코틀린에서는 자바보다 더 많은 기능을 쓸 수 있다.
```kotlin
val strings = listOf("first", "second", "fourteenth")
println(strings.last()) // fourteenth

val numbers = setOf(1, 14, 2)
println(numbers.max()) // 14
```

## 함수를 호출하기 쉽게 만들기
- 자바 컬렉션에는 디폴트 toString 구현이 들어있다. 하지만 그 디폴트 toString의 출력 형식은 고정돼 있고 우리에게 필요한 형식이 아닐 수도 있다.
```kotlin
val list = listOf(1, 2, 3)
println(list) // [1, 2, 3]
```
```kotlin
// joitnToString() 함수의 초기 구현
fun <T> joinToString(
    collection: Collection<T>,
    separator: string,
    prefix: String,
    postfix: String
): String {
    val result = StringBuilder(prefix)
    
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }
    
    result.append(postfix)
    return result.toString()
}
```
- joinToString 함수는 컬렉션의 원소를 StringBuilder의 뒤에 덧붙인다. 이때 원소 사이에 구분자(separator)를 추가하고, StringBuilder의 맨 앞과 맨 뒤에는 접두사(prefix)와 접미사(postfix)를 추가한다.
- 이 함수는 제네릭(generic)하다. 즉, 어떤 타입의 값을 원소로 하는 컬렉션이든 처리할 수 있다.
```kotlin
val list = listOf(1, 2, 3)
println(joinToString(list, "; ", "(", ")")) // (1; 2; 3)
```

### 이름 붙은 인자
```kotlin
joinToString(collection, separator = " ", prefix = " ", postfix = ".")
```
- 코틀린으로 작성한 함수를 호출할 때는 함수에 전달하는 인자 중 일부(또는 전부)의 이름을 명시할 수 있다. 호출 시 인자 중 어느 하나라도 이름을 명시하고 나면 혼동을 막기 위해 그 뒤에 오는 모든 인자는 이름을 꼭 명시해야 한다.

### 디폴트 파라미터 값
- 자바에서는 일부 클래스에서 `오버로딩(overloading)`한 메서드가 너무 많아진다는 문제가 있다. java.lang.Thread에 8가지 생성자 메소드들은 하위 호환성을 유지하거나 API 사용자에게 편의를 더하는 등의 여러 가지 이유로 만들어진다. 하지만 어느 경우든 중복이라는 결과는 같다.
- 오버로딩 함수에 대해 대부분의 설명을 반복해 달아야하고, 인자 중 일부가 생략된 오버로드 함수를 호출할 때 어떤 함수가 불릴지 모호한 경우가 생긴다.


- 코틀린에서는 함수 선언에서 파라미터의 디폴트 값을 지정할 수 있으므로 이런 오버로드 중 상당수를 피할 수 있다.
```kotlin
// 디폴트 파라미터 값을 사용해 joinToString() 정의하기
fun <T> joinToString(
    collection: Collection<T>,
    separator: string = ", ",
    prefix: String = "",
    postfix: String = ""
): String

joinToString(list, ", ", "", "") // 1, 2, 3
joinToString(list) // 1, 2, 3
joinToString(list, "; ") // 1; 2; 3
```
- 일반 호출 문법을 사용하려면 함수를 선언할 때와 같은 순서로 인자를 지정해야 한다. 그런 경우 일부를 생략하면 뒷부분의 인자들이 생략된다.
- 이름 붙인 인자를 사용하는 경우에는 인자 목록의 중간에 있는 인자를 생략하고, 지정하고 싶은 인자를 이름을 붙여서 순서와 관계없이 지정할 수 있다.
```kotlin
joinToString(list, postfix = ";", prefix = "# ") // # 1, 2, 3; 
```
> 함수의 디폴트 파라미터 값은 함수를 호출하는 쪽이 아니라 함수 선언 쪽에서 지정된다는 사실을 기억하라. 따라서 어떤 클래스 안에 정의된 함수의 디폴트 값을 바꾸고 그 클래스가 포함된 파일을 재컴파일하면 그 함수를 호출하는 코드 중에 값을 지정하지 않은 모든 인자는 자동으로 바뀐 디폴트 값을 적용받는다.

### 정적인 유틸리티 클래스 없애기: 최상위 함수와 프로퍼티
- 객체지향 언어인 자바에서는 모든 코드를 클래스의 메소드를 작성해야 한다. 보통 그런 구조는 잘 작동하지만 실전에서는 어느 한 클래스에 포함시키기 어려운 코드가 많이 생긴다.
- 일부 연산에는 비슷하게 중요한 역할을 하는 클래스가 둘 이상 있을 수도 있다. 중요한 객체는 하나뿐이지만 그 연산을 객체의 인스턴스 API에 추가해서 API를 너무 크게 만들고 싶지는 않은 경우도 있다.
  - 그 결과 다양한 정적 메서드를 모아두는 역할만 담당하며, 특별한 상태나 인스턴스 메소드는 없는 클래스가 생겨난다.


- 코틀린에서는 이런 무의미한 클래스가 필요없다. 대신 함수를 직접 소스 파일의 최상위 수준, 모든 다른 클래스의 밖에 위치시키면 된다.
- 그런 함수들은 여전히 그 파일의 맨 앞에 정의된 패키지의 멤버 함수이므로 다른 패키지에서 그 함수를 사용하고 싶을 때는 그 함수가 정의된 패키지를 임포트해야 한다. 하지만 임포트 시 유틸리티 클래스 이름이 추가로 들어갈 필요는 없다.
```kotlin
// joinToString() 함수를 최상위 함수로 선언하기
package strings

fun joinToString(...): String { ... }
```
- JVM이 클래스 안에 들어있는 코드만을 실행할 수 있기 때문에 컴파일러는 이 파일을 컴파일할 때 새로운 클래스를 정의해준다.

#### 최상위 프로퍼티
- 함수와 마찬가지로 프로퍼티도 파일의 최상위 수준에 놓을 수 있다.
```kotlin
var opCount = 0
fun performOperation() {
    opCount++
    // ...
}

fun reportOperationCount() {
    println("Operation performed $opCount times")
}
```
- 이런 프로퍼티의 값은 정적 필드에 저장된다.
- 최상위 프로퍼티를 활용해 코드에 상수를 추가할 수 있다.
```kotlin
val UNIX_LINE_SEPARATOR = "\n" 
```
- 기본적으로 최상위 프로퍼티도 다른 모든 프로퍼티처럼 접근자 메소드를 통해 자바 코드에 노출된다(val의 경우 게터, var의 경우 게터와 세터가 생긴다).
- 겉으론 상수처럼 보이는데, 실제로는 게터를 사용해야 한다면 자연스럽지 못하다.
- 더 자연스럽게 사용하려면 이 상수를 public static final 필드로 컴파일해야 한다.
  - const 변경자를 추가하면 프로퍼티를 public static final 필드로 컴파일하게 만들 수 있다(단, 원시 타입과 String 타입의 프로퍼티만 const로 지정할 수 있다).
```kotlin
const val UNIX_LINE_SEPARATOR = "\n"
```
앞의 코드는 다음 자바 코드와 동등한 바이트코드를 만들어낸다.
```java
public static final String UNIX_LINE_SEPARATOR = "\n";
```

## 메소드를 다른 클래스에 추가: 확장 함수와 확장 프로퍼티
- 확장함수는 어떤 클래스의 멤버 메소드인 것처럼 호출할 수 있지만 그 클래스의 밖에 선언된 함수다.
```kotlin
package strings

fun String.lastChar(): Char = this.get(this.length - 1)

println("Kotlin".lastChar()) // n
```
- 확장 함수를 만들려면 추가하려는 함수 이름 앞에 그 함수가 확장한 클래스의 이름을 덧붙이기만 하면 된다.
- 클래스의 이름을 `수신 객체 타입(receiver type)`이라 부르며, 확장 함수가 호출되는 대상이 되는 값(객체)을 `수신 객체(receiver object)`라고 부른다.
  - 이 예제에서는 String이 수신 객체 타입이고 "kotlin"이 수신 객체다.
- 일반 메소드와 마찬가지로 확장 함수 본문에서도 this를 생략할 수 있다.
```kotlin
package strings

fun String.lastChar(): Char = get(length - 1) 
```
- 확장 함수 내부에서는 일반적인 인스턴스 메소드의 내부에서와 마찬가지로 수신 객체의 메소드나 프로퍼티를 바로 사용할 수 있다. 하지만 확장 함수가 캡슐화를 깨지는 않는다.
- 클래스 안에서 정의한 메서드와 달리 확장 함수 안에서는 클래스 내부에서만 사용할 수 있는 비공개(private) 멤버나 보호된(protected) 멤버를 사용할 수 없다.

### 임포트와 확장 함수
- 확장 함수를 사용하기 위해서는 그 함수를 다른 클래스나 함수와 마찬가지로 임포트해야만 한다.
```kotlin
import strings.lastChar

val c = "Kotlin".lastChar()
```
```kotlin
import strings.*

val c = "Kotlin".lastChar()
```
- as 키워드를 사용하면 임포트한 클래스나 함수를 다른 이름으로 부를 수 있다.
```kotlin
import strings.lastChar as last

val c = "Kotlin".last() 
```
- 한 파일 안에서 다른 여러 패키지에 속해있는 이름이 같은 함수를 가져와 사용해야 하는 경우 이름을 바꿔서 임포트하면 이름 충돌을 막을 수 있다.
- 물론 일반적인 클래스나 함수라면 그 전체 이름을 써도 된다. 하지만 코틀린 문법상 확장 함수는 반드시 짧은 이름을 써야 한다. 따라서 임포트할 때 이름을 바꾸는 것이 확장 함수 이름 충돌을 해결할 수 있는 유일한 방법이다.

### 자바에서 확장 함수 호출
- 내부적으로 확장 함수는 수신 객체를 첫 번째 인자로 받는 정적 메소드다. 그래서 확장 함수를 호출해도 다른 어댑터 객체나 실행 시점 부가 비용이 들지 않는다.

### 확장 함수로 유틸리티 함수 정의
```kotlin
fun <T> joinToString(
    collection: Collection<T>,
    separator: string = ", ",
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

val list = listOf(1, 2, 3)
println(list.joinToString(separator = "; ", prefix = "(", postfix = ")")) // (1; 2; 3)
```
- joinToString을 마치 클래스의 멤버인 것처럼 호출할 수 있다.
- 확장 함수는 단지 정적 메소드 호출에 대한 문법적인 편의일 뿐이다. 그래서 클래스가 아닌 더 구체적인 타입을 수신 객체 타입으로 지정할 수도 있다.
```kotlin
// 문자열 컬렉션에 대해서만 호출할 수 있는 join 함수 정의
fun Collection<String>.join(
    separator: string = ", ",
    prefix: String = "",
    postfix: String = ""
) = joinToString(separator, prefix, postfix)

println(listOf("one", "two", "eight").join(" ")) // one two eight
```

### 확장 함수는 오버라이드할 수 없다.
```kotlin
// 맴버 함수 오버라이드하기
open class View {
    open fun click() = println("View clicked")
}

class Button: View() {
    override fun click() = println("Button clicked")
}

val view: View = Button()
view.click() // Button clicked
```
> 확장 함수는 클래스의 일부가 아니다. 확장 함수는 클래스 밖에 선언된다.

> 이름과 파라미터가 완전히 같은 확장 함수를 기반 클래스와 하위 클래스에 대해 정의해도 실제로는 확장 함수를 호출할 때 수신 객체로 지정한 변수의 정적 타입에 의해 어떤 확장 함수가 호출될지 결정되지, 그 변수에 저장된 객체의 동적인 타입에 의해 확장 함수가 결정되지 않는다.

```kotlin
// 확장 함수는 오버라이드할 수 없다.
fun View.showOff() = println("I'm a view!")

fun Button.showOff() = println("I'm a button!")

val view: View = Button()
view.showOff() // I'm a view!
```
> 어떤 클래스를 확장한 함수와 그 클래스의 멤버 함수의 이름과 시그니처가 같다면 확장 함수가 아니라 멤버 함수가 호출된다(멤버 함수의 우선 운위가 더 높다).

### 확장 프로퍼티
- 확장 프로퍼티를 사용하면 기존 클래스 객체에 대한 프로퍼티 형식의 구문으로 사용할 수 있는 API를 추가할 수 있다.
- 프로퍼티라는 이름으로 불리기는 하지만 상태를 저장할 적적한 방법이 없기 때문에(기존 클래스의 인스턴스 객체에 필드를 추가할 방법은 없다) 실제로 확장 프로퍼티는 아무 상태도 가질 수 없다.
- 하지만 프로퍼티 묹법으로 더 짧게 코드를 작성할 수 있어서 편한 경우가 있다.
```kotlin
// 확장 프로퍼티 선언하기
val String.lastChar: Char
  get() = get(length - 1)
```
- 확장 함수의 경우와 마찬가지로 확장 프로퍼티도 일반적인 프로퍼티와 같은데, 단지 수신 객체 클래스가 추가됐을 뿐이다.
- 뒷받침하는 필드가 없어서 기본 게터 구현을 제공할 수 없으므로 최소한 게터는 꼭 정의해야 한다. 마찬가지로 초기화 코드에서 계산한 값을 담을 장소가 전혀 없으므로 초기화 코드도 쓸 수 없다.
```kotlin
var StringBuilder.lastChar: Char
  get() = get(length - 1)
  set(value: Char) {
      this.setCharAt(length - 1, value)
  }

println("Kotlin".lastChar) // n

val sb = StringBuilder("Kotlin?")
sb.lastChar = '!'
println(sb) // Kotlin!
```