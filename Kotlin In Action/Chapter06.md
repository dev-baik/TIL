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
- 컴파일러는 널이 될 수 있는 값을 strLen에게 인자로 넘기지 못하게 막는다. 따라서 strLen 함수가 결코 실행 시점에 NullPointerException을 발생시키지 않으리라 장담할 수 있다.


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

### 타입의 의미
- [타입](https://ko.wikipedia.org/wiki/%EC%9E%90%EB%A3%8C%ED%98%95) : 분류로 ... 타입은 어떤 값이 가능한지와 그 타입에 대해 수행할 수 있는 연산의 종류를 결정한다.
- 코틀린은 널이 될 수 있는 타입과 널이 될 수 없는 타입을 구분하면 각 타입의 값에 대해 어떤 연산이 가능할 지 명확히 이해할 수 있고, 실행 시점에 예외를 발생시킬 수 있는 연산을 판단할 수 있다. 따라서 그런 연산을 아예 금지시킬 수 있다.

> 실행 시점에 널이 될 수 있는 타입이나 널이 될 수 없는 타입의 객체는 같다. 널이 될 수 있는 타입은 널이 될 수 없는 타입을 감싼 래퍼 타입이 아니다. 모든 검사는 컴파일 시점에 수행된다. 따라서 코틀린에서는 널이 될 수 있는 타입을 처리하는 데 별도의 실행 시점 부가 비용이 들지 않는다.

### 안전한 호출 연산자: ?.
- ?. : null 검사와 메소드 호출을 한 번의 연산으로 수행한다.
  - `s?.toUpperCase()`와 `if (s != null) s.toUpperCase() else null`는 같다.
- 안전한 호출(?.)의 결과 타입도 널이 될 수 있는 타입이라는 사실에 유의하라.
```kotlin
fun printAllCaps(s: String?) {
    val allCaps: String? = s?.toUpperCase() // allCaps는 널일 수도 있다.
    println(allCaps)
}

printAllCaps("abc") // ABC
printAllCaps(null) // null
```
- 메서드 호출뿐 아니라 프로퍼티를 읽거나 쓸 때도 안전한 호출을 사용할 수 있다.
```kotlin
class Employee(val name: String, val manager: Employee?)

fun managerName(employee: Employee): String? = employee.manager?.name

val ceo = Employee("Da Boss", null)
val developer = Employee("Bob smith", ceo)

println(managerName(developer)) // Da Boss
println(ManagerName(ceo)) // null
```
- 객체 그래프에서 널이 될 수 있는 중간 객체가 여럿 있다면 한 식 안에서 안전한 호출을 연쇄해서 함께 사용하면 편할 때가 자주 있다.
```kotlin
// 안전한 호출 연쇄시키기
class Address(val streetAddress: String, val zipCode: Int,
            val city: String, val country: String)

class Company(val name: String, val address: Address?)

class Person(val name: String, val company: Company?)

fun Person.countryName(): String {
    // 여러 안전한 호출 연산자를 연쇄해 사용한다.
    val country = this.company?.address?.country
    return if (country != null) country else "Unknown"    
}

val person = Person("Dmitry", null)
println(person.countryName()) // Unknown
```

### 엘비스 연산자: ?:
- 엘비스 연산자 : null 대신 사용할 디폴트 값을 지정할 때 편리하게 사용할 수 있는 연산자를 제공한다.
```kotlin
fun foo(s: String?) {
    val t: String = s ?: ""
}
```
```kotlin
// 엘비스 연산자를 활용해 널 값 다루기
fun strLenSafe(s: String?): Int = s?.length ?: 0

print(strLenSafe("abc")) // 3
println(strLenSafe(null)) // 0
```
```kotlin
fun Person.countryName() = company?.address?.country ?: "Unknown"
```
- 코틀린에서는 return이나 throw 등의 연산도 식이다. 따라서 엘비스 연산자의 우항에 return, throw 등의 연산을 넣을 수 있고, 엘비스 연산자를 더욱 편하게 사용할 수 있다.
  - 함수의 전체 조건을 검사하는 경우 특히 유용하다.

```kotlin
// throw와 엘비스 연산자 함께 사용하기
class Address(val streetAddress: String, val zipCode: Int, 
              val city: String, val country: String)

class Company(val name: String, val address: Address?)

class Person(val name: String, val company: Company?)

fun printShippingLabel(person: Person) {
    val address = person.company?.address
      ?: throw IllegalArgumentException("No address") // 주소가 없으면 예외를 발생시킨다.
  
    with(address) {
        println(streetAddress)
        println("$zipCode $city, $country")
    }
}

val address = Address("Elsestr. 47", 80687, "Munich", "Germany")
val jetbrains = Company("JetBrains", addres)
val person = Person("Dmitry", jetbrains)

printShippingLabel(person)
// Elsestr. 47
// 80687 Munich, Germany

printShippingLabel(Person("Alexey", null))
// java.lang.IllegalArgumentException: No address
```

### 안전한 캐스트: as?
- 자바 타입 캐스트와 마찬가지로 대상 값을 as로 지정한 타입으로 바꿀 수 없으면 ClassCastException이 발생한다.
- as? 연산자는 어떤 값을 지정한 타입으로 캐스트한다. as?는 값을 대상 타입으로 변환할 수 없으면 null을 반환한다.
```kotlin
// 안전한 연산자를 사용해 equals 구현하기
class Person(val firstName: String, val lastName: String) {
    override fun equals(o: Any?): Boolean {
        // 타입이 서로 일치하지 않으면 false를 반환한다.
        val otherPerson = o as? Person ?: return false
        
        // 안전한 캐스트를 하고나면 otherPerson이 Person 타입으로 스마트 캐스트된다.
        return otherPerson.firstName == firstName &&
                otherPerson.lastName == lastName
    }
  
    override fun hashCode(): Int =
        firstName.hashCode() * 37 + lastName.hashCode()
}

val p1 = Person("Dmitry", "Jemerov")
val p2 = Person("Dmitry", "Jemerov")
println(p1 == p2) // true
println(p1.equals(42)) // false
```

### 널 아님 단언: !!
- 널 아님 단언은 코틀린에서 널이 될 수 있는 타입의 값을 다룰 때 사용할 수 있는 도구 중에서 가장 단순하면서도 무딘 도구다.
- 느낌표를 이중(!!)으로 사용하면 어떤 값이든 널이 될 수 없는 타입으로 (강제로) 바꿀 수 있다. 실제 널에 대해 !!를 적용하면 NPE가 발생한다.
```kotlin
fun ignoreNulls(s: String?) {
    val sNotNull: String = s!! // 예외는 이 지점을 가리킨다.
    println(sNotNull.length)
}

ignoreNulls(null)
// Expceion in thread "main" kotlin.KotlinNullPointerException ...
```
- 예외(NullPointerException의 한 종류)를 던지는 일 외에 코틀린이 택할 수 있는 대안이 별로 없다. 하지만 발생한 예외는 null 값을 사용하는 코드(sNotNull.length가 있는 줄)가 아니라 단언문이 위치한 곳을 가리킨다는 점에 유의하라.
- 근본적으로 !!는 컴파일러에게 "나는 이 값이 null이 아님을 잘 알고 있다. 내가 잘못 생각했다면 예외가 발생해도 감수하겠다"라고 말하는 것이다.
- 호출된 함수가 언제나 다른 함수에서 널이 아닌 값을 전달받는 사실이 분명하다면 굳이 검사를 다시 수행하지 않는다.
```kotlin
// 스윙 액션에서 널 아님 단언 사용하기
class CopyRowAction(val list: JList<String>) : AbstractAction() {
    override fun isEnabled(): Boolean =
        list.selectedValue != null
  
    // actionPerformed는 isEnabled가 "true"인 경우에만 호출된다.
    override fun actionPerformed(e: ActionEvent) {
        val value = list.selectedValue!!
    }
    // value를 클립보드로 복사
}
```
- 이 경우 !!를 사용하지 않으려면 `val value = list.selectedValue ?: return`처럼 널이 될 수 없는 타입의 값을 얻어야 한다.
  - list.selectedValue가 null이면 함수가 조기 종료되므로 함수의 나머지 본문에서는 value가 항상 널이 아니게 된다.
- 이 식에서 엘비스 연산자는 중복이라 할 수 있지만 나중에 isEnabled가 더 복잡해질 가능성에 대비해 미리 보호 장치를 마련해 둔다고 생각할 수도 있다.

> !!를 널에 대해 사용해서 발생하는 예외의 스택 트레이스(stack trace)에는 어떤 파일의 몇 번째 줄인지에 대한 정보는 들어있지만 어떤 식에서 예외가 발생햇는지에 대한 정보는 들어있지 않다. 어떤 값이 널이었는지 확실히 하기 위해 여러 !! 단언문을 한 줄에 함께 쓰는 일을 피하라.

### let 함수
- let 함수를 안전한 호출 연산자와 함께 사용하면 원하는 식을 평가해서 결과가 널인지 검사한 다음에 그 결과를 변수에 넣는 작업을 간단한 식을 사용해 한꺼번에 처리할 수 있다.
- let 함수는 널이 될 수 있는 값을 널이 아닌 값만 인자로 받는 함수에 넘기는 경우에 자주 사용된다.
```kotlin
fun sendEmailTo(email: String) { /*...*/ }

val email: string? = ...
sendEmailTo(email)
// 결과: Error: Type mismatch: inferred type is String? but String was expected
```
```kotlin
// 인자를 넘기기 전에 주어진 값이 널인지 검사해야 한다.
if (email != null) sendEmailTo(email)
```
- let 함수는 자신의 수신 객체를 인자로 전달받은 람다에게 넘긴다.
  - 널이 될 수 있는 값에 대해 안전한 호출 구문을 사용해 let을 호출하되 널이 될 수 없는 타입을 인자로 받는 람다를 let에 전달한다.
```kotlin
email?.let { email -> sendEmailTo(email) }
```
```kotlin
// let을 사용해 null이 아닌 인자로 함수 호출하기
fun sendEmailTo(email: String) {
    println("Sending email to $email")
}

var email: String? = "yole@example.com"
email?.let { sendEmailTo(it) }
// Sending email to yole@example.com

email = null
email?.let { sendEmailTo(it) } // 아무 일도 일어나지 않는다.
```
- let을 쓰면 긴 식의 결과를 저장하는 변수를 따로 만들 필요가 없다.
```kotlin
fun getTheBestPersonInTheWorld(): Person? = null

val person: Person? = getTheBestPersonInTheWorld()
if (person != null) sendEmailTo(person.email)

getTheBestPersonInTheWorld()?.let { sendEmailTo(it.email) }
```
- 여러 값이 널인지 검사해야 한다면 let 호출을 중첩시켜서 처리할 수 있다. 그렇게 let을 중첩시켜 처리하면 코드가 복잡해져서 알아보기 어려워진다. 그런 경우 일반적인 if를 사용해 모든 값을 한꺼번에 검사하는 편이 낫다.

### 나중에 초기화할 프로퍼티
- 코틀린에서 클래스 안의 널이 될 수 없는 프로퍼티를 생성자 안에서 초기화하지 않고 특별한 메소드 안에서 초기화할 수 없다.
- 코틀린은 일반적으로 생성자에서 모든 프로퍼티를 초기화해야 한다. 게다가 프로퍼티 타입이 널이 될 수 없는 타입이라면 반드시 널이 아닌 값으로 그 프로퍼티를 초기화해야 한다. 
- 그런 초기화 값을 제공할 수 없으면 널이 될 수 있는 타입을 사용할 수 밖에 없다. 하지만 널이 될 수 있는 타입을 사용하려면 모든 프로퍼티 접근에 널 검사를 넣거나 !! 연산자를 써야 한다.

```kotlin
// 널 아님 단언을 사용해 널이 될 수 있는 프로퍼티 접근하기
class MyService {
    fun performAction(): String = "foo"
}

class MyTest {
    // null로 초기화하기 위해 널이 될 수 있는 타입인 프로퍼티를 선언한다.
    private var myService: MyService? = null
  
    // setUp 메소드 안에서 진짜 초기값을 지정한다.
    @Before fun setUp() {
        myService = myService()
    }
  
    @Test fun testAction() {
        // 반드시 널 가능성에 신경 써야 한다. !!나 ?을 꼭 써야 한다.
        Assert.assertEquals("foo", myService!!.performAction())
    }
}
```
- 이 코드는 보기 나쁘다. 특히 프로퍼티를 여러 번 사용해야 하면 코드가 더 못생겨진다.
- 이를 해결하기 위해 myService 프로퍼티를 나중에 초기화(late-initialized) 할 수 있다.
```kotlin
// 나중에 초기화하는 프로퍼티 사용하기
class myService {
    fun performAction(): String = "foo"
}

class MyTest {
    // 초기화하지 않고 널이 될 수 없는 프로퍼티를 선언한다/
    private lateinit var myService: MyService
    
    @Before fun setUp() {
        myService = MyService()
    }
  
    // 널 검사를 수행하지 않고 프로퍼티를 사용한다.
    @Test fun testAction() {
        Assert.assertEquals("foo", myService.performAction())
    }
}
```
- 나중에 초기화하는 프로퍼티는 항상 var여야 한다. 
  - WHY) val 프로퍼티는 final 필드로 컴파일되며, 생성자 안에서 반드시 초기화해야 한다. 따라서 생성자 밖에서 초기화해야 하는 나중에 초기화하는 프로퍼티는 항상 var여야 한다.
  ```kotlin
  class Example {
      // 필드 선언시 초기값 설정 
      val name: String = "DefaultName"
  }
  
  // val 프로퍼티를 선언과 동시에 초기화하면 해당 초기화 구문이 생성자의 일부로 취급되어, 
  // 컴파일러는 이를 내부적으로 생성된 생성자 코드로 변환
  class Example {
      val name: String
      
      constructor() {
          this.name = "DefaultName"
      }
  }
  ```

