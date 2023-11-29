# Java Data Class
자바에서는 데이터 클래스를 직접 제공하지 않기 때문에 `toString()`을 오버라이딩하고, Getter/Setter 등을 정의해줘야 한다.
```java
class Client {
    String name;
    int postalCode;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

public class Main {
    public static void main(String[] args) {
        Client client1 = new Client();

        client1.setName("오현석");
        client1.postalCode = 4122;

        System.out.println(client1.toString()); // Client@58d25a40
    }
}
```
- 보일러 플레이트 코드가 너무 비대하다.

> ✅ **보일러 플레이트 코드** : 여러 곳에서 재사용되며, 반복적으로 비슷한 형태를 띄는 코드

# Kotlin Data Class
데이터 관리에 최적화된 클래스로 toString(), equals(), hashCode(), copy(), componentN() 5가지 유용한 함수들을 내부적으로 컴파일러가 자동 생성해준다.
```kotlin
data class Client(
    var name: String,
    val postCode: Int
)

val client1 = Client("오현석", 4122)
println(client1) // Client(name=오현석, postalCode=4122)
```

## toString()
객체의 문자열 표현을 반환하는 메소드
```java
class People {
    String name;
    int postalCode;

    @Override
    public String toString() {
        return "Client(name=" + name + ", postalCode=" + Integer.toString(postalCode) + ")";
    }
    ...
}
```
```kotlin
class Client(var name: String, val postalCode: Int) {
    override fun toString() = "Client(name=$name, postalCode=$postalCode)"
}
```

## equals()
객체의 동등성을 비교하는 메소드

<table>
    <tr>
        <th></th>
        <th>객체의 동등성</th>
        <th>참조 동일성</th>
    </tr>
    <tr>
        <th>Java</th>
        <td>equals()</td>
        <td>==</td>
    </tr>
    <tr>
        <th>Kotlin</th>
        <td>==</td>
        <td>===</td>
    </tr>
</table>

> ✅ **객체의 동등성** : 두 객체가 동일한 값을 가지고 있는지를 비교
>
> ✅ **참도 동일성** : 두 객체가 메모리 상에서 같은 위치에 있는지, 즉 동일한 객체인지를 비교

```kotlin
class Client(val name: String, val postalCode: Int)
    
fun main() { 
    val client1 = Client("오현석", 4122)
    val client2 = Client("오현석", 4122)
    
    print(client1 == client2) // false
}
```
- 참조 동일성을 검사하지 않고 객체의 동등성을 검사한다.
```kotlin
class Client(val name: String, val postalCode: Int) {
    override fun equals(other: Any?): Boolean {
        if (other == null || other !is Client)
            return false
        return name == other.name &&
              postalCode == other.postalCode
    }
}

fun main() {
    val client1 = Client("오현석", 4122)
    val client2 = Client("오현석", 4122)

    print(client1 == client2) // true
}
```

