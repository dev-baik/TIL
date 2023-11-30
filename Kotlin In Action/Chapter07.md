# 연산자 오버로딩과 기타 관례

> - 연산자 오버로딩
> - 관례: 여러 연산을 지원하기 위해 특별한 이름이 붙은 메소드
> - 위임 프로퍼티

- - -

## 산술 연산자 오버로딩
### 이항 산술 연산 오버로딩
```kotlin
// plus 연산자 구현하기
data class Point(val x: Int, val y: Int) {
    operator fun plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }
}

val p1 = Point(10, 20)
val p2 = Point(30, 40)
println(p1 + p2) // Point(x=40, y=60)
```
- 연산자를 오버로딩하는 함수 앞에는 꼭 operator가 있어야 한다.
- <span style='background-color: #fff5b1'/>operator 키워드를 붙임으로써 어떤 함수가 관례를 따르는 함수임을 명확히 할 수 있다.
```kotlin
// 연산자를 확장 함수로 정의하기
operator fun Point.plus(other: Point): Point {
    return Point(x + other.x, y + other.y)
}
```

> 외부 함수의 클래스에 대한 연산자를 정의할 때는 관례를 따르는 이름의 확장 함수로 구현하는 게 일반적인 패턴이다.

- 코틀린에서는 프로그래머가 직접 연산자를 만들어 사용할 수 없고 언어에서 미리 정해둔 연산자만 오버로딩할 수 있으며, 관례에 따르기 위해 클래스에서 정의해야 하는 이름이 연산자별로 정해져 있다.

#### 오버로딩 가능한 이항 산술 연산자
<table>
    <tr>
        <th>식</th>
        <th>함수 이름</th>
    </tr>
    <tr>
        <td>a * b</td>
        <td>times</td>
    </tr>
    <tr>
        <td>a / b</td>
        <td>div</td>
    </tr>
    <tr>
        <td>a % b</td>
        <td>mod(1.1부터 rem)</td>
    </tr>
    <tr>
        <td>a + b</td>
        <td>plus</td>
    </tr>
    <tr>
        <td>a - b</td>
        <td>minus</td>
    </tr>
</table>

- 연산자를 정의할 때 두 피 연산자(연산자 함수의 두 파라미터)가 같은 타입일 필요는 없다.
```kotlin
// 두 피연산자의 타입이 다른 연산자 정의하기
operator fun Point.times(scale: Double): Point {
    return Point((x * scale).toInt(), (y * scale).toInt())
}

val p = Point(10, 20)
println(p * 1.5) // Point(x=15, y=30)
```

> 코틀린 연산자가 자동으로 교환 법칙(a op b == b op a인 설정)을 지원하지는 않음에 유의하라.

```kotlin
operator fun Double.times(point: Point): Point {
    return Point((this * point.x).toInt(), (this * point.y).toInt())
}

fun main() {
    val p = Point(10, 20)
    println(1.5 * p) // Point(x=15, y=30)
}
```

- 연산자 함수의 반환 타입이 꼭 두 피연산자 중 하나와 일치해야만 하는 것도 아니다.
```kotlin
// 결과 타입이 피연산자 타입과 다른 연산자 정의하기
operator fun Char.times(count: Int): String {
    return toString().repeat(count)
}

println('a' * 3) // aaa
```
- 일반 함수와 마찬가지로 operator 함수도 오버로딩할 수 있다.

> **비트 연산자에 대해 특별한 연산자 함수를 사용하지 않는다.**
> 
> 코틀린은 표준 숫자 타입에 대해 비트 연산자를 정의하지 않는다. 따라서 커스텀 타입에서 비트 연산자를 정의할 수 없다. 대신에 중의 연산자 표기법을 지원하는 일반 함수(and, shl, xor 등)를 사용해 비트 연산을 수행한다. 커스텀 타입에서도 그와 비슷한 함수를 정의해 사용할 수 있다.

### 복합 대입 연산자 오버로딩
- 연산자를 오버로딩하면 + 연산자뿐 아니라 복합 대입 연산자(+=, -= 등)도 자동으로 함께 지원한다.
```kotlin
var point = Point(1, 2)
point += Point(3, 4)
println(point) // Point(x=4, y=6)

val numbers = ArrayList<Int>()
numbers += 42
println(numbers[0]) // 42
```
- 반환 타입이 Unit인 plusAssign 함수를 정의하면 코틀린은 += 연산자에 그 함수를 사용한다.
```kotlin
operator fun <T> MutableCollection<T>.plusAssign(element: T) {
    this.add(element)
}
```

> plus와 plusAssign 연산을 동시에 정의하지 말라.

- 코틀린 표준 라이브러리는 컬렉션에 대해 두 가지 접근 방법을 함께 제공한다. 
- +와 -는 항상 새로운 컬렉션을 반환하며, +=와 -= 연산자는 항상 변경 가능한 컬렉션에 적용해 메모리에 있는 객체 상태를 변화시킨다.
  - 또한 읽기 전용 컬렉션에서 +=와 -=는 변경을 적용한 복사본을 반환한다(따라서 var로 선언한 변수가 가리키는 읽기 전용 컬렉션에만 +=와 -=를 적용할 수 있다).
