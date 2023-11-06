# 람다로 프로그래밍
> - 람다 식과 멤버 참조
> - 함수형 스타일로 컬렉션 다루기
> - 시퀀스: 지연 컬렉션 연산
> - 자바 함수형 인터페이스를 코틀린에서 사용
> - 수신 객체 지정 람다 사용

- - -

- 람다 식 또는 람다 : 다른 함수에 넘길 수 있는 작은 코드 조각
- 람다를 사용하면 쉽게 콩통 코드 구조를 라이브러리 함수로 뽑아낼 수 있다. 코틀린 표준 라이브러리는 람다를 아주 많이 사용한다. 람다를 자주 사용하는 경우로 컬렉션 처리를 들 수 있다.
- 수신 객체 람다(lambda with receiver)은 특별한 람다로, 람다 선언을 둘러싸고 있는 환경과는 다른 상황에서 람다 본문을 실행할 수 있다.

## 람다 식과 멤버 참조
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
  - 함수형 프로그래밍에서는 함수를 값처럼 다루는 접근 방법을 택함으로써 이 문제를 해결한다.
    - 클래스를 선언하고 그 클래스의 인스턴스를 함수에 넘기는 대신 함수형 언어에서는 함수를 직접 다른 함수에 전달할 수 있다.
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
```kotlin
- 멤버 참조를 사용해 컬렉션 검색하기
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

- run은 인자로 받은 람다를 실행해주는 라이브러리 함수다.
```kotlin
run { println(42) } // 42 
```
- 코틀린이 코드를 줄여 쓸 수 있게 제공했던 기능을 제거하고 정식으로 람다를 작성하면 다음과 같다.
```kotlin
people.maxBy({ p: Person -> p.age }) 
```
- 여기서 어떤 일이 벌어지고 있는지 더 명확히 알 수 있다. 하지만 이 코드는 번잡하다.
  - 구분자가 너무 많이 쓰여서 가동성이 떨어지고, 컴파일러가 문맥으로부터 유추할 수 있는 인자 타입을 굳이 적을 필요는 없다.
  - 인자가 단 하나뿐인 경우 굳이 인자에 이름을 붙이지 ㅇ낳아도 된다.

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

//// 결과
// Computingthe sum of 1 and 2...
// 3
```

### 현재 영역에 있는 변수에 접근
- 람다를 함수 안에서 정의하면 파라미터뿐 아니라 람다 정의의 앞에 선언된 로컬 변수까지 람다에서 모두 사용할 수 있다.
- forEach는 컬렉션의 모든 원소에 대해 람다를 호출해준다. 일반적으로 for 루프보다 훨씬 간결하지만 그렇다고 다른 장점이 많지는 않아 기존 for 루프를 forEach로 모두 바꿀 필요는 없다.
```kotlin
// 함수 파라미터를 람다 안에서 사용하기
fun printMessagesWithPrefix(messages: Collection<String>, prefix: String) {
    message.forEach { // 각 원소에 대해 수행할 작업을 람다로 받는다.
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
- 이 예제의 prefix, clientErrors, serverErrors와 같이 람다 안에서 사용하는 외부 변수를 '람다가 포획한 변수'라고 부른다.
- 기본적으로 함수 안에 정의된 로컬 변수의 생명주기는 함수가 반환되면 끝난다. 하지만 어떤 함수가 자신의 로컬 변수를 포획한 람다를 반환하거나 다른 변수에 저장한다면 로컬 변수의 생명주기와 함수의 생명주기가 달라질 수 있다.


- 람다를 이벤트 핸들러나 다른 비동기적으로 실행되는 코드로 활용하는 경우 함수 호출이 끝난 다음에 로컬 변수가 변경될 수 있다.
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
- 코틀린에서는 함수를 값으로 바꿀 수 있는 이중 콜론(::)을 사용한다.
```kotlin
val getAge = { person: Person -> person.age }
```
```kotlin
val getAge = Person::age 
```
- ::를 사용하는 식을 멤버 참조라고 부른다. 멤버 참조는 프로퍼티나 메소드를 단 하나만 호출하는 함수 값을 만들어준다. ::는 클래스 이름과 여러분이 참조하려는 멤버(프로퍼티나 메소드) 이름 사이에 위치한다.
- 참조 대상이 함수인지 프로퍼티인지와는 관계없이 멤버 참조 뒤에는 괄호를 넣으면 안된다.
- 멤버 참조는 그 멤버를 호출하는 람다와 같은 타입이다
```kotlin
people.maxBy(Person::age)
people.maxBy { p -> p.age }
people.maxBy {it.age }
```
- 최상위에 선언된(그리고 다른 클래스의 멤버가 아닌) 함수나 프로퍼티를 참조할 수도 있다.
```kotlin
fun salute() = println("Salute!")
return(::salute) // 최상위 함수를 참조한다.
// 결과: Saluate!
```
- 클래스 이름을 생략하고 ::로 참조를 바로 시작한다. ::salute라는 멤버 참조를 run 라이브러리 함수에 넘긴다(run은 인자로 받은 람다를 호출한다).
- 람다가 인자가 여럿인 다른 함수한테 작업을 위임하는 경우 람다를 정의하지 않고 직접 위임 함수에 대한 참조를 제공하면 편리하다
```kotlin
val action = { person: Person, message: String -> // 이 람다는 sendEmail 함수에게 작업을 위임한다.
    sendEmail(person, message)
}
val nextAction = ::sendEmail // 람다 대신 멤버 참조를 쓸 수 있다.
```
- 생성자 참조를 사용하면 클래스 생성 작업을 연기하거나 저장해둘 수 있다. :: 뒤에 클래스 이름을 넣으면 생성자 참조를 만들 수 있다.
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
  - 바운드 멤버 참조를 사용하면 멤버 참조를 생성할 때 클래스 인스턴스를 함께 저장한 다음 나중에 그 인스턴스에 대해 멤버를 호출해준다. 따라서 호출 시 수신 대상 객체를 별도로 지정해 줄 필요가 없다.
```kotlin
val p = Person("Dmitry", 34)
val personsAgeFunction = Person::age
println(personsAgeFunction(p)) // 34

val dmitrysAgeFunction = p::age
println(dmitrysAgeFunction()) // 45
```
- personsAgeFunction은 인자가 하나(인자로 받은 사람의 나이를 반환)이지만, dmitrysAgeFunction은 인자가 없는(참조를 만들 때 p가 가리키던 사람의 나이를 반환) 함수라는 점에 유의하라.
- 코틀린 1.0에서는 p::age 대신에 { p.age }라고 직접 객체의 프로퍼티를 돌려주는 람다를 만들어야만 한다.