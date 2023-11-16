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

> 외부 함수의 클래스에 대한 연산자를 정의할 떄는 관례를 따르는 이름의 확장 함수로 구현하는 게 일반적인 패턴이다.

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