- 이런 연산자의 피연산자로는 개별 원소를 사용하거나 원소 타입이 일치하는 다른 컬렉션을 사용할 수 있다.
```kotlin
val list = arrayListOf(1, 2)
// +=는 "list"를 변경한다.
list += 3
// +는 두 리스트의 모든 원소를 포함하는 새로운 리스트를 반환한다.
val newList = list + listOf(4, 5)
println(list) // [1, 2, 3]
println(newList) // [1, 2, 3, 4, 5]
```

### 단항 연산자 오버로딩
```kotlin
// 단항 minus(음수) 함수는 파라미터가 없다.
operator fun Point.unaryMinus(): Point {
    // 좌표에서 각 성분의 음수를 취한 새 점을 반환한다.
    return Point(-x, -y)
}

val p = Point(10, 20)
println(-p) // Point(x=-10, y=-20)
```
- 단항 연산자를 오버로딩하기 위해 사용하는 함수는 인자를 취하지 않는다.
#### 오버로딩할 수 있는 단항 산술 연산자
<table>
    <tr>
        <th>식</th>
        <th>함수 이름</th>
    </tr>
    <tr>
        <td>+a</td>
        <td>unaryPlus</td>
    </tr>
    <tr>
        <td>-a</td>
        <td>unaryMinus</td>
    </tr>
    <tr>
        <td>!a</td>
        <td>not</td>
    </tr>
    <tr>
        <td>++a, a++</td>
        <td>inc</td>
    </tr>
    <tr>
        <td>--a, a--</td>
        <td>dec</td>
    </tr>
</table>

```kotlin
operator fun BigDecimal.inc() = this + BigDecimal.ONE

var bd = BigDecimal.ZERO
// 후위 증가 연산자는 println이 실행된 다음에 값을 증가시킨다.
println(bd++) // 0
// 전위 증가 연산자는 println이 실행되기 전에 값을 증가시킨다.
println(++bd) // 2
```

## 비교 연산자 오버로딩
### 동등성 연산자: equals
- ==와 !=는 내부에서 인자가 널인지 검사하므로 다른 연산과 달리 널이 될 수 있는 값에도 적용할 수 있다.
  - a == b라는 비교를 처리할 때 코틀린은 a가 널인지 판단해서 널이 아닌 경우에만 a.equals(b)를 호출한다.
  - `a == b` ➡️ `a?.equals(b) ?: (b == null)`
```kotlin
class Point(val x: Int, val y: Int) {
    // Any에 정의된 메소드를 오버라이딩한다.
    override fun equals(obj: Any?): Boolean {
        // 최적화: 파라미터가 "this"와 같은 객체인지 살펴본다.
        if (obj === this) return true
        // 파라미터 타입을 검사한다.
        if (obj !is Point) return false
        // Point로 스마트 캐스트해서 x와 y 프로퍼티에 접근한다.
        return obj.x == x && obj.y == y
    }
}

println(Point(10, 20) == Point(10, 20)) // true
println(Point(10, 20) != Point(5, 5)) // true
println(null == Point(1, 2)) // false
```
- 식별자 비교(identity equals) 연산자(===)를 사용해 equals의 파라미터가 수신 객체와 같은지 살펴본다.
  - 즉, 자신의 두 피연산자가 서로 같은 객체를 가리키는지(원시 타입인 경우 두 값이 같은지) 비교한다.
- 다른 연산자 오버로딩 관례와 달리 equals는 Any에 정의된 메소드이므로 override가 필요하다(동등성 비교를 모든 코틀린 객체에 대해 적용할 수 있다).
  - Any의 equals에는 operator가 붙어있지만 그 메소드를 오버라이드하는 (하위 클래스) 메소드 앞에는 operator 변경자를 붙이지 않아도 자동으로 상위 클래스의 operator 지정이 적용된다.
- Any에서 상속 받은 equals가 확장 함수보다 우선순위가 높기 때문에 equals를 확장 함수로 정의할 수는 없다.

### 순서 연산자: compareTo
- 자바에서 사용하는 Comparable 인터페이스에 들어있는 compareTo 메소드는 한 객체와 다른 객체의 크기를 비교해 정수로 나타내준다.
- 코틀린도 똑같은 Comparable 인터페이스를 지원한다. 게다가 코틀린은 Comparable 인터페이스 안에 있는 compareTo 메소드를 호출하는 관례를 제공한다.
  - 따라서 비교 연산자(<, >, <=, >=)는 compareTo 호출로 컴파일된다. compareTo가 반환하는 값은 Int다.
  - `a >= b` ➡️ `a.compareTo(b) >= 0`
```kotlin
class Person(
    val firstName: String, val lastName: String
): Comparable<Person> {
    override fun compareTo(other: Person): Int {
        // 인자로 받은 함수를 차례로 호출하면서 값을 비교한다.
        return compareValuesBy(this, other,
            Person::lastName, Person::firstName)    
    }
}

val p1 = Person("Alice", "Smith")
val p2 = Person("Bob", "Johnson")
println(p1 < p2) // false
```
- equals와 마찬가지로 Comparable의 compareTo에도 operator 변경자가 붙어있으므로 하위 클래스의 오버라이딩 함수에 operator를 붙일 필요가 없다.
- compareValuesBy 함수 : 두 객체와 여러 비교 함수를 인자로 받는다. 첫 번째 비교 함수에 두 객체를 넘겨서 두 객체가 같지 않다는 결과(0)가 나오면 두 번째 비교 함수를 통해 두 객체를 비교한다.
  - 두 객체의 대소를 알려주는 0이 아닌 값이 처음 나올 때까지 인자로 받은 함수를 차례로 호출해 두 값을 비교하며, 모든 함수가 0을 반환하면 0을 반환한다.