### 널이 될 수 있는 타입 확장
- 널이 될 수 있는 타입에 대한 확장 함수를 정의하면 null 값을 다루는 강력한 도구로 활용할 수 있다.
  - 어떤 메소드를 호출하기 전에 수신 객체 역할을 변수가 널이 될 수 없다고 보장하는 대신, 직접 변수에 대해 메소드를 호출해도 확장 함수인 메소드가 알아서 널을 처리해준다.
- 일반 멤버 호출은 객체 인스턴스를 통해 디스패치되므로 그 인스턴스가 널인지 여부를 검사하지 않는다.

> **동적 디스패치** : 객체지향 언어에서 객체의 동적 타입에 따라 적절한 메소드를 호출해주는 방식
> 
> **직접 디스패치** : 반대로 컴파일러가 컴파일 시점에 어떤 메소드가 호출될지 결정해서 코드를 생성하는 방식
> 
> 일반적으로 동적 디스패치를 처리할 때는 객체별로 자신의 메소드에 대한 테이블을 저장하는 방법을 가장 많이 사용한다. 물론 대부분의 객체지향 언어에서 같은 클래스에 속한 객체는 같은 메소드 테이블을 공유하므로 보통 메소드 테이블은 클래스마다 하나씩만 만들고 각 객체는 자신의 클래스에 대한 참조를 통해 그 메소드 테이블을 찾아보는 경우가 많다.

