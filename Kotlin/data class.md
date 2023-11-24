## Java Data Class
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

> ✅ 보일러 플레이트 코드 : 여러 곳에서 재사용되며, 반복적으로 비슷한 형태를 띄는 코드

## Kotlin Data Class
데이터 관리에 최적화된 클래스로 toString(), equals(), hashCode(), copy(), componentN() 5가지 유용한 함수들을 내부적으로 컴파일러가 자동 생성해준다.
```kotlin
data class Client(
    var name: String,
    val postCode: Int
)

val client1 = Client("오현석", 4122)
println(client1) // Client(name=오현석, postalCode=4122)
```

### toString()
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

### equals()
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

### hashCode()
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
```kotlin
class Client(val name: String, val postalCode: Int) {
    ...
    override fun hashCode(): Int = name.hashCode() * 31 + postalCode
}
```

### Copy()
- 객체의 값들을 변경하여 새로운 복사본을 생성하는 메소드
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

    println(lee.hashCode() == copyLee.hashCode()) // 새 복사본 생성
    println(copyLee) // Client(name=이계영, postalCode=4000)
}
```

### componentN()
객체를 분해하여 해당 객체의 각 속성을 순서대로 반환하는 메서드
```kotlin
val client1 = Client("오현석", 4122)
val (name, postalCode) = client1

print("Client(name=$name, postalCode=$postalCode)") 

// Client(name=오현석, postalCode=4122)
```

#### 참고 사이트
- https://velog.io/@haero_kim/Kotlin-%EA%B0%90%EB%8F%99-%EC%8B%A4%ED%99%94-Data-Class-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0