> 그렇지만 필드를 직접 비교하면 코드는 조금 더 복잡해지지만 비교 속도는 훨씬 더 빨라진다. 언제나 그렇듯이 처음에는 성능에 신경 쓰지 말고 이해하기 쉽고 간결하게 코드를 작성하고, 나중에 그 코드가 자주 호출됨에 따라 성능이 문제가 되면 성능을 개선하라

```kotlin
class Person(
    val firstName: String, val lastName: String
): Comparable<Person> {
    override fun compareTo(other: Person): Int {
        // lastName 비교
        val lastNameComparison = this.lastName.compareTo(other.lastName)
        if (lastNameComparison != 0) {
            return lastNameComparison
        }

        // lastName이 같으면 firstName 비교
        return this.firstName.compareTo(other.firstName)
    }
}

fun main() {
    val p1 = Person("Alice", "Smith")
    val p2 = Person("Bob", "Johnson")
    println(p1 < p2) // false
}
```

## 컬렉션과 범위에 대해 쓸 수 있는 관례
### 인덱스로 원소에 접근: get과 set
- 인덱스 연산자를 사용해 원소를 읽는 연산은 get 연산자 메소드로 변환되고, 원소를 쓰는 연산은 set 연산자 메소드로 변환된다.
```kotlin
// get 관례 구현하기
operator fun Point.get(index: Int): Int {
    return when(index) {
        0 -> x
        1 -> y
        else ->
            throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
}

val p = Point(10, 20)
println(p[1]) // 20
```
- 인덱스에 해당하는 컬렉션 원소를 쓰고 싶을 때는 set이라는 이름의 함수를 정의하면 된다.
  - Point 클래스는 불변 클래스이므로 set이 의미가 없다. 대신 변경 가능한 점을 표현하는 다른 클래스를 만들어서 예제로 사용하자.
```kotlin
data class MutablePoint(var x: Int, var y: Int)

operator fun MutablePoint.set(index: Int, value: Int) {
    when(index) {
        0 -> x = value
        1 -> y = value
        else ->
            throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
}

val p = MutablePoint(10, 20)
p[1] = 42
println(p) // MutablePoint(x=10, y=42)
```

### in 관례
- in은 객체가 컬렉션에 들어있는지 검사(멤버십 검사)한다. 그런 경우 in 연산자와 대응하는 함수는 contains다.
```kotlin
data class Rectangle(val upperLeft: Point, val lowerRight: Point)

operator fun Rectangle.contains(p: Point): Boolean {
    // 범위를 만들고 좌표가 그 범위 안에 있는지 검사한다.
    return p.x in upperLeft.x unitl lowerRight.x &&
            p.y in upperLeft.y until lowerRight.y
}

val rect = Rectangle(Point(10, 20), Point(50, 50))
println(Point(20, 30) in rect) // true
println(Point(5, 5) in rect) // false
```
- <span style='background-color: #fff5b1'/>in의 우항에 있는 객체는 contains 메소드의 수신 객체가 되고, in의 좌항에 있는 객체는 contains 메소드에 인자로 전달된다.

### rangeTo 관례
- rangeTo 함수는 범위를 반환한다. 이 연산자를 아무 클래스에나 정의할 수 있다.
- 하지만 어떤 클래스가 Comparable 인터페이스를 구현하면 rangeTo를 정의할 필요가 없다.
  - rangeTo 함수는 Comparable에 대한 확장 함수다.
```kotlin
operator fun <T: Comparable<T>> T.rangeTo(that: T): ClosedRange<T>
```
- 이 함수는 범위를 반환하며, 어떤 원소가 그 범위 안에 들어있는지 in을 통해 검사할 수 있다.
```kotlin
// 날짜의 범위 다루기
val now = LocalDate.now()

// val vacation = now.rangeTo(now.plusDays(10))
val vacation = now..now.plusDays(10)
println(now.plusWeeks(1) in vacation) // true
```
- rangeTo 연산자는 다른 산술 연산자보다 우선순위가 낮다. 하지만 혼동을 피하기 위해 괄호로 인자를 감싸주면 더 좋다.
```kotlin
val n = 9
println(0..(n + 1)) // 0..10
```
- 또한 `0..n.forEach {}`와 같은 식은 컴파일할 수 없음에 유의하라. 범위 연산자는 우선 순위가 낮아서 범위의 메소드를 호출하려면 범위를 괄호로 둘러싸야 한다.
```kotlin
(0..n).forEach { println(it) } // 0123456789 
```