```kotlin
// null이 될 수 있는 수신 객체에 대해 확장 함수 호출하기
fun verifyUserInput(input: String?) {
    if (input.isNullOrBlank()) {
        println("Please fill in the required fields")
    }
}

verifyUserInput(" ") // Please fill in the required fields

// isNullOrBlank에 "null"을 수신 객체로 전달해도 아무런 예외가 발생하지 않는다.
verifyUserInput(null) // Please fill in the required fields
```
- 안전한 호출 없이도 널이 될 수 있는 수신 객체 타입에 대해 선언된 확장 함수를 호출 가능하다.
  - isNullOrBlank는 널을 명시적으로 검사해서 널인 경우 true를 반환하고, 널이 아닌 경우 isBlank를 호출한다.
    - isBlank는 널이 아닌 문자열 타입의 값에 대해서만 호출할 수 있다.
```kotlin
// 널이 될 수 있는 String의 확장
fun String?.isNullOrBlank(): Boolean =
    // 두 번째 "this"에는 스마트 캐스트가 적용된다.
    this == null || this.isBlank()
```
- 널이 될 수 있는 타입에 대한 확장을 정의하면 널이 될 수 있는 값에 대해 그 확장 함수를 호출할 수 있다.
  - 그 함수의 내부에서 this는 널이 될 수 있다. 따라서 명시적으로 널 여부를 검사해야 한다.

