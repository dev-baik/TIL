# 람다로 프로그래밍
> - 람다 식과 멤버 참조
> - 함수형 스타일로 컬렉션 다루기
> - 시퀀스: 지연 컬렉션 연산
> - 자바 함수형 인터페이스를 코틀린에서 사용
> - 수신 객체 지정 람다 사용

- - -

- 람다 식 또는 람다 : 다른 함수에 넘길 수 있는 작은 코드 조각
- 람다를 사용하면 쉽게 공통 코드 구조를 라이브러리 함수로 뽑아낼 수 있다. 코틀린 표준 라이브러리는 람다를 아주 많이 사용한다. 람다를 자주 사용하는 경우로 컬렉션 처리를 들 수 있다.
- 수신 객체 람다(lambda with receiver)는 특별한 람다로, 람다 선언을 둘러싸고 있는 환경과는 다른 상황에서 람다 본문을 실행할 수 있다.

## 람다 식과 멤버 참조

> 💡 익명 함수를 간단히 표현하는 함수로서, 값처럼 여러 곳에 전달할 수 있는 동작의 모음이다.

### 람다 소개: 코드 블록을 함수 인자로 넘기기
- "이벤트가 발생하면 이 핸들러를 실행하자"나 "데이터 구조의 모든 원소에 이 연산을 적용하자"와 같은 생각을 코드로 표현하기 위해 일련의 동작을 변수에 저장하거나 다른 함수에 넘겨야 하는 경우가 자주 있다.


- 자바는 무명 내부 클래스를 사용하여 코드를 함수에 넘기거나 변수에 저장할 수 있기는 하지만 상당히 번거롭다.
```java
// 무명 내부 클래스로 리스너 구현하기
button.setOnClickListener(new OnClickListener() { 
    @Override
    public void onClick(View view) {
        /* 클릭 시 수행할 동작 */
    }   
})
```
```kotlin
button.setOnClickListener(object : View.OnClickListener {
    override fun onClick(view: View) {
        /* 클릭 시 수행할 동작 */
    }
})
```
- 함수형 프로그래밍에서는 함수를 값처럼 다루는 접근 방법을 택함으로써 이 문제를 해결한다.
  - <span style='background-color: #fff5b1'/>클래스를 선언하고 그 클래스의 인스턴스를 함수에 넘기는 대신 함수형 언어에서는 함수를 직접 다른 함수에 전달할 수 있다.
  - 람다 식을 사용하면 코드가 더욱 더 간결해진다. 함수를 선언할 필요가 없고 코드 블록을 직접 함수의 인자로 전달할 수 있다.
```kotlin
// 람다로 리스너 구현하기
button.setOnClickListener { /* 클릭 시 수행할 동작 */ }
```

### 람다와 컬렉션
- 컬렉션을 직접 검색하기
```kotlin
data class Person(val name: String, val age: Int)

fun findTheOldest(people: List<Person>) {
    var maxAge = 0 // 가장 많은 나이를 저장한다.
    var theOldest: Person? = null // 가장 연장자인 사람을 저장한다.
    for (person in people) {
        if(person.age > maxAge) {
            maxAge = person.age
            theOldest = person
        }
    }
    println(theOldest)
}

val people = list(Person("Alice", 29), Person("Bob", 31))
findTheOldest(people) // Person(name=Bob, age=31)
```
- 람다를 사용해 컬렉션 검색하기
```kotlin
val people = listOf(Person("Alice" 29), Person("Bob", 31))
println(people.maxBy { it.age }) // 나이 프로퍼티를 비교해서 값이 가장 큰 원소 찾기
// 결과: Person(name=Bob, age=31)
```
- 모든 컬렉션에 대해 maxBy 함수를 호출할 수 있다.
- maxBy는 가장 큰 원소를 찾기 위해 비교에 사용할 값을 돌려주는 함수를 인자로 받는다.
- 중괄호로 둘러싸인 코드 `{ it.age }`는 비교에 사용할 값을 돌려주는 함수다.
- 이 예제에서는 컬렉션의 원소가 Person 객체이므로 이 함수가 반환하는 값은 Person 객체의 age 필드에 저장된 나이 정보다.


- 멤버 참조를 사용해 컬렉션 검색하기
```kotlin
people.maxBy(Person::age)
```

### 람다 식의 문법
- 람다는 값처럼 여기저기 전달할 수 있는 동작의 모음이다. 람다를 따로 선언해서 변수에 저장할 수도 있다. 함수에 인자로 넘기면서 바로 람다를 정의하는 경우가 대부분이다.
- 코틀린 람다 식은 항상 중괄호로 둘러싸여 있다. 인자 목록 주변에 괄호가 없다는 사실을 기억하라. 화살표(->)가 인자 목록과 람다 본문을 구분해준다.
```kotlin
val sum = { x: Int, y: Int -> x + y }
println(sum(1, 2)) // 3

{ println(42) }() // 42
```

> 굳이 람다를 만들자마자 바로 호출하느니 람다 본문을 직접 실행하는 편이 낫다. 이렇게 코드의 일부분을 블록으로 둘러싸 실행할 필요가 있다면 run을 사용한다.
 
> ✅ **람다 본문을 직접 실행하는 편이 낫다?** : 람다를 만들면 객체가 생성되어 가비지 컬렉션의 대상이 된다. 그러나 바로 실행하면 객체가 생성되지 않으므로 가비지 컬렉션 부하가 감소한다.

