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
