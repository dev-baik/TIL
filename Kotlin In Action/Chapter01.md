# 코틀린이란 무엇이며, 왜 필요한가?
> - 코틀린 기본 기능 데모
> - 코틀린 언어의 주요 특성
> - 코틀린이 다른 언어보다 더 나은 점
> - 코틀린으로 코드를 작성하고 실행하는 방법

---

## [코틀린은 무엇인가?](https://velog.io/@dev-baik/Kotlin)
> 자바 플랫폼에서 돌아가는 새로운 프로그래밍 언어다.
> 간결하고 실용적이며, 자바 코드와의 상호운용성을 중시한다.
> 현재 자바가 사용 중인 곳이라면 거의 대부분 코틀린을 활용할 수 있다.
> 기존 자바 라이브러리 프레임워크와 함께 잘 작동하며, 성능도 자바와 같은 수준이다.

## 코틀린 맛보기
```kotlin
// Person 이라는 클래스 정의하기
data class Person(val name: String?, val age: Int? = null)

fun main(args: Array<String>) {
    // Person 클래스를 사용해 사람을 모아둔 컬렉션을 만들기
    val persons = listOf(Person("영희"), Person("철수", age = 29))
  
    // 가장 나이가 많은 사람을 찾아 결과를 출력하기
    val oldest = persons.maxBy { it.age ?: 0 }
    println("나이가 가장 많은 사람: $oldest")
}

// 결과: 나이가 가장 많은 사람: Person(name=철수, age=29)
```
1. name과 age라는 프로퍼티가 들어간 간단한 데이터 클래스를 정의한다.
2. age 프로퍼티의 디폴트 값은 (따로 지정하지 않은 경우) null 이다.
    - 사람 리스트를 만들면서 영희의 나이를 지정하지 않았기 때문에 null이 대신 쓰인다.
3. 리스트에서 가장 나이가 많은 사람을 찾기 위해 maxBy 함수를 사용한다.
    - maxBy 함수에 전달한 람다 식은 파라미터를 하나 받는다.
        - it이라는 이름을 사용하면 (별도로 파라미터 이름을 정의하지 않아도) 람다식의 유일한 인자를 사용할 수 있다.
        - 엘비스 연산자라고 부르는 ?:는 age가 null인 경우 0을 반환하고, 그렇지 않은 경우 age의 값을 반환한다.
4. 영희의 나이를 지정하지는 않았지만 엘비스 연산자 null을 0으로 변환해주기 때문에 철수가 가장 나이가 많은 사람으로 선정될 수 있다.

## 코틀린의 주요 특성
### 코틀린의 주 목적
> 현재 자바가 사용되고 있는 모든 용도에 적합하면서도 더 간결하고 생산적이며 안전한 대체 언어를 제공하는 것

### 정적 타입 지정 언어
#### 자바와 마찬가지로 코틀린도 정적 타입 지정 언어다.
- 정적 타입 지정 : 모든 프로그램 구성 요소의 타입을 컴파일 시점에 알 수 있고 프로그램 안에서 객체의 필드나 메서드를 사용할 때마다 컴파일러가 타입을 검증해준다는 뜻
- 동적 타입 지정
    - 타입과 관계없이 모든 값을 변수에 넣을 수 있고, 메서드나 필드 접근에 대한 검증이 실행 시점에 일어나며, 그에 따라 코드가 더 짧아지고 데이터 구조를 더 유연하게 생성하고 사용할 수 있다.
        - BUT) 이름을 잘못 입력하는 등의 실수도 컴파일 시 걸러내지 못하고 실행 시점에 오류가 발생한다.

#### 자바와 달리 코틀린에서는 모든 변수의 타입을 프로그래머가 직접 명시할 필요가 없다.
- 대부분의 경우 코틀린 컴파일러가 문맥으로부터 변수 타입을 자동으로 유추할 수 있기 때문에 타입 선언을 생략해도 된다. (타입 추론)

### 정적 타입 지정의 장점
- **성능** : 실행 시점에 어떤 메서드를 호출할지 알아내는 과정이 필요없으므로 메서드 호출이 더 빠르다.

- **신뢰성** : 컴파일러가 프로그램의 정확성을 검증하기 때문에 실행 시 프로그램이 오류로 중단될 가능성이 더 작아진다.

- **유지 보수성** : 코드에서 다루는 객체가 어떤 타입에 속하는지 알 수 있기 때문에 처음 보는 코드를 다룰 때도 더 쉽다.