- [run](https://velog.io/@dev-baik/%EB%B2%94%EC%9C%84-%EC%A7%80%EC%A0%95-%ED%95%A8%EC%88%98#run-%ED%95%A8%EC%88%98)은 인자로 받은 람다를 실행해주는 라이브러리 함수다.
```kotlin
run { println(42) } // 42 
```
- 코틀린이 코드를 줄여 쓸 수 있게 제공했던 기능을 제거하고 정식으로 람다를 작성하면 다음과 같다.
```kotlin
people.maxBy({ p: Person -> p.age }) 
```
- 여기서 어떤 일이 벌어지고 있는지 더 명확히 알 수 있다. 하지만 이 코드는 번잡하다.
  - 구분자가 너무 많이 쓰여서 가독성이 떨어지고, 컴파일러가 문맥으로부터 유추할 수 있는 인자 타입을 굳이 적을 필요는 없다.
  - 인자가 단 하나뿐인 경우 굳이 인자에 이름을 붙이지 않아도 된다.

> 코틀린에서는 함수 호출 시 맨 뒤에 있는 인자가 람다 식이라면 그 람다를 괄호 밖으로 빼낼 수 있다는 문법 관습이 있다.

- 이 예제에서는 람다가 유일한 인자이므로 마지막 인자이기도 하다. 따라서 괄호 뒤에 람다를 둘 수 있다.
```kotlin
people.maxBy() { p: Person -> p.age } 
```
- 람다가 어떤 함수의 유일한 인자이고 괄호 뒤에 람다를 썻다면 호출 시 빈 괄호를 없애도 된다.
```kotlin
people.maxBy { p: Person -> p.age } 
```
- 인자가 여럿 있는 경우에는 람다를 밖으로 빼낼 수도 있고 람다를 괄호 안에 유지해서 함수의 인자임을 분명히 할 수 있다. 두 방식 모두 정당하다.
- 둘 이상의 람다를 인자로 받는 함수라고 해도 인자 목록의 맨 마지막 람다만 밖으로 뺄 수 있다. 따라서 그런 경우에는 괄호를 사용하는 일반적인 함수 호출 구문을 사용하는 편이 낫다.
```kotlin
fun operateOnNumbers(a: Int, b: Int, operation1: (Int, Int) -> Int, operation2: (Int, Int) -> Unit) {
    val result = operation1(a, b)
    println("Result of the operation: $result")
  
    operation2(a, b)
}

fun main() {
    // trailing lambda syntax를 이용하여 두 번째 람다를 함수 호출 밖으로 뺄 수 있음
    operateOnNumbers(5, 3 ,{ x, y -> x + y }) { x, y -> println("Numbers: $x, $y") }
    
    // 일반적인 함수 호출 구문을 사용하여 두 람다를 전달
    operateOnNumbers(5, 3,
        { x, y -> x + y },
        { x, y -> println("Numbers: $x, $y") }
    )
} 
```

- 표준 라이브러리의 joinToString은 맨 마지막 인자로 함수를 더 받는다는 차이가 있다.
  - 리스트의 원소를 toString이 아닌 다른 방식을 통해 문자열로 변환하고 싶은 경우 이 인자를 활용한다.
```kotlin
// 이름 붙인 인자를 사용해 람다 넘기기
val people = listOf(Person("이몽룡", 29), Person("성춘향", 31))
val name = people.joinToString(separator = " ",
                    transform = { p: Person -> p.name })
println(names) // 이몽룡 성춘향
```
- 이름 붙인 인자를 사용해 람다를 넘김으로써 람다를 어떤 용도로 쓰는 지 더 명확히 했다.
```kotlin
// 람다를 괄호 밖에 전달하기
people.joinToString(" ") { p.person -> p.name }
```
- 더 간결하지만 람다의 용도를 분명히 알아볼 수는 없다.
```kotlin
// 람다 파라미터 타입 제거하기
people.maxBy { p: Person -> p.age } // 파라미터 타입을 명시
people.maxBy { p -> p.age } // 파라마티 타입을 생략(컴파일러가 추론)
```
- 로컬 변수처럼 컴파일러는 람다 파라미터의 타입도 추론할 수 있다. 따라서 파라미터 타입을 명시할 필요가 없다.
```kotlin
// 디폴트 파라미터 이름 it 사용하기
people.maxBy { it.age }
```
> it을 사용하는 관습은 코드를 아주 간단하게 만들어준다. 하지만 이를 남용하면 안된다. 특히 람다 안에 람다가 중첩되는 경우 각 람다의 파라미터를 명시하는 편이 낫다. 파라미터를 명시하지 않으면 각각의 it이 가리키는 파라미터가 어떤 람다에 속했는지 파악하기 어려울 수 있다. 문맥에서 람다 파라미터의 의미나 파라미터의 타입을 쉽게 알 수 없는 경우에도 파라미터를 명시적으로 선언하면 도움이 된다.

- 람다를 변수에 저장할 때는 파라미터의 타입을 추론할 문맥이 존재하지 않는다. 따라서 파라미터 타입을 명시해야 한다.
```kotlin
val getAge = { p: Person -> p.age }
people.maxBy(getAge)
```
- 본문이 여러 줄로 이뤄진 경우 본문의 맨 마지막에 있는 식이 람다의 결과 값이 된다.
```kotlin
val sum = { x: Int, y: Int ->
    println("Computing the sum of $x and $y...")
    x + y
}
println(sum(1, 2))

// Computingthe sum of 1 and 2...
// 3
```

### 현재 영역에 있는 변수에 접근
- 람다를 함수 안에서 정의하면 파라미터뿐 아니라 람다 정의의 앞에 선언된 로컬 변수까지 람다에서 모두 사용할 수 있다.
- forEach는 컬렉션의 모든 원소에 대해 람다를 호출해준다. 일반적으로 for 루프보다 훨씬 간결하지만 그렇다고 다른 장점이 많지는 않아 기존 for 루프를 forEach로 모두 바꿀 필요는 없다.
```kotlin
// 함수 파라미터를 람다 안에서 사용하기
fun printMessagesWithPrefix(messages: Collection<String>, prefix: String) {
    messages.forEach { // 각 원소에 대해 수행할 작업을 람다로 받는다.
        println("$prefix $it") // 람다 안에서 함수의 "prefix" 파라미터를 사용한다.
    }
}

val errors = listOf("403 Forbidden", "404 Not Found")
printMessagesWithPrefix(errors, "Error:")
// Error: 403 Forbidden
// Error: 404 Not Found
```
- 코틀린에서는 자바와 달리 람다에서 람다 밖 함수에 있는 파이널이 아닌 변수에 접근할 수 있고, 그 변수를 변경할 수 있다.
```kotlin
fun printProblemCount(responses: Collection<String>) {
    var clientErrors = 0
    var serverErrors = 0
    
    responses.forEach {
        if (it.startsWith("4")) {
            clientErrors++
        } else if (it.startsWith("5")) {
            serverErrors++
        }
    }
    println("$clientErrors client errors, $serverErrors server errors")
}

val response = listOf("200 OK", "418 I'm a teapot", "500 Internal Server Error")
printProblemCount(responses) // 1 client errors, 1 server errors
```
- 이 예제의 prefix, clientErrors, serverErrors와 같이 람다 안에서 사용하는 외부 변수를 `람다가 포획한 변수`라고 부른다.
- 기본적으로 함수 안에 정의된 로컬 변수의 생명주기는 함수가 반환되면 끝난다. 하지만 <span style='background-color: #fff5b1'/>어떤 함수가 자신의 '로컬 변수를 포획한 람다'를 반환하거나 다른 변수에 저장한다면 로컬 변수의 생명주기와 함수의 생명주기가 달라질 수 있다.
  - **클로저** : 람다식으로 표현된 내부 함수에서 외부 범위에 선언된 변수에 접근할 수 있는 구조다. 이 때 람다식 안에 있는 외부 변수는 값을 유지하기 위해 '클로저가 포획한 변수'라고 부른다.
    - 실행 시점에서 람다식의 모든 참조가 포함된 닫힌(closed) 객체를 람다 코드와 함께 저장한다. 이때 이러한 데이터 구조를 클로저(closure)라고 부르는 것이다. 
```kotlin
fun createClosure(): () -> Unit {
    var counter = 0
  
    val closure: () -> Unit = {
        println("Counter inside lambda: $counter")
        counter++
    }
  
    return closure
}

fun main() {
    val closureInstance = createClosure()
  
    closureInstance() // Counter inside lambda: 0
    closureInstance() // Counter inside lambda: 1
  
    // 외부 변수 counter의 생명주기가 끝났지만, 여전히 람다가 참조하고 있음
    closureInstance() // Counter inside lambda: 2
}
```

- <span style='background-color: #fff5b1'/>람다를 이벤트 핸들러나 다른 비동기적으로 실행되는 코드로 활용하는 경우 함수 호출이 끝난 다음에 로컬 변수가 변경될 수 있다.
```kotlin
fun main() = runBlocking {
    var counter = 0
  
    // 비동기적으로 값을 변경하는 코루틴 런치
    val job = launch {
        delay(1000) // 시간이 오래 걸리는 작업 시뮬레이션
        counter = 42
    }
  
    // 함수 호출이 끝난 후에도 코루틴이 실행되고 로컬 변수가 변경될 수 있음
    println("Counter before job join: $counter")
  
    job.join() // 코루틴이 끝날 때까지 대기
  
    println("Counter after job join: $counter")
}

// Counter before job join: 0
// Counter after job join: 42
```
```kotlin
fun tryToCountButtonClicks(button: Button): Int {
    var clicks = 0
    button.onClick { clicks++ }
    return clicks
}
```
- 이 함수는 항상 0을 반환한다. onClick 핸들러는 호출될 때마다 clicks의 값을 증가시키지만 그 값의 변경을 관찰할 수는 없다. 핸들러는 tryToCountButtonClicks가 clicks를 반환한 다음에 호출되기 때문이다.
  - 이 함수를 제대로 구현하려면 클릭 횟수를 세는 카운터 변수를 함수의 내부가 아니라 클래스의 프로퍼티나 전역 프로퍼티 등의 위치로 빼내서 나중에 변수 변화를 살펴볼 수 있게 해야 한다.

### 멤버 참조

> 💡 특정 객체의 메소드나 프로퍼티를 참조하여 호출할 수 있는 함수를 생성하는 기능이다. 이를 통해 해당 멤버에 대한 참조를 함수처럼 전달하거나 저장할 수 있다.

- 코틀린에서는 함수를 값으로 바꿀 수 있는 `이중 콜론(::)`을 사용한다.
```kotlin
val getAge = { person: Person -> person.age }
```
```kotlin
val getAge = Person::age 
```
- `::`를 사용하는 식을 `멤버 참조`라고 부른다. 멤버 참조는 프로퍼티나 메소드를 단 하나만 호출하는 함수 값을 만들어준다.
- 클래스 이름과 여러분이 참조하려는 멤버(프로퍼티나 메소드) 이름 사이에 위치한다.
- 참조 대상이 함수인지 프로퍼티인지와는 관계없이 멤버 참조 뒤에는 괄호를 넣으면 안된다.
- 멤버 참조는 그 멤버를 호출하는 람다와 같은 타입이다
```kotlin
people.maxBy(Person::age)
people.maxBy { p -> p.age }
people.maxBy { it.age }
```
- 최상위에 선언된(그리고 다른 클래스의 멤버가 아닌) 함수나 프로퍼티를 참조할 수도 있다.
```kotlin
val salute = "Salute!"

fun greet(name: String): String {
    return "Hello, $name!"
}

fun main() {
    val propertyReference: () -> String = ::salute
    val result1 = propertyReference()
  
    println(result1)  // Salute!
  
    val greetFunction: (String) -> String = ::greet
    val result2 = greetFunction("Alice")
  
    println(result2)  // Hello, Alice!
}
```
- 클래스 이름을 생략하고 ::로 참조를 바로 시작한다. ::salute라는 멤버 참조를 run 라이브러리 함수에 넘긴다(run은 인자로 받은 람다를 호출한다).
```kotlin
fun salute() = println("Salute!")

fun main() {
    run(::salute) // Salute!
}
```
- 람다가 인자가 여럿인 다른 함수한테 작업을 위임하는 경우, 람다를 정의하지 않고 직접 위임 함수에 대한 참조를 제공하면 편리하다
```kotlin
data class Person(val name: String, val email: String)

fun sendEmail(person: Person, message: String) {
    println("Sending email to ${person.name} at ${person.email}: $message")
}

fun delegateOperation(person: Person, message: String, operation: (Person, String) -> Unit) {
    operation(person, message)
}

fun main() {
    // 람다를 사용하여 sendEmail 함수에 작업을 위임
    val action: (Person, String) -> Unit = { person, message ->
        sendEmail(person, message)
    }
  
    // 멤버 참조를 사용하여 sendEmail 함수에 작업을 위임
    val nextAction: (Person, String) -> Unit = ::sendEmail
  
    // Person 객체 생성
    val alice = Person("Alice", "alice@example.com")
  
    // 작업을 위임한 함수 호출
    delegateOperation(alice, "Hello from lambda", action)
    delegateOperation(alice, "Hello from member reference", nextAction)
} 
```
```kotlin
data class Person(val name: String, val email: String)

fun sendEmail(person: Person, message: String) {
    println("Sending email to ${person.name} at ${person.email}: $message")
}

fun main() {
    // 람다를 사용하여 sendEmail 함수에 작업을 위임
    val action = { person: Person, message: String ->
        sendEmail(person, message)
    }
  
    // 멤버 참조를 사용하여 sendEmail 함수에 작업을 위임
    val nextAction = ::sendEmail
  
    // Person 객체 생성
    val alice = Person("Alice", "alice@example.com")
  
    // 작업을 위임한 함수 호출
    action(alice, "Hello from lambda")
    nextAction(alice, "Hello from member reference")
}

// Sending email to Alice at alice@example.com: Hello from lambda
// Sending email to Alice at alice@example.com: Hello from member reference 
```
- <span style='background-color: #fff5b1'/>생성자 참조를 사용하면 클래스 생성 작업을 연기하거나 저장해둘 수 있다. :: 뒤에 클래스 이름을 넣으면 생성자 참조를 만들 수 있다.
```kotlin
val createPerson = ::Person // "Person"의 인스턴스를 만드는 동작을 값으로 저장한다.
val p = createPerson("Alice", 29)
println(p) // Person(name=Alice, age=29)
```
- 확장 함수도 멤버 함수와 똑같은 방식으로 참조할 수 있다는 점을 기억하라.
```kotlin
fun Person.isAdult() = age >= 21
val predicate = Person::isAdult
```
- isAdult는 Person 클래스의 멤버가 아니고 확장 함수다. 그렇지만 isAdult를 호출할 때 person.isAdult()로 인스턴스 멤버 호출 구문을 쓸 수 있는 것처럼 Person::isAdult로 멤버 참조 구문을 사용해 이 확장 함수에 대한 참조를 얻을 수 있다.

#### 바운드 멤버 참조
- 코틀린 1.0에서는 클래스의 메소드나 프로퍼티에 대한 참조를 얻은 다음에 그 참조를 호출할 때 항상 인스턴스 객체를 제공해야 했다.
- 코틀린 1.1부터는 바운드 멤버 참조를 지원한다.
  - <span style='background-color: #fff5b1'/>바운드 멤버 참조를 사용하면 멤버 참조를 생성할 때 클래스 인스턴스를 함께 저장한 다음 나중에 그 인스턴스에 대해 멤버를 호출해준다. 따라서 호출 시 수신 대상 객체를 별도로 지정해 줄 필요가 없다.
```kotlin
val p = Person("Dmitry", 34)
val personsAgeFunction = Person::age
println(personsAgeFunction(p))

val dmitrysAgeFunction = p::age
println(dmitrysAgeFunction())
```

> personsAgeFunction은 인자가 하나(인자로 받은 사람의 나이를 반환)이지만, dmitrysAgeFunction은 인자가 없는(참조를 만들 때 p가 가리키던 사람의 나이를 반환) 함수라는 점에 유의하라.

- 코틀린 1.0에서는 p::age 대신에 { p.age }라고 직접 객체의 프로퍼티를 돌려주는 람다를 만들어야만 한다.

## 컬렉션 함수형 API
- 함수형 프로그래밍 스타일을 사용하면 컬렉션을 다룰 때 편리하다. 대부분의 작업에 라이브러리 함수를 활용할 수 있고 그로 인해 코드를 아주 간결하게 만들 수 있다.

### 필수적인 함수: filter와 map
- filter와 map은 컬렉션을 활용할 때 기반이 되는 함수다. 대부분의 컬렉션 연산을 이 두 함수를 통해 표현할 수 있다.

> 함수형 프로그래밍에는 람다나 다른 함수를 인자로 받거나 함수를 반환하는 함수를 `고차 함수(HOF, Higt Order Function)`라고 부른다. 고차함수는 기본 함수를 조합해서 새로운 연산을 정의하거나, 다른 고차 함수를 통해 조합된 함수를 또 조합해서 더 복잡한 연산을 쉽게 정의할 수 있다는 장점이 있다. 이런 식으로 고차 함수와 단순한 함수를 이리저리 조합해서 코드를 작성하는 기법을 `컴비네이터 패턴(combinator pattern)`이라 부르고, 컴비네이터 패턴에서 복잡한 연산을 만들기 위해 값이나 함수를 조합할 때 사용하는 고차 함수를 `컴비네이터`라고 부른다.

- filter 함수는 이터레이션하면서 주어진 람다에 각 원소를 넘겨서 람다가 true를 반환하는 원소만 모은다.
```kotlin
data class Person(val name: String, val age: Int)

val list = listOf(1, 2, 3, 4)
println(list.filter { it % 2 == 0 }) // [2, 4]
```
- 결과는 입력 컬렉션의 원소 중에서 주어진 술어를 만족하는 원소만으로 이뤄진 새로운 컬렉션이다.
```kotlin
val people = listOf(Person("Alice", 29), Person("Bob", 31))
println(people.filter { it.age > 30 }) // [Person(name=Bob, age=31)]
```

> ✅ **술어(predicate)** : 참/거짓을 반환하는 함수

- filter 함수는 컬렉션에서 원치 않는 원소를 제거하지만, 원소를 변환할 수 없다.
- map 함수는 주어진 람다를 컬렉션의 각 원소에 적용한 결과를 모아서 새 컬렉션을 만든다.
```kotlin
val list = listOf(1, 2, 3, 4)
println(list.map { it * it }) // [1, 4, 9, 16 ]
```
```kotlin
// 사람 이름의 리스트 출력
val people = listOf(Person("Alice", 29), Person("Bob", 31))
println(people.map { it.name }) // [Alice, Bob]
```
```kotlin
// 30살 이상인 사람의 이름 출력 // 멤버 참조 사용
// people.filter({it.age > 30}).map(Person::name)
people.filter { it.age > 30 }.map(Person::name)
```
```kotlin
// 가장 나이 많은 사람의 이름(들)
people.filter { it.age == people.maxBy(Person::age)!!.age }
```
- 단점 : 최댓값을 구하는 작업을 계속 반복
```kotlin
val maxAge = people.maxBy(Person::age)!!.age
people.filter { it.age == maxAge }
```

> 꼭 필요하지 않은 경우 굳이 계산을 반복하지 말라! 람다를 인자로 받는 함수에 람다를 넘기면 겉으로 볼 때는 단순해 보이는 식이 내부 로직의 복잡도로 인해 실제로는 엄청나게 불합리한 계산식이 될 수 때가 있다.

```kotlin
val numbers = mapOf(0 to "zero", 1 to "one")
println(numbers.mapValues { it.value.toUpperCase() }) {0=ZERO, 1=ONE}
```
- 맵의 경우 키와 값을 처리하는 함수가 따로 존재한다. filterKeys와 mapKeys는 키를 걸러 내거나 반환하고, filterValues와 mapValues는 값을 걸러 내거나 반환한다.

### all, any, count, find: 컬렉션에 술어 적용
- 컬렉션에 대해 자주 수행하는 연산으로 컬렉션의 모든 원소가 어떤 조건을 만족하는 지 판단하는(또는 그 변종으로 컬렉션 안에 어떤 조건을 만족하는 원소가 있는지 판단하는) all과 any 연산이 있다.
- count 함수는 조건을 만족하는 원소의 개수를 반환하며, find 함수는 조건을 만족하는 첫 번째 원소를 반환한다.
```kotlin
// 모든 원소가 이 술어를 만족하는 지 궁금하다면 all 함수를 쓴다.
val canBeInClub27 = { p: Person -> p.age <= 27 }

val people = listOf(Person("Alice", 27), Person("Bob", 31))
println(people.all(canBeInClub27)) // false
```
```kotlin
// 술어를 만족하는 원소가 하나라도 있는지 궁금하다면 any를 쓴다.
println(people.any(canBeInClub27)) // true 
```
- 어떤 조건에 대해 !all을 수행한 결과와 그 조건의 부정에 대해 any를 수행하는 결과는 같다(드 모르강의 법칙). 또 어떤 조건에 대해 !any를 수행한 결과와 그 조건의 부정에 대해 all을 수행한 결과도 같다.
```kotlin
val list = listOf(1, 2, 3)
println(!list.all { it == 3 }) // true

// 가독성을 높이려면 any와 all 앞에 !를 붙이지 않는 편이 낫다.
println(list.any { it != 3 }) // true
```
```kotlin
// 술어를 만족하는 원소의 개수를 구하려면 count를 사용한다.
val people = listOf(Person("Alice", 27), Person("Bob", 31))
println(people.count(canBeInClub27)) // 1
```

> **함수를 적재적소에 사용하라: count와 size**
> 
> count가 있다는 사실을 잊어버리고, 컬렉션을 필터링한 결과의 크기를 가져오는 경우가 있다.
> ```kotlin
> println(people.filter(canBeInClub27).size) // 1
> ```
> 하지만 이렇게 처리하면 조건을 만족하는 모든 원소가 들어가는 `중간 컬렉션`이 생긴다. 반면 count는 조건을 만족하는 원소의 개수만을 추적하지 조건을 만족하는 원소를 따로 저장하지 않는다. 따라서 count가 훨씬 더 효율적이다.

```kotlin
// 술어를 만족하는 원소를 하나 찾고 싶으면 find 함수를 사용한다.
val people = listOf(Person("Alice", 27), Person("Bob", 31))
println(people.find(canBeInClub27)) // Person(name=Alice, age=27)
```
- 이 식은 조건을 만족하는 원소가 하나라도 있는 경우 가장 먼저 조건을 만족한다는 확인된 원소를 반환하며, 만족하는 원소가 전혀 없는 경우 null을 반환한다.
- 조건을 만족하는 원소가 없으면 null이 나온다는 사실을 더 명확히 하고 싶다면 `firstOrNull`을 쓸 수 있다.

### groupBy: 리스트를 여러 그룹으로 이뤄진 맵으로 변경
```kotlin
val people = listOf(Person("Alice", 27), Person("Bob", 31), Person("Carol", 31))
println(people.groupBy { it.age })

// {29=[Person(name=Bob, age=29)],
// 31=[Person(name=Alice, age=31), Person(name=Carol, age=31)]}
```
- 각 그룹은 리스트다. 따라서 groupBy의 결과 타입은 `Map<Int, List<Person>>`이다. 필요하면 이 맵을 mapKeys나 mapValues 등을 사용해 변경할 수 있다.
```kotlin
val list = listOf("a", "ab", "b")
println(list.groupBy(String::first)) // {a=[a, ab], b=[b]}
```
- first는 String의 멤버가 아니라 확장 함수지만, 여전히 멤버 참조를 사용해 first에 접근할 수 있다.

### flapMap과 flatten: 중첩된 컬렉션 안의 원소 처리
```kotlin
class Book(val title: String, val authors: List<String>)

books.flatMap { it.authors }.toSet() // books 컬렉션에 있는 책을 쓴 모든 저자의 집합
```
- flatMap 함수는 먼저 인자로 주어진 람다를 컬렉션의 모든 객체에 적용하고(또는 매핑하기(map)) 람다를 적용한 결과 얻어지는 여러 리스트를 한 리스트로 한데 모은다(또는 펼치기(flatten)).
```kotlin
val strings = listOf("abc", "def")
println(strings.flatMap { it.toList() }) // [a, b, c, d, e, f]
```
- toList 함수를 문자열에 적용하면 그 문자열에 속한 모든 문자로 이뤄진 리스트가 만들어진다.
```kotlin
val books = listOf(Book("Thursday Next", listOf("Jasper Fforde")),
                Book("Mort", listOf("Terry Pratchett")), 
                Book("Good Omens", listOf("Terry Pratchett", "Neil Gaiman")))

println(books.flatMap { it.authors }.toSet())
// 결과: [Jasper Fforde, Terry Pratchett, Neil Gaiman]
```
- toSet은 flatMap의 결과 리스트에서 중복을 없애고 집합을 만든다.
- 특별히 반환해야 할 내용이 없어 리스트의 리스트를 평평하게 펼치기만 하면 되는 경우 listOfLists.flatten()처럼 flatten 함수를 사용할 수 있다.

## 지연 계산(lazy) 컬렉션 연산
- map이나 filter 같은 몇가지 컬렉션 함수들은 결과 컬렉션을 즉시(eagerly) 생성한다.
  - <span style='background-color: #fff5b1'/>즉, 컬렉션 함수를 연쇄하면 매 단계마다 계산 중간 결과를 새로운 컬렉션에 임시로 담는다는 말이다.
- **시퀀스**를 사용하면 중간 임시 컬렉션을 사용하지 않고도 컬렉션 연산을 연쇄할 수 있다.
```kotlin
people.map(Person::name).filter { it.startsWith("A") } 
```
- 코틀린 표준 라이브러리 참조 문서에는 filter와 map이 리스트를 반환한다고 써 있다. 이는 이 연쇄 호출이 리스트를 2개 만든다는 뜻이다.
  - 한 리스트는 filter의 결과를 담고, 다른 하나는 map의 결과를 담는다.
- 이를 더 효율적으로 만들기 위해서는 각 연산이 컬렉션을 직접 사용하는 대신 시퀀스를 사용하게 만들어야 한다.
```kotlin
people.asSequence() // 원본 컬렉션을 시퀀스로 변환한다.
    .map(Person::name)
    .filter { it.startsWith("A") }
    .toList() // 결과 시퀀스를 다시 리스트로 변환한다.
```
- 중간 결과를 저장하는 컬렉션이 생기지 않기 때문에 원소가 많은 경우 성능이 눈에 띄게 좋아진다.


- 코틀린 지연 계산 시퀀스는 Sequence 인터페이스에서 시작한다. 이 인터페이스는 단지 한 번에 하나씩 열거될 수 있는 원소의 시퀀스를 표현할 뿐이다.
- Sequence 안에는 iterator라는 단 하나의 메소드가 있다. 그 메소드를 통해 시퀀스로부터 원소 값을 얻을 수 있다.
- Sequence 인터페이스의 강점은 그 인터페이스 위에 구현된 연산이 계산을 수행하는 방법 때문에 생긴다.
  - <span style='background-color: #fff5b1'/>시퀀스의 원소는 필요할 때 비로소 계산된다. 따라서 중간 결과를 저장하지 않고도 연산을 연쇄적으로 적용해서 효율적으로 계산을 수행할 수 있다.
- asSequence 확장 함수를 호출하면 어떤 컬렉션이든 시퀀스로 바꿀 수 있다. 시퀀스를 리스트로 만들 때는 toList를 사용한다.

> **왜 시퀀스를 다시 컬렉션으로 되돌려야 할까? 컬렉션보다 시퀀스가 훨씬 더 낫다면 그냥 시퀀스를 쓰는 편이 낫지 않을까?**
> 
> - 항상 그렇지는 않다. 시퀀스의 원소를 차례로 이터레이션해야 한다면 시퀀스를 직접 써도 된다. 하지만 시퀀스 원소를 인덱스를 사용해 접근하는 등의 다른 API 메소드가 필요하다면 시퀀스를 리스트로 변환해야 한다.

> 큰 컬렉션에 대해서 연산을 연쇄시킬 때는 시퀀스를 사용하는 것을 규칙으로 삼아라.
> 
> 8장에서는 중간 컬렉션을 생성함에도 불구하고 코틀린에서 즉시 계산 컬렉션에 대한 연산이 더 효율적인 이유를 설명한다. 하지만 컬렉션에 들어있는 원소가 많으면 중간 원소를 재배열하는 비용이 커지기 때문에 지연 계산이 더 낫다.

### 시퀀스 연산 실행: 중간 연산과 최종 연산
- 시퀀스에 대한 연산은 중간 연산과 최종 연산으로 나뉜다.
  - 중간 연산은 다른 시퀀스를 반환한다. 그 시퀀스는 최초 시퀀스의 원소를 변환하는 방법을 안다.
  - 최종 연산은 결과를 반환한다. 결과는 최초 컬렉션에 대해 변환을 적용한 시퀀스로부터 일련의 계산을 수행해 얻을 수 있는 컬렉션이나 원소, 숫자 또는 객체다.
- <span style='background-color: #fff5b1'/>중간 연산은 항상 지연 계산된다.
```kotlin
listOf(1, 2, 3, 4).asSequence()
            .map { print("map($it) "); it * it }
            .filter { print("filter($it) "); it % 2 == 0 }
```
- 이 코드를 실행하면 아무 내용도 출력되지 않는다. 이는 map과 filter 변환이 늦춰져서 결과를 얻을 필요가 있을 때(즉 최종 연산이 호출될 때) 적용된다는 뜻이다.
```kotlin
listOf(1, 2, 3, 4).asSequence()
            .map { print("map($it) "); it * it }
            .filter { print("filter($it) "); it % 2 == 0 }
            .toList()

// 결과: map(1) filter(1) map(2) filter(4) map(3) filter(9) map(4) filter(16)
```
- 최종 연산을 호출하면 연기됐던 모든 계산이 수행된다.
- 직접 연산을 구현한다면 map 함수를 각 원소에 대해 먼저 수행해서 새 시퀀스를 얻고, 그 시퀀스에 대해 다시 filter를 수행할 것이다. 컬렉션에 대한 map과 filter는 그런 방식으로 작동한다.
- 시퀀스의 경우 모든 연산은 각 원소에 대해 순차적으로 적용된다. 즉 첫 번째 원소가 (변환된 다음에 걸러지면서) 처리되고, 다시 두 번째 원소가 처리되며, 이런 처리가 모든 원소에 대해 적용된다.
- 따라서 원소에 연산을 차례대로 적용하다가 결과가 얻어지면 그 이후의 원소에 대해서는 변환이 이뤄지지 않을 수도 있다.
```kotlin
println(listOf(1, 2, 3, 4).asSequence()
                          .map { it * it }.find { it > 3 }) // 4        
```
- 컬렉션에 수행하면 map의 결과가 먼저 평가돼 최초 컬렉션의 모든 원소가 변환된다. 두 번째 단계에서는 map을 적용해서 얻은 중간 컬렉션으로부터 술어를 만족하는 원소를 찾는다.
  - 컬렉션을 사용하면 리스트가 다른 리스트로 변환된다.


- 시퀀스를 사용하면 find 호출이 원소를 하나씩 처리하기 시작한다. 최초 시퀀스로부터 수를 하나 가져와서 map에 지정된 변환을 수행한 다음에 find에 지정된 술어를 만족하는지 검사한다.
  - 시퀀스를 사용하면 지연 계산으로 인해 원소 중 일부의 계산은 이뤄지지 않는다.


- 컬렉션에 대해 수행하는 연산의 순서도 성능에 영향을 끼친다.
```kotlin
val people = listOf(Person("Alice", 29), Person("Bob", 31),
                Person("Charles", 31), Person("Dan", 21))

println(people.asSequence().map(Person::name)
    .filter { it.length < 4 }.toList() ) // [Bob, Dan]

println(people.asSequence().filter { it.lenth < 4 }
    .map(Person::name).toList()) // [Bob, Dan]
```
- map을 먼저 하면 모든 원소를 반환한다. 하지만 filter를 먼저 하면 부적절한 원소를 먼저 제외하기 때문에 그런 원소는 반환하지 않는다.

### 시퀀스 만들기
```kotlin
// 자연수의 시퀀스를 생성하고 사용하기
val naturalNumbers = generateSequence(0) { it + 1 }
val numbersTo100 = naturalNumbers.takeWhile { it <= 100 }
println(numbersTo100.sum()) // 5050
```
- 이 예제에서 naturalNumbers와 numbersTo100은 모두 시퀀스며, 연산을 지연 계산한다. 최종 연산을 수행하기 전까지는 시퀀스의 각 숫자는 계산되지 않는다(여기서는 sum이 최종 연산이다).

```kotlin
fun File.isInsideHiddenDirectory() =
    generateSequence(this) { it.parentFile }.any { it.isHidden }

val file = File("/Users/svtk/.HiddenDir/a.txt")
println(file.isInsideHiddenDirectory()) // true
```
- 여기서도 첫 번째 원소를 지정하고 시퀀스의 한 원소로부터 다음 원소를 제공함으로써 시퀀스를 만든다.
- 이렇게 시퀀스를 사용하면 조건을 만족하는 디렉터를 찾은 뒤에는 더 이상 상위 디렉터리를 뒤지지 않는다.

## 자바 함수형 인터페이스 활용
```kotlin
button.setOnClickListener { /* 클릭 시 수행할 동작 */ } // 람다를 인자로 넘김
```
- Button 클래스는 setOnClickListener 메소드를 사용해 버튼의 리스너를 설정한다. 이때 인자의 타입은 OnClickListener다.
```java
public class Button {
    public void setOnClickListener(OnClickListener l) { ... }
}
```
- OnClickListener 인터페이스는 onClick이라는 메서드만 선언된 인터페이스다.
```java
public interface OnClickListener {
    void onClick(View v);
}
```
- 자바 8 이전의 자바에서는 setOnClickListener 메서드에게 인자로 넘기기 위해 무명 클래스의 인스턴스를 만들어야만 했다.
```java
button.setOnClickListener(new OnclickListener() {
    @Override
    public void onClick(View v) {
        ...
    }
})
```
- 코틀린에서는 무명 클래스 인스턴스 대신 람다를 넘길 수 있다.
```kotlin
button.setOnClickListener { view -> ... }
```
- 이런 코드가 작동하는 이유는 OnClickListener에 <span style='background-color: #fff5b1'/>추상 메소드가 단 하나만 있기 때문이다. 그런 인터페이스를 `함수형 인터페이스` 또는 `SAM(Single Abstract Method) 인터페이스`라고 한다.

### 자바 메소드에 람다를 인자로 전달
- 함수형 인터페이스를 인자로 원하는 자바 메소드에 코틀린 람다를 전달할 수 있다.
```java
void postponeComputation(int delay, Runnable computation);
```
- 코틀린에서 람다를 이 함수에 넘길 수 있다. 컴파일러는 자동으로 람다를 Runnable(을 구현한 무명 클래스의) 인스턴스로 변환해준다.
```kotlin
postponeComputation(1000) { println(42) }
```
- 컴파일러는 자동으로 그런 무명 클래스와 인스턴스를 만들어준다. 이때 그 무명 클래스에 있는 유일한 추상 메소드를 구현할 때 람다 본문을 메소드 본문으로 사용한다. 여기서는 Runnable의 run이 그런 추상 메소드다.
- Runnable을 구현하는 무명 객체를 명시적으로 만들어서 사용할 수도 있다.
```kotlin
// 객체 식을 함수형 인터페이스 구현으로 넘긴다.
postponeComputation(1000, object: Runnable {
    override fun run() {
        println(42)
    }
}) 
```
- 하지만 람다와 무명 객체 사이에는 차이가 있다.
  - <span style='background-color: #fff5b1'/>객체를 명시적으로 선언하는 경우 메소드를 호출할 때마다 새로운 객체가 생성된다.
  - <span style='background-color: #fff5b1'>람다는 다르다. <span style='background-color: #F7DDBE'>정의가 들어있는 함수의 변수에 접근하지 않는 람다에 대응하는</span> 무명 객체를 메소드를 호출할 때마다 반복 사용한다.</span>

> ✅ **함수의 변수에 접근하지 않는 람다?** : 클로저를 형성하지 않은 람다

```kotlin
// 프로그램 전체에서 Runnable의 인스턴스는 단 하나만 만들어진다.
postponeComputation(1000) { println(42) }
```
- 따라서 명시적인 object 선언을 사용하면서 람다와 동일한 코드는 다음과 같다. 이 경우 Runnable 인스턴스를 변수에 저장하고 메소드를 호출할 때마다 그 인스턴스를 재사용한다.
```kotlin
// 전역 변수로 컴파일되므로 프로그램 안에 단 하나의 인스턴스만 존재한다.
val runnable = Runnable { println(42) }

fun handleComputation() {
    // 모든 handleComputation 호출에 같은 객체를 사용한다.
    postponeComputation(1000, runnable) 
}
```
- 람다가 주변 영역의 변수를 포획한다면 매 호출마다 같은 인스턴스를 사용할 수 없다. 그런 경우 컴파일러는 매번 주변 영역의 변수를 포획한 새로운 인스턴스를 생성해준다.
```kotlin
fun handleComputation(id: String) {
    // handleComputation을 호출할 때마다 새로 Runnable 인스턴스를 만든다.
    postponeComputation(1000, runnable)
} 
```
- 람다에 대해 무명 클래스를 만들고 그 클래스의 인스턴스를 만들어서 메소드에 넘긴다는 설명은 함수형 인터페이스를 받는 자바 메소드를 코틀린에서 호출할 때 쓰는 방식을 설명해준다.
- 하지만, 컬렉션을 확장한 메소드에 람다를 넘기는 경우 코틀린은 그런 방식을 사용하지 않는다.
  - <span style='background-color: #fff5b1'/>코틀린 inline으로 표시된 코틀린 함수에게 람다를 넘기면 아무런 무명 클래스도 만들어지지 않는다.
    - 대부분의 코틀린 확장 함수들은 inline 표시가 붙어있다. (8장)
- 대부분의 경우 람다와 자바 함수형 인터페이스 사이의 변환은 자동으로 이뤄진다.

### SAM 생성자: 람다를 함수형 인터페이스로 명시적으로 변경

> 💡 람다를 함수형 인터페이스의 인스턴스로 변환할 수 있게 컴파일러가 자동으로 생성한 함수(람다 표현식)다.

- 컴파일러가 자동으로 람다를 함수형 인터페이스 무명 클래스로 바꾸지 못하는 경우 SAM 생성자를 사용할 수 있다.
  - 함수형 인터페이스의 인스턴스를 반환하는 메소드가 있다면 람다를 직접 반환할 수 없고, 반환하고픈 람다를 SAM 생성자로 감싸야 한다.
```kotlin
fun createAllDoneRunnable(): Runnable {
    return Runnable { println("All done!") }
}

createAllDoneRunnable().run() // All done!
```
- SAM 생성자의 이름은 사용하려는 함수형 인터페이스의 이름과 같다.
- SAM 생성자는 그 함수형 인터페이스의 유일한 추상 메소드의 본문에 사용할 람다만을 인자로 받아서 함수형 인터페이스를 구현하는 클래스의 인스턴스를 반환한다.


- 람다로 생성한 함수형 인터페이스 인스턴스를 변수에 저장해야 하는 경우에도 SAM 생성자를 사용할 수 있다.
```kotlin
// SAM 생성자를 사용해 listener 인스턴스 재사용하기
val listener = OnClickListener { view -> 
    val text = when (view.id) {
        R.id.button1 -> "First button"
        R.id.button2 -> "Second button"
        else -> "Unknown button"
    }
    toast(text)
}

button1.setOnClickListener(listener)
button2.setOnClickListener(listener)
```
- OnClickListener를 구현하는 객체 선언을 통해 리스너를 만들 수도 있지만 SAM 생성자를 쓰는 쪽이 더 간결하다.
```kotlin
val listener = object : View.OnClickListener {
  override fun onClick(view: View) {
    val text = when (view.id) {
      R.id.button1 -> "First button"
      R.id.button2 -> "Second button"
      else -> "Unknown button"
    }
    toast(text)
  }
} 
```

> **람다와 리스너 등록/해제하기**
>
> 람다에는 무명 객체와 달리 인스턴스 자신을 가리키는 this가 없다는 사실에 유의하라. 따라서 람다를 변환한 무명 클래스의 인스턴스를 참조할 방법이 없다. 컴파일러 입장에서 보면 람다는 코드 블록일 뿐이고, 객체가 아니므로 객체처럼 람다를 참조할 수는 없다. 람다 안에서 this는 그 람다를 둘러싼 클래스의 인스턴스를 가리킨다.
> ```kotlin
> class MyClass {
>    private val myProperty: Int = 42
>
>    fun doSomething() {
>        // 람다 표현식 내부에서 this는 MyClass의 인스턴스를 가리킴
>        val myLambda: () -> Unit = {
>            println("Inside lambda: $this.myProperty")
>        }
>
>        myLambda()
>
>        println("Outside lambda: $this.myProperty")
>    }
> }
>
> fun main() {
>     val myInstance = MyClass()
>     myInstance.doSomething()
> }
> 
> // Inside lambda: MyClass@506e1b77.myProperty
> // Outside lambda: MyClass@506e1b77.myProperty
> ```

- 이벤트 리스너가 이벤트를 처리하다가 자기 자신의 리스너 등록을 해제해야 한다면 람다를 사용할 수 없다. 그런 경우 람다 대신 무명 객체를 사용해 리스너를 구현한다. 무명 객체 안에서는 this가 그 무명 객체 인스턴스 자신을 가리킨다. 따라서 리스너를 해제하는 API 함수에게 this를 넘길 수 있다.
```kotlin
import android.view.View
import android.widget.Button

class MyActivity {

    private var button: Button? = null
    private var myListener: View.OnClickListener? = null

    init {
        // 무명 객체를 사용하여 OnClickListener 구현
        myListener = object : View.OnClickListener {
            override fun onClick(v: View?) {
                // 이벤트 처리 코드

                // 자기 자신의 리스너 등록 해제
                button?.setOnClickListener(null)
            }
        }
        // 버튼에 리스너 등록
        button?.setOnClickListener(myListener)
    }
}
```

- 함수형 인터페이스를 요구하는 메소드를 호출할 때 대부분의 SAM 변환을 컴파일러가 자동으로 수행할 수 있지만, 가끔 오버라이드한 메소드 중에서 어떤 타입의 메소드를 선택해 람다를 변환해 넘겨줘야 할지 모호한 때가 있다.
  - 그런 경우 명시적으로 SAM 생성자를 적용하면 컴파일 오류를 피할 수 있다.

## [수신 객체 지정 람다: with와 apply](https://velog.io/@dev-baik/%EB%B2%94%EC%9C%84-%EC%A7%80%EC%A0%95-%ED%95%A8%EC%88%98)

> 💡 수신 객체를 명시하지 않고, 람다의 본문 안에서 해당 객체의 메서드를 호출할 수 있게 하는 람다

### with 함수
- 알파벳 만들기
```kotlin
fun alphabet(): String {
    val result = StringBuilder()
  
    for (letter in 'A'..'Z') {
        result.append(letter)
    }
    result.append("\nNow I know the alphabet!")
    return result.toString()
}

println(alphabet())

// ABCDEFGHIJKLMNOPQRSTUVWXYZ
// Now I know the alphabet!
```
- with 사용해 알파벳 만들기
```kotlin
fun alphabet(): String {
    val result = StringBuilder()
  
    return with(stringBuilder) { // 메소드를 호출하려는 수신 객체를 지정한다.
        for (letter in 'A'..'Z') {
            this.append(letter) // "this"를 명시해서 앞에서 지정한 수신 객체의 메소드를 호출한다.
        }
        append("\nNow I know the alphabet!") // "this"를 생략하고 메소드를 호출한다.
        this.toString() // 람다에서 값을 반환한다.
    }
}

println(alphabet())

// ABCDEFGHIJKLMNOPQRSTUVWXYZ
// Now I know the alphabet!
```
- with는 실제로 파라미터가 2개 있는 함수다. 첫 번째 파라미터는 stringBuilder이고, 두  번째 파라미터는 람다다.
- 람다를 괄호 밖으로 빼내는 관례를 사용함에 따라 전체 함수 호출이 언어가 제공하는 특별 구문처럼 보인다.
  - `with(stringBuilder, { ... })`라고 쓸 수 있지만 더 읽기 나빠진다.
- <span style='background-color: #fff5b1'/>with 함수는 첫 번째 인자로 받은 객체를 두 번째 인자로 받은 람다의 수신 객체로 만든다.
  - 인자로 받은 람다 본문에서는 this를 사용해 그 수신 객체에 접근할 수 있다.

> **수신 객체 지정 람다와 확장 함수 비교**
>
> 확장 함수 안에서 this는 그 함수가 확장하는 타입의 인스턴스를 가리킨다. 그리고 그 수신 객체 this의 멤버를 호출할 때는 this.를 생략할 수 있다. 어떤 의미에서는 확장 함수를 수신 객체 지정 함수라 할 수도 있다. 람다는 일반 함수와 비슷한 동작을 정의하는 한 방법이다. 수신 객체 지정 람다는 확장 함수와 비슷한 동작을 정의하는 한 방법이다.

- with와 식을 본문으로 하는 함수를 활용해 알파벳 만들기
```kotlin
fun alphabet() = with(StringBuilder()) {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet!")
    toString()
}
```
- 불필요한 stringBuilder 변수를 없애면 alphabet 함수가 식의 결과를 바로 반환하게 된다. 따라서 식을 본문으로 하는 함수로 표현할 수 있다.
  - StringBuilder의 인스턴스를 만들고 즉시 with에게 인자로 넘기고, 람다 안에서 this를 사용해 그 인스턴스를 참조한다.

> **메소드 이름 충돌**
>
> with에게 인자로 넘긴 객체의 클래스와 with를 사용하는 코드가 들어있는 클래스 안에 이름이 같은 메소드가 있으면 무슨 일이 생길까? 그런 경우 this 참조 앞에 레이블을 붙이면 호출하고 싶은 메소드를 명확하게 정할 수 있다.
>
> alphabet 함수가 OuterClass의 메소드라고 하자. StringBuilder가 아닌 바깥쪽 클래스(OuterClass)에 정의된 toString을 호출하고 싶다면 `this@OuterClass.toString()` 구문을 사용해야 한다.

- with가 반환하는 값은 람다 코드를 실행한 결과며, 그 결과는 람다 식의 본문에 있는 마지막 식의 값이다.

### apply 함수
- apply 함수는 거의 with와 같다.
- 유일한 차이란 apply는 항상 자신에게 전달된 객체(즉 수신 객체)를 반환한다는 점뿐이다.
```kotlin
fun alphabet() = StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet")
}.toString()
```
- apply는 확장 함수로 정의돼 있다. apply의 수신 객체가 전달받은 람다의 수신 객체가 된다. 이 함수에서 apply를 실행한 결과는 StringBuilder 객체다. 따라서 그 객체의 toString을 호출해서 String 객체를 얻을 수 있다.
- <span style='background-color: #fff5b1'/>apply 함수는 객체의 인스턴스를 만들면서 즉시 프로퍼티 중 일부를 초기화해야 하는 경우 유용하다.
  - 자바에서는 보통 별도의 Builder 객체가 이런 역할을 담당한다.
  - 코틀린에서는 어떤 클래스가 정의돼 있는 라이브러리의 특별한 지원 없이도 그 클래스 인스턴스에 대해 apply를 활용할 수 있다.
```kotlin
// apply를 TextView 초기화에 사용하기
fun createViewWithCustomAttributes(context: Context) =
    TextView(context).apply {
        text = "Sample Text"
        textSize = 20.0
        setPadding(10, 0, 0, 0)
    }
```
- 새로운 TextView 인스턴스를 만들고 즉시 그 인스턴스를 apply에 넘긴다. apply에 전달된 람다 안에서는 TextView가 수신 객체가 된다. 따라서 원하는 대로 TextView의 메소드를 호출하거나 프로퍼티를 설정할 수 있다.
- 람다를 실행하고 나면 apply는 람다에 의해 초기화된 TextView 인스턴스를 반환한다. 그 인스턴스는 createViewWithCustomAttributes 함수의 결과가 된다.
- 표준 라이브러리의 buildString 함수를 사용하면 alphabet 함수를 더 단순화할 수 있다.
  - buildString은 StringBuilder 객체를 만드는 일과 toString을 호출해주는 일을 알아서 해준다.
  - buildString의 인자는 수신 객체 지정 람다며, 수신 객체는 항상 StringBuilder가 된다.
```kotlin
fun alphabet() = buildString {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet!") 
}
```