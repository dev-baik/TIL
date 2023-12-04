# by 키워드
프로퍼티의 값을 다른 객체에게 위임할 때 사용하는 키워드로, 위임 패턴을 구현하여 프로퍼티의 동작을 다른 객체에게 위임할 수 있다.

## 위임 패턴(Delegate Pattern)
- **WHAT)** 객체가 특정 기능을 직접 처리하지 않고, 다른 객체에게 해당 기능의 처리를 위임하여 그 객체가 일을 처리하도록 하는 디자인 패턴
- **WHEN)** 코틀린의 클래스와 메소드는 가시성 변경자가 기본적으로 final이다. open 변경자를 통해 상속을 허용할 수 있지만, 하위 클래스가 기반 클래스에 대해 가졌던 가정이 기반 클래스를 변경함으로서 깨져버리는 경우, 하위 클래스의 동작이 예기치 않게 바뀔 수 있다. 
  - 그래서 상속을 허용하지 않는 클래스에 새로운 동작을 추가해야 할 때 사용한다.

> 💡 **상속(Inheritance)** : 하나의 클래스가 다른 클래스로부터 속성과 동작을 물려받는 것을 의미한다.
>
> 💡 **상속의 장점** : 클래스의 변수와 메소드를 모두 받아오기 때문에 하위 클래스에서 재사용할 수 있다. 클래스 간에 계층 구조를 형성하여 코드를 구조화할 수 있다.
> 
> 💡 **상속의 단점** : 결합도가 높아져 부모(기반) 클래스의 변경이 하위 클래스에 영향을 미칠 수 있다.

### 상속 vs 구성
- 상속은 "is-a" 관계를 나타낸다 : 자식 클래스는 부모 클래스의 일종이다.
- 구성은 "has-a" 관계를 나타낸다 : 클래스 A가 클래스 B의 객체를 가지고 있다.

> ✅ **구성Composition)** : 한 클래스가 다른 클래스의 인스턴스를 필드로 가지는 것을 의미하며, 두 클래스 간의 결합도를 낮출 수 있다.  

## 데코레이터 패턴(Decorator Pattern)
기존 객체를 변경하지 않고 그 기능을 동적으로 확장하려는 목적으로 사용되는 패턴으로, 서브클래스를 만들지 않고도 기능을 확장할 수 있다.

### 만약 서브클래스를 만든다면?
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
- 준비코드가 상당히 많이 필요하다.

### 위임 패턴을 적용하여 보일러 플레이트 코드 줄이기
```kotlin
class DelegatingCollection<T>(
    innerList: Collection<T> = ArrayList<T>()
) : Collection<T> by innerList { }
```
- 이런 위임을 언어가 제공하는 일급 시민 기능으로 지원한다는 점이 코틀린의 장점이다.

> ✅ **일급 시민 기능으로 지원?** : 위임을 변수(innerList)에 할당하고 함수로 전달하며, 일반적인 값과 동일하게 다룰 수 있다는 것을 의미한다.

- 인터페이스를 구현할 때 by 키워드를 통해 그 인터페이스에 대한 구현을 다른 객체에 위임 중이라는 사실을 명시할 수 있다.

### 일부 동작 변경 및 확장
```kotlin
interface CustomFunctionality {
    fun customFunction(): String
}

class CountingSet<T>(
    val innerSet: MutableCollection<T> = HashSet<T>()
) : MutableCollection<T> by innerSet, CustomFunctionality { // 다중 상속
    var objectsAdded = 0
  
    override fun add(element: T): Boolean {
        objectsAdded++
        return innerSet.add(element)
    }
  
    override fun addAll(c: Collection<T>): Boolean {
        objectsAdded += c.size
        return innerSet.addAll(c)
    }

    override fun customFunction(): String {
        return "Custom functionality from DelegatingCollection"
    }
}

fun main() {
    val cset = CountingSet<Int>()
    cset.addAll(listOf(1, 1, 2))
    println("${cset.objectsAdded} objects were added, ${cset.size} remain")    
}

// 3 objects were added, 2 remain
```

## 위임 프로퍼티
프로퍼티의 getter와 setter 로직을 다른 객체에 위임하여 값을 뒷받침하는 필드에 단순히 저장하는 것보다 더 복잡한 방식으로 프로퍼티를 구현할 수 있다.