- **도구 지원** : 정적 타입 지정을 활용하면 더 안전하게 리팩토링 할 수 있고, 도구는 더 정확한 코드 완성 기능을 제공할 수 있으며, IDE의 다른 지원 기능도 더 잘 만들 수 있다.


### 자바와 달리 코틀린의 중요한 특징
> 널이 될 수 있는 타입을 지원함에 따라 컴파일 시점에 널 포인터 예외가 발생할 수 있는지 여부를 검사할 수 있어서 좀 더 프로그램의 신뢰성을 높일 수 있다.

### 함수형 프로그래밍의 핵심 개념
#### 일급 시민인 함수
함수(프로그램의 행동을 나타내는 코드 조각)를 일반 값처럼 다룰 수 있다. 함수를 변수에 저장할 수 있고, 함수를 인자로 다른 함수에 전달할 수 있으며, 함수에서 새로운 함수를 만들어서 반환할 수 있다.
```kotlin
// 함수를 변수에 저장
val add: (Int, Int) -> Int = { a, b -> a + b }

// 함수를 인자로 다른 함수에 전달
fun calculate(x: Int, y: Int, operation: (Int, Int) -> Int): Int {
    return operation(x, y)
}

val result = calculate(10, 5, add)
println(result) // 출력: 15
```
```kotlin
// 함수에서 새로운 함수를 만들어서 반환
fun addX(x: Int): (Int) -> Int {
    // x를 받아들이고 x를 더하는 함수를 반환
    return { y -> x + y }
}

val add5 = addX(5) // addX 함수를 호출하여 x=5인 함수를 생성
val result = add5(3) // 반환된 함수를 호출하여 5 + 3 계산
println(result) // 출력: 8
```
#### 불변성
함수형 프로그래밍에서는 일단 만들어지고 나면 내부 상태가 절대로 바뀌지 않는 불변 객체를 사용해 프로그램을 작성한다.
```kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val person1 = Person("Alice", 30)
    val person2 = person1.copy(name = "Bob")

    println(person1) // 출력: Person(name=Alice, age=30)
    println(person2) // 출력: Person(name=Bob, age=30)
}
```
#### 부수 효과 없음
함수형 프로그래밍에서는 입력이 같으면 항상 같은 출력을 내놓고 다른 객체의 상태를 변경하지 않으며, 함수 외부나 다른 바깥 환경과 상호작용하지 않는 순수 함수를 사용한다.
```kotlin
// 순수하지 않은 함수: 외부 상태를 변경하는 함수
var total = 0

fun impureAdd(a: Int) {
    total += a
}

fun main() {
    impureAdd(5) // 외부 상태 변경
    println(total) // 출력: 5

    impureAdd(3) // 다시 호출하여 외부 상태 변경
    println(total) // 출력: 8
}
```
### 함수형 프로그래밍의 이점
#### 간결성
함수형 코드는 그에 상응하는 명령형 코드에 비해 더 간결하며 우아하다. (순수) 함수를 값처럼 활용할 수 있으면 더 강력한 추상화를 할 수 있고 강력한 추상화를 사용해 코드 중복을 막을 수 있다.
```kotlin
// 고차 함수 예시 1: 다른 함수를 인자로 받는 함수
fun calculate(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}

fun add(x: Int, y: Int): Int {
    return x + y
}

fun subtract(x: Int, y: Int): Int {
    return x - y
}

fun main() {
    val result1 = calculate(10, 5, ::add)
    println("Result of addition: $result1")

    val result2 = calculate(10, 5, ::subtract)
    println("Result of subtraction: $result2")
}
```
```kotlin
// 고차 함수 예시 2: 함수를 반환하는 함수
fun operationChoice(operator: String): (Int, Int) -> Int {
    return when (operator) {
        "add" -> ::add
        "subtract" -> ::subtract
        else -> { a, b -> a * b }
    }
}

fun main() {
    val operation = operationChoice("add")
    val result = operation(10, 5)
    println("Result of chosen operation: $result")
}
```
```kotlin
fun List<Int>.filterAndSum(predicate: (Int) -> Boolean): Int {
    return this.filter { predicate(it) }.sum()
}

fun main() {
    val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

    val evenSum = numbers.filterAndSum { it % 2 == 0 }
    val oddSum = numbers.filterAndSum { it % 2 != 0 }

    println("Sum of even numbers: $evenSum") // 출력: Sum of even numbers: 30
    println("Sum of odd numbers: $oddSum")   // 출력: Sum of odd numbers: 25
}
```
#### 다중 스레드를 사용해도 안전하다
다중 스레드 프로그램에서는 적절한 동기화 없이 같은 데이터를 여러 스레드가 변경하는 경우 가장 많은 문제가 생긴다. 하지만, 불변 데이터 구조를 사용하고 순수 함수를 그 데이터 구조에 적용한다면 다중 스레드 환경에서 같은 데이터를 여러 스레드가 변경할 수 없다. 즉, 복잡한 동기화를 적용하지 않아도 된다.
```kotlin
fun main() {
    // 불변한 데이터 구조인 리스트를 생성
    val numbers = listOf(1, 2, 3, 4, 5)

    // 여러 스레드가 이 데이터를 동시에 읽어도 문제 없음

    // 순수 함수를 사용하여 리스트를 변경하지 않고 결과를 계산
    val sum = calculateSum(numbers)
    val product = calculateProduct(numbers)

    println("Sum: $sum")       // 출력: Sum: 15
    println("Product: $product") // 출력: Product: 120
}

fun calculateSum(numbers: List<Int>): Int {
    return numbers.sum()
}

fun calculateProduct(numbers: List<Int>): Int {
    // reduce : 컬랙션 내의 데이터를 모두 모으는 함수, 초기값이 없이 첫번째 요소로 시작
    return numbers.reduce { acc, num -> acc * num }
}
```
#### 테스트하기 쉽다
부수 효과가 있는 함수는 그 함수를 실행할 때 필요한 전체 환경을 구성하는 준비 코드가 따로 필요하지만, 순수 함수는 그런 준비 코드 없이 독립적으로 테스트할 수 있다.