> 자바에서는 메서드 안의 this는 그 메소드가 호출된 수신 객체를 가리키므로 항상 널이 아니다.
>   
> 코틀린에서는 널이 될 수 있는 타입의 확장 함수 안에서는 this가 널이 될 수 있다는 점이 자바와 다르다.


- let 함수도 널이 될 수 있는 타입의 값에 대해 호출할 수 있지만 let은 this가 널인지 검사하지 않는다.
  - 널이 될 수 있는 타입의 값에 대해 안전한 호출을 사용하지 않고 let을 호출하면 람다의 인자는 널이 될 수 있는 타입으로 추론된다.
```kotlin
val person: Person? = ...
// 안전한 호출을 하지 않음. 따라서 "it"은 널이 될 수 있는 타입으로 취급됨
person.let { sendEmailTo(it) }
// 결과: Error: Type mismatch: inferred type is Person? but Person was expected
person?.let { sendEmailTo(it) } // 개선
```

> 직접 확장 함수를 작성한다면 그 확장 함수를 널이 될 수 있는 타입에 대해 정의할지 여부를 고민할 필요가 있다. 처음에는 널이 될 수 없는 타입에 대한 확장 함수를 정의하라. 나중에 대부분 널이 될 수 있는 타입에 대해 그 함수를 호출했다는 사실을 깨닫게 되면 확장 함수 안에서 널을 제대로 처리하게 하면(그 확장 함수를 사용하는 코드가 깨지지 않으므로) 안전하게 그 확장 함수를 널이 될 수 있는 타입에 대한 확장 함수로 바꿀 수 있다.

