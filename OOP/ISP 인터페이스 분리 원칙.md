# ISP (Interface Segregation Principle)
- 범용적인 인터페이스 보다는 클라이언트(사용자)가 실제로 사용하는 Interface를 만들어야 한다는 의미로, 인터페이스를 사용에 맞게 끔 각기 분리해야 한다는 설계 원칙
- 인터페이스를 잘게 분리함으로써, 클라이언트의 목적과 용도에 적합한 인터페이스만을 제공하는 것
  - 클래스의 기능을 쉽게 파악할 수 있고, 유연하게 객체의 기능을 확장하거나 수정할 수 있다.
- SRP 원칙이 클래스의 단일 책임을 강조한다면, ISP는 인터페이스의 단일 책임을 강조한다.
- 핵심은 관련 있는 기능끼리 하나의 인터페이스에 모으되 지나치게 커지지 않도록 크기를 제한하라는 점이다.

```kotlin
// 사용자가 실제로 호출하는 인터페이스 정의
interface UserService {
    fun registerUser(username: String, password: String): Boolean
    fun loginUser(username: String, password: String): Boolean
    fun getUserInfo(username: String): String
    fun updateUserPassword(username: String, newPassword: String): Boolean
}

class UserServiceImpl : UserService {
    private val userDatabase = mutableMapOf<String, String>() // 간단한 사용자 데이터베이스

    override fun registerUser(username: String, password: String): Boolean {
        if (userDatabase.containsKey(username)) {
            println("Username already exists.")
            return false
        }
        userDatabase[username] = password
        println("User registered successfully.")
        return true
    }

    override fun loginUser(username: String, password: String): Boolean {
        if (userDatabase.containsKey(username) && userDatabase[username] == password) {
            println("Login successful.")
            return true
        }
        println("Invalid username or password.")
        return false
    }

    override fun getUserInfo(username: String): String {
        return "User: $username, Password: ${userDatabase[username] ?: "Not available"}"
    }

    override fun updateUserPassword(username: String, newPassword: String): Boolean {
        if (userDatabase.containsKey(username)) {
            userDatabase[username] = newPassword
            println("Password updated successfully.")
            return true
        }
        println("User not found.")
        return false
    }
}

fun main() {
    val userService: UserService = UserServiceImpl()

    userService.registerUser("john_doe", "password123")
    userService.loginUser("john_doe", "password123")
    println("User Info: ${userService.getUserInfo("john_doe")}")

    userService.updateUserPassword("john_doe", "new_password")
    userService.loginUser("john_doe", "new_password")
}

// User registered successfully.
// Login successful.
// User Info: User: john_doe, Password: password123
// Password updated successfully.
// Login successful.
```
- 만약 인터페이스의 추상 메서드들을 범용적으로 이것저것 구현한다면, 그 인터페이스를 상속받은 클래스는 자신이 사용하지 않는 인터페이스마저 억지로 구현해야 하는 상황이 올 수 있다.
- 또한 사용하지도 않는 인터페이스의 추상 메소드가 변경된다면 클래스에서도 수정이 필요하게 된다.

> ✅ **범용적인 인터페이스** : 특정 도메인이나 기능에 제한되지 않고, 여러 다양한 상황에서 사용될 수 있는 일반적인 형태의 인터페이스

```kotlin
interface Animal {
    fun makeSound()
}

class Cat : Animal {
    override fun makeSound() {
        println("냐옹")
    }
}

class Dog : Animal {
    override fun makeSound() {
        println("멍멍")
    }
}

fun main() {
    val cat: Animal = Cat()
    val dog: Animal = Dog()

    cat.makeSound() // 냐옹
    dog.makeSound() // 멍멍
}
```

# ISP 원칙 위반 예제
- 스마트폰 인터페이스 : 스마트폰이라면 가지고 있을 통화나 메세지 기능 이외에도 무선 충전, AR 뷰어, 생체인식 등의 다채로운 기능을 포함하고 있다.
```kotlin
interface ISmartPhone {
    fun call(number: String)
    fun message(number: String, text: String)
    fun wirelessCharge()
    fun AR()
    fun biometrics()
}
```
- 만약 갤럭시 S20이나 S21 클래스를 구현한다면, 최신 스마트폰 기종인 만큼 객체의 동작 모두가 필요하므로 ISP 원칙을 만족하게 된다.
```kotlin
class S20 : ISmartPhone {
    override fun call(number: String) { /*...*/ }

    override fun message(number: String, text: String) { /*...*/ }

    override fun wirelessCharge() { /*...*/ }

    override fun AR() { /*...*/ }

    override fun biometrics() { /*...*/ }
}

class S21 : ISmartPhone {
    override fun call(number: String) { /*...*/ }

    override fun message(number: String, text: String) { /*...*/ }

    override fun wirelessCharge() { /*...*/ }

    override fun AR() { /*...*/ }

    override fun biometrics() { /*...*/ }
}
```
- **문제점** : 최신 기종 스마트폰 뿐만 아니라 구형 기종 스마트폰 클래스도 다뤄야 할 상황처럼 갤럭시 S3 클래스를 구현해야 한다면 무선 충전, 생체인식과 같은 기능은 포함되어 있지 않다.
- 추상 메소드 구현 규칙상 오버라이딩은 하되, 메서드 내부는 빈공간으로 두거나 예외를 던지도록 설정해야 한다.
  - 결국 필요하지도 않은 기능을 어쩔 수 없이 구현해야하는 낭비가 발생된 것이다.
