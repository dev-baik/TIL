# 클래스, 객체, 인터페이스
> - 클래스와 인터페이스
> - 뻔하지 않은 생성자와 프로퍼티
> - 데이터 클래스
> - 클래스 위임
> - object 키워드 사용

- - -

## 클래스 계층 정의
### 코틀린 인터페이스
- 코틀린 인터페이스 안에는 추상 메서드뿐 아니라 구현이 있는 메소드도 정의할 수 있다.
  - 단, 인터페이스에는 아무런 상태(필드)도 들어갈 수 없다.
- 인터페이스를 구현하는 모든 비추상 클래스(또는 구체적 클래스)는 인터페이스에 대한 구현을 제공해야 한다.

```kotlin
interface Clickable {
    fun click()
}

class Button : Clickable {
    override fun click() = println("I was clicked")
}

Button().click() // I was clicked
```
- 클래스 이름 뒤에 콜론(:)을 붙이고 인터페이스와 클래스 이름을 적는 것으로 클래스 확장과 인터페이스 구현을 모두 처리한다.
- 자바와 마찬가지로 클래스는 인터페이스를 원하는 만큼 개수 제한 없이 마음대로 구현할 수 있지만, 클래스는 오직 하나만 확장할 수 있다.
- override 변경자 : 상위 클래스나 상위 인터페이스에 있는 프로퍼티나 메소드를 오버라이드한다는 표시
  - 자바와 달리 코틀린에서는 override 변경자를 꼭 사용해야 한다.
    - WHY) 실수로 상위 클래스의 메소드를 오버라이드하는 경우를 방지해준다.


- 인터페이스 메소드도 디폴트 구현을 제공할 수 있다. 그냥 메소드 본문을 메소드 시그니처 뒤에 추가하면 된다.
```kotlin
// 인터페이스 안에 본문이 있는 메소드 정의하기
interface Clickable {
    fun click() // 일반 메소드 선언
    fun showOff() = println("I'm clickable") // 디폴트 구현이 있는 메소드
}

// 동일한 메소드를 구현하는 다른 인터페이스 정의하기
interface Focusable {
    fun setFocus(b: Boolean) =
        println("I ${if (b) "got" else "lost"} focus.")

    fun showOff() = println("I'm focusable")
}
```
- 한 클래스에서 이 두 인터페이스(Clickable, Focusable)를 함께 구현하면 showOff 메소드가 중복되어 컴파일러 오류가 발생한다.
- 해결책) 하위 클래스에 직접 구현하게 강제해야한다.
```kotlin
// 상속한 인터페이스의 메소드 구현 호출하기
class Button : Clickable, Focusable {
    override fun click() = println("I was clicked")

    // 이름과 시그니처가 같은 멤버 메소드에 대해 둘 이상의 디폴트 구현이 있는 경우
    // 인터페이스를 구현하는 하위 클래스에서 명시적으로 새로운 구현을 제공해야 한다.
    override fun showOff() {
        // 상위 타입의 이름을 꺾쇠 괄호(<>) 사이에 넣어서 "super"를 지정하면
        // 어떤 상위 타입의 멤버 메소드를 호출할지 지정할 수 있다.
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
}

fun main(args: Array<String>) {
    val button = Button()
    button.showOff()
    button.setFocus(true)
    button.click()
}

//// 결과
// I'm clickable!
// I'm focusable!
// I got focus.
// I was clicked.
```
- 상속한 구현 중 단 하나만 호출해도 된다면
```kotlin
override fun showOff() = super<Clickable>.showOff() 
```

### open, final, abstract 변경자: 기본적으로 final
- 자바에서 final로 명시적으로 상속을 금지하지 않는 모든 클래스를 다른 클래스가 상속할 수 있다.
- 취약한 기반 클래스 문제
  - WHEN) 하위 클래스가 기반 클래스에 대해 가졌던 가정이 기반 클래스를 변경함으로써 깨져버린 경우 발생
  - WHY) 어떤 클래스가 자신을 상속하는 방법에 대해 정확한 규칙(어떤 메소드를 어떻게 오버라이드해야 하는지 등)을 제공하지 않는다면 그 클래스의 클라이언트는 기반 클래스를 변경하는 경우 하위 클래스의 동작이 예기치 않게 바뀔수도 있다.
> 상속을 위한 설계와 문서를 갖추거나, 그럴 수 없다면 상속을 금지하라 (Feat. Effective Java)
>
> 특별히 하위 클래스에서 오버라이드하게 의도된 클래스와 메소드가 아니라면 모두 final로 만들라는 뜻

- 코틀린도 마찬가지로 철학을 따른다.
  - 자바의 클래스와 메소드는 기본적으로 상속에 대해 열려있지만 코틀린의 클래스와 메소드는 기본적으로 final이다.
- 어떤 클래스의 상속을 허용하려면 클래스 앞에 open 변경자를 붙여야 한다. 그와 더불어 오버라이드를 허용하고 싶은 메소드나 프로퍼티의 앞에도 open 변경자를 붙여야 한다.
```kotlin
// 열린 메소드를 포함하는 열린 클래스 정의하기
// 이 클래스는 열려있다. 다른 클래스가 이 클래스를 상속할 수 있다.
open class RichButton : Clickable {
    // 이 함수는 final이다. 하위 클래스가 이 메소드를 오버라이드할 수 없다.
    fun disable() {}
  
    // 이 함수는 열려있다. 하위 클래스에서 이 메소드를 오버라이드해도 된다.
    open fun animate() {}
  
    // 이 함수는 (상위 클래스에서 선언된) 열려있는 메소드를 오버라이드한다.
    // 오버라이드한 메소드는 기본적으로 열려있다.
    override fun click() {}
} 
```
- 기반 클래스나 인터페이스의 멤버를 오버라이드하는 경우 그 메소드는 기본적으로 열려있다.
- 오버라이드하는 메소드의 구현을 하위 클래스에서 오버라이드하지 못하게 금지하려면 오버라이드하는 메소드 앞에 final을 명시해야 한다.
```kotlin
// 오버라이드 금지하기
open class RichButton : Clickable {
    // 여기 있는 "final"은 쓸데 없이 붙은 중복이 아니다.
    // "final"이 없는 "override" 메소드나 프로퍼티는 기본적으로 열려있다.
    final override fun click() {}
}
```
> **열린 클래스와 스마트 캐스트**
>
> 클래스의 기본적인 상속 가능 상태를 final로 함으로써 얻을 수 있는 큰 이익은 다양한 경우에 스마트 캐스트가 가능하다는 점이다. 스마트 캐스트는 타입 검사 뒤에 변경될 수 없는 변수에만 적용 가능하다. 클래스의 프로퍼티의 경우 이는 val이면서 커스텀 접근자가 없는 경우에만 스마트 캐스를 쓸 수 있다는 의미다. 이 요구사항은 또한 프로퍼티가 final이어야만 한다는 뜻이기도 하다. 프로퍼티가 final이 아니라면 그 프로퍼티를 다른 클래스가 상속하면서 커스텀 접근자를 정의함으로써 스마트 캐스트의 요구 사항을 깰 수 있다. 프로퍼티는 기본적으로 final이기 때문에 따로 고민할 필요 없이 대부분의 프로퍼티를 스마트 캐스트에 활용할 수 있다. 이는 코드를 더 이해하기 쉽게 만든다.