### for 루프를 위한 iterator 관례
- `for (x in list) { ... }`와 같은 문장은 list.iterator()를 호출해서 이터레이터를 얻은 다음, 자바와 마찬가지로 그 이터레이터에 대해 hasNext와 next 호출을 반복하는 식으로 변환된다.
```kotlin
fun main() {
  val list = listOf(1, 2, 3, 4, 5)
  
    val iterator = list.iterator()
    while (iterator.hasNext()) {
        val x = iterator.next()
        println(x)
    }
} 
```

- 하지만 코틀린에서는 이 또한 관례이므로 iterator 메소드를 확장 함수로 정의할 수 있다.
  - 이런 성질로 인해 일반 자바 문자열에 대한 for 루프가 가능하다.
- 코틀린 표준 라이브러리는 String의 상위 클래스인 CharSequence에 대한 iterator 확장 함수를 제공한다.
```kotlin
// CharaIterator 라이브러리 : 문자열을 이터레이션할 수 있게 해준다.
operator fun CharSequence.iterator(): CharaIterator

for (c in "abc") {}
```
```kotlin
// 날짜 범위에 대한 이터레이터 구현하기
operator fun ClosedRange<LocalDate>.iterator(): Iterator<LocalDate> =
    // 이 객체는 LocalDate 원소에 대한 Iterator를 구현한다.
    object : Iterator<LocalDate> {
        var current = start
        
        // compareTo 관례를 사용해 날짜를 비교한다.
        override fun hasNext() = current <= endInclusive
        
        // 현재 날짜를 저장한 다음에 날짜를 변경한다. 그 후 저장해둔 날짜를 반환한다.
        override fun next() = current.apply { 
            current = plusDays(1) // 현재 날짜를 1일 뒤로 변경한다.
        }
    }

val newYear = LocalDate.ofYearDay(2017, 1)
val daysOff = newYear.minusDays(1)..newYear
// daysOff에 대응하는 iterator 함수가 있으면 daysOff에 대해 이터레이션한다.
for (dayOff in daysOff) { println(dayOff) }
// 2016-12-31
// 2017-01-01
```
- ClosedRange<LocalData>에 대한 확장 함수 iterator를 정의했기 때문에 LocalDate의 범위 객체를 for 루프에 사용할 수 있다.

## 구조 분해 선언과 component 함수
- 구조 분해를 사용하면 복합적인 값을 분해해서 여러 다른 변수를 한꺼번에 초기화할 수 있다.
```kotlin
val p = Point(10, 20)
// x와 y 변수를 선언한 다음에 p의 여러 컴포넌트로 초기화한다.
val (x, y) = p
println(x) // 10
println(y) // 20
```
- 구조 분해 선언은 일반 변수 선언과 비슷해 보인다. 다만 `=`의 좌변에 여러 변수를 괄호로 묶었다는 점이 다르다.
- 내부에서 구조 분해 선언은 다시 관례를 사용한다. 구조 분해 선언의 각 변수를 초기화하기 위해 componentN이라는 함수를 호출한다.
  - N : 구조 분해 선언에 있는 변수 위치에 따라 붙는 번호