### 타입 파리미터의 널 가능성
- 코틀린에서는 함수나 클래스의 모든 타입 파라미터는 기본적으로 널이 될 수 있다. 널이 될 수 있는 타입을 포함하는 어떤 타입이라도 타입 파라미터를 대신할 수 있다.
- 따라서 파라미터 T를 클래스나 함수 안에서 타입 이름으로 사용하면 이름 끝에 물음표가 없더라도 T가 널이 될 수 있는 타입이다.
```kotlin
// 널이 될 수 있는 타입 파라미터 다루기
fun <T> printHashCode(t: T) {
    // "t"가 null이 될 수 있으므로 안전한 호출을 써야만 한다.
    println(t?.hashCode())
}

// "T"의 타입은 "Any?"로 추론된다.
printHashCode(null) // null
```
- printHashCode 호출에서 타입 파라미터 T에 대해 추론한 타입은 널이 될 수 있는 Any? 타입이다.
- t 파라미터의 타입 이름 T에는 물음표가 붙어있지 않지만 t는 null을 받을 수 있다.


- 타입 파라미터가 널이 아님을 확실히 하려면 널이 될 수 없는 타입 상한(upper bound)를 지정해야 한다. 이렇게 널이 될 수 없는 타입 상한을 지정하면 널이 될 수 있는 값을 거부하게 된다.
```kotlin
// 타입 파라미터에 대해 널이 될 수 없는 상한을 사용하기
fun <T: Any> printHashCode(t: T) { // 이제 "T"는 널이 될 수 없는 타입이다.
    println(t.hashCode())
}
printHashCode(null) // 널이 될 수 없는 타입의 파라미터에 널을 넘길 수 없다.
// Error: Type parameter bound for `T` is not satisfied
printHashCode(42) // 42
```

### 널 가능성과 자바
- 코틀린 컴파일러는 공개(public) 가시성인 코틀린 함수의 널이 아닌 타입인 파라미터와 수신 객체에 대한 널 검사를 추가해준다. 따라서 공개 가시성 함수에 널 값을 사용하면 즉시 예외가 발생한다.
  - 이런 파라미터 값 검사는 함수 내부에서 파라미터를 사용하는 시점이 아니라 함수 호출 시점에 이뤄진다.
```java
// 널 가능성 애노테이션이 없는 자바 클래스
public class Person {
  private final String name;
    
    public Person(String name) {
        this.name = name;
    }
    
    public String getName() {
        return name;
    }
}
```
- 코틀린 컴파일러는 String 타입의 널 가능성에 대해 전혀 알지 못한다. 따라서 널 가능성을 직접 처리해야만 한다.
```kotlin
// 널 검사를 통해 자바 클래스 접근하기
fun yellAtSafe(person: Person) {
    println((person.name ?: "Anyone").toUpperCase() + "!!!")
}

yellAtSafe(Person(null)) // ANYONE!!!
```
- ! 표기 : String! 타입의 널 가능성에 대해 아무 정보도 없다는 뜻
```kotlin
val i: Int = person.name
// 결과: Error: Type mismatch: inferred type is String! but Int was expected

// 자바 프로퍼티를 널이 될 수 있는 타입으로 볼 수 있다.
val s: String? = person.name
// 자바 프로퍼리트를 널이 될 수 없는 타입으로 볼 수 있다.
val s1: String = person.name
```