- 자바처럼 코틀린에서도 클래스를 abstract로 선언할 수 있다. abstract로 선언한 추상 클래스는 인스턴스화 할 수 없다.
- 추상 클래스에는 구현이 없는 추상 멤버가 있기 때문에 하위 클래스에서 그 추상 멤버를 오버라이드해야만 하는 게 보통이다.
- 추상 멤버는 항상 열려있다. 따라서 추상 멤버 앞에 open 변경자를 명시할 필요가 없다.
```kotlin
// 이 클래스는 추상클래스다. 이 클래스의 인스턴스를 만들 수 없다.
abstract class Animated {
    // 이 함수는 추상함수다. 이 함수에는 구현이 없다. 하위 클래스에서는 이 함수를 반드시 오버라이드해야 한다.
    abstract fun animate()
  
    // 추상 클래스에 속했더라도 비추상 함수는 기본적으로 파이널이지만 원한다면 open으로 오버라이드를 허용할 수 있다.
    open fun stopAnimating() {}
    fun animateTwice() {}
}
```
- 인터페이스 멤버의 경우 final, open, abstract를 사용하지 않는다.
- 인터페이스 멤버는 항상 열려 있으며 final로 변경할 수 없다.
- 인터페이스 멤버에게 본문이 없으면 자동으로 추상 멤버가 되지만, 그렇더라도 따로 멤버 선언 앞에 abstract 키워드를 덧붙일 필요가 없다.
<table>
  <tr>
    <th>변경자</th>
    <th>이 변경자가 붙은 멤버는...</th>
    <th>설명</th>
  </tr>
  <tr>
    <td>final</td>
    <td>오버라이드할 수 없음</td>
    <td>클래스 멤버의 기본 변경자다.</td>
  </tr>
  <tr>
    <td>open</td>
    <td>오버라이드할 수 있음</td>
    <td>반드시 open을 명시해야 오버라이드할 수 있다.</td>
  </tr>
  <tr>
    <td>abstract</td>
    <td>반드시 오버라이드해야 함</td>
    <td>추상 클래스의 멤버에만 이 변경자를 붙일 수 있다. 추상 멤버에는 구현이 있으면 안 된다.</td>
  </tr>
  <tr>
    <td>override</td>
    <td>상위 클래스나 상위 인스턴스의 멤버를 오버라이드하는 중</td>
    <td>오버라이드하는 멤버는 기본적으로 열려있다. 하위 클래스의 오버라이드를 금지하려면 final을 명시해야 한다.</td>
  </tr>
</table>

### 가시성 변경자: 기본적으로 공개
- 가시성 변경자(visibility modifier)는 코드 기반에 있는 선언에 대한 클래스 외부 접근을 제어한다. 어떤 클래스의 구현에 대한 접근을 제한함으로써 그 클래스에 의존하는 외부 코드를 깨지 않고도 클래스 내부 구현을 변경할 수 있다.
- 코틀린의 기본 가시성은 아무 변경자도 없는 경우 모두 공개(public)된다.
- 자바의 기본 가시성인 패키지 전용(package-private)은 코틀린에 없다. 코틀린은 패키지를 네임스페이스를 관리하기 위한 용도로만 사용한다. 그래서 패키지를 가시성 제어에 사용하지 않는다.
- 패키지 전용 가시성에 대한 대안으로 코틀린에서는 internal이라는 새로운 가시성 변경자를 도입했다.
  - internal은 "모듈 내부에서만 볼 수 있음"이라는 뜻이다.
    - 모듈은 한 번에 한꺼번에 컴파일되는 코틀린 파일들을 의미한다. 인텔리J나 이클립스, 메이븐, 그레이들 등의 프로젝트가 모듈이 될 수 있고 앤트 태스크가 한 번 실행될 때 함께 컴파일되는 파일의 집합도 모듈이 될 수 있다.
- 모듈 내부 가시성은 여러분의 모듈의 구현에 대해 진정한 캡슐화를 제공한다는 장점이 있다.
  > 자바에서는 패키지가 같은 클래스를 선언하기만 하면 어떤 프로젝트의 외부에 있는 코드라도 패키지 내부에 있는 패키지 전용 선언에 쉽게 접근할 수 있다. 그래스 모듈의 캡슐화가 쉽게 깨진다.
- 코틀린에서는 최상위 선언에 대해 private 가시성(비공개 가시성)을 허용한다는 점이다. 그런 최상위 선언에는 클래스, 함수, 프로퍼티 등이 포함된다.
- 비공개 가시성인 최상위 선언은 그 선언이 들어있는 파일 내부에서만 사용할 수 있다. 이 또한 하위 시스템의 자세한 구현 사항을 외부에 감추고 싶을 때 유용한 방법이다.
<table>
  <tr>
    <th>변경자</th>
    <th>클래스 멤버</th>
    <th>최상위 선언</th>
  </tr>
  <tr>
    <td>public(기본 가시성임)</td>
    <td>모든 곳에서 볼 수 있다.</td>
    <td>모든 곳에서 볼 수 있다.</td>
  </tr>
  <tr>
    <td>internal</td>
    <td>같은 모듈 안에서만 볼 수 있다.</td>
    <td>같은 모듈 안에서만 볼 수 있다.</td>
  </tr>
  <tr>
    <td>protected</td>
    <td>하위 클래스 안에서만 볼 수 있다.</td>
    <td>(최상위 선언에 적용할 수 없음)</td>
  </tr>
  <tr>
    <td>private</td>
    <td>하위 클래스 안에서만 볼 수 있다.</td>
    <td>같은 파일 안에서만 볼 수 있다.</td>
  </tr>
</table>

```kotlin
internal open class TalkativeButton : Focusable {
    private fun yell() = println("Hey!")
    protected fun whisper() = println("Let's talk")
}

// 오류: "public" 멤버가 자신의 "internal" 수신 타입인 "TalkativeButton"을 노출함
fun TalkativeButton.giveSpeech() {
    // 오류: "yell"에 접근할 수 없음: "yell"은 "TalkativeButton"의 "private" 멤버임
    yell()
    // 오류: "whisper"에 접근할 수 없음: "whisper"는 "TalkativeButton"의 "protected" 멤버임
    whisper()
}
```
> 어떤 클래스의 기반 타입 목록에 들어있는 타입이나 제네릭 클래스의 타입 파라미터에 들어있는 타입의 가시성은 그 클래스 자신의 가시성과 같거나 더 높아야 하고, 메소드의 시그니처에 사용된 모든 타입의 가시성은 그 메소드의 가시성과 같거나 더 높아야 한다는 규칙
>
> 어떤 함수를 호출하거나 어떤 클래스를 확장할 때 필요한 모든 타입에 접근할 수 있게 보장해준다.
- public 함수인 giveSpeech 안에서 그보다 가시성이 더 낮은(이 경우 internal) 타입인 TalkativeButton을 참조하지 못하게 한다.
  - 해결책 : giveSpeech 확장 함수의 가시성을 internal로 바꾸거나, TalkativeButton 클래스의 가시성을 public으로 바꿔야 한다.
