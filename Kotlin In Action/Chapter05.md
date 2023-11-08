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

## 컬렉션 함수형 API
- 함수형 프로그래밍 스타일을 사용하면 컬렉션을 다룰 때 편리하다. 대부분의 작업에 라이브러리 함수를 활용할 수 있고 그로 인해 코드를 아주 간결하게 만들 수 있다.

### 필수적인 함수: filter와 map
- filter와 map은 컬렉션을 활용할 때 기반이 되는 함수다. 대부분의 컬렉션 연산을 이 두 함수를 통해 표현할 수 있다.

> 함수형 프로그래밍에는 람다나 다른 함수를 인자로 받거나 함수를 반환하는 함수를 고차 함수(HOF, Higt Order Function)라고 부른다. 고차함수는 기본 함수를 조합해서 새로운 연산을 정의하거나, 다른 고차 함수를 통해 조합된 함수를 또 조합해서 더 복잡한 연산을 쉽게 정의할 수 있다는 장점이 있다. 이런 식으로 고차 함수와 단순한 함수를 이리저리 조합해서 코드를 작성하는 기법을 컴비네이터 패턴(combinator pattern)이라 부르고, 컴비네이터 패턴에서 복잡한 연산을 만들기 위해 값이나 함수를 조합할 때 사용하는 고차 함수를 컴비네이터라고 부른다.

- filter 함수는 이터레이션하면서 주어진 람다에 각 원소를 넘겨서 람다가 true를 반환하는 원소만 모은다.
```kotlin
data class Person(val name: String, val age: Int)

val list = listOf(1, 2, 3, 4)
println(list.filter { it % 2 == 0 }) // [2, 4]
```
- 결과는 입력 컬렉션의 원소 중에서 주어진 술어(참/거짓을 반환하는 함수를 술어(predicate)라고 한다)를 만족하는 원소만으로 이뤄진 새로운 컬렉션이다.
```kotlin
val people = listOf(Person("Alice", 29), Person("Bob", 31))
println(people.filter { it.age > 30 }) // [Person(name=Bob, age=31)]
```
- filter 함수는 컬렉션에서 원치 않는 원소를 제거한다. 하지만 filter는 원소를 변환할 수는 없다.
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
// 멤버 참조 사용
people.map(Person::name)
```
```kotlin
// 30살 이상인 사람의 이름 출력
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
  - 가독성을 높이려면 any와 all 앞에 !를 붙이지 않는 편이 낫다.
```kotlin
val list = listOf(1, 2, 3)
println(!list.all { it == 3 }) // true

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
> 하지만 이렇게 처리하면 조건을 만족하는 모든 원소가 들어가는 중간 컬렉션이 생긴다. 반면 count는 조건을 만족하는 원소의 개수만을 추적하지 조건을 만족하는 원소를 따로 저장하지 않는다. 따라서 count가 훨씬 더 효율적이다.

```kotlin
// 술어를 만족하는 원소를 하나 찾고 싶으면 find 함수를 사용한다.
val people = listOf(Person("Alice", 27), Person("Bob", 31))
println(people.find(canBeInClub27)) // Person(name=Alice, age=27)
```
- 이 식은 조건을 만족하는 원소가 하나라도 있는 경우 가장 먼저 조건을 만족한다는 확인된 원소를 반환하며, 만족하는 원소가 전혀 없는 경우 null을 반환한다.
- 조건을 만족하는 원소가 없으면 null이 나온다는 사실을 더 명확히 하고 싶다면 firstOrNull을 쓸 수 있다.

### groupBy: 리스트를 여러 그룹으로 이뤄진 맵으로 변경
```kotlin
val people = listOf(Person("Alice", 27), Person("Bob", 31), Person("Carol", 31))
println(people.groupBy { it.age })

// 결과
{29=[Person(name=Bob, age=29)],
 31=[Person(name=Alice, age=31), Person(name=Carol, age=31)]}
```
- 각 그룹은 리스트다. 따라서 groupBy의 결과 타이은 Map<Int, List<Person>>이다. 필요하면 이 맵을 mapKeys나 mapValues 등을 사용해 변경할 수 있다.
```kotlin
val list = listOf("a", "ab", "b")
println(list.groupBy(String::first)) // {a=[a, ab], b=[b]}
```
- first는 String의 멤버가 아니라 확장 함수지만 여전히 멤버 참조를 사용해 first에 접근할 수 있다.

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

println(book.flatMap { it.authors }.toSet())
// 결과: [Jasper Fforde, Terry Pratchett, Neil Gaiman]
```
- toSet은 flatMap의 결과 리스트에서 중복을 없애고 집합을 만든다.
- 특별히 반환해야 할 내용이 없어 리스트의 리스트를 평평하게 펼치기만 하면 되는 경우 listOfLists.flatten()처럼 flatten 함수를 사용할 수 있다.

