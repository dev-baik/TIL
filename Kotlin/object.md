## 코틀린 클래스 안에는 정적인 멤버가 없다.
코틀린에서는 자바 static 키워드를 지원하지 않는다.

**HOW)**
1. 패키지 수준의 최상위 함수를 이용하여 자바의 정적 메소드 역할을 거의 대신할 수 있다.
2. 객체 선언을 이용하여 자바의 정적 메소드 역할 중 코틀린 최상위 함수가 대신할 수 없는 역할이나 정적 필드를 대신할 수 있다.

## 객체
사전적 정의 : 인식 가능한 물체/물건을 의미하며, 객체들은 각자의 고유한 속성과 동작을 갖고 있다.

소프트웨어 관점 : 서로 연관있는 변수(속성: property)들을 묶어놓은 데이터 덩어리

객체의 상태는 변수들을 통해 나타내며, 객체의 행동은 메소드를 통해 정의된다.

## object 키워드

> 💡 클래스 선언과 그 클래스에 속한 단일 인스턴스의 선언을 합친 선언으로, 클래스가 처음으로 참조될 때 초기화되며, 이후에는 동일한 인스턴스를 재사용하므로 싱글턴 패턴을 구현하기 좋다.

> ✅ **싱글턴(singleton) 패턴** : 어떤 클래스의 인스턴스가 하나만 생성되도록 보장하고, 그 인스턴스에 접근할 수 있는 전역적인 지점을 제공하는 디자인 패턴으로, 인스턴스가 필요한 시점에 생성된다.
>
> ✅ **지연 초기화** : 인스턴스가 처음으로 필요한 시점에 생성되므로, 프로그램이 실행되는 동안 불필요한 자원 소비를 피할 수 있다.

> **Q1. 인스턴스가 단 하나만 생성된다면?** : 동일한 인스턴스에 접근할 수 있어 데이터와 동작이 일관되게 유지된다.

- 클래스와 마찬가지로 프로퍼티, 메소드, 초기화 블록(init) 등이 들어갈 수 있다.
  - 객체 선언과 동시에 생성자 호출 없이 바로 만들어지기 때문에 주/부 생성자는 사용할 수 없다.
  - 객체 선언도 클래스나 인터페이스를 상속할 수 있다.
    - 인터페이스를 구현해야 하는데 그 구현 내부에 다른 상태가 필요하지 않은 경우 사용하면 좋다.
```kotlin
object CaseInsensitiveFileComparator : Comparator<File> {
    override fun compare(file1: File, file2: File): Int {
        return file1.path.compareTo(file2.path, ignoreCase = false)
    }
}

println(CaseInsensitiveFileComparator.compare(File("/User"), File("/User"))) // 0
```

> **Q2. 다른 상태가 필요하지 않으면 뭐가 좋은데?** : Object를 사용하면 동일한 인스턴스가 공유되므로 객체의 상태가 일관되게 유지되며, 한 객체의 상태 변경이 다른 객체에 영향을 미치지 않는다.

> **Q3. 그냥 최상위 함수로 정의하면 안돼?** : 함수가 여러 객체와 관계를 맺지 않고 하나의 특정 객체(File)와만 관계를 맺는다면, 최상위 함수보다는 클래스 내부에 정의하는 것이 좋다.
> 
> **Q3-1. 왜 좋은데?** : 특정 객체의 멤버 함수로 정의된 함수는 해당 객체의 내부 상태와 메서드에 쉽게 접근할 수 있다.

> **Q4. 객체 선언이 기본적으로 싱글턴이면 문제없어?** : 객체 생성을 제어할 방법이 없고 생성자 파라미터를 지정할 수 없어 객체 선언이 항상 적합하지는 않다.
>
> ☑️ 단위 테스트를 하거나 소프트웨어 시스템의 설정이 달라질 때 객체를 대체하거나 객체의 의존관계를 바꿀 수 없다. 따라서 의존관계 주입 프레임워크와 코틀린 클래스를 함께 사용해야 한다.

- 클래스 안에서 객체를 선언할 수도 있다. 해당 객체도 인스턴스는 단 하나뿐이다.
```kotlin
// 중첩 객체를 사용해 Comparator 구현하기
data class Person(val name: String) {
    object NameComparator : Comparator<Person> {
        override fun compare(p1: Person?, p2: Person?): Int =
            p1.name.compareTo(p2.name)
    }
}

val persons = listOf(Person("Bob"), Person("Clice"), Person("Alice"))
println(persons.sortedWith(Person.NameComparator))
// 결과: [Person(name=Alice), Person(name=Bob), Person(name=Clice)]
```

## Companion Object

