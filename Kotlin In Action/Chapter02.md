# 코틀린 기초
> - 함수, 변수, 클래스, enum, 프로퍼티를 선언하는 방법
> - 제어 구조
> - 스마트 캐스트
> - 예외 던지기와 예외 잡기

- - -

## 기본 요소: 함수와 변수
### Hello, World!
```kotlin
fun main(args: Array<String>) {
  println("Hello, world!")
}
```
- 함수를 선언할 때 fun 키워드를 사용한다. **실제로도 코틀린 프로그래밍은 수많은 fun을 만드는 재미있는 일이다!**
- 파라미터 이름 뒤에 그 파라미터의 타입을 쓴다.
- 함수를 최상위 수준에 정의할 수 있다. (자바와 달리) 꼭 클래스 안에 함수를 넣어야 할 필요가 없다.
- 자바와 달리 배열 처리를 위한 문법이 따로 존재하지 않는다.
- 코틀린 표준 라이브러리는 여러 가지 표준 자바 라이브러리 함수를 간결하게 사용할 수 있게 감싼 래퍼를 제공한다.
  - System.out.println ➡️ println
- 최신 프로그래밍 언어 경향과 마찬가지로 줄 끝에 세미클론(;)을 붙이지 않아도 좋다.

### 함수
- 함수 이름, 파라미터 목록, 반환 타입, 함수 본문


- 블록이 본문인 함수 : 본문이 중괄호로 둘러싸인 함수
```kotlin
fun max(a: Int, b: Int): Int {
    return if (a > b) a else b
}

println(max(1, 2)) // 2
```

#### 문(statement)과 식(expression)의 구분
> 코틀린에서 if는 식이지 문이 아니다.
> - 식은 값을 만들어 내며 다른 식의 하위 요소로 계산해 참여할 수 있는 반면 문은 자신을 둘러싸고 있는 가장 안쪽 블록의 최상위 요소로 존재하며 아무런 값을 만들어내지 않는다는 차이가 있다.
> - 자바에서는 모든 제어 구조가 문인 반면 코틀린에서는 루프를 제외한 대부분의 제어 구조가 식이다.
> - 제어 구조를 다른 식으로 엮어낼 수 있으면 여러 일반적인 패턴을 아주 간결하게 표현할 수 있다.
> - 반면 대입문은 자바에서는 식이었으나 코틀린에서는 문이 됐다. 그로인해 자바와 달리 대입식과 비교식을 잘못 바꿔 써서 버그가 생기는 경우가 없다.

#### 식이 본문인 함수
- 식이 본문인 함수 : 등호와 식으로 이뤄진 함수
```kotlin
fun max(a: Int, b: Int): Int = if (a > b) a else b
```
- 반환 타입을 생략할 수 있는 이유는 무엇일까?
- 코틀린은 정적 타입 지정 언어이므로 컴파일 시점에 모든 식의 타입을 지정해야 하지 않는가?
> - 실제로 모든 변수나 모든 식에는 타입이 있으며, 모든 함수는 반환 타입이 정해져야 한다.
> - 하지만 식이 본문인 함수의 경우
    >   - 사용자가 반환 타입을 적지 않아도 컴파일러가 함수 본문 식을 분석해서 식의 결과 타입을 함수 반환 타입으로 정해준다.
          >     - 타입 추론 : 컴파일러가 타입을 분석해 프로그래머 대신 프로그램 구성 요소의 타입을 정해주는 기능
```kotlin
fun max(a: Int, b: Int) = if (a > b) a else b
```

### 변수
- 자바에서는 변수를 선언할 때 타입이 맨 앞에 온다.
  - 타입으로 변수 선언을 시작하면 타입을 생략할 경우 식과 변수 선언을 구별할 수 없다.
- 코틀린에서는 타입 지정을 생략하는 경우가 흔하다.
  - 키워드로 변수 선언을 시작하는 대신 변수 이름 뒤에 타입을 명시하거나 생략하게 허용한다.
```kotlin
val question = "삶, 우주, 그리고 모든 것에 대한 궁극적인 질문"

val answer = 42

val answer: Int = 42

val yearsToCompute = 7.5e5 // 7500000.0

val answer: Int
answer = 42
```
- 식이 본문인 함수에서와 마찬가지로 여러분이 타입을 지정하지 않으면 컴파일러가 초기화 식을 분석해서 초기화 식의 타입을 변수 타입으로 지정한다.
- 초기화 식이 없다면 변수에 저장될 값에 대해 아무 정보가 없기 때문에 컴파일러가 타입을 추론할 수 없다.
  - 따라서 그런 경우 타입을 반드시 지정해야 한다.

