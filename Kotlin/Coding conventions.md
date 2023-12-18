# 1. Source code organization
## 1.1 Directory structure
- 순수 Kotlin 프로젝트에서 권장되는 디렉터리 구조는 공통 루트 패키지가 생략된 패키지 구조를 따른다.
- 예를 들어 프로젝트의 모든 코드가 `org.example.kotlin` 패키지와 해당 하위 패키지에 있는 경우
  - `org.example.kotlin` 패키지에 있는 파일은 소스 루트(Source Root) 바로 아래에 위치해야 한다.
  - `org.example.kotlin.network.socket`에 있는 파일은 소스 루트의 `network/socket` 하위 디렉터리에 있어야 한다.

> **JVM**: Kotlin이 Java와 함께 사용되는 프로젝트에서 Kotlin 소스 파일은 Java 소스 파일과 동일한 소스 루트에 있어야 하며, 같은 디렉터리 구조를 따라야 한다. 각 파일은 각 패키지 문(statement)에 해당하는 디렉터리에 저장되어야 한다. 이렇게 하면 Java와 Kotlin 코드가 함께있어도 체계적으로 관리할 수 있다.

## 1.2 Source file names
- Kotlin 파일이 단일 클래스나 인터페이스를 포함하고 있다면(관련 최상위 선언이 포함될 수 있음), 그 파일 이름은 클래스의 이름과 동일하게 하고, `.kt` 확장자를 붙여야 한다.
- <span style='background-color: #fff5b1'/>파일이 여러 개의 클래스를 포함하고 있거나, 오직 최상위 선언만을 포함하고 있다면, 파일이 무엇을 포함하고 있는지를 설명하는 이름을 선택하고, 그에 따라 파일의 이름을 지정하면 된다.
#### 여러 클래스가 포함된 경우
```kotlin
// 파일 이름: ProcessDeclarations.kt

// 선언 처리 관련 클래스
class DeclarationProcessor {
    fun processDeclaration(declaration: String): Boolean {
        // 선언 처리 로직
        println("Processing declaration: $declaration")
        return true
    }
}

// 선언 유효성 검사 관련 클래스
class DeclarationValidator {
    fun validateDeclaration(declaration: String): Boolean {
        // 선언 유효성 검사 로직
        println("Validating declaration: $declaration")
        return true
    }
}

// 선언 관련 유틸리티 함수들
fun formatDeclaration(declaration: String): String {
    // 선언 포맷 로직
    return declaration.trim()
}

// 기타 선언 처리와 관련된 클래스, 함수들...
```
#### 최상위 수준 선언만 포함된 경우
```kotlin
// 파일 이름: FileProcessingOperations.kt

const val DEFAULT_FILE_PATH = "/path/to/default/file"

fun readFromFile(filePath: String): String {
    // 파일에서 읽기 로직
    return "File content from $filePath"
}

fun writeToFile(filePath: String, content: String) {
    // 파일에 쓰기 로직
    println("Writing to file at $filePath: $content")
}

fun validateFilePath(filePath: String): Boolean {
    // 파일 경로 유효성 검사 로직
    return filePath.isNotBlank()
}

// 다른 최상위 수준 선언들...
```
- ProcessDeclarations.kt와 같이 첫 글자가 대문자인 `카멜 표기법(파스칼 표기법)`을 사용한다.
- <span style='background-color: #fff5b1'/>파일 이름은 파일의 코드가 수행하는 작업을 설명해야한다. 따라서 파일 이름에 Util과 같은 의미 없는 단어를 사용하는 것은 피해야 한다.

## 1.3 Source file organization(구성)
- <span style='background-color: #fff5b1'/>같은 Kotlin 소스 파일에 여러 선언(클래스, 최상위 함수 또는 프로퍼티)을 배치하는 것은 이러한 선언들이 의미적으로 서로 밀접하게 관련되어 있고, 파일 크기가 합리적인 범위(몇 백 줄을 넘지 않는) 내에 있을 때 권장된다.
  - 특히, 모든 클라이언트에게 관련된 확장 함수를 클래스에 정의하는 경우, 그 함수들을 클래스 자체와 같은 파일에 배치하라.
  - 특정 클라이언트에게만 의미가 있는 확장 함수를 정의하는 경우, 그 함수들을 해당 클라이언트의 코드 옆에 배치하라.
- 어떤 클래스의 모든 확장을 담기 위해 파일을 생성하지 마라.

## 1.4 Class layout
- 메소드 선언을 알파벳 순서로 정렬하거나 가시성에 따라 정렬하거나, 일반 메소드와 확장 메소드를 분리하지 마라.
- <span style='background-color: #fff5b1'/>대신, 관련 있는 것들을 함께 두어서, 클래스를 위에서 아래로 읽는 사람이 어떤 로직이 진행되고 있는지를 따라갈 수 있게 하라.
  - 순서를 선택하고(상위 수준의 것을 먼저 두거나 그 반대로 하거나) 그 순서를 지켜라.