## 지연 계산(lazy) 컬렉션 연산
- map이나 filter 같은 몇가지 컬렉션 함수들은 결과 컬렉션을 즉시(eagerly) 생성한다.
  - 즉, 컬렉션 함수를 연쇄하면 매 단계마다 계산 중간 결과를 새로운 컬렉션에 임시로 담는다는 말이다.
- 시퀀스를 사용하면 중간 임시 컬렉션을 사용하지 않고도 컬렉션 연산을 연쇄할 수 있다.
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
  - 시퀀스의 원소는 필요할 때 비로소 계산된다. 따라서 중간 결과를 저장하지 않고도 연산을 연쇄적으로 적용해서 효율적으로 계산을 수행할 수 있다.
- asSequence 확장 함수를 호출하면 어떤 컬렉션이든 시퀀스로 바꿀 수 있다. 시퀀스를 리스트로 만들 때는 toList를 사용한다.

> **왜 시퀀스를 다시 컬렉션으로 되돌려야 할까?**
> 
> **컬렉션보다 시퀀스가 훨씬 더 낫다면 그냥 시퀀스를 쓰는 편이 낫지 않을까?**
> 
> - 항상 그렇지는 않다
> - 시퀀스의 원소를 차례로 이터레이션해야 한다면 시퀀스를 직접 써도 된다. 하지만 시퀀스 원소를 인덱스를 사용해 접근하는 등의 다른 API 메소드가 필요하다면 시퀀스를 리스트로 변환해야 한다.

> 큰 컬렉션에 대해서 연산을 연쇄시킬 때는 시퀀스를 사용하는 것을 규칙으로 삼아라.
> 
> 8장에서는 중간 컬렉션을 생성함에도 불구하고 코틀린에서 즉기 계산 컬렉션에 대한 연산이 더 효율적인 이유를 설명한다. 하지만 컬렉션에 들어있는 원소가 많으면 중간 원소를 재배열하는 비용이 커지기 때문에 지연 계산이 더 낫다.

### 시퀀스 연산 실행: 중간 연산과 최종 연산
- 시퀀스에 대한 연산은 중간 연산과 최종 연산으로 나뉜다.
  - 중간 연산은 다른 시퀀스를 반환한다. 그 시퀀스는 최초 시퀀스의 원소를 변환하는 방법을 안다.
  - 최종 연산은 결과를 반환한다. 결과는 최초 컬렉션에 대해 변환을 적용한 시퀀스로부터 일련의 계산을 수행해 얻을 수 있는 컬렉션이나 원소, 숫자 또는 객체다.
- 중간 연산은 항상 지연 계산된다.
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
- 시퀀스를 사용하면 지연 계산으로 인해 원소 중 일부의 게산은 이뤄지지 않는다.
- 컬렉션을 사용하면 리스트가 다른 리스트로 변환된다.
- 시퀀스를 사용하면 find 호출이 원소를 하나씩 처리하기 시작한다. 최초 시퀀스로부터 수를 하나 가져와서 map에 지정된 변환을 수행한 다음에 find에 지정된 술어를 만족하는지 검사한다.
  - 최초 시퀀스에서 2를 가져오면 제곱 값(4)이 3보다 커지기 때문에 그 제곱 값을 결과로 반환한다. 이때 이미 답을 찾았으므로 3과 4를 처리할 필요가 없다.


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
- 이 예제에서 naturalNumbersdhk numbersTo100은 모두 시퀀스며, 연산을 지연 계산한다. 최종 연산을 수행하기 전까지는 시퀀스의 각 숫자는 계산되지 않는다(여기서는 sum이 최종 연산이다).

```kotlin
fun File.isInsideHiddenDirectory() =
    generateSequence(this) { it.parentFile }.any { it.isHidden }

val file = File("/Users/svtk/.HiddenDir/a.txt")
println(file.isInsideHiddenDirectory()) // true
```
- 여기서도 첫 번째 원소를 지정하고 시퀀스의 한 원소로부터 다음 원소를 제공함으로써 시퀀스를 만든다.
- any를 find로 바꾸면 원하는 디렉터리를 찾을 수도 있다.
- 이렇게 시퀀스를 사용하면 조건을 만족하는 디렉터를 찾은 뒤에는 더 이상 상위 디렉터리를 뒤지지 않는다.