#### 변경 가능한 변수와 변경 불가능한 변수
- val(value) : 변경 불가능한 참조를 저장하는 변수다. val로 선언된 변수는 일단 초기화하고 나면 재대입이 불가능하다. 자바로 말하자면 final 변수에 해당한다.
  - val 변수는 블록을 실행할 떄 정확히 한 번만 초기화돼야 한다.
    - 하지만, 어떤 블록이 실행될 때 오직 한 초기화 문장만 실행됨을 컴파일러가 확인할 수 있다면 조건에 따라 val 값을 다른 여러 값으로 초기화할 수도 있다.
    ```kotlin
    val message: String
    if (canPerfromOperation()) {
        mesage = "Success"
    } else {
        message = "Failed"
    }
    ```
  - val 참조 자체는 불변일지라도 그 참조가 가리키는 객체의 내부 값은 변경될 수 있다.
  ```kotlin
  val languages = arrayListOf("Java")
  languages.add("Kotlin")
  ```
- var(variable) : 변경 가능한 참조다. 이런 변수의 값은 바뀔 수 있다. 자바의 일반 변수에 해당한다.
> 기본적으로는 모든 변수를 val 키워드를 사용해 불변 변수로 선언하고, 나중에 꼭 필요할 때에만 var로 변경하라.
- 변경 불가능한 참조와 변경 불가능한 객체를 부수 효과가 없는 함수와 조합해 사용하면 함수형 코드에 가까워진다.
- 변수의 값을 변경할 수 있지만 변수의 타입은 고정돼 바뀌지 않는다.
```kotlin
var answer = 42
answer = "no answer" // Error: type mismatch
```

### 더 쉽게 문자열 형식 지정: 문자열 템플릿
```kotlin
fun main(args: Array<String>) {
    val name = if (args.size > 0) args[0] else "Kotlin"
    // println("Hello, " + name + "!")
    println("Hello, $name")
    
    // $ 문자를 문자열에 넣고 싶으면 $를 이스케이프 시켜야한다.
    println("\$x") // $x
    
    // 문자열 템플릿 안에 사용할 수 있는 대상은 간단한 변수 이름만으로 한정되지 않는다.
    if (args.size > 0) {
        println("Hello, ${args[0]}!")
    }
    
    // 중괄호로 둘러싼 식 안에서 큰 따옴표를 사용할 수도 있다.
    println("Hello, ${if (args.size > 0) args[0] else "someone"}!")
}
```
- 컴파일러는 각 식을 정적으로 (컴파일 시점에) 검사하기 때문에 존재하지 않는 변수를 문자열 템플릿 안에서 사용하면 컴파일 오류가 발생한다.