- 코틀린에서는 외부 클래스가 내부 클래스나 중첩된 클래스의 private 멤버에 접근할 수 없다.

### 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스
- 클래스 안에 다른 클래스를 선언하면 도우미 클래스를 캡슐화하거나 코드 정의를 그 코드를 사용한 곳 가까이에 두고 싶을 때 유용하다.
- 코틀린의 중첩 클래스는 명시적으로 요청하지 않는 한 바깥쪽 클래스 인스턴스에 대한 접근 권한이 없다.
```kotlin
// 직렬화할 수 있는 상태가 있는 뷰 선언
interface State: Serializable

interface View {
    fun getCurrentState(): State
    fun restoreState(state: State) {}
}
```
```java
// 자바에서 내부 클래스를 사용해 View 구현하기
public class Button implements View {
    @Override
    public State getCurrentState() {
      return now ButtonState();
    }
  
    @Override
    public void restoreState(State state) { /*...*/ }
    public class ButtonState implements State { /*...*/ }
}
```
- java.io.NotSerializableException: Button 오류 발생
  - ButtonState 타입의 state인 변수를 직렬화하는 데 Button을 직렬화할 수 없다는 예외가 발생
- 자바에서 다른 클래스 안에 정의한 클래스는 자동으로 내부 클래스가 된다.
  - 이 예제의 ButtonState 클래스는 바깥쪽 Button 클래스에 대한 참조를 묵시적으로 포함한다.
    - 원인) Button을 직렬화할 수 없으므로 버튼에 대한 참조가 ButtonState의 직렬화를 방해한다.
    - 해결책) ButtonState를 static 클래스로 선언하면 해당 클래스를 둘러싼 바깥쪽 클래스에 대한 묵시적인 참조가 사라진다.
```kotlin
// 중첩 클래스를 사용해 코틀린에서 View 구현하기
class Button : View {
    override fun getCurrentState(): State = ButtonState()
  
    override fun restoreState(state: State) { /*...*/ }
  
    class ButtonState : State { /*...*/ }
}
```
- 코틀린 중첩 클래스에 아무런 변경자가 붙지 않으면 자바 static 중첩 클래스와 같다.
- 이를 내부 클래스로 변경해서 바깥쪽 클래스에 대한 참조를 포함하게 만들고 싶다면 inner 변경자를 붙여야 한다.
<table>
  <tr>
    <th>클래스 B 안에 정의된 클래스 A</th>
    <th>자바에서는</th>
    <th>코틀린에서는</th>
  </tr>
  <tr>
    <td>중첩 클래스(바깥쪽 클래스에 대한 참조를 저장하지 않음)</td>
    <td>static class A</td>
    <td>class A</td>
  </tr>
  <tr>
    <td>내부 클래스(바깥쪽 클래스에 대한 참조를 저장함)</td>
    <td>class A</td>
    <td>inner class A</td>
  </tr>
</table>

- 내부 클래스 Inner 안에서 바깥쪽 클래스 Outer의 참조에 접근하려면 this@Outer라고 써야한다.
```kotlin
class Outer {
    inner class Inner {
        fun getOuterReference(): Outer = this@Outer
    }
}
```
- 클래스 계층을 만들되 그 계층에 속한 클래스의 수를 제한하고 싶은 경우 중첩 클래스를 쓰면 편리하다.

### 봉인된 클래스: 클래스 계층 정의 시 계층 확장 제한
```kotlin
// 인터페이스 구현을 통해 식 표현하기
interface Expr

class Num(val value: Int) : Expr

class Sum(val left: Expr, val right: Expr) : Expr

fun eval(e: Expr): Int =
    when (e) {
        is Num -> e.value
        is Sum -> eval(e.right) + evel(e.left)
        else -> // else 분기가 꼭 있어야 한다.
            throw IllegalArgumentException("Unknown expression")
    }
```
- 디폴트 분기(else)가 있으면 클래스 계층에 새로운 하위 클래스를 추가하더라도 컴파일러가 when이 모든 경우를 처리하는지 제대로 검사할 수 없다. 혹 실수로 새로운 클래스 처리를 잊어버렸더라도 디폴트 분기가 선택되기 때문에 심각한 버그가 발생할 수 있다.
- 해결책) sealed 클래스
  - 상위 클래스에 sealed 변경자를 붙이면 그 상위 클래스를 상속한 하위 클래스 정의를 제한할 수 있다.
  - sealed 클래스의 하위 클래스를 정의할 때는 반드시 상위 클래스 안에 중첩시켜야 한다.
```kotlin
// 기반 클래스를 sealed로 봉인한다.
sealed class Expr {
    // 기반 클래스의 모든 하위 클래스를 중첩 클래스로 나열한다.
    class Num(val value: Int) : Expr()
    class Sum(val left: Expr, val right: Expr) : Expr()
}

// "When" 식이 모든 하위 클래스를 검사하므로 별도의 "else" 분기가 없어도 된다.
fun eval(e: Expr): Int =
    when (e) {
        is Num -> e.value
        is Sum -> eval(e.right) + evel(e.left)
    }  
```
- sealed로 표기된 클래스는 자동으로 open임을 기억하라.
- 내부적으로 Expr 클래스는 private 생성자를 가진다. 그 생성자는 클래스 내부에서만 호출할 수 있다.
- sealed 인터페이스를 정의할 수는 없다.

## 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언
- 코틀린은 주 생성자(보통 주 생성자는 클래스를 초기화할 때 주로 사용하는 간략한 생성자로, 클래스 본문 밖에서 정의한다)와 부 생성자(클래스 본문 안에서 정의한다)를 구분한다. 또한 초기화 블록을 통해 초기화 로직을 추가할 수 있다.
### 클래스 초기화: 주 생성자와 초기화 블록
```kotlin
class User(val nickname: String)
```
- 주 생성자는 클래스 이름 뒤에 오는 괄호로 둘러싸인 코드이다.
- 생성자 파라미터를 지정하고 그 생성자 파라미터에 의해 초기화되는 프로퍼티를 정의하는 두 가지 목적에 쓰인다.
```kotlin
class User constructor(_nickname: String) {
    val nickname: String
    
    init {
        nickname = _nickname
    }
} 
```
- constructor 키워드는 주 생성자나 부 생성자 정의를 시작할 때 정의한다. init 키워드는 초기화 블록을 시작한다.
- 초기화 블록에는 클래스의 객체가 만들어질 때(인스턴스화될 때) 실행될 초기화 코드가 들어간다.
- 초기화 블록은 주 생성자와 함께 사용된다.
- 주 생성자는 제한적이기 때문에 별도의 코드를 포함할 수 없으므로 초기화 블록이 필요하다. 필요하다면 클래스 안에 여러 초기화 블록을 선언할 수 있다.
- 생성자 파라미터 _nickname에서 맨 앞의 밑줄(\_)은 프로퍼티와 생성자 파라미터를 구분해준다.
  - 다른 방법으로 this.nickname = nickname 방식처럼 this를 써서 모호성을 없애도 된다.