**위임 객체(delegate)** : 작업을 대신 처리하는 도우미 객체이다. 즉, 프로퍼티의 getter와 setter 동작을 정의하고, 프로퍼티에 대한 실제 로직을 수행한다.

- Delegate 클래스의 인스턴스를 위임 객체로 사용하는 경우
  - 프로퍼티 위임 객체가 따라야 하는 관례를 따르는 모든 객체를 위임에 사용할 수 있다.
```kotlin
class Delegate {
    operator fun getValue(...) { ... }
    operator fun setValue(..., value: Type) { ... }
}

class Foo {
    var p: Type by Delegate()
}

// p의 게터나 세터는 Delegate 타입의 위임 프로퍼티 객체에 있는 메소드를 호출한다.
val foo = Foo()
val oldValue = foo.p
foo.p = newValue
```
- 위임 객체를 사용하지 않는 경우
```kotlin
class Foo {
    private val delegate: Delegate() // 컴파일러가 생성한 도우미 프로퍼티
  
    // "p" 프로퍼티를 위해 컴파일러가 생성한 접근자는
    // "delegate"의 getValue와 setValue 메소드를 호출한다.
    var p: Type
      set(value: Type) = delegate.setValue(..., value)
      get() = delegate.getValue(...)
}
```

### by lazy()를 사용한 프로퍼티 초기화 선언

> 💡 특정 프로퍼티의 초기화를 지연시켰다가 실제로 그 부분의 값이 필요한 경우 초기화할 때 쓰이는 패턴

- 지연 초기화를 뒷받침하는 프로퍼티를 통해 구현하기
```kotlin
class Email { }

fun loadEmails(person: Person): List<Email> {
    println("${person.name}의 이메일을 가져옴")
    return listOf()
}

class Person(val name: String) {
  // 데이터를 저장하고 emails의 위임 객체 역할을 하는 emails 프로퍼티
  private var _emails: List<Email>? = null

    val emails: List<Email> 
        get() { 
            if (_emails == null) {
                // 최초 접근 시 이메일을 가져온다.
                _emails = loadEmails(this)
            }
        // 저장해둔 데이터가 있으면 그 데이터를 반환한다.
            return _emails!!
        }
}

val p = Person("Alice")
p.emails
// 최초로 emails를 읽을 때 단 한번만 이메일을 가져온다.
// 결과: Load emails for Alice
p.emails
```
- **문제점)** <span style='background-color: #fff5b1'/>멀티 스레드 환경에서 스레드 안전하지 않아서 여러 스레드가 동시에 접근하는 경우 데이터(emails 프로퍼티)의 일관성이 보장되지 않는다.

- **HOW)** 지연 초기화를 위임 프로퍼티를 통해 구현하기 : 데이터를 저장할 때 쓰이는 뒷받침하는 프로퍼티와 값이 오직 한 번만 초기화됨을 보장하는 게터 로직을 함께 캡슐화해준다.
```kotlin
class Person(val name: String) {
    val emails by lazy { loadEmails(this) }
}
```
- 필요에 따라 동기화에 사용할 락을 lazy 함수에 전달할 수 있다.
```kotlin
import kotlin.concurrent.thread

class ExpensiveObject {
    init {
        println("ExpensiveObject initialized")
    }
}

class Example {
    // 동기화에 사용할 락을 lazy 함수에 전달
    val lazyProperty: ExpensiveObject by lazy(LazyThreadSafetyMode.SYNCHRONIZED) {
        println("Initializing Lazy Property")
        ExpensiveObject()
    }

    // 다중 스레드 환경에서 사용하지 않을 프로퍼티를 위해 lazy 함수가 동기화를 하지 못하게 막기
    // val lazyProperty: ExpensiveObject by lazy(LazyThreadSafetyMode.NONE) {
    //    ...
    // }
}

fun main() {
    val example = Example()
  
    // 여러 스레드에서 동시에 접근하는 상황 시뮬레이션
    repeat(5) {
        thread {
            println("Accessing Lazy Property: ${example.lazyProperty}")
        }
    }
  
    // 잠시 대기하여 모든 스레드가 실행을 마치도록 함
    Thread.sleep(1000)
}
```