## 클래스와 프로퍼티
```java
// 간단한 자바 클래스 Person
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
- 필드가 둘 이상으로 늘어나면 생성자인 Person(String name)의 본문에서 파라미터를 이름이 같은 필드에 대입하는 대입문의 수도 늘어난다.
- 자바에서는 생성자 본문에 이 같은 코드가 반복적으로 들어가는 경우가 많다.


- 코틀린에서는 그런 필드 대입 로직을 훨씬 더 적은 코드로 작성할 수 있다.
```kotlin
// 코틀린으로 변환한 Person 클래스
class Person(val name: String) 
```
- 값 객체 : 코드가 없이 데이터만 저장하는 클래스
- 코틀린의 기본 가시성은 public 이므로 이런 경우 변경자를 생략해도 된다.

### 프로퍼티
- 클래스 목적
> 데이터를 캡슐화하고 캡슐화한 데이터를 다루는 코드를 한 주체 아래 가두는 것이다.
- 자바에서는 데이터를 필드에 저장하며, 멤버 필드의 가시성은 보통 비공개(private)이다.
- 클래스는 자신을 사용하는 클라이언트가 그 데이터에 접근하는 통로로 쓸 수 있는 접근자 메서드를 제공한다.
  - 보통은 필드를 읽기 위한 게터를 제공하고 필드를 변경하게 허용해야 할 경우 세터를 추가로 제공할 수 있다.
- 자바에서는 필드와 접근자를 한데 묶어 프로퍼티라고 부르며, 프로퍼티라는 개념을 활용하는 프레임워크가 많다.
- 코틀린은 프로퍼티를 언어 기본 기능으로 제공하며, 코틀린 프로퍼티는 자바의 필드와 접근자 메서드를 완전히 대신한다.
  - val로 선언한 프로퍼티는 읽기 전용이며, var로 선언한 프로퍼티는 변경 가능하다.
  ```kotlin
  // 클래스 안에서 변경 가능한 프로퍼티 선언하기
  class Person(
      // 읽기 전용 프로퍼티로, 코틀린은 (비공개) 필드와 필드를 읽는 단순한 (공개) 게터를 만들어낸다.
      val name: String,
      // 쓸 수 있는 프로퍼티로, 코틀린은 (비공개) 필드, (공개) 세터, (공개) 세터를 만들어낸다.
      var isMarried: Boolean
  ) 
  ```
- 코틀린에서 프로퍼티를 선언하는 방식은 프로퍼티와 관련 있는 접근자를 선언하는 것이다.
- 읽기 전용 프로퍼티의 경우 게터만 선언하며 변경할 수 있는 프로퍼티의 경우 게터와 세터를 모두 선언한다.
> 코틀린은 값을 저장하기 위한 비공개 필드와 그 필드에 값을 저장하기 위한 세터, 필드의 값을 읽기 위한 게터로 이뤄진 간단한 디폴트 접근자 구현을 제공한다.
```java
// 자바에서 Person 클래스를 사용하는 방법
Person person = new Person("Bob", true);
System.out.println(person.getName()); // Bob
System.out.println(person.isMarried()); // true
```
- 게터와 세터의 이름을 정하는 규칙에는 예외가 있다.
  - 이름이 is로 시작하는 프로퍼티의 게터에는 get이 붙지 않고 원래 이름을 그대로 사용하며, 세터에는 is를 set으로 바꾼 이름을 사용한다.
    - 따라서 자바에서 isMarried 프로퍼티의 게터를 호출하려면 isMarried()를 사용해야 한다.
```kotlin
// 코틀린에서 Person 클래스 사용하기
val person = Person("Bob", true) // new 키워드를 사용하지 않고 생성자 호출한다.
// 프로퍼티 이름을 직접 사용해도 코틀린이 자동으로 게터를 호출해준다.
println(person.name) // Bob
println(person.isMarried) // true
```
- 대부분의 프로퍼티에는 그 프로퍼티의 값을 저장하기 위한 필드가 있다.
  이를 프로퍼티를 뒷받침하는 필드라고 부른다. 하지만 원한다면 프로퍼티 값을 그때그때 계산(예를 들어 다른 프로퍼티들로부터 값을 계산할 수도 있다)할 수 도 있다.
  커스텀 게터를 작성하면 그런 프로퍼티를 만들 수 있다.

### 커스텀 접근자 : 프로퍼티의 접근자를 직접 작성하는 방법
```kotlin
class Rectangle(val height: Int, val width: Int) {
    // 직사각형이 정사각형인지를 별도의 필드에 저장할 필요가 없다.
    val isSquare: Boolean
      // get() {
      //    return height == width
      // }
      get() = height == width
}