- 주 생성자 앞에 별다른 애노테이션이나 가시성 변경자가 없다면 constructor를 생략해도 된다.
```kotlin
class User(_nickname: String) { // 파라미터가 하나뿐인 주 생성자
    val nickname = _nickname // 프로퍼티를 주 생성자의 파라미터로 초기화한다.
} 
```
> 프로퍼티를 초기화하는 식이나 초기화 블록 안에서만 주 생성자의 파라미터를 참조할 수 있다는 점에 유의하라.

- 주 생성자 파라미터 이름 앞에 val을 추가하는 방식으로 프로퍼티 정의와 초기화를 간략히 쓸 수 있다.
```kotlin
class User(val nickname: String) // "val"은 이 파라미터에 상응하는 프로퍼티가 생성된다는 뜻이다. 
```
- 함수 파라미터와 마찬가지로 생성자 파라미터에도 디폴트 값을 정의할 수 있다.
```kotlin
class User(val nickname: String, val isSubscribed: Boolean = true) // 생성자 파라미터에 대한 디폴트 값을 제공한다. 
```
- 클래스의 인스턴스를 만들려면 new 키워드 없이 생성자를 직접 호출하면 된다.
```kotlin
val hynn = User("현석") // isSubscribed 파라미터에는 디폴트 값이 쓰인다.
println(hyun.isSubscribed) // true

val gye = User("계영", false) // 모든 인자를 파라미터 선언 순서대로 지정할 수도 잇다.
println(gye.isSubscribed) // false

val hey = User("혜원", isSubscribed = false) // 생성자 인자 중 일부에 대해 이름을 지정할 수도 있다.
println(hey.isSubscribed) // false
```
- 클래스에 기반 클래스가 있다면 주 생성자에서 기반 클래스의 생성자를 호출해야 할 필요가 있다. 기반 클래스를 초기화하려면 기반 클래스 이름 뒤에 괄호를 치고 생성자 인자를 넘긴다.
```kotlin
open class User(val nickname: String) { ... }

class TwitterUser(nickname: String) : User(nickname) { ... }
```
- 클래스를 정의할 때 별도로 생성자를 정의하지 않으면 컴파일러가 자동으로 아무 일도 하지 않는 인자가 없는 디폴트 생성자를 만들어준다.
```kotlin
open class Button // 인자가 없는 디폴트 생성자가 만들어진다.
```
- Button의 생성자는 아무 인자도 받지 않지만, Button 클래스를 상속한 하위 클래스는 반드시 Button 클래스의 생성자를 호출해야 한다.
```kotlin
class RadioButton: Button() 
```
- 이 규칙으로 인해 기반 클래스의 이름 뒤에는 꼭 빈 괄호가 들어간다(물론 생성자 인자가 있다면 괄호 안에 인자가 들어간다).
- 반면 인터페이스는 생성자가 없기 때문에 어떤 클래스가 인터페이스를 구현하는 경우 그 클래스의 상위 클래스 목록에 있는 인터페이스 이름 뒤에는 아무 괄호도 없다.


- 어떤 클래스를 클래스 외부에서 인스턴스화하지 못하게 막고 싶다면 모든 생성자를 private으로 만들면 된다.
```kotlin
class Secretive private constructor() {} // 이 클래스의 (유일한) 주 생성자는 비공개다. 
```
- Secretive 클래스 안에는 주 생성자밖에 없고 그 주 생성자는 비공개이므로 외부에서는 Secretive를 인스턴스화할 수 없다.
> 유틸리티 함수를 담아두는 역할만을 하는 클래스는 인스턴스화할 필요가 없고, 싱글턴인 클래스는 미리 정한 팩토리 메소드 등의 생성 방법을 통해서만 객체를 생성해야 한다. 코틀린에서는 이런 경우를 언어에서 기본 지원한다. 정적 유틸리티 함수 대신 최상위 함수를 사용할 수 있고, 싱글톤을 사용하고 싶으면 객체를 선언하면 된다.

### 부 생성자: 상위 클래스를 다른 방식으로 초기화
> 인자에 대한 디폴트 값을 제공하기 위해 부 생성자를 여럿 만들지 말라. 대신 파라미터의 디폴트 값을 생성자 시그니처에 직접 명시하라.
```kotlin
open class View {
    constructor(ctx: Context) { // 부 생성자
        // 코드
    }
    constructor(ctx: Context, attr: AttributeSet) { // 부 생성자
        // 코드
    }
}
```
- 이 클래스는 주 생성자를 선언하지 않고(클래스 헤더에 있는 클래스 이름 뒤에 괄호가 없다), 부 생성자만 2가지 선언한다.
- 부 생성자는 constructor 키워드로 시작한다.
```kotlin
// 상위 클래스의 생서자를 호출한다.
class MyButton : View {
    constructor(ctx: Context) : super(ctx) {
        // ...
    }
    constructor(ctx: Contextm attr: AttributeSet) : super(ctx, attr) {
        // ...
    }
} 
```
- 두 부 생성자는 super() 키워드를 통해 자신에 대응하는 상위 클래스 생성자를 호출한다.
- MyButton의 생성자가 상위 클래스(View) 생성자에게 객체 생성을 위임한다.


- 생성자에서 this()를 통해 클래스 자신의 다른 생성자를 호출할 수 있다.
```kotlin
class MyButton: View {
    constructor(ctx: Context): this(ctx, MY_STYLE) { // 이 클래스의 다른 생성자에게 위임한다.
        // ...
    }
    constructor(ctx: Context, attr: AttributeSet): super(ctx, attr) {
        // ...
    }
} 
```
- MyButton 클래스의 생성자 중 하나가 파라미터의 디폴트 값을 넘겨서 같은 클래스의 다른 생성자(this를 사용해 참조함)에게 생성을 위임한다. 두번째 생성자는 여전히 super()를 호출한다.
- 클래스에 주 생성자가 없다면 모든 부 생성자는 반드시 상위 클래스를 초기화하거나 다른 생성자에게 생성을 위임해야 한다.
  - 위 예시처럼 각 부 생성자에게 객체 생성을 위임하는 화살표를 따라가면 그 끝에는 상위 클래스 생성자를 호출하는 화살표가 있어야 한다는 뜻이다.

### 인터페이스에 선언된 프로퍼티 구현
- 코틀린에서는 인터페이스에 추상 프로퍼티 선언을 넣을 수 있다.
```kotlin
interface Uesr {
    val nickname: String
}
```
- User 인터페이스를 구현하는 클래스가 nickname의 값을 얻을 수 있는 방법을 제공해야 한다는 뜻이다.
- 인터페이스에 있는 프로퍼티 선언에는 뒷받침하는 필드나 게터 등의 정보가 들어있지 않다.
  - 사실 인터페이스는 아무 상태도 포함할 수 없으므로 상태를 저장할 필요가 있다면 인터페이스를 구현한 하위 클래스에서 상태 저장을 위한 프로퍼티 등을 만들어야 한다.