- 중첩 클래스는 그 클래스를 사용하는 코드 옆에 배치한다.
- <span style='background-color: #fff5b1'/>만약 그 클래스들이 외부에서 사용되고 클래스 내부에서 참조되지 않는다면, 해당 클래스의 맨 뒤에 companion object 이후에 놓아라.

## 1.5 Interface implementation layout
- 인터페이스를 구현할 때는, 인터페이스의 멤버들과 동일한 순서로 멤버들을 구현하라(필요하다면, 구현에 사용되는 추가적인 private 메소드와 교차하여 배치하라).

## 1.6 Overload layout
- 클래스 내에서 항상 오버로드된 메서드를 서로 옆에 배치하라.

# 2. Naming rules
- 패키지의 이름은 항상 소문자이며 밑줄을 사용하지 않는다. (org.example.project)
- 다중 단어 이름 사용은 일반적으로 권장되지 않지만, 만약 여러 단어를 사용해야 한다면, 단어들을 그냥 연결하거나 카멜 케이스를 사용할 수 있다(org.example.myProject).


- 클래스와 객체의 이름은 대문자로 시작하고 카멜 케이스를 사용한다.
```kotlin
open class DeclarationProcessor { /*...*/ }

object EmptyDeclarationProcessor : DeclarationProcessor() { /*...*/ }
```

## 2.1 Function names
- 함수, 프로퍼티, 그리고 지역 변수의 이름은 소문자로 시작하고 카멜 케이스를 사용하며 밑줄을 사용하지 않는다.
```kotlin
fun processDeclarations() { /*...*/ }
var declarationCount = 1
```
- 예외: 클래스의 인스턴스를 생성하는 데 사용되는 팩토리 함수는 추상 반환 타입과 동일한 이름을 가질 수 있다.
```kotlin
interface Foo { /*...*/ }

class FooImpl : Foo { /*...*/ }

fun Foo(): Foo { return FooImpl() }
```

## 2.2 Names for test methods
- 테스트에서 (그리고 오직 테스트에서만), 백틱으로 둘러싸인 공백이 있는 메소드 이름을 사용할 수 있다. 그러나 이런 메소드 이름은 현재 안드로이드 런타임에서 지원되지 않는다는 점을 주의하라.
- 메소드 이름에 밑줄을 사용하는 것도 테스트 코드에서는 허용된다.
```kotlin
class MyTestCase {
     @Test fun `ensure everything works`() { /*...*/ }

     @Test fun ensureEverythingWorks_onAndroid() { /*...*/ }
}
```

## 2.3 Property names
- 상수의 이름(상수로 표시된 프로퍼티, 또는 사용자 정의 get 함수가 없는 깊이 불변 데이터를 가진 최상위 또는 객체 val 프로퍼티)은 대문자 밑줄 구분(스크리밍 스네이크 케이스) 이름을 사용해야 한다.
```kotlin
const val MAX_COUNT = 8
val USER_NAME_FIELD = "UserName"
```
- 행동이 있는 객체 또는 가변 데이터를 가진 최상위 또는 객체 프로퍼티의 이름은 카멜 케이스 이름을 사용해야 한다.
```kotlin
val mutableCollection: MutableSet<String> = HashSet()
```
- 싱글톤 객체에 대한 참조를 가진 프로퍼티의 이름은 객체 선언과 동일한 명명 스타일을 사용할 수 있다.
```kotlin
val PersonComparator: Comparator<Person> = /*...*/
```
- 열거형 상수의 경우, 사용법에 따라 대문자 밑줄 구분 이름(스크리밍 스네이크 케이스) (예: `enum class Color { RED, GREEN }`) 또는 대문자 카멜 케이스 이름을 사용하는 것이 좋다

## 2.4 Names for backing properties
- <span style='background-color: #fff5b1'/>클래스에 개념적으로 동일하지만 하나는 공개 API의 일부이고 다른 하나는 구현 세부 사항인 두 개의 프로퍼티가 있다면, 비공개 프로퍼티의 이름에는 밑줄을 접두사로 사용하라. 
```kotlin
class C {
    private val _elementList = mutableListOf<Element>()

    val elementList: List<Element>
         get() = _elementList
}
```

## 2.5 Choose good names
- 클래스의 이름은 보통 명사 또는 클래스가 무엇인지를 설명하는 명사구이다 : List, PersonReader


- 메소드의 이름은 보통 메소드가 무엇을 하는지를 말하는 동사 또는 동사구이다 : close, readPersons
- 이름은 메소드가 객체를 변형하는지 아니면 새로운 것을 반환하는지도 나타내야 한다. 예를 들어, sort는 컬렉션을 제자리에서 정렬하는 반면, sorted는 컬렉션의 정렬된 복사본을 반환한다.


- <span style='background-color: #fff5b1'/>이름은 엔티티의 목적이 무엇인지 명확하게 해야 하므로, 이름에서 의미없는 단어(Manager, Wrapper)를 사용하는 것을 피하는 것이 좋다.


- <span style='background-color: #fff5b1'/>선언 이름의 일부로 약어를 사용할 때는 두 글자로 이루어진 경우 대문자로 표시하고(IOStream), 길이가 더 긴 경우에는 첫 글자만 대문자로 표시한다(XmlFormatter, HttpInputStream).