```kotlin
class Point(val x: Int, val y: Int) {
    operator fun component1() = x
    operator fun component2() = y
}
```
- data 클래스의 주 생성자에 들어있는 프로퍼티에 대해서는 컴파일러가 자동으로 [componentN](https://velog.io/@dev-baik/Data-Class#componentn) 함수를 만들어준다.
```kotlin
// 값을 저장하기 위한 데이터 클래스 선언한다.
data class NameComponents(val name: String, val extension: String)

fun splitFilename(fullName: String): NameComponents {
    // val result = fullName.split('.', limit = 2)
    // return NameComponents(result[0], result[1])
    
    // 컬렉션에 대해 구조 분해 선언 사용하기
    val (name, extension) = fullName.split('.', limit=2)
    return NameComponents(name, extension)
}

// 구조 분해 선언을 사용해 여러 값 반환하기
val (name, next) = splitFilename("example.kt")
println(name) // example
println(ext) // kt
```
- 표준 라이브러리의 Pair나 Triple 클래스를 사용하면 함수에서 여러 값을 더 간단하게 반환할 수 있다.
- Pair와 Triple은 그 안에 담겨있는 원소의 의미를 말해주지 않으므로 경우에 따라 가독성이 떨어질 수 있는 반면, 직접 클래스를 작성할 필요가 없으므로 코드는 더 단순해진다.

### 구조 분해 선언과 루프
```kotlin
// 구조 분해 선언을 사용해 맵 이터레이션하기
fun printEntries(map: Map<String, String>) {
    // 루프 변수에 구조 분해 선언을 사용한다.
    for ((key, value) in map) {
        println("$key -> $value")
    }
}

val map = mapOf("Oracle" to "Java", "JetBrains" to "Kotlin")
printEntries(map)
// Oracle -> Java
// JetBrains -> Kotlin
```
- 코틀린 표준 라이브러리에는 맵에 대한 확장 함수로 iterator가 들어있다. 그 iterator는 맵 원소에 대한 이터레이터를 반환한다. 따라서 자바와 달리 코틀린에서는 맵을 직접 이터레이션할 수 있다.
- 또한 Map.Entry에 대한 확장 함수로 component1과 component2를 제공한다.
```kotlin
for (entry in map.entries) {
    val key = entry.component1()
    val value = entry.component2()
    // ...
}
```

## 프로퍼티 접근자 로직 재활용: 위임 프로퍼티
- 위임 프로퍼티를 사용하면 값을 뒷받침하는 필드에 단순히 저장하는 것보다 더 복잡한 방식으로 자동하는 프로퍼티를 쉽게 구현할 수 있다. 또한 그 과정에서 접근자 로직을 매번 재구현할 필요도 없다.
- 위임은 객체가 직접 작업을 수행하지 않고 다른 도우미 객체가 그 작업을 처리하게 맡기는 디자인 패턴을 말한다.
  - 이때 작업을 처리하는 도우미 객체를 위임 객체(delegate)라고 한다.

### 위임 프로퍼티 소개
```kotlin
class Foo {
    var p: Type by Delegate()
}
```
- p 프로퍼티는 접근자 로직을 다른 객체에게 위임한다. 여기서는 Delegate 클래스의 인스턴스를 위임 객체로 사용한다.
- by 뒤에 있는 식을 계산해서 위임에 쓰일 객체를 얻는다.
  - 프로퍼티 위임 객체가 따라야 하는 관례를 따르는 모든 객체를 위임에 사용할 수 있다.
- 컴파일러는 숨겨진 도우미 프로퍼티를 만들고 그 프로퍼티를 위임 객체의 인스턴스로 초기화한다.
  - p 프로퍼티는 바로 그 위임 객체에게 자신의 작업을 위임한다.
```kotlin
class Foo {
    private val delegate: Delegate() // 컴파일러가 생성한 도우미 프로퍼티다.
  
    // "p" 프로퍼티를 위해 컴파일러가 생성한 접근자는
    // "delegate"의 getValue와 setValue 메소드를 호출한다.
    var p: Type
      set(value: Type) = delegate.setValue(..., value)
      get() = delegate.getValue(...)
}
```
- 프로퍼티 위임 관례를 따르는 Delegate 클래스는 getValue와 setValue 메소드를 제공해야 한다(물론 변경 가능한 프로퍼티만 setValue를 사용한다).
- 관례를 사용하는 다른 경우와 마찬가지로 getValue와 setValue는 멤버 메소드이거나 확장 함수일 수 있다.
```kotlin
class Delegate {
    // getValue는 게터를 구현하는 로직을 담는다.
    operator fun getValue(...) { ... }
    // setValue 메소드는 세터를 구현하는 로직을 담는다.
    operator fun setValue(..., value: Type) { ... }
}

class Foo {
    // "by" 키워드는 프로퍼티와 위임 객체를 연결한다.
    var p: Type by Delegate()
}

val foo = Foo()
val oldValue = foo.p
foo.p = newValue
```
- p의 게터나 세터는 Delegate 타입의 위임 프로퍼티 객체에 있는 메소드를 호출한다.

### 위임 프로퍼티 사용: by lazy()를 사용한 프로퍼티 초기화 선언
- 지연 초기화는 객체의 일부분을 초기화하지 않고 남겨뒀다가 실제로 그 부분의 값이 필요한 경우 초기화할 때 흔히 쓰이는 패턴이다.
  - 초기화 과정에 많이 사용하거나 객체를 사용할 때마다 꼭 초기화하지 않아도 되는 프로퍼티에 대해 지연 초기화 패턴을 사용할 수 있다.
```kotlin
class Email { /*...*/ }

fun loadEmails(person: Person): List<Email> {
    println("${person.name}의 이메일을 가져옴")
    return listOf(/*...*/)
}
```
- 지연 초기화를 뒷받침하는 프로퍼티를 통해 구현하기
```kotlin
class Person(val name: String) {
    // 데이터를 저장하고 emails의 위임 객체 역할을 하는 emails 프로퍼티
    private var _emails: List<Email>? = null
    
    val emails: List<Email>
      get() {
          if (_emails == null) {
              // 최초 접근 시 이메일을 가져온다.
              _emails = loadEmails(this)
          }
          // 저장해둔 데이터가 있으면 그 데이터를 반환한다.
          return _emails!!
      }
}

val p = Person("Alice")
p.emails
// 최초로 emails를 읽을 때 단 한번만 이메일을 가져온다.
// 결과: Load emails for Alice
p.emails
```
- 뒷받침하는 프로퍼티(_emails)는 값을 저장하고, 다른 프로퍼티인 emails는 _emails라는 프로퍼티에 대한 읽기 연산을 제공한다.
- _emails는 널이 될 수 있는 타입인 반면 emails는 널이 될 수 없는 타입이므로 프로퍼티를 두 개 사용해야 한다.
- 이 구현은 스레드 안전하지 않아서 언제나 제대로 작동한다고 말할 수 없다.
- HOW) 위임 프로퍼티 사용
  - 위임 프로퍼티 : 데이터를 저장할 때 쓰이는 뒷받침하는 프로퍼티와 값이 오직 한 번만 초기화됨을 보장하는 게터 로직을 함께 캡슐화해준다.