```kotlin
// 인터페이스의 프로퍼티 구현하기
class PrivateUser(override val nickname: String) : User // 주 생서자에 있는 프로퍼티

class SubscribingUser(val email: String) : User {
    override val nickname: String
      get() = email.substringBefore('@') // 커스텀 게터
}

class FacebookUser(val accountId: Int) : User {
    override val nickname = getFacebookName(accountId) // 프로퍼티 초기화 식
}

println(PrivateUser("test@kotlinlang.org").nickname) // test@kotlinlang.org
println(SubscribingUser("test@kotlinlang.org").nickname) // test
```
- PrivateUser는 주 생성자 안에 프로퍼티를 직접 선언하는 간결한 구문을 사용한다. 이 프로퍼티는 User의 추상 프로퍼티를 구현하고 있으므로 override를 표시해야 한다.
- SubscribingUser는 커스텀 게터로 nickname 프로퍼티를 설정한다. 이 프로퍼티는 뒷받침하는 필드에 값을 저장하지 않고 매번 이메일 주소에서 별명을 계산해 반환한다.
- FacebookUser에서는 초기화 식으로 nickname 값을 초기화한다. 이때 페이스북 사용자 ID를 받아서 그 사용자의 이름을 반환해주는 getFacebookName 함수를 호출해서 nickname을 초기화한다.

> SubscribingUser와 FacebookUser의 nickname 구현 차이에 주의하라.

- SubscribingUser의 nickname은 매번 호출될 때마다 substringBefore를 호출해 계산하는 커스텀 게터를 활용한다.
- FacebookUser의 nickname은 객체 초기화 시 계산한 데이터를 뒷받침하는 필드에 저장햇다가 불러오는 방식을 활용한다.


- 인터페이스에는 추상 프로퍼티뿐 아니라 게터와 세터가 있는 프로퍼티를 선언할 수도 잇다. 물론 그런 게터와 세터는 뒷받침하는 필드를 참조할 수 없다(뒷받침하는 필드가 있다면 인터페이스에 상태를 추가하는 셈인데 인터페이스는 상태를 저장할 수 없다).
```kotlin
interface User {
    val email: String
    
    // 프로퍼티에 뒷받침하는 필드가 없다. 대신 매번 결과를 계산해 돌려준다.
    val nickname: String
      get() = email.substringBefore('@') 
} 
```
- 하위 클래스는 추상 프로퍼티인 email을 반드시 오버라이드해야 한다. 반면 nickname은 오버라이드하지 않고 상속할 수 있다.

### 게터와 세터에서 뒷받침하는 필드에 접근
- 값을 저장하는 동시에 로직을 실행할 수 있게 하기 위해서는 접근자 안에서 프로퍼티를 뒷받침하는 필드에 접근할 수 있어야 한다.
```kotlin
// 세터에서 뒷받침하는 필드 접근하기
class User(val name: String) {
    var address: String = "unspecified"
      set(value: String) {
          println("""
            Address was changed for $name:
            "$field" -> "$value".""".trimIndent()) // 뒷받침하는 필드 값 읽기 
          field = value
      }
}

val user = User("Alice")
user.address = "Elsenheimerstrasse 47, 80687 Muenchen"
```
- 코틀린에서 프로퍼티의 값을 바꿀 때는 user.address = "new value"처럼 필드 설정 구문을 사용한다.
  - 이 구문은 내부적으로 address의 세터를 호출한다.
- 접근자의 본문에서는 field라는 특별한 식별자를 통해 뒷받침하는 필드에 접근할 수 있다.
  - 게터에서는 field 값을 읽을 수만 있고, 세터에서는 field 값을 읽거나 쓸 수 있다.

> 뒷받침하는 필드가 있는 프로퍼티와 그런 필드가 없는 프로퍼티에 어떤 차이가 있나?

- 클래스의 프로퍼티를 사용하는 쪽에서 프로퍼티를 읽는 방법이나 쓰는 방법은 뒷받침하는 필드의 유무와는 관계가 없다.
- 컴파일러는 디폴트 접근자 구현을 사용하건 직접 게터나 세터를 정의하건 관계없이 게터나 세터에서 field를 사용하는 프로퍼티에 대해 뒷받침하는 필드를 생성해준다.
- 다만 field를 사용하지 않는 커스텀 접근자 구현을 정의한다면 뒷받침하는 필드는 존재하지 않는다(프로퍼티가 val인 경우에는 게터에 field가 없으면 되지만, var인 경우에는 게터나 세터 모두에 field가 없어야 한다).

### 접근자의 가시성 변경
- 접근자의 가시성은 기본적으로는 프로퍼티의 가시성과 같다. 하지만 원한다면 get이나 set 앞에 가시성 변경자를 추가해서 접근자의 가시성을 변경할 수 있다.
```kotlin
// 비공개 세터가 있는 프로퍼티 선언하기
class LengthCounter {
    var counter: Int = 0
      private set // 이 클래스 밖에서 이 프로퍼티의 값을 바꿀 수 없다.
    
    fun addWord(word: String) {
        counter += word.length
    }
}

val lengthCounter = LengthCounter()
lengthCounter.addWord("Hi!")
println(lengthCounter.counter) // 3
```
- 이 클래스는 자신에게 추가된 모든 단어의 길을 합산한다. 전체 길이를 저장하는 프로퍼티는 클라이언트에게 제공하는 API의 일부분이므로 public으로 외부에 공개한다.
- 단어 길이의 합을 마음대로 바꾸지 못하게 기본 가시성을 가진 게터를 컴파일러가 생성하게 내버려 두는 대신 세터의 가시성을 private으로 지정한다.

## 컴파일러가 생성한 메소드: 데이터 클래스와 클래스 위임
### 모든 클래스가 정의해야 하는 메소드
```kotlin
class Client(val name: String, val postalCode: Int)
```
#### 문자열 표현: toString()
- 자바처럼 코틀리느이 모든 클래스도 인스턴스의 문자열 표현을 얻을 방법을 제공한다. 주로 디버깅과 로깅 시 이 메소드를 사용한다.

```kotlin
class Client(val name: String, val postalCode: Int) {
    override fun toString() = "Client(name=$name, postalCode=$postalCode")
}

val client1 = Client("오현석", 4122)
println(client1) // Client(name=오현석, postalCode=4122)
```
#### 객체의 동등성: equals()
- Client 클래스를 사용하는 모든 계산은 클래스 밖에서 이뤄진다. Client는 단지 데이터를 저장할 뿐이며, 그에 따라 구조도 단순하고 내부 정보를 투명하게 외부에 노출하게 설계됐다.
```kotlin
val client1 = Client("오현석", 4122)
val client2 = Client("오현석", 4122)

// 코틀린에서 == 연산자는 참조 동일성을 검사하지 않고 객체의 동등성을 검사한다.
// 따라서 == 연산은 equals를 호출하는 식으로 컴파일된다.
println(client1 == client2) // false
```