# 3. Formatting
## 3.1 Indentation (들여쓰기)
- 들여쓰기에는 네 개의 공백을 사용하라. 탭은 사용하지 마라.
- 중괄호에 대해서는, 구조가 시작되는 줄의 끝에 여는 중괄호를 놓고, 닫는 중괄호를 여는 구조와 수평으로 정렬된 별도의 줄에 놓아라.
```kotlin
if (elements != null) {
    for (element in elements) {
        // ...
    }
} 
```
- 코틀린에서 세미콜론은 선택적이며, 따라서 줄바꿈이 중요하다. 언어 디자인은 자바 스타일의 중괄호를 가정하고 있으며, 다른 형식의 스타일을 사용하려고 하면 놀랄 만한 동작을 경험할 수 있다.

## 3.2 Horizontal whitespace (가로 공백)
- 이진 연산자 주위에는 공백을 넣는다(a + b).
  - 예외: `range to 연산자 주위에는 공백을 넣지 않는다(0..i).


- 제어 흐름 키워드(if, when, for, while)와 해당하는 여는 괄호 사이에 공백을 넣는다.


- 기본 생성자 선언, 메소드 선언 또는 메소드 호출에서 여는 괄호 앞에 공백을 넣지 않는다.
```kotlin
class A(val x: Int)

fun foo(x: Int) { ... }

fun bar() {
    foo(1)
} 
```

- `(`, `[` 뒤에 또는 `]`, `)` 앞에는 절대로 공백을 넣지 마라.


- `.`이나 `?:` 주위에는 절대로 공백을 넣지 마라 : `foo.bar().filter { it > 2 }.joinToString()`, `foo?.bar()`


- `//` 뒤에는 공백을 넣어라 : `// this is a comment`


- 타입 파라미터를 지정하기 위해 사용되는 꺽쇠 괄호 <> 주위에는 공백을 넣지 마라 : `class Map<K, V> { ... }`


- `::` 주위에는 절대로 공백을 넣지 마라 : `Foo::class`, `String::length`


- 널 가능 타입을 표시하는 데 사용되는 `?` 앞에는 공백을 넣지 마라 : `String?`


- 일반적인 규칙으로, 모든 종류의 수평 정렬을 피해라. 식별자의 이름을 다른 길이의 이름으로 변경하더라도, 선언이나 사용의 형식에는 영향을 주지 않아야 한다.

## 3.3 Colon(:)
- 다음과 같은 경우에는 `:` 앞에 공백을 넣어라:
  - 타입과 상위 타입을 구분할 때
  - 슈퍼클래스 생성자 또는 같은 클래스의 다른 생성자에 위임할 때
  - object 키워드 다음에


- 선언과 그 타입을 구분할 때는 `:` 앞에 공백을 넣지 않는다.


- `:` 뒤에는 항상 공백을 넣는다.
```kotlin
// 타입과 상위 타입을 구분할 때
abstract class Foo<out T : Any> : IFoo {
    abstract fun foo(a: Int): T
}

// 슈퍼클래스 생성자 또는 같은 클래스의 다른 생성자에 위임할 때
class FooImpl : Foo() {
    constructor(x: String) : this(x) { /*...*/ }

    val x = object : IFoo { /*...*/ }
}

// 선언과 그 타입을 구분할 때
val foo: Int = 1

// ':' 뒤에는 항상 공백을 넣는다.
val foo: Int = 1
class Foo : Bar {
    ...
}
```

## 3.4 Class headers
- 기본 생성자 매개변수가 몇 개인 클래스는 한 줄로 작성할 수 있다.
```kotlin
class Person(id: Int, name: String)
```
- 헤더가 긴 클래스는 각 기본 생성자 매개변수가 들여쓰기가 적용된 별도의 줄에 위치하도록 포맷되어야 한다. 또한 닫는 괄호는 새로운 줄에 위치해야 한다.
- 만약 상속을 사용한다면, 슈퍼클래스 생성자 호출 또는 구현된 인터페이스 목록은 괄호와 같은 줄에 위치해야 한다.
```kotlin
class Person(
    id: Int,
    name: String,
    surname: String
) : Human(id, name) { /*...*/ }
```
- 여러 인터페이스의 경우, 슈퍼클래스 생성자 호출이 먼저 위치해야 하며, 각 인터페이스는 다른 줄에 위치해야 한다.
```kotlin
class Person(
    id: Int,
    name: String,
    surname: String
) : Human(id, name),
    KotlinMaker { /*...*/ }
```
- 슈퍼타입 목록이 긴 클래스의 경우, 콜론 다음에 줄바꿈을 하고 모든 슈퍼타입 이름을 수평으로 정렬한다.
```kotlin
class MyFavouriteVeryLongClassHolder :
    MyLongHolder<MyFavouriteVeryLongClass>(),
    SomeOtherInterface,
    AndAnotherOne {

    fun foo() { /*...*/ }
}
```
- 클래스 헤더가 긴 경우 클래스 헤더와 본문을 명확하게 구분하기 위해, 클래스 헤더 다음에 빈 줄을 넣거나, 여는 중괄호를 별도의 줄에 놓는다.
```kotlin
// 클래스 헤더 다음에 빈 줄을 넣기
class MyFavouriteVeryLongClassHolder :
    MyLongHolder<MyFavouriteVeryLongClass>(),
    SomeOtherInterface,
    AndAnotherOne {

    fun foo() { /*...*/ }
}

// 여는 중괄호를 별도의 줄에 놓기
class MyFavouriteVeryLongClassHolder :
    MyLongHolder<MyFavouriteVeryLongClass>(),
    SomeOtherInterface,
    AndAnotherOne
{
    fun foo() { /*...*/ }
}
```
- 생성자 매개변수에는 일반 들여쓰기(네 개의 공백)를 사용한다. 이렇게 하면 기본 생성자에서 선언된 프로퍼티가 클래스 본문에서 선언된 프로퍼티와 동일한 들여쓰기를 갖게 된다.

