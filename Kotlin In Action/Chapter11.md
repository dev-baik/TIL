# DSL 만들기

> - 영역 특화 언어 만들기
> - 수신 객체 지정 람다 사용
> - invoke 관례 사용
> - 기존 코틀린 DSL 예제

## API에서 DSL로
- 코드의 가독성과 유지 보수성을 가장 좋게 유지하기 위해서 상호작용이 일어나는 연결 지점(인터페이스)를 살펴봐야한다.
- 클래스 간의 상호작용을 이해하기 쉽고 명확하게 표현할 수 있게 만들어야 프로젝트를 계속 유지 보수할 수 있다.

#### API가 깔끔하다.
- 코드를 읽는 독자들이 어떤 일이 벌어질지 명확하게 이해할 수 있어야 한다. 이름과 개념을 잘 선택하면 이런 목적을 달성할 수 있다. 어떤 언어를 사요하건 이름을 잘 붙이고 적절한 개념을 사용하는 것은 매우 중요하다.
- 코드가 간결해야 한다. 불필요한 구문이나 번잡한 준비 코드가 가능한 한 적어야 한다. (간결함) 깔끔한 API는 언어에 내장된 기능과 거의 구분할 수 없다.

#### 코틀린이 간결한 구문을 어떻게 지원하는가?
<table>
    <tr>
        <th>일반 구문</th>
        <th>간결한 구문</th>
        <th>사용한 언어 특성</th>
    </tr>
    <tr>
        <td>StringUtil.capitalize(s)</td>
        <td>s.capitalize()</td>
        <td>확장 함수</td>
    </tr>
    <tr>
        <td>1.to("one")</td>
        <td>1 to "one"</td>
        <td>중위 호출</td>
    </tr>
    <tr>
        <td>set.add(2)</td>
        <td>set += 2</td>
        <td>연산자 오버로딩</td>
    </tr>
    <tr>
        <td>map.get("key")</td>
        <td>map["key"]</td>
        <td>get 메소드에 대한 관례</td>
    </tr>
    <tr>
        <td>file.use({ f -> f.read() })</td>
        <td>file.use { it.read() }</td>
        <td>람다를 괄호 밖으로 빼내는 관례</td>
    </tr>
    <tr>
        <td>sb.append("yes") sb.append("no")</td>
        <td>with(sb) { append("yes") append("no") }</td>
        <td>수신 객체 지정 람다</td>
    </tr>
</table>

- 코틀린 DSL은 간결한 구문을 제공하는 기능과 그런 구문을 확장해서 여러 메소드 호출을 조합함으로써 구조를 만들어내는 기능에 의존한다.
- 온전히 컴파일 시점에 타입이 정해진다. 따라서 컴파일 시점 오류 감지, IDE 지원 등 모든 정적 타입 지정 언어의 장점을 코틀린 DSL을 사용할 떄도 누릴 수 있다.

### 영역 특화 언어라는 개념
#### 컴퓨터가 발명된 초기부터 컴퓨터로 풀 수 있는 모든 문제를 충분히 풀 수 있는 기능을 제공하는 범용 프로그래밍 언어와 특정 과업 또는 영역에 초점을 맞추고 그 영역에 필요하지 않은 기능을 없앤 영역 특화 언어(DSL)를 구분한다.
1. SQL 문장을 실행할 필요가 있는 경우 클래스나 함수를 선언하는 것부터 시작할 필요가 없다. 대신에 모든 SQL 문장은 첫 키워드가 수행하려는 연산의 종류를 지정하고, 각 연산은 처리해야 할 작업에 맞춰 각각 서로 다른 문법과 키워드를 사용한다.


2. 정규식 프로그램은 압축적인 기호 문법을 사용해 텍스트가 어떻게 달라질 수 있는지 지정함으로써 대상 텍스트를 직접 기술한다. 이런 압축적인 문법을 사용함으로서 DSL은 범용 언어를 사용하는 경우보다 특정 영역에 대한 연산을 더 간결하게 기술할 수 있다.