val rectangle = Rectangle(41, 43)
println(rectangle.isSquare)
```
> 파라미터가 없는 함수를 정의하는 방식과 커스텀 게터를 정의하는 방식은 구현이나 성능상 차이는 없다. 차이가 나는 부분은 가독성뿐이다. 일반적으로 클래스의 특성을 정의하고 싶다면 프로퍼티로 그 특성을 정의해야 한다.

### 코틀린 소스코드 구조: 디렉터리와 패키지
- 자바의 경우 모든 클래스를 패키지 단위로 관리하다. 코틀린에서도 자바와 비슷한 개념의 패키지가 있다.
- 모든 코틀린 파일의 맨 앞에 package 문을 넣을 수 있다. 그러면 그 파일 안에 있는 모든 선언(클래스, 함수, 프로퍼티 등)이 해당 패키지에 들어간다.
- 같은 패키지에 속해있다면 다른 파일에서 정의한 선언일지라도 직접 사용할 수 있다.
- 반면 다른 패키지에서 정의한 선언을 사용하려면 임포트를 통해 선언을 불러와야 한다.
  - 자바와 마찬가지로 임포트문은 파일의 맨 앞에 와야 하며 import 키워드를 사용한다.
  ```kotlin
  // 클래스와 함수 선언을 패키지에 넣기
  package geometry.shape // 패키지 선언
  
  import java.util.Random // 표준 자바 라이브러리 클래스를 임포트한다.
  
  class Rectangle(val height: Int, val width: Int) {
      val isSquare: Boolean
        get() = height == width
  }
  
  fun createRandomRectangle(): Rectangle {
      val random = Random()
      return Rectangle(random.nextInt(), random.nextInt())
  }
  ```
  - 코틀린에서는 클래스 임포트와 함수 임포트에 차이가 없으며, 모든 선언을 import 키워드로 가져올 수 있다. 최상위 함수는 그 이름을 써서 임포트할 수 있다.
  ```kotlin
  // 다른 패키지에 있는 함수 임포트하기
  package geometry.example
  
  import geometry.shapes.createRandomRectangle // 이름으로 함수 임포트하기
  
  fun main(args: Array<String>) {
      println(createRandomRectangle().isSquare)
  } 
  ```
  - 패키지 이름 뒤에 .*를 추가하면 패키지 안의 모든 선언을 임포트할 수 있다.
  - 이런 스타 임포트를 사용하면 패키지 안에 있는 모든 클래스뿐 아니라 최상위에 정의된 함수나 프로퍼티까지 모두 불러온다는 점에 유의하라
- 자바에서는 패키지의 구조와 일치하는 디렉터리 계층 구조를 만들고 클래스의 소스코드를 그 클래스가 속한 패키지와 같은 디렉터리에 위치시켜야 한다.
- 코틀린에서는 원하는 대로 소스코드를 구성할 수 있다.
  - 여러 클래스를 한 파일에 넣을 수 있고, 파일의 이름도 마음대로 정할 수 있다.
  - 디스크상의 어느 디렉터리에 소스코드 파일을 위치시키든 관계없다.

> 하지만 대부분의 경우 자바와 같이 패키지별로 디렉터리를 구성하는 편이 낫다. 특히 자바와 코틀린을 함께 사용하는 프로젝트에서는 자바의 방식을 따르는 게 중요하다. 자바의 방식을 따르지 않으면 자바 클래스를 코틀린 클래스로 마이그레이션할 때 문제가 생길 수 있다.

> 하지만 여러 클래스를 한 파일에 넣는 것을 주저해서는 안 된다. 특히 각 클래스를 정의하는 소스코드 크기가 아주 작은 경우 더욱 그렇다(코틀린에서는 클래스 소스코드 크기가 작은 경우가 자주 있다).

## 선택 표현과 처리: enum과 when
### enum 클래스 정의
```kotlin
enum class Color {
    RED, ORANGE, YELLOW, GREEN, BLUE, INDIGO, VIOLET
}
```
- 코틀린에서 enum은 소프트 키워드라 부르는 존재다. class 앞에 있을 때는 특별한 의미를 지니지만 다른 곳에서는 이름에 사용할 수 있다.
- 반면 class는 키워드다. 따라서 class라는 이름을 사용할 수 없으므로 클래스를 표현하는 변수 등을 정의할 때는 clazz나 aClass와 같은 이름을 사용해야 한다.
- 자바와 마찬가지로 enum은 단순히 값만 열거하는 존재가 아니다. enum 클래스 안에도 프로퍼티나 메서드를 정의할 수 있다.
```kotlin
// 프로퍼티와 메소드가 있는 enum 클래스 선언하기
enum class Color(
    val r: Int, val g: Int, val b: Int // 상수의 프로퍼티를 정의한다.
) {
    RED(255, 0, 0), ORANGE(255, 165, 0), // 각 상수를 생성할 때, 그에 대한 프로퍼티 값을 지정한다.
    YELLOW(255, 255, 0), GREEN(0, 255, 0), BLUE(0, 0, 255),
    INDIGO(75, 0, 130), VIOLET(238, 130, 238); // 여기 반드시 세미콜론을 사용해야 한다.
  
    fun rgb() = (r * 256+ g) * 256 + b // enum 클래스 안에서 메소드를 정의한다.
}

println(Color.BLUE.rgb()) // 255
```
- enum 클래스 안에 메소드를 정의하는 경우 반드시 enum 상수 목록과 메소드 정의 사이에 세미콜론을 넣어야 한다.

### when으로 enum 클래스 다루기
```kotlin
// when을 사용해 올바른 enum 값 찾기
fun getMnemonic(color: Color) =
    when (color) {
        Color.RED -> "Richard"
        Color.ORANGE -> "Of"
        Color.YELLOW -> "York"
        Color.GREEN -> "GAVE"
        Color.BLUE -> "Battle"
        Color.INDIGO -> "In"
        Color.VIOLET -> "Vain"
    }