> 💡 클래스 당 하나의 객체를 가지며 인스턴스를 생성하지 않고도 클래스 내부 정보에 접근해야 하는 필드와 메소드를 관리할 때 사용하는 객체로서, 해당 클래스가 로드되면 초기화가 이루어진다. 

- 클래스의 인스턴스와 관계없이 클래스 내부 정보에 접근해야 하는 필드와 메소드를 관리할 때 사용한다.
```kotlin
class MyClass {
    private val secretData = "This is a secret."
  
    companion object MyCompanion {
        fun accessSecretData(myClassInstance: MyClass) {
            println(myClassInstance.secretData)
        }
    }
}

fun main() {
    val myInstance = MyClass()
  
    // MyClass의 private 멤버에 접근하는 함수 호출
    MyClass.MyCompanion.accessSecretData(myInstance)
}
```
- 동반 객체는 자신을 둘러싼 클래스의 모든 private 멤버에 접근할 수 있다. 
  - 따라서 동반 객체는 바깥쪽 클래스의 private 생성자도 호출할 수 있다.
```kotlin
class User private constructor(private val nickname: String) {
    // 객체 선언을 사용한 팩토리 메소드 정의
    companion object Factory {
        fun newSubscribingUser(email: String): User {
            // 동반 객체에서 바깥쪽 클래스의 private 생성자 호출
            return User(email.substringBefore('@'))
        }
    }

    // 클래스 내부의 멤버 함수
    fun getNickName(): String {
        return nickname
    }
}

fun main() {
    val instance = User.newSubscribingUser("bob@gmail.com")
    println(instance.getNickName()) // bob
}
```

- 동반 객체에 이름을 지정하지 않는 경우 자동으로 Companion이 된다.
  - 동반 객체는 클래스 안에 정의된 일반 객체로서 인터페이스를 상속하거나, 동반 객체 안에 확장 함수와 프로퍼티를 정의할 수 있다.

#### 동반 객체에서 인터페이스 구현하기 (+ 팩토리 메소드 패턴)
```kotlin
interface JSONFactory<T> {
    fun fromJSON(json: String): T
}

data class Person(val name: String, val age: Int) {
    companion object : JSONFactory<Person> {
        override fun fromJSON(json: String): Person {
            val jsonObj = TODO("JSON 문자열을 JSON 객체로 파싱하는 로직")
            val name = jsonObj.getString("name")
            val age = jsonObj.getInt("age")
            return Person(name, age)
        }
    }
}

fun <T> loadFromJSON(factory: JSONFactory<T>): T {
    val jsonString = "{ \"name\": \"John\", \"age\": 30 }"
    return factory.fromJSON(jsonString)
}

fun main() {
    val person: Person = loadFromJSON(Person)
    println(person) // Person(name=John, age=30)
}
```

> ✅ **팩토리 메소드 패턴** : 객체 생성 코드와 실제 구현 코드를 분리하여 코드의 유지보수성을 향상시키는 디자인 패턴으로, companion object를 활용하여 객체 생성 로직을 클래스 내부에 캡슐화하는 방식으로 동작한다.

#### 동반 객체에 대한 확장 함수 정의하기
```kotlin
class Person(val firstName: String, val lastName: String) {
    companion object { } // 비어있는 동반 객체를 선언한다.
}

fun Person.Companion.fromJSON(json: String): Person {
    val jsonObj = TODO("JSON 문자열을 JSON 객체로 파싱하는 로직")
    val firstName = jsonObj.getString("firstName")
    val lastName = jsonObj.getString("lastName")
    return Person(firstName, lastName)
}

fun main() {
    val json = "{ \"firstName\": \"John\", \"lastName\": \"Doe\" }"
    val person: Person = Person.fromJSON(json)
    println(person) // Person(firstName=John, `lastName=Doe)
}
```

- 동반 객체를 포함하는 클래스를 확장해야하는 경우에는 동반 객체 멤버를 하위 클래스에서 오버라이드할 수 없으므로 부 생성자를 사용하는 편이 더 낫다.

## Object와 Companion Object 차이점
- 객체 선언은 한 클래스 내에 여러개 존재할 수 있지만, 동반 객체는 단 하나만 존재할 수 있다.
- 객체 선언은 클래스 전체가 하나의 싱글턴 객체로 선언되지만, 동반 객체는 클래스 내에 일부분이 싱글턴 객체로 선언된다.
- 객체 선언이 선언된 클래스는 해당 클래스가 최초로 사용될 때 초기화되지만, 동반 객체는 해당 클래스가 로드되면 초기화가 이루어진다.

#### 참고 사이트
- https://umbum.dev/597/
- https://hodie.tistory.com/54