#### 상속
```java
// String 파라미터가 있는 자바 인터페이스
interface StringProcessor {
    void process(String value);
}
```
```kotlin
// 자바 인터페이스를 여러 다른 널 가능성으로 구현하기
class StringPrinter: StringProcessor {
    override fun process(value: String) {
        print(value)
    }
}

class NullableStringPrinter : StringProcessor {
    override fun process(value: String?) {
        if (value != null) {
            print(value)
        }
    }
}
```

## 코틀린의 원시 타입
- 코틀린은 원시 타입과 래퍼 타입을 구분하지 않는다.

### 원시 타입: Int, Boolean 등
- 원시 타입(Int 등)의 변수에는 그 값이 직접 들어가지만, 참조 타입(String 등)의 변수에는 메모리상의 객체 위치가 들어간다.
- 원시 타입의 값을 더 효율적으로 저장하고 여기저기 전달할 수 있다. 하지만 그런 값에 대해 메소드를 호출하거나 컬렉션에 원시 타입 값을 담을 수 없다.
- 자바는 참조 타입이 필요한 경우 특별한 래퍼 타입(java.lang.Integer 등)으로 원시 타입 값을 감싸서 사용한다.


- 코틀린은 원시 타입과 래퍼 타입을 구분하지 않으므로 항상 같은 타입을 사용한다.
```kotlin
val i: Int = 1
val list: List<Int> = listOf(1, 2, 3)
```
- 코틀린에서는 숫자 타입 등 원시 타입의 값에 대해 메소드를 호출할 수 있다.
```kotlin
fun showProgress(progress: Int) {
    val percent = progress.coerceIn(0, 100)
    println("We're ${percent}% done!")
}

showProgress(146) // We're 100% done!
```

### 널이 될 수 있는 원시 타입: Int?, Boolean? 등
- null 참조를 자바의 참조 타입의 변수에만 대입할 수 있기 때문에 널이 될 수 있는 코틀린 타입은 자바 원시 타입으로 표현할 수 없다.
  - 따라서 코틀린에서 널이 될 수 있는 원시 타입을 사용하면 그 타입은 자바의 래퍼 타입으로 컴파일된다.