## 3.5 Modifiers order(순서)
- 선언에 여러 개의 수식어가 있는 경우, 항상 다음 순서로 배치하라.
```kotlin
public / protected / private / internal
expect / actual
final / open / abstract / sealed / const
external
override
lateinit
tailrec
vararg
suspend
inner
enum / annotation / fun // as a modifier in `fun interface`
companion
inline / value
infix
operator
data 
```
- 모든 어노테이션은 수정자 앞에 위치시켜라.
```kotlin
@Named("Foo")
private val foo: Foo 
```
- 라이브러리를 작업하는 경우가 아니라면, 불필요한 수식어(예: public)는 생략하라.

## 3.6 Annotations
- 어노테이션은 그것이 첨부된 선언 전에 별도의 줄에 위치시키고, 동일한 들여쓰기를 적용하라.
```kotlin
@Target(AnnotationTarget.PROPERTY)
annotation class JsonExclude 
```
- 인수가 없는 어노테이션은 같은 줄에 위치시킬 수 있다.
```kotlin
@JsonExclude @JvmField
var x: String 
```
- 인수가 없는 단일 어노테이션은 해당 선언과 같은 줄에 배치될 수 있다.
```kotlin
@Test fun foo() { /*...*/ }
```

## 3.7 File annotations
- 파일 어노테이션은 파일 어노테이션(있는 경우) 다음, 패키지 선언 전에 위치하며, 패키지와는 빈 줄로 구분된다((이것은 그들이 패키지가 아닌 파일을 대상으로 한다는 사실을 강조하기 위함이다).
```kotlin
/** License, copyright and whatever */
@file:JvmName("FooBar")

package foo.bar
```

## 3.8 Functions
- 만약 함수 시그러니가 한 줄에 맞지 않는 경우, 다음과 같은 문법을 사용하라.
```kotlin
fun longMethodName(
    argument: ArgumentType = defaultValue,
    argument2: AnotherArgumentType,
): ReturnType {
    // body
}
```
- 함수 매배견수에는 일반 들여쓰기(4 개의 공백)를 사용하라. 이렇게 함으로써 생성자 매개변수와 일관성을 유지할 수 있다.

- <span style='background-color: #fff5b1'/>함수의 본문이 단일 표현식으로 구성된 경우 표현식 본문을 사용하는 것을 선호하라.
```kotlin
fun foo(): Int {     // bad
    return 1
}

fun foo() = 1        // good
```

## 3.9 Expression bodies
- 만약 함수가 표현식 본문을 가지고 있고 그 첫 번째 줄이 선언과 같은 줄에 맞지 않는 경우, 첫 번째 줄에 `=` 기호를 두고 표현식 본문을 네 칸 들여써라.
```kotlin
fun f(x: String, y: String, z: String) =
    veryLongFunctionCallWithManyWords(andLongParametersToo(), x, y, z)
```

## 3.10 Properties
- 매우 간단한 읽기 전용 프로퍼티의 경우, 한 줄 형식을 고려해보라.
```kotlin
val isEmpty: Boolean get() = size == 0
```
- 더 복잡한 프로퍼티의 경우, 항상 get과 set 키워드를 별도의 줄에 놓는 것을 고려해보라.
```kotlin
val foo: String
    get() { /*...*/ } 
```
- 초기화가 있는 프로퍼티의 경우, 초기화자가 길 경우 `=` 기호 뒤에 줄 바꿈을 추가하고 초기화를 네 개의 공백으로 들여써라.
```kotlin
private val defaultCharset: Charset? =
    EncodingRegistry.getInstance().getDefaultCharsetForPropertiesFiles(file)
```

## 3.11 Control flow statements (제어 흐름문)
- if 또는 when 문의 조건이 여러 줄인 경우, 항상 문장의 본문 주위에 중괄호를 사용하라.
- 조건문의 각 후속 줄은 문장 시작에 대해 네 개의 공백으로 들여써라.
- 조건의 닫는 괄호를 별도의 줄에 있는 여는 중괄호와 함께 놓아라.
```kotlin
if (!component.isSyncing &&
    !hasAnyKotlinRuntimeInScope(module)
) {
    return createKotlinNotConfiguredPanel(module)
}
```
- else, catch, finally 키워드와 do-while 루프의 while 키워드를 선행하는 중괄호와 같은 줄에 놓아라.
  - 이 방법은 조건과 문장 본문을 정렬하는 데 도움이 된다.
```kotlin
if (condition) {
    // body
} else {
    // else part
}

try {
    // body
} finally {
    // cleanup
}
```
- when 문에서, 한 브랜치가 한 줄 이상인 경우, 인접한 case 블록과 빈 줄로 분리하는 것을 고려하라.
```kotlin
private fun parsePropertyValue(propName: String, token: Token) {
    when (token) {
        is Token.ValueToken ->
            callback.visitValue(propName, token.value)

        Token.LBRACE -> { // ...
        }
    }
}
```
- 짧은 브랜치는 조건과 동일한 줄에 중괄호 없이 배치하라.
```kotlin
when (foo) {
    true -> bar() // good
    false -> { baz() } // bad
}
```

## 3.12 Method calls
- 긴 인수 목록에서는 여는 괄호 다음에 줄 바꿈을 넣어라. 인수를 네 칸 들여써라.
- <span style='background-color: #fff5b1'/>밀접하게 관련된 여러 인수를 같은 줄에 그룹화한다.
```kotlin
drawSquare(
    x = 10, y = 10,
    width = 100, height = 100,
    fill = true
)
```
- 인수 이름과 값 사이를 구분하는 = 기호 주위에 공백을 넣어라.

## 3.13 Wrap chained calls
- 연결된 호출을 줄 바꿈할 때는 `.` 문자나 `?.` 연산자를 다음 줄에 넣고, 한 칸 들여써라.
```kotlin
val anchor = owner
    ?.firstChild!!
    .siblings(forward = true)
    .dropWhile { it is PsiComment || it is PsiWhiteSpace } 
```
- 체인의 첫 번째 호출은 일반적으로 그 앞에 줄 바꿈이 있어야 하지만, 코드가 더 이해하기 쉬울 경우 생략해도 괜찮다.

## 3.14 Lambdas
- 람다 표현식에서는 중괄호 주변에 공백을 사용해야하며, 또한 매개변수와 본문을 구분하는 화살표 주변에도 공백을 사용해야한다.
- 단일 람다를 전달하는 경우 가능한 경우 괄호 외부로 전달하라.
```kotlin
list.filter { it > 10 }
```
- 람다에 레이블을 할당하는 경우 레이블과 여는 중괄호 사이에 공백을 넣지마라.
```kotlin
fun foo() {
    ints.forEach lit@{
        // ...
    }
}
```
- 다중 라인 람다에서 매개변수 이름을 선언할 때는 첫 번째 줄에 이름을 두고, 그 다음에 화살표와 줄 바꿈을 넣어라.
```kotlin
appendCommaSeparated(properties) { prop ->
    val propertyValue = prop.get(obj)  // ...
}
```
- 매개변수 목록이 한 줄에 들어가지 않을 정도로 길다면, 화살표를 별도의 줄에 둬라.
```kotlin
foo {
        context: Context,
        environment: Env
    ->
    context.configureEnv(environment)
} 
```

## 3.15 Trailing commas
- 마지막 콤마는 요소 시리즈의 마지막 항목 뒤에 오는 콤마 기호를 의미한다.
```kotlin
class Person(
    val firstName: String,
    val lastName: String,
    val age: Int, // trailing comma
)
```
#### 마지막 콤아 사용 이점
- 버전 컨트롤 diff를 깔끔하게 만든다 - 모든 초점이 변경된 값에 집중된다.
- 요소를 추가하고 재정렬하는 것이 쉬워진다. - 요소를 조작하더라도 콤마를 추가하거나 삭제할 필요가 없다.
- 코드 생성을 단순화한다.  예를 들어, 객체 초기화자의 경우 마지막 요소에도 콤마를 사용할 수 있다.


- 마지막 콤마는 완전히 선택 사항이다. `-` 이것 없이도 코드는 작동한다. 코틀린 스타일 가이드는 선언 위치에서 마지막 콤마 사용을 권장하며, 호출 위치의 경우 사용자의 재량에 따른다.

# 4. Documentation comments
- 긴 문서 주석의 경우, 개행을 사용하여 /**를 별도의 줄에 위치시키고 이후 각 줄의 시작에 별표(*)를 붙인다.
```kotlin
/**
 * This is a documentation comment
 * on multiple lines.
 */
```
- 짧은 주석은 한 줄에 배치할 수 있다.
```kotlin
/** This is a short documentation comment. */
```
- 일반적으로 `@param` 및 `@return` 태그의 사용을 피하라. 대신, 매개변수와 반환 값의 설명을 문서 주석에 직접 포함시키고, 매개변수가 언급될 때마다 링크를 추가하라. 주 텍스트의 흐름에 맞지 않는 긴 설명이 필요할 때만 @param 및 @return을 사용하라.
```kotlin
// Avoid doing this:

/**
 * 주어진 숫자의 절댓값을 반환한다.
 * @param 숫자 절댓값을 반환할 숫자.
 * @return 절댓값.
 */
fun abs(number: Int): Int { /*...*/ }

// Do this instead:

/**
 * 주어진 [숫자]의 절댓값을 반환한다.
 */
fun abs(number: Int): Int { /*...*/ } 
```

# 5. Avoid redundant constructs (중복 구성 방지)
- 일반적으로 코틀린에서 특정 구문 구조가 선택 사항이며 IDE에서 불필요하다고 강조하는 경우, 코드에서 생략해야 한다. "명확성을 위해" 불필요한 구문 요소를 코드에 남겨두지 마라.  

## 5.1 Unit return type
- 함수가 Unit을 반환하는 경우, 반환 타입은 생략해야 한다.
```kotlin
fun foo() { // ": Unit" is omitted here

}
```

## 5.2 Semicolons
- 가능한 한 세미콜론을 생략하라.

## 5.3 String templates
- 문자열 템플릿에 간단한 변수를 삽입할 때 중괄호를 사용하지 마라. 긴 표현식에만 중괄호를 사용하라. 
```kotlin
println("$name has ${children.size} children")
```

# 6. Idiomatic use of language features (언어 기능의 관용적 사용)
## <span style='background-color: #fff5b1'/>6.1 Immutability
- 불변 데이터를 사용하는 것이 좋다. 초기화 후에 수정되지 않는 경우, 로컬 변수와 프로퍼티는 항상 var 대신 val로 선언해야 한다.
- 변경되지 않는 컬렉션을 선언할 때는 항상 불변 컬렉션 인터페이스(Collection, List, Set, Map)를 사용하라.
- 컬렉션 인스턴스를 생성하기 위해 팩토리 함수를 사용할 때는 가능한 한 불변 컬렉션 타입을 반환하는 함수를 사용하라.
```kotlin
// Bad: 변경되지 않는 값에 변경 가능한 컬렉션 유형을 사용한다.
fun validateValue(actualValue: String, allowedValues: HashSet<String>) { ... }

// Good: 대신에 불변 컬렉션 유형를 사용한다.
fun validateValue(actualValue: String, allowedValues: Set<String>) { ... }

// Bad: arrayListOf()는 변경 가능한 컬렉션 유형인 ArrayList<T>를 반환환다.
val allowedValues = arrayListOf("a", "b", "c")

// Good: listOf()는 List<T>를 반환한다.
val allowedValues = listOf("a", "b", "c") 
```

## 6.2 Default parameter values
- 오버로드된 함수를 선언하는 것보다 기본 매개변수 값을 가진 함수를 선언하는 것이 좋다.
```kotlin
// Bad
fun foo() = foo("a")
fun foo(a: String) { /*...*/ }

// Good
fun foo(a: String = "a") { /*...*/ } 
```

## 6.3 Type aliases (타입 별칭)
- 코드베이스에서 여러 번 사용되는 함수형 타입이나 타입 매개변수가 있는 타입이 있다면, 이에 대한 타입 별칭을 정의하는 것이 좋다.
```kotlin
typealias MouseClickHandler = (Any, MouseEvent) -> Unit
typealias PersonIndex = Map<String, Person>
```
- 이름 충돌을 피하기 위해 private 또는 internal 타입 별칭을 사용한다면, '패키지와 임포트'에서 언급한 `import ... as ...`를 선호하는 것이 좋다.

## 6.4 Lambda parameters
- 짧고 중첩되지 않은 람다에서는 매개변수를 명시적으로 선언하는 대신 it 컨벤션을 사용하는 것이 권장된다.
- <span style='background-color: #fff5b1'/>매개변수가 있는 중첩 람다에서는 항상 매개변수를 명시적으로 선언해야 한다.
```kotlin
// Bad
val doubled = listOf(1, 2, 3).map { number -> number * 2 }

// Good
val doubled = listOf(1, 2, 3).map { it * 2 }

listOf(1, 2, 3).map { number ->
    listOf('a', 'b', 'c').map { char ->
        // 이곳에서 'it'을 사용하면 어떤 'it'인지 명확하지 않다.
        // 따라서 매개변수를 명시적으로 선언하는 것이 좋다.
    }
}
```

## 6.5 Returns in a lambda
- 람다에서 여러 레이블이 붙은 반환을 사용하는 것은 피해라. 람다가 단일 종료 지점을 가질 수 있도록 구조를 재구성하라. 만약 그것이 불가능하거나 충분히 명확하지 않다면, 람다를 익명 함수로 변환하는 것을 고려하라.
- 람다의 마지막 문장에 레이블이 붙은 반환을 사용하지 마라.
```kotlin
// Bad
val result = run loop@{
    listOf(1, 2, 3).forEach {
        if (it == 2) return@loop it
    }
    return@loop null
}

// Good
val result = listOf(1, 2, 3).firstOrNull { it == 2 }
```

## 6.6 Named arguments
- 메서드가 동일한 기본 타입의 여러 매개변수를 취하거나 Boolean 타입의 매개변수에 대해서는, 모든 매개변수의 의미가 문맥에서 절대적으로 명확하지 않는 한, 명명된 인수 구문을 사용하는 것이 좋다.
```kotlin
drawSquare(x = 10, y = 10, width = 100, height = 100, fill = true)
```

## 6.7 Conditional statements (조건문)
- try, if, 그리고 when의 표현식 형태를 사용하는 것이 좋다.
```kotlin
return if (x) foo() else bar()
```
```koltin
return when(x) {
    0 -> "zero"
    else -> "nonzero"
}
```
- 위의 내용이 다음보다 바람직하다.
```kotlin
if (x)
    return foo()
else
    return bar()
```
```kotlin
when(x) {
    0 -> return "zero"
    else -> return "nonzero"
}
```

## 6.8 if versus when
- 이진 조건에 대해서는 when 대신 if를 사용하는 것이 좋다.
```kotlin
if (x == null) ... else ...
```
- `When`을 사용하는 대신
```kotlin
when (x) {
    null -> // ...
    else -> // ...
}
```
- <span style='background-color: #fff5b1'/>옵션이 3개 이상일 때 사용하는 것이 좋다.

## 6.9 Nullable Boolean values in conditions
- 조건문에서 nullable Boolean을 사용해야 하는 경우, `if (value == true)` 또는 `if (value == false)` 검사를 사용하는 것이 좋다.

## <span style='background-color: #fff5b1'/>6.10 Loops
- 루프 대신 고차 함수(filter, map 등)를 사용하는 것이 좋다.
  - 예외: forEach (forEach의 수신자가 nullable이거나 forEach가 더 긴 호출 체인의 일부로 사용되는 경우를 제외하고는, 일반 for 루프를 사용하는 것이 좋다.)
```kotlin
data class Item(val name: String)

fun processItems(items: List<Item>?) {
    // Nullable 체크 후 forEach 사용
    if (items != null) {
        items.forEach {
            println(it.name)
        }
    }

    // 안전한 호출 연산자와 forEach 사용
    items?.forEach {
        println(it.name)
    }

    // forEach가 더 긴 호출 체인의 일부로 사용될 때
    val result = someFunctionReturningNullableList()?.forEach {
        // 이 부분에서 각 아이템을 처리
        println(it.name)
    }
}

fun someFunctionReturningNullableList(): List<Item>? {
    return emptyList()
}

fun main() {
    val items: List<Item>? = listOf(Item("A"), Item("B"), Item("C"))
    processItems(items)
} 
```

## 6.11 Loops on ranges
- 범위를 열린 범위로 반복할 때 `..<` 연산자를 사용하라.
```kotlin
for (i in 0..n - 1) { /*...*/ }  // bad
for (i in 0..<n) { /*...*/ }  // good 
```
## 6.12 Strings
- 문자열 연결보다는 문자열 템플릿을 선호한다.
- 일반 문자열 리터럴에 이스케이프 시퀀스를 포함시키는 것보다는 여러 줄 문자열을 선호한다.
- 여러 줄 문자열에서 들여쓰기를 유지하려면, 결과 문자열이 내부 들여쓰기를 필요로하지 않을 때는 `trimIndent`를 사용하거나, 내부 들여쓰기가 필요한 경우에는 `trimMargin`를 사용하라.
```kotlin
println("""
    Not
    trimmed
    text
    """
       )

// 
// Not
// trimmed
// text
// 

println("""
    Trimmed
    text
    """.trimIndent()
       )

// Trimmed
// text

println()

val a = """Trimmed to margin text:
          |if(a > 1) {
          |    return a
          |}""".trimMargin()

println(a)

// Trimmed to margin text:
// if(a > 1) {
//     return a
// }
```

## 6.13 Functions vs properties
- 일부 경우에는 인수가 없는 함수를 읽기 전용 프로퍼티와 교환하여 사용할 수 있다. 의미론적으로는 비슷하지만, 어떤 것을 선호해야 할지에 대한 스타일 가이드라인이 있다.

#### 다음 조건을 만족하는 경우, 함수보다 프로퍼티를 선호한다. 
- 예외를 발생시키지 않는다.
- 계산 비용이 적거나(또는 첫 번째 실행에서 캐시된다).
- 객체 상태가 변경되지 않았다면 호출에 대해 항상 같은 결과를 반환한다.
```kotlin
// Before
fun calculateArea(): Double {
    return width * height
}

// After
val area: Double
    get() = width * height
```

## 6.14 Extension functions (확장 함수)
- 확장 함수를 자유롭게 사용하라. 객체를 주로 대상으로 작동하는 함수가 있을 때마다 해당 객체를 수신자로 받는 확장 함수로 만드는 것을 고려하라. 
- API 오염을 최소화하기 위해, 가능한 한 확장 함수의 가시성을 제한하라. 필요에 따라 로컬 확장 함수, 멤버 확장 함수, 또는 private 가시성을 가진 최상위 확장 함수를 사용하라.
```kotlin
fun String.isNumeric(): Boolean {
    return this.all { it.isDigit() }
}

val str = "12345"
println(str.isNumeric())  // true
```

## 6.15 Infix functions
- 함수를 중위(infix)로 선언하는 것은 두 객체가 유사한 역할을 수행할 때만 한다. 좋은 예로는 and, to, zip 등이 있다. 나쁜 예로는 add가 있다.
- 수신자 객체를 변경하는 경우에는 메서드를 중위로 선언하지마라.
#### Bad: 중위 함수로 선언하여 사용
```kotlin
class MutablePoint(var x: Int, var y: Int) {
    infix fun translate(pair: Pair<Int, Int>) {
        x += pair.first
        y += pair.second
    }
}

fun main() {
    val point = MutablePoint(10, 20)
    point translate (5 at 5)
    println("Translated Point: (${point.x}, ${point.y})")
}

infix fun Int.at(dy: Int) = Pair(this, dy)
```
#### Good: 중위 함수로 선언하지 않은 경우
```kotlin
class MutablePoint(var x: Int, var y: Int) {
    fun translate(dx: Int, dy: Int) {
        x += dx
        y += dy
    }
}

fun main() {
    val point = MutablePoint(10, 20)
    point.translate(5, 5)
    println("Translated Point: (${point.x}, ${point.y})") // Translated Point: (15, 25)
}
```

## 6.16 Factory functions
- 클래스에 대한 팩토리 함수를 선언할 때, 클래스 자체와 같은 이름을 주는 것은 피하라. 팩토리 함수의 동작이 특별한 이유를 명확하게 나타내는 독특한 이름을 사용하는 것을 선호하라. 정말로 특별한 의미가 없을 때만 클래스와 같은 이름을 사용할 수 있다. 
```kotlin
class Point(val x: Double, val y: Double) {
    companion object {
        fun fromPolar(angle: Double, radius: Double) = Point(...)
    }
}
```
- 다른 상위 클래스 생성자를 호출하지 않고, 기본 인자 값으로 단일 생성자로 줄일 수 없는 여러 개의 오버로드된 생성자를 가진 객체가 있다면, 오버로드된 생성자를 팩토리 함수로 대체하는 것이 좋다.
```kotlin
// Before
class User {
    constructor(name: String) { ... }
    constructor(name: String, age: Int) { ... }
    constructor(name: String, age: Int, address: String) { ... }
} 

// After
class User private constructor(name: String, age: Int, address: String) {
    companion object {
        fun createWithName(name: String) = User(name, 0, "")
        fun createWithNameAndAge(name: String, age: Int) = User(name, age, "")
        fun createWithNameAgeAndAddress(name: String, age: Int, address: String) = User(name, age, address)
    }
}
```

## 6.17 Platform types
- 플랫폼 타입의 표현식을 반환하는 공개 함수/메서드는 그것의 코틀린 타입을 명시적으로 선언해야 한다.
```kotlin
fun apiCall(): String = MyJavaApi.getProperty("name")
```
- 플랫폼 타입의 표현식으로 초기화된 모든 속성(패키지 레벨 또는 클래스 레벨)은 코틀린 타입을 명시적으로 선언해야 한다.
```kotlin
class Person {
    val name: String = MyJavaApi.getProperty("name")
}
```
- 플랫폼 타입의 표현식으로 초기화된 로컬 값은 타입 선언을 가질 수도 있고 가지지 않을 수도 있다.
```kotlin
fun main() {
    val name = MyJavaApi.getProperty("name")
    println(name)
}
```

## 6.18 [Scope functions](https://velog.io/@dev-baik/%EB%B2%94%EC%9C%84-%EC%A7%80%EC%A0%95-%ED%95%A8%EC%88%98)
- 코틀린은 주어진 객체의 컨텍스트에서 코드 블록을 실행하기 위한 일련의 함수를 제공한다 : let, run, with, apply, also
- 공식 문서 : https://kotlinlang.org/docs/scope-functions.html

# 7. Coding conventions for libraries
- 라이브러리를 작성할 때에는 API 안정성을 보장하기 위해 추가적인 규칙 집합을 따르는 것이 권장된다.
  - 항상 멤버 가시성을 명시적으로 지정하라(무심코 선언을 공개 API로 노출하는 것을 피하기 위해). 
  - 항상 함수의 반환 타입과 속성 타입을 명시적으로 지정하라(구현이 변경될 때 반환 타입이 무심코 변경되는 것을 피하기 위해).
  - 모든 공개 멤버에 대해 [KDoc](https://kotlinlang.org/docs/kotlin-doc.html) 주석을 제공하라. 단, 새로운 문서화가 필요하지 않은 오버라이드는 제외한다(라이브러리에 대한 문서화를 생성하는데 도움이 되기 때문이다).
- 라이브러리의 API를 작성할 때 고려해야 할 최고의 사례와 아이디어에 대해 더 자세히 알아보려면 [라이브러리 제작자 가이드라인](https://kotlinlang.org/docs/jvm-api-guidelines-introduction.html)을 참조하라.

#### 참고한 사이트
- https://kotlinlang.org/docs/coding-conventions.html