```kotlin
// 지연 초기화를 위임 프로퍼티를 통해 구현하기
class Person(val name: String) {
    val emails by lazy { loadEmails(this) }
}
```
- lazy 함수는 코틀린 관례에 맞는 시그니처의 getValue 메소드가 들어있는 객체를 반환한다. 따라서 lazy를 by 키워드와 함께 사용해 위임 프로퍼티를 만들 수 있다.
- lazy 함수의 인자는 값을 초기화할 때 호출할 람다. 기본적으로 스레드 안전하다.
- 하지만 필요에 따라 동기화에 사용할 락을 lazy 함수에 전달할 수 있고, 다중 스레드 환경에서 사용하지 않을 프로퍼티를 위해 lazy 함수가 동기화를 하지 못하게 막을 수 있다.
```kotlin
import kotlin.concurrent.thread

class ExpensiveObject {
    init {
        println("ExpensiveObject initialized")
    }
}

class Example {
    // 동기화에 사용할 락을 lazy 함수에 전달
    val lazyProperty: ExpensiveObject by lazy(LazyThreadSafetyMode.SYNCHRONIZED) {
        println("Initializing Lazy Property")
        ExpensiveObject()
    }

    // 다중 스레드 환경에서 사용하지 않을 프로퍼티를 위해 lazy 함수가 동기화를 하지 못하게 막기
    // val lazyProperty: ExpensiveObject by lazy(LazyThreadSafetyMode.NONE) {
    //    ...
    // }
}

fun main() {
    val example = Example()
  
    // 여러 스레드에서 동시에 접근하는 상황 시뮬레이션
    repeat(5) {
        thread {
            println("Accessing Lazy Property: ${example.lazyProperty}")
        }
    }
  
    // 잠시 대기하여 모든 스레드가 실행을 마치도록 함
    Thread.sleep(1000)
}
```

### 위임 프로퍼티 구현
- PropertyChangeSupport 클래스 : 리스너의 목록을 관리하고 PropertyChangeEvent 이벤트가 들어오면 목록의 모든 리스너에게 이벤트를 통지한다.
- 자바 빈 클래스의 필드에 PropertyChangeSupport 인스턴스를 저장하고 프로퍼티 변경 시 인스턴스에게 처리를 위임하는 방식으로 이런 통지 기능을 주로 구현한다.

> ✅ **자바 빈 클래스** : 일정한 규약을 따르는 클래스로, 기본 생성자, getter 및 setter 메서드를 포함하며, 멤버 변수는 private으로 선언되고 이에 접근하기 위한 표준 메서드를 제공한다. 이러한 클래스는 재사용 가능한 컴포넌트를 만드는데 사용된다.

```kotlin
// PropertyChangeSupport를 사용하기 위한 도우미 클래스
open class PropertyChangeAware {
    protected val changeSupport = PropertyChangeSupport(this)
  
    fun addPropertyChangeListener(listener: PropertyChangeListener) {
        changeSupport.addPropertyChangeListener(listener)
    }
  
    fun removePropertyChangeListener(listener: PropertyChangeListener) {
        changeSupport.removePropertyChangeListener(listener)      
    }
}
```
```kotlin
// 프로퍼티 변경 통지를 직접 구현하기
class Person(
    val name: String, age: Int, salary: Int
) : PropertyChangeAware() {
    var age: Int = age
      set(newValue) {
          val oldValue = field
          field = newValue
        
          // 프로퍼티 변경을 리스너에게 통지한다.
          changeSupport.firePropertyChange(
              "age", oldValue, newValue)
      }
  
    var salary: Int = salary
      set(newValue) {
          val oldValue = field
          field = newValue
        
          changeSupport.firePropertyChange(
              "salary", oldValue, newValue)
      }
}

val p = Person("Dmitry", 34, 2000)
p.addPropertyChangeListener(
    PropertyChangeListener { event ->
        println("Property ${event.propertyName} changed " +
            "from ${event.oldValue} to ${event.newValue}")
    }
)
p.age = 35 // Property age changed from 34 to 35
p.salary = 2100 // Property salary changed from 2000 to 2100
```
- 세터 코드를 보면 중복이 많이 보인다. 이제 프로퍼티의 값을 저장하고 필요에 따라 통지를 보내주는 클래스를 추출해보자.