### 위임 프로퍼티 컴파일 규칙
```kotlin
// 위임 객체
class Delegate {
    private var field: Type? = null
    
    operator fun getValue(thisRef: C, property: KProperty<*>): Type {
        return field
    }
  
    operator fun setValue(thisRef: C, property: KProperty<*>, newValue: Type) {
        field = newValue
    }
}
```

<table>
  <tr>
    <th>getValue</th>
    <th>setValue</th>
  </tr>
  <tr>
    <td><strong>thisRef</strong> : 위임을 사용하는 클래스와 같은 타입이거나 Any 타입이어야 한다.</td>
    <td><strong>thisRef</strong> : 위임을 사용하는 클래스와 같은 타입이거나 Any 타입이어야 한다.</td>
  </tr>
  <tr>
    <td><strong>property</strong> : Property<*>거나 Any 타입이어야 한다.</td>
    <td><strong>property</strong> : Property<*>거나 Any 타입이어야 한다.</td>
  </tr>
  <tr>
    <td></td>
    <td><strong>newValue</strong> : 위임을 사용하는 프로퍼티와 같은 타입이거나 Any 타입이어야 한다.</td>
  </tr>
</table>

> ✅ **KProperty** : 코틀린의 리플렉션 API 중 하나로, 프로퍼티를 나타내는 역할을 한다. KProperty의 인스턴스는 `::` 연산자로 얻을 수 있습니다.
>
> ✅ **리플렉션(Reflection)** : 실행 중인 프로그램의 클래스, 메소드, 필드 등과 같은 구조를 동적으로 살펴보거나 수정할 수 있는 능력

```kotlin
class C {
    var prop: Type by Delegate()
}
```
- 컴파일러는 MyDelegate 클래스의 인스턴스를 감춰진 프로퍼티에 저장하며 그 감춰진 프로퍼티를 `<delegate>`라는 이름으로 부른다.
- 컴파일러는 프로퍼티를 표현하기 위해 KProperty 타입의 객체를 사용한다. 이 객체를 <property>라고 부른다.

```kotlin
class C {
    // 컴파일러가 생성하는 도우미 프로퍼티
    private val <delegate> = Delegate()
    
    // 컴파일러가 생성하는 접근자
    var prop: Type
      get() = <delegate>.getValue(this, <property>)
      set(value : Type) = <delegate>.setValue(this, <property>, value)
}
```

### 위임 프로퍼티 구현
**1. 자바 빈 클래스의 필드에 PropertyChangeSupport 인스턴스를 저장하고 프로퍼티 변경 시 인스턴스에게 처리를 위임하는 방식으로 통지 기능을 구현하기**
- **PropertyChangeSupport 클래스** : 리스너의 목록을 관리하고 PropertyChangeEvent 이벤트가 들어오면 목록의 모든 리스너에게 이벤트를 통지한다.
- **자바 빈 클래스** : 일정한 규약을 따르는 클래스로, 기본 생성자, getter 및 setter 메서드를 포함하며, 멤버 변수는 private으로 선언되고 이에 접근하기 위한 표준 메서드를 제공한다. 이러한 클래스는 재사용 가능한 컴포넌트를 만드는데 사용된다.
```kotlin
open class PropertyChangeAware {
    protected val changeSupport = PropertyChangeSupport(this)
  
    fun addPropertyChangeListener(listener: PropertyChangeListener) {
        changeSupport.addPropertyChangeListener(listener)
    }
  
    fun removePropertyChangeListener(listener: PropertyChangeListener) {
        changeSupport.removePropertyChangeListener(listener)      
    }
}
```
- 프로퍼티 변경 통지를 직접 구현하기
```kotlin
class Person(
    val name: String, age: Int, salary: Int
) : PropertyChangeAware() {
    var age: Int = age
      set(newValue) {
          val oldValue = field
          field = newValue
    
          // 프로퍼티 변경을 리스너에게 통지한다.
          changeSupport.firePropertyChange(
            "age", oldValue, newValue)
        }

    var salary: Int = salary
      set(newValue) {
          val oldValue = field
          field = newValue
    
          changeSupport.firePropertyChange(
            "salary", oldValue, newValue)
      }
}

val p = Person("Dmitry", 34, 2000)
p.addPropertyChangeListener(
  PropertyChangeListener { event ->
    println("Property ${event.propertyName} changed " +
            "from ${event.oldValue} to ${event.newValue}")
  }
)
p.age = 35 // Property age changed from 34 to 35
p.salary = 2100 // Property salary changed from 2000 to 2100 
```