println(getMnemonic(Color.BLUE)) // Battle
```
- 자바의 switch에 해당하는 코틀린 구성 요소는 when이다.
- color로 전달된 값과 같은 분기를 찾는다. 자바와 달리 각 분기의 끝에 break을 넣지 않아도 된다.
```kotlin
// 한 when 분기 안에 여러 값 사용하기
fun getWarmth(color: Color) = when(color) {
    Color.RED, Color.ORANGE, Color.YELLOW -> "warm"
    Color.GREEN -> "neutral"
    Color.BLUE, Color.INDIGO, Color.VIOLET -> "cold"
}

println(getWarmth(Color.ORANGE)) // warm
```
```kotlin
// enum 상수 값을 임포트해서 enum 클래스 수식자 없이 enum 사용하기
import ch02.colors.Color // 다른 패키지에서 정의한 Color 클래스를 임포트한다.
import ch02.colors.Color.* // 짧은 이름으로 사용하기 위해 enum 상수를 모두 임포트한다.

fun getWarmth(color: Color) = when(color) {
    // 임포트한 enum 상수를 이름만으로 사용한다.
    RED, ORANGE, YELLOW -> "warm"
    GREEN -> "neutral"
    BLUE, INDIGO, VIOLET -> "cold"
}
```

### when과 임의의 객체를 함께 사용
- 코틀린에서 when은 자바의 switch보다 훨씬 강력하다. 분기 조건에 상수(enum 상수나 숫자 리터럴)만을 사용할 수 있는 자바 switch와 달리 코틀린 when의 분기 조건은 임의의 객체를 허용한다.
```kotlin
// when의 분기 조건에 여러 다른 객체 사용하기
fun mix(c1: Color, c2: Color) =
    when (setOf(c1, c2)) {
        setOf(RED, YELLOW) -> ORANGE
        setOf(YELLOW, BLUE) -> GREEN
        setOf(BLUE, VIOLET) -> INDIGO
        else -> throw Exception("Dirty color")
    }

println(mix(BLUE, YELLOW)) // GREEN
```
> 위 함수는 호출될 때마다 함수 인자로 주어진 두 색이 when의 분기 조건에 있는 다른 두 색과 같은지 비교하기 위해 여러 Set 인스턴스를 생성한다. 보통은 이런 비효율성이 크게 문제가 되지 않는다.

> 하지만 이 함수가 아주 자주 호출된다면 불필요한 가비지 객체가 늘어나는 것을 방지하기 위해 함수를 고쳐쓰는 편이 낫다.

### 인자 없는 when 사용
- 코드는 약간 읽기 어려워지지만 성능을 더 향상시키기 위해 그 정도 비용을 감수해야 하는 경우도 자주 있다.
```kotlin
// 인자가 없는 when
fun mixOptimized(c1: Color, c2: Color) =
    when {
      (c1 == RED && c2 == YELLOW) ||
              (c1 == YELLOW && c2 == RED) -> ORANGE
      (c1 == YELLOW && c2 == BLUE) ||
              (c1 == BLUE && c2 == YELLOW) -> GREEN
      (c1 == BLUE && c2 == VIOLET) ||
              (c1 == VIOLET && c2 == BLUE) -> INDIGO
      else -> throw Exception("Dirty color")
    }

println(mixOptimized(BLUE, YELLOW)) // GREEN
```
> 추가 객체를 만들지 않는다는 장점이 있지만 가독성은 더 떨어진다.

### 스마트 캐스트: 타입 검사와 타입 캐스트를 조합
```kotlin
// 식을 표현하는 클래스 계층 : 우선 식을 인코딩하는 방법을 생각해야 한다. 식을 트리 구조로 저장하자.
// 아무 메서드도 선언하지 않으며, 단지 여러 타입의 식 객체를 아우르는 공통 타입 역할만 수행한다.
interface Expr
// value라는 프로퍼티만 존재하는 단순한 클래스로 Expr 인터페이스를 구현한다.
class Num(val value: Int) : Expr
// Expr 타입의 객체라면 어떤 것이나 Sum 연산의 인자가 될 수 있다. 따라서 Num이나 다른 Sum이 인자로 올 수 있다.
class Sum(val left: Expr, val right: Expr) : Expr

println(eval(Sum(Sum(Num(1), Num(2)), Num(4)))) // 7
```
- Expr 인터페이스에는 두 가지 구현 클래스가 존재한다. 따라서 식을 평가하려면 두 가지 경우를 고려해야 한다.
  - 어떤 식이 수라면 그 값을 반환한다.
  - 어떤 식이 합계라면 좌항과 우항의 값을 계산한 다음에 그 두 값을 합한 값을 반환한다.
```kotlin
// if 연쇄를 사용해 식을 계산하기 (자바 스타일)
fun eval(e: Expr): Int {
    if (e is Num) {
        val n = e as Num // 여기서 Num으로 타입을 변환하는 데 이는 불필요한 중복이다.
        return n.value
    }
    if (e is Sum) {
        return eval(e.right) + eval(e.left)
    }
    throw IllegalArgumentException("Unknown expression")
}