```kotlin
// 도우미 클래스를 통해 프로퍼티 변경 통지 구현하기
class ObservableProperty(
  val propName: String, var propValue: Int,
  val changeSupport: PropertyChangeSupport
) {
    fun getValue(): Int = propValue
  
    fun setValue(newValue: Int) {
        val oldValue = propValue
        propValue = newValue
        changeSupport.firePropertyChange(propName, oldValue, newValue)
    }
}

class Person(
    val name: String, age: Int, salary: Int
) : PropertyChangeAware() {
    val _age = ObservableProperty("age", age, changeSupport)
    var age: Int
      get() = _age.getValue()
      set(value) { _age.setValue(value) }
  
    val _salary = ObservableProperty("salary", salary, changeSupport)
    val salary: Int
      get() = _salary.getValue()
      set(value) { _salary.setValue(value) }
}
```
- 코틀린의 위임이 실제로 작동하는 방식과 비슷하다. 프로퍼티 값을 저장하고 그 값이 바뀌면 자동으로 변경 통지를 전달해주는 클래스를 만들었고, 로직의 중복을 상당 부분 제거했다.
- 하지만 아직도 각각의 프로퍼티마다 ObservableProperty를 만들고 게터와 세터에서 ObservableProperty에 작업을 위임하는 준비 코드가 상당 부분 필요하다.
- **HOW)** 코틀린의 위임 프로퍼티 기능을 활용하면 이런 준비 코드를 없앨 수 있다.
```kotlin
// ObservableProperty를 프로퍼티 위임에 사용할 수 있게 바꾼 모습
class ObservableProperty(
    var propValue: Int, val changeSupport: PropertyChangeSupport
) {
    operator fun getValue(p: Person, prop: KProperty<*>): Int = propValue
  
    operator fun setValue(p: Person, prop: KProperty<*>, newValue: Int) {
        val oldValue = propValue
        propValue = newValue
        changeSupport.firePropertyChange(prop.name, oldValue, newValue)
    }
}
```
- 코틀린 관례에 사용하는 다른 함수와 마찬가지로 getValue와 setValue에도 operator 변경자가 붙는다.
- getValue와 setValue는 프로퍼티가 포함된 객체(여기서는 Person 타입인 p)와 프로퍼티를 표현하는 객체를 파라미터로 받는다. 코틀린은 [KProperty](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-property/) 타입의 객체를 사용해 프로퍼티를 표현한다.

<table>
  <tr>
    <th>getValue</th>
    <th>setValue</th>
  </tr>
  <tr>
    <td><strong>thisRef</strong> : 위임을 사용하는 클래스와 같은 타입이거나 Any 타입이어야 한다.</td>
    <td><strong>thisRef</strong> : 위임을 사용하는 클래스와 같은 타입이거나 Any 타입이어야 한다.</td>
  </tr>
  <tr>
    <td><strong>property</strong> : Property<*>거나 Any 타입이어야 한다.</td>
    <td><strong>property</strong> : Property<*>거나 Any 타입이어야 한다.</td>
  </tr>
  <tr>
    <td></td>
    <td><strong>newValue</strong> : 위임을 사용하는 프로퍼티와 같은 타입이거나 Any 타입이어야 한다.</td>
  </tr>
</table>

- KProperty 인자를 통해 프로퍼티 이름을 전달받으므로 주 생성자에서는 name 프로퍼티를 없앤다.

> ✅ **KProperty** : 코틀린의 리플렉션 API 중 하나로, 프로퍼티를 나타내는 역할을 한다. KProperty의 인스턴스는 `::` 연산자로 얻을 수 있습니다.
> 
> ✅ **리플렉션(Reflection)** : 실행 중인 프로그램의 클래스, 메소드, 필드 등과 같은 구조를 동적으로 살펴보거나 수정할 수 있는 능력

```kotlin
// 위임 프로퍼티를 통해 프로퍼티 변경 통지 받기
class Person(
    val name: String, age: Int, salary: Int
) : PropertyChangeAware() {
    var age: Int by ObservableProperty(age, changeSupport)
    var salary: Int by ObservableProperty(salary, changeSupport)
}
```
- by 키워드를 사용해 위임 객체를 지정하면 여러 작업을 코틀린 컴파일러가 자동으로 처리해준다.
- by 오른쪽에 오는 객체를 위임 객체라고 부른다.
- 코틀린은 위임 객체를 감춰진 프로퍼티에 저장하고, 주 객체의 프로퍼티를 읽거나 쓸 때마다 위임 객체의 getValue와 setValue를 호출해준다.


- 관찰 가능한 프로퍼티 로직을 직접 작성하는 대신 코틀린 표준 라이브러리를 사용해도 된다.
- 표준 라이브러리에는 이미 ObservableProperty와 비슷한 클래스가 있다. 다만 이 표준 라이브러리의 클래스는 PropertyChangeSupport와는 연결돼 있지 않다.
  - 따라서 프로퍼티 값의 변경을 통지할 때 PropertyChangeSupport를 사용하는 방법을 알려주는 람다를 그 표준 라이브러리 클래스에게 넘겨야 한다.
```kotlin
// Delegates.observable을 사용해 프로퍼티 변경 통지 구현하기
class Person(
    val name: String, age: Int, salary: Int
) : PropertyChangeAware() {
    private val observer = {
        prop: KProperty<*>, oldValue: Int, newValue: Int ->
        changeSupport.firePropertyChange(prop.namme, oldValue, newValue)
    }
    
    var age: Int by Delegates.observable(age, observer)
    var salary: Int by Delegates.observable(salary, observer)
}
```
- by의 오른쪽에 있는 식이 꼭 새 인스턴스를 만들 필요는 없다. 함수 호출, 다른 프로퍼티, 다른 식 등이 by의 우항에 올 수 있다.
  - <span style='background-color: #fff5b1'/>다만 우항에 있는 식을 계산한 결과인 객체는 컴파일러가 호출할 수 있는 올바른 타입의 getValue와 setValue를 반드시 제공해야 한다.
  - 다른 관례와 마찬가지로 getValue와 setValue 모두 객체 안에 정의된 메소드이거나 확장 함수일 수 있다.