```kotlin
class S3 : ISmartPhone {
    override fun call(number: String) { /*...*/ }

    override fun message(number: String, text: String) { /*...*/ }

    override fun wirelessCharge() {
        println("지원하지 않는 기능입니다.")
    }

    override fun AR() {
        println("지원하지 않는 기능입니다.")
    }

    override fun biometrics() {
        println("지원하지 않는 기능입니다.")
    }
}
```
- **HOW)** 각각의 기능에 맞게 인터페이스를 잘게 분리하도록 구성한다.
- 그리고 잘게 분리된 인터페이스를 클래스가 지원되는 기능만을 선별하여 implements 하면 ISP 원칙이 지켜지게 된다.
```kotlin
interface IPhone {
    fun call(number: String) // 통화 기능
    fun message(number: String, text: String) // 문제 메세지 전송 기능
}

interface WirelessChargable {
    fun wirelessCharge() // 무선 충전 기능
}

interface ARable {
    fun AR() // 증강 현실(AR) 기능
}

interface Biometricsable {
    fun biometrics() // 생체 인식 기능
}
```
```kotlin
class S21 : IPhone, WirelessChargable, ARable, Biometricsable {
    override fun call(number: String) { /*...*/ }}

    override fun message(number: String, text: String) { /*...*/ }

    override fun wirelessCharge() { /*...*/ }

    override fun AR() { /*...*/ }

    override fun biometrics() { /*...*/ }
}

class S3 : IPhone {
    override fun call(number: String) { /*...*/ }

    override fun message(number: String, text: String) { /*...*/ }
}
```

# ISP 원칙 적용 주의점
## SRP와 ISP 원칙 사이의 관계
- SRP가 클래스의 단일 책임 원칙이라면, ISP는 인터페이스의 단일 책임 원칙이라고 했다. 즉, 인터페이스에 기능에 대한 책임에 맞게 추상 메소드를 구성하면 된다.
- **BUT)** 책임을 준수하더라도 실무에서는 ISP가 만족되지 않을 수 있는 케이스가 존재한다.

<img src="https://velog.velcdn.com/images/dev-baik/post/d7358a96-e9f2-47ca-8dba-ca84a9065d65/image.png"/>

- 예를 들어 위와 같이 게시판 인터페이스엔 글쓰기, 읽기, 삭제 추상 메서드가 정의되어 있다. 이들은 모두 게시팡판에 필요한 기능들이며 게시판을 이용하는 단일 책침에 위배되지 않는다.
  
- **문제점)** 이를 구현하는 일반 사용자입장에선 게시글 강제 삭제 기능은 사용할 수 없기 때문에 결국 ISP 위반으로 이어진다.
- **HOW)** 책임을 잘 구성해 놓은 것 같지만 실제 적용되는 객체에겐 부합되지 않을 수 있기 때문에 책임을 더 분리해야 한다.

> **ISP는 SRP를 만족하면 성립되는가?** : 반드시 그렇다고 볼 수 없다.

## 인터페이스 분리는 한번만
- 한번 인터페이스를 분리하여 구성해놓고 나중에 무언가 수정사항이 생겨서 또 인터페이스들을 분리하는 행위를 가하지 말라는 점이다.
- **WHY)** 이미 구현되어 있는 프로젝트에 또 인터페이스들을 분리한다면, 이미 해당 인터페이스를 구현하고 있는 온갖 클래스들과 이를 사용하고 있는 클라이언트(사용자)에서 문제가 일어날 수 있기 떄문이다.

> 본래 인터페이스란 건 한번 구성하였으면 왠만해선 변하면 안되는 정책같은 개념이다. 따라서 처음 설계부터 기능의 변화를 생각해두고 인터페이스를 설계해야 하는데, 이는 현실적으로 참 힘든 부분이며 역시 개발자의 역량에 달렸다.

#### 참고한 사이트
- https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EC%95%84%EC%A3%BC-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EB%8A%94-ISP-%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4-%EB%B6%84%EB%A6%AC-%EC%9B%90%EC%B9%99