```kotlin
// Client에 equals() 구현하기
class Client(val name: String, val postalCode: Int) {
    override fun equals(other: Any?): Boolean {
        if (other == null || other !is Client)
            return false
        return name == other.name &&
              postalCode == other.postalCode
    }

    override fun toString() = "Client(name=$name, postalCode=$postalCode)"
}
```
#### 해시 컨테이너: hashCode()
```kotlin
val processed = hashSetOf(Client("오현석", 4122))
println(processed.contains(Client("오현석", 4122))) // false
```
- hashSet은 원소를 비교할 때 비용을 줄이기 위해 먼저 객체의 해시 코드를 비교하고 해시 코드가 같은 경우에만 실제 값을 비교한다.
- 원소 객체들이 해시 코드에 대한 규칙을 지키지 않는 경우 HashSet은 제대로 작동할 수 없다.
  - 해시 코드 규칙 : equals()가 true를 반환하는 두 객체는 반드시 같은 hashCode()를 반환해야 한다.
```kotlin
// Client에 hashCode 구현하기
class Client(val name: String, val postalCode: Int) {
    ...
    override fun hashCode(): Int = name.hashCode() * 31 + postalCode
}
```
- 코틀린 컴파일러는 이 모든 메소드를 자동으로 생성해줄 수 있다.

### 데이터 클래스: 모든 클래스가 정의해야 하는 메소드 자동 생성
- data라는 변경자를 클래스 앞에 붙이면 필요한 메소드(toString, equals, hashCode)를 컴파일러가 자동으로 만들어준다.
- data 변경자가 붙은 클래스를 데이터 클래스라고 부른다.
```kotlin
// Client를 데이터 클래스로 선언하기
data class Client(val name: String, val postalCode: Int)
```
> - 인스턴스 간 비교를 위한 equals
> - HashMap과 같은 해시 기반 컨테이너에서 키로 사용할 수 있는 hashCode
> - 클래스의 각 필드를 선언 순서대로 표시하는 문자열 표현을 만들어주는 toString

#### 데이터 클래스와 불변성: copy() 메소드
> 데이터 클래스의 모든 프로퍼티를 읽기 전용으로 만들어서 데이터 클래스를 불변 클래스로 만들라고 권장한다.

- WHY) HashMap 등의 컨테이너에 데이터 클래스 객체를 담는 경우엔 불변성이 필수적이다.
  - 데이터 클래스 객체를 키로 하는 값을 컨테이너에 담은 다음에 키로 쓰인 데이터 객체의 프로퍼티를 변경하면 컨테이너 상태가 잘못될 수 있다.
  - 게다가 불변 객체를 사용하면 프로그램에 대해 훨씬 쉽게 추론할 수 있다.
  - 불변 객체를 주로 사용하는 프로그램에서는 스레드가 사용 중인 데이터를 다른 스레드가 변경할 수 없으므로 스레드를 동기화해야 할 필요가 줄어든다. (다중스레드 프로그램의 경우)

- 데이터 클래스 인스턴스를 불변 객체로 더 쉽게 활용할 수 있게 코틀린 컴파일러는 한 가지 편의 메소드를 제공한다.
  - 객체를 복사하면서 일부 프로퍼티를 바꿀 수 있게 해주는 copy 메소드다.
    - 복사본은 원본과 다른 생명주기를 가지며, 복사를 하면서 일부 프로퍼티 값을 바꾸거나 복사본을 제거해도 프로그램에서 원본을 참조하는 다른 부분에 전혀 영향을 끼치지 않는다.
  ```kotlin
  class Client(val name: String, val postalCode: Int) {
      ...
      fun copy(nam: String = this.name,
              postalCode: Int = this.postalCode) =
         Client(name, postalCode)
  }
  
  val lee = Client("이계영", 4122)
  println(lee.copy(postalCode = 4000)) // Client(name=이계영, postalCode=4000)
  ```

### 클래스 위임: by 키워드 사용
- 대규모 객체지향 시스템을 설계할 때 시스템을 취약하게 만드는 문제는 보통 구현 상속에 의해 발생한다.
  - 하위 클래스가 상위 클래스의 메소드 중 일부를 오버라이드하면 하위 클래스는 상위 클래스의 세부 구현 사항에 의존하게 된다.
    - 시스템이 변함에 따라 상위 클래스의 구현이 바뀌거나 상위 클래스에 새로운 메소드가 추가된다. 그 과정에서 하위 클래스가 상위 클래스에 대해 갖고 있던 가정이 깨져서 코드가 정상적으로 작동하지 못하는 경우가 생길 수 있다.


- 모든 클래스를 기본적으로 final로 취급하면 상속을 염두에 두고 open 변경자로 열어둔 클래스만 확장할 수 있다. 열린 상위 클래스의 소스코드를 변경할 때는 open 변경자를 보고 해당 클래스를 다른 클래스가 상속하리라 예상할 수 있으므로, 변경 시 하위 클래스를 깨지 않기 위해 좀 더 조심할 수 있다.
- BUT) 상속을 허용하지 않는 클래스에 새로운 동작을 추가해야 할 때가 있다.
- HOW) 데코레이터(Decorator) 패턴
  - 핵심) 상속을 허용하지 않는 클래스(기존 클래스) 대신 사용할 수 있는 새로운 클래스(데코레이터)를 만들되 기존 클래스와 같은 인터페이스를 데코레이터가 제공하게 만들고, 기존 클래스를 데코레이터 내부에 필드로 유지하지는 것이다.
  - 새로 정의해야 하는 기능은 데코레이터의 메소드에 새로 정의하고(물론 이때 기존 클래스의 메소드나 필드를 활용할 수 있다) 기존 기능이 그대로 필요한 부분은 데코레이터의 메소드가 기존 클래스의 메소드에게 요청을 전달한다.
  - 단점) 준비 코드가 상당히 많이 필요하다. 그래서 IDE는 컴파일러를 통해 자동 생성해준다.
```kotlin
class DelegatingCollection<T> : Collection<T> {
    private val innerList = arrayListOf<T>()
  
    override val size: Int get() = innerList.size()
    override fun isEmpty(): Boolean = innerList.isEmpty()
    override fun contains(element: T): Boolean = innerList.contains(element)
    override fun iterator(): Iterator<T> = innerList.iterator()
    override fun containsAll(elements: Collection<T>): Boolean =
        innerList.containsAll(elements)
}
```
- 이런 위임을 언어가 제공하는 일급 시민 기능으로 지원한다는 점이 코틀린의 장점이다.
- 인터페이스를 구현할 때 by 키워드를 통해 그 인터페이스에 대한 구현을 다른 객체에 위임 중이라는 사실을 명시할 수 있다.
```kotlin
class DelegatingCollection<T>(
    innerList: Collection<T> = ArrayList<T>()
) : Collection<T> by innerList { }
```
- 메소드 중 일부의 동작을 변경하고 싶은 경우 메소드를 오버라이드하면 컴파일러가 생성한 메소드 대신 오버라이드한 메소드가 쓰인다.
```kotlin
class CountingSet<T>(
    val innerSet: MutableCollection<T> = HashSet<T>()
) : MutableCollection<T> by innerSet {
    var objectsAdded = 0
  
    override fun add(element: T): Boolean {
        objectsAdded++
        return innerSet.add(element)
    }
  
    override fun addAll(c: Collection<T>): Boolean {
        objectsAdded += c.size
        return innerSet.addAll(c)
    }
}

val cset = CountingSet<Int>()
cset.addAll(listOf(1, 1, 2))
println("${cset.objectsAdded} objects were added, ${cset.size} remain") // 3 objects were added, 2 remain
```
- add와 addAll을 오버라이드해서 카운터를 증가시키고, MutableCollection 인터페이스의 나머지 메소드는 내부 컨테이너(innerSet)에게 위임한다.
- 핵심) CountingSet에 MutableCollection의 구현 방식에 대한 의존관계가 생기지 않는다는 점이 중요하다.