#### DSL이 범용 프로그래밍 언어와 달리 더 선언적(declarative)이라는 점이 중요하다.
- 범용 프로그래밍 언어는 보통 명령적(imperative)이다. 명력적 언어는 어떤 연산을 완수하기 위해 필요한 각 단계를 순서대로 정확히 기술한다.
- 선언적 언어는 원하는 결과를 기술하기만 하고 그 결과를 달성하기 위해 필요한 세부 실행은 언어를 해석하는 엔진에 맡긴다. 실행 엔진이 결과를 얻는 과정을 전체적으로 한꺼번에 최적화하기 때문에 선언적 언어가 더 효율적인 경우가 자주 있다.
- 반면 명령적 접근법에서는 각 연산에 대한 구현을 독립적으로 최적화해야 한다.

- **DSL 단점** : DSL을 범용 언어로 만든 호스트 애플리케이션과 함께 조합하기가 어렵다.

### 내부 SDL
- 독립적인 문법 구조를 가진 외부 DSL과는 반대로 내부 DSL은 범용언어로 작성된 프로그램의 일부이며, 범용 언어와 동일한 문법을 사용한다.
- 따라서 내부 DSL은 완전히 다른 언어가 아니라 DSL의 핵심 장점을 유지하면서 주 언어를 톡별한 방법으로 사용하는 것이다.

### DSL의 구조
- 전형적인 라이브러리 : 여러 메소드로 이뤄지며, 클라이언트는 그런 메소드를 한 번씩 호출함으로써 라이브러리를 사용
  - 명령-질의 API : 함수 호출 시퀀스에는 아무런 구조가 없으며, 한 호출과 다른 호출 사이에는 아무 맥락도 존재하지 않는다.
- 반대로 DSL의 메소드 호출은 DSL 문법에 의해 정해지는 더 커다란 구조에 속한다.

## 구조화된 API 구축: DSL에서 수신 객체 지정 DSL 사용
### 수신 객체 지정 람다와 확장 함수 타입
- 람다를 인자로 받는 buildString() 정의하기
```kotlin
fun buildString(
    builderAction: (StringBuilder) -> Unit
) : String {
    val sb = StringBuilder()
    // 람다 인자로 StringBuilder 인스턴스를 넘긴다.
    builderAction(sb)
    return sb.toString()
}

val s = buildString {
    // it은 StringBuilder 인스턴스를 가리킨다.
    it.append("Hello, ")
    it.append("World!")
}

println(s) // Hello, World!
```
- 수신 객체 지정 람다를 사용해 다시 정의한 buildString()
```kotlin
fun buildString(
    // 확장 함수 타입 : 수신 객체가 있는 함수 타입의 파라미터를 선언한다.
    // 수신 객체 타입.파라미터 타입 -> 반환 타입
    builderAction: StringBuilder.() -> Unit
) : String {
    val sb = StringBuilder()
    // StringBuilder 인스턴스를 람다의 수신 객체로 넘긴다.
    sb.builderAction()
    return sb.toString()
}

val s = buildString {
    // "this" 키워드는 StringBuilder 인스턴스를 가리킨다.
    this.append("Hello, ")
    // "this"를 생략해도 묵시적으로 StringBuilder 인스턴스가 수신 객체로 취급된다.
    append("World!")
}

println(s) // Hello, World!
```
- 확장 함수나 수신 객체 지정 람다에서는 모든 함수(람다)를 호출할 때 수신 객체를 지정해야만 하고, 함수(람다) 본문 안에서는 그 수신 객체를 특별한 수식자 없이 사용할 수 있다.
- builderActiondms StringBuilder 클래스 안에 정의가 있는 함수가 아니며, StringBuilder 인스턴스인 sb는 확장 함수를 호출할 때와 동일한 구문으로 호출할 수 있는 함수 타입(따라서 확장 함수 타입)의 인자일 뿐이다.


