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
- operator 키워드를 붙임으로써 어떤 함수가 관례를 따르는 함수임을 명확히 할 수 있다.
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
- in의 우항에 있는 객체는 contains 메소드의 수신 객체가 되고, in의 좌항에 있는 객체는 contains 메소드에 인자로 전달된다.

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
- 또한 0..n.forEach {}와 같은 식은 컴파일할 수 없음에 유의하라. 범위 연산자는 우선 순위가 낮아서 범위의 메소드를 호출하려면 범위를 괄호로 둘러싸야 한다.
```kotlin
(0..n).forEach { println(it) } // 0123456789 
```

### for 루프를 위한 iterator 관례
- `for (x in list) { ... }`와 같은 문장은 list.iterator()를 호출해서 이터레이터를 얻은 다음, 자바와 마찬가지로 그 이터레이터에 대해 hasNext와 next 호출을 반복하는 식으로 변환된다.
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
operator fun ClosedRange<LocalDate>.iter(): Iterator<LocalDate> =
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

val nextYear = LocalDate.ofYearDay(2017, 1)
val daysOff = nextYear.minusDays(1)..newYear
// daysOff에 대응하는 iterator 함수가 있으면 daysOff에 대해 이터레이션한다.
for (dayOff in daysOff) { println(dayOff) }
// 2016-12-31
// 2017-01-01
```
- ClosedRange<LocalData>에 대한 확장 함수 iterator를 정의했기 때문에 LocalDate의 범위 객체를 for 루프에 사용할 수 있다.