### 위임 프로퍼티 컴파일 규칙
```kotlin
class C {
    var prop: Type by MyDelegate()
}

val c = C()
```
- 컴파일러는 MyDelegate 클래스의 인스턴스를 감춰진 프로퍼티에 저장하며 그 감춰진 프로퍼티를 `<delegate>`라는 이름으로 부른다.
- 또한 컴파일러는 프로퍼티를 표현하기 위해 KProperty 타입의 객체를 사용한다. 이 객체를 `<property>`라고 부른다.
```kotlin
class C {
    private val <delegate> = MyDelegate()
    var prop: Type
      get() = <delegate>.getValue(this, <property>)
      set(value: Type) = <delegate>.setValue(this, <property>, value)
}
```

### 프로퍼티 값을 맵에 저장
- 자신의 프로퍼티를 동적으로 정의할 수 있는 객체를 만들 때 위임 프로퍼티를 활용하는 경우가 자주 있다. 그런 객체를 `확장 가능한 객체`라고 부르기도 한다.
```kotlin
// 값을 맵에 저장하는 프로퍼티 정의하기
class Person {
    // 추가 정보
    private val _attributes = hashMapOf<String, String>()
  
    fun setAttribute(attrName: String, value: String) {
        _attributes[attrName] = value
    }
  
    // 필수 정보
    val name: String
      // 수동으로 맵에서 정보를 꺼낸다.
      get() = _attributes["name"]!!
}

val p = Person()
val data = mapOf("name" to "Dmitry", "company" to "JetBrains")
for ((attrName, value) in data)
    p.setAttribute(attrName, value)
println(p.name) // Dmitry
```
- 이 코드는 추가 데이터를 저장하기 위해 일반적인 API를 사용하고(실제 프로젝트에서는 JSON 역직렬화 등의 기술을 활용할 수 있다), 특정 프로퍼티(name)를 처리하기 위해 구체적인 개별 API를 제공한다.
- 이를 위임 프로퍼티를 활용하게 변경할 수 있다. by 키워드 뒤에 맵을 직접 넣으면 된다.
```kotlin
// 값을 맵에 저장하는 위임 프로퍼티 사용하기
class Person {
    private val _attributes = hashMapOf<String, String>()
    
    fun setAttribute(attrName: String, value: String) {
        _attributes[attrName] = value
    }
  
    // 위임 프로퍼티로 맵을 사용한다.
    val name: String by _attributes
}
```
- 이런 코드가 작동하는 이유는 표준 라이브러리가 Map과 MutableMap 인터페이스에 대해 getValue와 setValue 확장 함수를 제공하기 때문이다.
  - getValue에서 맵에 프로퍼티 값을 저장할 때는 자동으로 프로퍼티 이름을 키로 활용한다.
- `p.name`은 `_attributes.getValue(p, prop)`라는 호출을 대신하고, `_attributes.getValue(p, prop)`는 다시 `_attributes[prop.name]`을 통해 구현된다.

### 프레임워크에서 위임 프로퍼티 활용
- 객체 프로퍼티를 저장하거나 변경하는 방법을 바꿀 수 있으면 프레임워크를 개발할 때 유용하다.
```kotlin
// 위임 프로퍼티를 사용해 데이터베이스 칼럼 접근하기
object Users: IdTable() { // 객체는 데이터베이스 테이블에 해당한다.
    val name = varchar("name", length = 50).index()
    val age = integer("age")
}

// 각 User 인스턴스는 테이블에 들어있는 구체적인 엔티티에 해당한다.
class User(id: EntityID) : Entity(id) {
    // 사용자 이름은 데이터베이스 "name" 칼럼에 들어 있다.
    var name: String by Users.name
    var age: Int by Users.age
}
```
- User의 상위 클래스인 Entity 클래스는 데이터베이스 칼럼을 엔티티의 속성 값으로 연결해주는 매핑이 있다.
  - 이 프레임워크를 사용하면 User의 프로퍼티에 접근할 때 자동으로 Entity 클래스에 정의된 데이터베이스 매핑으로부터 필요한 값을 가져오므로 편리하다.
- 어떤 User 객체를 변경하면 그 객체는 변경됨 dirty 상태로 변하고, 프레임워크는 나중에 적절히 데이터베이스에 변경 내용을 반영한다.
- 각 엔티티 속성(name, age)은 위임 프로퍼티며, 칼럼 객체(Users.name, Users.age)를 위임 객체로 사용한다.
- 프레임워크는 Column 클래스 안에 getValue와 setValue 메소드를 정의한다. 이 두 메소드는 코틀린의 위임 객체 관례에 따른 시그니처 요구 사항을 만족한다.
```kotlin
operator fun <T> Column<T>.getValue(o: Entity, desc: KProperty<*>): T {
    // 데이터베이스에서 칼럼 값 가져오기
}

operator fun <T> Column<T>.setValue(o: Entity, desc: KProperty<*>, value: T) {
    // 데이터베이스의 값 변경하기
}
```
- `user.age += 1`이라는 식을 코드에서 사용하면 그 식은 `user.ageDelegate.setValue(user.ageDelegate.getValue() + 1)`과 비슷한 코드로 변환한다(객체 인스턴스와 프로퍼티 파라미터는 생략함)
- getValue와 setValue 메소드는 데이터베이스에서 데이터를 가져오고 기록하는 작업을 처리한다.