## object 키워드: 클래스 선언과 인스턴스 생성
- 코틀린에서는 object 키워드를 다양한 상황에서 사용하지만 모든 경우 클래스를 정의하면서 동시에 인스턴스(객체)를 생성한다는 공통점이 잇다.
> - 객체 선언은 싱글턴을 정의하는 방법 중 하나다.
> - 동반 객체(companion object)는 인스턴스 메소드는 아니지만 어떤 클래스와 관련 있는 메소드와 팩토리 메소드를 담을 때 쓰인다. 동반 객체 메소드에 접근할 떄는 동반 객체가 포함된 클래스의 이름을 사용할 수 있다.
> - 객체 식은 자바의 무명 내부 클래스 대신 쓰인다.

### 객체 선언: 싱글턴을 쉽게 만들기
- 객체지향 시스템을 설계하다 보면 인스턴스가 하나만 필요한 클래스가 유용한 경우가 많다.
- 자바에서는 보통 클래스의 생성자를 private으로 제한하고 정적인 필드에 그 클래스의 유일한 객체를 저장하는 실글턴을 통해 이를 구현한다.
- 코틀린은 객체 선언 기능을 통해 싱글턴을 언어에서 기본 지원한다. 객체 선언은 클래스 선언과 그 클래스에 속한 단일 인스턴스의 선언을 합친 선언이다.
```kotlin
object Payroll {
    val allEmployees = arrayListOf<Person>()
  
    fun calculateSalary() {
        for (person in allEmployees) {
            ...
        }
    }
} 
```
- 객체 선언은 object 키워드로 시작한다. 객체 선언은 클래스를 정의하고 그 클래스의 인스턴스를 만들어서 변수에 저장하는 모든 작업을 단 한 문장으로 처리한다.
- 클래스와 마찬가지로 객체 선언 안에도 프로퍼티, 메소드, 초기화 블록 등이 들어갈 수 있다. 하지만 생성자는(주 생성자와 부 생성자 모두) 객체 선언에 쓸 수 없다.
- 일반 클래스 인스턴스와 달리 싱글턴 객체는 객체 선언문이 있는 위치에서 생성자 호출 없이 즉시 만들어진다. 따라서 객체 선언에는 생성자 정의가 필요 없다.
- 변수와 마찬가지로 객체 선언에 사용한 이름 뒤에 마침표(.)를 붙이면 객체에 속한 메소드나 프로퍼티에 접근할 수 있다.
```kotlin
Payroll.allEmployees.add(Person(...))
Payroll.calculateSalary()
```
- 객체 선언도 클래스나 인터페이스를 상속할 수 있다. 프레임워크를 사용하기 위해 특정 인터페이스를 구현해야 하는데, 그 구현 내부에 다른 상태가 필요하지 않은 경우에 이런 기능이 유용하다.
  - 어떤 클래스에 속한 객체를 비교할 때 사용하는 Comparator는 보통 클래스마다 단 하나씩만 있으면 된다. 따라서 Comparator 인스턴스를 만드는 방법으로는 객체 선언이 가장 좋은 방법이다.
```kotlin
object CaseInsensitiveFileComparator : Comparator<File> {
    override fun compare(file1: File?, file2: File?): Int {
        return file1.path.compareTo(file2.path, ignoreCase = true)
    }
}

println(CaseInsensitiveFileComparator.compare(File("/User"), File("/user"))) // 0
```
- 일반 객체(클래스 인스턴스)를 사용할 수 있는 곳에서는 항상 싱글턴 객체를 사용할 수 있다.
```kotlin
val files = listOf(File("/Z"), File("/a"))
println(files.sortedWith(CaseInsensitiveFileComparator)) // [/a, /Z]
```
> 대규모 소프트웨어 시스템에서는 객체 생성을 제어할 방법이 없고 생성자 파라미터를 지정할 수 없어 객체 선언이 항상 적합하지는 않다. 단위 테스트를 하거나 소프트웨어 시스템의 설정이 달라질 때 객체를 대체하거나 객체의 의존관계를 바꿀 수 없다. 따라서 의존관계 주입 프레임워크와 코틀린 클래스를 함께 사용해야 한다.
- 클래스 안에서 객체를 선언할 수도 있다. 그런 객체도 인스턴스는 단 하나뿐이다(바깥 클래스의 인스턴스마다 중첩 객체 선언에 해당하는 인스턴스가 하나씩 따로 생기는 것이 아니다).
- 어떤 클래스의 인스턴스를 비교하는 Comparator를 클래스 내부에 정의하는 게 더 바람직하다.
```kotlin
// 중첩 객체를 사용해 Comparator 구현하기
data class Person(val name: String) {
    object NameComparator : Comparator<Person> {
        override fun compare(p1: Person?, p2: Person?): Int =
            p1.name.compareTo(p2.name)
    }
}

val persons = listOf(Person("Bob"), Person("Alice"))
println(persons.sortedWith(Person.NameComparator)) // [Person(name=Alice), Person(name=Bob)]
```

### 동반 객체: 팩토리 메소드와 정적 멤버가 들어갈 장소
- 코틀린 클래스 안에는 정적인 멤버가 없다. 코틀린 언어는 자바 static 키워드를 지원하지 않는다.
- 그 대신 코틀린에서는 패키지 수준의 최상위 함수(자바의 정적 메소드 역할을 거의 대신 할 수 있다)와 객체 선언(자바의 정적 메소드 역할 줄 코틀린 최상위 함수가 대신할 수 없는 역할이나 정적 필드를 대신할 수 있다)을 활용한다.
- 대부분의 경우 최상위 함수를 활용하는 편을 더 권장한다.