- 수신 객체 지정 람다를 변수에 저장하기
```kotlin
// appendExcl은 확장 함수 타입의 값이다.
val appendExcl : StringBuilder.() -> Unit = { this.append("!") }

val stringBuilder = StringBuilder("Hi")
// appendExcl을 확장 함수처럼 호출할 수 있다.
stringBuilder.appendExcl()

// appendExcel을 인자로 넘길 수 있다.
println(StringBuilder) // Hi!
println(buildString(appendExcl)) // !
```
- builderAction을 apply 함수에게 인자로 넘기기
```kotlin
fun buildString(builderAction: StringBuilder.() -> Unit): String =
    StringBuilder().apply(builderAction).toString()
```
- apply 함수는 인자로 받은 람다나 함수(builderAction)를 호출하면서 자신의 수신 객체(StringBuilder의 인스턴스)를 람다나 함수의 묵시적 수신 객체로 사용한다.
- apply는 수신 객체 타입에 대한 확장 함수로 선언됐기 때문에 수신 객체의 메소드처럼 불리며, 수신 객체를 묵시적 인자(this)로 받는다.
```kotlin
inline fun <T> T.apply(block: T.() -> Unit) : T {
    block()
    return this // 수신 객체를 다시 반환한다.
}
```
- with는 수신 객체를 첫 번째 파라미터로 받는다.
```kotlin
inline fun <T, R> with(receiver: T, block: T.() -> R): R =
    receiver.block() // 람다를 호출해 얻은 결과를 반환한다.
```
- 결과를 받아서 쓸 필요가 없다면 두 함수를 서로 바꿔 쓸 수 있다.
```kotlin
val map = mutableMapOf(1 to "one")
map.apply { this[2] = "two" }
with(map) { this[3] = "three" }
println(map) // {1=one, 2=two, 3=three}
```

## invoke 관례를 사용한 더 유연한 블록 중첩
- invoke 관례를 사용하면 객체를 함수처럼 호출할 수 있고, 함수처럼 호출할 수 있는 객체를 만드는 클래스를 정의할 수 있다.

### invoke 관례: 함수처럼 호출할 수 있는 객체
- 관례 : 특별한 이름이 붙은 함수를 일반 메소드 호출 구문으로 호출하지 않고 더 간단한 다른 구문으로 호출할 수 있게 지원하는 기능
- operator 변경자가 붙은 invoke 메소드 정의가 들어있는 클래스의 객체를 함수처럼 호출할 수 있다.
```kotlin
// 클래스 안에서 invoke 메소드 정의하기
class Greeter(val greeting: String) {
    // Greeter 안에 "invoke" 메소드를 정의한다.
    operator fun invoke(name: String) {
        println("$greeting, $name")
    }
}

val bavarianGreeter = Greeter("Servus")
// Greeter 인스턴스를 함수처럼 호출한다.
// 내부적으로 bavarianGreeter.invoke("Dmitry")로 컴파일된다.
bavarianGreeter("Dmitry") // Servus, Dmitry!
```
- 여러 파라미터 타입을 지원하기 위해 invoke를 오버로딩할 수 있다. 오버로딩한 invoke가 있는 클래스의 인스턴스를 함수처럼 사용할 때는 오버로딩한 여러 시그니처를 모두 다 활용할 수 있다.

### invoke 관례와 함수형 타입
- 인라인하는 람다를 제외한 모든 람다는 함수형 인터페이스(Function1 등)를 구현하는 클래스로 컴파일된다. 각 함수형 인터페이스 안에는 그 인터페이스 이름이 가리키는 개수만큼 파라미터를 받는 invoke 메소드가 들어있다.
```kotlin
// 이 인터페이스는 인자를 2개 받는 함수를 표현한다.
interface Function2<in P1, in P2, out R> {
    operator fun invoke(p1: P1, p2: P2): R
} 
```
- 람다를 함수처럼 호출하면 이 관례에 따라 invoke 메소드 호출로 변환된다.