println(eval(Sum(Sum(Num(1), Num(2)), Num(4)))) // 7
```
- 코틀린에서는 is를 사용해 변수 타입을 검사한다. is 검사는 자바의 instanceof와 비슷한다.
- 하지만 자바에서 어떤 변수의 타입을 instanceof로 확인한 다음에 그 타입에 속한 멤버에 접근하기 위해서는 명시적으로 변수 타입을 캐스팅해야 한다.
  - 이런 멤버 접근을 여러 번 수행해야 한다면 변수에 따로 캐스팅한 결과를 저장한 후 사용해야 한다.
- 코틀린에서는 프로그래머 대신 컴파일러가 캐스팅을 수행해준다. 이를 스마트 캐스트라고 부른다.
- 스마트 캐스트는 is로 변수에 든 값의 타입을 검사한 다음에 그 값이 바뀔 수 없는 경우에만 작동한다.
  - 예를들어 앞에서 본 예제처럼 클래스의 프로퍼티에 대해 스마트 캐스트를 사용한다면 그 프로퍼티는 반드시 val이어야 하며 커스텀 접근자를 사용한 것이어도 안된다.
    - val이 아니거나 val이지만 커스텀 접근자를 사용하는 경우에는 해당 프로퍼티에 대한 접근이 항상 같은 값을 내놓는다고 확신할 수 없기 때문이다.
      - 원하는 타입으로 명시적으로 타빙 캐스팅하려면 as 키워드를 사용한다.

### 리팩토링: if를 when으로 변경
- 코틀린에서는 if가 값을 만들어내기 때문에 자바와 달리 3항 연산자가 따로 없다.
```kotlin
// 값을 만들어내는 if 식
fun evel(e: Expr): Int =
    if (e is Num) {
        e.value
    } else if (e is Sum) {
        eval(e.right) + evel(e.left)
    } else {
        throw IllegalArgumentException("Unknown expression")
    }

println(evel(Sum(Num(1), Num(2))))
```
- if의 분기에 식이 하나밖에 없다면 중괄호를 생략해도 된다.
- if 분기에 블록을 사용하는 경우 그 블록의 마지막 식이 그 분기의 결과 값이다.
```kotlin
// if 중첩 대신 when 사용하기
fun evel(e: Expr): Int =
    when (e) {
        is Num ->
            e.value
        is Sum ->
            eval(e.right) + evel(e.left)
        else ->
            throw IllegalArgumentException("Unknown expression")
    }
```
> 타입을 검사하고 나면 스마트 캐스트가 이뤄진다. 따라서 Num이나 Sum의 멤버에 접근할 때 변수를 강제로 캐스팅할 필요가 없다.

- if나 when의 각 분기에서 수행해야 하는 로직이 복잡해지면 분기 본문에 블록을 사용할 수 있다.
- 그런 경우 블록의 마지막 문장이 블록 전체의 결과가 된다.
```kotlin
fun evalWithLogging(e: Expr): Int =
    when (e) {
        is Num -> {
            println("num: ${e.value}")
            e.value
        }
        is Sum -> {
            val left = evalWithLogging(e.left)
            val right = evalWithLogging(e.right)
            println("sum: $left + $right")
            left + right
        }
        else -> throw IllegalArgumentException("Unknown expression")
    }

println(evalWithLogging(Sum(Sum(Num(1), Num(2)), Num(4))))
//// 결과
// num: 1
// num: 2
// sum: 1 + 2
// num: 4
// sum: 3 + 4
// num: 7
```
- '블록의 마지막 식이 블록의 결과'라는 규칙은 블록이 값을 만들어내야 하는 경우 항상 성립한다.
- 이 규칙은 함수에 대해서는 성립하지 않는다. 식이 본문인 함수는 블록을 본문으로 가질 수 없고 블록이 본문인 함수는 내부에 return문이 반드시 있어야 한다.

## 대상을 이터레이션: while과 for 루프
### while 루프
- 두 루프의 문법은 자바와 다르지 않다.
```kotlin
while (조건) { // 조건이 참인 동안 본문을 반복 실행한다.
    /*...*/
}