**2. 프로퍼티의 값을 저장하고 필요에 따라 통지를 보내주는 클래스를 도입하여 세터의 중복 코드를 줄이기**
- 도우미 클래스를 통해 프로퍼티 변경 통지 구현하기
```kotlin
class ObservableProperty(
    val propName: String, var propValue: Int,
    val changeSupport: PropertyChangeSupport
) {
    fun getValue(): Int = propValue
  
    fun setValue(newValue: Int) {
        val oldValue = propValue
        propValue = newValue
        changeSupport.firePropertyChange(propName, oldValue, newValue)
    }
}

class Person(
    val name: String, age: Int, salary: Int
) : PropertyChangeAware() {
    val _age = ObservableProperty("age", age, changeSupport)
    var age: Int
      get() = _age.getValue()
      set(value) { _age.setValue(value) }
  
    val _salary = ObservableProperty("salary", salary, changeSupport)
    val salary: Int
      get() = _salary.getValue()
      set(value) { _salary.setValue(value) }
} 
```

**3. 각각의 프로퍼티마다 ObservableProperty를 만들고 게터와 세터에서 ObservableProperty에 작업을 위임하는 준비 코드가 상당 부분을 줄이기 위해 코틀린의 위임 프로퍼티 기능을 활용하기**
```kotlin
class ObservableProperty(
    var propValue: Int, val changeSupport: PropertyChangeSupport
) {
    operator fun getValue(p: Person, prop: KProperty<*>): Int = propValue
  
    operator fun setValue(p: Person, prop: KProperty<*>, newValue: Int) {
        val oldValue = propValue
        propValue = newValue
        changeSupport.firePropertyChange(prop.name, oldValue, newValue)
    }
}

class Person(
    val name: String, age: Int, salary: Int
) : PropertyChangeAware() {
    var age: Int by ObservableProperty(age, changeSupport)
    var salary: Int by ObservableProperty(salary, changeSupport)
}
```

**기타 : 관찰 가능한 프로퍼티 로직을 제공하는 코틀린 표준 라이브러리**  
- Delegates.observable을 사용해 프로퍼티 변경 통지 구현하기
```kotlin
class Person(
    val name: String, age: Int, salary: Int
) : PropertyChangeAware() {
    private val observer = {
          prop: KProperty<*>, oldValue: Int, newValue: Int ->
        changeSupport.firePropertyChange(prop.namme, oldValue, newValue)
    }
  
    var age: Int by Delegates.observable(age, observer)
    var salary: Int by Delegates.observable(salary, observer)
}
```

### 프로퍼티 값을 맵에 저장
- **확장 가능한 객체** : 위임 프로퍼티를 활용하여 자신의 프로퍼티를 동적으로 정의하거나 변경할 수 있는 객체


- 값을 맵에 저장하는 프로퍼티 정의하기
```kotlin
class Person {
    // 추가 정보
    private val _attributes = hashMapOf<String, String>()
  
    fun setAttribute(attrName: String, value: String) {
        _attributes[attrName] = value
    }
  
    // 필수 정보
    val name: String
      // 수동으로 맵에서 정보를 꺼낸다.
      get() = _attributes["name"]!!
}
```
```kotlin
class Person {
    private val _attributes = hashMapOf<String, String>()
  
    fun setAttribute(attrName: String, value: String) {
        _attributes[attrName] = value
    }
  
    // 위임 프로퍼티로 맵을 사용한다.
    val name: String by _attributes
}

val p = Person()
val data = mapOf("name" to "Dmitry", "company" to "JetBrains")
for ((attrName, value) in data)
    p.setAttribute(attrName, value)
println(p.name) // Dmitry
```


#### 참고 사이트
- https://velog.io/@haero_kim/Kotlin-by-%ED%82%A4%EC%9B%8C%EB%93%9C%EC%9D%98-%EC%97%AD%ED%95%A0-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0
- https://velog.io/@heetaeheo/Kotlin-by-%ED%82%A4%EC%9B%8C%EB%93%9C%EC%9D%98-%EC%97%AD%ED%95%A0-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0