```kotlin
// 널이 될 수 있는 원시 타입
data class Person(val name: String, val age: Int? = null) {
    fun isOlderThan(other: Person): Boolean? {
        if (age == null || other.age == null)
            return null
        return age > other.age
    }
}

println(Person("Sam", 35).isOlderThan(Person("Amy", 42))) // false
println(Person("Sam", 35).isOlderThan(Person("Jane"))) // null
```
- [제네릭 클래스](https://slow-and-steady-wins-the-race.tistory.com/104)의 경우 래퍼 타입을 사용한다. 어떤 클래스의 타입 인자로 원시 타입을 넘기면 코틀린은 그 타입에 대한 박스 타입을 사용한다.

### 숫자 변환
- 코틀린은 한타입의 숫자를 다른 타입의 숫자로 자동 변환하지 않는다. 결과 타입이 허용하는 숫자의 범위가 원래 타입의 범위보다 넓은 경우조차도 자동 변환은 불가능하다.
```kotlin
val i = 1
val l: Long = i // Error: type mismatch

// 직접 변환 메소드를 호출
val i = 1
val l: Long = i.toLong()
```
- 코틀린은 모든 원시 타입(단 Boolean 제외)에 대한 변환 함수를 제공한다. 양방향 변환 함수가 모두 제공한다.
```kotlin
val x = 1
println(x in listOf(1L, 2L, 3L)) // false
println(x.toLong() in listOf(1L, 2L, 3L)) // true
```
- 숫자 리터럴을 사용할 때는 보통 변환 함수를 호출할 필요가 없다.
```kotlin
fun foo(l: Long) = println(l)

val b: Byte = 1 // 상수 값은 적절한 타입으로 해석된다.
val l = b + 1L // +는 Byte와 Long을 인자로 받을 수 있다.
foo(42) // 컴파일러는 42를 Long 값으로 해석한다.
```
- 코틀린 산술 연산자에서도 자바와 같이 숫자 연산 시 값 넘침(overflow)이 발생할 수 있다. 코틀린은 값 넘침을 검사하느라 추가 비용을 들이지 않는다.

### Any, Any?: 최상위 타입
- 자바에서 Objectr가 클래스 계층의 최상위 타입이듯 코틀린에는 Any 타입이 모든 널이 될 수 없는 타입의 조상 타입이다.
- 하지만 자바에서는 참조 타입만 Object를 정점으로 하는 타입 계층에 포함되며, 원시 타입은 그런 계층에 들어있지 않다.
  - 즉 자바에서 Object 타입의 객체가 필요할 경우 int와 같은 원시 타입을 java.lang.Integer 같은 래퍼 타입으로 감싸야 한다는 뜻이다.


- 코틀린에서는 Any가 Int 등의 원시 타입을 포함한 모든 타입의 조상 타입이다.
- 자바와 마찬가지로 코틀린에서도 원시 타입 값을 Any 타입의 변수에 대입하면 자동으로 값을 객체로 감싼다.
```kotlin
val answer: Any = 42 // Any가 참조 타입이기 때문ㅇ 42가 박싱된다. 
```

### Unit 타입: 코틀린의 void
- 반환 타입 선언 없이 정의한 블록이 복문인 함수
```kotlin
fun f(): Unit { ... }

fun f() { ... }
```
- 코틀린 함수의 반환 타입이 Unit으고 그 함수가 제네릭 함수를 오버라이드하지 않는다면 그 함수는 내부에서 자바 void 함수로 컴파일된다.
- Unit은 모든 기능을 갖는 일반적인 타입이며, void와 달리 Unit을 타입 인자로 쓸 수 있다.
```kotlin
inteface Processor<T> {
    fun progress(): T
}

class NoResultProcess : Processor<Unit> {
    override fun process() { // Unit을 반환하지만 타입을 지정할 필요가 없다.
        // 업무 처리 코드
        // 여기서 return을 명시할 필요가 없다.
    }
}
```
- 인터페이스의 시그니처는 process 함수가 어떤 값을 반환하라고 요구한다. 
  - Unit 타입도 Unit 값을 제공하기 때문에 메소드에서 Unit 값을 반환하는 데는 아무 문제가 없다.
- NoResultProcessor에서 명시적으로 Unit을 반환할 필요는 없다. 컴파일러가 묵시적으로 return Unit을 넣어준다.
- 함수형 프로그래밍에서 전통적으로 Unit은 '단 하나의 인스턴스만 갖는 타입'을 의미해왔고 바로 그 유일한 인스턴스의 유무가 자바 void와 코틀린 Unit을 구분하는 가장 큰 차이다.

### Nothing 타입: 이 함수는 결코 정상적으로 끝나지 않는다
- 코틀린에는 결코 성공적으로 값을 돌려주는 일이 없으므로 '반환 값'이라는 개념 자체가 의미 없는 함수가 일부 존재한다.
```kotlin
fun fail(message: String): Nothing {
    throw IllegalStateException(message)
}

fail("Error occurred")
// 결과: java.lang.IllegalStateException: Error occurred
```
```kotlin
fun <T> throwError(message: String): T {
    throw IllegalArgumentException(message)
}

fun main() {
    try {
        val result: String = throwError("Error message")
        // 이 부분은 실행되지 않음. throwError에서 예외가 던져짐.
        println(result)
    } catch (e: IllegalArgumentException) {
        println("Caught an exception: ${e.message}")
    }
}
```
- Nothing 타입은 아무 값도 포함하지 않는다. 따라서 Nothing은 함수의 반환 타입이나 반환 타입으로 쓰일 타입 파라미터로만 쓸 수 있다.
- 그 외의 다른 용도로 사용하는 경우 Nothing 타입의 변수를 선언하더라도 그 변수에 아무 값도 저장할 수 없으므로 아무 으미도 없다.
- Nothing을 반환하는 함수를 엘비스 연산자의 우항에 사용해서 전체 조건을 검사할 수 있다.
```kotlin
val address = company.address ?: fail("No address")
println(address.city)
```