do {
   /*...*/ 
} while (조건) // 맨 처음에 무조건 본문을 한 번 실행한 다음, 조건이 참인 동안 본문을 반복 실행한다.
```

### 수에 대한 이터레이션: 범위와 수열
- 코틀린에서는 자바의 for 루프(어떤 변수를 초기화하고 그 변수를 루프를 한 번 실행할 때마다 갱신하고 루프 조건이 거짓이 될 때 반복을 마치는 형태의 루프)에 해당하는 요소가 없다.
- 이런 루프의 가장 흔한 용례인 초깃값, 증가 값, 최종 값을 사용한 루프를 대신하기 위해 코틀린에서는 범위(range)를 사용한다.
- 범위는 기본적으로 두 값으로 이뤄진 구간이다. 보통은 그 두 값은 정수 등의 숫자 타입의 값이며, .. 연산자로 시작 값과 끝 값을 연결해서 범위를 만든다.
- 코틀린의 범위는 폐구간(닫힌 구간) 또는 양끝을 포함하는 구간이다
- 이런 식으로 어떤 범위에 속한 값을 일정한 순서로 이터레이션하는 경우를 수열이라고 부른다.
```kotlin
// when을 사용해 피즈버즈 게임 구현하기
fun fizzBuzz(i: Int) = when {
    i % 15 == 0 -> "FizzBuzz "
    i % 3 == 0 -> "Fizz "
    i % 5 == 0 -> "Buzz "
    else -> "$i "
}

// 결과: 1 2 Fizz 4 Buzz Fizz 7 ...
for (i in 1..100) {
    print(fizzBuzz(i))
}

// 결과: Buzz 98 Fizz 94 92 Fizz Buzz 88 ...
for (i in 100 downTo 1 step 2) {
    print(fizzBuzz(i))
}
```
- 끝 값을 포함하지 않는 반만 닫힌 범위(반폐구간 또는 반개 구간)에 대해 이터레이션하면 편할 때가 자주 있다.
  - 그런 범위를 만들고 싶다면 until 함수를 사용하라.
  - `for (x in 0 until size)`라는 루프는 `for (x in 0..size-1)`과 같지만 좀 더 명확하게 개념을 표현한다.

### 맵에 대한 이터레이션
```kotlin
// 맵을 초기화하고 이터레이션하기
val binaryReps = TreeMap<Char, String>() // 키에 대해 정렬하기 위해 TreeMap을 사용한다.

for (c in 'A'..'F') {
    val binary = Integer.toBinaryString(c.toInt()) // 아스키 코드를 2진 표현으로 바꾼다.
    binaryReps[c] = binary // c를 키로 c의 2진 표현을 맵에 넣는다.
}

// 맵에 대해 이터레이션한다. 맵의 키와 값을 두 변수에 각각 대입한다.
for ((letter, binary) in binaryReps) {
    println("$letter = $binary")
}
```
- 맵에 사용햇던 구조 분해 구문을 맵이 아닌 컬렉션에도 활용할 수 있다.
- 그런 구조 분해 구문을 사용하면 원소의 현재 인덱스를 유지하면서 컬렉션을 이터레이션할 수 있다.
- 인덱스를 저장하기 위한 변수를 별도로 선언하고 루프에서 매번 그 변수를 증가시킬 필요가 없다.
```kotlin
val list = arrayListOf("10", "11", "1001")
for ((index, element) in list.withIndex()) {
    println("$index: $element")
}

//// 결과
// 0: 10
// 1: 11
// 2: 1001
```

### in으로 컬렉션이나 범위의 원소 검사
- in 연산자를 사용해 어떤 값이 범위에 속하는지 검사할 수 있다. 반대로 !in을 사용하면 어떤 값이 범위에 속하지 않는지 검사할 수 있다.
```kotlin
// in을 사용해 값이 범위에 속하는 검사하기
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'

fun isNotDigit(c: Char) = c !in '0'..'9'

println(isLetter('q')) // true

println(isNotDigit('x')) // true
```
```kotlin
fun recognize(c: Char) = when (c) {
    in '0'..'9' -> "It's a digit!"
    in 'a'..'z', in 'A'..'Z' -> "It's a letter!"
    else -> "I don't know..."
}