### 함수형 프로그래밍을 위한 지원
- 함수 타입을 지원함에 따라 어떤 함수가 다른 함수를 파라미터로 받거나 함수가 새로운 함수를 반환할 수 있다.

- 람다 식을 지원함에 따라 번거로운 준비 코드를 작성하지 않아도 코드 블록을 쉽게 정의하고 여기저기 전달할 수 있다.

- 데이터 클래스는 불변적인 값 객체를 간편하게 만들 수 있는 구문을 제공한다.

- 코틀린 표준 라이브러리는 객체와 컬렉션을 함수형 스타일로 다룰 수 있는 API를 제공한다.

## 코틀린의 철학
### 실용성
다른 프로그래밍 언어가 채택한 이미 성공적으로 검증된 해법과 기능에 의존한다. 이로 인해 언어의 복잡도가 줄어들고 이미 알고 기존 개념을 통해 코틀린을 더 쉽게 배울 수 있다.
### 간결성
코틀린을 만들면서 의미가 없는 부분을 줄이고, 언어가 요구하는 구조를 만족시키기 위해 프로그램에 꼭 넣어야 하는 부수적인 요소를 줄이기 위해 많은 노력을 기울였다. 게터, 세터, 생성자 파라미터를 필드에 대입하기 위한 로직 등 묵시적으로 제공한다.
### 안전성
#### 안전성과 생산성 사이에는 [트레이드오프 관계](https://ko.wikipedia.org/wiki/%ED%8A%B8%EB%A0%88%EC%9D%B4%EB%93%9C%EC%98%A4%ED%94%84)가 성립한다.
코틀린을 만들면서 자바보다 더 높은 수준의 안전성을 달성하되 전체 비용은 더 적게 지불하고 싶었다. 타입 안전성, 한걸음 더 나아가 실행 시점에 오류를 발생시키는 대신 컴파일 시점 검사를 통해 오류를 더 많이 방지해준다. 코틀린의 타입 시스템은 null이 될 수 없는 값을 추적하며, 실행 시점에 NullPointerException이 발생할 수 있는 연산을 사용하는 코드를 금지한다.

또한, 어떤 객체를 다른 타입으로 캐스트하기 전에 타입을 미리 검사하지 않아 발생할 수 있는 ClassCastException을 방지해준다. 타입 검사와 캐스트가 한 연산자에 의해 이뤄진다.
```kotlin
if (value is String)
    println(value.toUpperCase())
```
### 상호운용성
기존 자바 라이브러리를 그대로 사용할 수 있다. 자바 메서드를 호출하거나 자바 클래스를 상속하거나 인터페이스를 구현하거나 자바 애노테이션을 코틀린 코드에 적용하는 등의 일이 모두 가능하다.

자체 컬렉션 라이브러리를 제공하지 않는다. 자바 표준 라이브러리 클래스에 의존한다. 다만 코틀린에서 더 쉽게 활용할 수 있게 몇 가지 기능을 더할 뿐이다.