## hashCode()
HashMap과 같은 해시 기반 컨테이너에서 키로 사용할 수 있는 객체의 해시 코드를 반환하는 메소드
```kotlin
class Client(val name: String, val postalCode: Int)
    
fun main() { 
    val processed = hashSetOf(Client("오현석", 4122))
    println(processed.contains(Client("오현석", 4122))) // false
}
```
- hashSet : 원소를 비교할 때 비용을 줄이기 위해 먼저 객체의 해시 코드를 비교하고 해시 코드가 같은 경우에만 실제 값을 비교한다.
- 해시 코드 규칙 : equals()가 true를 반환하는 두 객체는 반드시 같은 hashCode()를 반환해야 한다.
  - **WHY)** 컬렉션들은 객체를 저장하거나 검색할 때 객체의 hashCode() 값을 사용하여 [버킷](https://velog.io/@syoung125/%ED%95%B4%EC%8B%9C%ED%85%8C%EC%9D%B4%EB%B8%94%EC%9D%B4%EB%9E%80)을 결정한다. 따라서 equals()가 true를 반환하는 객체는 같은 버킷에 위치해야 한다.

```kotlin
class Client(val name: String, val postalCode: Int) {
    ...
    override fun hashCode(): Int = name.hashCode() * 31 + postalCode
}
```

> ✅ **버킷** : 해시 테이블은 각각 Key 값에 해시 함수를 적용해 배열의 고유한 index를 생성하고, 해당 인덱스를 활용해 값을 저장하거나 검색하게 된다. 여기서 실제 값이 저장되는 장소를 버킷 또는 슬롯이라고 한다.
> 
> ✅ **객체의 해시 코드** : 객체를 식별하기 위한 정수 값으로, 객체의 내용을 기반으로 생성되며 동일한 내용의 객체는 동일한 해시 코드를 가질 수 있다.

## Copy()
객체의 값들을 변경하여 새로운 복사본을 생성하는 메소드
```kotlin
class Client(val name: String, val postalCode: Int) {
    ...
    override fun toString() = "Client(name=$name, postalCode=$postalCode)"
  
    override fun hashCode(): Int = name.hashCode() * 31 + postalCode
  
    fun copy(name: String = this.name,
             postalCode: Int = this.postalCode) =
      Client(name, postalCode)
}

fun main() {
    val lee = Client("이계영", 4122)
    val copyLee = lee.copy(postalCode = 4000)

    println(lee.hashCode() == copyLee.hashCode()) // false
    println(copyLee) // Client(name=이계영, postalCode=4000)
}
```

### 얕은 복사(Shallow Copy)
객체를 복사할 때, 복사된 객체와 원본 객체가 같은 데이터에 대한 참조를 공유하게 된다. 즉, 복사된 객체에서 참조하는 객체의 값을 변경하면, 원본 객체에서 참조하는 객체의 값도 함께 변경된다.
```kotlin
class Person(var name: String, val age: Int)

fun main() {
    val originalPerson = Person("John", 25)
    val copiedPerson = originalPerson

    copiedPerson.name = "Kim"

    println(originalPerson) // Person(name=Kim, age=25)
    println(copiedPerson) // Person(name=Kim, age=25)
}
```
### Copy는 얕은 복사!
- Copy 함수는 data class 내의 원시(Primitive) 타입을 제외한 참조 타입은 깊은 복사를 지원하지 않는다.
```kotlin
data class Example(var id: Int, var list: MutableList<Int>)

fun main() {
val instance1 = Example(1, mutableListOf(1, 2))
val instance2 = instance1.copy()

    instance1.id = 3
    instance1.list.add(3)

    println(instance2.id) // 1
    println(instance2.list) // [1, 2, 3]
}
```
```kotlin
data class CustomObject(val name: String)

data class Example(val primitiveField: Int, val customField: CustomObject)

fun main() {
    val original = Example(42, CustomObject("original"))
    val copy = original.copy()

    println(original)  // Example(primitiveField=42, customField=CustomObject(name=original))
    println(copy) // Example(primitiveField=42, customField=CustomObject(name=original))
    
    println(original.customField === copy.customField)  // true
}
```

### 깊은 복사(Deep Copy)
객체를 복사할 때, 객체가 참조하는 모든 객체까지 복사한다. 즉, 복사된 객체와 원본 객체가 서로 독립적으로 존재하여 복사된 객체에서 참조하는 객체의 값을 변경해도, 원본 객체에서 참조하는 객체의 값은 변경되지 않는다.

**1. data class의 copy 메서드 사용**
```kotlin
data class Person(var name: String, val age: Int)

fun main() {
    val originalPerson = Person("John", 25)

    val copiedPerson = originalPerson.copy()
    
    copiedPerson.name = "Kim"

    println(originalPerson) // Person(name=John, age=25)
    println(copiedPerson) // Person(name=Kim, age=25)
}
```

**2. 복사 생성자 이용**
```kotlin
data class CustomObject(val name: String) {
    constructor(original: CustomObject) : this(original.name)
}

data class Example(val primitiveField: Int, val customField: CustomObject) {
    constructor(original: Example) : this(original.primitiveField, CustomObject(original.customField))
}

fun main() {
    val original = Example(42, CustomObject("original"))
    val copy = Example(original)

    println(original.customField === copy.customField)  // false
}
```

**3. 복사 팩토리 이용**
```kotlin
data class CustomObject(val name: String) {
    companion object {
        fun newInstance(original: CustomObject): CustomObject {
            return CustomObject(original.name)
        }
    }
}

data class Example(val primitiveField: Int, val customField: CustomObject) {
    companion object {
        fun newInstance(original: Example): Example {
            return Example(original.primitiveField, CustomObject.newInstance(original.customField))
        }
    }
}

fun main() {
    val original = Example(42, CustomObject("original"))
    val copy = Example.newInstance(original) // 깊은 복사 수행

    println(original.customField === copy.customField)  // false
} 
```

**4. 직접 객체 생성**
```kotlin
data class User(var id: Int = 0, var name: String = "")

fun main() {
    val originalUser = User(1, "Lee")
    val copiedUser = User()

    copiedUser.id = originalUser.id
    copiedUser.name = originalUser.name

    originalUser.id = 3
    originalUser.name = "Kim"

    println(originalUser) // User(id=3, name=Kim)
    println(copiedUser) // User(id=1, name=Lee)
} 
```

**5-1. Cloneable 인터페이스를 상속받아서 clone() 메서드를 오버라이딩**
```kotlin
class SampleClass(var id: Int, var list: MutableList<Int>) : Cloneable {
    public override fun clone(): SampleClass {
        return super.clone() as SampleClass
    }
}

fun main() {
    val instance1 = SampleClass(1, mutableListOf(1, 2))
    val instance2 = instance1.clone()

    instance1.id = 3
    instance1.list.add(3)

    println(instance2.id) // 1
    println(instance2.list) // [1, 2, 3]
}
```
**5-2. Clonable도 Copy 함수와 같이 원시 타입을 제외한 참조 타입은 깊은 복사를 지원하지 않는다.**
```kotlin
class SampleClass(var id: Int, var list: MutableList<Int>) : Cloneable{
    public override fun clone(): SampleClass {
        val sampleClass = super.clone() as SampleClass
        sampleClass.list = mutableListOf<Int>().apply { addAll(list) }
        return sampleClass
    }
}

fun main() {
    val instance1 = SampleClass(1, mutableListOf(1, 2))
    val instance2 = instance1.clone()

    instance1.id = 3
    instance1.list.add(3)

    println(instance2.id) // 1
    println(instance2.list) // [1, 2]
}
```

## componentN()
객체를 분해하여 해당 객체의 각 속성을 순서대로 반환하는 메서드
```kotlin
val client1 = Client("오현석", 4122)
val (name, postalCode) = client1

print("Client(name=$name, postalCode=$postalCode)") 

// Client(name=오현석, postalCode=4122)
```

#### 참고 사이트
- https://velog.io/@haero_kim/Kotlin-%EA%B0%90%EB%8F%99-%EC%8B%A4%ED%99%94-Data-Class-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0
- https://velog.io/@wlsrhkd4023/Kotlin-%EC%96%95%EC%9D%80%EB%B3%B5%EC%82%ACShallow-Copy%EC%99%80-%EA%B9%8A%EC%9D%80%EB%B3%B5%EC%82%ACDepp-Copy
- https://velog.io/@dddooo9/Kotlin-%EA%B9%8A%EC%9D%80-%EB%B3%B5%EC%82%ACDeep-Copy-%ED%95%98%EB%8A%94-3%EA%B0%80%EC%A7%80-%EB%B0%A9%EB%B2%95