- 최상위 함수는 private으로 표시된 클래스 비공개 멤버에 접근할 수 없다. 그래서 클래스의 인스턴스와 관계없이 호출해야 하지만, 클래스 내부 정보에 접근해야 하는 함수가 필요할 때는 클래스에 중첩된 객체 선언의 멤버 함수로 정의해야 한다. 그런 함수의 대표적인 예로 팩토리 메소드를 들 수 있다.
- 클래스 안에 정의된 객체 중 하나에 companion이라는 특별한 표시를 붙이면 그 클래스의 동반 객체를 만들 수 있다.
- 동반 객체의 프로퍼티나 메소드에 접근하려면 그 동반 객체가 정의된 클래스의 이름을 사용한다. 이때 객체의 이름을 따로 지정할 필요가 없다.
- 그 결과 동반 객체의 멤버를 사용하는 구문은 자바의 정적 메소드 호출이나 정적 필드 사용 구문과 같아진다.
```kotlin
class A {
    companion object {
        fun bar() {
            println("Companion object called")
        }
    }
}

A.bar() // Companion object called
```
- 동반 객체가 private 생성자를 호출하기 좋은 위치다. 동반 객체는 자신을 둘러싼 클래스의 모든 private 멤버에 접근할 수 있다. 따라서 동반 객체는 바깥쪽 클래스의 private 생성자도 호출할 수 있다. 따라서 동반 객체는 팩토리 패턴을 구현하기 가장 적합한 위치다.
```kotlin
// 부 생성자가 여럿 있는 클래스 정의하기
class User {
    val nickname: String
    
    constructor(email: String) {
        nickname = email.substringBefore('@')
    }
  
    constructor(facebookAccountId: Int) {
        nickname = getFacebookName(facebookAccountId)
    }
}
```
- 이런 로직을 표현하는 더 유용한 방법으로 클래스의 인스턴스를 생성하는 팩토리 메소드가 있다.
- 생성자를 통해 User 인스턴스를 만들 수 없고 팩토리 메소드를 통해야만 한다.
```kotlin
class User private constructor(val nickname: String) { // 주 생성자를 비공개로 만든다.
    companion object {
        fun newSubscribingUser(email: String) =
            User(email.substringBefore('@'))
        
        fun newFacebookUser(accountId: Int) =
            User(getFacebookName(accountId))
    }
}

val subscribingUser = User.newSubscribingUser("bob@gmail.com")
val facebookUser = User.newFacebookUser(4)
println(subscribingUser.nickname) // bob
```
- 팩토리 메소드는 그 팩토리 메소드가 선언된 클래스의 하위 클래스 객체를 반환할 수 있다. SubscribingUser와 Facebookuser 클래스가 따로 존재한다면 그때그때 필요에 따라 적당한 클래스의 객체를 반환할 수 있다.
> 클래스를 확장해야만 하는 경우에는 동반 객체 멤버를 하위 클래스에서 오버라이드할 수 없으므로 여러 생성자를 사용하는 편이 더 나은 해법이다.

### 동반 객체를 일반 객체처럼 사용
- 동반 객체는 클래스 안에 정의된 일반 객체다. 따라서 동반 객체에 이름을 붙이거나, 동반 객체가 인터페이스를 상속하거나, 동반 객체 안에 확장 함수와 프로퍼티를 정의할 수 잇다.
```kotlin
// 동반 객체에 이름 붙이기
class Person(val name: String) {
    companion object Loader { // 동반 객체에 이름을 붙인다.
        fun fromJSON(jsonText: String): Person = ...
    }
}

person = Person.Loader.fromJSON("{name: 'Dmitry'}")
person.name // Dmitry

person2 = Person.Loader.fromJSON("{name: 'Brent'}")
person2.name // Brent
```
#### 동반 객체에서 인터페이스 구현
```kotlin
interface JSONFactory<T> {
    fun fromJSON(jsonText: String): T
}

class Person(val name: String) {
    companion object : JSONFactory<Person> {
        override fun fromJSON(jsonText: String): Person = ... // 동반 객체가 인터페이스를 구현한다.
    }
}
```
- 이제 JSON으로부터 각 원소를 다시 만들어내는 추상 팩토리가 있다면 Person 객체를 그 팩토리에게 넘길 수 있다.
```kotlin
fun loadFromJSON<T>(factory: JSONFactory<T>): T {
    ...
}

loadFromJSON(Person) // 동반 객체의 인스턴스를 함수에 넘긴다.
```
- 동반 객체가 구현한 JSONFactory의 인스턴스를 넘길 때 Person 클래스의 이름을 사용했다는 점에 유의하라.

#### 동반 객체 확장
// 동반 객체에 대한 확장 함수 정의하기
```kotlin
class Person(val firstName: String val lastName: String) {
    companion object { } // 비어잇는 동반 객체를 선언한다.
}

fun Person.Companion.fromJSON(json: String) : Person { // 확장 함수를 선언한다.
    ...
}

val p = Person.fromJSON(json)
```
- 마치 동반 객체 안에서 fromJSON 함수를 정의한 것처럼 fromJSON을 호출할 수 있다. 하지만 실제로 fromJSON은 클래스 밖에서 정의한 확장 함수다. 다른 보통 확장 함수처럼 fromJSON도 클래스 멤버 함수처럼 보이지만, 실제로는 멤버 함수가 아니다.
> 동반 객체에 대한 확장 함수를 작성할 수 있으려면 원래 클래스에 동반 객체를 꼭 선언해야 한다는 점에 주의하라. 설령 빈 객체라도 동반 객체가 꼭 있어야 한다.

### 객체 식: 무명 내부 클래스를 다른 방식으로 작성
- 무명 객체는 자바의 무명 내부 클래스를 대신한다.
```kotlin
// 무명 객체로 이벤트 리스너 구현하기
window.addMouseListener(
    object : MouseAdapter() { // MouseAdapter를 확장하는 무명 객체를 선언한다.
        // MouseAdapter의 메소드를 오버라이드한다.
        override fun mouseClicked(e: MouseEvent) {
            // ...
        }
      
        override fun mouseEntered(e: MouseEvent) {
            // ...
        }
    }
)
```
- 객체 식은 클래스를 정의하고 그 클래스에 속한 인스턴스를 생성하지만, 그 클래스나 인스턴스에 이름을 붙이지는 않는다. 
- 이런 경우 보통 함수를 호출하면서 인자로 무명 객체를 넘기기 때문에 클래스와 인스턴스 모두 이름이 필요하지 않다. 하지만 객체에 이름을 붙여야 한다면 변수에 무명 객체를 대입하면 된다.
```kotlin
val listener = object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) { ... }
    override fun mouseEntered(e: MouseEvent) { ... }
}
```
- 한 인터페이스만 구현하거나 한 클래스만 확장할 수 잇는 자바의 무명 내부 클래스와 달리 코틀린 무명 클래스는 여러 인터페이스를 구현하거나 클래스를 확장하면서 인터페이스를 구현할 수 있다.

> 객체 선언과 달리 무명 객체는 싱글턴이 아니다. 객체 식이 쓰일 때마다 새로운 인스턴스가 생성된다.

- 자바의 무명 클래스와 같이 객체 식 안의 코드는 그 식이 포함된 함수의 변수에 접근할 수 있다. 하지만 자바와 달리 final이 아닌 변수도 객체 식 안에서 사용할 수 있다. 따라서 객체 식 안에서 그 변수의 값을 변경할 수 있다.
```kotlin
fun countClicks(window: Window) {
    var clickCount = 0
  
    window.addMouseListener(object: MouseAdapter() {
        override fun mouseClicked(e: MouseEvent) {
            clickCount++
        }
    })
} 
```
> 객체 식은 무명 객체 안에서 여러 메소드를 오버라이드해야 하는 경우에 훨씬 더 유용하다. 메소드가 하나뿐인 인터페이스를 구현해야 한다면 코틀린의 SAM 변환(함수 리터럴을 변환해 SAM으로 만듦) 지원을 활용하는 편이 낫다. SAM 변환을 사용하려면 무명 객체 대신 함수 리터럴을 사용해야 한다.