#### 이런 사실을 알면 어떤 점이 좋을까?
- 복잡한 람다를 여러 메소드로 분리하면서도 여전히 분리 전의 람다처럼 외부에서 호출할 수 있는 객체를 만들 수 있다.
```kotlin
class ComplexLambda {
    private var value: Int = 0

    fun initialize(initialValue: Int): ComplexLambda {
        value = initialValue
        return this
    }

    fun add(number: Int): ComplexLambda {
        value += number
        return this
    }

    fun multiply(factor: Int): ComplexLambda {
        value *= factor
        return this
    }
    
    operator fun invoke(): Int {
        return value
    }
}

fun main() {
    // 복잡한 람다를 여러 메소드로 분리하면서도 호출 가능한 객체 생성
    val complexLambda = ComplexLambda()
        .initialize(10)
        .add(5)
        .multiply(2)
    
    val result = complexLambda()
    println(result) // 30
}
```
- 함수 타입 파라미터를 받는 함수에게 그 객체를 전달할 수 있다.
```kotlin
class DataProcessor {
    // 함수 타입 파라미터를 받는 함수
    fun <T> processData(data: T, processorFunction: (T) -> Unit) {
        // 전달된 함수에 객체 전달
        // processorFunction.invoke(data)
        processorFunction(data)
    }
}

fun main() {
    val dataProcessor = DataProcessor()
    
    val stringProcessor: (String) -> Unit = { value ->
        println("Processing string: $value")
    }
    
    val intProcessor: (Int) -> Unit = { value ->
        println("Processing integer: $value")
    }
    
    dataProcessor.processData("Hello, Kotlin!", stringProcessor)
    dataProcessor.processData(42, intProcessor)
} 
```
- 이런 식으로 기존 람다를 여러 함수로 나눌 수 있으려면 함수 타입 인터페이스를 구현하는 클래스를 정의해야 한다.
- 이때 기반 인터페이스를 FunctionN<P1, ..., PN, R> 타입이나 (P1, ..., PN) -> R 타입으로 명시해야 한다.
```kotlin
interface MyLambda : Function3<Int, Int, Int, Int> {
    fun initialize(initialValue: Int): MyLambda
    fun add(number: Int): MyLambda
    fun multiply(factor: Int): MyLambda

    override operator fun invoke(p1: Int, p2: Int, p3: Int): Int {
        return p1 + p2 + p3
    }
}

class LambdaProcessor : MyLambda {
    private var value: Int = 0

    override fun initialize(initialValue: Int): MyLambda {
        value = initialValue
        return this
    }

    override fun add(number: Int): MyLambda {
        value += number
        return this
    }

    override fun multiply(factor: Int): MyLambda {
        value *= factor
        return this
    }
}

fun main() {
    val lambdaProcessor = LambdaProcessor()
        .initialize(10)
        .add(5)
        .multiply(2)

    val result = lambdaProcessor(1, 2, 3)
    println(result) // 18
}
```
- 함수 타입을 확장하면서 invoke()를 오버라이딩하기
```kotlin
data class Issue(
    val id: String, val project: String, val type: String,
    val property: String, val description: String
)

// 함수 타입을 부모 클래스로 사용한다.
class ImportantIssuesPredicate(val project: String) : (Issue) -> Boolean {
    override fun invoke(issue: Issue): Boolean {
        return issue.project == project && issue.isImportant()
    }
    
    private fun Issue.isImportant(): Boolean {
        return type == "Bug" && (priority == "Major" || priority == "Critical")
    }
}

val i1 = Issue("IDEA-154446", "IDEA", "Bug", "Major", "Save settings failed")
val i2 = Issue("KT-12183", "Kotlin", "Feature", "Normal",
    "Intention: convert several calls on the same receiver to with/apply")
val predicate = ImportantIssuesPredicate("IDEA")

for (issue in listOf(i1, i2).filter(predicate)) {
    println(issue.id)
} // IDEA-154446
``` 