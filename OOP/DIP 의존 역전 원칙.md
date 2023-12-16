# DIP (Dependency Inversion Principle)
- 객체에서 어떤 Class를 참조해서 사용해야하는 상황이 생긴다면, 그 Class를 직접 참조하는 것이 아니라 그 대상의 상위 요소(추상 클래스 or 인터페이스)로 참조하라는 원칙
- 객체들이 서로 정보를 주고 받을 때는 의존 관계가 형성되는데, 이때 객체들은 나름대로의 원칙을 갖고 정보를 주고 받아야 하는 약속이 있다.

> ✅ **나름대로의 원칙** : 추상성이 낮은 클래스보다 추상성이 높은 클래스와 통신을 하는 것을 의미한다. 추상성이 낮은 경우 세부 구현에 의존성이 높아질 수 있고, 추상성이 높은 경우 보다 유연하고 재사용 가능한 코드를 작성할 수 있다.

```kotlin
// 추상성이 낮은 클래스
class Dog {
    fun bark() {
        println("멍멍")
    }
}

// 추상성이 높은 클래스 또는 인터페이스
interface Animal {
    fun makeSound()
}

// 추상성이 높은 클래스 또는 인터페이스를 사용하는 클라이언트 코드
class AnimalSoundPlayer {
    fun playSound(animal: Animal) {
        animal.makeSound()
    }
}

// Dog 클래스를 Animal 인터페이스에 맞춰 확장
class DogAdapter : Animal {
    private val dog = Dog()

    override fun makeSound() {
        dog.bark()
    }
}
```

> ✅ **클래스 간 의존 관계** : 한 클래스가 어떤 기능을 수행하려고 할 때, 다른 클래스의 서비스가 필요한 경우를 말한다. 대표적으로 A 클래스의 메소드에서 매개변수를 다른 B 클래스의 타입으로 받아 B 객체의 메서드를 사용할 때, A 클래스는 B 클래스와 의존한다고 보면 된다.

- 즉, 클라이언트(사용자)가 상속 관계로 이루어진 모듈을 가져다 사용할 때, 하위 모듈을 직접 인스턴스를 가져다 쓰지 말라는 뜻이다.
- **WHY)** 하위 모듈의 구체적인 내용에 클라이언트가 의존하게 되어 하위 모듈에 변화가 있을 때마다 클라이언트나 상위 모듈의 코드를 자주 수정해야 되기 때문이다.
- **HOW)** 상위의 인터페이스 타입의 객체로 통신하라는 원칙이다.

<img src="https://velog.velcdn.com/images/dev-baik/post/d22da0e8-219f-43b2-8ad3-08f181977e64/image.png"/>

```kotlin
// 인터페이스
interface Toy

class Robot : Toy
class Lego : Toy
class Doll : Toy

// 클라이언트
class Kid {
    var toy: Toy? = null // 합성

    fun setToY(toy: Toy) {
        this.toy = toy
    }

    fun play() {
        println("Kid is playing with ${toy?.javaClass?.simpleName}")
    }
}

// 메인 메소드
fun main() {
    val boy = Kid()

    // 1. 아이가 로봇을 가지고 놀 때
    val robotToy: Toy = Robot()
    boy.setToY(robotToy)
    boy.play() // Kid is playing with Robot

    // ...

    // 2. 아이가 레고를 가지고 놀 때
    val legoToy: Toy = Lego()
    boy.setToY(legoToy)
    boy.play() // Kid is playing with Lego
}
```

> 실제 자바에서 인터페이스에 대해 학습할 때 매개변수로 객체를 받을 때 구체 클래스 타입으로 받는게 아니라, 다형성을 이용해 인터페이스 타입으로 통신하는 것이 좋다고 배웠을 것이다.

> **구체 클래스** : 객체를 생성할 수 있는 클래스

