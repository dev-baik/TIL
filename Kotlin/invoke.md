## invoke
특정 클래스나 객체에서 호출 가능한 함수처럼 이름 없이 호출할 수 있게 도와주는 함수

```kotlin
class Greeter(val greeting: String) {
    operator fun invoke(name: String) {
        println("$greeting, $name!")
    }
}

val greeter = Greeter("Hello")

// invoke 함수의 특별한 기능 : 이름 없이 호출이 가능하다.
// greeter.invoke("World")
greeter("World")  // Hello, World!
```

## 연산자
### operator 키워드
특정한 이름의 함수를 사용하여 연산자의 동작을 재정의할 수 있게 해주는 기능으로, 연산자 오버로딩을 통해 사용자는 기존 연산자의 동작을 클래스나 타입에 맞게 재정의할 수 있다. 

### 관례
사전적 정의 : 이전부터 있었던 사례. 예로부터 전하여 내려오는 일 처리의 관습.

소프트웨어 관점 : 특별한 이름이 붙은 함수를 일반 메소드 호출 구문으로 호출하지 않고 더 간단한 다른 구문으로 호출할 수 있게 지원하는 기능

- 연산자가 자동으로 교환 법칙(a op b == b op a인 설정)을 지원하지 않는다.
```kotlin
// 두 피연산자의 타입이 다른 연산자 정의하기
operator fun Point.times(scale: Double): Point {
    return Point((x * scale).toInt(), (y * scale).toInt())
}

val p = Point(10, 20)
println(p * 1.5) // Point(x=15, y=30)
```
```kotlin
operator fun Double.times(point: Point): Point {
    return Point((this * point.x).toInt(), (this * point.y).toInt())
}

fun main() {
    val p = Point(10, 20)
    println(1.5 * p) // Point(x=15, y=30)
}
```
- 반환 타입이 Unit인 plusAssign 함수를 정의하면 코틀린은 `+=` 연산자에 그 함수를 사용한다.
  - Unit 타입 : 의미 있는 값을 반환하지 않는 함수 반환 타입에 쓰는 특별한 타입
```kotlin
operator fun <T> MutableCollection<T>.plusAssign(element: T) {
    this.add(element)
}

val numbers = ArrayList<Int>()
numbers += 42
println(numbers[0]) // 42
```

## 람다와 invoke의 관계
함수 타입인 변수는 인자 개수에 따라 적당한 FunctionN 인터페이스를 구현하는 클래스의 인스턴스를 저장하며, 그 클래스의 invoke 메소드 본문에는 람다의 본문이 들어간다.
```kotlin
// 코틀린
val processTheAnswer = { number: Int -> number.toString() }

fun processTheAnswer() {
    val function: (Int) -> String = object : Function1<Int, String> {
        override fun invoke(number: Int): String {
            return number.toString()
        }
    }
}

// 자바로 컴파일된 코드
// Function인자의개수<인자1, 인자2, ..., 결과>
processTheAnswer {
    new Function1<Integer, String>() {
        @Override
        public String invoke(Integer number) {
            return number.toString();
        }
    }
}
```
- 널이 될 수 있는 함수 타입으로 함수를 받으면 그 함수를 직접 호출할 수 없다.
```kotlin
fun foo(callback: (() -> Unit)?) {
    if (callback != null) {
        callback()
    }
}
```
- 함수 타입이 invoke 메소드를 구현하는 인터페이스라는 사실을 활용하면 더 짧게 만들 수 있다.
```kotlin
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

#### 참고 사이트
- https://wooooooak.github.io/kotlin/2019/03/21/kotlin_invoke/