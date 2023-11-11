# 코틀린 타입 시스템

> - 널이 될 수 있는 타입과 널을 처리하는 구문의 문법
> - 코틀린 원시 타입 소개와 자바 타입과 코틀린 원시 타입의 관계
> - 코틀린 컬렉션 소개와 자바 컬렉션과 코틀린 컬렉션의 관계

- - -

- 코틀린의 타입 시스템은 코드의 가독성을 향상시키는 데 도움이 되는 몇 가지 특성을 제공한다.
  - 널이 될 수 있는 타입과 읽기 전용 컬렉션이 있다.
- 코틀린은 자바 타입 시스템에서 불필요하거나 문제가 되던 부분을 제거했다.

## 널 가능성
- NullPointerException 오류(NPE)를 피할 수 있게 돕기 위한 코틀린 타입 시스템의 특성이다.
- null에 대한 접근 방법은 가능한 한 이 문제를 실행 시점에서 컴파일 시점으로 옮기는 것이다.
  - 널이 될 수 있는지 여부를 타입 시스템에 추가함으로써 컴파일러가 여러 가지 오류를 컴파일 시 미리 감지해서 실행 시점에 발생할 수 있는 예외의 가능성을 줄일 수 있다.

### 널이 될 수 있는 타입
- 코틀린과 자바의 가장 중요한 차이는 코틀린 타입 시스템이 널이 될 수 있는 타입을 명시적으로 지원한다는 점이다.
  - 널이 될 수 있는 타입은 프로그램 안의 프로퍼티나 변수에 null을 허용하게 만드는 방법이다.
- 어떤 변수가 널이 될 수 있다면 그 변수에 대해(그 변수를 수신 객체로) 메소드를 호출하면 NullPointerException이 발생할 수 있으므로 안전하지 않다. 코틀린은 그런 메소드 호출을 금지함으로써 많은 오류를 방지한다.
```java
int strLen(String s) {
    return s.lenth();
}
```
```kotlin
fun strLen(s: String) = s.length

strLen(null) // Error: Null can not be a value of a non-null type String
```
- 컴파일러는 널이 될 수 있는 값을 strLen에게 인자로 넘기지 못하게 막는다. 따라서 strLen 함수가 결고 실행 시점에 NullPointerException을 발생시키지 않으리라 장담할 수 있다.


- 이 함수가 널과 문자열을 인자로 받을 수 있게 하려면 타입 이름 뒤에 물음표(?)를 명시해야 한다.
```kotlin
fun strLenSafe(s: String?) = ... 
```
- String?, Int?, MyCustomType? 등 어떤 타입이든 타입 이름 뒤에 물음표를 붙이면 그 타입의 변수나 프로퍼티에 null 참조를 저장할 수 있다는 뜻이다.
- 따라서 모든 타입은 기본적으로 널이 될 수 없는 타입이다. 


- 널이 될 수 있는 타입의 변수가 있다면 그에 대해 수행할 수 있는 연산이 제한된다.
```kotlin
fun strLenSafe(s: String?) = s.length()
// 결과: Error: only safe(?.) or non-null asserted (!!.) calls are allowed on ..
```
- 널이 될 수 있는 값을 널이 될 수 없는 타입의 변수에 대입할 수 없다.
```kotlin
val x: String? = null
var y: String = x
// 결과: Error: Type mismatch: inferred type is String? but String was expected
```
- 널이 될 수 있는 타입의 값을 널이 될 수 없는 타입의 파리미터를 받는 함수에 전달할 수 없다.
```kotlin
strLen(x)
// 결과: Error: Type mismatch: inferred type is String? but String was expected
```

> **이렇게 제약이 많으면 널이 될 수 있는 타입의 값으로 대체 뭘 할 수 있을까?**
> 
> 가장 중요한 일은 바로 null과 비교하는 것이다. 일단 null과 비교하고 나면 컴파일러는 그 사실을 기억하고 null이 아님이 확실한 영역에서는 해당 값을 널이 될 수 없는 타입의 값처럼 사용할 수 있다.

```kotlin
// if 검사를 통해 null 값 다루기
fun strLenSafe(s: String?): Int =
    if (s != null) s.length else 0 // null 검사를 추가하면 코드가 컴파일된다.

val x: String? = null
println(strLenSafe(x)) // 0

println(strLenSafe("abc")) // 3
```