- 대표적으로 컬렉션 프레임워크를 들 수 있는데, 보통 ArrayList나 HashSet 자료형을 인스터스화할 때 변수 타입을 ArrayList, HashSet 같은 구체 클래스 타입으로 선언하는 것이 아닌, List나 Set 같은 인터페이스 타입으로 선언한다.
- 이것도 DIP 원칙을 따른 코드 선언이라고 봐도 무방하다.
```java
// 변수 타입을 고수준의 모듈인 인터페이스 타입으로 선언하여 저수준의 모듈을 할당
List<String> myList = new ArrayList()<>;
    
Set<String> mySet = new HashSet()<>;

Map<int, String> myMap = new HashMap()<>;
```

> **의존 역전 원칙** : 자신보다 변하기 쉬운 것에 의존하던 것을 추상화된 인터페이스나 상위 클래스를 두어 변하기 쉬운 것의 변화에 영향받지 않게 하는 원칙

# DIP 원칙 위반 예제
- RPG 게임에는 캐릭터가 장착할 수 있는 다양한 무기들이 존재한다. 다음과 같이 한손검, 양손검, 전투도끼, 망치 클래스가 있다고 가정하자.
```kotlin
class OneHandSword(val name: String, val damage: Int) {
    fun attack(): Int {
        return damage
    }
}

class TwoHandSword { /*...*/ }

class BattleAxe { /*...*/ }

class WarHammer { /*...*/ }
```
- 무기들을 장착할 Character 클래스
  - Character의 인스턴스 생성 시 OneHandSword에 의존성을 가지게 되어, 공격 동작을 담당하는 attack() 메소드 역시 OneHandSword에 의존성을 가지게 된다.
```kotlin
class Character(
    val name: String, 
    var health: Int, 
    var weapon: OneHandSword // 의존 저수준 객체
) {
    
    fun attack(): Int {
        return weapon.attack()
    }

    fun changeWeapon(newWeapon: OneHandSword) {
        weapon = newWeapon
    }

    fun getInfo() {
        println("이름: $name")
        println("체력: $health")
        println("무기: $weapon")
    }
} 
```
- 하지만 무기엔 한손검 타입만 있는 게 아니다. 여러 무기들을 장착하려면, 캐릭터 클래스 필드 변수 타입을 교체해줘야 한다.
- **문제점** : 이미 완전하게 구현된 하위 모듈을 의존하고 있다.
- **HOW)** : 구체 모듈을 의존하는 것이 아닌 추상적인 고수준 모듈을 의존하도록 리팩토링 하면 된다.
1. 모든 무기들을 포함할 수 있는 고수준 모듈인 Weaponable 인터페이스를 생성한다. 그리고 모든 공격 가능한 무기 객체는 이 인터페이스를 implements 하게 한다.
2. Character 클래스의 기존의 OneHandSword 타입의 필드 변수를 좀 더 고수준 모듈인 Weaponable 인터페이스 타입으로 변경한다.
```kotlin
// 고수준 모듈
interface Weaponable {
    fun attack(): Int
}

class OneHandSword(val name: String, val damage: Int) : Weaponable {
    override fun attack(): Int {
        return damage
    }
}

class TwoHandSword : Weaponable { /*...*/ }

class BattleAxe : Weaponable { /*...*/ }

class WarHammer : Weaponable { /*...*/ }
```
- 게임 시스템 내부적으로 모든 공격 가능한 무기는 Weaponable을 구현하기로 가정했으므로, 공격 가능한 모든 무기를 할당 받을 수 있게 된 것이다.
```kotlin
class Character(val name: String, var health: Int, var weapon: Weaponable) {

    fun attack(): Int {
        return weapon.attack()
    }

    fun changeWeapon(newWeapon: Weaponable) {
        weapon = newWeapon
    }

    fun getInfo() {
        println("이름: $name")
        println("체력: $health")
        println("무기: $weapon")
    }
} 
```
- DIP 원칙을 따름으로써, 무기의 변경에 따라 Character의 코드를 변경할 필요가 없고 또다른 타입의 무기 확장에도 무리가 없으니 OCP 원칙 또한 준수한 것이라고 볼 수도 있다.

#### 참고한 사이트
- https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EC%95%84%EC%A3%BC-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EB%8A%94-DIP-%EC%9D%98%EC%A1%B4-%EC%97%AD%EC%A0%84-%EC%9B%90%EC%B9%99