println(recognize('8')) // It's a digit!
```
- 범위는 문자에만 국한되지 않는다. 비교가 가능한 클래스라면(java.lang.Comparable 인터페이스를 구현한 클래스라면) 그 클래스의 인스턴스 객체를 사용해 범위를 만들 수 있다.
- Comparable을 사용하는 범위의 경우 그 범위 내의 모든 객체를 항상 이터레이션하지는 못한다.
  - 'Java'와 'Kotlin' 사이의 모든 문자열을 이터레이션 할 수 없다.
    - 하지만 in 연산자를 사용하면 값이 범위 안에 속하는지 항상 결정할 수 있다.
    ```kotlin
    // String에 있는 Comparable 구현이 두 문자열을 알파벳 순서로 비교하기 때문에 
    // 여기 있는 in 검사에서도 문자열을 알파벳 순서로 비교한다.
    println("Kotlin" in "Java".."Scale") // true 
    ```
  - 컬렉션에도 마찬가지로 in 연산을 사용할 수 있다.
  ```kotlin
  println("Kotlin" in setOf("Java", "Scale")) // false
  ```

## 코틀린의 예외 처리
- 코틀린의 예외 처리는 자바나 다른 언어의 예외 처리와 비슷하다.
- 함수는 정상적으로 종료할 수 있지만 오류가 발생하면 예외를 던질 수 있다.
- 함수를 호출하는 쪽에서는 그 예외를 잡아 처리할 수 있다.
  - 발생한 예외를 함수 호출 단에서 처리하지 않으면 함수 호출 스택을 거슬러 올라가면서 예외를 처리하는 부분이 나올 때까지 예외를 다시 던진다.
```kotlin
if (percentage !in 0..100) {
    throw IllegalArgumentException(
        "A percentage value must be between 0 and 100: $percentage"
    )
}
```
- 다른 클래스와 마찬가지로 예외 인스턴스를 만들 때도 new를 붙일 필요가 없다.
- 자바와 달리 코틀린의 throw는 식이므로 다른 식에 포함될 수 있다.
```kotlin
val percentage =
    if (number in 0..100)
        number
    else
        throw IllegalArgumentException(
          "A percentage value must be between 0 and 100: $percentage")
```

### try, catch, finally
```kotlin
fun readNumber(reader: BufferedReader): Int? { // 함수가 던질 수 있는 예외를 명시할 필요가 없다.
    try {
        val line = reader.readLine()
        return Integer.parseInt(line)
    }
    catch (e: NumberFormatException) {
        return null
    }
    finally {
        reader.close()
    }
}

val reader = BufferedReader(StringReader("239"))
println(readNumber(reader)) // 239
```
- 자바 코드와 가장 큰 차이는 throws 절이 코드에 없다는 점이다.
- 자바에서는 함수를 작성할 때 함수 선언 뒤에 throws IOException을 붙여야 한다.
  - 이유는 IOException이 체크 예외이기 때문이다.
  - 자바에서는 체크 예외를 명시적으로 처리해야 한다.
- 어떤 함수가 던질 가능성이 있는 예외나 그 함수가 호출한 다른 함수에서 발생할 수 있는 예외를 모두 catch로 처리해야 하며, 처리하지 않은 예외는 throws 절에 명시해야 한다.
- 코틀린은 체크 예외나 언체크 예외를 구별하지 않는다.
  - 함수가 던지는 예외를 지정하지 않고 발생한 예외를 잡아내도 되고 잡아내지 않아도 된다.
- 자바는 체크 예외 처리를 강제한다.
  - 하지만 프로그래머들이 의미 없이 예외를 다시 던지거나, 예외를 잡되 처리하지는 않고 그냥 무시하는 코드를 작성하는 경우가 흔하다.
  - 그로 인해 예외 처리 규칙이 실제로는 오류 발생을 방지하지 못하는 경우가 자주 있다.

### try를 식으로 사용
```kotlin
fun readNumber(reader: BufferedReader) {
    val number = try {
        Integer.parseInt(reader.readLine())
    } catch (e: NumberFormatException) {
        return
    }
  
    println(number)
}

val reader = BufferedReader(StringReader("not a number"))
readNumber(reader) // 아무것도 출력되지 않는다.
```
- 코틀린의 try 키워드는 if나 when과 마찬가지로 식이다. 따라서 try의 값을 변수에 대입할 수 있다.
- if와 달리 try의 본문을 반드시 중괄호 {}로 둘러싸야 한다.
- 다른 문장과 마찬가지로 try의 본문도 내부에 여러 문장이 있으면 마지막 식의 값이 전체 결과 값이다.
```kotlin
fun readNumber(reader: BufferedReader) {
  val number = try {
    Integer.parseInt(reader.readLine())
  } catch (e: NumberFormatException) {
    null
  }

  println(number)
}

val reader = BufferedReader(StringReader("not a number"))
readNumber(reader) // null
```
- try 코드 블록의 실행이 정상적으로 끝나면 그 블록의 마지막 식의 값이 결과다.